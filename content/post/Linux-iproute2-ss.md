+++
title = "Linux Iproute2 Ss"
description = ""
tags = ['linux', 'shell', 'iproute2']
date =  "2021-04-08T17:03:27+08:00"
lastmod = "2021-04-08T17:03:27+08:00"
+++

ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。
<!--more-->

## 帮助信息
如果不使用任何选项，ss将显示已建立连接的打开的非监听套接字（例如TCP/UNIX/UDP）的列表。

- `-h, --help`
显示帮助信息.
- `-V, --version`
显示版本信息.
- `-H, --no-header`
不显示标题头
- `-n, --numeric`
不要尝试解析服务名称
- `-r, --resolve`
尝试解析数字地址/端口
- `-a, --all`
显示侦听和非侦听（对于TCP，这意味着已建立的连接）套接字。
- `-l, --listening`
仅显示侦听套接字（默认情况下将省略这些套接字）。
- `-o, --options`
显示计时器信息
- `-e, --extended`
显示详细的套接字信息
- `-m, --memory`
显示套接字内存使用情况
- `-p, --processes`
显示使用套接字的进程
- `-i, --info`
显示内部TCP信息
- `-K, --kill`
尝试强行关闭套接字。 此选项显示已成功关闭的套接字，并以静默方式跳过内核不支持关闭的套接字。 它仅支持IPv4和IPv6套接字。
- `-s, --summary`
打印摘要统计信息。此选项不解析从各种源获取摘要的套接字列表。当套接字的数量太大以至于解析/proc/net/tcp非常痛苦时，它非常有用。
- `-Z, --context`
显示进程安全上下文
- `-z, --contexts`
显示套接字上下文
- `-N NSNAME, --net=NSNAME`
切换到指定的网络命名空间名称
- `-b, --bpf`
显示套接字BPF筛选器（仅允许管理员获取这些信息）
- `-4, --ipv4`
只显示ipv4的套接字, - `-f inet`的别名
- `-6, --ipv6`
只显示ipv6的套接字, - `-f inet6`的别名
- `-0, --packet`
只显示PACKET的套接字, - `-f link`的别名
- `-t, --tcp`
只显示TCP套接字
- `-u, --udp`
只显示UDP套接字
- `-d, --dccp`
只显示DCCP套接字
- `-w, --raw`
只显示RAW套接字
- `-x, --unix`
显示Unix域套接字, - `-f unix`的别名
- `-S, --sctp`
只显示SCTP套接字
- `-f FAMILY, --family=FAMILY`
显示指定FAMILY类型的套接字, 支持unix, inet, inet6, link, netlink
- `-A QUERY, --query=QUERY, --socket=QUERY`
要转储的套接字表的列表，用逗号分隔。支持all, inet, tcp, udp, raw, unix, packet, netlink, unix_dgram, unix_stream, unix_seqpacket, packet_raw, packet_dgram
- `-D FILE, --diag=FILE`
不显示任何内容，仅在应用过滤器后将有关TCP套接字的原始信息转储到FILE中。 如果FILE是- `-`使用`stdout`。
- `-F FILE, --filter=FILE`
从FILE读取过滤器信息。 FILE的每一行都被解释为单个命令行选项。 如果FILE是- `-`使用`stdin`。

## 示例
```bash
# 显示所有TCP套接字
$ ss -t -a

# 显示所有UDP套接字
$ ss -u -a

# 查看已侦听原始地址端口的套接字, 可用于查看端口
$ ss -ln
```
