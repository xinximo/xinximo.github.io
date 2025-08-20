---
title: Linux搭建hexo服务器
date: 2021-11-23 17:28:04
tags: [Linux, hexo, nginx]
---
<meta name="referrer" content="no-referrer"/>

# 一.在自己的服务器上建立git仓库

## 1.首先安装git

```bash
yum update
yum install git -y
```

## 2.nginx安装教程参考我的这篇Docker安装nginx教程:
https://xinximo.com/blog/2021/11/23/Linux%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/

## 3.建立文件夹:

```bash
mkdir ~/software/repo
cd ~/software/repo
git init --bare {自定义仓库名name}.git
```
![Linux搭建hexo服务器图片1.png](images/Linux搭建hexo服务器图片1.png)
<!--more-->
## 4.Git钩子

```bash
vim ~/software/repo/xinximoBlog.git/hooks/post-receive
```

编辑文件:

```bash
#!/bin/bash

git --work-tree=~/software/nginx/www/blog --git-dir=/var/repo/xinximoBlog.git checkout -f
```

给文件赋予执行权限:

```bash
chmod +x ~/software/repo/xinximoBlog.git/hooks/post-receive
```

## 5.使用Git部署本地hexo到远端服务器

```bash
git clone root@{云服务器IP}:~/software/repo/xinximoBlog.git
```

## 6.编辑**站点**配置文件`_config.yml`

将 url 改成`https://{云服务器IP}/`

```bash
#url: https://xinximo.github.io/
url: https://{ip}/blog/
```

将 deploy 目标改为 `{服务器用户名}@{服务IP}:~/software/repo/xinximoBlog.git`：

```bash
deploy:
  type: git
  #repo: git@github.com:xinximo/xinximo.github.io.git
  repo: root@{ip}:~/software/repo/xinximoBlog.git
```

在本地hexo目录下,执行部署命令:

```bash
hexo clean && hexo deploy
or
hexo clean && hexo g -d
```
