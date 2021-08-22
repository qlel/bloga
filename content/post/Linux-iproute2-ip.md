+++
title = "Linux Iproute2 Ip"
description = ""
tags = ['linux', 'shell', 'iproute2']
date =  "2021-04-08T17:00:27+08:00"
lastmod = "2021-04-08T17:00:27+08:00"
+++

iproute2是Linux网络工具包，它代替了net-tools（ifconfig，vconfig，route，arp等）。ip是iproute2中的一个命令之一。
<!--more-->

## ip选项
```bash
$ ip -h
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { link | address | addrlabel | route | rule | neigh | ntable |
                   tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm |
                   netns | l2tp | fou | macsec | tcp_metrics | token | netconf | ila |
                   vrf | sr }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
                    -h[uman-readable] | -iec | -j[son] | -p[retty] |
                    -f[amily] { inet | inet6 | ipx | dnet | mpls | bridge | link } |
                    -4 | -6 | -I | -D | -M | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } | -br[ief] |
                    -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] [filename] |
                    -rc[vbuf] [size] | -n[etns] name | -a[ll] | -c[olor]}
```
提供以下输出选项:
- `-o`,`--oneline`
用反斜杠替换每个换行符。
- `-br`,`--brief`
产生简洁的，面向机器的输出。 非常适合用awk/cut进行解剖
- `-j`,`--json`
json输出, 可配合`--pretty`|`-p`和`--brief`选项

```bash
# 默认输出
$ ip addr show lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

# 用反斜杠替换每个换行符
$ ip -o addr show lo
1: lo    inet 127.0.0.1/8 scope host lo\       valid_lft forever preferred_lft forever
1: lo    inet6 ::1/128 scope host \       valid_lft forever preferred_lft forever

# 产生简洁的，面向机器的输出
$ ip -br addr show lo
lo               UNKNOWN        127.0.0.1/8 ::1/128

# json输出
$ ip -j -p addr show lo
[ {
        "ifindex": 1,
        "ifname": "lo",
        "flags": [ "LOOPBACK","UP","LOWER_UP" ],
        "mtu": 65536,
        "qdisc": "noqueue",
        "operstate": "UNKNOWN",
        "group": "default",
        "txqlen": 1000,
        "link_type": "loopback",
        "address": "00:00:00:00:00:00",
        "broadcast": "00:00:00:00:00:00",
        "addr_info": [ {
                "family": "inet",
                "local": "127.0.0.1",
                "prefixlen": 8,
                "scope": "host",
                "label": "lo",
                "valid_life_time": 4294967295,
                "preferred_life_time": 4294967295
            },{
                "family": "inet6",
                "local": "::1",
                "prefixlen": 128,
                "scope": "host",
                "valid_life_time": 4294967295,
                "preferred_life_time": 4294967295
            } ]
    },{} ]
```
其它选项:
- `-V, -Version`
显示版本信息
- `-h, -human, -human-readable`
输出带有人类可读值和后缀的统计信息
- `-b, -batch <FILENAME>`
从提供的文件或标准输入中读取命令并调用它们。第一次失败将导致ip命令终止。
- `-force`
不以批处理方式终止执行ip命令时的错误. 如果执行命令期间出现错误将返回非0.
- `-s, -stats, -statistics`
输出更多信息。如果该选项出现两次或更多次，则信息量会增加。通常，这些信息是统计信息或一些时间值。
例如: 显示enp0s3网卡的流量
```bash
$ ip -s -h link show dev enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast
    3.97M      50.5k    0       0       0       0
    TX: bytes  packets  errors  dropped carrier collsns
    4.31M      33.5k    0       0       0       0

# 显示更多信息
$ ip -s -s -h link show dev enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast
    3.96M      50.5k    0       0       0       0
    RX errors: length   crc     frame   fifo    missed
               0        0       0       0       0
    TX: bytes  packets  errors  dropped carrier collsns
    4.30M      33.4k    0       0       0       0
    TX errors: aborted  fifo   window heartbeat transns
               0        0       0       0       30
```
- `-d, -details`
输出更多详细信息
- `-l, -loops <COUNT>`
指定`ip addr flush`命令逻辑放弃前将尝试的最大循环数。默认值为10。零（0）表示循环，直到删除所有地址。
- `-f, -family <FAMILY>`
指定要使用的协议族。 协议族标识符可以是inet，inet6，bridge，mpls或link中的一种。 如果不存在此选项，则从其他参数中猜测协议族.
- `-4`
`-family inet`的简写
- `-6`
`-family inet6`的简写
- `-4`
`-family inet`的简写
- `-B`
`-family bridge`的简写
- `-M`
`-family mpls`的简写
- `-0`
`-family link`的简写
- `-r, -resolve`
使用系统的名称解析器来打印DNS名称而不是主机地址
- `-N, -Numeric`
直接打印协议，范围，dsfield等的编号，而不是将其转换为人类可读的名称。
- `-a, -all`
在所有对象上执行指定的命令，这取决于命令是否支持此选项。
- `-c[color][={always|auto|never}`
配置颜色输出。 如果省略参数或设置参数为always，则不管标准输出状态如何，都启用颜色输出。 如果参数为auto，则在启用颜色输出之前，将stdout检查为终端。 如果参数是never，则禁用彩色输出。 如果多次指定，则最后一个优先。 如果还给出了`-json`，则忽略此标志。
使用的调色板会受到COLORFGBG环境变量的影响（请参阅环境）。
- `-t, -timestamp`
使用监视器monitor选项时显示当前时间
- `-ts, -tshort`
与`-timestamp`类似，但使用较短的格式
- `-rc, -rcvbuf<SIZE>`
设置netlink套接字接收缓冲区的大小，默认为1MB。
- `-iec`
以IEC单位打印可读速率（例如1Ki=1024）。

