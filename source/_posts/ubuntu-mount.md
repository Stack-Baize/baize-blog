---
title: ubuntu 16.04 硬盘分区，挂载，硬盘分区方案
abbrlink: 0005
date: '2020-03-25 13:00'
categories: 操作系统
tags:
 - Linux
 - Ubuntu
 - 硬盘分区
---

## Ubuntu 挂载硬盘分区

### 0x01 查看硬盘及所属分区情况

```bash
$ lsblk
NAME           MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda              8:0    0 465.8G  0 disk  
sda1           8:1    0   512M  0 part  /boot/efi
sda2           8:2    0 464.3G  0 part  /
sda3           8:3    0   976M  0 part  
  cryptswap1 252:0    0 975.5M  0 crypt
sdb              8:16   0   5.5T  0 disk  
sdc              8:32   0   5.5T  0 disk  
#可以查看上面有三块硬盘，二块未分配
sudo fdisk -lu
```
![fdisk -lu](/medias/ubuntu/1585103977.jpg)

显示当前的硬盘及所属分区的情况。如图所示：
图中有两块硬盘，我们要对第二块硬盘进行分区。
上面480G是我安装ubuntu的位置。但是2个2T的机械硬盘没有识别出来。


### 0x02 对硬盘进行分区

我现在先分区/dev/sdb。再挂载这一块硬盘。

```bash
sudo fdisk /dev/sdb
#输入`n`
#输入`p`
#完成后输入`w`保存
```

GPT格式分区
![fdisk sdb](/medias/ubuntu/1585104274.jpg)

如有必要可重新初始化硬盘,再格式化

``` bash
parted /dev/sdb mklabel gpt
mkfs.ext4 /dev/sdb
```

### 0x03 查看刚刚操作的硬盘详情。

``` bash
sudo fdisk -l
```

![fdisk l](/medias/ubuntu/1585104428.jpg)

已经发现/dev/sdb这一块硬盘type 修改为gpt。

### 0x04 格式化该分区

将分区格式化成ext4文件系统类型，无法进入和查看。

``` bash
sudo mkfs -t ext4 /dev/sdb1
```

![mkfs ext4](/medias/ubuntu/1585104525.jpg)

### 0x05 挂载硬盘分区
新硬盘需要挂载在一个新的目录下面。且该目录应该为空。
我首先创建一个文件夹。

``` bash
sudo mkdir /data_1
#再把该硬盘挂载在/data_1下面。
sudo mount -t ext4 -o rw /dev/sdb1 /data_1/
```

### 0x06 配置硬盘在系统启动自动挂载

查看/dev/sdb1 这个分区的UUID

``` bash
sudo blkid /dev/sdb1
#其它方式获取 UUID 
blkid /dev/sdb |awk '{print $2}'|sed 's/"//g'
UUID=0b238fbf-ea33-49dd-bb5c-adf7b763d9c6
#打开文件/etc/fstab
sudo gedit /etc/fstab
#增加一行
UUID=0b238fbf-ea33-49dd-bb5c-adf7b763d9c6 /data ext4 defaults 0 0
#此处UUID为上面找到的
```
### 0x07 检查并挂载新添项

``` bash
sudo mount -a
```

mount -a 会/etc/fstab中的项全部挂载，如果有错，则会提示错误，然后根据错误找出原因修改。

注：修改/etc/fstab 一定注意，不要修改错误，很有可能就重启进不了系统，我之前就是修改错误，没有进去系统，我也将修改错误，最后怎么修改进去系统的步骤写出来。

进入grub模式，修改/etc/fstab

``` bash
sudo vi /etc/fstab
```

把最后自己增加的删除掉。使用方法请自己百度VIM使用。
最后esc返回。输入：wq 保存。关机重启即可。
