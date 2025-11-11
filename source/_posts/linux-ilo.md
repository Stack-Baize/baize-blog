---
title: Linux 查看服务器远程管理IP(DELL iDrac和HP iLO)
date: '2020-05-20 11:00'
summary: 在我们运维工作中经常会有使用ilo口管理服务器情况，这时如果忘记啦或都不知道默认IP，这时我们就可以在系统中查看。
categories: 操作系统
tags:
  - Linux
  - CentOS
  - Ubuntu
abbrlink: 7f56
---


### CentOS

```bash
yum install -y OpenIPMI ipmitool
[root@localhost ~]# ipmitool lan print
Set in Progress         : Set Complete
IP Address Source       : Static Address
IP Address              : 10.1.6.200
Subnet Mask             : 255.255.255.0
MAC Address             : e4:72:e2:c8:70:a5
SNMP Community String   : TrapAdmin12#$
IP Header               : TTL=0x40 Flags=0x40 Precedence=0x00 TOS=0x10
Default Gateway IP      : 10.1.6.254
```

### Ubuntu

```bash
apt-get install ipmitool
[root@localhost ~]# ipmitool lan print
Set in Progress         : Set Complete
IP Address Source       : Static Address
IP Address              : 10.1.6.200
Subnet Mask             : 255.255.255.0
MAC Address             : e4:72:e2:c8:70:a5
SNMP Community String   : TrapAdmin12#$
IP Header               : TTL=0x40 Flags=0x40 Precedence=0x00 TOS=0x10
Default Gateway IP      : 10.1.6.254
```
