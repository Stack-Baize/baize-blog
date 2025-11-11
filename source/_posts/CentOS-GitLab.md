---
title: CentOS上搭建GitLab的详细过程
abbrlink: 0002
date: 2020-03-04 13:00
summary: '稍具规模一点的公司都会搭建属于自己的git，svn，而内部git用的最多的则是gitlab，虽然官网已经提供了非常多的功能，但内网搭建更能保证项目的私有性，只有公司内部员工才可以访问，更加安全。'
categories: 
 - 环境部署
 - GitLab
tags:
 - CentOS
 - GitLab
---

## Git的优点和缺点介绍
### Git优点

- 1、适合分布式开发，强调个体
- 2、公共服务器压力和数据量都不会太大
- 3、速度快、灵活
- 4、任意两个开发者之间可以很容易的解决冲突
- 5、离线可以正常提交代码和工作

### Git缺点
- 1、学习周期相对而言比较长
- 2、不符合常规思维
- 3、代码保密性差，一旦开发者把整个库克隆下来就可以完全公开所有代码和版本信息

## 1.准备环境

操作系统： CentOS 8 （搞清楚自己的环境，如果不知道 请输入以下命令）：

```bash
CentOS:#  cat /etc/redhat-release
CentOS Linux release 8.1.1911 (Core)
```
Ubuntu 系统命令

``` bash
Ubuntu:# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04
```

### 1.安装依赖包：

``` bash
sudo dnf install curl openssh-server ca-certificates postfix
# Ubuntu 命令为 apt
```

注：执行完成后，出现邮件配置，选择Internet那一项（不带Smarthost的）,选择完后，后面的东西，随便填吧，没啥卵用~

### 2.修改镜像源地址

常用的国内源地址

- 阿里源： [http://mirrors.aliyun.com/](http://mirrors.aliyun.com/ "阿里云源")
- 清华大学：[https://mirrors.tuna.tsinghua.edu.cn/](https://mirrors.tuna.tsinghua.edu.cn/ "清华大学源")
- 中科大：[https://mirrors.ustc.edu.cn/](https://mirrors.ustc.edu.cn/ "中科大源")

我们利用清华大学的镜像
```txt
https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/
```
来进行主程序的安装。首先信任 GitLab 的 GPG 公钥：

```bash
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
# 查看修改源地址
sudo  vi /etc/yum.repos.d/gitlab_gitlab-ee.repo
[gitlab_gitlab-ee]
name=gitlab_gitlab-ee
baseurl=https://packages.gitlab.com/gitlab/gitlab-ee/el/8/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
gpgkey=https://packages.gitlab.com/gitlab/gitlab-ee/gpgkey
       https://packages.gitlab.com/gitlab/gitlab-ee/gpgkey/gitlab-gitlab-ee-3D645A26AB9FBD22.pub.gpg
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[gitlab_gitlab-ee-source]
name=gitlab_gitlab-ee-source
baseurl=https://packages.gitlab.com/gitlab/gitlab-ee/el/8/SRPMS
repo_gpgcheck=1
gpgcheck=1
enabled=1
gpgkey=https://packages.gitlab.com/gitlab/gitlab-ee/gpgkey
       https://packages.gitlab.com/gitlab/gitlab-ee/gpgkey/gitlab-gitlab-ee-3D645A26AB9FBD22.pub.gpg
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

安装 gitlab-ce:

```bash
sudo dnf update
sudo dnf install gitlab-ce
```

注： 有点慢 耐心等吧~
修改配置 ：

```bash
vi /etc/gitlab/gitlab.rb

# 更改 external_url
external_url =http://192.168.1.10 # (IP换成你本机的IP地址)
```

### 3.启动sshd和postfix服务

```bash
sudo systemctl start sshd
sudo systemctl start postfix
```

### 4.添加防火墙规则

```bash
sudo iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
```

### 5.启动各项服务

```bash
sudo gitlab-ctl reconfigure
```

有点慢，须要稍等下。

上面这一步可能会失败，报错如下：

```txt
ERROR: user[git] (gitlab''users line 34) ......
```

解决办法：进入到文件： /etc/gitlab/gitlab.rb，找到下面他们俩：

```bash
user[‘username’]=’git’
User[‘group’]=’git’
# 讲git改为gitlab。然后初始化配置
sudo gitlab-ctl reconfigure
```

这时又报错，查了下说是至少大于等于2G CPU运行内存，修改内存后解决

### 6.查看安装是否成功

```bash
sudo gitlab-ctl status
```

出现一下画面就OK了：

```bash
sudo gitlab-ctl status
run: alertmanager: (pid 22060) 15s; run: log: (pid 22146) 15s
run: gitaly: (pid 21990) 17s; run: log: (pid 22003) 16s
run: gitlab-monitor: (pid 22026) 16s; run: log: (pid 22030) 16s
run: gitlab-workhorse: (pid 21973) 17s; run: log: (pid 21981) 17s
run: logrotate: (pid 21526) 64s; run: log: (pid 21983) 17s
run: nginx: (pid 21498) 66s; run: log: (pid 21982) 17s
run: node-exporter: (pid 21753) 52s; run: log: (pid 22004) 16s
run: postgres-exporter: (pid 22153) 15s; run: log: (pid 22161) 14s
run: postgresql: (pid 21187) 201s; run: log: (pid 21964) 17s
run: prometheus: (pid 22039) 15s; run: log: (pid 22053) 15s
run: redis: (pid 21117) 207s; run: log: (pid 21963) 17s
run: redis-exporter: (pid 21791) 44s; run: log: (pid 22031) 16s
run: sidekiq: (pid 21465) 73s; run: log: (pid 21966) 17s
run: unicorn: (pid 21428) 79s; run: log: (pid 21965) 17s
```

登陆地址 ，就是刚才你刚才添加到配置文件的那个地址登陆访问（无需输入端口）：

```txt
http://192.168.1.10
```

上来先让你初始化密码，剩下的就是界面画操作。 (帐号密码在上面gitlab.rb中有设置)
安装到此结束~

注：gitlab在服务器中的默认代码存放的位置是 /var/opt/gitlab/git-data/repositories
