+++
title = "Linux挂载硬盘"
date = 2021-07-17T09:47:47+08:00
description = "linux挂载硬盘和lvm相关了解。"
tags = ['linux', 'lvm']
aplayer = false
showToc = true
math = false
showLicense = false
+++
linux挂载硬盘和lvm相关了解。
<!--more-->
## 概述
在Linux中，所有的设备都是以文件的形式表现，硬盘也不例外，硬盘设备的文件名格式一般为 `/dev/xxyn`： 
- `xx` 标明硬盘的类型，如：`sd`:表示硬盘接口是 SCSI 或 SATA 或 USB ；`hd`:表示硬盘接口是 IDE 。
- `y`(范围：`a-z`)标明了硬盘是使用此类接口第几个硬盘，如：`/dev/hdd`表示这是第四个使用IDE接口的硬盘。
- `n`表示分区，用数字表示

如：`/dev/hda3`表示在第一个IDE接口硬盘上的第三个主分区或扩展分区。

>**主分区**
>>也称为主磁盘分区，是一种分区类型。主分区中不能再划分其他类型的分区，因此每个主分区都相当于一个逻辑磁盘（在这一点上主分区和逻辑分区很相似，但主分区是直接在硬盘上划分的，逻辑分区则必须建立于扩展分区中）。

>**扩展分区**
>>是硬盘磁盘分区的一种。严格地讲它不是一个实际意义的分区，它仅仅是一个指向下一个分区的指针，这种指针结构将形成一个单向链表。这样在主引导扇区中除了主分区外，仅需要存储一个被称为扩展分区的分区数据，通过这个扩展分区的数据可以找到下一个分区。扩展分区不能直接存储数据，扩展分区的作用是保存逻辑分区。

>**逻辑分区**
是硬盘上一块连续的区域，不同之处在于，每个主分区只能分成一个驱动器，每个主分区都有各自独立的引导块，可以用`fdisk`设定为启动区。一个硬盘上最多可以有4个主分区，而扩展分区上可以划分出多个逻辑驱动器。这些逻辑驱动器没有独立的引导块，不能用`fdisk`设定为启动区。主分区和扩展分区都是dos分区。

这些分区的区别：
- 地位不同
逻辑分区属于扩展分区，扩展分区属于主分区。
给新硬盘上建立分区时都要遵循以下的顺序：
建立主分区→建立扩展分区→建立逻辑分区→激活主分区→格式化所有分区。 
- 位置不同
主分区又叫做引导分区，老的MBR分区最多只能创建四个，现在基本采用的GPT分区几乎无限制。
扩展分区是主分区之外的部分。
逻辑分区在扩展分区之内可以创建无数个。
![分区步骤.webp](/Linux挂载硬盘/分区步骤.webp)
- 作用不同
主分区是独立的，对应磁盘上的第一个分区，“一般”就是C盘。
扩展分区是一个概念，实际上是看不到的。
逻辑分区相当于一块存储介质，和操作系统还有别的逻辑分区、主分区没有什么关系，是“独立的”。
- 格式化情况不同
格式化是针对主分区和逻辑分区的。要格式化是因为这和操作系统管理文件系统有关系。没有格式化的分区就像一张白纸，要写入数据，必须对白纸打上“格子”，每个格子里面写一块。而操作系统只认这些格子。
- 大小不同
我们假定扩展分区为字母`X`，用一个公式来总结它们之间的关系：
硬盘的容量＝主分区的容量＋扩展分区的容量（硬盘=C盘+`X`）
扩展分区的容量＝各个逻辑分区的容量之和（`X`=D盘+E盘+F盘）

## 查看硬盘信息
- `fdisk -l`
可查看详细信息，DiskLabel type是`dos`，表示是MBR类型分区；如果是`gpt`，就是GPT类型分区。

![fdisk_l.png](/Linux挂载硬盘/fdisk_l.png)

- `lsblk`或者`lsblk -f`
列出块设备信息

