---
title: Redis6.0.10编译安装配置
date: '2021-02-10 11:00'
summary: ' Redis是一款开源的、高性能的键-值存储（key-value store）。它常被称作是一款数据结构服务器（data structure server）。'
categories: 系统配置
tags:
  - CentOS
  - Redis
abbrlink: 1c3a
---

### 0x01 下载Redis

直接登录官网下载你需要的版本https://redis.io/

```bash
#安装编译依赖gc++
$ sudo yum install gcc-c++
#下载、编译、安装redis
$ wget http://download.redis.io/releases/redis-6.0.10.tar.gz
$ tar xzf redis-6.0.10.tar.gz
$ cd redis-6.0.10
```
### 0x02 安装编译

```bash
$ mkdir -p /usr/local/redis
#编译包并安装到指定目录
$ make PREFIX=/usr/local/redis install
#启动服务
$ src/redis-server
#运行shell 及 基本操作
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

编译后文件

redis-benchmark（压力测试工具）、redis-check-aof（检查.aof文件完整性的工具）、redis-check-rdb（检查数据文件完整性的工具）、redis-sentinel（监控集群运行状态）、redis-server（服务端）、redis-cli（客户端）

### 0x03 编译出错

![编译出错](medias/redis-error.png)

这时我们需要检查 gcc 版本

```sh
gcc -v
```

配置scl源，升级版本：

```bash
##安装scl源，修改官方源地址
yum -y install centos-release-scl
vim CentOS-SCLo-scl-rh.repo
vim CentOS-SCLo-scl.repo
##安装新版gcc
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
##永久生效
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
yum install tcl -y
```

### 0x04 修改配置文件

编辑"/etc/rc.d/init.d/redis"文件，做以下几处修改

```bash
vi redis.conf
#!/bin/sh
# chkconfig: 2345 90 10
# description: Start And Stop Redis

protected-mode no # 关闭保护模式,不然远程还是连接不了
daemonize yes     # 守护进程模式开启,设为后台运行
bind 127.0.0.1    # 绑定IP按需修改
port 6379         # 端口按需修改
```

启动 Redis服务端

```bash
cd /usr/local/redis/bin/
./redis-server /usr/local/redis/redis.conf
```

查看监听

```bash
ps -ef | grep redis
```

启动Redis客户端

```bash
./redis-cli
127.0.0.1:6379> set hello word
OK
127.0.0.1:6379> get hello
"word"
127.0.0.1:6379> exit
```

关闭Redis服务 查看监听已经关闭

```bash
./redis-cli shutdown
ps -ef | grep redis
```

设置开机自动启动Redis服务。首先复制启动脚本到资源目录

```bash
cp /root/redis-6.0.10/utils/redis_init_script /etc/rc.d/init.d/redis
```

修改启动脚本

```bash
chmod 755 /etc/rc.d/init.d/redis
##然后将Redis服务加入到系统服务
chkconfig --add redis
##最后检查Redis服务设置是否已经生效
chkconfig --list redis
```

现在就可以使用service命令来启动和停止Redis服务了

```bash
systemctl status redis.service
systemctl start redis.service
systemctl stop redis.service
```

防火墙中放通端口

```bash
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload
##查看端口是否放通
firewall-cmd --list-ports 
```

至此Redis安装完毕
