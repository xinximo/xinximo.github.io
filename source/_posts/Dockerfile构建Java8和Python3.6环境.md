---
title: Dockerfile构建Java8和Python3.6环境
date: 2021-11-01 17:24:08
tags: [Docker, Linux]
---
# 一. Dockerfile构建脚本:
```
# docker pull xinximo/java8-python3.6
# Author: xinximo
# Version: 2021-10-27


FROM python:3.6-alpine

MAINTAINER xinximo "woshiwangxin123@126.com"

ENV LANG C.UTF-8
RUN { \
        echo '#!/bin/sh'; \
        echo 'set -e'; \
        echo; \
        echo 'dirname "$(dirname "$(readlink -f "$(which javac || which java)")")"'; \
    } > /usr/local/bin/docker-java-home \
    && chmod +x /usr/local/bin/docker-java-home
ENV JAVA_HOME /usr/lib/jvm/java-1.8-openjdk
ENV PATH $PATH:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin

ENV JAVA_VERSION 8u282
ENV JAVA_ALPINE_VERSION 8.282.08-r1

RUN set -x \
    && apk add --no-cache openjdk8="$JAVA_ALPINE_VERSION" \
    && [ "$JAVA_HOME" = "$(docker-java-home)" ]
<!--more-->
# 工作目录
WORKDIR /app

# 解决时区问题
ENV TZ "Asia/Shanghai"

# 终端设置
# 默认值是dumb，这时在终端操作时可能会出现：terminal is not fully functional
ENV TERM xterm

#pythonpath
ENV PYTHONPATH="/app:$PYTHONPATH"
```
# 二. 拉取镜像并运行
注意:因为在Mac和Linux系统镜像构建时的核心有区别,会导致互相运行时概率报错,请使用对应的镜像: \
Mac下运行:
```
docker pull xinximo/java8-python3.6
docker run -it --rm --name xinximo/java8-python3.6 xinximo/java8-python3.6 sh
```
Linux下运行:
```
docker pull xinximo/java8-python3.6-linux
docker run -it --rm --name xinximo/java8-python3.6-linux xinximo/java8-python3.6-linux sh
```

