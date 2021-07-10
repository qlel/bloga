+++
title = "MySQL基于GTID主从复制"
description = ""
tags = ['mysql']
date =  "2021-04-11T16:33:52+08:00"
lastmod = "2021-04-11T16:33:52+08:00"
+++
为了解决MySQL的单点故障以及提高MySQL的整体服务性能，一般都会采用「主从复制」。
<!--more-->

## 主从复制的简介与原理
主从复制中分为主服务器（master）和从服务器（slave），主服务器负责写，而从服务器负责读，Mysql的主从复制的过程是一个「异步的过程」。

主从复制原理:
![主从复制原理.png](主从复制原理.png)
1. 当Master节点进行insert、update、delete操作时，会按顺序写入到binlog中。
2. salve从库连接master主库，Master有多少个slave就会创建多少个binlog dump线程。
3. 当Master节点的binlog发生变化时，binlog dump 线程会通知所有的salve节点，并将相应的binlog内容推送给slave节点。
4. I/O线程接收到 binlog 内容后，将内容写入到本地的 relay-log。
5. SQL线程读取I/O线程写入的relay-log，并且根据 relay-log 的内容对从数据库做对应的操作。

## GTID 简介
从 MySQL 5.6.5 版本新增了一种主从复制方式：GTID，其全称是`Global Transaction Identifier`，即全局事务标识。通过GTID保证每个主库提交的事务在集群中都有唯一的一个事务ID。强化了数据库主从的一致性和故障恢复数据的容错能力。在主库宕机发生主从切换的情况下。GTID方式可以让其他从库自动找到新主库复制的位置，而且GTID可以忽略已经执行过的事务，减少了数据发生错误的概率。

## GTID 组成
`GTID`是对一个已经提交事务的编号，并且是全局唯一的。`GTID`是由`UUID`和`TID`组成的。`UUID`是MySQL实例的唯一标识，`TID`代表该实例上已经提交的事务数量，随着事务提交数量递增。

举个例子：`3E11FA47-71CA-11E1-9E33-C80AA9429562:23`，冒号前面是`UUID`，后面是`TID`。

## GTID 工作原理
1. 主库 master 提交一个事务时会产生 GTID，并且记录在 binlog 日志中
2. 从库 salve I/O 线程读取 master 的 binlog 日志文件，并存储在 slave 的 relay log 中。slave 将 master 的 GTID 这个值，设置到 gtid_next 中，即下一个要读取的 GTID 值。
3. slave 读取这个 gtid_next，然后对比 slave 自己的 binlog 日志中是否有这个 GTID
4. 如果有这个记录，说明这个 GTID 的事务已经执行过了，可以忽略掉
5. 如果没有这个记录，slave 就会执行该 GTID 事务，并记录到 slave 自己的 binlog 日志中。在读取执行事务前会先检查其他 session 持有该 GTID，确保不被重复执行。
6. 在解析过程中会判断是否有主键，如果没有就用二级索引，如果没有就用全部扫描。

## GTID使用中的限制条件
GTID复制是针对事务来说的，一个事务只对应一个GTID，好多的限制就在于此。
1. 不能使用create table table_name select * from table_name。
2. 在一个事务中既包含事务表的操作又包含非事务表。
3. 不支持CREATE TEMPORARY TABLE or DROP TEMPORARY TABLE语句操作。
4. 使用GTID复制从库跳过错误时，不支持执行该sql_slave_skip_counter参数的语法。

## GTID实现主从复制
### 环境
两个操作系统: win10主机, Debian虚拟机.
- win10的ip地址为: `192.168.1.101`
- Debian虚拟机的ip地址为: `10.0.2.15`
- 虚拟机网关ip地址: `192.168.56.1`
- win10设置的端口转发: `10.0.2.15:3306` -> `192.168.56.1:3333`

MySQL8数据库地址:
- 主库: `192.168.1.101:3306`
- 从库: `10.0.2.15:3306` 

