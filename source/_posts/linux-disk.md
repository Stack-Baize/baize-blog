---
title: Linux 存储扩容，分区扩容
date: '2020-06-10 11:00'
summary: 扩容存储盘指定分区，调整LVM分区存储空间大小
categories: 操作系统
tags:
  - Linux
  - CentOS
  - Ubuntu
abbrlink: fa9f
---

### 0x01 查看系统磁盘

使用`lsblk`可查看分区情况与磁盘大小，使用 `df -h` 命令，可以看到系统的磁盘使用情况，

```bash
lsblk
df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs          tmpfs     3.9G  8.6M  3.9G   1% /run
tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vda2      xfs        92G  3.2G   89G   4% /
tmpfs          tmpfs     783M     0  783M   0% /run/user/0
```

###  0x02 Linux 磁盘扩容情况

磁盘扩容时会有几种情况

1. 横向扩容（最后一个分区扩容）
2. LVM分区扩容

横向扩容需要扩容分区与未分区空间相邻，或最后一个分区 

LVM 分区扩容。主要的扩容方法有两种：

- 通过空余的磁盘进行扩容，这个方法比较简单，不会对原有数据有影响。

- 将其他 LVM 分区空间取出一部分给需要扩容的 LVM 分区。

下面就不同情况分别具体介绍。

### 0x03 LVM分区利用空余磁盘扩容

利用硬盘中空闲空间与添加别一个硬盘扩容基本相同，首先使用磁盘分区工具创建一个分区，再使用以下方式扩容。

1. 首先使用命令 fdisk -l 查看磁盘情况，此系统有两块硬盘， /dev/sda 21.5G， /dev/sdb 21.5G 

2. 创建 pv ,通过 pvcreate 命令将磁盘/dev/sdb 创建为一个系统 PV  

   ```bash
   pvcreate /dev/sdb1
   ```

   

3. 将 PV /dev/sdb 添加到卷组 VolGroup 中。磁盘已经添加到 VolGroup ，而且卷组的空间增加了 20G。使用命令

   ```bash
    vgextend VolGroup /dev/sdb1 
   ```

    

4. 为/ 添加 10G 的空间。使用命令 

   ```bash
   lvextend -r -L +10G /dev/mapper/VolGroup-lv_root
   #使用 -r 添加后自动刷新，不再需要第5步
   ```

   

5. 逻辑卷扩展后并不会马上生效，需要使用“resize2fs” 命令重新加载逻辑卷的大小。使用命令 

   ```bash
   resize2fs /dev/VolGroup/lv_root
   ```

   

再使用命令 df -h 查看发现/已经多了 10G。  



### 0x04 利用其他 LVM 分区空余空间进行扩容

1. 使用 df -h 查看每个分区的使用情况。如下，发现/dev/mapper/VolGroup-lv_home 容量很充裕，本次扩容通过减少

   /dev/mapper/VolGroup-lv_home 的空间给/dev/mapper/VolGroup-lv_root。  

   ```bash
   df -Th
   ```

   

2. 卸载/home  

   ```bash
   umount /home
   ```

   umount /home 如果提示无法卸载，因为有进程占用/home，使用如下命令来终止占用进程：

   ```bash
   fuser -m /home
   ```

   如果依然无法卸载，使用以下命令：  

   ```bash
   umount -l /home
   ```

   

3. 调整/dev/mapper/VolGroup-lv_home 分区大小
   需要先进行磁盘检测 ，输入命令 e2fsck -f /dev/mapper/VolGroup-lv_home。 注意：遇到 Abort< y >? 这边输入的是 n，才能继续进行。 

   ```bash
   e2fsck -f /dev/mapper/VolGroup-lv_home
   ```

    然后输入命令 resize2fs -p /dev/mapper/VolGroup-lv_home 100G，进行磁盘重订大小。  

   ```bash
   resize2fs -p /dev/mapper/VolGroup-lv_home 100G
   ```

   

4. 重新挂载/home  

   重新挂载后，输入 df -h，发现/dev/mapper/VolGroup-lv_home 已经改变。  

   ```bash
   mount /home
   df -Th
   ```

   

5. 设置空闲空间
   使用命令 vgdisplay，可以看到 Free PE/Size 25760 / 100.62 GiB，有了 100G 的空余空间。  

   ```bash
   lvreduce -L 100G /dev/mapper/VolGroup-lv_home
   ## 查询lvg空闲容量
   vgdisplay
   ```

   
   
