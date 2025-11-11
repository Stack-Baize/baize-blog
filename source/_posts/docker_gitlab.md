---
title: 基于Docker部署GitLab环境搭建
date: '2020-09-24 11:00'
summary: 最近在学习自动化部署的一些内容，涉及到的内容有Docker、Jenkins、Gitlab等内容，今天通过docker玩了一遍gitlab，下面是一些心得。
categories: 环境部署
tags:
  - Linux
  - Docker
  - GitLab
abbrlink: '3184'
---

最近在学习自动化部署的一些内容，涉及到的内容有Docker、Jenkins、Gitlab等内容，今天通过docker玩了一遍gitlab，下面是一些心得。

## 0x01 前提条件

- （1）存在docker
- （2）服务器可以联网（外网）
- （3）服务器内存至少4G（内存不够会出现502错误）

## 0x02 安装
本次安装在CentOS7下进行，下面的命令建议复制到记事本后再从记事本复制

### 卸载旧版docker
```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 配置 docker 拉取源
如不配置源因网络原因可能会拉取失败，配置国内拉取源提升稳定性。

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

### 安装docker并配置docker源
```bash
yum -y install yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-nightly
yum-config-manager --enable docker-ce-test
yum makecache fast
yum -y install docker-ce docker-ce-cli containerd.io
systemctl start docker && systemctl enable docker
```

### gitlab镜像拉取
> gitlab-ce为稳定版本，后面不填写版本则默认pull最新latest版本

```bash
$ docker pull gitlab/gitlab-ce:latest
$ mkdir -p /home/gitlab/{config,logs,data}
```

## 0x03 配置环境

### 运行gitlab镜像
拉取gitlab镜像并配置环境
```bash
$ docker run -d \
 -p 443:443 \
 -p 80:80 \
 -p 222:22 \
 --name gitlab \
 --restart always \
 -v /home/gitlab/config:/etc/gitlab \
 -v /home/gitlab/logs:/var/log/gitlab \
 -v /home/gitlab/data:/var/opt/gitlab \
 gitlab/gitlab-ce
 ```

- -d：后台运行
- -p：将容器内部端口向外映射
- --name：命名容器名称
- -v：将容器内数据文件夹或者日志、配置等文件夹挂载到宿主机指定目录

### 修改配置

按上面的方式，gitlab容器运行没问题，但在gitlab上创建项目的时候，生成项目的URL访问地址是按容器的hostname来生成的，也就是容器的id。作为gitlab服务器，我们需要一个固定的URL访问地址，于是需要配置gitlab.rb（宿主机路径：`/home/gitlab/config/gitlab.rb`）。

gitlab.rb文件内容默认全是注释
```bash
$ vim /home/gitlab/config/gitlab.rb
# 配置http协议所使用的访问地址,不加端口号默认为80
external_url 'http://192.168.199.231'
# 配置ssh协议所使用的访问地址和端口
gitlab_rails['gitlab_ssh_host'] = '192.168.199.231'
gitlab_rails['gitlab_shell_ssh_port'] = 222
# 此端口是run时22端口映射的222端口
:wq #保存配置文件并退出
```

修改邮箱

在gitlab.rb文件的最后添加如下代码
```bash
# 是否启用
gitlab_rails['smtp_enable'] = true
# SMTP服务的地址
gitlab_rails['smtp_address'] = "smtp.qq.com"
# 端口
gitlab_rails['smtp_port'] = 465
# 你的QQ邮箱（发送账号）
gitlab_rails['smtp_user_name'] = "xxx@qq.com"
# 授权码
gitlab_rails['smtp_password'] = "********"
# 域名
gitlab_rails['smtp_domain'] = "smtp.qq.com"
# 登录验证
gitlab_rails['smtp_authentication'] = "login"
# 使用了465端口，就需要配置下面三项
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'none'

# 你的QQ邮箱（发送账号）
gitlab_rails['gitlab_email_from'] = 'xxx@qq.com'
```

应用配置
```bash
sudo gitlab-ctl reconfigure
```

修改port

修改gitlab.yml文件
```bash
// 文件路径 /opt/gitlab/embedded/service/gitlab-rails/config
sudo cd /opt/gitlab/embedded/service/gitlab-rails/config
vim gitlab.yml
// 修改port 为8090
```

重启gitlab容器
```bash
$ docker restart gitlab
# 查看启动情况
$ docker ps
# 登录容器
$ docker exec -it gitlab #!/usr/bin/env bash
```

验证邮箱服务

```bash
// 在容器中进入命令行
sudo gitlab-rails console
// 测试邮件发送
sudo Notify.test_email("xxx@163.com","title","gitlab").deliver_now
// 退出命令行
sudo exit
// 退出容器
sudo exit
```

此时项目的仓库地址就变了。如果ssh端口地址不是默认的22，就会加上ssh:// 协议头

打开浏览器输入ip地址(因为我的gitlab端口为80，所以浏览器url不用输入端口号，如果端口号不是80，则打开为：ip:端口号)

第一次进入要输入新的root用户密码，设置好之后确定就行
下面我们就可以新建一个项目了，点击Create a project
创建项目。
