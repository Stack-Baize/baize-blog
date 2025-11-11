---
title: Linux 系统修复
date: '2020-09-07 11:00'
summary: 有时我们会因为各种问题导致操作系统损坏，如何修复呢？现在我们通过光盘来修复。
categories: 操作系统
tags:
  - Linux
  - CentOS
  - Ubuntu
abbrlink: 462d
---

### CentOS 系统修复

首先，光盘引导选择`Troubleshooting`-`Rescue a CentOS system`,进入光盘引导修改系统，进入界面后输入`3`，进入`Skip to shell`, shell 操作界面

```bash
$ lsblk #查看磁盘情况，随意挂载个磁盘查看是否为根目录。
$ mkdir /demo 
$ mount /dev/vda3 /demo
$ ls  #查看目录是否为根目录
$ umount /demo   #卸载磁盘
$ blkid  /dev/vda3  #查看磁盘分区格式
$ fsck  -fy   /dev/vda3    #ext格式使用此命令修复
$ xfs_repair /dev/vda3     #xfs 格式使用此命令修复
done     #说明修复完了，把iso卸载掉然后reboot就好
```

> **Ubuntu**系统修复相同

