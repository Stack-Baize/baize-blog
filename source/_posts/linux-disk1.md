---
title: Linux磁盘分区并挂载
date: '2021-10-20 11:05'
summary: Linux系统一般都会有未挂载的磁盘，如果我们想使用这些为挂载的磁盘就需要挂载到指定目录才能使用。
categories: 操作系统
tags:
  - Linux
  - CentOS
  - Ubuntu
abbrlink: 76b2
---

## 0x01 查看系统磁盘

使用`lsblk`可查看分区情况与磁盘大小，使用 `df -h` 命令，可以看到系统的磁盘使用情况，

如需要挂载 `/dev/sdb` 此存储到 `/data` 目录

```bash
##查看设备中磁盘
lsblk
##查看磁盘挂载与使用情况
df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs          tmpfs     3.9G  8.6M  3.9G   1% /run
tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vda2      xfs        92G  3.2G   89G   4% /
tmpfs          tmpfs     783M     0  783M   0% /run/user/0
```

##  0x02 Linux 磁盘分区

### 标准分区挂载

#### fdisk 分区工具

磁盘少于2T时可以使用`fdisk`分区，大于2T需要使用 `gdisk`工具

```bash
## 刷新硬件信息
[root@i-351D0B02 ~]# partprobe
## 查看磁盘情况
[root@i-351D0B02 ~]# lsblk
##新建分区
[root@i-351D0B02 ~]# fdisk /dev/vda
Welcome to fdisk (util-linux 2.23.2).
##创建分区
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 2
First sector (16779264-209715199, default 16779264):
Using default value 16779264
Last sector, +sectors or +size{K,M,G} (16779264-209715199, default 209715199):
Using default value 209715199
Partition 2 of type Linux and of size 92 GiB is set
## 再次查看分区
Command (m for help): p

Disk /dev/vda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000504a5

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1            2048    16779263     8388608   82  Linux swap / Solaris
/dev/vda2        16779264   209715199    96467968   83  Linux
## 检查分区是不否有错误
Command (m for help): v
Remaining 2047 unallocated 512-byte sectors
## 保存分区信息
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
## 刷新存储文件，centos6 使用 kpartx /dev/vda
[root@i-351D0B02 ~]# partprobe
##格式化分区
[root@i-351D0B02 ~]# mkfs.xfs /dev/vda2
##修改fstab并挂载
[root@i-351D0B02 ~]# vim /etc/fstab
/dev/vda2    /data   xfs     defaults   0 0
[root@i-351D0B02 ~]# mount -a
##查看分区挂载
[root@i-351D0B02 ~]# df -Th
```



#### gdisk 分区工具

```bash
##安装gdisk包
yum install gdisk
##刷新存储
[root@i-5C222F91 ~]# partprobe
##创建分区
[root@i-5C222F91 ~]# gdisk /dev/vda
Command (? for help): n
Partition number (3-128, default 3): 
#查看分区情况
Command (? for help): p
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1026047   500.0 MiB   EF00  EFI System Partition
   2         1026048         1640447   300.0 MiB   0700
   3         1640448       104857566   49.2 GiB    8300  Linux filesystem
#修改分区类型
Command (? for help): t
Partition number (1-3): 3
Hex code or GUID (L to show codes, Enter = 8300): 0700
#检查分区情况
Command (? for help): v
#保存分区
Command (? for help): w

Do you want to proceed? (Y/N): y
#写入分区信息
##刷新存储
[root@i-5C222F91 ~]# partprobe
##格式化分区
[root@i-351D0B02 ~]# mkfs.xfs /dev/vda2
##修改fstab并挂载
[root@i-351D0B02 ~]# vim /etc/fstab
/dev/vda2    /data   xfs     defaults   0 0
[root@i-351D0B02 ~]# mount -a
##查看分区挂载
[root@i-351D0B02 ~]# df -Th
```

#### parted 分区工具

linux操作系统下有fdisk和parted两个分区工具，超过2T的磁盘只能使用parted进行分区，fdisk和parted分区方法也有很大的不同

使用`parted`分区工具需要注意磁盘数据，运行命令就已经开始执行分区操作

```bash
####刷新存储
[root@i-5C222F91 ~]# partprobe
##创建分区
[root@i-5C222F91 ~]# parted  /dev/sdb
##转换磁盘为gpt
(parted)mklable gpt
##创建分区,0为开始分区大小，2T为结束分区大小
(parted)mkpart primary 0 2T
(parted)p
(parted)q
##刷新存储
[root@i-5C222F91 ~]# partprobe
##格式化分区
[root@i-351D0B02 ~]# mkfs.xfs /dev/vda2
##修改fstab并挂载
[root@i-351D0B02 ~]# vim /etc/fstab
/dev/vda2    /data   xfs     defaults   0 0
[root@i-351D0B02 ~]# mount -a
##查看分区挂载
[root@i-351D0B02 ~]# df -Th
```



### LVM分区挂载

由于传统的磁盘管理不能对磁盘进行磁盘管理，因此诞生了LVM技术，LVM技术最大的特点就是对磁盘进行动态管理。由于LVM的逻辑卷的大小更改可以进行动态调整，且不会出现丢失数据的情况。

LVM（Logic Volume Manager）是逻辑卷管理的简称。它是Linux环境下对磁盘分区管理的一种机制。对于其他的的UNIX（AIX/HP/SUM)操作系统，以及Windows系统也有类似的磁盘管理软件。

LVM管理的方式非常简单，就是通过将底层的物料磁盘抽象并封装起来，然后以逻辑的方式呈现给上层应用。

首先需要对磁盘分区，再把一个或多个磁盘分区加入pv

```bash
## 刷新硬件信息
[root@i-351D0B02 ~]# partprobe
## 查看磁盘情况
[root@i-351D0B02 ~]# lsblk
##新建分区
[root@i-351D0B02 ~]# fdisk /dev/vda
Welcome to fdisk (util-linux 2.23.2).
##创建分区
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 2
First sector (16779264-209715199, default 16779264):
Using default value 16779264
Last sector, +sectors or +size{K,M,G} (16779264-209715199, default 209715199):
Using default value 209715199
Partition 2 of type Linux and of size 92 GiB is set
## 再次查看分区
Command (m for help): p

Disk /dev/vda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000504a5

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1            2048    16779263     8388608   82  Linux swap / Solaris
/dev/vda2        16779264   209715199    96467968   83  Linux
## 检查分区是不否有错误
Command (m for help): v
Remaining 2047 unallocated 512-byte sectors
## 保存分区信息
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
## 刷新存储文件，centos6 使用 kpartx /dev/vda
[root@i-351D0B02 ~]# partprobe
##创建pv卷
[root@i-351D0B02 ~]# pvcreate /dev/vda2
##创建vg卷组
[root@i-351D0B02 ~]# vgcreate data /dev/vda2
##创建lv逻辑卷
[root@i-351D0B02 ~]# lvcreate -l 100%FREE -n data data
##格式化逻辑卷
[root@i-351D0B02 ~]# mkfs.xfs /dev/mapper/data-data
##修改fstab并挂载
[root@i-351D0B02 ~]# vim /etc/fstab
/dev/mapper/data-data    /data   xfs     defaults   0 0
[root@i-351D0B02 ~]# mount -a
##查看分区挂载
[root@i-351D0B02 ~]# df -Th
```


-------

**相关链接**

- [Linux存储扩容，分区扩容](https://baize.cc/posts/fa9f.html)