win10主机访问远程虚拟机MySQL的`qlel`用户:
```bash
mysql -h 192.168.56.1 -P 3333 -u qlel -p
```
Debian虚拟机访问远程win10主机MySQL的`main`用户:
```bash
mysql -h 192.168.56.1 -P 3306 -u qlel -p
```
两主机皆克利用网关来正常访问.

### master主库配置
win10安装目录有个配置文件`my.ini`, 在内容`[mysqld]`下添加配置, 配置完需要重启.
```ini
[mysqld]
# 主从复制设置
# gtid
# 服务器id, 唯一值
server_id=111
# 开启gtid模式
gtid_mode=on
# 强制gtid一致性，开启后对于特定的create table不被支持
enforce_gtid_consistency=on

# 指定用于二进制日志文件的基本名称, 默认binlog
log_bin=binlog
# 使从库也保存binlog日志, 默认on
log_slave_updates=on
# binlog格式, 默认值ROW
binlog_format=ROW
# 设置二进制日志的有效期限（以秒为单位）, 默认2592000, 为30天
binlog_expire_logs_seconds=2592000
```
重启, 以管理员身份打开powershell:
```powershell
> Get-Service -Name '*mysql*'

Status   Name               DisplayName
------   ----               -----------
Running  MySQL              MySQL

> Stop-Service -Name 'MySQL'
WARNING: Waiting for service 'MySQL (MySQL)' to stop...

> Get-Service -Name '*mysql*'

Status   Name               DisplayName
------   ----               -----------
Stopped  MySQL              MySQL

> Start-Service -Name 'MySQL'
WARNING: Waiting for service 'MySQL (MySQL)' to start...
WARNING: Waiting for service 'MySQL (MySQL)' to start...
WARNING: Waiting for service 'MySQL (MySQL)' to start...
WARNING: Waiting for service 'MySQL (MySQL)' to start...

> Get-Service -Name '*mysql*'

Status   Name               DisplayName
------   ----               -----------
Running  MySQL              MySQL
```

### slave从库配置
Debian安装MySQL后, 配置文件为`/etc/mysql/my.cnf`, 在内容`[mysqld]`下添加配置, 配置完需要重启.
```ini
[mysqld]
# 服务器id, 唯一值
server_id=222
# 开启gtid模式
gtid_mode=on
# 强制gtid一致性，开启后对于特定的create table不被支持
enforce_gtid_consistency=on
# 指定用于二进制日志文件的基本名称, 默认binlog
log_bin=binlog
# 使从库也保存binlog日志, 默认on
log_slave_updates=on
# binlog格式, 默认值ROW
binlog_format=ROW
# 设置二进制日志的有效期限（以秒为单位）, 默认2592000, 为30天
binlog_expire_logs_seconds=2592000

# 过滤器, 可以都不设置, 复制所有数据库和表的操作
# 只复制数据库test1下的所有表, 和replicate-wild-ignore-table互斥
replicate-wild-do-table=test1.%

# 忽略以下数据库的表, 和replicate-wild-do-table互斥, 可以不设置
replicate-wild-ignore-table=mysql.%
replicate-wild-ignore-table=sys.%
replicate-wild-ignore-table=information_schema.%
replicate-wild-ignore-table=performance_schema.%
```
配置完成后重启:
```bash
# 重启
$ sudo systemctl restart mysql.service

# 查看状态
$ systemctl status mysql.service
```
### 检查GTID是否开启
```sql
mysql> show variables like '%gtid%';
+----------------------------------+-------------------------------------------+
| Variable_name                    | Value                                     |
+----------------------------------+-------------------------------------------+
| binlog_gtid_simple_recovery      | ON                                        |
| enforce_gtid_consistency         | ON                                        |
| gtid_executed                    | ed3b6e48-9b04-11e9-8c96-f0761c2c3242:1-20 |
| gtid_executed_compression_period | 1000                                      |
| gtid_mode                        | ON                                        |
| gtid_next                        | AUTOMATIC                                 |
| gtid_owned                       |                                           |
| gtid_purged                      |                                           |
| session_track_gtids              | OFF                                       |
+----------------------------------+-------------------------------------------+
```
### 保持数据一致
备份主库需要复制的数据, 导入到从库, 必须保持数据一致.
```bash
# 设置了gtid, 必须加上--set-gtid-purged=OFF备份binlog
mysqldump --set-gtid-purged=OFF -u qlel -p test1 --result-file=test1.sql

# 传输到远程虚拟机
scp -P 2222 .\test1.sql qlel@192.168.56.1:~
```
导入数据到从库中:
```bash
mysql -u qlel -p test1 < ~/test1.sql
```

