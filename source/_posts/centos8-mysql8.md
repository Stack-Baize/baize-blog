---
title: CentOS8编译安装配置MySQL8数据库
date: '2020-09-22 11:00'
summary: CentOS8已发布至8.1，现在使用CentOS8下编译安装配置MySQL数据库。
categories: 数据库
tags:
  - MySQL
  - 数据库
  - CentOS
abbrlink: a7ac
---

## 安装环境

操作系统：CentOS 8.1

MySQL版本：MySQL-8.0.18

## 环境配置

### 1、系统安装

略

### 防火墙配置

CentOS 从7.x开始默认使用的是firewall作为防火墙，这里改为iptables防火墙。

#### 1、关闭firewall

```bash
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
systemctl mask firewalld
systemctl stop firewalld
yum remove firewalld
```

#### 2、安装iptables防火墙

```bash
yum install iptables-services #安装
vi /etc/sysconfig/iptables #编辑防火墙配置文件，开发mysql默认端口3306
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
systemctl restart iptables.service #最后重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动
/usr/libexec/iptables/iptables.init restart #重启防火墙
```

### 关闭SELINUX

```bash
vi /etc/selinux/config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存退出
setenforce 0 #使配置立即生效
```

## 编译安装

### 规划目录

软件源代码包存放位置：`/usr/local/src`

源码包编译安装位置：`/usr/local/软件名字`

### 下载软件包

#### 1、mysql

`https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.18.tar.gz`

#### 2、rpcsvc-proto #CentOS 8编译安装MySQL8需要

`https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4/rpcsvc-proto-1.4.tar.gz`

#### 3、boost_1_70_0 #CentOS 8编译安装MySQL8需要

`https://dl.bintray.com/boostorg/release/1.70.0/source/boost_1_70_0.tar.gz`

#### 4、cmake #编译安装MySQL需要

`https://github.com/Kitware/CMake/releases/download/v3.15.4/cmake-3.15.4.tar.gz`

#### 五、安装编译工具及库文件（使用yum命令安装）

```bash
yum install apr* autoconf automake bison bzip2 bzip2* cpp curl curl-devel fontconfig fontconfig-devel freetype-devel gcc gcc-c++ gd gd-devel gettext gettext-devel glibc kernel kernel-headers keyutils keyutils-libs-devel krb5-devel libcom_err-devel libpng libpng-devel libjpeg* libsepol-devel libselinux-devel libstdc++-devel libtool* libgomp libxml2 libxml2-devel libXpm* libxml* libXaw-devel libXmu-devel libtiff libtiff* make openssl openssl-devel patch pcre-devel perl php-common php-gd policycoreutils telnet wget zlib-devel ncurses-devel libtirpc-devel gtk* ntpstat na* bison*
```

### 安装篇

以下是用putty工具远程登录到服务器，在命令行下面操作的

#### 一、安装MySQL

##### 1、安装rpcsvc-proto

```bash
cd /usr/local/src
tar zxvf rpcsvc-proto-1.4.tar.gz
cd rpcsvc-proto-1.4
./configure
make
make install
```

##### 2、安装cmake

```bash
cd /usr/local/src
tar zxvf cmake-3.15.4.tar.gz
cd cmake-3.15.4
./configure
make
make install
```

##### 3、安装MySQL

```bash
cd /usr/local/src
mkdir -p /usr/local/boost
cp boost_1_70_0.tar.gz /usr/local/boost
groupadd mysql #添加mysql组
useradd -g mysql mysql -s /bin/false #创建用户mysql并加入到mysql组，不允许mysql用户直接登录系统
mkdir -p /data/mysql #创建MySQL数据库存放目录
chown -R mysql:mysql /data/mysql #设置MySQL数据库存放目录权限
mkdir -p /usr/local/mysql #创建MySQL安装目录
cd /usr/local/src #进入软件包存放目录
tar zxvf mysql-8.0.18.tar.gz #解压
cd mysql-8.0.18 #进入目录
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DINSTALL_DATADIR=/data/mysql -DMYSQL_USER=mysql -DMYSQL_UNIX_ADDR=/tmp/mysqld.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_EMBEDDED_SERVER=1 -DFORCE_INSOURCE_BUILD=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DENABLED_LOCAL_INFILE=1 -DEXTRA_CHARSETS=all -DWITH_BOOST=/usr/local/boost
make #编译
make install #安装
rm -rf /etc/my.cnf  #删除系统默认的配置文件（如果默认没有就不用删除）
cd /usr/local/mysql #进入MySQL安装目录
./bin/mysqld --user=mysql --initialize --basedir=/usr/local/mysql --datadir=/data/mysql #生成mysql系统数据库 --initialize表示默认生成密码, --initialize-insecure 表示不生成密码, 密码为空。
#看到这一行[Note] [MY-010454] [Server] A temporary password is generated for root@localhost: !w1YKyVFFa?-
```
记录下自动生成的mysql管理员root账号登录密码`!w1YKyVFFa?-`

