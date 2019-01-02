---
author: edmund
comments: true
date: 2018-06-05 12:53:10+00:00
layout: post
link: http://118.25.17.78/blog/2018/06/05/linux%e7%bd%91%e7%bb%9c%e7%ae%a1%e7%90%86%e4%b9%8bifcfg%e5%92%8croute%e9%85%8d%e7%bd%ae%e6%96%87%e4%bb%b6/
slug: linux%e7%bd%91%e7%bb%9c%e7%ae%a1%e7%90%86%e4%b9%8bifcfg%e5%92%8croute%e9%85%8d%e7%bd%ae%e6%96%87%e4%bb%b6
title: Linux网络管理之ifcfg和route配置文件
wordpress_id: 368
categories:
- Linux技术
post_format:
- 日志
tags:
- CentOS7
- linux
- 命令
- 命令帮助
- 网络
---

# Linux网络配置




关于网络基础知识这里就不讲了，这篇博文假设读者已经具备了基础的网络知识。




在Linux系统中，有很多配置网络环境的方法，而这里除开图形化界面的配置外，一般分为静态配置和动态配置。




## 静态配置




(1) 通过ifcfg套件完成配置，可以实现立即生效，但是重启网卡后配置将丢失。




(2) 通过ip套件完成配置，实现立即生效，但是重启网卡后配置将丢失。




(3) 通过接口的配置文件完成配置，需要重启网卡后才能生效，但是永久有效。




## 动态配置




通过DHCP服务器动态配置。




 




这里要讲的是通过ifcfg和route配置文件进行网络的静态配置。在之前所讲的ifcfg命令套件和ip命令套件中，虽然都能很方便、很灵活地实现很多网络配置，但是一旦接口重启或者是下线，操作系统就会重读ifcfg和route配置文件，那么之前通过命令配置的网络配置信息将会全部丢失。所以，通过配置文件完成的网络配置才能够实现一劳永逸。







# ifcfg配置文件




