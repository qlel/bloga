+++
title = "配置sftp"
description = ""
tags = ['linux', 'ssh', 'sftp']
date =  "2021-04-11T16:49:23+08:00"
lastmod = "2021-04-11T16:49:23+08:00"
+++

sftp采用的是ssh加密隧道，安装性方面较ftp强，而且依赖的是系统自带的ssh服务，不像ftp还需要额外的进行安装。
<!--more-->

## 默认sshd配置的sftp
Linux安装ssh并启动后，`/etc/ssh/sshd_config`默认配置中就已经开启了sftp：
```bash
Subsystem      sftp    /usr/libexec/openssh/sftp-server
```
**此时Linux任何用户都可以使用相应的用户名和密码登陆sftp。**

以上配置使用的是openssh的`sftp-server`服务，也可以配置性能较好的`internal-sftp`：
```bash
Subsystem       sftp    internal-sftp
```
配置改变后，记得重启sshd服务。Linux任何用户也都可以使用相应的用户名和密码登陆sftp。

使用`internal-sftp`带来两个好处：
- 性能更好，在连接sftp的时候，系统不会fork出一个新的sftp进程；
- 可以在ssh的配置文件`/etc/ssh/sshd_config`中，使用ChrootDirectory，Match，ForceCommand等指令来限制登录sftp用户的行为。(以下内容)


## 创建禁止登陆ssh终端但可以登陆sftp的用户
切换到root用户操作：
```bash
$ su - root
```

### 1 给 sftp 创建一个组
```bash
$ groupadd mysftp

$ tail -1 /etc/group
mysftp:x:1001:
```

### 2 创建一个新用户
创建一个新用户，加入到mysftp组中；指定家目录，用于登陆时的家目录，也是之后`ChrootDirectory`指定的目录，会被视为根目录`/`，因为没有`/bin/bash`之类的命令，所以也不能登陆shell。
```bash
# 创建一个不能登陆的用户asftp
# -d 指定家目录
# -m 自动创建家目录
# -g 指定新用户所属组
# -s 指定新用户的登陆终端，这里/bin/false表示不能登陆
$ useradd -d /home/asftp -m -g mysftp -s /bin/false asftp
$ passwd asftp
# 密码设置为了：sftp_2021a
```
### 3 设置家目录所属者和权限
```bash
# 注意此目录如果用于后续的 ChrootDirectory 的活动目录，目录所有者必须是 root
$ chown root:mysftp /home/asftp

# 权限必须为755
$ chmod 755 /home/asftp
```
### 4 在配置的登陆目录创建一个上传目录
因为配置的登陆目录(家目录)的所属者为`root`，所以普通用户没有写权限，也就不能上传文件，所以需要新建一个相应所属用户的目录，用于相应的登陆用户上传文件。

