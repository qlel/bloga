+++
title = "配置sftp"
description = ""
tags = ['linux', 'ssh', 'sftp']
date =  "2021-04-11T16:49:23+08:00"
lastmod = "2021-04-11T16:49:23+08:00"
+++

sftp采用的是ssh加密隧道，安装性方面较ftp强，而且依赖的是系统自带的ssh服务，不像ftp还需要额外的进行安装
<!--more-->

切换到root用户执行：
```bash
$ su - root
```

## 1 给 sftp 创建一个组
```bash
$ groupadd mysftp

$ tail -1 /etc/group
mysftp:x:1001:
```

## 2 创建一个新用户加入到此组中
也可以指定现有用户到此组中。
```bash
# 创建一个不能登陆的用户asftp
$ useradd -g mysftp -s /bin/false asftp
$ passwd asftp
# 密码设置为了：asftp_2021
```
## 3 指定新用户的home目录和设置权限
感觉此步骤可有可无。
```bash
$ mkdir -p /home/asftp
$ usermod -d /home/asftp asftp
$ chown root:mysftp /home/asftp
$ chmod 755 /home/asftp
```
## 4 在用户的home目录中创建一个上传目录
目录所有者为asftp，所有组为mysftp，所有者有写入权限，所有组无写入权限。
```bash
$ mkdir -p /home/asftp/upload
$ chown asftp:mysftp /home/asftp/upload
$ chmod 755 /home/asftp/upload
```
## 5 修改sshd的配置文件
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
- `ChrootDirectory %h`
意味着用户只能在他们自己各自的家目录中更改目录，而不能超出他们各自的家目录。
- `ForceCommand internal-sftp`
用户被限制到只能使用内部 sftp 命令。

## 6 重启ssh服务
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
## 7 测试sftp
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