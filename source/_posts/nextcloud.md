---
title: Centos7 搭建开源个人网盘Nextcloud与常用插件
date: '2020-06-20 11:00'
categories: NextCloud
tags:
  - Linux
  - CentOS
  - NextCloud
abbrlink: e4ca
---

## 简介

文件服务器，是一个公司最常用的服务应用，每个公司企业基本都有自己的文件服务器实，现储存分享，上传下载文件文档等功能，常见的文件服务器就是ftp服务器，但是ftp服务器的功能实在有限，且对于普通用户使用入手难度较大，又缺乏界面，对于公司的普通用户来说，确实不是一个好的文件服务器。一般ftp也只是用户服务器，网站应用等方面。


在目前的公司企业环境中，企业网盘则是一个更好的文件服务器替代方案。百度云盘，相信大家基本都有用过吧，友好的界面交互，网页端，客户端都有，不需要任何命令，实现上传下载，分享等诸多功能。但是这种云盘毕竟不是自己的，很多公司对于将机密文件放在上面心存疑惑，且时不时的网盘关闭热潮也让人担心。既然如此，为何不搭建自己的网盘呢，于是就有了nextcloud。

对于私人网盘，市面上已经有很多的产品，很多开源半开源的云盘系统。其中最出名的就是 seafile和owncloud/nextcloud。seafile是国人开发的，分块处理，断点上传，速度比后者要快。有社区版和企业版。社区版免费，但是功能有限，企业版要收费，功能更强大。但是社区版的功能，老实说只能满足个人使用，无法满足企业使用。而企业版要收费，费用根据公司人数不同，还需要发邮件询问。

像我着这种穷人，穷公司，人数不多，又不像花钱的，那么nextcloud就是最好的选择，nextcloud是owncloud的一个分支，由原创始人团队维护，是在owncloud被别的公司收购后，由创始人团队创立的新分支。就像 mysql和mariadb。nextcloud完全开源，功能强大，是外国人开发维护的。具体的与seafile等的对比，这里不详细说明了，有兴趣的百度就好。下面记录一下在centos7 服务器上搭建nextcloud的具体过程。

 
## 安装环境

环境: centos8
nextcloud: 19.0.0 
selinux 关闭


1. nextcloud是php项目，这里我使用nginx，官方文档是用apache的，有一点点具别。但不大。在下载nextcloud之前，先安装nginx和php

```bash
# 先删除系统可能自带的PHP和nginx，用来面命令查找是否有安装
$ rpm -qa |grep php
$ rpm -qa |grep nginx
# 强制删除软件包，软件包全称就是上面通过rpm -qa 查询到的软件包全名
$ rpm -e --nodeps {软件包全称} 
 
# 安装yum的epel源，这里推荐用阿里云的镜像安装，包更全。
$ rpm -ivh https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
 
# 安装nginx
$ yum install -y nginx
 
# 安装php的源
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
 
# 安装php已经会用到扩展
$ yum install -y php70w-devel php70w-pear php70w-pecl php70w-gd php70w-opcache php70w-cli php70w-pdo php70w-process php70w-pecl-apcu php70w-mcrypt php70w-mysql php70w-fpm php70w-pecl-memcached php70w-common php70w-xml php70w-mbstring php70w-pecl-igbinary
 
# 检查是否安装成功
$ nginx -v
nginx version: nginx/1.12.2
$ php -v
PHP 7.0.30 (cli) (built: Apr 28 2018 08:14:08) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.0.30, Copyright (c) 1999-2017, by Zend Technologies
```

2. nextcloud 还需要数据库，这里我使用的mysql，mysql提前安装好或使用已有的数据库服务器，具体安装安装方式可参考我另一篇博文: 《CentOS7.2 安装mysql，并配置自动启动和远程访问》。在mysql种创建一个新的databse用于nextcloud使用.

```bash
# 在装有mysql的服务器上执行
$ mysql -u root -p
> create database nextcloud_db;
# 授权给自定义用户，这里以用户名：nextclouduser，密码：nextcloudpasswd，代替。
> grant all privileges on nextcloud_db.* to nextclouduser@localhost identified by 'nextcloudpasswd';
> flush privileges;
```

3.  为nextcloud 生成自签名ssl证书

