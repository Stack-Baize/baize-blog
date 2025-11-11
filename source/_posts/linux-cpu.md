---
title: Linux系统安装之后，如何调节CPU性能策略
date: '2020-04-25 11:00'
summary: >-
  CPU动态节能技术用于降低服务器功耗，通过选择系统空闲状态不同的电源管理策
  略，可以实现不同程度降低服务器功耗，更低的功耗策略意味着CPU唤醒更慢对性能影响更大。
categories: 操作系统
tags:
  - Linux
  - CPU
  - CentOS
abbrlink: efc
---

## 前言

- 1、ondemand：系统默认的超频模式，按需调节，内核提供的功能，不是很强大，但有效实现了动态频率调节，平时以低速方式运行，当系统负载提高时候自动提高频率。以这种模式运行不会因为降频造成性能降低，同时也能节约电能和降低温度。一般官方内核，还有CM7的默认的方式都是ondemand。
流畅度： 一般，流畅

- 2、interactive：交互模式，直接上最高频率，然后看CPU负荷慢慢降低，比较耗电。
流畅度： 最高，极流畅
Interactive 是以 CPU 排程数量而调整频率，从而实现省电。
InteractiveX 是以 CPU 负载来调整 CPU 频率，不会过度把频率调低。所以比 Interactive 反应好些，但是省电的效果一般

- 3、conservative：保守模式，类似于ondemand，但调整相对较缓，想省电就用他吧。Google官方内核，kang内核默认模式。
流畅度： 高，流畅

- 4、smartass：聪明模式，是I和C模式的升级，该模式在比i模式不差的响应的前提下会做到了更加省电
流畅度： 最高，流畅

- 5、performance：性能模式！只有最高频率，从来不考虑消耗的电量，性能没得说，但是耗电量…
流畅度：还需要说么？还有比这种模式更流畅的吗？

- 6、powersave 省电模式，通常以最低频率运行，打不死我也不用。
流畅度： 极低

- 7、userspace：用户自定义模式，系统将变频策略的决策权交给了用户态应用程序，并提供了相应的接口供用户态应用程序调节CPU 运行频率使用。也就是长期以来都在用的那个模式。可以通过手动编辑配置文件进行配置
流畅度：根据设置而定

- 8、Hotplug：类似于ondemand, 但是cpu会在关屏下尝试关掉一个cpu，并且带有deep sleep，比较省电。
流畅度：一般，流畅

## Linux下设置相关参数

### 设置performance模式

#### CentOS7下配置

首先，需要知道Linux有一个叫做cpupower的工具集，用来检查和调整处理器的能耗相关的一些features。其中的一个工具叫做“frequency-set”，可以用来调整cpu运行频率。

使用下面的命令来查看当下可用的drivers，即governors:

```bash
cpupower frequency-info --governors

# cpupower -c all frequency-info --governors
analyzing CPU 0:
   available cpufreq governors: performance powersave

analyzing CPU 1:
   available cpufreq governors: performance powersave
```

光手动的用`cpupower –c all frequency-set –g performance` 来修改是不够的，我们需要让这个配置在开机的时候就生效。所以，需要创建一个由systemd管理的服务，让这个服务在开机的时候就自动运行。

运行下面的命令，直接修改`/etc/systemd/system/cpupower.service`这个文件，使得这个服务开机就运行一次（oneshot）, 不始终保持运行。

```bash
$ cat << EOF | sudo tee /etc/systemd/system/cpupower.service
[Unit]
Description=CPU powersave

[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower -c all frequency-set -g powersave

[Install]
WantedBy=multi-user.target
EOF

```

问题解决。

#### 其它办法

方法一：在bios(cpu 选项，或者电源管理选项)直接配置为max performance（我的系统无法设置），重启即可；

方法二：

```bash
yum install cpupowerutils
cpupower -c all frequency-set -g performance  #（不用安装，自带cpupower 命令）
#或者
cpupower frequency-set -g performance
```

方法三：

```bash
service cpuspeed stop
```

这里按需重启系统，最好试一下重启能不能生效，有的服务器会在重启之后失效，必须在bios里面设置

### 查看当前governor

```bash
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
#powersave或者performance
cat /proc/cpuinfo | grep -i "cpu mhz"
#显示每个CPU的当前运行频率
cpupower frequency-info
```