## 地址管理
显示所有地址:
```bash
$ ip address show

# 缩写
$ ip addr show
```
显示指定接口的地址:
```bash
# 格式
$ ip address show ${interface name} 

# 示例
$ ip addr show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 65793sec preferred_lft 65793sec
    inet6 fe80::a00:27ff:fe9b:e4fb/64 scope link
       valid_lft forever preferred_lft forever
```
显示正在运行的接口地址:
```bash
$ ip address show up
```
显示静态或动态的接口地址:
```bash
# 格式, 静态地址
$ ip address show [dev ${interface}] permanent

# 格式, 动态地址
ip address show [dev ${interface}] dynamic

# 示例
# 显示所有动态地址
$ ip addr show dynamic
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 65549sec preferred_lft 65549sec

# 显示 enp0s3 动态地址
$ ip addr show dev enp0s3 dynamic
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 65515sec preferred_lft 65515sec
```
添加一个地址到接口:
```bash
# 格式
$ ip address add ${address}/${mask} dev ${interface name}

# 示例
# 添加一个ipv4地址到enp0s3
$ sudo ip address add 192.0.2.10/27 dev enp0s3
$ ip addr show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63847sec preferred_lft 63847sec
    inet 192.0.2.10/27 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe9b:e4fb/64 scope link
       valid_lft forever preferred_lft forever

# 也可以添加ipv6地址
$ sudo ip address add 2001:db8:1::/48 dev tun10
```
>可以添加多个地址.
>如果添加多个地址，则计算机将接受所有这些地址的数据包。添加的第一个地址将成为“主要地址”。默认情况下，接口的主地址用作传出数据包的源地址。设置的所有其他地址将成为辅助地址。

添加地址时还可以添加描述: 
```bash
# 格式, 只能在添加地址时才能添加描述
$ ip address add ${address}/${mask} dev ${interface name} label ${interface name}:${description} 

# 示例
$ sudo ip address add 192.0.2.11/27 dev enp0s3 label enp0s3:描述
$ ip addr show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63403sec preferred_lft 63403sec
    inet 192.0.2.10/27 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet 192.0.2.11/27 scope global secondary enp0s3:描述
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe9b:e4fb/64 scope link
       valid_lft forever preferred_lft forever
```
从接口删除地址:
```bash
# 格式, delete 可以缩写为 del
$ ip address delete ${address}/${mask} dev ${interface name}

# 示例
# 注意, 似乎将辅助地址都删除了??
$ sudo ip addr del 192.0.2.10/27 dev enp0s3
$ ip addr show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63025sec preferred_lft 63025sec
    inet6 fe80::a00:27ff:fe9b:e4fb/64 scope link
       valid_lft forever preferred_lft forever
```
清空一个接口的所有地址:
```bash
# 格式
ip address flush dev ${interface name}

# 示例
# 清空enp0s3的所有地址
$ sudo ip addr flush dev enp0s3

# 清空enp0s3的所有ipv4地址
$ sudo ip -4 addr flush dev enp0s3

# 清空enp0s3的所有ipv6地址
$ sudo ip -6 addr flush dev enp0s3
```
## 邻居(arp和ndp)表管理
arp: ipv4的地址解析协议
ndp: 包含arp的地址解析协议

