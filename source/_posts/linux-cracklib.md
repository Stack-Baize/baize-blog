---
title: Linux安全加固措施方案之密码加固
date: '2020-09-08 13:00'
summary: Linux系统被应用于大部分企业的服务器上，因此在等保测评中主机加固也是必须要完成的一项环节。
categories: 信息安全
tags:
  - Linux
  - Ubuntu
  - CentOS
  - 主机加固
abbrlink: d931
---


## 0x01 准备工作：

安装 PAM 的bai cracklib 模块，cracklib 能提供额外的密码du检查能力。

### 可用参数说明
- *debug* 此选项使模块的信息写入到syslog（3），显示模块的行为（此选项不写密码信息到日志文件）。

- *type=XXX* 默认的动作是模块使用以下提示时，要求口令：“新的UNIX密码：“和”重新输入UNIX密码：“。默认的Word UNIX可以被替换为这个选项。

- *retry=N* 改变输入密码的次数，默认值是1。就是说，如果用户输入的密码强度不够就退出。可以使用这个选项设置输入的次数，以免一切都从头再来。

- *difok=N* 默认值为10。这个参数设置允许的新、旧密码相同字符的个数。不过，如果新密码中1/2的字符和旧密码不同，则新密码被接受。

- *difignore=N* 多少个字符的密码应收到difok将被忽略。默认为23

- *minlen=N* 新的最低可接受的大小密码（加一个，如果没有禁用学分这是默认值）。除了在新密码的字符数，贷方（在长度+1），给出了各种人物的不同种类（其他，大写，小写，数字）。此参数的默认值是9，它是一个老式的UNIX密码的字符相同类型的所有好，但可能过低，利用一个MD5的系统增加安全性。请注意，有一个在Cracklib本身长度的限制，一“的方式太短“4极限是硬编码和定义的限制（6），将不参考minlen检查对。如果你想允许密码短短5个字符，你不应该使用这个模块。

- *dcredit=N* 限制新密码中至少有多少个数字。

- *ucredit=N* 限制新密码中至少有多少个大写字符。

- *lcredit=N* 限制新密码中至少有多少个小写字符。

- *ocredit=N* 限制新密码中至少有多少个其它的字符。

## 0x02 具体操作：

Debian、Ubuntu 或 Linux Mint 系统上：

```bash
$ sudo apt-get install libpam-cracklib
```

CentOS、Fedora、RHEL 系统已经默认安装了 cracklib PAM模块，所以在这些系统上无需执行上面的操作。

为了强制实施密码策略，需要修改 /etc/pam.d 目录下的 PAM 配置文件。一旦修改，策略会马上生效。
注意：此教程中的密码策略只对非 root 用户有效，对 root 用户无效。

策略设置：

### 禁止使用旧密码
找到同时有 “password” 和 “pam_unix.so” 字段并且附加有 “remember=5” 的那行，它表示禁止使用最近用过的5个密码（己使用过的密码会被保存在 /etc/security/opasswd 下面）。

Debian、Ubuntu 或 Linux Mint 系统上：

```bash
$ sudo vi /etc/pam.d/common-password
password [success=1 default=ignore] pam_unix.so obscure sha512 remember=5
```

CentOS、Fedora、RHEL 系统上：

```bash
$ sudo vi /etc/pam.d/system-auth
password sufficient pamunix.so sha512 shadow nullok tryfirstpass useauthtok remember=5
```

### 设置最短密码长度

找到同时有 “password” 和 “pam_cracklib.so” 字段并且附加有 “minlen=10” 的那行，它表示最小密码长度为（10 - 类型数量）。这里的 “类型数量” 表示不同的字符类型数量。PAM 提供4种类型符号作为密码（大写字母、小写字母、数字和标点符号）。如果密码同时用上了这4种类型的符号，并且 minlen 设为10，那么最短的密码长度允许是6个字符。

Debian、Ubuntu 或 Linux Mint 系统上：

```bash
$ sudo vi /etc/pam.d/common-password
password requisite pam_cracklib.so retry=3 minlen=10 difok=3
```

CentOS、Fedora、RHEL 系统上：

```bash
$ sudo vi /etc/pam.d/system-auth
password requisite pam_cracklib.so retry=3 difok=3 minlen=10
```

### 设置密码复杂度

找到同时有 “password” 和 “pam_cracklib.so” 字段并且附加有 “ucredit=-1 lcredit=-2 dcredit=-1 ocredit=-1” 的那行，表示密码必须至少包含一个大写字母（ucredit），两个小写字母（lcredit），一个数字（dcredit）和一个标点符号（ocredit）。

Debian、Ubuntu 或 Linux Mint 系统上：

```bash
$ sudo vi /etc/pam.d/common-password
password requisite pam_cracklib.so retry=3 minlen=10 difok=3 ucredit=-1 lcredit=-2 dcredit=-1 ocredit=-1
```

CentOS、Fedora、RHEL 系统上：

```bash
$ sudo vi /etc/pam.d/system-auth
password requisite pam_cracklib.so retry=3 difok=3 minlen=10 ucredit=-1 lcredit=-2 dcredit=-1 ocredit=-1
#允许有3个新、旧密码相同字符，最少长度10位，至少包含1位大写字母，2位小写字母，1位数字1个字符。
```

### 设置密码过期期限

编辑 /etc/login.defs 文件，可以设置当前密码的有效期限，具体变量如下所示：

```bash
$ sudo vi /etc/login.defs
PASSMAXDAYS 150
PASSMINDAYS 0
PASSWARNAGE 7
```

这些设置要求用户每6个月改变密码，并且会提前7天提醒用户密码快到期了。

如果想为每个用户设置不同的密码期限，使用 chage 命令。下面的命令可以查看某个用户的密码限期：

```bash
$ sudo chage -l xmoduloLast password change : Dec 30, 2013 Password expires : never Password inactive : never Account expires : never Minimum number of days between password change : 0 Maximum number of days between password change : 99999 Number of days of warning before password expires : 7
```

默认情况下，用户的密码永不过期。

### 下面的命令用于修改 xmodulo 用户的密码期限：

```bash
$ sudo chage -E 6/30/2014 -m 5 -M 90 -I 30 -W 14 xmodulo
```

上面的命令将密码期限设为2014年6月3日。另外，修改密码的最短周期为5天，最长周期为90天。密码过期前14天会发送消息提醒用户，过期后帐号会被锁住30天。

设置完后，验证效果。
