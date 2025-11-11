---
title: Centos7双网卡绑定配置 bonding
date: 2020-09-29 13:00
summary: '双网卡绑定技术在centos7中双网卡绑定既能使用teaming也可以使用bonding，这里使用bonding技术配置。'
categories: 系统配置
tags:
 - CentOS
 - Linux
 - Ubuntu
abbrlink: e3a7
---

## 0x01 简介

当linux系统上有多个单独网卡，又想充分利用这些网卡，同时对外提供一个统一的网络地址，以使得增大网络的吞吐量，同时也提高网络的可用性，这时就需要bond来帮助我们解决这个问题。

Linux网卡绑定mode共有七种(0~6) bond0、bond1、bond2、bond3、bond4、bond5、bond6，接下来我们一起简单看下这7中模式的工作原理概述。

### bond几种主要模式介绍
- 第一种模式：mod=0 ，即：(balance-rr) Round-robin policy（平衡抡循环策略）

bond0工作原理：

传输数据包顺序是依次传输（即：第1个包走eth0，下一个包就走eth1….一直循环下去，直到最后一个传输完毕），此模式提供负载平衡和容错能力；但是我们知道如果一个连接或者会话的数据包从不同的接口发出的话，中途再经过不同的链路，在客户端很有可能会出现数据包无序到达的问题，而无序到达的数据包需要重新要求被发送，这样网络的吞吐量就会下降

> 特点：有高可用 (容错) 和负载均衡的功能, 需要交换机的配置，每块网卡轮询发包 (流量分发比较均衡).

- 第二种模式：mod=1，即： (active-backup) Active-backup policy（主-备份策略）

bond1工作原理：

只有一个设备处于活动状态，当一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得，从外面看来，bond的MAC地址是唯一的，以避免switch(交换机)发生混乱。此模式只提供了容错能力；由此可见此算法的优点是可以提供高网络连接的可用性，但是它的资源利用率较低，只有一个接口处于工作状态，在有N个网络接口的情况下，资源利用率为1/N

> 特点：只有高可用 (容错) 功能, 不需要交换机配置, 这种模式只有一块网卡工作, 对外只有一个mac地址。缺点是端口利用率比较低

- 第三种模式：mod=2，即：(balance-xor) XOR policy（平衡策略）

bond2工作原理：

基于指定的传输HASH策略传输数据包。缺省的策略是：(源MAC地址 XOR 目标MAC地址) % slave数量。其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力

> 特点：基于指定的传输HASH策略传输数据包。缺省的策略是：(源MAC地址 XOR 目标MAC地址) % slave数量。其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力，但不常用

- 第四种模式：mod=3，即：broadcast（广播策略）

bond3工作原理：

在每个slave接口上传输每个数据包，此模式提供了容错能力。

>  特点：在每个slave接口上传输每个数据包，此模式提供了容错能力，不常用

- 第五种模式：mod=4，即：(802.3ad) IEEE 802.3ad Dynamic link aggregation（IEEE 802.3ad 动态链接聚合）

bond4工作原理：

创建一个聚合组，它们共享同样的速率和双工设定。根据802.3ad规范将多个slave工作在同一个激活的聚合体下。外出流量的slave选举是基于传输hash策略，该策略可以通过xmit_hash_policy选项从缺省的XOR策略改变到其他策略。需要注意的是，并不是所有的传输策略都是802.3ad适应的，尤其考虑到在802.3ad标准43.2.4章节提及的包乱序问题。不同的实现可能会有不同的适应性。

*必要条件：*
- 条件1：ethtool支持获取每个slave的速率和双工设定
-  条件2：switch(交换机)支持IEEE 802.3ad Dynamic link aggregation
-  条件3：大多数switch(交换机)需要经过特定配置才能支持802.3ad模式

> 特点： IEEE 802.3ad 动态链路聚合，需要交换机支持与配置交换机

- 第六种模式：mod=5，即：(balance-tlb) Adaptive transmit load balancing（适配器传输负载均衡）

bond5工作原理：

不需要任何特别的switch(交换机)支持的通道bonding。在每个slave上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。

该模式的必要条件：ethtool支持获取每个slave的速率

> 特点：该模式的必要条件：ethtool支持获取每个slave的速率。不常用

- 第七种模式：mod=6，即：(balance-alb) Adaptive load balancing（适配器适应性负载均衡）

bond6工作原理：

该模式包含了balance-tlb模式，同时加上针对IPV4流量的接收负载均衡(receive load balance, rlb)，而且不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的。bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

