---
title: nginx配置https证书
date: 2020-03-25 11:06:18
categories: 服务器配置问题
---

# 使用的证书颁发机构

使用的`Let's Encrypt`颁发机构，该机构颁发免费的https证书并且被大多数厂商认可，可以去他的官方[文档](https://letsencrypt.org/)查看具体信息（有中文的哦）。同时使用官方推荐的ACME协议软件`certbot`获取证书，其官网可以根据服务器使用的系统和软件给出相应的安装使用说明，详情见[这里](https://certbot.eff.org/)。

# 获取证书

在获取证书之前，按照官网的安装文档一般都没有问题。如果到了获取证书这一步，即执行`sudo cretbot --nginx`不成功，在`/etc/letsencrypt/live/`文件夹下没有以你域名为名字的文件夹，那么就表示获取证书失败，请按照报错提示百度解决。

如果一直没有办法成功获取证书，建议或一种模式获取。一般有`webroot`和`standalone`两种模式，我选择了后者，他不需要指定网站根目录，自动的启用443端口来验证，下面就是使用`standalone`这种模式获取证书。

> 在获取证书之前你需要停止nginx，否则它无法启用443端口

```shell
certbot certonly --standalone -d 域名
```

如果没有报错，并且在`/etc/letsencrypt/live/域名/`文件夹下出现了几个以`.pem`结尾的文件就表示证书获取成功了。

# nginx配置

```shell
    listen         443;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/域名/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/域名/privkey.pem;
```

设置完毕后重启nginx即可。

> 注意：在配置ssl_certificate选项的时候，有的文章是配置的cret.pem这个文件，经过验证有些浏览器不认可这个证书，通过阅读它的REDME也可以得知，所以最好使用fullchain.pem这个证书。

# 自动续期

Let‘s Encrypt的证书一般是3个月的有效期，为了保证服务不会中断，一般我们都要配置自动续期。不过certbot可以帮我们自动续期，所以不用我们自己去crontab中自己写脚本。

因为续订的时候需要占用80端口，所以为了避免端口冲突导致续订失败，我们可以把nginx默认的80端口修改为其他端口。

```shell
# 开启自动续订
certbot renew --quiet --no-self-upgrade

# 测试自动续订是否可用
certbot renew --dry-run

# 成功会有如下提示
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/liluyang.me.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator webroot, Installer None
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for liluyang.me
http-01 challenge for www.liluyang.me
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed without reload, fullchain is
/etc/letsencrypt/live/liluyang.me/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/liluyang.me/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -


```

>注意：如果进行手动续订的时候报错The following certs are not due for renewal yet，说明证书还没到期不用续订。
