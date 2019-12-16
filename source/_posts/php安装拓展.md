---
title: php安装拓展
date: 2019-12-16 17:23:55
categories: PHP
---

# 安装方式

PHP拓展的安装方式主要有两种，一种是直接使用PECL拓展管理工具安装，另一种是使用源码安装。因为PECL存在一些问题，所以一般比较推荐使用源码安装的方式。这里也是使用源码安装PHP拓展的完整记录。

# 下载解压

1. 打开[PECL](https://pecl.php.net/)网址，直接搜索你需要下载的拓展包。

2. 选择状态为stable的最新版，然后右键复制后面压缩包的下载链接。

3. 下载
```bash
cd /usr/local/src

wget 下载链接

#解压
tar -zxvf 压缩包名称
```

# 安装

```bash
cd 解压后的文件夹

#挂载到php，成功后会生成configure文件
phpize

#检查编译环境，配置编译参数
./configure

#编译
make

#安装到预定文件
make install
```

# 配置

找到php的配置文件（一般我都会安装在`/usr/local/php7.3/lib`这个文件夹下面）

进入`php.ini`文件，添加`extension=拓展名`选项。

重启php（记得带上配置文件`php-fpm -c /usr/local/php7.3/lib/php.ini`）。

```bash
#查看拓展是否已经安装成功
php -m
```

# 注意

如果`php -m`查看没有发现新安装的拓展，一般是下面几种情况：

1. 重启php的时候没有指定配置文件；
2. 重启php的时候警告没有找到拓展，一般是因为你拓展的安装位置有问题。可以修改php.ini文件的`extension_dir`选项指定拓展安装的位置；



