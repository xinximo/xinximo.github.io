---
title: 'Dockerfile中安装git,区分alpine和非alpine环境'
date: 2021-11-01 17:41:20
tags: [Docker, alpine]
---
# 安装配置
alpine环境安装:
```
# install git - apt-get replace with apk
RUN apk update && \
    apk upgrade && \
    apk add --no-cache bash git openssh
```
非alpine环境安装:
```
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y git
```

