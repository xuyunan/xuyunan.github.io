---
title: 服务器软件备忘录--MySQL
tags: [MySQL]
date: 2022-04-22T20:53:01+08:00
---

有些项目是需要特定版本的 mysql 数据库，这里记下 mysql5.7 安装的过程和一些基础命令  

## 推荐客户端

mac client: https://sequelpro.com/     
windows client: https://www.heidisql.com/   

## 常用命令

```bash
mysql -V  # 查看mysql版本 	
systemctl start mysqld      # 开启服务   		
systemctl restart mysqld    # 重启服务		
systemctl stop mysqld       # 停止服务		
systemctl enable mysqld     # 开启启动		
systemctl disable mysqld    # 禁止开机启动		
systemctl status mysqld     # 查看mysql状态		
```

## 卸载已安装的 mysql

卸载已安装的mysql   
sudo yum remove mysql mysql-server    # CentOS and RedHat systems 

删除mysql数据目录并备份

```bash
mv /var/lib/mysql /var/lib/mysql_old_backup
mv /etc/mysql /etc/mysql_old_backup
```

看看还有哪些mysql文件, 找出来删掉   
find / -name mysql

删除已经安装的mysql源 

```bash
rpm -qa|grep mysql
yum remove mysql*
rpm -e mysql57-community-release-el7-10.noarch
```

## CentOS 7 安装 mysql5.7

更新 yum   
yum update -y

打开网站 https://dev.mysql.com/downloads/repo/yum/

登录之后，选择 CentOS 7 版本的源，右键复制链接 https://dev.mysql.com/get/mysql80-community-release-el7-5.noarch.rpm

安装 wget

```bash
sudo yum install -y wget
cd /download
wget https://dev.mysql.com/get/mysql80-community-release-el7-5.noarch.rpm
```

下载好之后安装
sudo yum localinstall mysql80-community-release-el7-5.noarch.rpm

安装 yum 工具 yum-utils   
sudo yum install -y yum-utils

查看可用的 mysql 源   
yum repolist enabled | grep "mysql.*-community.*"

列出所有版本   
yum repolist all | grep mysql

禁用默认的最新8.0版本   
yum-config-manager --disable mysql80-community

启用5.7版本   
yum-config-manager --enable mysql57-community

查看启用版本是否为5.7   
yum repolist enabled | grep mysql

开始安装   
yum install -y mysql-community-server

启动 mysql 服务   
systemctl start mysqld

查看 mysql 状态   
systemctl status mysqld

## 初始化 mysql

初始化配置, 设置初始密码   
sudo grep 'temporary password' /var/log/mysqld.log

使用上一步得到的密码，做初始化操作，初始化数据库配置, 修改root密码，配置是否禁止root远程登录   
mysql_secure_installation   # /usr/bin/mysql_secure_installation

如果还不能远程登录, 做以下设置   
先登录, mysql -u root -p

```mysql
SHOW databases;
USE mysql;
UPDATE user SET host='%' WHERE user='root';
FLUSH PRIVILEGES;
```

参考   
[A Quick Guide to Using the MySQL Yum Repository](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/#repo-qg-yum-installing)   
[Host 'xxx.xx.xxx.xxx' is not allowed to connect to this MySQL server](https://stackoverflow.com/questions/1559955/host-xxx-xx-xxx-xxx-is-not-allowed-to-connect-to-this-mysql-server)   