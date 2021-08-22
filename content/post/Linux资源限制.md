+++
title = "Linux资源限制"
date = 2021-08-22T17:43:06+08:00
description = ""
tags = ['linux','ulimit']
aplayer = false
showToc = true
math = true
showLicense = false
+++

了解Linux资源限制对系统配置和优化有帮助.
<!--more-->

## Linux资源限制
- 永久修改: 
    - `/etc/security/limits.conf`
    - `/etc/security/limits.d/*`
    - systemd对单个应用的限制
- 临时修改: `ulimit`命令

### limits.conf
`/etc/security/limits.conf` 文件实际是 Linux PAM（插入式认证模块，Pluggable Authentication Modules）中 `pam_limits.so` 的配置文件。

`/etc/security/limits.conf` 配置解析:
```ini
# /etc/security/limits.conf
#
#此文件为通过 PAM 登录的用户设置资源限制。
#它不会影响系统服务的资源限制。
#
#请注意, /etc/security/limits.d下按照字母顺序排列的配置文件会覆盖 /etc/security/limits.conf中的domain相同的的配置
#
#每行描述一个用户的限制，格式如下：
#
#<domain>        <type>  <item>  <value>
#
#<domain> 可以是:
#        - 用户名
#        - 用户组, 语法：@group 
#        - 通配符 * ，表示所有用户
#        - 通配符 % , 也可用于 %group ，最大登录数限制 maxlogin limit
#        - NOTE: 组和通配符限制不适用于 root
#          要限制 root ，<domain> 必须明确指定为 root
#
#<type> :
#        - "soft" 强制软限制。也可以理解为警告值
#        - "hard" 强制硬限制。系统中所能设定的最大值。soft 的限制不能比 hard 限制高
#        - "-"    同时设置了soft和hard的值
#
#<item> :
#        - core - 限制内核文件的大小 (KB)
#        - data - 最大数据大小 (KB)
#        - fsize - 最大文件大小 (KB)
#        - memlock - 最大锁定内存地址空间 (KB)
#        - nofile - 最大打开的文件数(以文件描叙符，file descripter计数) 
#        - rss - 最大驻留设置大小 (KB)
#        - stack - 最大栈大小 (KB)
#        - cpu - 最多cpu占用时间 (MIN)
#        - nproc - 进程的最大数目
#        - as - 地址空间限制  (KB)
#        - maxlogins - 此用户允许登录的最大数目
#        - maxsyslogins - 系统最大同时在线用户数
#        - priority - 运行用户进程的优先级
#        - locks -  用户可以持有的文件锁的最大数量
#        - sigpending - 最大挂起信号数
#        - msgqueue - POSIX消息队列使用的最大内存 (bytes)
#        - nice - 允许提升到值的最大nice优先级: [-20, 19]
#        - rtprio - 最大实时优先级
#        - chroot - 改变家目录 (Debian特有)
#
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#root            hard    core            100000
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#ftp             -       chroot          /ftp
#@student        -       maxlogins       4

# End of file

```
>注意不能设置 `nofile`不能设置 `unlimited`，`noproc`可以.

### ulimit
`ulimit`是bash内键命令，它具有一套参数集，用于为由它生成的shell进程及其子进程的资源使用设置限制。

```bash
ulimit: ulimit [-SHabcdefiklmnpqrstuvxPT] [limit]
    修改shell资源限制。

    选项:
      -S        软限制 soft
      -H        硬限制 hard
      -a        打印当前所有限制
      -b        套接字缓冲区大小
      -c        创建核心文件的最大大小
      -d        进程数据段的最大大小
      -e        最大调度优先级（nice）
      -f        shell 及其子进程写入的文件的最大大小
      -i        最大挂起信号数
      -k        为该进程分配的最大 kqueue 数
      -l        进程可以锁定到内存中的最大大小
      -m        最大驻留集大小
      -n        打开的文件描述符的最大数量
      -p        管道缓冲区大小
      -q        POSIX 消息队列中的最大字节数
      -r        最大实时调度优先级
      -s        最大堆栈大小
      -t        以秒为单位的最大 CPU 时间
      -u        最大用户进程数
      -v        虚拟内存的大小
      -x        最大文件锁数
      -P        最大伪终端数
      -T        最大线程数

    并非所有选项都适用于所有平台。

    如果给定了 LIMIT，则为指定资源的新值； 
    特殊的 LIMIT 值“soft”、“hard”和“unlimited”分别代表当前软限制、当前硬限制和无限制。
    否则，打印指定资源的当前值。 如果没有给出选项，则假定为 -f。

    值以 1024 字节为增量，但 -t（以秒为单位）、-p（以 512 字节为增量）和 -u（未缩放的进程数）除外。

```