![lsblk.png](/Linux挂载硬盘/lsblk.png)

## 硬盘分区
使用`fdisk`命令进行分区，相关菜单选项：
- `m` ：显示菜单和帮助信息
- `a` ：活动分区标记/引导分区
- `d` ：删除分区
- `l` ：显示分区类型
- `n` ：新建分区
- `p` ：显示分区信息
- `q` ：退出不保存
- `t` ：指定分区号进行设置
- `v` ：进行分区检查
- `w` ：保存修改
- `x` ：扩展应用，高级功能

如:对`/dev/sdb`进行分区，分区后重启可看到分区：
```bash
[root@localhost /]# fdisk /dev/sdb              
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x6511fa23.

Command (m for help): p                   \\查看已有分区

Disk /dev/sdb: 240.1 GB, 240057409536 bytes, 468862128 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x6511fa23

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n                    \\创建分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p                   \\p表示主分区，e表示扩展分区
Partition number (1-4, default 1):  \\ 分区号
First sector (2048-468862127, default 2048): \\这里输入分区大小
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-468862127, default 468862127): 
Using default value 468862127
Partition 1 of type Linux and of size 223.6 GiB is set

Command (m for help): p                     \\查看确认分区
 
Disk /dev/sdb: 240.1 GB, 240057409536 bytes, 468862128 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x6511fa23

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   468862127   234430040   83  Linux

Command (m for help): w                    \\保存分区
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
## 分区格式化
- `mkfs`
一般用于格式化分区
    - `-t`：指定文件类型，如ext2, ext4等
- `mkswap`
设置交换分区

```bash
$ mkfs -t ext4 /dev/sda6 # 将该分区格式化成ext4文件系统
$ mkswap /dev/sda7 # 设置交换区
```

## 挂载和卸载分区
Linux的文件系统都是树形结构，所有的文件系统结合起来就形成了一个大的目录树，这个目录树的根就是根目录`/`，根分区在开机时就自动挂载到根目录上了，划分好并格式化的分区也要挂载到这个目录树的某个目录上，才能被我们使用，否则这个分区就没有访问的入口，而这个目录就被称为该分区的挂载点。
- 挂载分区
    - `mount 分区名 挂载点`
        - `-t` # 分区文件系统类型
        - `-l` # 列出所挂载的系统
        - `-o` # 指定挂载后的行为
- 卸载分区
    - `umount 分区名|挂载点`

```bash
mount -t ext4 /dev/sdb1 /project/music
```

## 开机自动挂载分区
为了保持挂载的持久性，需要修改配置启动文件`/etc/fstab`。

`/etc/fstab`文件中有有3种格式可用于挂载：
```bash
# <file system>       <mount point>  <type>  <options>  <dump>  <pass>
/dev/sdb1       /project/music        ext4   defaults     0       0 # 第一种方式
---------------------
e2label /dev/sda5 lvaa # 为硬盘设备加标签为lvaa
LABEL=lvaa      /project/vedio        ext4   defaults     0       0 # 第二种方式
-----------------------------------------
blkid # 查看硬盘设备UUID
UUID=设备UUID    /project/share       ext4   defaults      0       0 # 第三种方式 推荐
```
![fstab.png](/Linux挂载硬盘/fstab.png)
- `e2label` 设置分区标签名
    - `e2label 分区设备名 标签名`
- `blkid` 查看块设备的文件系统类型、LABEL、UUID、挂载目录等信息。
- `<file system>`
指定了要挂载的设备；有三种方式指定：
    1. /dev/sda6
    2. LABEL=某个设备标签 
    3. UUID=某个设备的UUID(推荐)
- `<mount point>`
挂载点
- `<type>`
挂载分区文件系统类型
- `<options>`
挂载参数，指定文件系统挂载后的一下行为属性
- `<dump>`
该选项被`dump`命令使用来检查一个文件系统是否应该进行`dump`备份
    - 若不需要则设置为`0`
    - 每天备份设置为`1`
    - 不定期备份设置为`2`
- `<pass>`
开机检查分区的次序，该选项被命令`fstab`用来确定系统开机进行文件系统检查时的顺序，`1`为优先，`2`为次优先，如果为`0`或者没有设置，开机将跳过检查。

## 什么是LVM
LVM是逻辑盘卷管理（Logical Volume Manager）的简称，它是Linux环境下对磁盘分区进行管理的一种机制，LVM是建立在硬盘和分区之上的一个逻辑层，来提高磁盘分区管理的灵活性。

在LVM中，有几个概念需要了解熟悉：
- `PV` (Physical Volume) 
**物理卷**就是实际对应的物理存储设备。传统的存储管理方式是将物理设备格式化后，就直接挂载到一个目录上。在LVM中，`PV`是一个基础物理单位，也是存储池的基本组成部分。存储分区要成为`PV`，需要调用LVM命令方法，在分区中记录相应的元数据信息。
- `VG` (Volume Group)
**卷组**类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。
- `LV` (Logical Volume)
**逻辑卷**建立在卷组基础上，卷组中未分配空间可用于建立新的逻辑卷，逻辑卷建立后可以动态扩展和缩小空间。
- `PE` (Physical Extent)
**物理块**是物理卷中可用于分配的最小存储单元，物理块大小在建立卷组时指定，一旦确定不能更改，同一卷组所有物理卷的物理块大小需一致，新的pv加入到vg后，pe的大小自动更改为vg中定义的pe大小。
- `LE` (Logical Extent)
**逻辑块**是逻辑卷中可用于分配的最小存储单元，逻辑块的大小取决于逻辑卷所在卷组中的物理区域的大小。由于受内核限制的原因，一个逻辑卷（Logic Volume）最多只能包含65536个PE（Physical Extent），所以一个PE的大小就决定了逻辑卷的最大容量，4 MB(默认) 的PE决定了单个逻辑卷最大容量为 256 GB，若希望使用大于256G的逻辑卷，则创建卷组时需要指定更大的PE。在Red Hat Enterprise Linux AS 4中PE大小范围为8 KB 到 16GB，并且必须总是 2 的倍数。

![lvm结构.jpg](/Linux挂载硬盘/lvm结构.jpg)

## 查看PV/VG/LV信息
- `pvs` 查看物理卷的信息
- `pvdisplay` 查看物理卷的详细信息
- `vgs` 查看卷组的信息
- `vgdisplay` 查看卷组的详细信息
- `lvs` 查看逻辑卷的信息
- `lvdisplay` 查看逻辑卷的详细信息

查看基本信息：
```bash
root@debianqlel:~# pvs
  PV         VG            Fmt  Attr PSize  PFree
  /dev/sda5  debianqlel-vg lvm2 a--  <7.52g    0
