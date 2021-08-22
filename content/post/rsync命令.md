+++
title = "Rsync命令"
date = 2021-08-22T17:54:53+08:00
description = ""
tags = ['shell', 'rsync']
aplayer = false
showToc = true
math = true
showLicense = false
+++

`rsync`是可以实现增量备份的工具。配合任务计划，`rsync`能实现定时或间隔同步，配合`inotify`或`sersync`，可以实现触发式的实时同步。
<!--more-->

`rsync`可以实现scp的远程拷贝(`rsync`不支持远程到远程的拷贝，但scp支持)、cp的本地拷贝、rm删除和"`ls -l`"显示文件列表等功能。但需要注意的是，`rsync`的最终目的或者说其原始目的是实现两端主机的文件同步，因此实现的`scp/cp/rm`等功能仅仅只是同步的辅助手段，且`rsync`实现这些功能的方式和这些命令是不一样的。事实上，`rsync`有一套自己的算法，其算法原理以及`rsync`对算法实现的机制可能比想象中要复杂一些。

## rsync同步基本说明
`rsync`的目的是实现本地主机和远程主机上的文件同步(包括本地推到远程，远程拉取到本地两种同步方式)，也可以实现本地不同路径下文件的同步，但不能实现远程路径1到远程路径2之间的同步(scp可以实现)。

`rsync`的文件同步涉及了 **源文件** 和 **目标文件** 的概念，还涉及了以哪边文件为 **同步基准** 。例如: 
- 想让目标主机上的文件和本地文件保持同步，则是以本地文件为同步基准，将本地文件作为源文件推送到目标主机上。
- 反之，如果想让本地主机上的文件和目标主机上的文件保持同步，则目标主机上的文件为同步基准，实现方式是将目标主机上的文件作为源文件拉取到本地。
- 当然，要保持本地的两个文件相互同步，`rsync`也一样能实现，这就像Linux中`cp`命令一样，以本地某文件作为源，另一文件作为目标文件，但请注意，虽然`rsync`和`cp`能达到相同的目的，但它们的实现方式是不一样的。
- ...

既然是文件同步，在同步过程中必然会涉及到 **源和目标两文件之间版本控制** 的问题，例如:
- 是否要删除源主机上没有但目标上多出来的文件
- 目标文件比源文件更新时是否仍要保持同步
- 遇到软链接时是拷贝软链接本身还是拷贝软链接所指向的文件
- 目标文件已存在时是否要先对其做个备份等等
- ...

`rsync`同步过程中由两部分模式组成:
- 决定哪些文件需要同步的 **检查模式**
    - 检查模式是指按照指定规则来检查哪些文件需要被同步，例如哪些文件是明确被排除不传输的。 **默认情况下，`rsync`使用"quick check"算法快速检查源文件和目标文件的大小、mtime(修改时间)是否一致，如果不一致则需要传输。** 当然，也可以通过在`rsync`命令行中指定某些选项来改变"quick check"的检查模式，比如"`--size-only`"选项表示"quick check"将仅检查文件大小不同的文件作为待传输文件。`rsync`支持非常多的选项，其中检查模式的自定义性是非常有弹性的。
- 文件同步时的 **同步模式**
    - 同步模式是指在文件确定要被同步后，在同步过程发生之前要做哪些额外工作。例如上文所说的是否要先删除源主机上没有但目标主机上有的文件，是否要先备份已存在的目标文件，是否要追踪链接文件等额外操作。rsync也提供非常多的选项使得同步模式变得更具弹性。

相对来说，为rsync手动指定同步模式的选项更常见一些，只有在有特殊需求时才指定检查模式，因为大多数检查模式选项都可能会影响rsync的性能。