```bash
$ cd /etc/nginx/cert/    # 没有则创建此文件夹
$ openssl req -new -x509 -days 365 -nodes -out /etc/nginx/cert/nextcloud.crt -keyout /etc/nginx/cert/nextcloud.key
# 会出现下面的选项需要填写，可以随便填。
Country Name (2 letter code) [XX]:cn                                 # 国家
State or Province Name (full name) []:guangdong                      # 省份
Locality Name (eg, city) [Default City]:guangzhou                    # 地区名字
Organization Name (eg, company) [Default Company Ltd]:Amos           # 公司名
Organizational Unit Name (eg, section) []:Technology                 # 部门
Common Name (eg, your name or your server's hostname) []:Amos        # CA主机名
Email Address []:Amos@Amos.com                                       # Email地址
# 修改证书和文件夹权限
$ chmod 600 /etc/nginx/cert/*
$ chmod 700 /etc/nginx/cert
```

4. 下载nextcloud，并配置php和nginx

```bash
# 下载nextcloud，官网地址为: https://nextcloud.com/install/#instructions-server
$ cd /usr/local/src
$ yum install -y wget unzip
$ wget https://download.nextcloud.com/server/releases/nextcloud-13.0.2.zip    # 下载
$ unzip nextcloud-13.0.2.zip    # 解压
$ mv nextcloud /usr/share/nginx/html/    # 移动到指定文件夹内
$ cd /usr/share/nginx/html/nextcloud     # 进行nextcloud 目录中
$ mkdir data    # 创建数据文件夹
$ chown nginx:nginx -R nextcloud/    # 将nextcloud文件授权给nginx
 
# 配置php-fpm
$ vim /etc/php-fpm.d/www.conf
--------------------------------------------------------------------------------
user = nginx                                   //将用户和组都改为nginx
group = nginx
listen = 127.0.0.1:9000
env[HOSTNAME] = $HOSTNAME                     //将以下几行，去掉注释
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
--------------------------------------------------------------------------------
 
 
# 为php创建session文件夹
$ mkdir -p /var/lib/php/session
$ chown nginx:nginx -R /var/lib/php/session/
 
 
# 配置nginx
$ cd /etc/nginx/conf.d/
$ vim nextcloud.conf
--------------------------------------------------------------------------------
upstream php-handler {
    server 127.0.0.1:9000;
    #server unix:/var/run/php5-fpm.sock;
}
 
 
server {
    listen 80;
    server_name localhost;
    # enforce https
    rewrite ^(.*)$ https://$host$1 permanent;
}
 
 
server {
    listen 443 ssl;
    server_name localhost;
 
    ssl_certificate /etc/nginx/cert/nextcloud.crt;
    ssl_certificate_key /etc/nginx/cert/nextcloud.key;
 
    # Add headers to serve security related headers
    # Before enabling Strict-Transport-Security headers please read into this
    # topic first.
    add_header Strict-Transport-Security "max-age=15768000;
    includeSubDomains; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
 
    # Path to the root of your installation
    root /usr/share/nginx/html/nextcloud/;
 
 
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
 
 
    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json
    # last;
 
 
    location = /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
 
 
    # set max upload size
    client_max_body_size 10240M;    # 上传文件最大限制，php.ini中也要修改，最后优化时会提及。
    fastcgi_buffers 64 4K;
 
    # Disable gzip to avoid the removal of the ETag header
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;
 
 
    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;
 
 
    error_page 403 /core/templates/403.php;
    error_page 404 /core/templates/404.php;
 
 
    location / {
        rewrite ^ /index.php$uri;
    }
 
 
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
        deny all;
    }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }
 
    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+|core/templates/40[34])\.php(?:$|/) {
        include fastcgi_params;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        #Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }
 
 
    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
        try_files $uri/ =404;
        index index.php;
    }
 
 
    # Adding the cache control header for js and css files
    # Make sure it is BELOW the PHP block
    location ~* \.(?:css|js)$ {
        try_files $uri /index.php$uri$is_args$args;
        add_header Cache-Control "public, max-age=7200";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        add_header Strict-Transport-Security "max-age=15768000;includeSubDomains; preload;";
        add_header X-Content-Type-Options nosniff;
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        # Optional: Don't log access to assets
        access_log off;
    }
 
    location ~* \.(?:svg|gif|png|html|ttf|woff|ico|jpg|jpeg)$ {
        try_files $uri /index.php$uri$is_args$args;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
----------------------------------------------------------------------------------
 
 
# 将nginx原配置中80端口的配置删除
$ vim /etc/nginx/nginx.conf
--------------------------------------------------------
server {          # 将80端口的server整个删除，应该我们在上面nextcloud.conf中已经配置了，这里不删除的话会导致冲突不生效。
    listen 80;
...
  }
--------------------------------------------------------
```