root@debianqlel:~# vgs
  VG            #PV #LV #SN Attr   VSize  VFree
  debianqlel-vg   1   2   0 wz--n- <7.52g    0
root@debianqlel:~# lvs
  LV     VG            Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   debianqlel-vg -wi-ao----   6.56g
  swap_1 debianqlel-vg -wi-ao---- 980.00m
```
查看详细信息：
```bash
root@debianqlel:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               debianqlel-vg
  PV Size               7.52 GiB / not usable 2.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              1925
  Free PE               0
  Allocated PE          1925
  PV UUID               6jP54b-JyR4-0Lqb-pzcN-WxSC-ijkU-3hKTQM

root@debianqlel:~# vgdisplay
  --- Volume group ---
  VG Name               debianqlel-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <7.52 GiB
  PE Size               4.00 MiB
  Total PE              1925
  Alloc PE / Size       1925 / <7.52 GiB
  Free  PE / Size       0 / 0
  VG UUID               fHeCqb-8A2x-Ec3E-JYGS-tAxY-M9Eg-f6JmF3

root@debianqlel:~# lvdisplay
  --- Logical volume ---
  LV Path                /dev/debianqlel-vg/root
  LV Name                root
  VG Name                debianqlel-vg
  LV UUID                buL332-eNms-0ZeH-QYsO-7Qe2-iBIn-WPRVro
  LV Write Access        read/write
  LV Creation host, time debianqlel, 2021-03-03 20:41:23 +0800
  LV Status              available
  # open                 1
  LV Size                6.56 GiB
  Current LE             1680
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:0

  --- Logical volume ---
  LV Path                /dev/debianqlel-vg/swap_1
  LV Name                swap_1
  VG Name                debianqlel-vg
  LV UUID                fy0aqG-tmRI-rsOQ-2Hd0-KWJ6-J2lr-wfH50E
  LV Write Access        read/write
  LV Creation host, time debianqlel, 2021-03-03 20:41:23 +0800
  LV Status              available
  # open                 2
  LV Size                980.00 MiB
  Current LE             245
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:1
```

## 将各物理磁盘或分区的系统类型设为Linux LVM
Linux LVM的system ID为`8e`，通过`fdisk`命令中的`t`选项设置。
```bash
[root@localhost ~]# fdisk /dev/sdb 
...
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 2
First sector (20973568-62914559, default 20973568): 
Using default value 20973568
Last sector, +sectors or +size{K,M,G} (20973568-62914559, default 62914559): +5G
...
Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 8e  # 指定system id为8e
Changed type of partition 'Linux' to 'Linux LVM'
...
Command (m for help): p
...
/dev/sdb1            2048    20973567    10485760   8e  Linux LVM
/dev/sdb2        20973568    31459327     5242880   8e  Linux LVM
Command (m for help): w
...
```
## 将各物理磁盘或分区初始化为PV
这一阶段可使用的命令有：
- `pvcreate` 创建物理卷
- `pvremove` 将物理卷信息删除，使其不再被视为一个物理卷
- `pvscan` 扫描当前系统上的所有物理卷
- `pvs` 查看物理卷的信息
- `pvdisplay` 查看物理卷的详细信息

### pvcreate
创建物理卷。
```bash
用法：pvcreate [option] DEVICE

