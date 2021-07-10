+++
title = "Mysqldump"
description = ""
tags = ['mysql']
date =  "2021-04-11T16:38:46+08:00"
lastmod = "2021-04-11T16:38:46+08:00"
+++

mysqldump 是 MySQL 自带的逻辑备份工具。
<!--more-->

它的备份原理是通过协议连接到 MySQL 数据库，将需要备份的数据查询出来，将查询出的数据转换成对应的 `insert` 语句，当我们需要还原这些数据时，只要执行这些 `insert` 语句，即可将对应的数据还原。

## 备份命令格式
```bash
mysqldump [选项] 数据库名 [表名] > 脚本名.sql

# 或者, 在powershell终端中必须用 --result-file
mysqldump [选项] 数据库名 [表名] --result-file=脚本名.sql

# 或者, --databases 后面所有名字参量都被看作数据库名
mysqldump [选项] --databases 数据库名1 数据库名2 ...  > 脚本名.sql

# 或者, --tables 覆盖 --databases
mysqldump [选项] --databases 数据库名1 --tables 表名1 > 脚本名.sql

# 或者, 备份所有数据库
mysqldump [选项] --all-databases [选项] > 脚本名.sql
```
>注意: 在win10的powershell终端中使用mysqldump备份会出现中文乱码, 原因是mysqldump不允许将UTF-16作为连接字符集 '`>`', 在powershell终端中的 '`>`' 是属于UTF-16, 可以使用`--result-file`解决, 该选项以ASCII格式创建输出.

## 实例
```bash
# 备份test1数据库
mysqldump -u qlel -p test1 --result-file=test1.sql

# 加上-d只备份表结构, 不备份数据
mysqldump -u qlel -p -d test1 --result-file=test1.sql

# 如果设置了gtid主从复制, 加上--set-gtid-purged=OFF备份binlog
mysqldump --set-gtid-purged=OFF -u qlel -p test1 --result-file=test1.sql
```