## rsync三种工作方式
以下是rsync的语法：
```bash
Local:
    rsync [OPTION...] SRC... [DEST]

Access via remote shell:
    Pull:
        rsync [OPTION...] [USER@]HOST:SRC... [DEST]
    Push:
        rsync [OPTION...] SRC... [USER@]HOST:DEST

Access via rsync daemon:
    Pull:
        rsync [OPTION...] [USER@]HOST::SRC... [DEST]
        rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
    Push:
        rsync [OPTION...] SRC... [USER@]HOST::DEST
        rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST)
```
由此语法可知，rsync有三种工作方式：
1. 本地文件系统上实现同步。
2. 本地主机与远程主机通信。
3. 本地主机通过TCP连接远程主机上的`rsync`守护进程

还有第四种工作方式：通过远程shell也能临时启动一个rsync daemon，这不同于方式(3)，它不要求远程主机上事先启动rsync服务，而是临时派生出rsync daemon，它是单用途的一次性daemon，仅用于临时读取daemon的配置文件，当此次rsync同步完成，远程shell启动的rsync daemon进程也会自动消逝。此通信方式的命令行语法格式同"Access via rsync daemon"，但要求options部分必须明确指定"`--rsh`"选项或其短选项"`-e`"。

>特殊情况：如果指定了源SRC，而没有指定目标DEST，会以类似命令`ls -l`的形式列出源SRC的文件。

Rsync将本地端称为客户端，将远程端称为服务器。不要将服务器与rsync守护进程混淆。守护进程始终是服务器，但服务器可以是守护进程，也可以是远程shell派生的进程。

>注意：源路径如果是一个目录的话，带上尾随斜线和不带尾随斜线是不一样的，不带尾随斜线表示的是整个目录包括目录本身，带上尾随斜线表示的是目录中的文件，不包括目录本身。

## 常用选项
```bash
-v：显示rsync过程中详细信息。可以使用"-vvvv"获取更详细信息。
--partial：默认情况下，如果传输中断，rsync将删除任何部分传输的文件。
           在某些情况下，更希望保留部分传输的文件。使用--partial选项告诉rsync保留部分文件，这将使文件其余部分的后续传输更快。
--progress：显示进度条
-P：显示文件传输的进度信息。(实际上"-P"="--partial --progress"，其中的"--progress"才是显示进度信息的)。
-n --dry-run  ：仅测试传输，而不实际传输。常和"-vvvv"配合使用来查看rsync是如何工作的。
-a --archive  ：归档模式，表示递归传输并保持文件属性。等同于"-rtopgDl"。
-r --recursive：递归到目录中去。
-t --times：保持mtime属性。强烈建议任何时候都加上"-t"，否则目标文件mtime会设置为系统时间，导致下次更新
          ：检查出mtime不同从而导致增量传输无效。
-o --owner：保持owner属性(属主)。
-g --group：保持group属性(属组)。
-p --perms：保持perms属性(权限，不包括特殊权限)。
-D        ：是"--device --specials"选项的组合，即也拷贝设备文件和特殊文件。
-l --links：如果文件是软链接文件，则拷贝软链接本身而非软链接所指向的对象。
-z        ：传输时进行压缩提高效率。
-R --relative：使用相对路径。意味着将命令行中指定的全路径而非路径最尾部的文件名发送给服务端，包括它们的属性。用法见下文示例。
--size-only ：默认算法是检查文件大小和mtime不同的文件，使用此选项将只检查文件大小。
-u --update ：仅在源mtime比目标已存在文件的mtime新时才拷贝。注意，该选项是接收端判断的，不会影响删除行为。
-d --dirs   ：以不递归的方式拷贝目录本身。默认递归时，如果源为"dir1/file1"，则不会拷贝dir1目录，使用该选项将拷贝dir1但不拷贝file1。
--max-size  ：限制rsync传输的最大文件大小。可以使用单位后缀，还可以是一个小数值(例如："--max-size=1.5m")
--min-size  ：限制rsync传输的最小文件大小。这可以用于禁止传输小文件或那些垃圾文件。
--exclude   ：指定排除规则来排除不需要传输的文件。
--delete    ：以SRC为主，对DEST进行同步。多则删之，少则补之。注意"--delete"是在接收端执行的，所以它是在
            ：exclude/include规则生效之后才执行的。
-b --backup ：对目标上已存在的文件做一个备份，备份的文件名后默认使用"~"做后缀。
--backup-dir：指定备份文件的保存路径。不指定时默认和待备份文件保存在同一目录下。
-e          ：指定所要使用的远程shell程序，默认为ssh。
--port      ：连接daemon时使用的端口号，默认为873端口。
--password-file：daemon模式时的密码文件，可以从中读取密码实现非交互式。注意，这不是远程shell认证的密码，而是rsync模块认证的密码。
-W --whole-file：rsync将不再使用增量传输，而是全量传输。在网络带宽高于磁盘带宽时，该选项比增量传输更高效。
--existing  ：要求只更新目标端已存在的文件，目标端还不存在的文件不传输。注意，使用相对路径时如果上层目录不存在也不会传输。
--ignore-existing：要求只更新目标端不存在的文件。和"--existing"结合使用有特殊功能，见下文示例。
--remove-source-files：要求删除源端已经成功传输的文件。
```