选项：

    -f：强制创建逻辑卷，不需用户确认

    -u：指定设备的UUID

    -y：所有问题都回答yes

例 pvcreate /dev/sdb1 /dev/sdb2
```

### pvscan
扫描当前系统上的所有物理卷
```bash
用法：pvscan [option]

  选项：

      -e：仅显示属于输出卷组的物理卷

      -n：仅显示不属于任何卷组的物理卷

      -u：显示UUID
```

### pvremove
将物理卷信息删除，使其不再被视为一个物理卷
```bash
用法：pvremove [option] PV_DEVICE

选项：

    -f：强制删除

    -y：所有问题都回答yes

例 pvremove /dev/sdb1
```

### 示例
```bash
[root@localhost ~]# pvcreate /dev/sdb{1,2}  # 将两个分区初始化为物理卷
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdb2" successfully created.
[root@localhost ~]# pvscan 
  PV /dev/sdb2                      lvm2 [5.00 GiB]
  PV /dev/sdb1                      lvm2 [10.00 GiB]
  Total: 2 [15.00 GiB] / in use: 0 [0   ] / in no VG: 2 [15.00 GiB]
[root@localhost ~]# pvdisplay /dev/sdb1   # 显示物理卷sdb1的详细信息
  "/dev/sdb1" is a new physical volume of "10.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name               
  PV Size               10.00 GiB
  Allocatable           NO
  PE Size               0   # 由于PE是在VG阶段才划分的，所以此处看到的都是0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               GrP9Gi-ubau-UAcb-za3B-vSc3-er2Q-MVt9OO
   
[root@localhost ~]# pvremove /dev/sdb2   # 删除sdb2的物理卷信息
  Labels on physical volume "/dev/sdb2" successfully wiped.
