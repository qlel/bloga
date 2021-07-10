+++
title = "Linux Systemd"
description = ""
tags = ['linux', 'shell']
date =  "2021-04-08T17:07:04+08:00"
lastmod = "2021-04-08T17:07:04+08:00"
+++

systemd 是一个专用于 Linux 操作系统的系统与服务管理器。 当作为启动进程(PID=1)运行时， 它将作为初始化系统运行， 也就是启动并维护各种用户空间的服务。
<!--more-->

```bash
# 显示帮助信息
systemctl -h

# 显示版本
systemctl --version
```
## 概念
systemd 架构图:
![systemd 架构图](/systemd/systemd_schema.png)

systemd 将各种系统启动和运行相关的对象， 表示为各种不同类型的单元(Unit)， 并提供了处理不同单元之间依赖关系的能力。

大部分单元都静态的定义在单元文件中， 但是有少部分单元则是动态自动生成的.

systemd 有12种不同类型的单元文件:
单元类型|单元文件扩展名|说明
-|-|-
service|`.service`|用于封装一个后台服务进程(守护进程)。
socket|`.socket`|监控来自于系统或网络的数据消息
target|`.target`|用于将多个单元在逻辑上组合在一起。
device|`.device`|用于封装一个设备文件，可用于基于设备的启动。
mount|`.mount`|用于封装一个文件系统挂载点(也向后兼容传统的 /etc/fstab 文件)
automount|`.automount`|用于封装一个文件系统自动挂载点。它取代了传统的 autofs 服务。
timer|`.timer`|用于封装一个基于时间触发的动作。它取代了传统的 atd,  crond 等任务计划服务。
swap|`.swap`|用于封装一个交换分区或者交换文件。
path|`.path`|用于根据文件系统上特定对象的变化来启动其他服务。
slice|`.slice`|用于控制特定 CGroup 内(例如一组 service 与 scope 单元)所有进程的总体资源占用。
scope|`.scope`|这种 Unit 文件不是用户创建的，而是 Systemd 运行时产生的，描述一些系统服务的分组信息
snapshot|`.snapshot`|用于表示一个由 systemctl snapshot 命令创建的 Systemd Units 运行状态快照，可以切回某个快照


单元有5种不同的状态:
单元状态|说明
-|-
`active`|活动
`inactive`|停止
`activating`|启动中
`deactivating`|暂停中
`failed`|失败
可用`systemctl status 某个单元`, 查看某个单元的状态. 例如: 
```bash
# 查看ssh服务单元状态
systemctl status ssh.service
```

## 系统管理
### systemctl
`systemctl` 是 Systemd 的主命令，用于管理系统。

中文文档: http://www.jinbuguo.com/systemd/systemctl.html
```bash
# 重启系统
$ sudo systemctl reboot

# 关闭系统，切断电源
$ sudo systemctl poweroff

# 关闭系统，但不切断电源, CPU停止工作
$ sudo systemctl halt

# 暂停系统, 休眠到内存。
$ sudo systemctl suspend

# 让系统进入冬眠状态, 休眠到硬盘。
$ sudo systemctl hibernate

# 让系统进入混合休眠状态, 同时休眠到内存和硬盘。
$ sudo systemctl hybrid-sleep

# 启动进入救援状态（单用户状态）
$ sudo systemctl rescue
```
### systemd-analyze
分析与调试 systemd 系统管理器

中文文档: http://www.jinbuguo.com/systemd/systemd-analyze.html
```bash
# 查看系统启动耗时
$ systemd-analyze

# 按照每个单元花费的启动时间降序列出所有当前正处于活动(active)状态的单元。
$ systemd-analyze blame

# 为指定的单元以树状形式显示时间关键链
# 显示的信息: `@时间` 表示时启动的时间点; `+时间` 表示启动完成消耗的时间
$ systemd-analyze critical-chain [UNIT…]

# 输出一个svg图像, 包含每个单元的启动时刻和启动完成的消耗时间
$ systemd-analyze plot > abc.svg

# 按照 GraphViz dot 格式输出单元间的依赖关系图。
# systemd-analyze dot | dot -Tsvg > systemd.svg
$ systemd-analyze dot

# 按照人类易读的格式输出全部单元的状态
$ systemd-analyze dump

# 列出与单元相关的全部目录
$ systemd-analyze unit-paths

# 打印出 systemd 守护进程当前的日志等级
$ systemd-analyze log-level

# 打印出 systemd 守护进程当前的日志目标
$ systemd-analyze log-target

# 校验指定的单元文件的正确性，并显示发现的错误
$ systemd-analyze verify 单元文件
```

### hostnamectl
用于查看当前主机的信息。

http://www.jinbuguo.com/systemd/hostnamectl.html

```bash
# 显示当前的主机名及其他相关信息
$ hostnamectl
$ hostnamectl status

# 设置主机名
$ hostnamectl set-hostname NAME
```

### localectl
本地化设置。

http://www.jinbuguo.com/systemd/localectl.html
```bash
# 查看本地化设置
$ localectl
$ localectl status

# 列出所有可用(已编译)的语言环境
$ localectl list-locales

# 本地化设置, 如: localectl set-locale LANG=en_GB.utf8
$ localectl set-locale VARIABLE=LOCALE…

# 列出所有可用(已编译)的键盘映射
$ localectl list-keymaps

# 设置键盘映射, 设置为 `us` 即可
$ localectl set-keymap MAP
```

### timedatectl
控制系统的时间与日期

http://www.jinbuguo.com/systemd/timedatectl.html
```bash
# 显示系统时钟与RTC的当前状态，包括时区设置以及网络时间同步服务的状态。
$ timedatectl
$ timedatectl status
$ timedatectl show # 以机器可读形式显示

# 列出所有可用时区, 可用于设置时区 set-timezone
$ timedatectl list-timezones

# 设置时区
$ timedatectl set-timezone [TIMEZONE]

# 将系统时钟设为指定的时间，并同时更新RTC时间
# [TIME] 是一个形如 "2012-10-30 18:17:16" 的时间字符串
$ timedatectl set-time [TIME]

# 接受一个布尔值，表示是否开启网络时间同步(若可用)
$ timedatectl set-ntp [BOOL]
```

