---
author: edmund
comments: true
date: 2018-05-24 08:35:48+00:00
layout: post
link: http://118.25.17.78/blog/2018/05/24/linux%e7%bd%91%e7%bb%9c%e7%ae%a1%e7%90%86%e4%b9%8bifcfg%e7%b3%bb%e5%88%97%e5%91%bd%e4%bb%a4/
slug: linux%e7%bd%91%e7%bb%9c%e7%ae%a1%e7%90%86%e4%b9%8bifcfg%e7%b3%bb%e5%88%97%e5%91%bd%e4%bb%a4
title: Linux网络管理之ifcfg系列命令
wordpress_id: 349
categories:
- Linux技术
post_format:
- 日志
tags:
- linux
- 命令
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







这里要讲的是通过ifcfg套件进行网络的静态配置。







# ifcfg套件




在Linux系统上，有一组非常简单易用的网络配置套件，它们已经被使用了几十年，虽然年代古老，但是目前的绝大多数Linux发行版中仍然会使用这一套组件。




这套组件包括 : ifconfig（网络接口配置）、route（路由表配置）、netstat（网络情况查看）。







## ifconfig命令




ifconfig命令是用来配置内核网络接口的，比如让接口上线下线、配置接口地址、配置接口子网掩码、配置接口的最大传输单元（MTU）等等，使用起来非常简单。







### 命令用法




#### 查看接口情况




**ifconfig [-a] [-s] [interface]**




如果直接使用ifconfig命令不加任何选项和参数，则表示显示所有up状态的接口。




**-a** : 显示所有的接口，包含up和down的接口。




**-s** : 显示接口的概要信息。类似于netstat -{I|i}




