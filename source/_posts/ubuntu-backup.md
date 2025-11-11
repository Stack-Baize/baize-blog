---
title: Ubuntu全盘备份与恢复，亲自总结，实测可靠
date: '2020-04-20 11:00'
summary: 初学者在使用Ubuntu这类Linux操作系统时，常常会由于不当操作导致系统崩溃，重装系统是难免的事情。
categories: 数据备份
tags:
  - Linux
  - Ubuntu
  - 备份
  - 恢复
abbrlink: 58b2
---

我该如何备份我的Ubuntu系统呢？很简单，就像你备份或压缩其它东西一样，使用TAR。和Windows不同，Linux不会限制root访问任何东西，你可以把分区上的所有东西都扔到一个TAR文件里去！

## Ubuntu 备份与恢复系统

### 系统备份

>使用TAR。和Windows不同，Linux不会限制root访问任何东西，你可以把分区上的所有东西都扔到一个TAR文件里去

打开一个终端，并运行 sudo su（回车后要求输入密码）；

继续在终端中输入以下指令，进入系统根目录：

```bash
$ cd /
#开始备份系统，在终端中输入
$ tar cvpzf backup.tgz --exclude=/proc --exclude=/lost+found --exclude=/backup.tgz --exclude=/mnt --exclude=/sys --exclude=/media /
```

让我们来简单看一下这个命令：

`tar` 是用来备份的程序
- c 新建一个备份文档
- v 详细模式， tar程序将在屏幕上实时输出所有信息。
- p 保存许可，并应用到所有文件。
- z 采用‘gzip’压缩备份文件，以减小备份文件体积。
- f 说明备份文件存放的路径， Ubuntu.tgz 是本例子中备份文件名。
- "/" 是我们要备份的目录，在这里是整个文件系统。

>在档案文件名"backup.tgz"和要备份的目录名"/"之间给出了备份时必须排除在外的目录。有些目录是无用的，例如"/proc"、"/lost + found"、"/sys"。当然，"backup.tgz"这个档案文件本身必须排除在外，否则你可能会得到一些超出常理的结果。如果不把"/mnt"排除在外，那么挂载在"/mnt"上的其它分区也会被备份。另外需要确认一下"/media"上没有挂载任何东西(例如光盘、移动硬盘)，如果有挂载东西， 必须把"/media"也排除在外.

备份完成后，在文件系统的根目录将生成一个名为"backup.tgz"的文件，它的尺寸有可能非常大。现在你可以把它烧录到DVD上或者放到你认为安全的地方去。
在备份命令结束时你可能会看到这样一个提示："tar: Error exit delayed from previous errors"，多数情况下你可以忽略它。

### 恢复系统

如果原来的Ubuntu系统已经崩溃，无法进入。则可以使用Ubuntu安装U盘（live USB）进入试用Ubuntu界面。

切换到root用户，找到之前Ubuntu系统的根目录所在磁盘分区（一般电脑上的磁盘分区（假设分区名称为sdaX）均可以在当前Ubuntu系统的根目录下的media目录下（即/media）找到。目录通常为当前根目录下 cd /media/磁盘名称/分区名称）。进入该分区，输入以下指令来删除该根目录下的所有文件：

```bash
$ sudo rm -rf /media/磁盘名称/分区名称*
#将备份文件”backup.tgz”拷入该分区；
$ sudo cp -i backup.tgz /media/磁盘名/分区名sdaX
#进入分区并将压缩文件解压缩，参数x是告诉tar程序解压缩备份文件。
$ sudo tar xvpzf backup.tgz
#如果你的档案文件是使用Bzip2压缩的，应该用
$ sudo tar xvpjf backup.tar.bz2 -C /
#注意：上面的命令会用档案文件中的文件覆盖分区上的所有文件。
#重新创建那些在备份时被排除在外的目录；
$ sudo mkdir proc lost+found mnt sys media
#或者这样：
mkdir proc
mkdir lost+found
mkdir mnt
mkdir sys
#如有必要检查下分区与引导
#查看当前分区UUID命令
$ blkid /dev/sdb1
#修改引导
$ vi /etc/fstab
#安装grub引导
$ grup-install /dev/sdb
#更新引导
$ update-grub2
#检查分区挂载是否正常
$ mount -a
```
当你重启电脑，你会发现一切东西恢复到你创建备份时的样子了！

## 备份工具
常用工具列表

- dd                   数据复制,转换实用工具
- tar                  GNU磁盘存档实用工具
- cpio                数据存档实用工具
- dump/restore

### dd 命令

备份mbr

```bash
 dd if=/dev/sda of=/backup/mbr.img bs=512 count=1
```

还原mbr

```bash
 dd if=/backup/mbr.img of=/dev/sda bs=446 count=1
```

还原分区表,跳过主引导记录

```bash
 dd if=/backup/mbr.img of=/dev/sda bs=1 count=64 skip=446 seek=446
```

### GNU/TAR

备份

```bash
 tar -cpzvf backup.tar.gz /media/usb/*
```

还原

```bash
 tar -xpzvf backup.tar.gz -C /media/usb/
```

### xfsdump/xfsrestore

备份

```bash
 sudo xfsdump - /boot > backup.file
```

还原

```bash
 sudo cat backup.file | xfsrestore - /boot
```

### 救援工具

可启动光盘

`Redo Backup and Recovery`

开源启动光盘的备份和恢复工具,具有GUI界面.