```bash
$ ip neighbor help
Usage: ip neigh { add | del | change | replace }
                { ADDR [ lladdr LLADDR ] [ nud STATE ] | proxy ADDR } [ dev DEV ]
                                 [ router ] [ extern_learn ]

       ip neigh { show | flush } [ proxy ] [ to PREFIX ] [ dev DEV ] [ nud STATE ]
                                 [ vrf NAME ]

STATE := { permanent | noarp | stale | reachable | none |
           incomplete | delay | probe | failed }
```
`lladdr`: 物理mac地址相关
`nud`: 状态相关
`proxy`: 代理相关
`dev`: 设备接口相关

状态说明:
状态|说明
-|-
`permanent`|邻居条目永久有效, 只能管理员删除
`noarp`|邻居条目有效, 不会验证此条目, 可以在生存期过后删除
`reachable`|邻居条目有效, 一直到过期
`stale`|邻居条目有效但不能信任
`none`|刚添加邻居条目的伪状态, 随时可删除
`incomplete`|邻居条目还未验证或解决
`delay`|邻居条目延迟验证
`probe`|邻居条目正在探索
`failed`|邻居条目验证失败

查看邻居表:
```bash
# neigh 是缩写
$ ip neighbor show
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 DELAY

# 查看 IPv4 (ARP) 邻居表
$ ip -4 neighbor show

# 查看 IPv6 (NDP) 邻居表
$ ip -6 neighbor show
```
>所有命令都可以使用`-4`和`-6`选项过滤 IPv4 (ARP) 和 IPv6 (NDP)

查看指定接口的邻居表:
```bash
# 格式
$ ip neighbor show dev ${interface name}

# 示例
$ ip neighbor show dev enp0s3
10.0.2.2 lladdr 52:54:00:12:35:02 REACHABLE
```
清空一个指定接口的邻居表:
```bash
$ ip neighbor flush dev ${interface name}

# 示例
$ ip neighbor flush dev enp0s3
```
添加和删除一个邻居条目:
```bash
# 在eth0接口上为192.0.2.1添加一个永久的mac地址邻居条目
$ ip neigh add 192.0.2.1 lladdr 22:ce:e0:99:63:6f dev eth0 nud permanent

# 删除
$ ip neigh del 192.0.2.1 lladdr 22:ce:e0:99:63:6f dev eth0
```
改变一个邻居条目状态:
```bash
# 将状态改变为 reachable
$ ip neigh change 192.0.2.1 dev eth0 nud reachable
```

