---
layout: post
title:  "Docker系列 - 基于Docker部署Mysql主从复制"
date:   2019-04-08 14:40:58 +0800
categories: posts
author: jasydong
---
本文介绍如何基于`Docker`系统部署`Mysql`主从复制。

## 1. 复制原理

Mysql复制是通过`binlog`来实现的, 具体实现流程如下图所示

![Mysql Replication](https://jasydong.github.io/assets/images/mysql/mysql_replication.jpg)

1.1 主库将所有的操作都记录到`binlog`中。当复制开启时，主库的dump线程根据从库io线程的请求将binlog中的内容发送到从库。

1.2 从库的io线程接受到主库dump线程发送的`binlog`事件后，将其写到本地的`relay-log`。

1.3 从库的sql线程重放`relay-log`中的事件。

## 2. 部署步骤

2.1 创建Master和Slave实例
```
docker run --name mysql_master -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mariadb:10.1.38
```

```
docker run --name mysql_slave -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 -d mariadb:10.1.38
```

2.2 Master实例`my.cnf`文件添加如下内容并重启服务
```
[mysqld]

## 同一局域网内注意ID要唯一
server-id=100
## 开启二进制日志功能
log-bin=mysql-bin
```

2.3 Master实例上创建数据同步用户`slave`并授权, 用于在主从库之间同步数据

```
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
```
```
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

2.4 Salve实例`my.cnf`文件添加如下内容并重启服务
```
[mysqld]

## 设置server_id 注意要唯一
server-id=101
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin
## relay_log配置中继日志
relay_log=mysql-relay-bin
```
2.5 链接Master和Slave实例

首先在Master实例终端中执行
```
show master status;
```
执行完后会得到如下结果

![Master Status](https://jasydong.github.io/assets/images/mysql/mysql_master_status.png)

记录下`File`字段和`Position`字段对应的值, 然后在Slave实例终端中执行
```
change master to master_host='172.17.0.3', master_port=3306, master_user='slave', master_password='123456', master_log_file='mysql-bin.000001', master_log_pos=313, master_connect_retry=30;
```
*上面命令说明:*

| 字段名    |  值               |
|----------|-------------------|
| master_host | 指的是Master实例的IP |
| master_port | 指的是Master实例的端口 |
| master_user | 指的是在Master实例上创建的用来数据同步的账号 |
| master_password | 指的是在Master实例上创建的用来数据同步的账号密码 |
| master_log_file | 指的是Slave实例从哪个日志文件开始复制数据，即上面记录的`File`字段的值 |
| master_log_pos | 指的是Slave实例从从哪个 Position 开始读，即上面记录的 `Position`字段的值 |
| master_connect_retry | 指的是如果连接失败，重试的时间间隔，单位是秒，默认是60秒 |

然后在Slave实例终端上执行
```
show slave status \G;
```
得到Slave实例的主从同步状态信息，如下图

![Slave Status](https://jasydong.github.io/assets/images/mysql/mysql_slave_status.png)

正常情况下`Slave_IO_Running`和`Slave_SQL_Running`字段都是No，因为我们还没有开启主从复制，接下来输入下面命令开启主从复制

```
start slave;
```
然后再次在Slave实例终端输入命令

```
show slave status \G;
```
得到Slave实例的主从同步状态信息，如下图
![Slave Status 2](https://jasydong.github.io/assets/images/mysql/mysql_slave_status2.png)

这次`Slave_IO_Running`和`Slave_SQL_Running`都是Yes，说明主从复制已经成功开启
