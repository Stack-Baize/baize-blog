---
title: 怎么搭建NTP时间服务器
abbrlink: 0007
date: 2020-04-18 11:00
summary: '为实现多个设备统一时钟，需要配置NTP时钟服务器。本操作过程详细介绍如何使用windows Server与CentOS操作系统分别搭建时钟服务器。'
categories: 环境部署
tags:
- Linux
- CentOS
- Windows Server
- NTP
---

## Linux中怎么搭建NTP服务器

>说明：本次实验采用两台加载CentOS7.7版本的镜像文件的虚拟机，要求一台配置好时间服务之后，由另一台进行同步。

说在前面：ntp和ntpdate区别
- 两个服务都是centos自带的（centos7中不自带ntp）。ntp的安装包名是ntp；ntpdate的安装包是ntpdate。他们并非由一个安装包提供。
- ntp守护进程为ntpd，配置文件是/etc/ntp.conf
- ntpdate用于客户端的时间矫正，非NTP服务器可以不启动NTP。

我们先讲一下时间服务器的搭建：

首先先说一下我们常用的Windows系统的时间是怎样的，我们通过控制面板打开日期和时间，然后选择Internet时间，点击更改设置就可以知道当前的服务器是怎样与时间同步的了。
这里要求时间服务器要能上网（为了保证精度），它从外部同步时间，最后给内部主机提供同步。

先讲几条基础命令

| 功能 | 命令 |
| :--: | :-- |
| 查看时间 | date |
| 查看硬件时间 | hwclock -r |
| 查看系统所在时区 | date -R |
| 查看所有时区 | ls /usr/share/zoneinfo/ 或者是：timedatectl list-timezones |
| 查看其它时区的当前时间 | zdump Hongkong |
| 修改系统时间 | date -s “20190408 17:41:00” |
| 修改时区① | tzselect（之后按数字进行选择） vim .bash_profile TZ=‘Asia/hanghai’; export TZ（粘贴在末尾）source ~/.bash_profile |
| 修改时区② | timedatectl set-timezone Europe/Lisbon |
| 保存时间修改 | clock -w |

### NTP服务的命令、配置

#### 1、有关ntp的命令

| 其他命令 | 功能 |
| :--: | :-- |
| ntpq -p |	查询网络中的NTP服务器，同时显示客户端和每个服务器的连接状态 |
| ntpdate -q 192.168.0.35 |	查看上层服务器状态 |
| ntpdate 192.168.0.35 | 更新时间 |
| watch ntpdate 192.168.0.35 | 这条命令为客户端使用，每2秒会发生一次变化 |

#### 2、有关ntpq -p的注解

| remote | 远程主机的主机名或IP |
| :--: | :-- |
| * | 目前正在使用的上层NTP |
| + | 已连线,可提供时间更新的候补服务器 |
| - | 远程服务器被clustering algorithm认为是不合格的NTP Server |
| x | 远程服务器不可用 |

| 其它 | 注解 |
| :--: | :-- |
| refid | 上级NTP的时间基准服务器 |
| st | 就是stratum 上层NTP的层级，层级0-15 |
| when | 几秒钟前曾做过时间同步更新 |
| poll | 下一次更新在几秒后，逐步增大 |
| reach | 八进制数，已经向上层服务器要求更新的次数 |
| delay | 网络传输过程中的延迟时间 |
| offset | 本地和服务器之间的时间差别，越接近0，说明和服务器的时间越接近 |
| jitter linux | 系统时间与bios硬件时钟之间的差异 |

#### 3、/etc/ntp.conf ntp的配置文件介绍

