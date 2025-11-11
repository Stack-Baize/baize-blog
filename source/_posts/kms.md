---
title: kms激活命令：一句命令激活Windows/Office
abbrlink: kms
date: 2020-03-27 13:00
summary: 'kms激活是操作系统批量激活授权版本的一种方式，即VL版，一般企业版都是VL版，专业版有零售和VL版，家庭版旗舰版OEM版等等那就肯定不能默认直接用kms激活。'
top: true
categories: 系统配置
tags:
- KMS
- Windows
- Office
---

对于新安装 Windows的用户而言，总是会对自己的激活状态感到惴惴不安，到底激活没？其实很简单，四种命令可以帮助您充分了解自己机器的激活状态

## Windows激活命令

1、命令：`slmgr.vbs -dli`

功能：操作系统版本、部分产品密钥、许可证状态。

2、命令：`slmgr.vbs -dlv`

功能：最为详尽的激活信息，包括：激活ID、安装ID、激活截止日期？ -------显示：显示操作系统版本。

3、命令：`slmgr.vbs -xpr`

功能：是否彻底激活？

4、命令：`winver`

功能：显示操作系统版本。

5､命令：`slmgr.vbs -ipk`

功能：安装产品密钥

6､命令：`slmgr.vbs -ato`

功能：激活 Windows

7､命令：`slmgr.vbs -skms`

功能：设置KMS服务器与端口

8､命令：`slmgr.vbs -ckms`

功能：清除所使用KMS服务器信息

### 激活系统

如果你是有key，只须要使用5､6命令就可以激活系统。

``` bat
slmgr.vbs -ipk xxxx-xxxx-xxxx-xxxx

slmgr.vbs -ato
```

但你是KMS激活一般来说，只要确保的下载的是VL批量版本并且没有手动安装过任何key，

你只需要使用管理员权限运行cmd执行一句命令就足够：

``` bat
slmgr.vbs -skms xxx.xxx.xxx
```

然后去计算机属性或者控制面板其他的什么的地方点一下激活就好了。

当然，如果你懒得点，可以多打一句命令手动激活：

``` bat
slmgr.vbs -ato
```

这句命令的意思是，马上对当前设置的key和服务器地址等进行尝试激活操作。

kms激活的前提是你的系统是批量授权版本，即VL版，一般企业版都是VL版，专业版有零售和VL版，家庭版旗舰版OEM版等等那就肯定不能默认直接用kms激活。一般建议从msdn我告诉你上面下载系统，带有VL字样即可。

VL版本的镜像一般内置GVLK key，用于kms激活。如果你手动输过其他key，那么这个内置的key就会被替换掉，这个时候如果你想用kms，那么就需要把GVLK key输回去。对于Windows，在不太方便找到VL版本的时候，也可以用相同版本的导入GVLK key来代替，比如从微软官网下载win10专业版，然后导入GVLK key来启用kms通道。首先，

在下面的列表获取你对应版本产品用于kms激活的GVLK key。同样地，位于该列表里的产品都可以用kms激活。

另外对于Window7，如果你的bios含有slic表，会有无法使用kms的情况。

- 所有版本GVLK https://muyun.info/posts/ipk.html
- office2016 https://technet.microsoft.com/zh-cn/library/dn385360(v=office.16).aspx
- office2013 https://technet.microsoft.com/ZH-CN/library/dn385360.aspx
- office2010 https://technet.microsoft.com/ZH-CN/library/ee624355(v=office.14).aspx
- Server/Windows https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys

如果不知道自己的系统是什么版本，可以运行以下命令查看系统版本：

``` bat
wmic os get caption
```

得到对应key之后，使用管理员权限运行cmd执行安装key：

``` bat
slmgr /ipk xxxxx-xxxxx-xxxxx-xxxxx
```

然后跟上面说的一样设置kms服务器地址，激活。

## Office激活
首先你的office必须是vol版本，否则无法激活。

找到你的office安装目录，比如C:\Program Files\Microsoft Office\Office16

64位系统安装86位Office的就是C:\Program Files (x86)\Microsoft Office\Office16

- office16是office2016
- office15就是2013
- office14就是2010

然后目录对的话，该目录下面应该有个OSPP.VBS。

接下来我们就cd到这个目录下面，例如（请更改为自己的实际安装目录）：

``` bat
cd "C:\Program Files\Microsoft Office\Office16"
```

然后执行注册kms服务器地址：

``` bat
cscript ospp.vbs /sethst:kms.xxx.xxx
```

/sethst参数就是指定kms服务器地址。

一般ospp.vbs可以拖进去cmd窗口，所以也可以这么弄：

``` bat
cscript "C:\Program Files\Microsoft Office\Office16\OSPP.VBS" /sethst:kms.xxx.xxx
```

“一句命令已经完成了”，但一般office不会马上连接kms服务器进行激活，所以我们额外补充一条手动激活命令：

``` bat
cscript ospp.vbs /act
```

如果提示看到successful的字样，那么就是激活成功了，重新打开office就好。

### ospp.vbs命令介绍

命令有很多，说说几个激活过程中的常用命令。

``` bat
cscript ospp.vbs /dstatus
```

显示当前已安装产品密钥的许可证信息。可以查看到自已安裝的版本有多少个序列号。

``` bat
cscript ospp.vbs /unpkey:xxxxx
```

卸载已安装的产品密钥。后面的数字是密钥的最后5位数。

此时再执行cscript ospp.vbs /dstatus发现产品密钥已经没有了，我重新进行导入。

``` bat
cscript ospp.vbs /inpkey:xxxxx
```

安装、替换现有的产品密钥。和上面的过程刚好相反。

``` bat
cscript ospp.vbs /sethst:x.x.x.x
```

设置KMS主机名。一般为IP地址。

``` bat
cscript ospp.vbs /act
```

激活当前安装的Office。

``` bat
cscript ospp.vbs /remhst
```

删除KMS主机名。

一般来说掌握这么几个就可以了，如果你想要全面了解，去微软官方网站上查找命令帮助说明，慢慢学习研究吧！

### 常见错误
如果遇到报错，请检查：
- 1、你的系统/office是否是批量VL版本
- 2、是否以管理员权限运行cmd
- 3、你的系统/office是否修改过key/未安装GVLK key
- 4、检查你的网络连接
- 5、本地的解析不对,或网络问题（检查服务器是否能连上）
- 6、根据出错代码自己搜索出错原因

*0x80070005错误*一般是你没用管理员权限运行CMD
