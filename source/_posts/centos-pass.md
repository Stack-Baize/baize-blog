---
title: Linux系统单用户模式
date: '2020-05-02 11:00'
summary: 在我们运维工作中经常会见到忘记密码，或工作交接时未及时交换密码，造成无法登录密码，这时我们就须要重置密码。
categories: 操作系统
tags:
  - Linux
  - CentOS
abbrlink: '9014'
---

### 0x01 单用户模式简介
忘记root密码这个问题出现的几率是很高的，不过，在linux下解决这个问题也很简单，只需重启linux系统，然后引导进入linux的单用户模式（init1），由于单用户模式是不需要输入登录密码的，因此，可以直接登录系统，修改root密码即可解决问题。

Centos6启动时读取的文件为：`/etc/grub.cfg`

Centos7启动时读取的文件为：`/etc/grub2.cfg`

### 0x02 CentOS6进入单用户模式

1、重启系统，进入系统欢迎界面按上下左右键进入GRUB界面；

2、在GRUB界面选择内核版本，按下'e'键；

3、在此界面可以进行编辑，在最后输入`single`再按回车键返回,选择kernel这行，并按下'b'键进入单用户模式。

4、修改密码

```bash
$ Passwd root #对root密码进行修改
```

5、reboot进行系统重启

### 0x03 CentOS7进入单用户模式

1、进入GRUB页面，选择相应的内核，按下'e'键；
备注：第一行为内核；第二行为援救模式。

2、进入内核修改信息界面，找到Linux16这一行；在这一行的末尾加上 `init=/bin/sh`按下 `Ctrl + x`进入单用户模式

3、进入单用户后，重新挂载根目录，使其可写；

```bash
$ mount -o remount,rw /
```

4、修改字符集（可选）

```bash
$ Locale #查看当前字符集
$ export LANG=en_US
```

将终端的字符集改为英文

5、修改密码

```bash
$ passwd root
```

6、当selinux防火墙启动时，修改密码后要创建文件

```bash
$ touch /.autorelabel
```
否则在系统重启时无法重启

7、重启系统

```bash
$ exec /sbin/init
```

### 0x04 CentOS8进入单用户模式

在开机引导时按 e 键进行编辑，在linux这一行的最末尾输入`rd.break`，并删除console相关的内容，按ctrl+x 继续启动


```bash
linux ...    rd.break
##按 ctrl+x 继续启动
mount -o remount,rw /sysroot/
chroot /sysroot/
```

### 0x05 OpenEuler进入单用户模式

openeuler为华为提供社区维护系统，引导界面输入 `e` ，linux这行最后面输入：`init=/bin/sh`  然后ctrl+x进入界面

```bash
init=/bin/sh
mount -o remount,rw /
##重置密码
passwd
##输入新密码2次
touch /.autorelabel
exit
##重启系统
```

### 0x06 UOS Euler进入单用户模式

引导输入 `e` 在linux行将 `ro` 修改为 `rw`，该行最后添加 `single console=ttyS0`，最后按ctrl+x进入系统

```bash
single console=ttyS0
```

经测试有时无法成功，可参考openeuler系统


### 0x07 Kylin进入单用户模式

引导输入 `e` 在linux行把 `quiet splash` 删除，添加如下

```bash
init=/bin/bash console=tty0
```

进入时需要输入用户 `root` 密码 `Kylin123123`