## 链路管理
```bash
$ ip link help
Usage: ip link add [link DEV] [ name ] NAME
                   [ txqueuelen PACKETS ]
                   [ address LLADDR ]
                   [ broadcast LLADDR ]
                   [ mtu MTU ] [index IDX ]
                   [ numtxqueues QUEUE_COUNT ]
                   [ numrxqueues QUEUE_COUNT ]
                   type TYPE [ ARGS ]

       ip link delete { DEVICE | dev DEVICE | group DEVGROUP } type TYPE [ ARGS ]

       ip link set { DEVICE | dev DEVICE | group DEVGROUP }
                          [ { up | down } ]
                          [ type TYPE ARGS ]
                          [ arp { on | off } ]
                          [ dynamic { on | off } ]
                          [ multicast { on | off } ]
                          [ allmulticast { on | off } ]
                          [ promisc { on | off } ]
                          [ trailers { on | off } ]
                          [ carrier { on | off } ]
                          [ txqueuelen PACKETS ]
                          [ name NEWNAME ]
                          [ address LLADDR ]
                          [ broadcast LLADDR ]
                          [ mtu MTU ]
                          [ netns { PID | NAME } ]
                          [ link-netns NAME | link-netnsid ID ]
                          [ alias NAME ]
                          [ vf NUM [ mac LLADDR ]
                                   [ vlan VLANID [ qos VLAN-QOS ] [ proto VLAN-PROTO ] ]
                                   [ rate TXRATE ]
                                   [ max_tx_rate TXRATE ]
                                   [ min_tx_rate TXRATE ]
                                   [ spoofchk { on | off} ]
                                   [ query_rss { on | off} ]
                                   [ state { auto | enable | disable} ] ]
                                   [ trust { on | off} ] ]
                                   [ node_guid { eui64 } ]
                                   [ port_guid { eui64 } ]
                          [ xdp { off |
                                  object FILE [ section NAME ] [ verbose ] |
                                  pinned FILE } ]
                          [ master DEVICE ][ vrf NAME ]
                          [ nomaster ]
                          [ addrgenmode { eui64 | none | stable_secret | random } ]
                          [ protodown { on | off } ]
                          [ gso_max_size BYTES ] | [ gso_max_segs PACKETS ]

       ip link show [ DEVICE | group GROUP ] [up] [master DEV] [vrf NAME] [type TYPE]

       ip link xstats type TYPE [ ARGS ]

       ip link afstats [ dev DEVICE ]

       ip link help [ TYPE ]

TYPE := { vlan | veth | vcan | vxcan | dummy | ifb | macvlan | macvtap |
          bridge | bond | team | ipoib | ip6tnl | ipip | sit | vxlan |
          gre | gretap | erspan | ip6gre | ip6gretap | ip6erspan |
          vti | nlmon | team_slave | bond_slave | bridge_slave |
          ipvlan | ipvtap | geneve | vrf | macsec | netdevsim | rmnet }
```
显示链路信息:
```bash
# 显示所有链路信息
$ ip link show

# 显示指定链路信息
ip link show dev eth0
ip link show eth0
```
设置链路状态:
```bash
# 关闭
$ ip link set dev eth0 down

# 打开
$ ip link set dev eth0 up
```
设置链路别名:
```bash
# 设置别名
$ ip link set dev eth0 alias "别名".
```
重命名设备接口:
```bash
# 重命名eth0为lan, 需要先设置此链路状态为关闭
$ ip link set dev eth0 name lan
```
改变链路地址(通常是mac地址):
```bash
# 改变 eth0 的mac地址
$ ip link set dev eth0 address 22:ce:e0:99:63:6f
```
改变链路的MTU(最大传输单元, 默认为1500):
```bash
$ ip link set dev eth0 mtu 1480
```
删除一个链路:
```bash
$ ip link delete dev eth0
```
>显然，只能删除虚拟链接，例如VLAN，网桥或隧道。对于物理接口，此命令无效。

在接口上启用或禁用多播:(除非你真的知道你在做什么，最好不要碰这个选项。)
```bash
# 启用多播
$ ip link set eth0 multicast on

# 禁用多播
$ ip link set eth0 multicast off
```
在接口上启用或禁用ARP:
```bash
# 启用ARP
$ ip link set eth0 arp on

# 禁用ARP
$ ip link set eth0 arp off
```
创建一个VLAN接口:
```bash
# 格式
$ ip link add name ${VLAN interface name} link ${parent interface name} type vlan id ${tag}

# 示例
$ ip link add name eth0.110 link eth0 type vlan id 110
```
Linux唯一支持的VLAN类型是IEEE 802.1q VLAN。

可以为VLAN接口使用任何名称。 以上示例中`eth0.110`是传统格式，但不是必需的。

任何类似以太网的设备都可以作为VLAN接口的父级：bridge, bonding, L2 tunnels.

***
创建一个QinQ接口:
```bash
# 格式
# 创建一个服务端标签接口
$ ip link add name ${service interface} link ${physical interface} type vlan proto 802.1ad id ${service tag}

# 创建一个客户端标签接口
$ ip link add name ${client interface} link ${service interface} type vlan proto 802.1q id ${client tag}

# 示例
# 创建一个服务端标签接口
$ ip link add name eth0.100 link eth0 type vlan proto 802.1ad id 100 

# 创建一个客户端标签接口
$ ip link add name eth0.100.200 link eth0.100 type vlan proto 802.1q id 200 
```
QinQ类似是一种扩展的VLAN, 也称为堆栈VLAN或双VLAN，由IEEE 802.1ad标准化。它用两层封装了VLAN标签:专用网络的内部标签和公共网络的外部标签。

***
创建一个虚拟mac地址(MACVLAN):
```bash
# 格式
$ ip link add name ${macvlan interface name} link ${parent interface} type macvlan

# 示例
$ ip link add name peth0 link eth0 type macvlan
```
这是一种网卡虚拟化的解决方案, 相当于一块物理网卡虚拟成多块虚拟网卡.