在操作系统被bootloader引导加载到内存后，操作系统会从bootloader手上接手计算机的控制权，在这个阶段中，操作系统会从硬盘中读取网络接口配置文件以决定哪些接口需要启动，哪些接口需要如何配置。除此之外，在接口重启或者是上线时，操作系统也会从硬盘中读取相应的网络接口配置文件以对相应接口进行网络参数配置。ifcfg配置文件位于**/etc/sysconfig/network-scripts/**下，名为**ifcfg-IFNAME**，IFNAME指的是网络接口的名字，每一个网络接口都对应一个配置文件。所以如果你想要配置eth0接口，那就只需要去编辑/etc/sysconfig/network-scripts/ifcfg-eth0文件即可。




在ifcfg配置文件中有众多的内置变量，用来定义每一个网络配置参数，每一行定义一个参数，并且使用=为变量赋值，注意=两边不要有空格。下面我就来讲一下常用的ifcfg文件配置参数。







## ifcfg配置文件常用内置变量




**TYPE=Ethernet** : 指定网络接口设备的类型，Ethernet表示为以太网接口。通常还有IPSEC、Bridge等类型。




**BOOTPROTO=none|bootp|dhcp** : 指定接口使用何种协议获取配置信息，none或者除了bootp和dhcp外的其他字符都表示使用静态配置方式，bootp和dhcp使用DHCP协议。




**IPV4_FAILURE_FATAL=yes|no** : 在接口获取DHCP配置失败后，是否立即禁用该接口。默认为no，表示不禁用。




**NAME=eth0** : 用于方便用户辨识的配置文件信息的标识名。




**DEVICE=eth0** : 用于指定该配置文件对哪个设备生效。




**UUID=0d4277ed-36f3-4c0d-bbd6-cf718380fab1** : 指定设备的通用唯一识别码（UUID）。




**ONBOOT=yes|no** : 指定是否在引导时激活该设备，yes表示激活。




**HWADDR=0e:a5:1a:b6:fc:86** : 指定设备的硬件地址（MAC）。




**IPADDR0=172.31.24.10** : 指定设备的IP地址，IPADDR0表示该设备的第一个IP，IPADDR1表示该设备的第二个IP。如果只有一个IP地址，那么IPADDR后的数字可以省略。




**PREFIX0=24** : 指定设备的主机位长度。如果和NETMASK同时被设置，则该变量生效。同上，0可以省略。




**GATEWAY0=172.31.24.1** : 指定设备的网关。同上，0可以省略。




**DNS1=192.168.154.3** : 指定设备的第一个DNS服务器。




**DNS2=10.216.106.3** : 指定设备的第二个DNS服务器。




**PEERDNS=yes|no** : 指定在DNS服务器在配置文件中被设置或者在DHCP中被设置后，是否将DNS信息覆盖到/etc/resolv.conf文件中。默认为yes，表示覆盖。




**NM_CONTROLLED=yes|no** : 指定该设备是否接受NetworkManager（在RHEL 6中开始出现的网络配置管理工具，用于代替network）的管理，默认为yes。




**USERCTL=yes|no** : 是否允许普通用户控制该设备。



    
    [root@edu ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
    TYPE="Ethernet"
    BOOTPROTO="static"
    DEFROUTE="yes"
    IPV4_FAILURE_FATAL="no"
    NAME="ens33"
    UUID="0d4277ed-36f3-4c0d-bbd6-cf718380fab1"
    DEVICE="ens33"
    ONBOOT="yes"
    IPADDR="10.60.72.190"
    PREFIX="24"
    GATEWAY="10.60.72.1"
    DNS1="10.60.72.1"
    IPV6_PRIVACY="no"







 




# route配置文件




和接口信息配置一样，路由条目也可以写在配置文件中，防止在重新启动接口或者系统时路由条目丢失。route配置文件位于/etc/sysconfig/network-scripts/下，名为route-IFNAME,IFNAME为接口名称，表示配置文件中的路由条目都经由该设备。需要注意的是，该配置文件只有在对应设备被激活时才会被加载，比如当eth0接口被启动时，route-eth0中的路由条目才会被内核加载。而且如果配置文件中的路由条目中的目标地址如果在该配置文件对应的设备中不存在的话，也是无法被加载的，比如配置文件名为route-lo,而路由条目指明了通往192.168.1.0网络需要经过10.60.72.1，但是lo设备并没有10.60.72.1这个地址，所以无法进行路由条目的配置。







## route配置文件配置方法




route配置文件中也有几个内置变量，但是是在新版本的配置文件语法中出现的，所以route配置文件有两种风格的写法，一种是老版本的语法，一个路由条目占用一行，另一种是新版本的语法，一个路由条目使用三个内置变量定义，每个路由条目会占据三行。




### 配置风格一




**ADDRESS via GATEWAY** : ADDRESS即为目标地址，使用CIDR表示法，GATEWAY为前往目标地址需要经由的接口地址。







例如 : 




在配置文件route-eth0中添加 192.168.1.0/24 via 10.60.72.1 等价于 ip route add 192.168.1.0/24 via 10.60.72.1 dev eth0。




### 配置风格二




**ADDRESSn=<network>**  

**NETMASKn=<network/prefix mask>**  

**GATEWAYn=<next-hop router/gateway IP address>**




即将上述风格中的一行拆分为三行，第一行ADDRESSn为目标地址，第二行NETMASKn为前缀掩码，第三行GATEWAYn为下一跳地址或者网关地址。n表示从0开始的自然数，同一条路由条目的n要相同。







例如 : 




在配置文件route-eth0中添加




ADDRESS0=192.168.2.0  

NETMASK0=255.255.255.0  

GATEWAY0=192.168.1.1




ADDRESS1=192.168.3.0  

NETMASK1=255.255.255.0  

GATEWAY1=192.168.1.1




相当于添加了两条路由条目。