[root@localhost ~]# pvscan     # 可以看到PV列表中已无sdb2
  PV /dev/sdb1                      lvm2 [10.00 GiB]
  Total: 1 [10.00 GiB] / in use: 0 [0   ] / in no VG: 1 [10.00 GiB]
[root@localhost ~]# pvcreate /dev/sdb2 
  Physical volume "/dev/sdb2" successfully created.
```

## 创建卷组VG
`VG`（volume group，卷组）。卷组将多个物理卷整合起来（屏蔽了底层细节），并划分`PE`（physical extend）。

`PE`是物理卷中的最小存储单元，有点类似于文件系统中的block，`PE`大小可指定，默认为`4M`。

这一阶段用到的命令有:
- `vgcreate` 创建卷组
- `vgscan` 查找系统中存在的LVM卷组，并显示找到的卷组列表
- `vgextend` 动态扩展LVM卷组，通过向卷组中添加物理卷来增加卷组的容量
- `vgreduce` 通过删除LVM卷组中的物理卷来减少卷组容量，不能删除LVM卷组中剩余的最后一个物理卷
- `vgremove` 删除卷组，其上的逻辑卷必须处于离线状态
- `vgchange` 改变卷组属性。常用来设置卷组的活动状态
- `vgs` 查看卷组的信息
- `vgdisplay` 查看卷组的详细信息

### vgcreate
创建卷组
```bash
用法：vgcreate [option] VG_NAME PV_DEVICE ...

选项：

    -s：卷组中的物理卷的PE大小，默认为4M

    -l：卷组上允许创建的最大逻辑卷数

    -p：卷级中允许添加的最大物理卷数

例 vgcreate -s 8M myvg /dev/sdb1 /dev/sdb2
```

### vgdisplay
显示卷组详细信息属性
```bash
用法：vgdisplay [option] [VG_NAME]

选项：

    -A：仅显示活动卷组的信息

    -s：使用短格式输出信息
```

### vgextend
动态扩展LVM卷组，它通过向卷组中添加物理卷来增加卷组的容量
```bash
用法：vgextend VG_NAME PV_DEVICE ...

例 vgextend myvg /dev/sdb3
```

### vgreduce
通过删除LVM卷组中的物理卷来减少卷组容量，不能删除LVM卷组中剩余的最后一个物理卷
```bash
用法：
vgreduce VG PV ...

从VG中删除所有未使用的PV:
vgreduce -a|--all VG_NAME 

从VG中删除所有丢失的PV:
vgreduce --removemissing VG_NAME 
```

### vgremove
删除卷组，其上的逻辑卷必须处于离线状态
```bash
用法：vgremove [-f] VG_NAME

-f：强制删除
```

### vgchange
改变卷组属性。常用来设置卷组的活动状态。
```bash
用法：vgchange -a n/y VG_NAME

-a n为休眠状态，休眠之前要先确保其上的逻辑卷都离线；

-a y为活动状态
```

### 示例
```bash
# 创建一个卷组myvg
[root@localhost ~]# vgcreate -s 8M myvg /dev/sdb{1,2}
  Volume group "myvg" successfully created
[root@localhost ~]# vgscan
  Reading volume groups from cache.
  Found volume group "myvg" using metadata type lvm2
[root@localhost ~]# vgdisplay
  --- Volume group ---
  VG Name               myvg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               14.98 GiB
  PE Size               8.00 MiB
  Total PE              1918
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1918 / 14.98 GiB
  VG UUID               aM3RND-aUbQ-7RjC-dCci-JiS4-Oj2Z-wv9poA