## 本地示例
只给出源SRC，会类似`ls -l`形式显示。
```bash
$ rsync .
drwxrwxrwx          4,096 2021/06/28 11:12:08 .
-rwxrwxrwx            102 2021/05/27 15:02:23 for_test.bat
-rwxrwxrwx             52 2021/05/27 14:46:04 test.txt
-rwxrwxrwx             54 2021/06/28 11:12:41 test1.sh

# 如果源SRC是一个文件或目录(未加反斜杠)，显示其信息
$ rsync dira
drwxrwxrwx          4,096 2021/08/18 17:38:55 dira

# 如果目录加了反斜杠，显示其所有内容项的信息
$ rsync dira/
drwxrwxrwx          4,096 2021/08/18 17:38:55 .
-rwxrwxrwx            102 2021/08/18 17:38:55 for_test.bat
-rwxrwxrwx             52 2021/08/18 17:38:55 test.txt
-rwxrwxrwx             54 2021/08/18 17:38:55 test1.sh
```
复制文件，类似`cp`命令。
```bash
# 类似 cp ，复制文件并改名，如果目标地址有同名文件则直接覆盖不提示
$ rsync for_test.bat for1.txt

# 复制文件到目录，目标地址DEST的目录加不加反斜杠都一样
rsync for1.txt dird
rsync for1.txt dird/
```
- `-r`或`--recursive`递归复制。
- `-a`或`--archive`递归传输并保持文件属性。

>复制目录时一定要加上`-r`或`-a`

```bash
# 将目录dira复制到目录dirb
$ rsync -r dira dirb

# 将目录dira的所有内容项复制到目录dirb，但是不包括目录dira本身
$ rsync -r dira/ dirb

# 保持文件属性的递归复制
$ rsync -a dira dirb
$ rsync -a dira/ dirb
```
以源SRC为基准，增量同步两个本地目录时，使用`-t`和`-a`选项保持文件修改时间和属性，以便下次同步时触发增量同步的条件。
```bash
# 增量同步目录dira和darc，并保持属性和修改时间，
$ rsync -t -a dira/ dirc
```
默认情况下，`rsync` 只确保源目录的所有内容（明确排除的文件除外）都复制到目标目录。它不会使两个目录保持相同，并且不会删除文件。如果要使得目标目录成为源目录的镜像副本，则必须使用`--delete`参数，这将删除只存在于目标目录、而不存在于源目录的文件。
```bash
# --delete 参数会使得目标DEST成为源SRC的一个镜像。
$ rsync -t -a --delete dira/ dirb
```
多个源SRC，最后一个目录是目标DEST。
```bash
# 将源SRC的 dira 、dirc 目录连同文件一起同步到目标DEST的 dirb 目录
$ rsync -av dira dirc dirb
```

## 排除和包含规则
- `--exclude=PATTERN`: 指定排除规则来排除不需要传输的文件。可以多次使用。
- `--exclude-from=FILE`: 从文件中指定排除规则来排除不需要传输的文件。
- `--include=PATTERN`: 指定包含规则来包含必须传输的文件。可以多次使用。
- `--include-from=FILE`: 从文件中指定包含规则来包含必须传输的文件。

