---
title: nvm的使用
date: 2019-09-18 15:08:31
categories: 开发小工具
---

## 前言

>在使用`windows`和`macOS`的时候没有感觉到`node`版本的有什么难处理的，但是当使用`linux`系统的时候，发现安装`node`是个比较麻烦的事情。比如：如果`ubuntu`自带的`apt-get`来安装，只能安装比较低版本的，如果想要升级版本也是个挺麻烦的事情。这个时候`node`版本管理神器`nvm`的作用就体现出来了，可以随意的下载各个版本的`node`，并且可以随时进行版本的切换。
<!-- more -->

## 安装nvm

mac可以使用`brew`安装

```bash
brew install nvm
```

ubuntu可以使用`curl`安装

```bash
# 使用curl安装的nvm，需要在安装完成后重新打开命令行
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
```

## nvm操作

下载最新版`node`

```bash
nvm install stable
```

下载指定版本

```bash
nvm install 版本号
```

查看已经下载的`node`版本列表

```bash
nvm ls
```

切换`node`版本

```bash
nvm use 版本号
```

修改默认版本（如果不修改默认版本，可能会导致关闭当前命令行后`node`版本又恢复为低版本）

```bash
nvm alias default 版本号
```

***
>具体`nvm`操作方法，请参考[官方文档](https://github.com/nvm-sh/nvm#usage "nvm官方文档")