6. 把闲置空间挂在到根目录下  

   刚才我们查询到还有 100.62G 的空闲空间，这时我们扩容空间时可以输入空间大小，也可以输入 +100%FREE 来表示扩容所以空闲容量

   ```bash
   lvextend -l +100%FREE /dev/mapper/VolGroup-lv_root
   ```

   使用命令 resize2fs -p /dev/mapper/VolGroup-lv_root， 可以不用重启，就显示最新的磁盘空间。  

   ```bash
   resize2fs -p /dev/mapper/VolGroup-lv_root
   ```

   

7. 查看结果 

   ```bash
   df -Th
   ```

   

### 0x05 利用parted 扩容分区（非活动分区）

parted 查看分区情况，只可扩容最后一个分区

```bash
parted /dev/vda print
parted /dev/vda
```

使用 resizepart 扩容最后一个分区，id为最后一个分区编号

```bash
resizepart id
End? [21.5GB]? 100%
```

结束位置大小输入 100% ，说明把后面所有空闲容量都加入

```bash
print
```

这时我们查看到已扩容完成

### 0x06 扩容GPT分区 （ gdisk 工具 ）

使用 parted 扩容活动分区时会提示卸载分区，这时我们就需要使用到gdisk 分区工具

```bash
yum install gdisk
# 安装 gdisk
[root@i-5C222F91 ~]# parted /dev/vda
# 输入 p 查看分区信息，这时会提示错误，输入 Fix 修复分区信息，q 退出。
[root@i-5C222F91 ~]# partprobe /dev/vda
# 刷新存储信息
[root@i-5C222F91 ~]# gdisk /dev/vda
Command (? for help): p
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1026047   500.0 MiB   EF00  EFI System Partition
   2         1026048         1640447   300.0 MiB   0700
   3         1640448        41936895   19.2 GiB    0700
#删除原分区
Command (? for help): d
Partition number (1-3): 3
#新建分区，序号使用原分区序号，一路回车
Command (? for help): n
Partition number (3-128, default 3): 3
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
[root@i-5C222F91 ~]# partprobe /dev/vda
[root@i-5C222F91 ~]# partprobe /dev/vda3
#刷新存储信息，查看分区情况
[root@i-5C222F91 ~]# lsblk
#更新扩容信息
[root@i-5C222F91 ~]# xfs_growfs /dev/vda3
```

### 0x07  扩容mbr分区（fdisk工具）

直接使用命令扩容系统分区会提示错误，这时我们可以使用 fdisk 扩容分区

```bash
## 刷新硬件信息
[root@i-351D0B02 ~]# partprobe /dev/vda
## 查看磁盘情况
[root@i-351D0B02 ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 1024M  0 rom
sr1     11:1    1 1024M  0 rom
vda    253:0    0  100G  0 disk
├─vda1 253:1    0    8G  0 part [SWAP]
└─vda2 253:2    0   22G  0 part /

[root@i-351D0B02 ~]# fdisk /dev/vda
Welcome to fdisk (util-linux 2.23.2).
Command (m for help): p
##查看原分区情况
Disk /dev/vda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000504a5

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1            2048    16779263     8388608   82  Linux swap / Solaris
/dev/vda2   *    16779264    62914559    23067648   83  Linux
##删除分区，删除分区后不可以保存
Command (m for help): d
Partition number (1,2, default 2): 2
Partition 2 is deleted
##重新创建分区
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
[root@i-351D0B02 ~]# partprobe /dev/vda
[root@i-351D0B02 ~]# partprobe /dev/vda2
[root@i-351D0B02 ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 1024M  0 rom
sr1     11:1    1 1024M  0 rom
vda    253:0    0  100G  0 disk
├─vda1 253:1    0    8G  0 part [SWAP]
└─vda2 253:2    0   92G  0 part /
## lsblk 可以查看到分区已扩容，这时可看到分区类型为xfs，需要自动扩展XFS文件系统到最大的可用大小。如为 ext4 等分区请使用 resize2fs /dev/vda2
[root@i-351D0B02 ~]# xfs_growfs /dev/vda2
meta-data=/dev/vda2              isize=512    agcount=4, agsize=1441728 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=5766912, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2815, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 5766912 to 24116992
[root@i-351D0B02 ~]# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs          tmpfs     3.9G  8.6M  3.9G   1% /run
tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vda2      xfs        92G  3.2G   89G   4% /
tmpfs          tmpfs     783M     0  783M   0% /run/user/0

```