***
创建一个虚拟接口:
```bash
# 格式
$ ip link add name ${dummy interface name} type dummy

# 示例
$ ip link add name dummy0 type dummy
```
在Linux中, 由于历史原因只有一个回环接口`lo`, 虚拟接口类似回环接口`lo`, 但是虚拟接口可以有多个.

可用于单个主机内的通信。 回环或虚拟接口也是在具有多个物理接口的路由器上分配管理地址的好地方。

***
创建一个网桥(bridge)接口:
```bash
# 格式
$ ip link add name ${bridge name} type bridge

# 示例
$ ip link add name br0 type bridge
```
网桥接口是虚拟以太网交换机。 

您可以使用它们将Linux机器变成慢速的L2交换机，或在虚拟机监控程序主机上的虚拟机之间启用通信。 

请注意，将Linux机器变成物理交换机并不是一个完全荒谬的想法，因为与笨拙的硬件交换机不同，它可以用作透明防火墙。

可以将IP地址分配给网桥，并且该IP地址将在所有网桥端口中可见。

添加网桥端口:
```bash
# 格式
$ ip link set dev ${interface name} master ${bridge name}

# 示例
$ ip link set dev eth0 master br0
```
添加到网桥的接口将成为虚拟交换机端口。它只在数据链路层上运行，并停止所有网络层操作。

删除网桥端口:
```bash
# 格式
$ ip link set dev ${interface name} nomaster

# 示例
$ ip link set dev eth0 nomaster
```

***
创建一个bonding接口:
```bash
# 格式
$ ip link add name ${name} type bond

# 示例
$ ip link add name bond1 type bond
```
添加和删除bonding成员的方式与网桥操作一致, 都是使用master和nomaster.

***
添加一个中间功能模块接口:
```bash
# 格式
$ ip link add ${interface name} type ifb

# 示例
$ ip link add ifb10 type ifb
```
中间功能块设备与`tc`一起用于流量重定向和镜像。具体查看`tc`文档.

***
创建一对虚拟以太网设备:
```bash
# 格式
$ ip link add name ${first device name} type veth peer name ${second device name}

# 示例
$ ip link add name veth-host type veth peer name veth-guest
```
> 注意：虚拟以太网设备是在UP状态下创建的，创建后无需手动将其启动。

虚拟以太网（virtualethernet，veth）设备总是成对出现，并作为一个双向管道工作：任何进入其中一个设备的东西都会从另一个设备出来。它们与系统分区特性（如网络名称空间和容器（OpenVZ或LXC））结合使用，用于将一个分区连接到另一个分区。

## 链路组管理
链路组类似于托管交换机中的端口范围。您可以将网络接口添加到编号的组中，并一次对该组中的所有接口执行操作。

未分配给任何组的链接属于组0（“默认”）。

将接口添加到组:
```bash
# 格式
$ ip link set dev ${interface name} group ${group number}

# 示例
$ ip link set dev eth0 group 42
$ ip link set dev eth1 group 42
```

从组中删除接口或分配到别的组: 可以通过将其分配给默认组0来完成。
```bash
# 格式
$ ip link set dev ${interface name} group 0
$ ip link set dev ${interface} group default

# 示例
$ ip link set dev tun10 group 0
```
为组指定符号名称:

组的符号名称配置在`/etc/iproute2/group`文件中, 像默认组0的符号`default`也在此配置文件中, 可以修改或添加其它组的符号名称. 最多255组.

按照格式:`${number} ${name}`写入配置文件即可.

对组执行操作:
```bash
# 格式
$ ip link set group ${group number} ${operation and arguments}

# 示例
$ ip link set group 42 down
$ ip link set group uplinks mtu 1200
```
查看指定组的信息:
```bash
# 查看组42的链路信息 
$ ip link show group 42

# 查看组42的地址信息 
$ ip addr show group 42
```

## 路由管理
对于IPv4路由，可以使用前缀长度或点分十进制子网掩码。如192.0.2.0/24和192.0.2.0/255.255.255.0是一样的。

Linux内核不保留下一跳无法到达的路由。如果链接断开，则将从该路由表中永久删除所有使用该链接的路由。您可能没有注意到这种现象，因为在许多情况下，当链路出现故障时，其他软件（例如NetworkManager或rp-pppoe）会负责恢复路由。

如果要将Linux计算机用作路由器，请考虑安装路由协议套件，例如FreeRangeRouting或BIRD。 它们会跟踪链接状态，并在链接断开后又断开时恢复路由。 当然，它们还允许您使用动态路由协议，例如OSPF和BGP。

