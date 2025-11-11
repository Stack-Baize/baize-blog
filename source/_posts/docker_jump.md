---
title: 基于Docker部署jumpserver环境
date: '2020-09-26 11:00'
summary: jumpserver部署环境可以分为多种，这次我们使用Docker来部署。
categories: 环境部署
tags:
  - Linux
  - jumpserver
  - Docker
abbrlink: 12a1
---

在 Docker中部署jumpserver环境也可以分为几次情况
- 环境都在一个docker镜像中，这种环境不建议
- 环境分为三个docker镜像，jumpserver、mysql、redis各一个镜像

本次我们部署环境为第二种情况

## 0x01 安装 Docker

### 配置国内映像源

```bash
mkdir /etc/docker
echo "{
    \"registry-mirrors\" : [
    \"https://registry.docker-cn.com\",
    \"https://docker.mirrors.ustc.edu.cn\",
    \"http://hub-mirror.c.163.com\",
    \"https://cr.console.aliyun.com/\"
  ]
}">>/etc/docker/daemon.json
```

### 安装配置docker

```bash
yum -y install yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum -y install docker-ce
systemctl start docker && systemctl enable docker
```

### 生成秘钥
```bash
if [ "$SECRET_KEY" = "" ]; then SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`; echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc; echo $SECRET_KEY; else echo $SECRET_KEY; fi

if [ "$BOOTSTRAP_TOKEN" = "" ]; then BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`; echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc; echo $BOOTSTRAP_TOKEN; else echo $BOOTSTRAP_TOKEN; fi
```
注：生成完 SECRET_KEY 和 BOOTSTRAP_TOKEN 变量后一定要确认一下，如果出现异常将会影响到后面的过程

```bash
#查看秘钥是否生成
echo $SECRET_KEY
echo $BOOTSTRAP_TOKEN
```

创建jms容器中的日志及数据挂到宿机的目录
```bash
mkdir -p /home/jumpserver/data
mkdir -p /home/koko/data
mkdir -p /home/nginx/logs
mkdir -p /home/mysql/{data,logs,conf}
```

## 0x02 映像拉取

### mysql 映像拉取

```bash
#docker pull mysql
docker run --restart=always \
--name mysql5.7 -id \
-e MYSQL_DATABASE="jumpserver" \
-e MYSQL_USER="jumpserver" \
-e MYSQL_PASSWORD="Ya0ling" \
-e MYSQL_ROOT_PASSWORD="Ya0ling" \
-v /home/mysql/data:/var/lib/mysql \
-v /home/mysql/logs:/var/log/mysql/ \
-v /home/mysql/conf:/etc/mysql/ \
-p 3306:3306 -d mysql:5.7.20
```

### redis 映像拉取
```bash
#docker pull redis
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo "vm.overcommit_memory=1">>/etc/sysctl.conf
echo "net.core.somaxconn= 1024">>/etc/sysctl.conf
echo "'echo never > /sys/kernel/mm/transparent_hugepage/enabled'">>/etc/rc.local
sysctl -p
# 拉取映像
docker run -p 6379:6379 --name redis -v /home/redis/data:/data -d redis redis-server --requirepass "Ya0ling" --appendonly yes
```

redis容器中登录方式

```bash
#查看映像运行情况
docker ps -a
# 登录 redis 映像
docker exec -it redis /bin/bash
# 映像中登录 redis 查看key
redis-cli -h localhost -p 6379
# 输入 redis 密码
auth Ya0ling
# 查看 key
auth key *
#退出
exit
```

>#注意映射关系修改配置为支持utf8mb4,或使用客户端登录修改jumpserver数据库编码

```bash
$ vim /data/mysql/conf/mysql.cnf
[mysql]
default-character-set=utf8mb4

root@ubuntu:~# vim /data/mysql/conf/mysqld.cnf
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
#log-error      = /var/log/mysql/error.log
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character-set-server=utf8mb4
```

创建数据库并设置为utf8mb4编码

```bash
create database jumpserver default charset 'utf8mb4' collate 'utf8mb4_general_ci';
grant all on jumpserver.* to 'jumpserver'@'%' identified by 'weakPassword';
```

修改数据库的字符集

```bash
mysql>use jumpserver
mysql>alter database jumpserver character set utf8mb4;
# 查看数据库编码
show variables like '%char%';
# 暂时设置编码
set character_set_client=utf8mb4;
```

修改my.conf设置编码

```bash
[client]
default-character-set=utf8mb4
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
```

### 拉取 jumpserver 映像

```bash
#docker pull jms
docker run --restart=always \
--name jms_all -d \
-p 80:80 -p 2222:2222 \
-e SECRET_KEY=$SECRET_KEY \
-e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN \
-v /home/jumpserver/data:/opt/jumpserver/data \
-v /home/jumpserver/logs:/opt/jumpserver/logs \
-v /home/koko/data:/jumpserver/koko/data \
-v /home/nginx/logs:/var/log/nginx/ \
-e DB_HOST="mysql5.7" \
-e DB_PORT=3306 \
-e DB_USER=root \
-e DB_PASSWORD=Ya0ling \
-e DB_NAME=jumpserver \
--link mysql5.7:mysql \
-e REDIS_HOST=redis \
-e REDIS_PORT=6379 \
-e REDIS_PASSWORD=Ya0ling \
--link redis:redis \
jumpserver/jms_all:latest
```

测试（其他机器连接，连接用户是admin，密码是admin）


docker容器设置开机自启动：
- --restart具体参数值详细信息
- no - 容器退出时，不重启容器
- on-failure - 只有在非0状态退出时才从新启动容器
- always - 无论退出状态是如何，都重启容器
使用 on-failure 策略时指定 Docker 将尝试重新启动容器的最大次数；默认情况下Docker将尝试永远重新启动容器；
- docker run --restart=on-failure:10 redis
如果创建容器时未指定 --restart=always ,可通过 update 命令更改；    
- docker update --restart=always 容器ID

>如未使用--restart=always选项，在服务器或其他情况导致服务器关机/重启，再次启动容器时需先起MySQL、redis，最后起jms