### loginctl
控制 systemd 登录管理器

http://www.jinbuguo.com/systemd/loginctl.html

session 会话命令: 
```bash
# 列出当前所有的会话
$ loginctl 
$ loginctl list-sessions

# 显示简洁的会话状态信息，后跟最近的日志。
$ loginctl session-status [ID…]

# 显示会话的各项属性值, 易于机器阅读形式
$ loginctl show-session [ID…]

# 激活会话。也就是将处于后台的会话切换到前台
$ loginctl activate [ID]

# 锁定/解锁会话 (如果会话支持屏幕锁)
$ loginctl lock-session [ID…]
$ loginctl unlock-session [ID…]

# 锁定/解锁 所有支持屏幕锁的会话
$ loginctl lock-sessions
$ loginctl unlock-sessions

# 结束指定的会话。也就是杀死指定会话的所有进程、释放所有与此会话相关的资源。
$ loginctl terminate-session ID…
```
user 用户命令: 
```bash
# 列出当前登录的用户
$ loginctl list-users

# 显示简洁的已登录用户状态信息，后跟最近的日志。
$ loginctl user-status [USER…]

# 显示用户的各项属性值
$ loginctl show-user [USER…]

# 启用/禁止用户逗留(相当于保持登录状态)。
$ loginctl enable-linger [USER…]
$ loginctl disable-linger [USER…]

# 结束指定用户的会话。这将杀死该用户的所有会话中的所有进程，同时释放与此用户有关的所有资源。
$ loginctl terminate-user USER…
```
seat 席位命令:
```bash
# 列出当前本机上的所有可用席位
$ loginctl list-seats

# 显示简洁的席位信息，后跟最近的日志。
$ loginctl seat-status [NAME…]

# 显示席位的各项属性值
$ loginctl seat-user [NAME…]

# 将指定的设备(DEVICE) 持久的连接到指定的席位(NAME)上。 
# 设备可以用相对于 /sys 文件系统的设备路径表示。 
# 要创建一个新席位，至少需要连接一个显卡。 
# 席位名称必须以 "seat" 开头， 后跟 a–z, A–Z, 0–9, "-", "_" 字符
$ loginctl attach NAME DEVICE…

# 删除所有先前用 attach 命令连接的设备
# 同时也删除了所有先前用 attach 命令创建的席位 
# 调用此命令之后，所有自动生成的席位将会被保留
# 同时所有席位设备将会连接到自动生成的席位上。
$ flush-devices

# 结束指定席位的会话。这将杀死指定席位上的所有会话进程，同时释放与之关联的所有资源。
$ loginctl terminate-user NAME…
```

## 单元Unit 
Systemd 可以管理所有系统资源。不同的资源统称为 Unit(单元)

前面概念说了一共12中单元, 简述为:
- Service unit：系统服务
- Target unit：多个 Unit 构成的一个组
- Device Unit：硬件设备
- Mount Unit：文件系统的挂载点
- Automount Unit：自动挂载点
- Path Unit：文件或路径
- Scope Unit：不是由 Systemd 启动的外部进程
- Slice Unit：进程组
- Snapshot Unit：Systemd 快照，可以切回某个快照
- Socket Unit：进程间通信的 socket
- Swap Unit：swap 文件
- Timer Unit：定时器

### list相关
`list-units`

列出相关单元信息, 可以使用相关选项过滤: `--state`, `--type`, `--failed`等
```bash
# 格式
$ systemctl list-units [PATTERN…]

# 已加载到内存中(正在运行)的单元 Unit
$ systemctl list-units

# 显示 ssh.service 相关信息
$ systemctl list-units ssh.service

# 列出所有Unit
$ systemctl list-units --all

# 列出所有正在运行的、类型为 service 的 Unit
$ systemctl list-units --type=service

# 列出所有加载失败的 Unit
$ systemctl list-units --failed

# 列出所有状态为 inactive 的 Unit
$ systemctl list-units --all --state=inactive

# 列出所有可用的类型
$ systemctl --type=help

# 列出所有可用的状态
$ systemctl --state=help
```
***
`list-sockets`

列出当前已加载到内存中的套接字(socket)单元，并按照监听地址排序。
- `--show-types` 显示套接字类型
- `--all` 列出所有套接字(socket)单元
- `--state=`  列出指定状态的套接字(socket)单元
```bash
# 格式
$ systemctl list-sockets [PATTERN…]

# 列出当前已加载到内存中的套接字(socket)单元，并按照监听地址排序。
$ systemctl list-sockets

# 列出已加载到内存中的套接字(socket)单元, 并显示套接字类型
$ systemctl list-sockets --show-types
```

***
`list-timers`

列出当前已加载到内存中的定时器(timer)单元，并按照下次执行的时间点排序。
- `--all` 列出所有定时器(timer)单元
- `--state=`  列出指定状态的定时器(timer)单元

显示的信息列:
- `NEXT` 列 显示下次执行的时间点
- `LEFT` 列 显示距离下次执行还剩多长时间
- `LAST` 列 显示上次执行的时间点
- `PASSED` 列 显示距离上次执行过去了多长时间
- `UNIT` 列 显示定时器单元的名称
- `ACTIVATES` 列 显示定时器单元将会启动的服务

```bash
# 格式
$ systemctl list-timers [PATTERN…]

# 列出当前已加载到内存中的定时器(timer)单元，并按照下次执行的时间点排序。
$ systemctl list-timers 

# 列出所有定时器(timer)单元
$ systemctl list-timers --all
```
### status
状态相关
```bash
# 格式
$ systemctl status [PATTERN…|PID…]

# 显示系统状态
$ systemctl status

# 显示 bluetooth.service 状态
$ systemctl status bluetooth.service
```
除了`status`命令，`systemctl`还提供了三个查询状态的简单方法，主要供脚本内部的判断语句使用。
```bash
# 显示某个 Unit 是否正在运行
$ systemctl is-active application.service

# 显示某个 Unit 是否处于启动失败状态
$ systemctl is-failed application.service

# 显示某个 Unit 服务是否建立了启动链接
$ systemctl is-enabled application.service
```

