---
author: edmund
comments: true
date: 2018-05-10 14:04:14+00:00
layout: post
link: http://118.25.17.78/blog/2018/05/10/linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bbtrfs%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f/
slug: linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bbtrfs%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f
title: Linux文件系统管理之btrfs文件系统
wordpress_id: 303
categories:
- Linux技术
post_format:
- 日志
tags:
- linux
- 文件系统管理
---

# btrfs简介




**Btrfs**（B-tree 文件系统，通常念成 **Butter FS**，**Better FS**或**B-tree FS**），是由Oracle公司于2007年宣布开发的开源文件系统，遵循GPL协议，并且在2014年8月宣布发布稳定版。开发Btrfs其实并不是突发奇想的，而是随着时间的积累，人们越发感觉到主宰Linux文件系统的旧时代产物ext系列文件系统在面对新时代时的缺陷，比如最新的ext4文件系统并不支持多设备管理，文件系统不能跨设备创建，虽然可以通过lvm（btrfs的有些功能与lvm实现的功能非常类似，对lvm不了解但是感兴趣的可以去看看我的另一篇博文 [逻辑卷管理器（LVM）](http://118.25.17.78/topics/266)）来实现，但是终归借了他人之手，而且lvm的操作并不简单，要上手还是需要一些功夫的。




所以由于种种原因，Btrfs应运而生，可以这么说，Btrfs带来的新特性就是ext系列文件系统在当今所不足的地方。





# btrfs核心特性




## 多物理卷支持




Btrfs文件系统支持动态添加和删除存储设备，这也就意味着btrfs支持创建在多个存储设备之上。而且在创建好btrfs文件系统后，btrfs会将存储空间以chunk为单位进行划分，这就使得btrfs能够为不同的chunk指定不同的磁盘空间使用策略。比如一些chunk用来存储数据，一些chunk用来存储元数据，而一些chunk可以配置成存储mirror（RAID 1），一些chunk可以配置成存储stripe（条带）。在用户新加了硬盘后，可以通过管理工具将硬盘加入到指定的btrfs中，并且btrfs的扩容是在线扩容，不需要将文件系统卸载。由于btrfs本身支持多设备，所以很自然的，btrfs也支持RAID，而且这个RAID不需要用户来手动配置，只要指定了RAID的级别，btrfs自己会完成其余的工作。





## 子卷（subvolume）




说到多物理卷支持就不得不说子卷了，子卷可以说是btrfs最为亮眼的一个特性，子卷就是创建在btrfs上的子文件系统，也就是说，将btrfs的一部分配置成一个独立的文件系统。子卷与父卷共享底层的存储空间，需要存储空间时，子卷就会向btrfs申请使用空间，就像malloc一样，而这个底层的存储空间可以称为存储池（pooling），这也是btrfs提供的新特性。说到动态管理文件系统空间，就不得不再次提到lvm了，lvm虽然也可以实现让lv动态的增长空间，但是，在lv空间不足时，系统管理员必须手动为lv分配更多的空间供lv使用，即使vg中仍然有空闲的空间，lv也无法自己完成扩容的任务。而btrfs中的子卷则不同，由于采用了池的概念，子卷的空间都是按需申请的，所以除非btrfs的空间使用完了，否则子卷不会出现存储空间不足的情况。





## 写时复制（COW）




所谓COW，就是在用户对一个数据发生了写操作的时候，改动不会被立即写入该数据所在空间，而是会先新开辟一块存储空间，将改动后的数据写入到新的存储空间中，然后让文件系统中指向该数据的指针重新指向新开辟的地址。这有什么好处呢？ext文件系统有一个通病，那就是数据一旦被删除就难以找回，而COW由于仍然保留着原来的数据，只是数据结构中的指针改变了指向的地址，所以很容易就能够恢复数据。





## 校验码（checksum）




在从ext文件系统中读取数据的时候，假如block中存储的是0011，而你读出来却变成了1011，这是完全有可能发生的，但是ext文件系统却不会告诉你这个数据是错误的，反而告诉你数据读取成功，所以上层调用者就无法发现数据是错的，让数据的可靠性无法得到保证。而btrfs在存储数据时，在每一个数据存储前都会计算这个数据的校验和（checksum），然后把数据和校验和一并存储下来，在下次读取的时候，会在返回数据之前计算一下数据的校验和，如果和存储的校验和一致则表示数据正常，那么就返回数据，如果校验和不一致，btrfs就会尝试读取这个数据的副本，如果副本也没有，就会返回数据错误，上层调用者就会知道数据错误，从而能够采取相应的补救措施。





## 快照（snapshot）




快照就是文件系统在某一时刻的完全备份，快照一旦创建，以后无论何时访问快照，访问到的都是在创建快照那一刻的数据。在业务进行的过程中，由于快照的创建只需要几秒钟，所以运维人员只需要将业务终止几秒钟，然后就可以恢复业务了，接下来只需要对快照做备份，就可以实现完全备份，而如果想要对大量的数据直接做完全备份，如果是在线备份，那么就会出现备份的数据不一致的情况，这一头是12点钟的数据，而那一头却是13点钟的数据，如果是离线备份，那么这个业务将要暂停几分钟到几小时不等。




而快照的实现机制其实用的就是COW。在快照卷注意到某个数据要发生写操作的时候，快照卷就会先将这个数据读出来写到另一个地方，然后再将数据写入，以后通过快照卷访问的时候，就会访问到被复制出来的那个数据，也就是做快照时的数据了。





## 透明压缩




btrfs在存储数据时，会先偷偷将数据做一次压缩操作，然后才会存入到硬盘中，然后在下次读取的时候，也会先偷偷将数据解压缩，然后再返回数据。而这一切对用户来说都是透明的，用户不会发觉到存储的数据被做了压缩，这一切都是btrfs在背后偷偷完成的。看起来由于在存储数据时需要额外的压缩操作，使得浪费了一些处理器时钟周期，但是考虑到磁盘I/O的速度远比cpu要慢，而假设一个数据的读取需要100次磁盘I/O,而在花费了一些处理器时钟周期后，只需要10次磁盘I/O就能够将数据读取出来，这个带来的效益可以说是极大的，因为处理器时间和磁盘I/O的时间比起来实在是微不足道。当然这个还要取决于压缩算法和压缩率。




当然btrfs还有许多的特性没有说，这里就不一一进行讲解了，有兴趣了解的可以去看看这一篇博客 [新一代 Linux 文件系统 btrfs 简介](https://www.ibm.com/developerworks/cn/linux/l-cn-btrfs/index.html)，讲的非常透彻优雅。





# btrfs文件系统的基本使用




## btrfs文件系统管理




btrfs命令是管理btrfs文件系统的工具箱，这个命令下包含了许多个子命令，不同的子命令完成不同的功能，比如子卷管理、设备管理等。




**命令用法**




**btrfs [COMMAND] [ARGS...] : COMMAND为子命令，ARGS为子命令的一些选项和参数。**




btrfs文件系统管理的子命令为filesystem。





### btrfs文件系统创建




**NAME**




mkfs.btrfs - create a btrfs filesystem




**DESCRIPTION**




mkfs.btrfs is used to create the btrfs filesystem on a single or multiple devices. <device> is typically a block device but can be a file-backed image as well.
Multiple devices are grouped by UUID of the filesystem.




mkfs.btrfs命令用于为一个或多个设备创建btrfs文件系统，设备可以是块设备或者是本地回环设备。





#### 常用选项




**-L|--label <string>**: 指定文件系统的卷标




**-d|--data <profile>**: 指定数据存储的组织形式。有效的组织形式为: raid0, raid1, raid5, raid6, raid10 or single or dup




**-m|--metadata <profile>**: 指定元数据存储的组织形式。有效的组织形式为:  raid0, raid1, raid5, raid6, raid10, single or dup




**-O|--features <feature1>[,<feature2>...]**: 指定文件系统要开启的特性，如果有需要关闭的特性，在特性前面加上^。如果想要查看支持的特性，可以运行：mkfs.btrfs -O list-all




**-f|--force**: 如果检测到块设备上已经有文件系统了，忽略该文件系统，直接进行覆盖。




    
    [root@edu ~]# mkfs.btrfs -L MYDATA /dev/sd{b,c}  #为/dev/sdb和/dev/sdc创建btrfs文件系统，卷标为MYDATA
    btrfs-progs v4.9.1
    See http://btrfs.wiki.kernel.org for more information.
    
    /dev/sdc appears to contain an existing filesystem (ext4).  #由于/dev/sdc上面已经存在ext4文件系统，所以无法创建
    ERROR: use the -f option to force overwrite of /dev/sdc  
    [root@edu ~]# mkfs.btrfs -f -L MYDATA /dev/sd{b,c}  #使用-f选项强制创建btrfs文件系统
    btrfs-progs v4.9.1
    See http://btrfs.wiki.kernel.org for more information.
    
    Label:              MYDATA
    UUID:               199eb28e-b42d-479e-b4fd-12894d483bba
    Node size:          16384
    Sector size:        4096
    Filesystem size:    40.00GiB
    Block group profiles:
      Data:             RAID0             2.00GiB
      Metadata:         RAID1             1.00GiB
      System:           RAID1             8.00MiB
    SSD detected:       no
    Incompat features:  extref, skinny-metadata
    Number of devices:  2
    Devices:
       ID        SIZE  PATH
        1    20.00GiB  /dev/sdb
        2    20.00GiB  /dev/sdc
    




### 




### 显示btrfs文件系统




**btrfs filesystem show [<path>|<uuid>|<device>|<label>]**




    
    [root@edu ~]# btrfs filesystem show 
    Label: 'centos'  uuid: c340f1a3-e07e-40e8-8ecc-29ebd0480fe0
    	Total devices 1 FS bytes used 1.27GiB
    	devid    1 size 55.88GiB used 4.02GiB path /dev/sda2
    
    Label: 'MYDATA'  uuid: 199eb28e-b42d-479e-b4fd-12894d483bba
    	Total devices 2 FS bytes used 112.00KiB
    	devid    1 size 20.00GiB used 2.01GiB path /dev/sdb
    	devid    2 size 20.00GiB used 2.01GiB path /dev/sdc
    
    Label: none  uuid: c6a0f0f7-3f24-4e37-9a65-6449d8a7e08f
    	Total devices 1 FS bytes used 112.00KiB
    	devid    1 size 20.00GiB used 2.02GiB path /dev/sda5
    




### 




### **显示btrfs文件系统空间使用情况**




**btrfs filesystem usage usage <path> [<path>...]**




    
    [root@edu ~]# mount /dev/sdb /mydata/  #挂载后才能使用下面的命令
    [root@edu ~]# btrfs filesystem usage /mydata/
    Overall:
        Device size:		  40.00GiB
        Device allocated:		   4.02GiB
        Device unallocated:		  35.98GiB
        Device missing:		     0.00B
        Used:			   1.00MiB
        Free (estimated):		  37.98GiB	(min: 19.99GiB)
        Data ratio:			      1.00
        Metadata ratio:		      2.00
        Global reserve:		  16.00MiB	(used: 0.00B)
    
    Data,RAID0: Size:2.00GiB, Used:768.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    Metadata,RAID1: Size:1.00GiB, Used:112.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    System,RAID1: Size:8.00MiB, Used:16.00KiB
       /dev/sdb	   8.00MiB
       /dev/sdc	   8.00MiB
    
    Unallocated:
       /dev/sdb	  17.99GiB
       /dev/sdc	  17.99GiB
    




### 




### 改变btrfs的存储空间大小




**btrfs filesystem resize [<devid>:][+/-]<size>[kKmMgGtTpPeE]|[<devid>:]max <path>**




    
    [root@edu ~]# btrfs filesystem resize -5G /mydata/  #将MYDATA的存储空间减小5G，默认会减小第一个存储设备的空间，此处为/dev/sdb
    Resize '/mydata/' of '-5G'
    [root@edu ~]# btrfs filesystem usage /mydata/  #查看MYDATA的存储空间，果然少了5G，且为/dev/sdb
    Overall:
        Device size:		  35.00GiB
        Device allocated:		   4.02GiB
        Device unallocated:		  30.98GiB
        Device missing:		     0.00B
        Used:			   1.00MiB
        Free (estimated):		  32.98GiB	(min: 17.49GiB)
        Data ratio:			      1.00
        Metadata ratio:		      2.00
        Global reserve:		  16.00MiB	(used: 0.00B)
    
    Data,RAID0: Size:2.00GiB, Used:768.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    Metadata,RAID1: Size:1.00GiB, Used:112.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    System,RAID1: Size:8.00MiB, Used:16.00KiB
       /dev/sdb	   8.00MiB
       /dev/sdc	   8.00MiB
    
    Unallocated:
       /dev/sdb	  12.99GiB
       /dev/sdc	  17.99GiB
    [root@edu ~]# btrfs filesystem show  #通过show命令可以看到/dev/sdc的devid为2
    Label: 'centos'  uuid: c340f1a3-e07e-40e8-8ecc-29ebd0480fe0
    	Total devices 1 FS bytes used 1.27GiB
    	devid    1 size 55.88GiB used 4.02GiB path /dev/sda2
    
    Label: 'MYDATA'  uuid: 199eb28e-b42d-479e-b4fd-12894d483bba
    	Total devices 2 FS bytes used 896.00KiB
    	devid    1 size 15.00GiB used 2.01GiB path /dev/sdb
    	devid    2 size 20.00GiB used 2.01GiB path /dev/sdc
    
    Label: none  uuid: c6a0f0f7-3f24-4e37-9a65-6449d8a7e08f
    	Total devices 1 FS bytes used 112.00KiB
    	devid    1 size 20.00GiB used 2.02GiB path /dev/sda5
    
    [root@edu ~]# btrfs filesystem resize 2:-5G /mydata/  #该命令也可以通过devid制定某个设备改变存储空间
    Resize '/mydata/' of '2:-5G'
    [root@edu ~]# btrfs filesystem usage /mydata/  #果然，/dev/sdc的存储空间也减小了5G
    Overall:
        Device size:		  30.00GiB
        Device allocated:		   4.02GiB
        Device unallocated:		  25.98GiB
        Device missing:		     0.00B
        Used:			   1.00MiB
        Free (estimated):		  27.98GiB	(min: 14.99GiB)
        Data ratio:			      1.00
        Metadata ratio:		      2.00
        Global reserve:		  16.00MiB	(used: 0.00B)
    
    Data,RAID0: Size:2.00GiB, Used:768.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    Metadata,RAID1: Size:1.00GiB, Used:112.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    System,RAID1: Size:8.00MiB, Used:16.00KiB
       /dev/sdb	   8.00MiB
       /dev/sdc	   8.00MiB
    
    Unallocated:
       /dev/sdb	  12.99GiB
       /dev/sdc	  12.99GiB
    [root@edu ~]# btrfs filesystem resize max /mydata/  #max命令可以将存储空间加到最大，默认增加第一个设备的存储空间，此处为/dev/sdb
    Resize '/mydata/' of 'max'
    [root@edu ~]# btrfs filesystem usage /mydata/  #果然，/dev/sdb的存储空间恢复了，且总空间也增加了5G。
    Overall:
        Device size:		  35.00GiB
        Device allocated:		   4.02GiB
        Device unallocated:		  30.98GiB
        Device missing:		     0.00B
        Used:			   1.00MiB
        Free (estimated):		  32.98GiB	(min: 17.49GiB)
        Data ratio:			      1.00
        Metadata ratio:		      2.00
        Global reserve:		  16.00MiB	(used: 0.00B)
    
    Data,RAID0: Size:2.00GiB, Used:768.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    Metadata,RAID1: Size:1.00GiB, Used:112.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    System,RAID1: Size:8.00MiB, Used:16.00KiB
       /dev/sdb	   8.00MiB
       /dev/sdc	   8.00MiB
    
    Unallocated:
       /dev/sdb	  17.99GiB
       /dev/sdc	  12.99GiB
    [root@edu ~]# btrfs filesystem resize 2:max /mydata/  #max命令也可以制定某个设备
    Resize '/mydata/' of '2:max'
    [root@edu ~]# btrfs filesystem usage /mydata/  #/dev/sdc也恢复了
    Overall:
        Device size:		  40.00GiB
        Device allocated:		   4.02GiB
        Device unallocated:		  35.98GiB
        Device missing:		     0.00B
        Used:			   1.00MiB
        Free (estimated):		  37.98GiB	(min: 19.99GiB)
        Data ratio:			      1.00
        Metadata ratio:		      2.00
        Global reserve:		  16.00MiB	(used: 0.00B)
    
    Data,RAID0: Size:2.00GiB, Used:768.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    Metadata,RAID1: Size:1.00GiB, Used:112.00KiB
       /dev/sdb	   1.00GiB
       /dev/sdc	   1.00GiB
    
    System,RAID1: Size:8.00MiB, Used:16.00KiB
       /dev/sdb	   8.00MiB
       /dev/sdc	   8.00MiB
    
    Unallocated:
       /dev/sdb	  17.99GiB
       /dev/sdc	  17.99GiB
    




## 




## btrfs设备管理




btrfs设备管理的子命令为device。





### 为btrfs添加新设备




**btrfs device add [-Kf] <dev> [<dev>...] <path>**




    
    [root@edu ~]# btrfs device add /dev/sdd /mydata/  #为MYDATA添加一个新设备/dev/sdd
    [root@edu ~]# btrfs filesystem show  #查看添加后的结果，多出了一个设备
    Label: 'centos'  uuid: c340f1a3-e07e-40e8-8ecc-29ebd0480fe0
    	Total devices 1 FS bytes used 1.27GiB
    	devid    1 size 55.88GiB used 4.02GiB path /dev/sda2
    
    Label: 'MYDATA'  uuid: 199eb28e-b42d-479e-b4fd-12894d483bba
    	Total devices 3 FS bytes used 896.00KiB
    	devid    1 size 20.00GiB used 2.01GiB path /dev/sdb
    	devid    2 size 20.00GiB used 2.01GiB path /dev/sdc
    	devid    3 size 20.00GiB used 0.00B path /dev/sdd
    
    Label: none  uuid: c6a0f0f7-3f24-4e37-9a65-6449d8a7e08f
    	Total devices 1 FS bytes used 112.00KiB
    	devid    1 size 20.00GiB used 2.02GiB path /dev/sda5
    




### 




### 为btrfs移除设备




**btrfs device remove <dev>|<devid> [<dev>|<devid>...] <path>**




    
    [root@edu ~]# btrfs device remove /dev/sdd /mydata/  #为MYDATA移除设备/dev/sdd
    [root@edu ~]# btrfs filesystem show
    Label: 'centos'  uuid: c340f1a3-e07e-40e8-8ecc-29ebd0480fe0
    	Total devices 1 FS bytes used 1.27GiB
    	devid    1 size 55.88GiB used 4.02GiB path /dev/sda2
    
    Label: 'MYDATA'  uuid: 199eb28e-b42d-479e-b4fd-12894d483bba
    	Total devices 2 FS bytes used 896.00KiB
    	devid    1 size 20.00GiB used 2.01GiB path /dev/sdb
    	devid    2 size 20.00GiB used 2.01GiB path /dev/sdc
    
    Label: none  uuid: c6a0f0f7-3f24-4e37-9a65-6449d8a7e08f
    	Total devices 1 FS bytes used 112.00KiB
    	devid    1 size 20.00GiB used 2.02GiB path /dev/sda5
    




### 




### 显示btrfs中指定设备或所有设备的I/O error的统计数据




**btrfs device stats [options] <path>|<device>**




    
    [root@edu ~]# btrfs device stats /mydata/  #显示MYDATA下所有设备的I/O error的统计数据
    [/dev/sdb].write_io_errs    0
    [/dev/sdb].read_io_errs     0
    [/dev/sdb].flush_io_errs    0
    [/dev/sdb].corruption_errs  0
    [/dev/sdb].generation_errs  0
    [/dev/sdc].write_io_errs    0
    [/dev/sdc].read_io_errs     0
    [/dev/sdc].flush_io_errs    0
    [/dev/sdc].corruption_errs  0
    [/dev/sdc].generation_errs  0
    [root@edu ~]# btrfs device stats /dev/sdc #显示/dev/sdc设备的I/O error的统计数据
    [/dev/sdc].write_io_errs    0
    [/dev/sdc].read_io_errs     0
    [/dev/sdc].flush_io_errs    0
    [/dev/sdc].corruption_errs  0
    [/dev/sdc].generation_errs  0
    
    




### 




### 显示btrfs中各个设备的空间分配情况




**btrfs device usage [options] <path> [<path>...]**




    
    [root@edu ~]# btrfs device usage /mydata/
    /dev/sdb, ID: 1
       Device size:            20.00GiB
       Device slack:              0.00B
       Data,RAID0:              1.00GiB
       Metadata,RAID1:          1.00GiB
       System,RAID1:            8.00MiB
       Unallocated:            17.99GiB
    
    /dev/sdc, ID: 2
       Device size:            20.00GiB
       Device slack:              0.00B
       Data,RAID0:              1.00GiB
       Metadata,RAID1:          1.00GiB
       System,RAID1:            8.00MiB
       Unallocated:            17.99GiB
    




## 




## btrfs存储均衡管理




btrfs存储均衡的子命令为balance。该命令用于使数据平均分布于各个设备中，不至于让某些设备负担过重。





### 开始存储均衡




**btrfs balance start [options] <path>**




    
    [root@edu ~]# btrfs balance /mydata/  #如果不使用--full-balance将会提示警告，因为balance默认会对所有数据做balance，而对大量数据来说将花费很多时间，所以建议使用一些过滤条件过滤掉一些数据。
    WARNING:
    
    	Full balance without filters requested. This operation is very
    	intense and takes potentially very long. It is recommended to
    	use the balance filters to narrow down the balanced data.
    	Use 'btrfs balance start --full-balance' option to skip this
    	warning. The operation will start in 10 seconds.
    	Use Ctrl-C to stop it.
    10 9 8 7 6^C
    [root@edu ~]# btrfs balance --full-balance /mydata/
    Done, had to relocate 3 out of 3 chunks
    





由于下面的命令没有足够的数据可以跑所以无法示范，但是用法很简单，只需要指定文件系统即可。





### 




### 暂停存储均衡




**btrfs balance pause <path>**





### 继续存储均衡




**btrfs balance resume <path>**





### 取消存储均衡




**btrfs balance cancel <path>**





## btrfs子卷管理




btrfs子卷管理的子命令为subvolume。





### 为btrfs创建子卷




**btrfs subvolume create [-i <qgroupid>] [<dest>/]<name>**




    
    [root@edu ~]# btrfs subvolume create /mydata/logs  #为MYDATA创建子卷logs
    Create subvolume '/mydata/logs'
    [root@edu ~]# cd /mydata/  #发现MYDATA中多了一个logs目录，这个就是子卷所在的目录
    [root@edu mydata]# ls
    logs
    




### 




### 为btrfs删除子卷




**btrfs subvolume delete [options] <subvolume> [<subvolume>...]**




    
    [root@edu mydata]# btrfs subvolume delete /mydata/logs  #为MYDATA删除logs子卷
    Delete subvolume (no-commit): '/mydata/logs'
    [root@edu mydata]# ls #发现logs目录消失了
    




### 




### 显示btrfs下的所有子卷




**btrfs subvolume list [options] [-G [+|-]<value>] [-C [+|-]<value>] [--sort=rootid,gen,ogen,path] <path>**




    
    [root@edu mydata]# btrfs subvolume create /mydata/logs #为MYDATA创建子卷logs
    Create subvolume '/mydata/logs'
    [root@edu mydata]# btrfs subvolume create /mydata/cache  #为MYDATA创建子卷cache
    Create subvolume '/mydata/cache'
    [root@edu mydata]# btrfs subvolume list /mydata #观察所有子卷，可以看见第一列为子卷ID，而最后一列为子卷名
    ID 261 gen 32 top level 5 path logs
    ID 262 gen 33 top level 5 path cache
    [root@edu mydata]# ls -l #果然MYDATA下多出了两个目录logs和cache
    total 0
    drwxr-xr-x. 1 root root 0 May 10 20:44 cache
    drwxr-xr-x. 1 root root 0 May 10 20:44 logs
    





### 显示btrfs下的指定子卷的详细信息




**btrfs subvolume show <path>**




    
    [root@edu mydata]# btrfs subvolume show /mydata/logs #显示logs子卷的详细信息
    /mydata/logs
    	Name: 			logs
    	UUID: 			4cc4e615-0854-a14c-be25-f633a2588640
    	Parent UUID: 		-
    	Received UUID: 		-
    	Creation time: 		2018-05-10 20:44:48 +0800
    	Subvolume ID: 		261
    	Generation: 		32
    	Gen at creation: 	32
    	Parent ID: 		5
    	Top level ID: 		5
    	Flags: 			-
    	Snapshot(s):
    [root@edu mydata]# btrfs subvolume show /mydata/cache  #显示cache子卷的详细信息
    /mydata/cache
    	Name: 			cache
    	UUID: 			d071cb9e-ebec-b041-83c9-095a77d72ba3
    	Parent UUID: 		-
    	Received UUID: 		-
    	Creation time: 		2018-05-10 20:44:51 +0800
    	Subvolume ID: 		262
    	Generation: 		33
    	Gen at creation: 	33
    	Parent ID: 		5
    	Top level ID: 		5
    	Flags: 			-
    	Snapshot(s):
    





### 单独挂载子卷




在之前，我们发现子卷直接被创建在父卷下面，就是说一旦挂载了父卷，所有子卷将一并被挂载，如果想要单独挂载某一个子卷的话，可以使用mount命令指定挂载选项。




    
    [root@edu mydata]# cp /etc/fstab /mydata/logs/ #将fstab复制到logs子卷中
    [root@edu mydata]# ls /mydata/logs/ #观察是否复制成功
    fstab
    [root@edu mydata]# mount -o subvol=logs /dev/sdb /mnt  #使用 -o subvol=xxx选项指定需要挂载的子卷，需要挂载的设备可以使用btrfs中的任意一个设备
    [root@edu mydata]# ls /mnt/  #观察子卷被挂载的目录/mnt,发现fstab，证明子卷挂载成功
    fstab
    





### 在btrfs下为某个子卷创建快照




**btrfs subvolume snapshot [-r] <source> <dest>|[<dest>/]<name>**




    
    [root@edu mydata]# btrfs subvolume snapshot /mydata/logs/ /mydata/logs_snapshot  #为logs子卷创建快照logs_snapshot
    Create a snapshot of '/mydata/logs/' in '/mydata/logs_snapshot'
    [root@edu mydata]# btrfs subvolume list /mydata #发现MYDATA下多出了一个子卷
    ID 261 gen 35 top level 5 path logs
    ID 262 gen 33 top level 5 path cache
    ID 263 gen 35 top level 5 path logs_snapshot
    [root@edu mydata]# mount -o subvol=logs_snapshot /dev/sdb /opt/  #将快照卷挂载到/opt目录下
    [root@edu mydata]# ls /opt/ #发现/opt目录下也有fstab，证明快照成功
    fstab
    [root@edu mydata]# echo edmund.newline >> /mnt/fstab #在logs子卷中修改fstab的内容，在末尾添加一行edmund.newline
    [root@edu mydata]# cat /mnt/fstab #观察是否添加成功
    
    #
    # /etc/fstab
    # Created by anaconda on Thu Apr 19 18:45:35 2018
    #
    # Accessible filesystems, by reference, are maintained under '/dev/disk'
    # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
    #
    UUID=c340f1a3-e07e-40e8-8ecc-29ebd0480fe0 /                       btrfs   subvol=root     0 0
    UUID=616bbfe3-4356-4c91-a0a9-86c83fc097de /boot                   xfs     defaults        0 0
    UUID=c340f1a3-e07e-40e8-8ecc-29ebd0480fe0 /home                   btrfs   subvol=home     0 0
    UUID=f0b1e78c-38fb-4036-afcb-9857b31498fd swap                    swap    defaults        0 0
    /dev/sr0				/media			iso9660	  defaults,ro	  0 0	 
    edmund.newline
    [root@edu mydata]# cat /opt/fstab  #观察快照卷中的fstab，发现没有变化，证明快照成功
    
    #
    # /etc/fstab
    # Created by anaconda on Thu Apr 19 18:45:35 2018
    #
    # Accessible filesystems, by reference, are maintained under '/dev/disk'
    # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
    #
    UUID=c340f1a3-e07e-40e8-8ecc-29ebd0480fe0 /                       btrfs   subvol=root     0 0
    UUID=616bbfe3-4356-4c91-a0a9-86c83fc097de /boot                   xfs     defaults        0 0
    UUID=c340f1a3-e07e-40e8-8ecc-29ebd0480fe0 /home                   btrfs   subvol=home     0 0
    UUID=f0b1e78c-38fb-4036-afcb-9857b31498fd swap                    swap    defaults        0 0
    /dev/sr0				/media			iso9660	  defaults,ro	  0 0




## 




## 开启透明压缩机制




开启btrfs的透明压缩机制只需要在挂载时指定压缩选项即可。




**mount -o compress={lzo|zlib} DEVICE DEST**




    
    [root@edu ~]# mount -o compress=lzo /dev/sdc /mydata/  #为MYDATA开启透明压缩
    [root@edu ~]# mount | tail -1  #观察挂载选项，发现多出了一个compress选项，证明开启了透明压缩
    /dev/sdb on /mydata type btrfs (rw,relatime,seclabel,compress=lzo,space_cache,subvolid=5,subvol=/)








## btrfs与ext系列文件系统的无损转换




**NAME**




btrfs-convert - convert from ext2/3/4 filesystem to btrfs in-place




**DESCRIPTION**




btrfs-convert is used to convert existing ext2/3/4 filesystem image to a btrfs filesystem
in-place. The original filesystem image is accessible subvolume named ext2_saved as file
image.




btrfs-convert命令可以将ext系列的文件系统无损转换为btrfs文件系统，且保留其中的所有文件。由于btrfs自研发之初就是为了弥补ext系列文件系统的不足，btrfs-convert命令可以让ext系列文件系统与btrfs文件系统很好的共存。





### 命令用法




**btrfs-convert DEVICE** 将DEVICE中的ext系列文件系统转换为btrfs文件系统




**btrfs-convert -r DEVICE**  将已经被转换为btrfs文件系统的设备恢复到原本的ext系列文件系统








### 将ext4文件系统转换为btrfs文件系统




需要注意的是，一旦将ext文件系统转换为btrfs文件系统后，千万不要使用btrfs balance指令，这将会导致文件系统无法恢复到ext文件系统。




    
    [root@edu ~]# mkfs.ext4 /dev/sdb #将/dev/sdb创建为ext4文件系统
    mke2fs 1.42.9 (28-Dec-2013)
    /dev/sdb is entire device, not just one partition!
    Proceed anyway? (y,n) y
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    1310720 inodes, 5242880 blocks
    262144 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=2153775104
    160 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks: 
    	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
    	4096000
    
    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done   
    
    [root@edu ~]# blkid /dev/sdb  #观察/dev/sdb的文件系统类型，为ext4
    /dev/sdb: UUID="9d70dea0-93ac-42bb-9d0b-102d1c323a07" TYPE="ext4" 
    [root@edu ~]# mount /dev/sdb /mnt/  #将/dev/sdb挂载到/mnt目录下
    [root@edu ~]# cp -a /etc/pam.d/ /mnt/  #将/etc/pam.d/ 目录复制到/mnt中
    [root@edu ~]# ls -l /mnt/ 观察/mnt下的文件，发现了lost+found和pam.d
    total 20
    drwx------. 2 root root 16384 May 11 10:18 lost+found
    drwxr-xr-x. 2 root root  4096 Apr 19 18:51 pam.d
    [root@edu ~]# ls -l /mnt/pam.d/  #观察/mnt/pam.d目录下的文件，发现存在文件
    total 100
    -rw-r--r--. 1 root root  192 Aug  4  2017 chfn
    -rw-r--r--. 1 root root  192 Aug  4  2017 chsh
    -rw-r--r--. 1 root root  232 Nov  6  2016 config-util
    -rw-r--r--. 1 root root  293 Aug  3  2017 crond
    lrwxrwxrwx. 1 root root   19 Apr 19 18:51 fingerprint-auth -> fingerprint-auth-ac
    -rw-r--r--. 1 root root  702 Apr 19 18:51 fingerprint-auth-ac
    -rw-r--r--. 1 root root  796 Aug  4  2017 login
    -rw-r--r--. 1 root root  154 Nov  6  2016 other
    -rw-r--r--. 1 root root  188 Jun 10  2014 passwd
    lrwxrwxrwx. 1 root root   16 Apr 19 18:51 password-auth -> password-auth-ac
    -rw-r--r--. 1 root root 1033 Apr 19 18:51 password-auth-ac
    -rw-r--r--. 1 root root  155 May 26  2017 polkit-1
    lrwxrwxrwx. 1 root root   12 Apr 19 18:51 postlogin -> postlogin-ac
    -rw-r--r--. 1 root root  330 Apr 19 18:51 postlogin-ac
    -rw-r--r--. 1 root root  681 Aug  4  2017 remote
    -rw-r--r--. 1 root root  143 Aug  4  2017 runuser
    -rw-r--r--. 1 root root  138 Aug  4  2017 runuser-l
    lrwxrwxrwx. 1 root root   17 Apr 19 18:51 smartcard-auth -> smartcard-auth-ac
    -rw-r--r--. 1 root root  752 Apr 19 18:51 smartcard-auth-ac
    lrwxrwxrwx. 1 root root   25 Apr 19 18:47 smtp -> /etc/alternatives/mta-pam
    -rw-r--r--. 1 root root   76 Jun 10  2014 smtp.postfix
    -rw-r--r--. 1 root root  904 Aug  7  2017 sshd
    -rw-r--r--. 1 root root  540 Aug  4  2017 su
    -rw-r--r--. 1 root root  202 Aug  4  2017 sudo
    -rw-r--r--. 1 root root  187 Aug  4  2017 sudo-i
    -rw-r--r--. 1 root root  137 Aug  4  2017 su-l
    lrwxrwxrwx. 1 root root   14 Apr 19 18:51 system-auth -> system-auth-ac
    -rw-r--r--. 1 root root 1031 Apr 19 18:51 system-auth-ac
    -rw-r--r--. 1 root root  129 Aug  5  2017 systemd-user
    -rw-r--r--. 1 root root   84 Aug  2  2017 vlock
    -rw-r--r--. 1 root root  278 Aug  8  2017 vmtoolsd
    [root@edu ~]# umount /dev/sdb   #将/dev/sdb卸载
    [root@edu ~]# btrfs-convert /dev/sdb  #将/dev/sdb转换为btrfs文件系统
    create btrfs filesystem:
    	blocksize: 4096
    	nodesize:  16384
    	features:  extref, skinny-metadata (default)
    creating ext2 image file
    creating btrfs metadatacopy inodes [o] [         0/        43]
    conversion complete[root@edu ~]# 
    [root@edu ~]# blkid /dev/sdb   #观察/dev/sdb的文件系统类型，发现已经是btrfs文件系统了
    /dev/sdb: UUID="16c49aca-0a1a-46ad-88fc-e541de9c12b7" UUID_SUB="5814a680-6c42-445d-a6a2-b0dce40bb716" TYPE="btrfs" 
    [root@edu ~]# mount /dev/sdb /mnt/ 将/dev/sdb重新挂载到/mnt下
    [root@edu ~]# ls /mnt/  观察/mnt目录，发现多出了一个ext2_saved，这里面保存的是原来ext系列文件系统的一些元数据，用于恢复回ext系列文件系统，请不要动这个目录
    ext2_saved  lost+found  pam.d
    [root@edu ~]# ls /mnt/pam.d/  #观察/mnt/pam.d目录，发现文件仍然存在，转换成功
    chfn                 login             postlogin       smartcard-auth-ac  sudo-i          vmtoolsd
    chsh                 other             postlogin-ac    smtp               su-l
    config-util          passwd            remote          smtp.postfix       system-auth
    crond                password-auth     runuser         sshd               system-auth-ac
    fingerprint-auth     password-auth-ac  runuser-l       su                 systemd-user
    fingerprint-auth-ac  polkit-1          smartcard-auth  sudo               vlock
    








### 将btrfs文件系统转换为ext4文件系统



    
    [root@edu ~]# umount /dev/sdb #将/dev/sdb卸载
    [root@edu ~]# btrfs-convert -r /dev/sdb #恢复/dev/sdb的ext4文件系统
    rollback complete
    [root@edu ~]# mount /dev/sdb /mnt #将/dev/sdb重新挂载回/mnt目录
    [root@edu ~]# ls /mnt/ #观察/mnt目录，发现pam.d目录和lost+found
    lost+found  pam.d
    [root@edu ~]# ls /mnt/pam.d/  #观察/mnt/pam.d目录中的文件，发现文件依然存在
    chfn                 login             postlogin       smartcard-auth-ac  sudo-i          vmtoolsd
    chsh                 other             postlogin-ac    smtp               su-l
    config-util          passwd            remote          smtp.postfix       system-auth
    crond                password-auth     runuser         sshd               system-auth-ac
    fingerprint-auth     password-auth-ac  runuser-l       su                 systemd-user
    fingerprint-auth-ac  polkit-1          smartcard-auth  sudo               vlock
    [root@edu ~]# blkid /dev/sdb  #观察/dev/sdb文件系统，发现已经恢复到了ext4文件系统
    /dev/sdb: UUID="9d70dea0-93ac-42bb-9d0b-102d1c323a07" TYPE="ext4"



