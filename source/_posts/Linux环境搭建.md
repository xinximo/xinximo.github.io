---
title: Linux环境搭建
date: 2021-11-23 17:48:13
tags: [Linux, mysql]
---
## 一. linux中安装MySQL

1.新建本地文件夹

```bash
mkdir -p  ~/software/mysql/conf  ~/software/mysql/logs  ~/software/mysql/data

```

2.远程拉取MySQL

```bash
docker pull mysql

```

3.运行一个实例

```bash
docker run -it -d  --name mysql -e MYSQL_ROOT_PASSWORD=123456 mysql

```
<!--more-->
4.复制该实例的配置文件到 server 当中：

```bash
docker cp mysql:/etc/mysql/conf.d ~/software/mysql/conf

```

5.删除旧的实例:

```bash
docker rm -f mysql

```

6.重新创建一个 mysql 容器：

```bash
docker run -d \\
--name mysql \\
-p 3306:3306 \\
-v ~/software/mysql/conf:/etc/mysql/conf.d \\
-v ~/software/mysql/logs:/logs \\
-v ~/software/mysql/data:/var/lib/mysql \\
-e MYSQL_ROOT_PASSWORD=123456 \\
--restart=always \\
mysql

```

7.查看是否创建成功

```bash
docker ps

```

## 二. 修改MySQL配置支持远程连接

1.进入数据库,输入密码

```bash
docker exec -it mysql  mysql  -uroot -p

```

2.设置远程连接,切换到 mysql 数据库：

```bash
use mysql;

```

3.修改数据库 user 表进行远程连接：

```bash
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

```

4.刷新修改：

```bash
flush privileges;

```

## 三. 安装wordpress

1.安装

```bash
远程仓库拉取 wordpress:
docker pull wordpress

先运行一个 wordpress 实例：
docker run -it -d --name wordpress --link mysql:mysql -p 9001:80 wordpress

复制现有的 wordpress  文件到当前 server ：
docker cp wordpress:/var/www/html  ~/software/wordpress

删除刚安装的容器：
docker rm -f wordpress

重新安装 wordpress 容器：

docker run -it -d --name wordpress \\
-e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root \\
-e WORDPRESS_DB_PASSWORD=123456 -e WORDPRESS_DB_NAME=myword \\
-p 9001:80 -v ~/software/wordpress/:/var/www/html \\
--link mysql:mysql \\
wordpress

docker run -it -d --name wordpress -p 9001:80 \\
-v ~/software/wordpress/:/var/www/html \\
--link mysql:mysql \\
wordpress

```

2.都这里我们的网站就出来了，下面打开浏览器输入：
[http://你自己server的ip:9001](http://xn--serverip-im2m261jmr0diu2a:9001/)

3.根据上面的步骤操作一个网站也就出来了，但是使用 wordpress 上传图片和视频的时候是限制大小的，如何配置大小呢？
修改 ~/software/wordpress 文件夹下面的 .htaccess 文件，打开增加如下两行：

```bash
RewriteEngine On
RewriteBase /
RewriteRule ^index\\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
#增加如下两行配置上传文件的大小
php_value post_max_size 24M
php_value upload_max_filesize 8M

```

## 四. 安装 nginx

1.安装

```bash
远程仓库拉取 nginx：
docker pull nginx

先运行一个 nginx 的容器：
docker run -it -d --name nginx  nginx

复制配置文件：
docker cp nginx:/etc/nginx/ ~/software/

删除 nginx 的容器：
docker rm -f nginx

重新建立一个 nginx 的容器：
docker run -it -d \\
--name nginx \\
-v ~/software/nginx/:/etc/nginx/ \\
-v ~/software/nginx/logs:/var/log/nginx/ \\
-v ~/software/nginx/www/:/usr/share/nginx/html/ \\
-p 80:80 \\
-p 443:443 \\
nginx

```

2.下面来进行 nginx 配置通过域名来访问我们的网站，打开 ~/software/nginx/conf.d/default.conf

```bash
vim  ~/software/nginx/conf.d/default.conf

```

3.修改配置文件如下：
server_name 配置新申请的域名，如下我的域名是 [www.xinximo.com](http://www.xinximo.com/) 和 [xinximo.com](http://xinximo.com/)
proxy_pass 配置自己 wordpress 的访问路径： [http://xinximo.com:9001](http://xinximo.com:9001/)

```bash
server {
    listen       80;
    listen  [::]:80;
    server_name  www.xinximo.com xinximo.com;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \\.php$ {
    #    proxy_pass   <http://127.0.0.1>;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \\.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\\.ht {
    #    deny  all;
    #}
}

```

## 五.配置https访问

1.使用scp命令复制证书文件到服务器

```bash
scp -r 文件夹 root@你的IP:~/software/nginx/

```

2.把对应的nginx的证书文件和私钥文件放在~/software/nginx/目录下(参考地址:[https://cloud.tencent.com/document/product/400/35244](https://cloud.tencent.com/document/product/400/35244))

3.配置 ～/software/nginx/conf.d/defalut.conf 文件：

```bash
vim  ~/software/nginx/conf.d/default.conf

```

```bash
server {
    listen       80;
    listen  [::]:80;
    #SSL 访问端口号为 443
    listen  443 ssl;
    server_name  www.xinximo.com xinximo.com;
    #证书文件名称
    ssl_certificate 1_xinximo.com_bundle.crt;
    #私钥文件名称
    ssl_certificate_key 2_xinximo.com.key;
    ssl_session_timeout 5m;
    #请按照以下协议配置
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        # root   /usr/share/nginx/html;
        # index  index.html index.htm;
	    proxy_pass <http://xinximo.com:9001/;>
        proxy_set_header        X-Real-IP       $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    error_page  404              /404.html;
    location = /40x.html {
    }

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \\.php$ {
    #    proxy_pass   <http://127.0.0.1>;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \\.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\\.ht {
    #    deny  all;
    #}
}

```