```bash
vi /usr/local/mysql/my.cnf
#mysql 8.x默认没有配置文件，我们自己创建一个。
[client]
port=3306
socket=/tmp/mysql.sock
[mysqld]
port=3306
user = mysql
socket=/tmp/mysql.sock
tmpdir = /tmp
key_buffer_size=16M
max_allowed_packet=128M
default_authentication_plugin=mysql_native_password
#设置加密方式为mysql_native_password，MySQL 8.0默认使用caching_sha2_password加密。
open_files_limit = 60000
explicit_defaults_for_timestamp
server-id = 1
character-set-server = utf8
federated
max_connections = 1000
max_connect_errors = 100000
interactive_timeout = 86400
wait_timeout = 86400
sync_binlog=0
back_log=100
default-storage-engine = InnoDB
log_slave_updates = 1
[mysqldump]
quick
[client]
# The following password will be sent to all standard MySQL clients
password="my password"
[mysqld-8.0]
sql_mode=TRADITIONAL
[mysqladmin]
force
[mysqld]
key_buffer_size=16M
```

:wq! #保存退出

```bash
ln -s /usr/local/mysql/my.cnf /etc/my.cnf  #添加到/etc目录的软连接
cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld #把Mysql加入系统启动
chmod 755 /etc/init.d/mysqld #增加执行权限
chkconfig mysqld on #加入开机启动
```

```bash
vi /etc/rc.d/init.d/mysqld #编辑
basedir=/usr/local/mysql #MySQL程序安装路径
datadir=/data/mysql #MySQl数据库存放目录
```

:wq! #保存退出

```bash
service mysqld start #启动
vi /etc/profile #把mysql服务加入系统环境变量：在最后添加下面这一行
export PATH=$PATH:/usr/local/mysql/bin
```

:wq! #保存退出

```bash
source /etc/profile #使配置立刻生效
```

下面这两行把myslq的库文件链接到系统默认的位置，这样你在编译类似PHP等软件时可以不用指定mysql的库文件地址。

```bash
ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
ln -s /usr/local/mysql/include/mysql /usr/include/mysql
mkdir /var/lib/mysql #创建目录
ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock #添加软链接
```

```bash
mysql -u root -p  #输入之前生成的密码!w1YKyVFFa?-，回车
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; #修改密码，NEVER表示密码永不过期
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456' PASSWORD EXPIRE NEVER;
#使用WITH mysql_native_password加密
#MySQL 8.0加密方式从mysql_native_password 更改为 caching_sha2_password，mysql8.x之前的客户端远程连接可能报错：authentication plugin caching_sha2
flush privileges;  #刷新系统授权表
exit  #退出mysql控制台
```

### 二、创建MySQL数据库、添加用户并授权

数据库名称：aiotlab-db
数据库用户名：aiotlab
数据库密码：aiotlab
授权aiotlab用户对aiotlab-db具有全部操作权限
继续在mysql控制台操作

```bash
mysql -u root -p
#输入刚刚修改过的密码123456，回车
Create DATABASE IF NOT EXISTS aiotlab-db default charset utf8 COLLATE utf8_general_ci;
#创建数据库
CREATE USER 'aiotlab'@'localhost' IDENTIFIED BY 'aiotlab';
#创建用户
CREATE USER 'aiotlab'@'127.0.0.1' IDENTIFIED WITH mysql_native_password BY 'aiotlab';
#加密方式为mysql_native_password
grant all privileges on aiotlab-db.* to 'aiotlab'@'localhost';
#授权用户osyunwei.com对数据库www.osyunwei.com具有全部操作权限
grant all privileges on aiotlab-db.* to 'aiotlab'@'127.0.0.1';
#授权用户
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;
#授权root用户具有全部数据库本地权限
```

MySQL5.7版本后要授权用户对所有数据库有SUPER权限，否则上一步对用户的授权只能连接但无任何操作权限。

```bash
grant SUPER on *.* to 'aiotlab'@'localhost';
#授予用户对所有数据库有SUPER权限，否则只能连接无任何操作权限。
grant SUPER on *.* to 'aiotlab'@'127.0.0.1';
flush privileges;
#刷新系统授权表
exit #退出mysql控制台
service mysqld restart
#重启mysql数据库
```

至此，CentOS 8.x安装MySQL 8.x并创建数据库添加用户对用户进行授权完成。