```bash
#系统时间和硬件时间的偏差记录
driftfile /var/lib/ntp/drift
#允许所有的访问，
restrict default nomodify notrap nopeer noquery`
#允许 192.168.221.0网段访问
restrict [192.168.221.0] mask [255.255.255.0] [parameter]
```

| parameter的参数 | 注解 |
| :-----: | :--------- |
| ignore | 拒绝所有类型的ntp连接 |
| nomodify | 客户端不能使用ntpc与ntpq两支程式来修改服务器的时间参数 |
| noquery | 客户端不能使用ntpq、ntpc等指令来查询服务器时间，等于不提供ntp的网络校时 |
| notrap | 不提供trap这个远程时间登录的功能 |
| notrust | 拒绝没有认证的客户端 |
| nopeer | 不与其他同一层的ntp服务器进行时间同步 |

### 搭建NTP服务

关闭防火墙
```bash
systemctl stop firewalld.service
```
安装ntp包
```bash
yum install ntp -y
```
编辑配置文件
```bash
vim /etc/ntp.conf
```
可以将中间的几行默认上层时间的服务器注释掉，也可以不注释掉。注释掉之后我们这台时间服务器就没法向外部服务器同步了，我是为了方便演示，后面加上图中的指令：表示使用本地时间作为ntp服务器提供给ntp客户端。

```bash
restrict 192.168.0.0 mask 255.255.255.0 modify notrap
#为192.168.0.0网段提供授时服务
server 127.127.1.0  prefer
#prefer代表这台主机优先级最高
fudge 1127.127.1.0 stratum 8
#指定服务器为本地，设置层级为8
systemctl restart ntpd
#重启服务。
```

我们可以使用 `netstat -nlutp | grep ntp` 可以查看与ntp服务有关联的进程。

如果客户机上安装有ntp，在客户端需要修改`/etc/ntp.conf`，添加以下内容

```bash
server 192.168.0.1    #指名上层NTP服务器
restrict 192.168.0.1       #放行156.0.26.6
```

修改保存后使用`ntpq -p`命令查看

```bash
[root@centos]# ntpq -p
   remote       refid      st t  when  poll reach  delay  offset  jitter
 ========================================================================
   192.168.0.1    LOCAL(0)  3 u   93   1   377   0.676   -0.563   0.545
```

未安装ntp可使用ntpdate定时同步
设置计划任务，编辑配置文件之后，让它实现一开机就自动同步。

```bash
vim /etc/crontab
#插入下面这句话：
*/30 * * * * root /usr/sbin/ntpdate 192.168.0.1 > /dev/null 2>&1
#每30秒同步一次
```
linux shell中`"2>&1"`含义

| 命令 | 注解 |
| :--: | :--- |
| > | 重定向 |
| /dev/null | 代表空设备 |
| 2	stderr | 标准错误 |
| & | 表示等同于 |
| 2>1&1 | 将标准错误重定向到标准输出 |

## windows搭建NTP时钟服务器

### 配置 NTP

#### 修改注册表项

在搜索框中打开注册表，使用命令： `regedit`，进入注册表项`HKEY_LOCAL_MACHINE—>SYSTEM—>CurrentControlSet—>Services—>W32Time—>TimeProviders—>NtpServer`

把`Enabled` 值设置为 1打开NTP，（系统默认0）。

进入注册表项`HKEY_LOCAL_MACHINE—>SYSTEM—>CurrentControlSet—>Services—>W32Time—>Config`

把`AnnounceFlags`值设置为 5 （系统默认 10）。该设定强制主机将它自身宣布为可靠的时间源，从而使用内置的互补金属氧化物半导体(CMOS) 时钟。

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpClient`的「enable」设定为0 以防止作为客户端自动同步外界的时间服务。



#### 启动时间服务

找到Windows time服务，使用命令：`services.msc`

将Windows time服务设置为自动启动，点击应用和确定，重新启动电脑或者主机，所有配置生效。

NTP时钟搭建完毕，此时只需其他设备将本电脑IP填入时钟同步，点击更新即可同步改时钟

打开客户机控制面板，日期和时间，修改 Internet 时间中地址。

#### 域控中配置组策略

运行输入`gpedit.msc`，计算机配置-->管理模板-->系统-->Windows时间服务-->时间提供程序-->右单击“配置Window NTP客户端”，选择属性。
- 选择“已启用”
- 在Ntp Server对应栏位输入时间同步服务器的地址。
- Tpye栏位选择NTP。
- SpecialPollInterval栏位输入需要同步的时间周期，单位：秒，如：每10分钟同步一次，输入600。

计算机配置-->管理模板-->系统-->Windows时间服务-->时间提供程序-->右单击“启用Window NTP客户端”，选择“已启用”

计算机配置-->管理模板-->系统-->Windows时间服务-->时间提供程序-->右单击“启用Window NTP服务端”，选择“已禁用”

时间和日期属性中，填入时间同步服务器地址，方便必要时进行手动同步。

#### 时间同步服务间隔时间太长

由于Windows时间同步服务距上次同步时间较长，造成时间显示不正常。

解决方法：打开注册表编辑器（在运行对话框输入“regedit”），定位到`HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\W32Time\TimeProviders\NtpClient`然后找到为`SpecialPolllnterval`的键，将键值的基数改为“十进制”接着把键值数据改为“1800”(30分钟)默认是“604800”(7天),（根据自己的需求填入，记住单位是秒），按F5刷新一下，就可以了。

#### 时间同步地址注册表快速更改

```bat
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\DateTime\Servers]
@="0"
"0"="ntp1.aliyun.com"
"1"="ntp2.aliyun.com"
"2"="time.windows.com"
"3"="time.nist.gov"

[HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\W32Time\TimeProviders\NtpClient]
'SpecialPolllnterval'="1800"
```

另存为NTP.reg文件,双击导入