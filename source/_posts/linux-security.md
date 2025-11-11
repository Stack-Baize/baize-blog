---
title: linux主机加固
date: '2020-09-28 11:00'
summary: 'Linux系统被应用于大部分企业的服务器上，因此在等保测评中主机加固也是必须要完成的一项环节。'
categories: 信息安全
tags:
  - Linux
  - PAM认证
  - 主机加固
abbrlink: ebe6
---

## 0x01 前 言

Linux系统被应用于大部分企业的服务器上，因此在等保测评中主机加固也是必须要完成的一项环节。
由于在之后项目开始要进行主机加固，因此对linux的加固流程进行总结学习。
Linux的主机加固主要分为：账号安全、认证授权、协议安全、审计安全。简而言之，就是4A（统一安全管理平台解决方案）。

这边就使用我自己kali的虚拟机进行试验学习。

## 0x02 基础加固

### 口令生存期

```bash
gedit /etc/login.defs
#在此处对密码的长度、时间、过期警告进行修改
PASS_MAX_DAYS   90  #密码最长过期天数
PASS_MIN_DAYS   10   #密码最小过期天数
PASS_WARN_AGE   7   #密码过期警告天数
如果修改设置有最小长度也需要修改
PASS_MIN_LEN    8   #密码最小长度
```

### 口令复杂度（很重要）

```bash
vim /etc/pam.d/system-auth
#在文件中找到 password requisite  pam_cracklib.so
#将其修改为:
password requisite  pam_cracklib.so try_first_pass retry=3 dcredit=-1 lcredit=-1 ucredit=-1 ocredit=-1 minlen=8
```

备注：至少包含一个数字、一个小写字母、一个大写字母、一个特殊字符、且密码长度>=8


### 版本信息

```bash
cat /proc/version
```

### 限制xx用户登录

```bash
vim /etc/hosts.deny
#添加内容：
sshd : 192.168.1.1
#禁止192.168.1.1对服务器进行ssh的登陆
```

### 检查是否有其他uid=0的用户

```bash
awk -F “：” '($3==0)  {print  $1} ' /etc/passwd
```

### 登陆超时限制

```bash
cp -p /etc/profile /etc/profile_bak #（备份）
gedit /etc/profile
#增加
TMOUT=300
export TMOUT

#或者
echo 'export TMOUT=300'>>/etc/profile
echo 'readonly TMOUT' >>/etc/profile
source /etc/profile
```

### 检查是否使用PAM认证模块禁止wheel组之外的用户su为root

```bash
gedit /etc/pam.d/su
#新增
auth          sufficient     pam_rootok.so
auth          required     pam_wheel.so use_uid
```

备注：auth与sufficient之间由两个tab建隔开，sufficient与动态库路径之间使用一个tab建隔开

### 禁用无用账户

```bash
cat /etc/passwd #查看口令文件，确认不必要的账号。

passwd -l user # 锁定不必要的账号
```

### 账户锁定

```bash
gedit /etc/pam.d/system-auth
#在文件中修改或者添加
auth required pam_tally.so onerr=fail deny=3 unlock_time=7200

#锁定账户举例：
passwd -l bin
passwd -l sys
passwd -l adm
```

### 检查系统弱口令

```bash
john /etc/shadow --single
john /etc/shadow --wordlist=pass.dic
```

我这边有报错 就不展示了
使用passwd 用户 命令为用户设置复杂的密码

## 0x03 软件协议安全

### openssh升级（按需做）

```bash
yum update  openssh
```

### 定时任务（防止病毒感染）

```bash
#定时任务检查：
crontab -l

#一次性任务检查：
at -l
```

### 限制ssh登录（看是否需要）

```bash
vim /etc/ssh/sshd_config
#查看PermitRootLogin是否为no
PermitRootLogin no #不允许root登陆
Protocol 2 #修改ssh使用的协议版本
MaxAuthTries 3 #修改允许密码错误次数

echo "tty1" > /etc/securetty
hmod 700 /root
```

### 限制su为root用户

```bash
gedit /etc/pam.d/su
#在头部添加
auth required /lib/security/pam_wheel.so group=wheel
```

### 禁止root用户登录ftp

```bash
#因为我的kali下没有这个文件，因此借鉴一下网上的
cat /etc/pam.d/vsftpd
Auth required pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
#其中file=/etc/vsftpd/ftpusers即为当前系统上的ftpusers文件.
echo  “root”   >>   /etc/vsftpd/ftpusers
```

### 防止flood攻击

```bash
gedit  /etc/sysctl.conf
#增加
net.ipv4.tcp_syncookies = 1
sysctl  -p
```

### 禁ping

```bash
echo 0 > /proc/sys/net/ipv4/icmp_echo_igore_all
```

### 检查异常进程

```bash
ps aux|sort -rn -k +3|head
#检查cpu占用前10
ps aux|sort -rn -k +4|head
#检查内存占用前10
```

### 关闭无效的服务及端口

```bash
#比如邮箱
service postfix status
chkconfig --del postfix
chkconfig postfix off

#比如cpus
service cups status
chkconfig --del cups
chkconfig cups off
```

### 设置防火墙策略

```bash
#或者用防火墙策略：
service iptables status
echo '请根据用户实际业务端口占用等情况进行设置！'
```

例如：
```bash
gedit /etc/sysconfig/iptables #添加如下策略
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 8080 -j ACCEPT
```

以下举例：
```bash
iptables -I INPUT -s 22.48.11.11 -j DROP
# 22.48.11.11的包全部屏蔽
iptables -I INPUT -s 22.48.11.0/24 -j DROP
#22.48.11.1到22.48.11.255的访问全部屏蔽
iptables -I INPUT -s 192.168.1.1 -p tcp --dport 80 -j DROP
# 192.168.1.1的80端口的访问全部屏蔽
iptables -I INPUT -s 192.168.1.0/24 -p tcp --dport 80 -j DROP
#192.168.1.1到192.168.1.255的80端口的访问全部屏蔽
service iptabels restart
#重启防火墙
```

### 设置历史记录数量

```bash
cp /etc/profile /etc/profile_xu_bak #（备份）
sed -i s/'HISTSIZE=1000'/'HISTSIZE=5000'/g /etc/profile  #（修改）
cat /etc/profile |grep HISTSIZE|grep -v export   #（检查）
```

### 配置用户最小权限

```bash
chmod 644 /etc/passwd
chmod 400 /etc/shadow
chmod 644 /etc/group
```

### 文件与目录缺省权限控制

```bash
cp /etc/profile /etc/profile.bak  #（备份）
gedit  /etc/profile
#增加
umask 027
source  /etc/profile
```

## 0x04 日志审计

### 启用远程日志功能

```bash
gedit /etc/rsyslog.conf
*.*         @Syslog日志服务器IP
```
>注意：* 和@之间存在的是tab键，非空格。

### 检查是否记录安全事件日志

```bash
gedit  /etc/syslog.conf 或者 /etc/rsyslog.conf
#在文件中加入如下内容:
*.err;kern.debug;daemon.notice     /var/log/messages
chmod 640 /var/log/messages
service rsyslog restart
```

### 日志保留半年以上

```bash
cp/etc/logrotate.conf /etc/logrotate.conf_xu_bak   #（备份）
sed -i s/'rotate 4'/'rotate 12'/g /etc/logrotate.conf   #（修改）
service syslog restart    #（重启）
cat /etc/logrotate.conf |grep -v '#' |grep rotate    #（检查）
```