为接口分配地址后，系统将计算其网络地址并创建到该网络的路由（这就是为什么需要子网掩码的原因）。 这样的路由称为**连接路由**。

例如，如果您将203.0.113.25/24分配给eth0，则将创建到203.0.113.0/24网络的连接路由，系统将知道可以直接访问该网络中的主机。

***
查看路由表：
```bash
ip route

# 或者
ip route show
```
查看到网络和所有子网的路由：
```bash
# 语法，to 在命令中是可选的词
ip route show to root ${address}/${mask}

# 查看网络192.168.0.0/24下的所有子网，
# 会包括(如果有)如192.168.0.0/25、192.168.0.128/25等子网
ip route show to root 192.168.0.0/24
```
查看到网络和所有超网的路由：
```bash
# 语法，to 在命令中是可选的词
ip route show to match ${address}/${mask}

# 示例
ip route show to match 10.0.2.0/24
```
查看到精确子网的路由：
```bash
# 语法，to 在命令中是可选的词
ip route show to exact ${address}/${mask}

# 查看网络192.168.0.0/24的子网
# 不会会包括(如果有)如192.168.0.0/25、192.168.0.128/25等子网
ip route show to exact 192.168.0.0/24
```
仅查看内核实际当前使用的路由：
```bash
# 语法
ip route get ${address}/${mask}

# 示例
ip route get 192.168.0.0/24
```
***
通过网关添加路由：
```bash
# 语法
ip route add ${address}/${mask} via ${next hop}

# 示例
ip route add 192.0.2.128/25 via 192.0.2.1
ip route add 2001:db8:1::/48 via 2001:db8:1::1
```
通过接口添加路由：
```bash
# 语法
ip route add ${address}/${mask} dev ${interface name}

# 示例
ip route add 192.0.2.0/25 dev enp0s3
```
接口路由通常与点对点接口（例如PPP隧道）一起使用，而不需要下一跳地址。

添加默认路由：
```bash
# 语法，快捷方式
ip route add default via ${address}/${mask}
ip route add default dev ${interface name}
# 等价于
ip route add 0.0.0.0/0 via ${address}/${mask}
ip route add 0.0.0.0/0 dev ${interface name}

# 对于ipv6，default等价于::/0
ip -6 route add default via 2001:db8::1
```
***
改变或替换路由：
```bash
ip route change 192.168.2.0/24 via 10.0.0.1
ip route replace 192.168.2.0/24 via 10.0.0.1

ip route change 192.0.2.1/27 dev tun0
ip route replace 192.0.2.1/27 dev tun0
```
它们之间的区别在于，如果您尝试更改不存在的路由，则`change`命令将产生错误。 如果该命令尚不存在，则`replace`命令将创建一条路由。
***
删除路由：
```bash
# 语法
ip route delete ${route specifier}

# 示例
ip route delete 10.0.1.0/25 via 10.0.0.1
ip route delete default dev ppp0
```
***
黑洞路由：
```bash
# 语法
ip route add blackhole ${address}/${mask}

# 示例
ip route add blackhole 192.0.2.1/32
```
与黑洞路径匹配的目的地的流量将被自动静默丢弃。

黑洞路由有两种用例。 

首先，它们可以用作非常快速的出站流量过滤器，例如，使已知的僵尸网络控制器无法访问或保护网络内的服务器免受传入的DDoS攻击。 

其次，如果您只有到其部分的真实路由，但希望将其汇总发布，则可以使用它们来欺骗路由协议守护程序，使其认为您具有到网络的路由。

***
其它特殊路由：
```bash
ip route add unreachable ${address}/${mask}

ip route add prohibit ${address}/${mask}

ip route add throw ${address}/${mask}
```
这些路由使系统丢弃数据包，并以ICMP错误消息答复发件人。
- `unreachable`：发送 "host unreachable" 主机不可达
- `prohibit`：发送 "administratively prohibited" 管理上禁止
- `throw`：发送 "net unreachable" 网络不可达

与黑洞路由不同，不建议使用这些路由来阻止不必要的流量（例如DDoS），因为它们会为每个丢弃的数据包生成一个回复数据包，从而产生更大的流量。 它们可以很好地实施内部访问策略，但是通常使用防火墙是一个更好的主意。

***
路由的度量(metric)设置：
```bash
# 语法
ip route add ${address}/${mask} via ${gateway} metric ${number}

# 示例
ip route add 192.168.2.0/24 via 10.0.1.1 metric 5
ip route add 192.168.2.0 dev ppp0 metric 10
```
如果有多个通往同一网络的度量值不同的路由，则内核会**首选度量值最低**的路由。