递归排除所有`.txt`结尾的文件，这也包括子目录以`.txt`结尾的文件。
```bash
$ rsync -v -a --exclude='*.txt' dira/ dirb
```
如果只排除dira目录以`.txt`结尾的文件，可以加`/`表示源SRC的根(root)目录。`rsync`以源SRC为基准作为根目录。
```bash
$ rsync -v -a --exclude='/*.txt' dira/ dirb
```
>注意，`rsync` 会同步以"点"开头的隐藏文件，如果要排除隐藏文件，可以这样写`--exclude=".*"`

多个排除规则，可以使用多个`--exclude`，也可以使用大括号`{}`。
```bash
$ rsync -v -a --exclude='/*.txt' --exclude='/*.sh' dira/ dirb

# 等同于
$ rsync -v -a --exclude={'/*.txt','/*.sh'} dira/ dirb
```
如果`--exclude`和`--delete`选项一起使用，则被排除的文件不会被删除。
```bash
$ rsync -v -a --exclude='/*.txt' --exclude='/*.sh' --delete dira/ dirb
```
如果排除模式很多，可以将它们写入一个文件，每个模式一行，然后用`--exclude-from`参数指定这个文件。
```bash
$ cat rules.txt
/*.txt
/*.bat

$ rsync -v -a --exclude-from='rules.txt' dira/ dirb
```

## 远程同步
远程同步有2种方式：
- 使用 ssh 
- 使用 rsync 守护进程

本地端和远程端必须安装 rsync 。

centos 安装相关软件：
```bash
yum install openssh-clients openssh-server rsync
```

### ssh
与使用`scp`命令的语法结构一致。
```bash
# 本地同步到远程
rsync -avt 本地源   远程用户名@远程地址:远程目的地址

# 远程同步到本地
rsync -avt 远程用户名@远程地址:远程源   本地目的地址
```
示例：
```bash
# 将本地 dira 目录同步到远程的 /root目录
rsync -avt dira root@172.17.0.2:/root

# 将远程的 /root/dira/ 的内容同步到本地相对路径的 dira 目录
rsync -avt root@172.17.0.2:/root/dira/ dira
```
rsync 默认使用 SSH 进行远程登录和数据传输。

由于早期 rsync 不使用 SSH 协议，需要用`-e`参数指定协议，后来策略改变了。所以上面的`-e ssh`可以省略。
```bash
rsync -av -e ssh SRC/ user@remote_host:/DEST
```
但是，如果 ssh 命令有附加的参数，则必须使用`-e`参数指定所要执行的 SSH 命令。
```bash
rsync -av -e 'ssh -p 2234' SRC/ user@remote_host:/DEST
```

### rsync daemon
rsync daemon 需要在一台主机上使用`rsync --daemon`读取配置文件启动rsync的守护进程。默认读取的配置文件为`/etc/rsyncd.conf`，有些版本的系统上可能该文件默认不存在，可以自己创建。

