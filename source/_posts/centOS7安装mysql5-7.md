---
title: centOS7安装mysql5.7
date: 2019-12-17 16:30:12
categories: 服务器配置问题
---

# 介绍

centos安装软件基本上是两种方式，一种是RPM二进制包的方式，另一种tarble（源码）的方式。源码安装软件的方式可以参考[centOS安装php](https://www.xiaotaotao.vip/2019/12/10/centOS%E5%AE%89%E8%A3%85php/#more)。这里安装mysql5.7主要就是使用RPM二进制包的方式。

# 下载RPM包

推荐去mysql官网的[yum库下载](https://dev.mysql.com/downloads/repo/yum/)与系统匹配的包，右键复制下载链接，然后去下载目录下载。

```bash
cd /usr/local/src/

#使用wget下载
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

# yum本地安装（配置源）

```bash
#在下载目录执行`yum localinstall RPM包名`
yum localinstall  mysql80-community-release-el7-3.noarch.rpm
```
# 选择安装版本（可选）

通过下载的RPM包名就可以看出，我们当前默认安装的话MySQL的版本就是8.0的，如果不需要更改安装版本，直接`yum install mysql-community-server`下载就可以了。而我们要安装5.7版本的，这时候就需要修改yum的配置文件，选择我们需要的版本。

```bash
#修改配置文件
vim /etc/yum.repos.d/mysql-community.repo 


#可以看到8.0版本中的enable选项值为1，如果需要改为5.7版本，
#需要将8.0版本中的enable选项值改为0,5.7版本的enable值改为1

[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

修改好了配置文件后，再使用`yum install mysql-community-server`安装MySQL就是5.7版本。

# 启动MySQL服务

使用RPM包安装的MySQL已经默认配置了systemctl，直接使用systemctl启动就可以了。

```bash
#启动
systemctl start mysqld

#关闭
systemctl stop mysqld
```

>注意：如果原来安装过其他版本的MySQL，这里启动的时候可能会报错，自行到`/var/log/mysqld.log`日志文件中查看，然后百度吧。

# 修改密码

MySQL默认创建了root用户，密码是随机生成的，需要到日志文件中查看。进入数据库系统后，第一时间改掉root密码。

```bash
#获取root密码
cat /var/log/mysqld.log | grep "temporary password"

#登录
mysql -u root -p

#修改密码
SET PASSWORD = PASSWORD('密码（有复杂度验证，简单密码会报错）');

```