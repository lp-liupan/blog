---
title: php上传大文件配置
date: 2019-08-16 09:37:34
categories: 服务器配置问题
---

# PHP配置问题

## PHP上传文件大小限制
>PHP配置文件默认的post上传数据`post_max_size`大小为2M，默认上传文件`upload_max_filesize`的大小为8M。所以如果文件大小超过了默认的最大值就会出现上传失败的问题。
<!-- more -->

## PHP最长执行时间限制
>大文件上传失败的另一个原因就是PHP执行时间的限制问题。因为大文件的上传比较慢，而PHP默认的最长执行时间`max_execut_time`为30秒，如果超过了30秒脚本就会停止执行。

## PHP配置文件的修改
>PHP的配置文件一般位于`/usr/local/php7/lib/php.ini`，用vim编辑器打开，进入文件后找到`post_max_size`这个变量，然后将数值修改为合适的大小。同样的将`upload_max_filesize`变量也改为合适的大小。执行时间的变量`max_execut_time`的值改为`0`表示没有时间限制。

## 重启PHP
>修改了PHP的配置文件`php.ini`后需要对PHP进行重启，具体如何重启自行谷歌。我这里用的是最简陋的方法，先关闭`killall php-fpm`，然后在`/usr/local/php7/sbin`文件夹下启动`/usr/local/php7/sbin/php-fpm –c ../lib/php.ini –y ../etc/php-fpm.conf `。

>如果在页面报错`502 BadGateway`或者`Gateway Timeout`则表示`php-fpm`没有启动。

# nginx配置问题

## nginx设置上传文件最大值
>如果上述两总情况都发现没有问题则可以去nginx的配置文件看一下`client_max_body_size`变量，这个变量是用来设置上传文件大小的，自行设置好合适的值即可。我设置的是`client_max_body_size 500m;`。

## 重启nginx
>根据nginx安装的不同，自行谷歌重启nginx即可。我在项目中使用的是`/usr/local/openresty/nginx/nginx/sbin/nginx -s reload`。