### Unit管理
```bash
# 立即启动一个服务
$ sudo systemctl start apache.service

# 立即停止一个服务
$ sudo systemctl stop apache.service

# 重启一个服务
$ sudo systemctl restart apache.service

# 杀死一个服务的所有子进程
$ sudo systemctl kill apache.service

# 重新加载一个服务的专属配置文件, 例如 httpd.conf; 注意区别单元配置文件
$ sudo systemctl reload apache.service

# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload

# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.service

# 显示某个 Unit 的指定属性的值
$ systemctl show -p CPUShares httpd.service

# 设置某个 Unit 的指定属性
$ sudo systemctl set-property httpd.service CPUShares=500
```
### 依赖关系
`systemctl list-dependencies [UNIT]`

显示单元的依赖关系。

如果没有明确指定单元的名称，那么表示显示 default.target 的依赖关系树。

默认情况下，仅以递归方式显示 target 单元的依赖关系树，而对于其他类型的单元，仅显示一层依赖关系(不递归)。 但如果使用了 `--all` 选项， 那么将对所有类型的单元都强制递归的显示完整的依赖关系树。

还可以使用 `--reverse`, `--after`, `--before` 选项指定 仅显示特定类型的依赖关系。

注意， 因为此命令仅列出当前已加载的单元(并不包含未加载单元中定义的依赖关系)， 所以，此命令不能全部列出反向依赖于指定单元的完整列表。

```bash
# 显示 default.target 的依赖关系树
$ systemctl list-dependencies

# 显示 ssh.service 的依赖关系树
$ systemctl list-dependencies ssh.service

# 递归显示 ssh.service 的依赖关系树
$ systemctl list-dependencies ssh.service --all 
```

## 单元配置文件
每一个 Unit 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit 。

systemd 将会从一组在编译时设定好的"单元目录"中加载单元文件(详见下面的两个表格)， 并且较先列出的目录拥有较高的优先级(细节见后文)。 也就是说，高优先级目录中的文件， 将会覆盖低优先级目录中的同名文件。

如果设置了 `$SYSTEMD_UNIT_PATH` 环境变量， 那么它将会取代预设的单元目录。 如果 `$SYSTEMD_UNIT_PATH` 以 "`:`" 结尾， 那么预设的单元目录将会被添加到该变量值的末尾。

**当 systemd 以系统实例(--system)运行时，加载单元的先后顺序(较前的目录优先级较高)：**
系统单元目录|描述
-|-
`/etc/systemd/system.control`|通过 dbus API 创建的永久系统单元
`/run/systemd/system.control`|通过 dbus API 创建的临时系统单元
`/run/systemd/transient`|动态配置的临时单元(系统与全局用户共用)
`/run/systemd/generator.early`|生成的高优先级单元(系统与全局用户共用)
`/etc/systemd/system`|由管理员创建的系统单元
`/run/systemd/system`|运行时配置的系统单元
`/run/systemd/generator`|生成的中优先级系统单元
`/usr/local/lib/systemd/system`|管理员安装的系统单元
`/usr/lib/systemd/system`|发行版软件包安装的系统单元
`/run/systemd/generator.late`|生成的低优先级系统单元

**当 systemd 以用户实例(--user)运行时，加载单元的先后顺序(较前的目录优先级较高)：**
用户单元目录|描述
-|-
`$XDG_CONFIG_HOME/systemd/user.control` 或 `~/.config/systemd/user.control`|通过 dbus API 创建的永久私有用户单元(仅在未设置 $XDG_CONFIG_HOME 时才使用 ~/.config 来替代)
`$XDG_RUNTIME_DIR/systemd/user.control`|通过 dbus API 创建的临时私有用户单元
`/run/systemd/transient`|动态配置的临时单元(系统与全局用户共用)
`/run/systemd/generator.early`|生成的高优先级单元(系统与全局用户共用)
`$XDG_CONFIG_HOME/systemd/user` 或 `$HOME/.config/systemd/user`|用户配置的私有用户单元(仅在未设置 $XDG_CONFIG_HOME 时才使用 ~/.config 来替代)
`$XDG_CONFIG_DIRS/systemd/user` 或 `/etc/xdg/systemd/user`|XDG基本目录规范指定的其他配置目录（如果设置，则使用$XDG_CONFIG_DIRS，否则使用/etc/xdg）
`/etc/systemd/user`|管理员创建的用户单位
`$XDG_RUNTIME_DIR/systemd/user`|运行时配置的私有用户单元(仅当 $XDG_RUNTIME_DIR 已被设置时有效)
`/run/systemd/user`|运行时配置的全局用户单元
`$XDG_RUNTIME_DIR/systemd/generator`|生成的中优先级私有用户单元
`$XDG_DATA_HOME/systemd/user` 或 `$HOME/.local/share/systemd/user`|软件包安装在用户家目录中的私有用户单元(仅在未设置 $XDG_DATA_HOME 时才使用 ~/.local/share 来替代)
`$XDG_DATA_DIRS/systemd/user` 或 `/usr/local/share/systemd/user` 和 `/usr/share/systemd/user`|由XDG基本目录规范指定的其他数据目录
`$dir/systemd/user`(对应 `$XDG_DATA_DIRS` 中的每一个目录($dir))|额外安装的全局用户单元，对应 $XDG_DATA_DIRS(默认值="/usr/local/share/:/usr/share/") 中的每一个目录
`/usr/local/lib/systemd/user`|管理员安装的全局用户单元
`/usr/lib/systemd/user`|发行版软件包安装的全局用户单元
`$XDG_RUNTIME_DIR/systemd/generator.late`|生成的低优先级私有用户单元



