---
title: 安装配置Docker环境
date: '2020-09-23 11:00'
summary: 'Docker是一个软件集装箱化平台，这意味着您可以构建应用程序，将它与其依赖关系一起打包到一个容器中，然后这些容器可以很容易地运送到其他机器上运行。'
categories: 环境部署
tags:
  - Linux
  - Docker
abbrlink: 5d00
---

## 0x01 Docker 简介
Docker是一个软件集装箱化平台，这意味着您可以构建应用程序，将它与其依赖关系一起打包到一个容器中，然后这些容器可以很容易地运送到其他机器上运行。
但什么是集装箱？集装化（也称为基于容器的虚拟化和应用程序集装箱化）是用于部署和运行分布式应用程序的OS级虚拟化方法，无需为每个应用程序启动整个VM。 相反，多个独立的系统（称为容器）在单个控制主机上运行并访问单个内核。
容器映像是一个轻量级的、独立的、可执行的软件包，它包括运行它所需的一切：代码、运行时、系统工具、系统库设置。
所以主要目标是将软件打包成标准化的单元进行开发，发货和部署。

## 0x02 安装前配置

### Docker版本检查

docker要求CentOS 系统的内核版本高于 3.10 ，内存须 4G 以上，安装之前首先要验证你的CentOS 版本是否支持 Docker 。

通过uname -r 命令查看你当前的内核版本（建议使用xshell连接虚拟机进行命令操作）：

```bash
$ uname -r
```

### 更新系统

使用root 权限登录 CentOS。确保 yum 包更新到最新。

```bash
$ yum -y update
```

### 卸载旧版本（如果安装过就版本的话）

```bash
$ yum remove docker \
             docker-common \
             docker-selinux \
             docker-engine \
             docker-client \
             docker-client-latest \
             docker-latest \
             docker-latest-logrotate \
             docker-logrotate
```

如果安装过旧版本docker，有就会卸载当前版本；如果没安装过，运行上面的命令也没关系，只是提示未安装


### 安装需要的软件包

yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 设置yum源

设置国内常用源，如清华镜像仓库，速度很快
如果没有安装wget则安装，如已安装则会跳过

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-nightly
yum-config-manager --enable docker-ce-test
#清缓存
yum makecache fast
```

## 0x03 安装 docker-ce

```bash
#安装 docker-ce
yum -y install docker-ce docker-ce-cli containerd.io
systemctl start docker && systemctl enable docker
```

### 搜索安装指定版本

查看源中所有版本 docker-ce

```bash
yum list docker-ce --showduplicates | sort -r
# 安装指定版本 docker
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

### 搜索映像

```bash
docker search mysql
```

### 运行hello-world 映像来验证是否正确安装了Docker Engine

```bash
docker run hello-world
```

#### 下载映像后面可加版本号

```bash
docker pull mysql:5.7
```

### 查看运行中的映像

```bash
docker ps
```

### 查看本地所有映像

```bash
docker ps -a
```

### 查看本地映像

```bash
docker images
```

### 停止运行的映像

```bash
docker stop name/id
```

### 启动映像

```bash
docker start name/id
```

### 登录docker映像

```bash
docker exec -it name/id  /bin/sh
```

### 卸载Docker

```bash
yum remove docker-ce docker-ce-cli containerd.io
```

### 删除所有图像，容器和卷

```bash
rm -rf /var/lib/docker
```
