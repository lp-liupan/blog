# 在其他计算机上搭建当前博客开发步骤

## 前提

配置好`hexo`环境，参考`hexo`[文档](https://hexo.io/zh-cn/docs/ "hexo文档")

## 下载

```bash
git clone https://github.com/lp-liupan/blog.git

cd blog

npm install
```

## 配置

下载主题

```bash
git clone https://github.com/theme-next/hexo-theme-next.git themes/next
```

修改配置文件

```bash
rm -rf themes/next/_config.yml

cp _config.yml.example themes/next/_config.yml
```