可用`$ systemd-analyze unit-paths`查看相关单元文件路径:
```bash
$ systemd-analyze unit-paths
/etc/systemd/system.control
/run/systemd/system.control
/run/systemd/transient
/run/systemd/generator.early
/etc/systemd/system
/etc/systemd/system.attached
/run/systemd/system
/run/systemd/system.attached
/run/systemd/generator
/usr/local/lib/systemd/system
/lib/systemd/system
/usr/lib/systemd/system
/run/systemd/generator.late
```

Systemd 默认从目录`/etc/systemd/system/`读取配置文件。但是，里面存放的大部分文件都是**符号链接**，指向目录`/usr/lib/systemd/system/`，真正的配置文件存放在此目录。

```bash
$ ls -l /etc/systemd/system
total 28
drwxr-xr-x 2 root root 4096 Mar  3 21:06 bluetooth.target.wants
lrwxrwxrwx 1 root root   42 Mar  3 21:08 dbus-fi.w1.wpa_supplicant1.service -> /lib/systemd/system/wpa_supplicant.service
lrwxrwxrwx 1 root root   37 Mar  3 21:06 dbus-org.bluez.service -> /lib/systemd/system/bluetooth.service
lrwxrwxrwx 1 root root   45 Mar  3 20:43 dbus-org.freedesktop.timesync1.service -> /lib/systemd/system/systemd-timesyncd.service
drwxr-xr-x 2 root root 4096 Mar  3 20:43 getty.target.wants
drwxr-xr-x 2 root root 4096 Mar  3 21:08 multi-user.target.wants
drwxr-xr-x 2 root root 4096 Mar  3 20:43 network-online.target.wants
drwxr-xr-x 2 root root 4096 Mar  3 20:45 sockets.target.wants
lrwxrwxrwx 1 root root   31 Mar  3 21:08 sshd.service -> /lib/systemd/system/ssh.service
drwxr-xr-x 2 root root 4096 Mar  3 20:46 sysinit.target.wants
lrwxrwxrwx 1 root root   35 Mar  3 20:43 syslog.service -> /lib/systemd/system/rsyslog.service
drwxr-xr-x 2 root root 4096 Mar  3 21:08 timers.target.wants
```

`systemctl enable UNIT…|PATH…`命令用于启用指定的单元, 在上面两个目录之间，建立符号链接关系。
```bash
$ sudo systemctl enable clamd@scan.service
# 等同于
$ sudo ln -s '/usr/lib/systemd/system/clamd@scan.service' '/etc/systemd/system/multi-user.target.wants/clamd@scan.service'
```
如果**配置文件**里面设置了开机启动，`systemctl enable`命令相当于激活开机启动。

与之对应的，`systemctl disable`命令用于在两个目录之间，撤销符号链接关系，相当于撤销开机启动。
```bash
$ sudo systemctl disable clamd@scan.service
```
配置文件的后缀名，就是该 Unit 的种类，比如`sshd.socket`。如果省略，Systemd 默认后缀名为`.service`，所以sshd会被理解成`sshd.service`。

### 状态
`systemctl list-unit-files [PATTERN…]`

列出所有已安装的单元配置文件及其状态.
```bash
# 列出所有已安装的单元配置文件及其状态.
$ systemctl list-unit-files

# 列出指定类型的单元配置文件及其状态.
$ systemctl list-unit-files --type=service
```
显示每个单元配置文件的状态，一共有四种。
- `enabled`：已建立启动链接
- `disabled`：没建立启动链接
- `static`：该配置文件没有`[Install]`部分（无法执行），只能作为其他配置文件的依赖
- `masked`：该配置文件被禁止建立启动链接

>注意，从配置文件的状态无法看出，该 Unit 是否正在运行。这必须执行前面提到的`systemctl status`命令。

修改单元配置文件后, 需要重新加载:
```bash
# 注意区别 reload , reload 只是重载服务进程的专属配置, 例如 httpd.conf
$ sudo systemctl daemon-reload
```

### 格式
单元配置文件就是普通的文本文件，类似于`.ini`风格的文件, 可以用文本编辑器打开。

语法可查看: http://www.jinbuguo.com/systemd/systemd.syntax.html

`systemctl cat NAME`命令可以查看单元配置文件的内容。
```bash
$ systemctl cat ssh.service
# /lib/systemd/system/ssh.service
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
Alias=sshd.service
```
从上面的输出可以看到，配置文件分成几个区块。每个区块的第一行，是用方括号表示的区块名，比如`[Unit]`。
>注意，配置文件的区块名和字段名，都是大小写敏感的。

每个区块内部是一些等号连接的键值对。
>注意，键值对的等号两侧不能有空格。

空行、以 "`#`" 或 "`;`" 开头的行(用作注释) 都将被忽略。