**interface** : 指定接口的名字



    
    [root@edu ~]# ifconfig  
    ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.60.72.190  netmask 255.255.255.0  broadcast 10.60.72.255
            inet6 fe80::ae75:db2b:9e42:c9ed  prefixlen 64  scopeid 0x20<link>
            ether 00:0c:29:dd:b3:ae  txqueuelen 1000  (Ethernet)
            RX packets 865618  bytes 1125008188 (1.0 GiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 2067  bytes 383665 (374.6 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1  (Local Loopback)
            RX packets 12  bytes 1008 (1008.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 12  bytes 1008 (1008.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    [root@edu ~]# ifconfig ens33
    ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.60.72.190  netmask 255.255.255.0  broadcast 10.60.72.255
            inet6 fe80::ae75:db2b:9e42:c9ed  prefixlen 64  scopeid 0x20<link>
            ether 00:0c:29:dd:b3:ae  txqueuelen 1000  (Ethernet)
            RX packets 865671  bytes 1125044892 (1.0 GiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 2084  bytes 386411 (377.3 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    [root@edu ~]# ifconfig -s 
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    ens33     1500   865724      0      0 0          2101      0      0      0 BMRU
    lo       65536       12      0      0 0            12      0      0      0 LRU




 




#### 配置接口




**ifconfig interface {options | address ... }**




**interface** : 接口的名字。




**address** : 接口的IP地址，在指定IP地址时可以通过CIDR表示法同时指定子网掩码（subnetmask）。比如 : 192.168.1.1/24。




**options** : ifconfig命令的选项，常用选项如下所示 : 




**up** : 让指定接口上线。




**down** : 让指定接口下线。




**[-]arp **: 启用或者禁用接口对arp协议的使用。-arp表示禁用，arp表示启用。




**[-]promisc** : 启用或禁用接口的混杂模式，启用了混杂模式则表示这个接口将会接收所有经过这个接口的数据包，而不管目的地址是不是该接口。-promisc表示禁用，promisc表示启用。




**mtu N** : 设置接口的MTU大小。




**netmask ADDRESS**: 设置接口的子网掩码。子网掩码同样可以在address选项中直接设置。







配置IP地址/子网掩码并启用该接口



    
    [root@edu ~]# ifconfig ens33 10.60.72.190/24 up



    
    [root@edu ~]# ifconfig ens33 10.60.72.190 netmask 255.255.255.0




 




让指定接口下线



    
    [root@edu ~]# ifconfig lo down  #让本地回环接口下线
    [root@edu ~]# ifconfig  #发现只剩下了ens33接口。稍后记得将lo接口重新上线。
    ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.60.72.190  netmask 255.255.255.0  broadcast 10.60.72.255
            inet6 fe80::ae75:db2b:9e42:c9ed  prefixlen 64  scopeid 0x20<link>
            ether 00:0c:29:dd:b3:ae  txqueuelen 1000  (Ethernet)
            RX packets 874161  bytes 1133898619 (1.0 GiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 2411  bytes 488549 (477.0 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    




 







## route命令




route命令用于显示和操作内核路由表，主要用来配置静态的主机路由、网络路由和默认路由。




### 命令用法




#### 查看内核路由表




**route [-nee]**




**-n** : 以数字形式显示地址，而不会尝试去反向解析数字地址的主机名。该选项很常用。




**-e** : 以netstat的风格显示路由表。




**-ee** : 以netstat的长风格显示路由表。



    
    [root@edu ~]# route
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         gateway         0.0.0.0         UG    100    0        0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33
    [root@edu ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         10.60.72.1      0.0.0.0         UG    100    0        0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33




 




#### 添加路由条目




**route add [-net|-host] target [netmask NETMASK] [gw GATEWAY ] [metric N] [mss M] [window W] [[dev] INTERFACE ]**




**-net target** : 添加一条网络路由。比如 : -net 192.168.1.0




**-host target** : 添加一条主机路由。比如 : -host 192.168.1.1




**netmask NETMASK** : 指定子网掩码。




**gw GATEWAY** : 指定默认网关的地址。




**metric N** : 指定路由权值，又称路由度量。当一个地址匹配了多个路由条目时，路由会选择最低权值的路由条目。




**mss M** : 指定路由的MTU为M bytes。




**window W** : 指定TCP的窗口大小为W bytes。




**dev INTERFACE** : 指定该路由条目走哪个接口过。




 




添加一条网络路由192.168.1.0，子网掩码为255.255.255.0，默认网关为10.60.72.1，从ens33接口过。



    
    [root@edu ~]# route add -net 192.168.1.0 netmask 255.255.255.0 gw 10.60.72.1 dev ens33
    [root@edu ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         10.60.72.1      0.0.0.0         UG    100    0        0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33
    192.168.1.0     10.60.72.1      255.255.255.0   UG    0      0        0 ens33




 




添加一条主机路由192.168.1.1，默认网关为10.60.72.1，从ens33接口过。



    
    [root@edu ~]# route add -host 192.168.1.1 gw 10.60.72.1 dev ens33  #主机路由不需要设置子网掩码，因为子网掩码对于主机路由来说没有意义。
    [root@edu ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         10.60.72.1      0.0.0.0         UG    100    0        0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33
    192.168.1.0     10.60.72.1      255.255.255.0   UG    0      0        0 ens33
    192.168.1.1     10.60.72.1      255.255.255.255 UGH   0      0        0 ens33




 




#### 删除路由条目




**route del [-net|-host] target [gw GATEWAY] [netmask NETMASK] [metric N] [[dev] If]**




删除路由条目的所有选项意义与添加路由条目相同，且所需指定的选项更少。当我们删除路由条目时，不一定要指定添加这个路由条目时指定的所有选项，只要有部分选项就能够唯一确定这一条路由条目，那么就只需要指定部分选项。







删除网络路由192.168.1.0，子网掩码为255.255.255.0，默认网关为10.60.72.1，从ens33接口过的路由条目。



    
    [root@edu ~]# route del -net 192.168.1.0 netmask 255.255.255.0  #可以发现我们在创建时指定了目标IP、子网掩码、网关、接口，但是删除时只指定了目标IP和子网掩码。这是因为IP+子网掩码在这里已经可以唯一确定这一条路由条目。
    [root@edu ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         10.60.72.1      0.0.0.0         UG    100    0        0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33
    192.168.1.1     10.60.72.1      255.255.255.255 UGH   0      0        0 ens33




 




删除一条主机路由192.168.1.1，默认网关为10.60.72.1，从ens33接口过的路由条目。



    
    [root@edu ~]# route del -host 192.168.1.1
    [root@edu ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         10.60.72.1      0.0.0.0         UG    100    0        0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33




 




#### 添加默认路由




**route add -net 0.0.0.0 netmask 0.0.0.0 gw GATEWAY [[dev] INTERFACE]**




**route add default gw GATEWAY [[dev] INTERFACE]**




这两种用法都可以添加一条默认路由，而下面的写法中的default其实相当于上面的 -net 0.0.0.0 netmask 0.0.0.0 。







**添加一条默认路由，网关为 10.60.72.1。**



    
    [root@edu ~]# route add default gw 10.60.72.1
    [root@edu ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         10.60.72.1      0.0.0.0         UG    0      0        0 ens33
    0.0.0.0         10.60.72.1      0.0.0.0         UG    100    0        0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U     100    0        0 ens33
    




 




## netstat命令




netstat命令用于显示网络连接情况、内核路由表、接口统计信息、IP地址掩蔽连接情况、多播成员关系等。




### 命令用法




#### 显示网络连接情况




**netstat  [--tcp|-t] [--udp|-u] [--raw|-w] [--listening|-l] [--all|-a] [--numeric|-n] [--numeric-hosts] [--numeric-ports] [--numeric-users]  [--extend|-e] [--program|-p]**




**--tcp|-t** : 只显示tcp连接。




**--udp|-u** : 只显示udp连接。




**--raw|-w** : 只显示原始套接字。原始套接字（raw socket）指的是不经过传输层封装直接发送和接收的IP报文。




**--listening|-l** : 只显示处于监听（Listening）状态的连接。默认不会显示监听状态的连接。




**--all|-a** : 显示所有的连接。包括监听和非监听状态的连接。




**--numeric|-n** : 以数字方式显示地址，而不会试图将数字反向解析成主机名、端口、或者用户名。




**--numeric-hosts** : 以数字方式显示主机名。




**--numeric-ports** : 以数字方式显示端口号。




**--numeric-users** : 以数字方式显示用户名。




**--extend|-e** : 显示额外的信息。




**--program|-p** : 显示每个套接字所属的程序名和PID。




 




显示当前所有的tcp连接



    
    [root@edu ~]# netstat -tan
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
    tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
    tcp        0     52 10.60.72.190:22         10.60.72.28:1654        ESTABLISHED
    tcp6       0      0 :::22                   :::*                    LISTEN     
    tcp6       0      0 ::1:25                  :::*                    LISTEN 




 




显示当前所有监听状态的tcp连接



    
    [root@edu ~]# netstat -tnl
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
    tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
    tcp6       0      0 :::22                   :::*                    LISTEN     
    tcp6       0      0 ::1:25                  :::*                    LISTEN 




 




显示当前所有监听状态的tcp连接以及其所属的程序



    
    [root@edu ~]# netstat -tnlp
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      936/sshd            
    tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1042/master         
    tcp6       0      0 :::22                   :::*                    LISTEN      936/sshd            
    tcp6       0      0 ::1:25                  :::*                    LISTEN      1042/master




 




#### 显示内核路由表




**netstat {--route|-r}  [--extend|-e] [--numeric|-n] [--numeric-hosts] [--numeric-ports] [--numeric-users]**




**--route|-r** : 显示内核路由表。




其它选项与上面的一样。



    
    [root@edu ~]# netstat -r
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    default         gateway         0.0.0.0         UG        0 0          0 ens33
    default         gateway         0.0.0.0         UG        0 0          0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U         0 0          0 ens33
    [root@edu ~]# netstat -rn
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    0.0.0.0         10.60.72.1      0.0.0.0         UG        0 0          0 ens33
    0.0.0.0         10.60.72.1      0.0.0.0         UG        0 0          0 ens33
    10.60.72.0      0.0.0.0         255.255.255.0   U         0 0          0 ens33
    




 




#### 显示接口统计信息




**netstat {--interfaces|-I|-i}  [--extend|-e] [--program|-p] [--numeric|-n] [--numeric-hosts] [--numeric-ports] [--numeric-users]**




**--interfaces|-I|-i** : 显示所有接口统计概要信息。类似于ifconfig -s。




其它选项与上面的一样。



    
    [root@edu ~]# netstat -i
    Kernel Interface table
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    ens33     1500  1589019      0      0 0          4815      0      0      0 BMRU
    lo       65536       12      0      0 0            12      0      0      0 LRU




 




 




## 拓展知识——指定DNS服务器




DNS服务器是负责域名解析的服务器，在windows中我们可以添加两个DNS服务器地址，而在Linux中我们可以添加最多三个DNS服务器地址。只需要编辑/etc/resolv.conf文件即可。




DNS服务器指定格式: 




**nameserver DNS_IP_ADDR1**




**nameserver DNS_IP_ADDR2**




**nameserver DNS_IP_ADDR3**




例如 :




**nameserver 192.168.1.1**




**nameserver 1.1.1.1**
