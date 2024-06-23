---
title: 服务器软件备忘录--Caddy
tags: [Caddy]
date: 2022-04-30T16:44:21+08:00
---

Caddy是一个高性能的web服务器, 采用go语言编写, 类似nginx的, 一个显著的特性是默认启用 HTTPS. Caddy2相对于Caddy1在配置上有诸多变动. 以下内容，如无特殊说明，均指Caddy2版本.

## 安装

推荐使用社区提供的一键安装方法, 更多官方安装方式 https://caddyserver.com/docs/install

```bash
curl -sS https://webinstall.dev/caddy | bash
```

安装之后可以通过 whereis caddy 找到安装目录, 配置文件在/etc/caddy/Caddyfile

## 常用命令

```bash
caddy           # 显示帮助信息
caddy run       # 前台运行
caddy start     # 后台运行
caddy stop      # 停止服务
caddy reload    # 重新加载服务
```

## 静态Web服务

最简单的配置

```
example.com
respond "Hello, privacy!"
```

通过端口访问配置

:2015 {
	root * /var/www/www.website.com/
	file_server
}

## 反向代理服务

```
:2016 {
    reverse_proxy 127.0.0.1:9000
}
```

## HTTPS配置

默认为所有的站点开启https，默认使用Let’s Encrypt, 在配置文件中指定申请证书的邮箱 tls helloworld@gmail.com
也可以设置不自动获取ssl证书，通过tls指定ssl证书 

```
example.com {
  root * /var/www/www.website.com/
  file_server
  tls cert.pem key.pem
}
```

## 日志

下面是一个简洁的日志配置, 甚至可以只写log

```
log {
	output file /var/log/access.log
}
```

下面是一个完整的日志配置

```
example.com {
    log {
	   output file <filename> {
    	   roll_disabled
    	   roll_size     <size>
    	   roll_uncompressed
    	   roll_local_time
    	   roll_keep     <num>
    	   roll_keep_for <days>
      }
	   format <encoder_module> ...
	   level  <level>
    }
}
```

output 配置输出目标, stdout(默认输出到console), stderr, file, discard(不输出日志),
format 格式化日志内容, 
level 自定义日志级别, INFO(默认)、ERROR

其中，output可配置项说明

filename 日志文件路径及名称.   
roll_disabled 不分割轮转日志文件，保持一个日志文件.      
roll_size 日志文件大小, 以M为单位, 1.1MiB 会四舍五入到 2MiB. 默认: 100MiB.   
roll_uncompressed 文件压缩. 默认: 开启gzip压缩.   
roll_local_time 设置应用到文件名时间格式. 默认: UTC时间格式.   
roll_keep 最多保持多少个日志文件. 默认: 10.   
roll_keep_for 日志保持时长，单位是h，但是会四舍五入到整天. 例如, 36h (1.5 days) 会四舍五入到 48h (2 days). 默认: 2160h (90 days).   

## 单页应用配置(React Vue...)

Caddy并不了解单页应用的路由组件，在页面刷新时，默认会去找路由做对应的资源文件，但是资源文件并没有，这时候就会报404，所以我们要把请求重定向到首页

```
try_files {path} /
```

参考   
https://router.vuejs.org/guide/essentials/history-mode.html#caddy-v2
https://caddyserver.com/docs/caddyfile/patterns#single-page-apps-spas