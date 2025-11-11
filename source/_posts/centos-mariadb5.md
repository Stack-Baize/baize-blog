---
title: CentOS编译安装配置MariaDB数据库
date: '2020-04-29 11:00'
summary: 编译安装数据库是每一位运维人员的必备技能，在CentOS下编译安装配置MariaDB数据库。
categories: 数据库
tags:
  - MariaDB
  - 数据库
  - CentOS
abbrlink: ed21
---

## 安装环境

操作系统：CentOS 7
MariaDB版本：mariadb-5.5.33a
MariaDB数据库存储目录：/data/mysql

## 配置环境

### 1、安装系统

略

### 2、配置网络

略

### 3、配置防火墙

开启 3306 端口

```bash
#iptables防火墙配置
vi /etc/sysconfig/iptables
#允许所有IP经过3306端口通过防火墙
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
#很多网友把这两条规则添加到防火墙配置的最后一行，导致防火墙启动失败，正确的应该是添加到默认的22端口这条规则的下面
#配置完后须要重启使防火墙生效
/etc/init.d/iptables restart
#firewalld防火墙配置
#检查防火墙是否启用,及开通的端口
firewall-cmd --list-all
#放通 3306端口
firewall-cmd --permanent --add-port=3306/tcp
#放通 10.248.0.0/28位访问 3306端口
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.248.0.0/28"  port port=3306 protocol=tcp accept'
#重新载入防火墙配置使用配置生效
firewall-cmd --reload
```

### 4、关闭 SELINUX

```bash
vi /etc/selinux/config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq!  #保存退出
shutdown -r now #重启系统
```

## 编译安装

### 规划目录

MariaDB安装包存放位置：`/usr/local/src`

MariaDB编译安装位置：`/usr/local/mysql`

### 下载安装包

#### 下载MariaDB

访问官网或国内源下载

#### 下载cmake（MariaDB编译工具）

`http://www.cmake.org/files/v2.8/cmake-2.8.12.1.tar.gz`

#### 安装编译工具及库文件（使用CentOS yum命令安装，安装的比较多，方便以后编译安装php、nginx等）

```bash
yum  install make apr* autoconf automake curl curl-devel gcc gcc-c++ gtk+-devel zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat*  cpp glibc libgomp libstdc++-devel keyutils-libs-devel libsepol-devel libselinux-devel krb5-devel  libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel  libjpeg* libpng* php-common php-gd gettext gettext-devel ncurses* libtool* libxml2 libxml2-devel patch policycoreutils bison
```

### 安装篇

#### 一、安装cmake

```bash
cd /usr/local/src
tar zxvf cmake-2.8.12.1.tar.gz
cd cmake-2.8.12.1
./configure
make   #编译
make install   #安装
```

#### 二、安装MariaDB

```bash
groupadd mysql  #添加MariaDB数据库安装用户组mysql
useradd -g mysql mysql -s /bin/false  #建用户mysql并加入到mysql组，不允许mysql用户直接登录系统
mkdir -p /data/mysql  #创建MariaDB数据库存放目录
chown -R mysql:mysql /data/mysql   #设置MariaDB数据库目录权限
mkdir -p /usr/local/mysql #创建MariaDB安装目录
cd /usr/local/src
tar zxvf mariadb-5.5.33a.tar.gz  #解压
cd mariadb-5.5.33a #进入安装目录
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql  -DMYSQL_DATADIR=/data/mysql  -DSYSCONFDIR=/etc #配置
make #编译
make install  #安装
cd /usr/local/mysql
cp ./support-files/my-huge.cnf  /etc/my.cnf   #拷贝配置文件（注意：如果/etc目录下面默认有一个my.cnf，直接覆盖即可）
vi /etc/my.cnf   #编辑配置文件,在 [mysqld] 部分增加
datadir = /data/mysql  #添加MariaDB数据库路径
./scripts/mysql_install_db --user=mysql  #生成MariaDB系统数据库
cp ./support-files/mysql.server  /etc/rc.d/init.d/mysqld  #把MariaDB加入系统启动
chmod 755 /etc/init.d/mysqld   #增加执行权限
chkconfig mysqld on  #加入开机启动
vi /etc/rc.d/init.d/mysqld  #编辑
basedir = /usr/local/mysql   #MariaDB程序安装路径
datadir = /data/mysql  #MariaDB数据库存放目录
service mysqld start  #启动
vi /etc/profile   #把MariaDB服务加入系统环境变量：在最后添加下面这一行
export PATH=$PATH:/usr/local/mysql/bin
#下面这两行把MariaDB的库文件链接到系统默认的位置，这样你在编译类似PHP等软件时可以不用指定MariaDB的库文件地址。
ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
ln -s /usr/local/mysql/include/mysql /usr/include/mysql
shutdown -r now     #需要重启系统，等待系统重新启动之后继续在终端命令行下面操作
mysql_secure_installation    #设置MariaDB数据库root账号密码
#根据提示按Y 回车输入2次密码
#或者直接修改密码/usr/local/mysql/bin/mysqladmin -u root -p password "123456" #修改密码
service mysqld restart  #重启
mysql -u root -p  #输入上面设置的root密码登录到mariadb控制台
```

到此，MariaDB数据库安装完成！
