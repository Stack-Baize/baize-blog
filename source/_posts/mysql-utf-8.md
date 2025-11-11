---
title: 修改MySQL数据库字符编码为utf8mb4解决中文乱码
date: '2020-09-21 11:00'
summary: '在开发时或调过程中，经常会出现mysql数据库中文乱码的情况，修改为UTF-8后将 emoji 文字直接写入 SQL 中，执行 insert 语句报错。现在我们来说说怎么解决这种问题。'
categories: 数据库
tags:
  - MySQL
  - 数据库
abbrlink: 688b
---

### 故障情况

由于MySQL编码原因会导致数据库出现乱码。修改为UTF-8后发现将emoji 文字直接写入 SQL 中，执行 insert 语句报错。

### 解决办法

修改MySQL数据库字符编码为utf8mb4，utf8mb4包含全世界所有国家需要用到的字符,是国际编码。

### 具体操作

#### 1、进入MySQL控制台

```bash
mysql -uroot -p
#输入密码进入
status;
#查看当前MySQL运行状态，
Server characterset: latin1
Db characterset: latin1
Client characterset: utf8mb4
Conn. characterset: utf8mb4
```

默认客户端和服务器端都用了latin1编码，所以会出现乱码。

#### 2、修改mysql配置文件

```bash
vi /etc/my.cnf
#在[client]段增加下面代码
default-character-set=utf8mb4
#在[mysqld]段增加下面的代码
default-storage-engine=INNODB
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
:wq! #保存退出
```

#### 3、service mysqld restart #重启MySQL

再次进入MySQL控制台查看字符编码

```bash
status;
Server characterset: utf8mb4
Db characterset: utf8mb4
Client characterset: utf8mb4
Conn. characterset: utf8mb4
#查看MySQL字符集
show variables like 'character_set_%';
```
MySQL数据库字符集编码修改完成！

`参数说明`
- character_set_client：客户端请求数据的字符集。
- character_set_connection：从客户端接收到数据，然后传输的字符集。
- character_set_database：默认数据库的字符集，无论默认数据库如何改变，都是这个字符集；如果没有默认数据库，使character_set_server指定的字符集，此参数无需设置。
- character_set_filesystem：把操作系统上文件名转化成此字符集，即把character_set_client转换character_set_filesystem，默认binary即可。
- character_set_results：结果集的字符集。
- character_set_server：数据库服务器的默认字符集。
- character_set_system：这个值总是utf8mb4，不需要设置，存储系统元数据的字符集。

`备注：`
MySQL 5.5之前的版本设置办法：
在[client]段下添加

```bash
default-character-set=utf8mb4
```

在[mysqld]段下添加

```bash
default-character-set=utf8mb4
```

注意，如果修改后不能启动报错，把`[mysqld]`段下`default-character-set=utf8`改为`character_set_server=utf8mb4`，取消`[client]`段的设置。

创建数据库的命令：

```bash
Create DATABASE IF NOT EXISTS mydata default charset utf8mb4 COLLATE utf8mb4_general_ci;
```
至此，修改MySQL数据库字符编码为utf8mb4解决中文乱码问题。
