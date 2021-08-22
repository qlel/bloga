+++
title = "Linux ps命令"
date = 2021-08-22T17:48:55+08:00
description = ""
tags = ['linux', 'shell', 'ps']
aplayer = false
showToc = true
math = true
showLicense = false
+++

`ps`命令显示正在运行进程的快照。如果要实时显示正在运行的进程，请使用`top`命令。
<!--more-->

## 命令选项风格
`ps` 命令支持的 3 种语法格式:
- UNIX 风格。选项可以组合在一起，并且选项前必须有 "`-`" 连字符
- BSD  风格。选项可以组合在一起，但是选项前没有 "`-`" 连字符
- GNU  风格的长选项。 选项前有 "`--`" 连字符

这里只介绍和使用 UNIX 风格。

## 选项

### 通用
- `-A`或`-e`: 显示所有进程
- `-a`: 显示同一终端下的所有程序
- `-d`: 显示所有进程，但省略所有的会话引线
- `-N`: 显示所有的程序，除了执行ps指令终端机下的程序之外

### 有列表值
这些选项接受以空格分隔或逗号分隔的列表。它们可以被使用多次。

例如：`ps -p "1 2" -p 3,4`
- `-123`或`-p cmdlist`: 查找pid为123的进程，查找多个pid可以`ps -1619 1583`
- `-C cmdlist`: 指定执行指令的名称，并列出这些指令的程序的状况。
- `-G cmdlist`或`-g cmdlist`: 列出这些群组RGID的程序的状况，也可使用群组名称来指定
- `-s cmdlist`: 查看指定会话id的进程
- `-t cmdlist`: 查看指定终端tty号码的进程，如`ps -t 1,2`
- `-U cmdlist`或`-u cmdlist`: 查看指定用户id或名称的进程

### 输出格式

- `-c`: 显示 `-l` 选项的不同调度的进程信息。多了CLS和PRI栏位
- `-f`或`-f`: 显示所有字段列
- `-j`: 工作模式
- `-l`: 长格式，一般与`-y`一起使用
- `-y`: 不显示F(flag)栏位，并以RSS栏位取代ADDR栏位
- `-M`: 添加一列安全数据。
- `-o format`: 自定义格式。
    - format是空格或逗号分隔的列表字段。
    - 字段名可根据需要重命名，如：`ps -o pid,ruser=RealUser -o comm=Command`
    - 提供了显式的宽度控制，如`ps -o pid,uid:10,cmd:40=命令,time`
- `-O format`: 类似`-o`，但是提供了一些默认字段列

### 输出修饰符
- `-H`: 显示树结构
- `-w`: 宽输出。使用此选项两次可获得无限宽度。

### 线程显示
- `-L`: 显示线程，可能包含LWP和NLWP列。
- `-m`: 在进程之后显示线程。
- `-T`: 显示线程，可能带有SPID列。

## 状态说明
### 进程标志F
`F`字段列说明：
- `1`: fork进程，但没执行
- `4`: 使用超级用户权限

### 进程状态码S
`S`或`STAT`或`STATE`字段列描述各种进程状态：
- `D`: 无法中断的休眠状态(通常 IO 的进程)
- `I`: 空闲内核线程
- `R`: 正在运行或可运行（在运行队列上）
- `S`: 休眠状态(等待事件完成)
- `T`: 通过控制信号停止
- `t`: 追踪debug期间被停止
- `W`: 进入内存交换(自 2.6.xx 内核起无效)
- `X`: 死了的进程(永远不会看到)
- `Z`: 僵尸进程。已终止但暂时无法消除

对应BSD风格，可能还有额外的状态：
- `<`: 高优先级
- `N`: 低优先级
- `L`: 已将页锁到内存中
- `s`: 主进程
- `l`: 表示多线程
- `+`: 位于前台的进程

## 常用示例
显示所有进程：
```bash
# ps -ef 和 ps -el 都是显示所有进程
$ ps  -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Jul15 ?        00:00:00 /init
root        17     1  0 Jul15 ?        00:07:58 /usr/sbin/smartdns
root      1582     1  0 Aug05 tty1     00:00:00 /init
qlel      1583  1582  0 Aug05 tty1     00:00:00 -bash
root      1592     1  0 Aug05 tty2     00:00:00 /init
qlel      1593  1592  0 Aug05 tty2     00:00:00 -bash
qlel      1736  1583  1 10:53 tty1     00:00:00 ps -ef

# -el 会多些字段
$ ps  -el
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S     0     1     0  0  80   0 -  2235 ?      ?        00:00:00 init
0 S     0    17     1  0  80   0 -  3235 ?      ?        00:07:58 smartdns
0 S     0  1582     1  0  80   0 -  2236 -      tty1     00:00:00 init
0 S  1000  1583  1582  0  80   0 -  3792 -      tty1     00:00:00 bash
0 S     0  1592     1  0  80   0 -  2236 -      tty2     00:00:00 init
0 S  1000  1593  1592  0  80   0 -  3759 -      tty2     00:00:00 bash
0 R  1000  1737  1583  0  80   0 -  4139 -      tty1     00:00:00 ps
```

显示指定名称的进程：
```bash
$ ps -C smartdns,bash
  PID TTY          TIME CMD
   17 ?        00:07:58 smartdns
 1583 tty1     00:00:00 bash
 1593 tty2     00:00:00 bash

$ ps -f -C smartdns,bash
UID        PID  PPID  C STIME TTY          TIME CMD
root        17     1  0 Jul15 ?        00:07:58 /usr/sbin/smartdns
qlel      1583  1582  0 Aug05 tty1     00:00:00 -bash
qlel      1593  1592  0 Aug05 tty2     00:00:00 -bash
```
根据线程来过滤进程:
```bash
ps -L 1234
```
显示指定字段列：
```bash
$ ps -e -o uid,pid,f,s,%cpu,%mem,cmd,comm
  UID   PID F S %CPU %MEM CMD                         COMMAND
    0     1 0 S  0.0  0.0 /init                       init
    0    17 0 S  0.0  0.0 /usr/sbin/smartdns          smartdns
    0  1582 0 S  0.0  0.0 /init                       init
 1000  1583 0 S  0.0  0.0 -bash                       bash
    0  1592 0 S  0.0  0.0 /init                       init
 1000  1593 0 S  0.0  0.0 -bash                       bash
 1000  1746 0 R  0.0  0.0 ps -e -o uid,pid,f,s,%cpu,% ps
```