5. 启动nginx和php-fpm

```bash
$ nginx -t    # 检查nginx配置是否正确，出现下面输入则正确。
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
 
# 启动并设为开机启动 nginx，php-fpm
$ systemctl start nginx
$ systemctl enable nginx
$ systemctl start php-fpm
$ systemctl enable php-fpm

# 配置防火墙，开放http和https的端口。
$ firewall-cmd --add-port=80/tcp --permanent
$ firewall-cmd --add-port=443/tcp --permanent
$ firewall-cmd --reload
``` 
 
PS： 这里我的selinux是关闭的，如果selinux没有关闭，则执行下面命令关闭selinux
```bash
$ setenforce 0    # 关闭selinux
$ vim /etc/selinux/config    # 修改配置，永久关闭。
------------------------------------------------------
SELINUX=disabled
------------------------------------------------------
```

6. 访问网页界面，完成安装。访问搭建nextcloud服务器的ip地址。（如果有域名就访问域名）
按照实际情况进行配置，配置完成后，点击安装完成。上图中mysql的主机名，使用上面我们自己的数据库服务器地址和端口

7. 性能优化，进入主界面后，右上角自己头像，点击设置，基本设置：
第一项就有 安全及设置警告，这里会有配置错误提示，优化提示等。根据提示进行优化：
① 修改php.ini 文件，添加如下配置：

```bash
$ vim /etc/php.ini
------------------------------------------------------------------------
[PHP]    # 在[PHP] 以下添加如下配置
opcache.enable=1
opcache.enable_cli=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
...
upload_max_filesize = 10240M    # 上传文件最大大小，可以自定义修改，默认为512M
------------------------------------------------------------------------
 
$ vim /etc/nginx/nginx.d/nextcloud.conf
```

② 设置缓存后端，可以使用redis，memcache。单机或集群模式都可以。不同的配置方式可以参考官方文档。

这里我直接单机安装并配置使用memcache。
```bash
$ yum install -y memcache
$ vim /etc/sysconfig/memcached
------------------------------------------------------------------------
PORT="11211"    # 端口
USER="memcached"    # 用户
MAXCONN="1024"    # 最大链接数
CACHESIZE="2048"    # 最大内存，单位M
OPTIONS=""
-----------------------------------------------------------------------
 
# 修改nextcloud的config配置文件，添加memcached缓存配置
$ vim /usr/share/nginx/html/nextcloud/config/config.php
-----------------------------------------------------------------------
'memcache.local' => '\OC\Memcache\APCu',
  'memcache.distributed' => '\OC\Memcache\Memcached',
  'memcached_servers' => array(
   array('localhost', 11211),
     ),
-----------------------------------------------------------------------
```

③ 重启nginx和php-fpm，是配置生效
```bash
$ systemctl start memcached
$ systemctl enable memcached
$ systemctl restart nginx
$ systemctl restart php-fpm
```

④ 设置后台任务，cron执行。
```bash
$ vim /etc/crontab    # 修改cron配置文件，添加如下配置
---------------------------------------------------------------------------------
*/15 * * * * -u nginx /usr/bin/php -f /usr/share/nginx/html/nextcloud/cron.php
---------------------------------------------------------------------------------
```

8. 配置邮件服务器

在设置 --> 其他设置 中，配置smtp服务器。并进行测试，收到邮件则为OK。

9. 到此，nextcloud已经安装完成。除此之外，nextcloud还有很多常用插件，用于拓展功能，包括官方的或个人的，点击右上角个人头像-->应用


在这里可以直接点击安装启动应用。非常方便。本身安装时便会自带有一些应用，这里推荐几个需要手动安装的常用应用：

①Announcement center  管理员可以发公告

②Circles 圈子，每个人都可以建立加入圈子，实现圈子的文件共享

③Group folders 组文件夹

④File access control 文件访问控制

⑤Impersonate 管理员可以模拟用户，可以以用户登陆到他们的网盘，可以看到个人用户的文件，这个有点不太隐私。

。。。

当然，还有很多有趣的应用，等你自己去发现。

OK，到此，nextcloud的搭建已经完成。自己去不断使用，不断探索功能吧。

 

PS：在设置中还会一直有一些报错，虽然按照要求进行配置了，却还是报错，这就在暂且忽略掉吧