行尾的反斜线(`\`)是续行符，表示将下一行拼接到本行末尾，同时将反斜线本身替换为一个空格。 这样可以将一个超长的行分拆为几个较短的行。

配置文件中的**布尔值**可以有多种写法。 真值可以写为： `1, yes, true, on` 之一，假值可以写为： `0, no, false, off` 之一。

配置文件中的时间长度可以有多种写法：不带任何后缀的一个纯数值表示秒数； 亦可在纯数值后面加上时间单位； 亦可以将多个带有时间单位后缀的时间长度使用空格连接起来，表示这几段时间长度之和。 例如： "50" 表示50秒； "2min 200ms" 表示2分200毫秒，也就是120200毫秒； 可使用的时间单位如下： "s", "min", "h", "d", "w", "ms", "us" 。

### `[Unit]`
http://www.jinbuguo.com/systemd/systemd.unit.html

`[Unit]`区块通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系。常用指令:
- `Description`
有利于人类阅读的、对单元进行简单描述的字符串。
- `Documentation`
文档地址. 可接受 "http://", "https://", "file:", "info:", "man:" 五种URI类型
- `Requires`
当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
- `Wants`
此选项是 Requires 的弱化版。 当此单元被启动时， 所有这里列出的其他单元只是尽可能被启动。 但是，即使某些单元不存在或者未能启动成功， 也不会影响此单元的启动。 推荐使用此选项来设置单元之间的依赖关系。
- `BindsTo`
与 Requires 类似，但是依赖性更强： 如果这里列出的任意一个单元停止运行或者崩溃，那么也会连带导致该单元自身被停止。 这就意味着该单元可能因为 这里列出的任意一个单元的 主动退出、某个设备被拔出、某个挂载点被卸载， 而被强行停止。
- `PartOf`
与 Requires 类似， 不同之处在于：仅作用于单元的停止或重启。 其含义是，当停止或重启这里列出的某个单元时， 也会同时停止或重启该单元自身。 注意，这个依赖是单向的， 该单元自身的停止或重启并不影响这里列出的单元。
- `Conflicts`
指定单元之间的冲突关系。 接受一个空格分隔的单元列表，表明该单元不能与列表中的任何单元共存， 也就是说：(1)当此单元启动的时候，列表中的所有单元都将被停止； (2)当列表中的某个单元启动的时候，该单元同样也将被停止。 注意，此选项与 After= 和 Before= 选项没有任何关系。
- `Before`
假定 foo.service 单元包含 Before=bar.service 设置， 那么当两个单元都需要启动的时候， bar.service 将会一直延迟到 foo.service 启动完毕之后再启动。 注意，停止顺序与启动顺序正好相反，也就是说， 只有当 bar.service 完全停止后，才会停止 foo.service 单元。
- `After`
假定 foo.service 单元包含 After=bar.service 设置， 那么当两个单元都需要启动的时候， foo.service 将会一直延迟到 bar.service 启动完毕之后再启动。 注意，停止顺序与启动顺序正好相反，也就是说， 只有当 foo.service 完全停止后，才会停止 bar.service 单元。
- `OnFailure`
当该单元进入失败("failed")状态时， 将会启动列表中的单元。
- `Condition...`
当前 Unit 运行必须满足的条件，否则不会运行
- `Assert...`
当前 Unit 运行必须满足的条件，否则会报启动失败

### `[Install]`
http://www.jinbuguo.com/systemd/systemd.unit.html

用来定义如何启动，以及是否开机启动。它的主要字段如下:
- `Alias`
启用时使用的别名，可以设为一个空格分隔的别名列表。 每个别名的后缀(也就是单元类型)都必须与该单元自身的后缀相同。
- `Also`
设置此单元的附属单元， 可以设为一个空格分隔的单元列表。 表示当使用 systemctl enable 启用 或 systemctl disable 停用 此单元时， 也同时自动的启用或停用附属单元。
- `WantedBy`
- `RequiredBy`
接受一个空格分隔的单元列表， 表示在使用 systemctl enable 启用此单元时， 将会在`/etc/systemd/system`目录中每个列表单元的 .wants/ 或 .requires/ 目录中创建一个指向该单元文件的软连接。 这相当于为每个列表中的单元文件添加了 Wants=此单元 或 Requires=此单元 选项。 这样当列表中的任意一个单元启动时，该单元都会被启动。

例如, 上面的ssh.service单元配置文件在`/etc/systemd/system`目录创建的软链接:
```bash
$ ls -l /etc/systemd/system/multi-user.target.wants/
total 0
lrwxrwxrwx 1 root root 35 Mar  3 21:06 anacron.service -> /lib/systemd/system/anacron.service
lrwxrwxrwx 1 root root 41 Mar  3 20:46 console-setup.service -> /lib/systemd/system/console-setup.service
lrwxrwxrwx 1 root root 32 Mar  3 20:43 cron.service -> /lib/systemd/system/cron.service
lrwxrwxrwx 1 root root 38 Mar  3 20:43 networking.service -> /lib/systemd/system/networking.service
lrwxrwxrwx 1 root root 36 Mar  3 20:43 remote-fs.target -> /lib/systemd/system/remote-fs.target
lrwxrwxrwx 1 root root 35 Mar  3 20:43 rsyslog.service -> /lib/systemd/system/rsyslog.service
lrwxrwxrwx 1 root root 31 Mar  3 21:08 ssh.service -> /lib/systemd/system/ssh.service
lrwxrwxrwx 1 root root 42 Mar  3 21:08 wpa_supplicant.service -> /lib/systemd/system/wpa_supplicant.service
```

### `[Service]`
http://www.jinbuguo.com/systemd/systemd.unit.html

用于 Service 的配置，只有 Service 类型的 Unit 才有这个区块。它的主要字段如下:
- `Type`：定义启动时的进程行为。它有以下几种值。
    - `Type=simple`：默认值，执行ExecStart指定的命令，启动主进程
    - `Type=forking`：以 fork 方式从父进程创建子进程，创建后父进程会立即退出
    - `Type=oneshot`：一次性进程，Systemd 会等当前服务退出，再继续往下执行
    - `Type=dbus`：当前服务通过D-Bus启动
    - `Type=notify`：当前服务启动完毕，会通知Systemd，再继续往下执行
    - `Type=idle`：若有其他任务执行完毕，当前服务才会运行
- `ExecStart`：启动当前服务的命令
- `ExecStartPre`：启动当前服务之前执行的命令
- `ExecStartPost`：启动当前服务之后执行的命令
- `ExecReload`：重启当前服务时执行的命令
- `ExecStop`：停止当前服务时执行的命令
- `ExecStopPost`：停止当其服务之后执行的命令
- `RestartSec`：自动重启当前服务间隔的秒数
- `Restart`：定义何种情况 Systemd 会自动重启当前服务，可能的值包括always（总是重启）、on-success、on-failure、on-abnormal、on-abort、on-watchdog
- `TimeoutSec`：定义 Systemd 停止当前服务之前等待的秒数
- `Environment`：指定环境变量

## Target
启动计算机的时候，需要启动大量的 Unit。如果每一次启动，都要一一写明本次启动需要哪些 Unit，显然非常不方便。Systemd 的解决方案就是 Target。

简单说，Target 就是一个 Unit 组，包含许多相关的 Unit 。启动某个 Target 的时候，Systemd 就会启动里面所有的 Unit。从这个意义上说，Target 这个概念类似于"状态点"，启动某个 Target 就好比启动到某种状态。

传统的`init`启动模式里面，有 RunLevel 的概念，跟 Target 的作用很类似。不同的是，RunLevel 是互斥的，不可能多个 RunLevel 同时启动，但是多个 Target 可以同时启动。

```bash
# 查看运行中的 target
$ systemctl list-units --type=target

# 查看所有的 target
$ systemctl list-units --type=target --all

# 列出已安装的target单元配置文件及其状态.
$ systemctl list-unit-files --type=target

# 查看某个 Target 包含的依赖
$ systemctl list-dependencies multi-user.target

# 查看启动时的默认 Target
$ systemctl get-default

# 设置启动时的默认 Target
$ sudo systemctl set-default multi-user.target

# 切换 Target 时，默认不关闭前一个 Target 启动的进程，
# systemctl isolate 命令改变这种行为，
# 关闭前一个 Target 里面所有不属于后一个 Target 的进程
$ sudo systemctl isolate multi-user.target
```
Target 与 传统 RunLevel 的对应关系如下:
```bash
Traditional runlevel      New target name     Symbolically linked to...

Runlevel 0           |    runlevel0.target -> poweroff.target
Runlevel 1           |    runlevel1.target -> rescue.target
Runlevel 2           |    runlevel2.target -> multi-user.target
Runlevel 3           |    runlevel3.target -> multi-user.target
Runlevel 4           |    runlevel4.target -> multi-user.target
Runlevel 5           |    runlevel5.target -> graphical.target
Runlevel 6           |    runlevel6.target -> reboot.target
```
它与init进程的主要差别如下:
（1）默认的 RunLevel（在`/etc/inittab`文件设置）现在被默认的 Target 取代，位置是`/etc/systemd/system/default.target`，通常符号链接到`graphical.target`（图形界面）或者`multi-user.target`（多用户命令行）。

（2）启动脚本的位置，以前是`/etc/init.d`目录，符号链接到不同的 RunLevel 目录 （比如`/etc/rc3.d`、`/etc/rc5.d`等），现在则存放在`/lib/systemd/system`和`/etc/systemd/system`目录。

（3）配置文件的位置，以前init进程的配置文件是`/etc/inittab`，各种服务的配置文件存放在`/etc/sysconfig`目录。现在的配置文件主要存放在`/lib/systemd`目录，在`/etc/systemd`目录里面的修改可以覆盖原始设置。

## 日志管理
Systemd 统一管理所有 Unit 的启动日志。带来的好处就是，可以只用`journalctl`一个命令，查看所有日志（内核日志和应用日志）。日志的配置文件是`/etc/systemd/journald.conf`。

```bash
# 查看所有日志（默认情况下 ，只保存本次启动的日志）
$ sudo journalctl

# 查看内核日志（不显示应用日志）
$ sudo journalctl -k

# 实时滚动显示最新日志
$ sudo journalctl -f

# 显示尾部的最新10行日志
$ sudo journalctl -n

# 显示尾部指定行数的日志
$ sudo journalctl -n 20

# 反转日志行的输出顺序，也就是显示最新的日志
$ sudo journalctl -r

# 以人类易于阅读的json格式输出
$ sudo journalctl -o json-pretty

# 查看系统本次启动的日志
$ sudo journalctl -b
$ sudo journalctl -b -0

# 查看上一次启动的日志（需更改设置）
$ sudo journalctl -b -1

# 查看指定单元的日志
# 如: sudo journalctl -u ssh.service
$ sudo journalctl -u UNIT|PATTERN

# 查看属于特定用户会话单元的日志
$ sudo journalctl --user-unit=qlel

# 根据日志等级过滤输出, 仅显示小于或等于此等级的日志
# 等级对应关系: "emerg" (0), "alert" (1), "crit" (2), "err" (3),
# "warning" (4), "notice" (5), "info" (6), "debug" (7)
$ sudo journalctl -p 6

# -S, --since= 显示晚于指定时间的日志
# -U, --until= 显示早于指定时间的日志
# 参数的格式类似 "2012-10-30 18:17:16", 
# 具体看http://www.jinbuguo.com/systemd/systemd.time.html
$ sudo journalctl -S '2021-03-07 12:00:00'
$ sudo journalctl -S '2021-03-07 12:00:00' -U '2021-03-07 13:00:00'
$ sudo journalctl -S '2 min ago'
$ sudo journalctl -S yesterday

# 查看指定服务的日志
$ sudo journalctl /usr/lib/systemd/systemd

# 查看指定进程的日志
$ sudo journalctl _PID=1

# 查看某个路径的脚本的日志
$ sudo journalctl /usr/bin/bash

# 查看指定用户的日志
$ sudo journalctl _UID=33 --since today

# 日志默认分页输出，--no-pager 改为正常的标准输出
$ sudo journalctl --no-pager

# 显示所有日志磁盘占用总量
$ sudo journalctl --disk-usage

# 限制日志归档文件的最大磁盘使用量(可以使用 "K", "M", "G", "T" 后缀)
$ sudo journalctl --vacuum-size=1G

# 指定日志文件保存多久
# 可以使用 "s", "m", "h", "days", "weeks", "months", "years" 后缀
$ sudo journalctl --vacuum-time=1years

# 限制日志归档文件的最大数量
$ sudo journalctl --vacuum-files=
```
## Timer
中文文档: http://www.jinbuguo.com/systemd/systemd.timer.html

以 "`.timer`" 为后缀的单元文件， 封装了一个由 systemd 管理的定时器， 以支持基于定时器的启动。

每个定时器单元都必须有一个与其匹配的单元，用于在特定的时间启动。可以在单元配置文件中`[Timer]`小结通过指令 `Unit=` 明确指定要匹配的单元。若未指定，则默认是与该单元名称相同的 `.service` 单元(不算后缀)。例如 `foo.timer` 默认匹配 `foo.service` 单元。

>注意，如果在启动时间点到来的时候，匹配的单元已经被启动， 那么将不执行任何动作，也不会启动任何新的服务实例。 因此，那些设置了 `RemainAfterExit=yes`(当该服务的所有进程全部退出之后，依然将此服务视为处于活动状态) 的服务单元一般不适合使用基于定时器的启动。 因为这样的单元仅会被启动一次，然后就永远处于活动(active)状态了。

此单元文件配置文件中有专门的`[Timer]`小结, 下面有一些配置指令:
- `OnActiveSec`
定时器生效后，多少时间开始执行任务
- `OnBootSec`
系统启动后，多少时间开始执行任务
- `OnStartupSec`
Systemd 进程启动后，多少时间开始执行任务
- `OnUnitActiveSec`
该单元上次执行后，等多少时间再次执行
- `OnUnitInactiveSec`
定时器上次关闭后多少时间，再次执行
- `OnCalendar`
基于绝对时间，而不是相对时间执行. 如果定时器单元在启动时已经超过了该指令设置的某个触发时间，那么错过就错过了，不会尝试去补救，只能等待下一个触发时间。受到 AccuracySec 选项的影响
- `AccuracySec`
设置定时器的触发精度。如果因为各种原因，任务必须推迟执行，推迟的最大秒数，默认是60秒
- `RandomizedDelaySec`
将此单元的定时器随机延迟一小段时间， 这一小段时间的长度 介于零到该指令设置的时间长度之间， 以均匀概率分布。 默认值是零，表示不延迟。
- `Unit`
真正要执行的任务单元，默认是同名的带有`.service`后缀的单元
- `Persistent`
此选项仅对 `OnCalendar` 指令定义的日历定时器有意义。 若设为"true"，则表示将匹配单元的上次触发时间永久保存在磁盘上。 这样，当定时器单元再次被启动时， 如果匹配单元本应该在定时器单元停止期间至少被启动一次， 那么将立即启动匹配单元。 这样就不会因为关机而错过必须执行的任务。 默认值为 false
在计时器单元上使用`systemctl clean --what = state…`从磁盘上删除此选项维护的时间戳文件。 特别是，在卸载计时器单元之前，请使用此命令。 
- `WakeSystem`
若设为"true"， 则表示当某个定时器到达触发时间点时， 唤醒正在休眠的系统并阻止系统进入休眠状态。 注意， 此选项仅确保唤醒系统， 而不关心任务执行完成之后是否需要再次休眠系统。 默认值为 false

### 时间格式
http://www.jinbuguo.com/systemd/systemd.time.html

语法|说明
-|-
`usec`, `us`, `µs`|微秒
`msec`, `ms`|毫秒
`seconds`, `second`, `sec`, `s`|秒
`minutes`, `minute`, `min`, `m`|分钟
`hours`, `hour`, `hr`, `h`|小时
`days`, `day`, `d`|天
`weeks`, `week`, `w`|星期
`months`, `month`, `M`|月[=30.44天]
`years`, `year`, `y`|年[=365.25天]

下面是一些有效的时长字符串实例：
```bash
2 h
2hours
48hr
1y 12month
55s500ms
300ms20s 5day
```

- `OnUnitActiveSec=1h`
表示一小时执行一次任务
- `OnCalendar=*-*-* 02:00:00`
表示每天凌晨两点执行.
这是日历事件表达式, 类似于专用于时间戳的正则表达式
- `OnCalendar=Mon *-*-* 02:00:00`

### Service和Timer示例
自定义一个定时任务——每过30s就向msg.txt输入当前时间.

1. 创建一个shell脚本: `abcd.sh`

```bash
$ nano abcd.sh

# 查看 abcd.sh 内容
$ cat abcd.sh
#!/bin/bash
date >> /home/qlel/msg.txt
echo 'hello abcddddddddd'

# 赋予执行权限
$ chmod +x ./abcd.sh

# 执行2次看看
$ ./abcd.sh

# 查看是否执行成功
$ cat msg.txt
Mon 08 Mar 2021 05:59:18 PM CST
Mon 08 Mar 2021 05:59:23 PM CST
```
再创建一个python文件脚本: `py_test.py`:
```py
import time

now_time = time.strftime('%Y-%m-%d %H:%M:%S') + '\n'

with open('/home/qlel/py_msg.txt','a') as f:
        f.write(now_time)
```
执行2次看看:
```bash
$ python3 /home/qlel/py_test.py
$ python3 /home/qlel/py_test.py
$ cat /home/qlel/py_msg.txt
2021-03-08 18:00:06
2021-03-08 18:00:10
```

2. 创建要执行的 Service 单元

使用命令`sudo systemctl edit --force --full mytimer.service`会自动创建`/etc/systemd/system/mytimer.service`.
- `--force`
如果指定的单元文件不存在，那么将会强制打开一个新的空单元文件以供编辑。
- `--full`
表示使用新编辑的单元文件完全取代原始单元文件，否则默认将新编辑的单元配置片段附加到原始单元文件的末尾

写入以下内容:
```ini
[Unit]
Description=mytimer service

[Service]
ExecStart=/home/qlel/abcd.sh
ExecStartPost=python3 /home/qlel/py_test.py
```
>注意, 单元文件中都要使用**绝对路径**, 还有使用到的**脚本文件中的内容**也要使用**绝对路径**

3. 启动`mytimer.service`单元:

```bash
$ sudo systemctl start mytimer.service
qlel@debianqlel:~$ systemctl status mytimer.service
● mytimer.service - mytimer service
   Loaded: loaded (/etc/systemd/system/mytimer.service; static; vendor preset: enabled)
   Active: inactive (dead)
```
4. 查看结果和日志
```bash
~$ cat ./msg.txt
Mon 08 Mar 2021 05:59:18 PM CST
Mon 08 Mar 2021 05:59:23 PM CST
Mon 08 Mar 2021 06:05:49 PM CST
~$ cat ./py_msg.txt
2021-03-08 18:00:06
2021-03-08 18:00:10
2021-03-08 18:05:49
~$ sudo journalctl -u mytimer.service -n 6
-- Logs begin at Sun 2021-03-07 17:17:01 CST, end at Mon 2021-03-08 18:08:32 CST. --
Mar 08 17:53:47 debianqlel systemd[1]: mytimer.service: Succeeded.
Mar 08 17:53:47 debianqlel systemd[1]: Started mytimer service.
Mar 08 18:05:49 debianqlel systemd[1]: Starting mytimer service...
Mar 08 18:05:49 debianqlel abcd.sh[3927]: hello abcddddddddd
Mar 08 18:05:49 debianqlel systemd[1]: mytimer.service: Succeeded.
Mar 08 18:05:49 debianqlel systemd[1]: Started mytimer service.
```
5. 创建 Timer 单元

使用`sudo systemctl edit --force --full myytimer.timer`会自动创建`/etc/systemd/system/myytimer.timer`

写入以下内容:
```ini
# /etc/systemd/system/myytimer.timer
[Unit]
Description=每10s运行mytimer.service单元

[Timer]
OnActiveSec=1s
AccuracySec=1us
OnUnitActiveSec=10s
Unit=mytimer.service

[Install]
WantedBy=multi-user.target
```
以上配置说明: 
- `OnActiveSec=1s`表示定时器生效后1s执行一次`mytimer.service`(只在定时器生效后执行一次)
- `AccuracySec=1us`表示精度为1微秒, 默认是60秒
- `OnUnitActiveSec=10s`表示该单元上次执行后10s执行一次(可用于循环执行)

> 注意, 只有`OnUnitActiveSec=10s`是不会执行的

- `[Install]`表示当使用`sudo systemctl enable...`设置开机自启动
- `WantedBy=multi-user.target` 详情请看`[Install]`章节

6. 启动定时器

```bash
# 启动定时器
~$ sudo systemctl start myytimer.timer

# 查看状态
~$ systemctl status myytimer.timer
● myytimer.timer - 每10s运行mytimer.service单元
   Loaded: loaded (/etc/systemd/system/myytimer.timer; disabled; vendor preset: enabled)
   Active: active (waiting) since Mon 2021-03-08 21:13:48 CST; 6s ago
  Trigger: Mon 2021-03-08 21:13:59 CST; 4s left

# 查看结果
~$ cat py_msg.txt
2021-03-08 21:13:49
2021-03-08 21:13:59
2021-03-08 21:14:09
2021-03-08 21:14:19
2021-03-08 21:14:29

~$ systemctl list-timers
NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIVATE
Mon 2021-03-08 21:15:19 CST  3s left       Mon 2021-03-08 21:15:09 CST  6s ago       myytimer.timer               mytimer.
Mon 2021-03-08 21:33:47 CST  18min left    Mon 2021-03-08 20:34:21 CST  40min ago    anacron.timer                anacron.
Tue 2021-03-09 00:00:00 CST  2h 44min left Mon 2021-03-08 00:00:38 CST  21h ago      logrotate.timer              logrotat
Tue 2021-03-09 00:00:00 CST  2h 44min left Mon 2021-03-08 00:00:38 CST  21h ago      man-db.timer                 man-db.s
Tue 2021-03-09 06:26:32 CST  9h left       Mon 2021-03-08 19:55:38 CST  1h 19min ago apt-daily.timer              apt-dail
Tue 2021-03-09 06:51:08 CST  9h left       Mon 2021-03-08 06:08:38 CST  15h ago      apt-daily-upgrade.timer      apt-dail
Tue 2021-03-09 13:13:38 CST  15h left      Mon 2021-03-08 13:13:38 CST  8h ago       systemd-tmpfiles-clean.timer systemd-

7 timers listed.
Pass --all to see loaded but inactive timers, too.

# 停止定时器 myytimer.timer
~$ sudo systemctl stop myytimer.timer

# 状态
~$ systemctl status myytimer.timer
● myytimer.timer - 每10s运行mytimer.service单元
   Loaded: loaded (/etc/systemd/system/myytimer.timer; disabled; vendor preset: enabled)
   Active: inactive (dead)
  Trigger: n/a
```
如果需要设置开机自启动:
```bash
# 设置开机自启动
~$ sudo systemctl enable myytimer.timer
[sudo] password for qlel:
Created symlink /etc/systemd/system/multi-user.target.wants/myytimer.timer → /etc/systemd/system/myytimer.timer.

# 取消开机自启动
~$ sudo systemctl disable myytimer.timer
Removed /etc/systemd/system/multi-user.target.wants/myytimer.timer.
```
如果要删除`myytimer.timer`定时器, 直接删除`myytimer.timer`单元文件即可.

### 自启动测试
设置某个脚本开机自启测试。
`test_sys_startup.sh`脚本：
```bash
#!/usr/bin/env bash

date "+%Y-%m-%d %H:%M:%S" >> /home/qlel/my_test_sys.txt
echo "启动测试" >> /home/qlel/my_test_sys.txt
```
切换到root用户，编写service单元文件：
```bash
$ systemctl edit --force --full test01.service
```
`test02.service`文件：
```ini
[Unit]
Description=自启动测试
After=network.target

[Service]
Type=forking
ExecStart=bash /home/qlel/test_sys_startup.sh
User=qlel
Group=qlel

[Install]
WantedBy=multi-user.target
```
如果修改了文件要重新加载：
```bash
$ systemctl daemon-reload
```
启动测试是否成功：
```bash
$ systemctl start test02.service
$ systemctl status test02.service
● test02.service - 自启动测试
   Loaded: loaded (/etc/systemd/system/test02.service; disabled; vendor preset: enabled)
   Active: inactive (dead)

Jun 17 10:32:36 debianqlel systemd[1]: Starting 自启动测试...
Jun 17 10:32:36 debianqlel systemd[1]: test02.service: Succeeded.
Jun 17 10:32:36 debianqlel systemd[1]: Started 自启动测试
```
用户设置开机自启动该脚本：
```bash
$ systemctl enable test02.service
Created symlink /etc/systemd/system/multi-user.target.wants/test02.service → /etc/systemd/system/test02.service.
```
查看：
```bash
$ systemctl list-unit-files --user test01*
UNIT FILE      STATE
test01.service enabled

1 unit files listed.
```
重启系统测试。