来自服务器端的接收流量也会被均衡。当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。当ARP应答从对端到达时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。使用ARP协商进行负载均衡的一个问题是：每次广播ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。这个问题可以通过给所有的对端发送更新 （ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。当新的slave加入到bond中时，或者某个未激活的slave重新 激活时，接收流量也要重新分布。接收的负载被顺序地分布（round robin）在bond中最高速的slave上

当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。

下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答不会被switch(交换机)阻截。

*必要条件：*
- 条件1：ethtool支持获取每个slave的速率；
- 条件2：底层驱动支持设置某个设备的硬件地址，从而使得总是有个slave(curr_active_slave)使用bond的硬件地址，同时保证每个bond 中的slave都有一个唯一的硬件地址。如果curr_active_slave出故障，它的硬件地址将会被新选出来的 curr_active_slave接管

其实mod=6与mod=0的区别：mod=6，先把eth0流量占满，再占eth1，….ethX；而mod=0的话，会发现2个口的流量都很稳定，基本一样的带宽。而mod=6，会发现第一个口流量很高，第2个口只占了小部分流量。

> 特点：有高可用 ( 容错 )和负载均衡的功能，不需要交换机配置 (流量分发到每个接口不是特别均衡)
服务器上两张物理网卡em1和em2, 通过绑定成一个逻辑网卡bond0。

注: ip地址配置在bond0上, 物理网卡不需要配置ip地址。

## 0x02 手动配置bond

### 关闭和停止NetworkManager服务

一定要关闭，不关会对做bonding有干扰

```bash
systemctl stop NetworkManager.service
# 停止NetworkManager服务
systemctl disable NetworkManager.service
# 禁止开机启动NetworkManager服务
```

### 加载bonding模块

```bash
modprobe --first-time bonding
```

没有提示说明加载成功, 如果出现 modprobe: ERROR: could not insert 'bonding': Module already in kernel 说明你已经加载了这个模块, 就不用管了

你也可以使用`lsmod | grep bonding`查看模块是否被加载

```bash
lsmod | grep bonding
bonding 136705 0
```

### 创建基于bond0接口的配置文件

```bash
/etc/sysconfig/network-scripts/ifcfg-bond0
#修改成如下，根据你的情况:
DEVICE=bond0
TYPE=Bond
IPADDR=172.16.0.183
NETMASK=255.255.255.0
GATEWAY=172.16.0.1
DNS1=114.114.114.114
USERCTL=no
BOOTPROTO=none
ONBOOT=yes
BONDING_MASTER=yes
BONDING_OPTS="miimon=100 mode=4 xmit_hash_policy=layer3+4"
#上面这个参数如已配置mode在系统文件中可注释
```

### 将需求mode配置在系统文件中

如在bond0接口上已配置不再须要配置

```bash
vim /etc/modprobe.d/bond.conf
#添加以下内容：
alias bond0 bonding
options bond0 miimon=100 mode=4 xmit_hash_policy=layer3+4
```

### 修改em1接口的配置文件

```bash
vim /etc/sysconfig/network-scripts/ifcfg-em1
#修改成如下:
DEVICE=em1
USERCTL=no
ONBOOT=yes
MASTER=bond0
# 需要和上面的ifcfg-bond0配置文件中的DEVICE的值对应
SLAVE=yes
BOOTPROTO=none
```

### 修改em2接口的配置文件

```bash
vim /etc/sysconfig/network-scripts/ifcfg-em2
#修改成如下:
DEVICE=em2
USERCTL=no
ONBOOT=yes
MASTER=bond0
# 与 ifcfg-bond0 配置文件中的DEVICE的值对应
SLAVE=yes
BOOTPROTO=none
```

## 0x03 nmcli命令配置

使用 nmcli 命令配置时需要启动 NetworkManager 服务

```bash
##查看网卡信息
nmcli connection show
##配置bond0链路聚合为主从模式
nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup,miimon=100"
##配置bond0网卡地址
nmcli connection modify bond0 ipv4.method manual ipv4.addresses '172.16.0.183/24' ipv4.gateway '172.16.0.254'
##刷新网络
nmcli connection reload
##添加网络到绑定网络，其中 enp3s0f0 按情况变更
nmcli connection add type bond-slave ifname enp3s0f0 master bond0
nmcli connection add type bond-slave ifname enp3s0f1 master bond0
##启动网络
nmcli connection up  bond0
```

## 0x04 测试

重启网络服务

```bash
systemctl restart network
ifconfig
#查看网络是否正常，其中bond0接口mac地址与em1等接口相同
ifconfig em1 down
#关闭一个接口查看网络是否正常
cat /proc/net/bonding/bond0
#查看网卡状态
ethtool bond0
#查看速率命令
```