```

## 在卷组上创建LV
为了便于管理，逻辑卷对应的设备文件保存在卷组目录下，为`/dev/VG_NAME/LV_NAME`。LV中可以分配的最小存储单元称为`LE`（logical extend），在同一个卷组中，`LE`的大小和`PE`是一样的，且一一对应。

这一阶段用到的命令有：
- `lvcreate` 创建逻辑卷或快照
- `lvscan` 扫描当前系统中的所有逻辑卷，及其对应的设备文件
- `lvextend` 在线扩展逻辑卷空间
- `lvreduce` 缩减逻辑卷空间，一般离线使用
- `lvresize` 调整逻辑卷的大小
- `lvremove` 删除逻辑卷，需要处于离线（卸载）状态
- `lvs` 查看逻辑卷的信息
- `lvdisplay` 查看逻辑卷的详细信息

### lvcreate
创建逻辑卷或快照。

使用该命令创建逻辑卷时当然必须指明卷组，创建快照时必须指明针对哪个逻辑卷。

选项：
- `-L` 指定大小
- `-l` 指定大小(LE数量)，可用百分比
- `-n` 指定名称
- `-s` 创建快照
- `-p r` 设置为只读（该选项一般用于创建快照中）

```bash
# 在名为vg_newlvm的卷组中创建15G大小的逻辑卷
$ lvcreate -L 15G vg_newlvm

# 在名为vgnewlvm的卷组中创建大小为2500MB的逻辑卷，并命名为centos7newvol
# 这样就创建了块设备/dev/vgnewlvm/centos7newvol
$ lvcreate -L 2500 -n centos7_newvol vg_newlvm

# 使用 -l 参数时可以使用百分比
$ lvcreate -l 50%VG -n centos7_newvol vg_newlvm

# 使用卷组剩下的所有空间创建逻辑卷
$ lvcreate --name centos7newvol -l 100%FREE vgnewlvm
```

### lvdisplay
显示逻辑卷属性

用法：`lvdisplay [/dev/VG_NAME/LV_NAME]`

### lvextend
在线扩展逻辑卷空间。

```bash
用法：lvextend -L|-l 扩展的大小 /dev/VG_NAME/LV_NAME  

选项：

    -L：指定扩展（后）的大小。例如，-L +800M表示扩大800M，而-L 800M表示扩大至800M

    -l：指定扩展（后）的大小（LE数）

例 lvextend -L 200M /dev/myvg/mylv
```

### lvreduce
缩减逻辑卷空间，一般离线使用。

```bash
用法：lvexreduce -L/-l 缩减的大小 /dev/VG_NAME/LV_NAME  

选项：

    -L：指定缩减（后）的大小。例如，-L -800M表示缩减800M，而-L 800M表示缩减至800M

    -l：指定缩减（后）的大小（LE数）

例 lvreduce -L 200M /dev/myvg/mylv
```

### lvremove
删除逻辑卷，需要处于离线（卸载）状态。

```bash
用法：lvremove [-f] /dev/VG_NAME/LV_NAME

-f：强制删除
```

### 示例
```bash
[root@localhost ~]# lvcreate -L 2G -n mylv myvg  
  Logical volume "mylv" created.
[root@localhost ~]# lvscan 
  ACTIVE            '/dev/myvg/mylv' [2.00 GiB] inherit
[root@localhost ~]# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/myvg/mylv
  LV Name                mylv
  VG Name                myvg
  LV UUID                2lfCLR-UEhm-HMiT-ZJil-3EJm-n2H3-ONLaz1
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2019-07-05 13:42:44 +0800
  LV Status              available
  # open                 0
  LV Size                2.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

## 格式化逻辑卷并挂载
和以上格式化和挂载分区一致。

## LV逻辑卷扩容后，必须对挂载目录在线扩容
- `resize2fs` 针对文件系统ext2 ext3 ext4
- `xfs_growfs` 针对文件系统 xfs

示例：
```bash
# xfs在线扩容
$ xfs_growfs /dev/mapper/vg--BHG-lv01
meta-data=/dev/mapper/vg--BHG-lv01 isize=512    agcount=4, agsize=32000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=128000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=855, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 128000 to 256000

# ext4在线扩容
$ resize2fs /dev/mapper/vg--BHG-lv02
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/mapper/vg--BHG-lv02 is mounted on /BHGPOS-data; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/mapper/vg--BHG-lv02 is now 5242880 blocks long.
```