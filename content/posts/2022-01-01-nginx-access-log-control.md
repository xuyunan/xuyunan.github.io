---
title: 对于NGINX的日志文件大小控制
tags: [nginx]
date: 2022-01-01T13:00:24+08:00
---

## 问题

新年伊始，系统就报起了磁盘即将被占满的警告，经排查是原因为nginx日志文件access.log文件太大了，临时先清空下文件 `echo "" > access.log`  

nginx有两个日志文件，一个是access.log，会记录每次成功的请求信息，包括ip地址，时间，浏览器等信息，格式可以通过在nginx.conf中的log_format配置；一个是error.log，会记录异常的信息。 关于磁盘大小报警，如何排查，可以通过du命令，可以参考 [How to Find Out Top Directories and Files (Disk Space) in Linux](https://www.tecmint.com/find-top-large-directories-and-files-sizes-in-linux/)

<!--truncate-->

## 解决方式:logrotate

logrotate是一个可以分割、压缩日志文件的工具

使用方式是在特定目录建立(/etc/logrotate)一个配置配置文件，通过配置项来管理日志文件，比如:
mysql日志配置文件 /etc/logrotate.d/mysql
nginx日志配置文件 /etc/logrotate.d/nginx

例如ngxin而配置之后，产生的access日志文件

access.log
access.log-20220217.gz
access.log-20220218.gz
access.log-20220219.gz
access.log-20220220

常用的三个命令选项 -f、-d、-v

logrotate通常是要通过cron任务来执行的，但是如果你现在就想看看转存后的文件，可以通过-f来强制执行一次转存。
logrotate -f /etc/logrotate.d/nginx

如果你想测试下写的配置是否正常，可以使用-d来测试，它不会真的去切割文件，而是会以日志输出的形式，把过程打印出来。
logrotate -d /etc/logrotate.d/nginx

如果你好奇转存时logrotate都做了哪些工作，可以使用-v来输出详细过程
logrotate -v /etc/logrotate.d/nginx

触发logrotate的方式有多种：

配置了size，文件到达具体大小时会触发分割；
daily、weekly等时间节点会触发分割；
通过-f强制执行分割；

配置文件关键点分两个部分，一个是日志文件的路径，一个是配置项。两个示例: 

一个是nginx日志配置，每天检查日志文件，会保留7个分割的日志文件，然后会执行一个脚本(postrotate后面一行)，来通知nginx，重新写入日志，这样就不会写到旧的日志文件里去。

```
/usr/local/nginx/logs/*.log  {
    daily
    rotate 7
    size 5k
    missingok
    dateext
    compress
    delaycompress
    notifempty
    sharedscripts
    postrotate
        [ -e /usr/local/nginx/logs/nginx.pid ] && kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
    endscript
    }
```

这个是我们自己的日志配置，每天检查日志文件，总共会保留3个分割的日志文件，没有脚本执行。
由于没有单独的管控写日志的进程，所以使用copytruncate来控制日志，让其先拷贝，然后在清空现有的日志文件。
如果你不加脚本处理，又不添加copytruncate来控制，在日志分割时，logrotate只会重命名日志文件，然后应用程序继续往重命名后的文件写日志，因为linux是通过inode标示来写入文件，而不是文件名，文件名只是在打开、寻找文件时起作用。

```
/root/go/logs/*.log  {
    daily
    rotate 3
    missingok
    dateext
    compress
    delaycompress
    notifempty
    copytruncate
}
```

## 一些配置项的作用

/root/go/logs/*.log 日志路径
daily: 日志文件分割频度。可选值为 daily，monthly，weekly，yearly
rotate 3: 一总共会保留3个分割的日志文件
missingok: 找不到日志文件，不会报错
dateext 追加日期作为文件名
compress: 使用gzip进行压缩。
nocompress: 如果你不希望对日志文件进行压缩，设置这个参数即可
delaycompress: 下次执行任务时，压缩本次的日志文件
notifempty: 如果日志文件为空，轮循不会进行。
sharedscripts 表示postrotate脚本在压缩了日志之后只执行一次
create 644 www root: 以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件
postrotate/endscript: 执行的脚本，最通常的作用是让应用重启，以便切换到新的日志文件

>配置好之后，添加定时任务去执行

```bash
crontab  -e  #添加以下代码
0 0 * * * /usr/sbin/logrotate -vf /etc/logrotate.d/nginx  #每天凌晨定时执行脚本 需要放到root用户下执行
```


## 参考

[How to limit nginx access log file size and compress?](https://serverfault.com/questions/427144/how-to-limit-nginx-access-log-file-size-and-compress)  
[Nginx Log Rotation Doesn't Seem to Be Working Correctly](https://unix.stackexchange.com/questions/259091/nginx-log-rotation-doesnt-seem-to-be-working-correctly)  
[nginx配置logrotate 配置](https://www.cnblogs.com/zhaoyingjie/p/14649006.html)     
[logrotate机制和原理](http://www.lightxue.com/how-logrotate-works)   
[HowTo: The Ultimate Logrotate Command Tutorial with 10 Examples](https://www.thegeekstuff.com/2010/07/logrotate-examples/)
[logrotate(8) - Linux man page](https://linux.die.net/man/8/logrotate)   
[linux环境下使用logrotate工具实现nginx日志切割](https://zhuanlan.zhihu.com/p/24880144)   