此功能通常用于实现到重要目标的备份连接。

***
多路径路由：
```bash
# 语法
ip route add ${addresss}/${mask} nexthop via ${gateway 1} weight ${number} nexthop via ${gateway 2} weight ${number}

# 示例
ip route add default nexthop via 192.168.1.1 weight 1 nexthop dev ppp0 weight 10
```
多路径路由使系统根据权重在多个链路上平衡数据包（首选更高的权重，因此权重为2的网关/接口将获得比权重为1的另一个网关/接口大约多两倍的流量）。您可以拥有任意数量的网关，并且可以混合使用网关和接口路由。

>警告：这种类型的负载平衡的不利之处在于，不能保证数据包将通过它们进入的同一链路发送回去。这称为“非对称路由”。 对于只转发数据包而不进行任何本地流量处理的路由器，通常很好，在某些情况下甚至是不可避免的。

## 策略路由
Linux中基于策略的路由（Policy-based routing，PBR）的设计方法如下：首先创建自定义路由表，然后创建规则来告诉内核哪些表用于哪些数据包。

## 网络命名空间
网络命名空间是单个计算机中的隔离网络堆栈实例。 它们可用于安全域分离，管理虚拟机之间的流量，等等。

每个命名空间都是网络堆栈的完整副本，具有自己的接口、地址、路由等。您可以在命名空间内运行进程，并将命名空间桥接到物理接口。\

namespace(命名空间)和cgroup是软件容器化（想想Docker）趋势中的两个主要内核技术。简单来说，cgroup是一种对进程进行统一的资源监控和限制，它控制着你可以使用多少系统资源（CPU，内存等）。而namespace是对全局系统资源的一种封装隔离，它通过Linux内核对系统资源进行隔离和虚拟化的特性，限制了您可以看到的内容。

Linux 3.8内核提供了6种类型的命名空间：Process ID (pid)、Mount (mnt)、Network (net)、InterProcess Communication (ipc)、UTS、User ID (user)。例如，pid命名空间内的进程只能看到同一命名空间中的进程。mnt命名空间，可以将进程附加到自己的文件系统（如chroot）。

网络命名空间为命名空间内的所有进程提供了全新隔离的网络协议栈。这包括网络接口，路由表和iptables规则。通过使用网络命名空间就可以实现网络虚拟环境，实现彼此之间的网络隔离，这对于云计算中租户网络隔离非常重要，Docker中的网络隔离也是基于此实现的。

### 帮助信息
```bash
$ ip netns help
Usage: ip netns list
       ip netns add NAME
       ip netns set NAME NETNSID
       ip [-all] netns delete [NAME]
       ip netns identify [PID]
       ip netns pids NAME
       ip [-all] netns exec [NAME] cmd ...
       ip netns monitor
       ip netns list-id
NETNSID := auto | POSITIVE-INT
```
### 新增一个网络命名空间
创建一个新的持久网络命名空间：此命令将创建一个名为n0的新网络命名空间。创建命名空间后，ip命令会在`/var/run/netns`目录下增加一个n0文件。
```bash
# 新增一个名称为 n0 的网络命名空间
ip netns add n0

# 列出存在的网络命名空间
ip netns list
```
### 删除一个网络命名空间
```bash
ip netns delete n0
```
### 在网络命名空间内运行进程
```bash
# 语法
ip [-all] netns exec [NAME] cmd ...

# 在 foo 网络命名空间内运行 /bin/sh
ip netns exec foo /bin/sh
```
### 列出分配给命名空间的所以进程PID
```bash
ip netns pids n0
```
### 识别进程所属的主网络命名空间
```bash
# 语法
ip netns identify [PID]

# 识别PID=9000的进程所属的主网络命名空间
ip netns identify 9000
```
### 分配一个网络接口到某个网络命名空间
网络接口就是网卡设备。

如果使用PID，则分配网络接口到PID所属的主命名空间中。
```bash
# 语法
ip link set dev ${interface name} netns ${namespace name}
ip link set dev ${interface name} netns ${pid}

# 分配 eth0.100 到 foo网络命名空间中
ip link set dev eth0.100 netns foo

# 将 eth001 设备从 n0 网络命名空间中移到默认的网络命名空间中
# 因为 PID=1 总是在默认的网络命名空间中
ip netns exec n0 ip link set dev eth001 netns 1
```
注意：当你将一个网络接口移到另一个命名空间中时，会丢失现有的所有配置。

