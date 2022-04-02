---
title: 对于NGINX的日志文件大小控制
tags: [nginx]
date: 2022-01-01T13:00:24+08:00
---

## 问题

新年伊始，系统就报起了磁盘即将被占满的警告，经排查是原因为nginx日志文件access.log文件太大了，临时先清空下文件 `echo "" > access.log`  

nginx有两个日志文件，一个是access.log，会记录每次成功的请求信息，包括ip地址，时间，浏览器等信息，格式可以通过在nginx.conf中的log_format配置；一个是error.log，会记录异常的信息。 关于磁盘大小报警，如何排查，可以通过du命令，可以参考 [How to Find Out Top Directories and Files (Disk Space) in Linux](https://www.tecmint.com/find-top-large-directories-and-files-sizes-in-linux/)

<!--truncate-->

## 如何解决

通过logrotate来配置记录日志的策略，logrotate是基于crontab定时任务的程序，一般的linux系统都已经默认安装了。 
建立/etc/logrotate.d/nginx配置文件，内容如下

```bash
/usr/local/nginx/logs/access_log {
    rotate 7
    size 5k
    dateext
    dateformat -%Y-%m-%d
    missingok
    compress
    sharedscripts
    postrotate
        test -r /usr/local/nginx/logs/nginx.pid && kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
    endscript
}
```

其中:  
rotate 7, 表示一次将存储7个归档日志。对于第8个归档，时间最久的归档将被删除
postrotate, 最通常的作用是让应用重启，以便切换到新的日志文件，注意并不是nginx重启。

配置好之后，添加定时任务去执行

```bash
crontab  -e  #添加以下代码
0 0 * * * /usr/sbin/logrotate -vf /etc/logrotate.d/nginx  #每天凌晨定时执行脚本 需要放到root用户下执行
```


## 参考

[How to limit nginx access log file size and compress?](https://serverfault.com/questions/427144/how-to-limit-nginx-access-log-file-size-and-compress)  
[Nginx Log Rotation Doesn't Seem to Be Working Correctly](https://unix.stackexchange.com/questions/259091/nginx-log-rotation-doesnt-seem-to-be-working-correctly)  
[nginx配置logrotate 配置](https://www.cnblogs.com/zhaoyingjie/p/14649006.html)  


