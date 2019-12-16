---
title: centOS7安装php
date: 2019-12-10 16:17:09
categories: 服务器配置问题
---

# 下载

>这里使用源码安装的方式，最好是直接去[官网](https://www.php.net/downloads.php)下载压缩包。选择合适的版本，右键复制链接地址。

```bash
#源码下载文件夹
cd /usr/local/src/

#使用wget下载（刚才复制的链接）
wget https://www.php.net/distributions/php-7.4.0.tar.xz

#解压
tar -Jxvf php-7.4.0.tar.xz

cd php-7.4.0
```

# 编译配置

>进入解压的文件夹（php-7.4.0）后需要检查编译环境，然后生成`Makefile`文件。在检查编译环境过程中可能会出现错误，主要是要保证检查后没有`error`，如果检查后有错误，则不能生成`Makefile`文件。

>`./configure`的`--prefix=/usr/local/php7.3 `是确定安装文件夹，`--with-config-file-path=/usr/local/php7.3/etc`是确定php运行配置文件的文件夹，这两个参数最好都使用，如果没有指定文件夹，最终会按照默认的方式安装，对整体的管理比较麻烦。

>`./configure`的参数详解自行百度了解。

```bash
#配置编译参数，检查编译环境是否符合
./configure --prefix=/usr/local/php7.3 --with-config-file-path=/usr/local/php7.3/lib --with-curl --with-freetype-dir --with-gd --with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --with-libxml-dir --with-openssl --with-pcre-regex --with-pdo-mysql --with-pdo-sqlite --with-pear --with-png-dir --with-xmlrpc --with-xsl --with-zlib --with-mhash --with-jpeg-dir --enable-fpm --enable-bcmath --enable-libxml --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-xml --enable-zip --enable-ftp --enable-intl --enable-mysqlnd --disable-rpath --disable-fileinfo
```

一般执行上面代码后都会出现找不到某个包的错误，因为配置参数中设置了php支持的功能需要下载包。主要是有以下几个问题，如果出现了其他问题处理方式类似。总之就是要保证执行完上述命令后不能有错误。

```bash
#1、
#报错找不到某个包(会出现多次这种类型的错误，解决方法都一样)
#configure: error: libxml2 not found. Please check your libxml2 installation.

yum install libxml2-devel




#2、
#有的包版本过低，不能通过yum下载了，需要用源码安装
#checking for libzip... configure: error: system libzip must be upgraded to version >= 0.11

yum remove libzip #删除旧版本

cd /usr/local/src #返回到源码下载目录

wget https://libzip.org/download/libzip-1.5.2.tar.xz #下载新版本

tar -Jxvf libzip-1.5.2 #解压

cd libzip-1.5.2

./configure #检查编译环境（举例的这个包比较特殊，不能用`./configure`命令，具体安装方式查看INSTALL.md文件）

make #编译

make install #安装到指定文件夹

cd /usr/local/src/php-7.4.0 #返回php文件夹继续执行`./configure ....`检查执行环境



#3、
#off_t undefind报错
#configure: error: off_t undefined; check your library configuration

# 添加搜索路径到配置文件
echo '/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64'>>/etc/ld.so.conf

ldconfig -v #更新配置

```

# 编译

>在确定检查编译没问题，并且生成了`Makefile`文件，这时候就可以进行编译了。

```bash
#编译
make

#安装到指定文件(具体安装到哪里根据`./configure`的参数确定)
make install

```

# 配置

>这里只针对`./configure`设置了`--prefix`和`--with-config-file-path`参数的情况进行说明。

```bash
# 进入php安装文件夹
cd /usr/local/php7.3

# 进入配置文件夹
cd /etc

# 使用默认配置（具体配置百度）
cp php-fpm.conf.default php-fpm.conf

cd php-fpm.d

cp www.conf.default www.conf

```

>环境变量设置（可选）

```bash
# 对profile文件进行编辑
vim /etc/profile

# 在profile文件的最后加上
exprot PATH=$PATH=:/usr/local/php7.3/bin:/usr/local/php7.3/sbin

# 重新读取文件
source /etc/profile
```


# 启动

```bash
# 如果配置了环境变量
php-fpm

# 如果没有配置环境变量
/usr/local/php7.3/sbin/php-fpm

# 查看是否启动成功
ps aux | grep php-fpm
```