### 将一个命名空间连接到另一个命名空间
可以通过创建一对`veth`链接并将它们分配到不同的命名空间来实现。

假设要将名为“foo”的命名空间连接到默认命名空间。

```bash
# 首先，创建一对veth设备：
ip link add name veth1 type veth peer name veth2

# 将veth2分配到命名空间foo
ip link set dev veth2 netns foo

# 在 foo 中开启veth2
ip netns exec foo ip link set dev veth2 up

# 添加一个IP地址到veth2
ip netns exec foo ip address add 10.1.1.1/24 dev veth2

# 在默认的网络命名空间中添加同类IP地址到veth1
ip address add 10.1.1.2/24 dev veth1
```
这样可以在默认的网络命名空间中ping通foo网络命名空间中的veth2地址。

### 示例
通过两个隔离的网络命名空间来实现互通。

首先创建两个网络命名空间: `n01`和`n02`
```bash
$ sudo ip netns add n01
$ sudo ip netns add n02
$ ip netns list
n02
n01
```
创建一对虚拟veth设备(类似网线)：`veth01`和`veth02`
```bash
$ sudo ip link add name veth01 type veth peer name veth02

$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:9b:e4:fb brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:7d:01:2a:61 brd ff:ff:ff:ff:ff:ff
4: veth02@veth01: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 36:e5:3a:dd:13:e5 brd ff:ff:ff:ff:ff:ff
5: veth01@veth02: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 0a:c8:db:da:a8:80 brd ff:ff:ff:ff:ff:ff
```
这对veth状态是`DOWN`，也就是未激活状态。

分配`veth01`到网络命名空间`n01`中，`veth02`到网络命名空间`n02`中：
```bash
$ sudo ip link set dev veth01 netns n01
$ sudo ip link set dev veth02 netns n02

# 在各自的网络命名空间中查看
$ sudo ip netns exec n01 ip link show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
5: veth01@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 0a:c8:db:da:a8:80 brd ff:ff:ff:ff:ff:ff link-netns n02
$ sudo ip netns exec n02 ip link show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
4: veth02@if5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 36:e5:3a:dd:13:e5 brd ff:ff:ff:ff:ff:ff link-netns n01
```
激活`veth01`和`veth02`：
```bash
$ sudo ip netns exec n01 ip link set dev veth01 up
$ sudo ip netns exec n02 ip link set dev veth02 up

# 查看激活状态
$ sudo ip netns exec n01 ip link show dev veth01
5: veth01@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 0a:c8:db:da:a8:80 brd ff:ff:ff:ff:ff:ff link-netns n02
$ sudo ip netns exec n02 ip link show dev veth02
4: veth02@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 36:e5:3a:dd:13:e5 brd ff:ff:ff:ff:ff:ff link-netns n01
```
添加IP地址到veth：
```bash
$ sudo ip netns exec n01 ip addr add 10.1.1.1/24 dev veth01
$ sudo ip netns exec n02 ip addr add 10.1.1.2/24 dev veth02

# 查看状态
$ sudo ip netns exec n01 ip addr show dev veth01
5: veth01@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 0a:c8:db:da:a8:80 brd ff:ff:ff:ff:ff:ff link-netns n02
    inet 10.1.1.1/24 scope global veth01
       valid_lft forever preferred_lft forever
    inet6 fe80::8c8:dbff:feda:a880/64 scope link
       valid_lft forever preferred_lft forever

$ sudo ip netns exec n02 ip addr show dev veth02
4: veth02@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 36:e5:3a:dd:13:e5 brd ff:ff:ff:ff:ff:ff link-netns n01
    inet 10.1.1.2/24 scope global veth02
       valid_lft forever preferred_lft forever
    inet6 fe80::34e5:3aff:fedd:13e5/64 scope link
       valid_lft forever preferred_lft forever
```
ping测试1：
```bash
$ sudo ip netns exec n01 ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.032 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.039 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.035 ms
^C
--- 10.1.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 28ms
rtt min/avg/max/mdev = 0.032/0.035/0.039/0.005 ms
```
如果测试1ping不通，可以将IP地址通过路由添加到veth：
```bash
sudo ip exec n01 ip route add 10.1.1.1/24 dev veth01
sudo ip exec n02 ip route add 10.1.1.2/24 dev veth02
```