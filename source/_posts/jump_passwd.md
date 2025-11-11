---
title: 如何重置jumpserver管理员密码
date: '2020-09-25 11:00'
summary: 之前运维交接的时候没有给到 jumpserver相关信息（悲剧），结果一直显示登录不进去，如何重置jumpserver管理员密码。
categories: 环境部署
tags:
  - Linux
  - jumpserver
abbrlink: 96f0
---

## 0x01 重置管理员密码

```bash
# 管理密码忘记了或者重置管理员密码
$ source /opt/py3/bin/activate
$ cd /opt/jumpserver/apps
$ python manage.py changepassword  admin
#输入新的密码
$ password
```

## 0x02 新建超级管理账号（死而复生）

```bash
# 新建超级用户的命令如下命令
$ python manage.py createsuperuser --username=user --email=user@domain.com
# 设备新用户密码
$ password:
```