### 主库建立授权用户
```sql
-- 创建用户
create user 'main'@'%' identified by 'main_1995';

-- 授权, 使从库能够从主库读取binlog日志
grant replication slave on *.* to 'main'@'%';

-- 刷新权限表
flush privileges;
```
也可以使用现有被授予所有权限的用户.

主从复制搭建完成后, 主库任何用户的操作修改都会异步复制到从库当中.

### salve连接到master
```sql
change master to 
master_host='192.168.56.1', 
master_user='main',
master_password='main_1995', 
master_port=3306, 
master_auto_position=1;
```
`master_auto_position`: 
- `1` 代表采用GTID协议复制; 
- `0` 代表采用老的binlog复制.

### 启动从库线程
在MySQL 8.0.22版本之前的版本使用:
```sql
start slave;
```
在MySQL 8.0.22版本之后的版本使用:
```sql
start replica;
```
### 查看从库状态
在MySQL 8.0.22版本之前的版本使用:
```sql
show slave status\G;
```
在MySQL 8.0.22版本之后的版本使用:
```sql
show replica status\G;
```
几个需要注意的字段:
- 含有以下信息表示GTID模式主从复制搭建成功.
```bash
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Master_UUID: ed3b6e48-9b04-11e9-8c96-f0761c2c3242
Executed_Gtid_Set: ad1b6c0f-8b1c-11eb-8f10-0800279be4fb:1-116
Auto_Position: 1
```
- 如果`Slave_SQL_Running: No`查看最后出现的错误, 一般都是数据不一致导致, 这时从主库备份一下导入到从库即可.
```bash
Slave_IO_Running: Yes
Slave_SQL_Running: No
Last_Error: ... Can't find record in 't2', Error_code: 1032; ...
```
### 在master上查看salve信息
在MySQL 8.0.22版本之前的版本使用:
```sql
show slave hosts;
```
在MySQL 8.0.22版本之后的版本建议使用别名:
```sql
show replicas;
```
### 从库设置只读
从库设置只读模式, 防止数据出现不一致, 以及实现读写分离, 主库进行写操作, 从库只进行读操作, 提高性能.
```sql
-- 锁定所有表为只读模式, 设置全局只读
mysql> FLUSH TABLES WITH READ LOCK;
mysql> SET GLOBAL read_only = ON;

-- 关闭全局只读模式, 解锁表的只读模式
mysql> SET GLOBAL read_only = OFF;
mysql> UNLOCK TABLES;
```

## 取消GTID主从复制
部署环境有时需要更换取消主从机制或者更换备机，需要将之前的主备关系解除.

### 从库流程
停止从库进程:
```sql
mysql>stop slave;

-- MySQL 8.0.22版本之后使用replica
mysql>stop replica;
```
清除所有元信息和中继日志:
```sql
mysql>reset slave;

-- MySQL 8.0.22版本之后使用replica
mysql>reset replica;

# 可以通过以下命令查看当前状态
mysql> show slave status\G

Emptyset (0,00 sec)
```
之后从库可以直接关闭下线.

如果之后不设置主从了, 可以修改配置文件后重启数据库.

### 主库流程
```sql
mysql> reset master;
```
次命令会删除所有现有的二进制日志文件并重置二进制日志索引文件，将服务器重置为二进制日志记录启动前的状态。将创建一个新的空二进制日志文件，以便可以重新启动二进制日志记录。

如果想彻底清除主从的机制，可以修改配置文件，删除主从相关的配置项，然后重启mysql即可。

>请谨慎使用此语句，以确保您不会丢失任何想要的二进制日志文件数据和GTID执行历史记录。