+++
title = "Crontab"
description = ""
tags = ['linux', 'shell']
date =  "2021-04-08T15:09:38+08:00"
lastmod = "2021-04-08T15:09:38+08:00"
+++

Linux定时任务。crontab提交和管理用户的需要周期性执行的任务
<!--more-->

Linux下的任务调度分为两类： **系统任务调度** 和 **用户任务调度** 。

**系统任务调度**： 系统周期性所要执行的工作，比如写缓存数据到硬盘、日志清理等。`/etc/crontab`文件是系统任务调度的配置文件。

**用户任务调度**： 用户定期要执行的工作，比如用户数据备份、定时邮件提醒等。用户可以使用 crontab 工具来定制自己的计划任务。所有用户定义的crontab文件都被保存在`/var/spool/cron/crontabs`目录中，其文件名与用户名一致。

使用者权限文件如下：
```bash
/etc/cron.deny     该文件中所列用户不允许使用crontab命令
/etc/cron.allow    该文件中所列用户允许使用crontab命令
/var/spool/cron/   所有用户crontab文件存放的目录,以用户名命名
```
查看 crontab 的状态：
```bash
$ systemctl status cron.service
● cron.service - Regular background program processing daemon
   Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2021-03-20 22:27:35 CST; 6 days ago
     Docs: man:cron(8)
 Main PID: 382 (cron)
    Tasks: 1 (limit: 1147)
   Memory: 2.1M
   CGroup: /system.slice/cron.service
           └─382 /usr/sbin/cron -f
```

## 参数
```bash
-e : 编辑用户的crontab定时任务
-l : 列出用户的crontab定时任务
-r : 删除用户的crontab定时任务
-u user : 指定要设定crontab任务的用户名称。
```

## 内容格式
crontab配置5个`*`代表5个部分，分别代表分钟，小时，天，月，周，如下:
```bash
 ┌───────────── 分钟(0 - 59) 一小时当中的第几分钟
 │ ┌───────────── 小时(0 - 23) 一天当中的第几小时
 │ │ ┌───────────── 天 (1 - 31) 一个月当中的第几天
 │ │ │ ┌───────────── 月 (1 - 12) 一年当中的第几个月
 │ │ │ │ ┌───────────── 周几 (0 - 6) (周日到周一，有的系统里面7表示周日)
 │ │ │ │ │               一周当中的星期几                   
 │ │ │ │ │
 │ │ │ │ │
 * * * * *  command
```
- `*`: 表示任意值。如第一个`*`就表示一小时中的每分钟都执行一次
- `,`：表示多个值。如：`0 8,12,16 * * * 命令`，表示在每天的`08:00,12:00,16:00`都执行一次命令
- `-`：表示连续的时间范围。如：`0 5 * * 1-6 命令`，表示在周一到周六的凌晨05:00执行命令
- `*/n`：表示每隔多久执行一次。如：`*/10 * * * *` 命令，表示每隔10分钟就执行一次命令
- `command`：表示要执行的命令或脚本。

## 示例
执行`crontab -e`会使用默认编辑器打开`/var/spool/cron/crontabs/<用户名>`的文件，可以在里面编写定时任务，如：
```bash
*/1 * * * * date -Iseconds >> ~/test_cron.txt
```
表示每分钟执行一次命令。