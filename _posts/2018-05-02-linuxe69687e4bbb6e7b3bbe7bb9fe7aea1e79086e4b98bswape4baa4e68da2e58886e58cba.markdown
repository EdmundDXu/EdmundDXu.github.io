---
author: edmund
comments: true
date: 2018-05-02 05:26:59+00:00
layout: post
link: http://118.25.17.78/blog/2018/05/02/linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bswap%e4%ba%a4%e6%8d%a2%e5%88%86%e5%8c%ba/
slug: linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bswap%e4%ba%a4%e6%8d%a2%e5%88%86%e5%8c%ba
title: Linux文件系统管理之swap交换分区
wordpress_id: 192
categories:
- Linux技术
post_format:
- 日志
tags:
- 命令
- 文件系统管理
---

## 交换分区（swap partition）




交换分区（swap partition）是服务于[分页技术](https://en.wikipedia.org/wiki/Paging)的一种计算机数据存储分区。而分页（paging）又是[虚拟内存技术](https://en.wikipedia.org/wiki/Virtual_memory)的重要部分。虚拟存储器可以通过分页技术，将存储在内存中的数据以页（page）为最小单位，在内存和磁盘之间进行数据传输，而磁盘中存放这些页的分区即为交换分区。




简单来说，通过虚拟内存技术，可以将内存拓展到磁盘，使用磁盘来保存内存中不常用的数据。但是需要注意的是，将内存拓展到磁盘只是使用虚拟内存技术后产生的结果，而不是作用。虚拟内存技术还可以提供给进程一段抽象出来的连续的内存地址空间，使得应用程序认为它拥有连续可用的内存，并且在计算机繁忙时，可以把处于不活动状态的程序以及它们的数据全部交换到磁盘上。





## 创建交换分区




在Linux系统中，创建交换分区的操作就和创建普通分区是一样的。




首先，我们需要给交换分区划分Blocks，即给交换分区分配硬盘空间。这里我们使用fdisk命令。





<blockquote>

> 
> [root@edu data]# **fdisk /dev/sda      **<del>这里我们使用/dev/sda这个设备文件</del>
Welcome to fdisk (util-linux 2.23.2).
> 
> 

> 
> Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
> 
> 

> 
> Command (m for help): **n           **<del>n表示新建一个分区</del>
All primary partitions are in use
Adding logical partition 8
First sector (142065664-251658239, default 142065664):                  <del> 这里直接回车表示默认起始块使用第一个还未被划分过的块</del>
Using default value 142065664
Last sector, +sectors or +size{K,M,G} (142065664-163037183, default 163037183): **+2G          **<del>+2G表示第一个块到最后一个块的间隔大小为2G，即分区的大小为2G</del>
Partition 8 of type Linux and of size 2 GiB is set
> 
> 

> 
> Command (m for help): **t       ** <del>t表示设置分区类型</del>
Partition number (1-8, default 8): **8          **<del>我们设置刚创建出来的第八个逻辑分区</del>
Hex code (type L to list all codes): **82        **<del>82表示分区类型为**Linux swap / Solaris**</del>
**Changed type of partition 'Linux' to 'Linux swap / Solaris'**
> 
> 

> 
> Command (m for help): **w    **<del>使用w将分区表的改动从内存写入磁盘中</del>
The partition table has been altered!
> 
> 

> 
> Calling ioctl() to re-read partition table.
> 
> 

> 
> WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
[root@edu data]# **partx -a /dev/sda       **<del> 由于分区表只有在开机的时候才会读取，所以对分区表的改动目前操作系统并不知情，我们需要执行该命令通知操作系统重新读取分区表</del>
partx: /dev/sda: error adding partitions 1-8
[root@edu data]# **fdisk -l /dev/sda8      **<del>可以看见/dev/sda8这个分区已经被创建出来了</del>
> 
> 

> 
> Disk /dev/sda8: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
> 
> 
</blockquote>




至此，我们已经划分出了一块swap分区的磁盘空间，接下来，我们将这块空间格式化成swap文件系统。使用**mkswap**命令。





<blockquote>

> 
> [root@edu data]# mkswap /dev/sda8
Setting up swapspace version 1, size = 10485756 KiB
no label, UUID=7d2bfcda-988d-4310-b34b-72f2277f3584
[root@edu data]# blkid /dev/sda8
/dev/sda8: UUID="7d2bfcda-988d-4310-b34b-72f2277f3584" TYPE="swap"
> 
> 
</blockquote>




可以看见，/dev/sda8已经是swap文件系统了，但是光创建出来还不够，和其他文件系统一样，swap分区也需要挂载之后才能使用，但是swap分区的挂载方式和其他文件系统有所区别，使用的是**swapon**命令。





## 使用swap分区




### 启用swap分区




**swapon [OPTIONS...] specialfile...**




最简单的使用方法就是swapon DEVICE，激活DEVICE，使其能够为换页（paging）和交换（swapping）服务。





<blockquote>

> 
> [root@edu data]# swapon /dev/sda8
> 
> 
</blockquote>




#### 常用选项




**-a, --all** 自动激活所有定义在/etc/fstab中的交换分区




**-p, --priority     priority** 为交换分区指定优先级，介于-1~32767之间，默认没有优先级（-1）。优先级高的交换分区会被优先使用，而优先级相同的交换分区之间会采用轮询（Round Robin）的方式使用交换分区。





### 禁用swap分区




**swapoff [OPTIONS...] specialfile...**




和swapon命令一样，可以直接使用swapoff DEVICE，禁用DEVICE，使其不对外提供服务。





<blockquote>

> 
> [root@edu data]# swapoff /dev/sda8
> 
> 
</blockquote>




#### 常用选项




**-a, --all** 自动禁用所有出现在**/proc/swaps**中的交换分区





### 查看激活的swap分区情况




**NAME**
free - Display amount of free and used memory in the system




**SYNOPSIS**




**free [OPTIONS...]**




free命令可以用来查看内存的使用和空闲情况，由于swap分区属于虚拟内存的一部分，所以也可以查看。




最简单的使用方法就是free不加任何选项。但是由于默认free会以KB为单位显示大小，不方便阅读，所以经常会使用**-m**（MB）,**-g**（GB），或者使用**-h**（human-readable）的方式显示。




![](http://118.25.17.78/wp-content/uploads/2018/05/free.jpg)