或者也可以使用systemd启动：
```bash
systemctl start rsyncd
```
配置文件`/etc/rsyncd.conf`的常见选项：
```ini
######### 全局配置参数 ##########

# 指定rsync端口。默认873
port = 873    
# 指定rsync daemon 的pid文件
pid file = /var/run/rsyncd.pid
# 指定在每次连接时向客户端显示信息，如站点信息或法律文件信息
motd file = /etc/rsyncd/rsyncd.motd
# 指定监听的IP，会覆盖默认的IP
address = 192.168.2.2 

######### 模块参数，有些模块参数也可用于全局 ###########

# 模块名
[ftp1]
# 客户端在获取模块列表时会显示 comment 定义的注释
comment = 一些关于此模块的描述
# 模块的根目录，这是必须的，其它选项是可选的
path = /data/ftp1
# 如果为true，则会将 path 定义的地址作为根目录，隔绝一些危险隐患，需要root权限
use chroot = true
# 最大同时连接数。默认0表示无限制
max connections = 0
# 用于支持 max connections 参数的文件。
# rsync守护进程对此文件使用记录锁定，以确保共享锁定文件的模块不会超过最大连接限制。
# 默认值为/var/run/rsyncd.lock。
lock file = 
# 日志文件
log file = /var/log/rsyncd.log
# 如果为true，客户端只能下载，不能上传文件。默认 true
read only = false
# 如果为true，客户端只能上传，不能下载文件。默认 false
write only = false
# 客户端请求显示模块列表时，是否显示出来，false表示该模块为隐藏模块。默认true
list = true
# rsync服务的运行用户和用户组，默认是nobody，文件传输成功后属主将是这个uid和gid
uid = nobody
gid = nobody
# 排除选项，以空格分隔的列表
exclude = test1/ test2/
# 指定连接到该模块的用户列表，只有列表里的用户才能连接到模块，用户名和对应密码保存在secrts file中
# 这里使用的不是系统用户，而是虚拟用户。不设置时，默认所有用户都能连接，但使用的是匿名连接
auth users = rsync_backup
# 保存auth users用户列表的用户名和密码，每行包含一个username:passwd。
secrets file = /etc/rsyncd.secrets
# 指定允许连接到该模块的机器，多个ip用空格隔开
hosts allow = 10.0.0.0/24
# 指定不允许连接到该模块的机器，多个ip用空格隔开
hosts deny = 0.0.0.0/32
# 是否忽略执行--delete选项时的io错误
ignore errors
# 是否忽略不可读取的文件
ignore nonreadable
# 是否记录传输文件日志
transfer logging
# 超时设置，单位秒。默认0表示表示无限，匿名用户最好设置600，也就是10分钟
timeout = 600
# 指定哪些文件不用进行压缩传输
dont compress = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2
```
#### 使用步骤示例
1. 创建配置文件`/etc/rsyncd.conf`:

```ini
port = 888
pid file = /var/run/rsyncd.pid

[testa]
path = /root/dirb
comment = 测试testa
log file = /var/log/rsyncd.log
read only = false
list = true
ignore errors
ignore nonreadable
transfer logging
timeout = 600
dont compress = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2
```
2. 启动

```bash
rsync --daemon
```

3. 到客户端主机传输文件

语法：
```bash
Access via rsync daemon:
    Pull:
        rsync [OPTION...] [USER@]HOST::模块名/SRC... [DEST]
        rsync [OPTION...] rsync://[USER@]HOST[:PORT]/模块名/SRC... [DEST]
    Push:
        rsync [OPTION...] SRC... [USER@]HOST::模块名/DEST
        rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/模块名/DEST)
```
示例：此处会出现一些错误，见下面
```bash
# 本地到远程
rsync -avt --port=888 /root/dira root@172.17.0.2::testa

# 或者
rsync -avt dira rsync://root@172.17.0.2:888/testa

# 远程到本地
rsync -avt rsync://root@172.17.0.2:888/testa dira
```
如果配置了身份验证功能, 所以客户端连接时会询问密码。如果不想手动输入密码，则可以使用"`--password-file`"选项提供密码文件，密码文件中只有第一行才是传递的密码，其余所有的行都会被自动忽略。

#### 常见错误
```bash
@ERROR: chroot failed
rsync error: error starting client-server protocol (code 5) at main.c(1661) [sender=3.1.3]
```
服务器端的目录不存在或无权限，创建目录并修正权限可解决问题。
```bash
sending incremental file list
rsync: recv_generator: mkdir "/dira" (in testa) failed: Permission denied (13)
*** Skipping any contents from this failed directory ***
dira/

sent 145 bytes  received 173 bytes  636.00 bytes/sec
total size is 0  speedup is 0.00
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1189) [sender=3.1.3]
```
目标目录的权限问题，修改所属者和所属组，或直接修改权限为`777`.