目录所有者为asftp，所有组为mysftp，所有者有写入权限，所有组无写入权限。
```bash
$ mkdir -p /home/asftp/upload
$ chown asftp:mysftp /home/asftp/upload
```
### 5 修改sshd的配置文件
编辑配置文件`/etc/ssh/sshd_config`
```bash
#Subsystem      sftp    /usr/lib/openssh/sftp-server
Subsystem       sftp    internal-sftp

Match Group mysftp
        X11Forwarding no
        AllowTcpForwarding no
        ChrootDirectory %h
        ForceCommand internal-sftp
```
注释掉默认指定的sftp程序，另起一行修改为`internal-sftp`，表示使用内部sftp，性能比较高。
- `Match Group mysftp`
如果用户是 mysftp 组中的一员，那么将应用下面提到的规则到这个条目。
这里是对登录用户的权限限定配置 Match 会对匹配到的用户或用户组起作用且高于 sshd 的通项配置
- `ForceCommand internal-sftp`
强制用户登录会话时使用的初始命令。如果如上配置了此项，则 Match 到的用户只能使用 sftp 协议登录，而无法使用 ssh 登录，会被提示`This service allows sftp connections only.`
- `ChrootDirectory %h`
用户的可活动目录(登陆目录)，可以用 `%h` 标识用户家目录; `%u` 代表用户名。 当 Match 匹配的用户登录后，会话的根目录会切换至此目录。
这里要尤其注意两个问题：
  - chroot 路径上的所有目录，所有者必须是 root，权限最大为 0755，这一点必须要注意而且符合。所以如果以非 root 用户登录时，我们需要在 chroot 下新建一个登录用户有权限操作的目录，这也是第4步创建upload目录的原因。
  - chroot 一旦设定，则相应的用户登录时会话的根目录 "`/`" 切换为此目录，如果你此时使用 ssh 而非 sftp 协议登录，则很有可能会被提示：`/bin/bash: No such file or directory`
  这则提示非常的正确，对于此时登录的用户，会话中的根目录 "`/`" 已经切换为你所设置的 chroot 目录，除非你的 chroot 就是系统的 "/" 目录，否则此时的 `chroot/bin` 下是不会有 `bash` 命令的，这就类似添加用户时设定的 `-s /bin/false` 参数，shell 的初始命令式 `/bin/false` 自然就无法远程 ssh 登录了。
  所以注意不要将正常可以使用ssh登陆的用户加入到匹配的组`mysftp`，会导致正常用户不能用ssh登陆。


### 6 重启ssh服务
```bash
$ systemctl restart ssh.service

# 查看状态
$ systemctl status ssh.service
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2021-04-10 16:18:02 CST; 4h 55min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
  Process: 2460 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
 Main PID: 2461 (sshd)
    Tasks: 1 (limit: 1147)
   Memory: 2.4M
   CGroup: /system.slice/ssh.service
           └─2461 /usr/sbin/sshd -D
```
>再次注意：!!
1. chroot 可能带来的问题，因为 chroot 会将会话的根目录切换至此，所以 ssh 登录很可能会提示 `/bin/bash: No such file or directory` 的错误，因为此会话的路径会为 `chroot/bin/bash`
2. `ForceCommand` 为会话开始时的初始命令 如果指定了比如 `internal-sftp`，则会提示 `This service allows sftp connections only.` 这就如同 `usermod -s /bin/false` 命令一样，用户登录会话时无法调用 `/bin/bash` 命令，自然无法 ssh 登录服务器。
### 7 测试sftp
登录到 sftp 服务器的同一个网络上的任何其它主机上进行连接测试。
```bash
$ sftp -P 2222 asftp@192.168.56.1
asftp@192.168.56.1's password:
Connected to asftp@192.168.56.1.
sftp> ls -l
drwxr-xr-x    2 1001     1001         4096 Apr 10 08:16 upload
sftp> put ./rd.txt /upload
Uploading ./rd.txt to /upload/rd.txt
./rd.txt                                     100%  957   478.6KB/s   00:00
```
- `ls [directory]`：列出一个远程目录的内容。如果没有指定目标目录，则默认列出当前目录。
- `lls [directory]`：列出一个本地目录的内容。如果没有指定目标目录，则默认列出当前目录。
- `pwd`：查看远程当前目录
- `lpwd`：查看本地当前目录
- `cd directory`：从当前目录改到指定目录。
- `mkdir directory`：创建一个远程目录。
- `rmdir path`：删除一个远程目录。
- `put localfile [remotefile]`：本地文件传输到远程主机。
- `get remotefile [localfile]`：远程文件传输到本地。
- `help`：显示帮助信息。
- `bye`：退出 sftp。
- `quit`：退出 sftp。
- `exit`：退出 sftp。

由于之前`qlel`用户配置了ssh公钥连接远程主机， 所以可以直接连接。
```bash
$ sftp -P 2222 qlel@192.168.56.1
Connected to qlel@192.168.56.1.
sftp>
```

也可以用ftp客户端连接，比如`FileZilla`客户端：
![sftp连接.png](/配置sftp/sftp连接.png)