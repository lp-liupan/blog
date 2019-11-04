---
title: 服务器环境安装node
date: 2019-11-04 17:05:59
categories: 服务器配置问题
---

# 下载node

前往[官网](https://nodejs.org/zh-cn/download/)下载Linux版的[二进制文件](https://nodejs.org/dist/v12.13.0/node-v12.13.0-linux-x64.tar.xz)。注意区别有两种后缀名不同的文件，一个是`.tar.xz`，另一个是`.tar.gz`。这里我使用的是`.tar.xz`。      

复制好下载连接后到服务器存放下载文件的文件夹中下载：
```bash
wget https://nodejs.org/dist/v12.13.0/node-v12.13.0-linux-x64.tar.xz
```

# 安装

先解压文件
```bash
tar xvf node-v12.13.0-linux-x64.tar.xz
```

cd到可执行node命令的目录中
```bash
cd node-v12.13.0-linux-x64/bin
```

测试是否安装好node
```bash
./node -v
```
如果看到了版本信息，则说明node已经安装好了。

# 全局使用

node虽然安装好了，但是发现只能在刚才指定的文件夹中使用，实际开发中肯定需要在其他地方也可以使用node命令，这时候我们就需要把node命令用软链接的方式放到全局，保证在其他地方也可以使用node命令。
```bash
ln 安装node包的局对地址/node-v12.13.0-linux-x64/bin/node /usr/bin/
```

目前我们只对node进行了软链接，接下来还需要对npm进行软链接
```bash
ln 安装node包的局对地址/node-v12.13.0-linux-x64/bin/npm /usr/bin/
```

软链接成功后，可以在任意目录中使用`node -v`或者`npm -v`，如果看到了相应的版本信息，则表明node全局配置成功。
