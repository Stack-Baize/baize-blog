---
title: Linux iostat命令详解
abbrlink: 0003
date: 2020-03-05 13:00
categories: Linux命令
tags:
- Linux
- iostat
---

iostat是I/O statistics（输入/输出统计）的缩写，iostat工具将对系统的磁盘操作活动进行监视。它的特点是汇报磁盘活动统计情况，同时也会汇报出CPU使用情况。iostat也有一个弱点，就是它不能对某个进程进行深入分析，仅对系统的整体情况进行分析

## 常见命令展示

### iostat 安装

``` bash
# iostat属于sysstat软件包。可以直接安装。
yum install sysstat
```

### 显示所有设备负载情况


``` bash
[root@i-003F281E ~]# iostat
Linux 3.10.0-1062.4.1.el7.x86_64 (i-003F281E) 	2020年03月05日 	_x86_64_	(4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.65    0.00    0.58    0.38    0.01   97.38

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda              10.84       126.28      1034.29     317347    2599151
dm-0             11.81       120.22      1011.98     302106    2543083
dm-1              0.05         1.29         0.00       3236          0
dm-2              0.03         0.43         0.81       1090       2048
```

cpu属性值说明：

%user：CPU处在用户模式下的时间百分比。

%nice：CPU处在带NICE值的用户模式下的时间百分比。

%system：CPU处在系统模式下的时间百分比。

%iowait：CPU等待输入输出完成时间的百分比。

%steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。

%idle：CPU空闲时间百分比。


备注：

如果%iowait的值过高，表示硬盘存在I/O瓶颈

如果%idle值高，表示CPU较空闲

如果%idle值高但系统响应慢时，可能是CPU等待分配内存，应加大内存容量。

如果%idle值持续低于10，表明CPU处理能力相对较低，系统中最需要解决的资源是CPU。


cpu属性值说明:

tps：该设备每秒的传输次数

kB_read/s：每秒从设备（drive expressed）读取的数据量；

kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；

kB_read：  读取的总数据量；

kB_wrtn：写入的总数量数据量；

### 定时显示所有信息

``` bash
#【每隔2秒刷新显示，且显示3次】
iostat 2  3
```

### 显示指定磁盘信息

``` bash
iostat -d /dev/sda
```

### 显示tty和Cpu信息

``` bash
iostat -t
```

### 以M为单位显示所有信息

``` bash
iostat -m
```

### 查看设备使用率（%util）、响应时间（await）

``` bash
#  【-d 显示磁盘使用情况，-x 显示详细信息】
#  d: detail
iostat -d -x -k 1 1

[root@i-003F281E ~]# iostat -d -x -k 1 1
Linux 3.10.0-1062.4.1.el7.x86_64 (i-003F281E) 	2020年03月05日 	_x86_64_	(4 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.01     1.86    4.47    6.09   123.00  1007.42   214.01     0.70   92.65    5.20  156.86   2.58   2.72
dm-0              0.00     0.00    4.21    7.31   117.09   985.69   191.59     1.30  112.62    5.71  174.19   2.33   2.68
dm-1              0.00     0.00    0.05    0.00     1.25     0.00    50.96     0.00    1.35    1.35    0.00   0.96   0.00
dm-2              0.00     0.00    0.03    0.00     0.42     0.79    72.98     0.00   10.29    2.84  131.00   3.81   0.01
```

说明：

rrqm/s：  每秒进行 merge 的读操作数目.即 delta(rmerge)/s

wrqm/s： 每秒进行 merge 的写操作数目.即 delta(wmerge)/s

%util： 一秒中有百分之多少的时间用于 I/O

如果%util接近100%，说明产生的I/O请求太多，I/O系统已经满负荷

   idle小于70% IO压力就较大了，一般读取速度有较多的wait。


### 查看cpu状态

``` bash
iostat -c 1 1

[root@i-003F281E ~]# iostat -c 1 1
Linux 3.10.0-1062.4.1.el7.x86_64 (i-003F281E) 	2020年03月05日 	_x86_64_	(4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.57    0.00    0.55    0.36    0.01   97.51
```

