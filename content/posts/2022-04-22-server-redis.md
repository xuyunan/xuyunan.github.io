---
title: 服务器软件备忘录--Redis
tags: [Redis]
date: 2022-04-22T22:55:04+08:00
---

一个基于内存的key-value存储系统

Redis 官网：https://redis.io/   
Redis 在线测试：http://try.redis.io/   
Redis 命令参考：http://doc.redisfans.com/   

## 常用命令

```bash
systemctl start redis       # 启动服务
systemctl stop redis        # 停止服务
systemctl restart redis     # 重启服务
systemctl enable redis      # 设置开机自启动
systemctl status redis      # 查看服务状态
ps -ef | grep redis         # 查看redis进程
netstat -lnp|grep 6379      # 查看端口
```

## 安装

```bash
yum install epel-release
yum install redis
```

## 初始化配置

vi /etc/redis.conf

```
# 设置允许远程连接, 注释掉即可, 默认只允许本机连接
# bind 127.0.0.1
# 将保护模式关闭，默认是yes
protected-mode no
# 为了安全着想,可以设置密码
requirepass xxxx
```

安全组需要开放6379端口 