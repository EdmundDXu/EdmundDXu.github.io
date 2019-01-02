---
author: edmund
comments: true
date: 2018-07-02 02:05:00+00:00
layout: post
link: http://118.25.17.78/blog/2018/07/02/linux%e7%b3%bb%e7%bb%9f%e5%90%af%e5%8a%a8%e6%b5%81%e7%a8%8b%e4%b9%8bcentos%e7%af%87/
slug: linux%e7%b3%bb%e7%bb%9f%e5%90%af%e5%8a%a8%e6%b5%81%e7%a8%8b%e4%b9%8bcentos%e7%af%87
title: Linux系统启动流程之CentOS篇
wordpress_id: 441
categories:
- Linux随笔
post_format:
- 日志
tags:
- BIOS
- bootloader
- CentOS 5
- CentOS 6
- CentOS7
- grub
- init
- initdisk
- kernel
- linux
- MBR
---

# 前言




在你按下计算机的电源键时，不知道你有没有想过，在显示屏中快速闪过的信息的背后到底发生着什么？事实上，一个操作系统的启动过程是非常复杂的，其中涉及到太多方面的问题，所以如果要把它完全搞懂恐怕是非常困难的。本文只对Linux系统启动流程的各个部分做概述，在适当的地方进行稍加深入的解释，只求做到入门。







# Linux系统启动流程简介




首先我们在这里讲的Linux其实是Linux内核，所以Linux系统启动流程其实是Linux内核启动流程，之所以不说操作系统是因为现代的操作系统实际上不仅仅包含内核，因为只有内核的操作系统是无法提供任何生产价值的（能提供生产价值的是跑在内核之上的应用程序），这种操作系统根本不能称为是操作系统。




其次，我们知道，Linux内核实际上是一个庞大的应用程序，也就是说，如果要启动Linux的话，就需要让Linux内核运行起来，而我们要如何让一个应用程序运行起来呢？以我们一般的方式，都是通过一个SHELL界面通知内核，然后由内核将程序加载进内存中，指挥CPU运行程序的指令。发现问题了吗，要运行一个程序的话，首先我们需要有一个内核帮助，但是我们的内核都还没有运行起来，难道需要让内核自己帮助自己？




当然了，内核是不可能帮助自己启动的，所以必须有人从旁辅助才行，这个贵人就是bootloader。bootloader是一段指令，能够指挥着计算机将内核加载进内存并让内核运行起来，好了，下面问题来了，bootloader是由谁加载的？实际上，bootloader位于磁盘的MBR（主引导记录，Master Boot Record）中，而MBR位于启动盘的第0磁道第0扇区的前512字节中。MBR的前447个字节存放的就是bootloader指令，后面64字节存放的是分区表，而最后两个字节则存放的是此MBR是否可用的标志，如果最后两个字节存放的是0x55和0xAA，那么就表示这个MBR是有效的，否则这个MBR就是无效的。




现在我们知道了bootloader的位置，那么只需要直接读取bootloader就行了，但是是谁读取的bootloader呢？而且又是怎么知道读取哪块硬盘的bootloader？这就又要一个贵人相助了，它就是BIOS（基本输入输出系统，Basic Input/Output System）。BIOS程序可以通过提前设置好的BootSequence（启动次序）来选择读取硬盘的MBR。




到了这里，你是不是在想BIOS是由谁加载的？其实问题到这里就被终结了，怎么解决的呢？在早期工程师们在面对着这个问题的时候非常头痛，于是想方设法把一段程序放到内存中去，但是我们知道内存是易失性存储，一断电就丢了，于是他们就把程序放到了ROM（Read-Only Memory，只读存储器）中，让计算机一启动就去ROM中读取这段程序，而这段程序就是BIOS程序。好了，问题解决！




## **让我们按顺序总结一下Linux系统的启动流程 : **




**1、**按下电源键后，计算机首先会去ROM中加载BIOS，在接管计算机后，BIOS程序首先检查，计算机硬件能否满足运行的基本条件，并且为驱动程序的运行提供环境，这叫做POST（加电自检，Power On Self Test）。如果硬件出现问题，主板会发出不同含义的蜂鸣，启动中止。




**2、**在POST结束后，如果没有问题，BIOS会去CMOS（**[CMOS](https://en.wikipedia.org/wiki/CMOS)为可读写RAM芯片，保存计算机基本启动信息，如时间、设备信息、启动设置等。靠钮扣电池供电，断电信息不丢失**）中保存的BootSequence（启动次序）中顺序查找启动设备，如果第一个启动设备有MBR，则读取第一个设备的MBR，否则顺序往下读取，直到读取到MBR或者扫描过所有设备为止。BootSequence可以通过BIOS界面进行配置，启动次序越靠前的设备越是优先能够获得系统的控制权。




**3、**BIOS从MBR将系统的控制权转交给bootloader，Boot Loader 选择要启动的内核，并将内核载入内存中使其运行起来。




**4、**内核启动时会探测所有硬件信息，载入适当的驱动程序来使计算机能够正常运行，并以只读方式挂载根文件系统。




**5、**内核最后会启动用户空间的第一个应用程序 init，这个程序负责管理用户空间的所有应用程序。至此，Linux系统启动初步完成。




![](https://preview.edge.edx.org/c4x/Linux/LFS101/asset/chapter03_flowchart_scr15_1.jpg)




这张图中描述了系统启动流程的各个阶段，在下面我会对各个阶段做介绍。







# Linux启动流程各阶段详述




## BIOS




前面我们提到了BIOS程序会被计算机首先加载执行，其实是因为在计算机加电后，计算机的一些寄存器会被设置初值，让指令寄存器指向 BIOS 的第一条指令，所以就能够实现开机首先加载BIOS程序。




前面还提到MBR，其实在一开始在硬盘的前512个字节找到MBR后，会先去读取MBR中的分区表（第447-510字节），然后扫描分区表中的每一个分区。分区表大小为64字节，只能容纳4个主分区，每个分区的第一个字节表示了该分区是不是激活分区，如果为0x80，就表示该分区是激活分区，控制权要转交给这个分区。四个主分区里面只能有一个是激活的。如果分区中存在拓展分区，也同样会扫描拓展分区（一般不会将操作系统安装在拓展分区）。计算机会读取激活分区的第一个扇区，叫做VBR（Volume Boot Record，卷引导记录）。VBR的主要作用是，告诉计算机，操作系统在这个分区里的位置。然后，计算机就会通过这个分区上的bootloader加载操作系统了。（关于VBR这一部分有待商榷）




 




## Boot Loader




在早期本来是将内核放在MBR的前446字节，但是显然对于内核来说446字节实在是太小了，所以就只能把内核存放在别的地方（文件系统上），但是这样一来BIOS就找不到内核了，于是人们想了个办法，在446个字节的位置写了个小程序，这个小程序的作用就是找到放在别处的内核，并且将其加载进内存，最后将系统的控制权交给内核。




目前比较主流的bootloader为grub（GRand Unified Bootloader） 和 grub2，其中grub也叫作grub legacy，而grub2是grub legacy的重构版，在CentOS 7 中使用的是grub2，而CentOS 5和6使用的是grub legacy。




grub legacy的任务分为3个阶段，分别是stage1、stage1.5和stage2，这里我围绕grub legacy的工作机制再次讲一下系统启动流程。




**1、**BIOS程序读取启动盘的MBR。




**2、**开始执行bootloader程序（grub stage 1）。




**3、**bootloader程序跳转到磁盘中的一个指定位置。这个指定位置会在grub legacy安装的时候被写入到bootloader程序代码中，并且这个位置通常指向stage 1.5所在的扇区。




**4、**stage 1.5知道/boot文件系统所在的分区，所以它会打开这个文件系统并且寻找可执行的stage 2，找到后就执行stage 2的代码。（因为stage 1.5知道/boot文件系统的位置，所以可以很容易地升级stage 2和内核等文件）




**5、**stage 2中包含了大部分的grub legacy逻辑代码，它会去加载菜单列表（如果有多个可选内核），最终会去读取指定的内核，然后将其加载进内存，同时也会初始化 RAM disk （[initrd](https://en.wikipedia.org/wiki/Initial_ramdisk) or [initramfs](https://en.wikipedia.org/wiki/Initramfs)）到内存，供内核使用，并把控制权交给内核（后面会讲到ramdisk文件系统，这里只需要知道它是一个辅助内核挂载根文件系统的文件即可）。




 




不知道你有没有注意到，上面这些过程中发生了一些read动作，首先是BIOS读取MBR，然后是stage 1读取stage1.5，stage1.5读取stage2，stage 2读取内核。下面我们来分析一下这个read动作是如何成功完成的。




首先是BIOS读取MBR，这里涉及到的是硬盘驱动，因为MBR并没有存储在文件系统上，所以BIOS只需要能够驱动硬盘就能够读取MBR，而显然BIOS中包含有硬盘驱动，所以BIOS能够读取MBR。




然后是stage 1读取stage 1.5，根据上面所述，stage 1.5存放在指定的扇区上，且stage 1.5并没有存储在文件系统上，所以stage 1也不需要有文件系统驱动，只需要能够驱动硬盘即可，因为stage 1只有446个字节，所以不可能在stage 1中存放硬盘驱动，实际上stage 1是通过BIOS的INIT13 中断将指定扇区的内容载入内存的。所以stage 1能够读取stage 1.5。




接下来是stage 1.5读取stage 2，因为stage 2存放在/boot文件系统中，所以stage1.5就必须要有/boot文件系统的驱动才能够访问/boot文件系统，事实上，正是由于stage 1没有识别文件系统的能力，才会需要stage 1.5的帮助。在stage 1.5中内置了/boot分区的文件系统驱动，能够用来识别/boot所在分区的文件系统。所以，stage 1.5能够读取stage 2。




最后是stage 2读取内核，显然在之前的stage 1.5阶段已经加载了/boot文件系统的驱动，所以stage 2可以利用stage 1.5提供的文件系统驱动去识别文件系统。所以stage 2能够读取内核。




 




## ramdisk




在讲ramdisk之前我们先了解一下linux内核的设计。在内核设计中主要分为两大阵营 : 单内核和微内核（第三阵营是外内核，主要用在科研系统中）。




单内核是一种比较简单的设计，所谓单内核就是将一个内核所需的所有功能整合在一起，作为一个单独的程序来实现，并且整个内核都运行在同一个地址空间中。因此，内核通常以单个二进制文件的形式存放在磁盘中。因为内核服务都在同一个内核空间中，所以内核之间的通信时间基本上可以忽略不计，所以单内核的设计具有简单和性能高的特点。大多数的Unix系统都是单内核设计。




微内核则与之相反，微内核将内核功能划分为多个独立的过程，每个过程称作一个服务器。那些需要运行特权指令的服务器会运行在内核空间中，而其他服务器则运行在用户空间中。由于每个服务器都运行在自己的地址空间上，所以不能像单内核那样直接调用函数，而只能通过消息传递机制通信，而微内核采用的是进程间通信（IPC）机制，因此各个服务器通过IPC互换消息。由于微内核的服务器之间各自独立，所以一个服务器出了问题不会祸及其他服务器，有效地保证了内核的稳定性。但是因为IPC的开销高于函数调用，而且服务器在运行特权指令时还会涉及内核空间与用户空间的上下文切换，所以实际上的微内核并没有采用完全的微内核设计，而是将大部分服务器都放在内核中，这样就可以直接调用函数并且消除频繁的上下文切换。但是这违背了微内核设计的初衷。




Linux是单内核设计，所以Linux内核运行在单独的内核地址空间上，设计简单而且性能高。同时，Linux在内核设计上汲取了微内核设计的精华，也就是其引以为豪的模块化设计，微内核设计中的内核功能被划分为单独的进程，而Linux则借鉴了这一点，将内核功能划分为多个模块，这使得Linux内核可以在需要的时候动态地卸载和加载内核模块，如此一来，单内核设计带来的庞大的内核体积的问题就解决了，我们可以在编译时只将那些必要的基础的功能编译进内核，而其他功能就编译成模块存放在文件系统上，这使得Linux内核显得极为实用和灵活。




很显然，文件系统也是内核的功能，所以在Linux内核中，文件系统也被划分成了模块，可以在需要的时候加载相应的文件系统。（CentOS 系统上Linux内核模块位于/lib/modules/KERNEL_RELEASE/kernel下，其中**KERNEL_RELEASE**为内核的版本号加上CentOS发行号，比如 3.10.0-693.el7.x86_64。文件系统模块就在fs子目录中）




因为将很多功能编译成了模块，所以内核在编译时只需要保留最基础的功能，而在需要时再去动态加载模块即可。这就是为什么Linux内核只有区区几兆的原因。但是这也为系统启动带来了一些问题。我们回想一下系统的启动流程，首先是BIOS读取启动盘的MBR，然后执行bootloader程序，bootloader程序加载并引导内核，然后内核探测硬件，并挂载根文件系统，最后启动init进程。




问题就出在挂载根文件系统的时候，如果想要挂载文件系统，首先内核需要加载对应文件系统的驱动，比如 : ext2,ext3,ext4,xfs,btrfs等，然后通过文件系统驱动识别文件系统以实现挂载，但是此时内核中的部分文件系统驱动被做成了模块存储在文件系统上（一般都存放在根文件系统上），此时如果内核想要将根文件系统挂载，就需要根文件系统对应的驱动，但是驱动在根文件系统上，所以内核需要去根文件系统上加载文件系统驱动，如果想要加载根文件系统上的驱动，就需要先挂载根文件系统。注意到问题所在了吗？这就像是一个先有鸡还是先有蛋的问题，永远无法找到源头。同时我们还要注意到，为了节省空间，内核一般都是压缩存放的，所以将内核镜像加载进内存后并不能直接使用，还需要将其解压后才能使用，所以为了解压内核我们还需要解压所需的驱动，显然，这个驱动也在根文件系统中。此外，为了能够挂载根文件系统，我们的内核还需要能够驱动计算机中的硬件设备，而内核中只有一些最基础的硬件驱动，所以内核不一定能够驱动当前计算机的硬件，当然，这个硬件驱动也在根文件系统中。




为了解决这个先有鸡还是先有蛋的问题，有人就想到了一个办法，就像stage 1需要stage1.5的帮助一样，内核也可以借助其他人的帮助。在系统安装时，有一些程序会去探测你的硬件信息，并且在安装的最后阶段，根据安装信息和硬件信息，会生成一个文件，这个文件中包含了驱动你的硬件和文件系统所需的最基本的驱动程序，这样的话，只需要将这个文件加载进内存，就能够利用这个文件中提供的驱动程序将根文件系统挂载。一旦完成根文件系统的挂载，接下来的事情也就水到渠成了。问题是这个文件也在文件系统上，要怎么加载呢？




聪明的你可能已经注意到了，其实这个文件就是ramdisk，而ramdisk会在stage 2加载内核的同时被加载，所以实际上，ramdisk和内核存放在同一个分区上。




在CentOS 6和CentOS 7中使用的ramdisk为initramfs，在CentOS 5中使用的为initrd。而initramfs和initrd文件除了在安装系统时被创建，还可以通过命令创建。initrd通过mkinitrd命令创建，而initramfs通过dracut命令创建（后面会介绍命令的用法）。




initrd（Initial ramdisk）是ramdisk类型的文件，本质上是块设备，只提供了块设备的读写接口，内部有ext2类文件系统的存在。在initrd中，包含了根文件系统所拥有的部分目录，包括bin、dev、etc、lib、proc、sys、sysroot、 init等文件或目录。其中包含了一些设备的驱动，比如 scsi、ata 等设备驱动，同时还有几个基本的可执行程序 insmod, modprobe,nash。主要目的是加载一些存储设备的驱动模块，把真正的根文件系统以只读方式挂载。内核在挂载真正的根之前会先挂载initrd，将其作为临时根，然后通过临时根中提供的必要的驱动和程序，挂载真正的根，最后将根切换到真正的根上，并卸载initrd。




initramfs是initrd的一个替代品，以另外一种方式实现了曾经initrd的功能。initrd是一个被加载的块设备，于是由于Linux内核的缓存机制，其中的内容还会被缓存到内存上，相当于访问一个文件需要在内存中保存两份，造成一定的内存空间浪费。而且由于initrd上有ext2文件系统存在，所以内核必须将ext2文件系统驱动编译进内核。而initramfs本身就是一个tmpfs的内存盘，拥有最小化的设计，绕过了缓存机制，也消除了多余的内存占用。




 




## kernel




内核一般都是压缩存放，所以在加载内核的同时，还需要将其解压，一般来说，内核文件在开头部分内嵌有gzip解压缩代码，能够进行自解压。在内核获得系统的控制权后，会去探测所有可识别的硬件设备，以只读方式挂载根文件系统，加载硬件驱动程序，这些动作可能会需要initdisk文件的帮助。在内核做完所有的初始化环境的准备后，就基本上完成了内核空间的任务，我们知道操作系统最终是给用户使用的，操作系统需要用户空间的进程提供生产价值，所以在最后，内核需要启动用户空间的第一个应用程序 : init（/sbin/init）。




 




## init




**init**（_initialization_）是内核启动期间启动的第一个用户空间进程。init是一个守护进程，它是所有其他进程的直接或间接的祖先，并自动收养所有的孤儿进程，init负责管理所有的用户空间进程。如果内核无法启动init的话，就会发生一个kernel panic。Init通常被分配pid 1。




init在被启动时，会根据配置文件（centos 5下为/etc/inittab）初始化系统环境，启动或关闭其他预先定义的进程，并在初始化完成后为用户提供一个登录终端。




init的工作方式在不同的Linux发行版中都是不同的，大多数Linux发行版是和 System V 相兼容的，但是一些发行版如Slackware 采用的是BSD风格，其它的如 Gentoo 是自己定制的。后来Ubuntu 和其他一些发行版采用 Upstart来代替传统的 init 进程。至2015年，大部分Linux发行版都已采用新的systemd替代传统的init和Upstart，但systemd向下兼容传统的System V风格的init。




接下来的内容由于涉及到Linux发行版，所以根据发行版的不同，有些内容可能会不同，所以请酌情对下面内容做理解，下面要使用的分别是CentOS的5、6两个版本。




其中CentOS 6使用的是System V风格的init。CentOS 6使用的是Upstart。CentOS 7使用的是systemd。




### CentOS 5




在CentOS 5中使用的是System V风格的init，顾名思义，它源于 System V 系列 UNIX。它提供了比 BSD 风格 init 系统更高的灵活性。是已经风行了几十年的 UNIX init 系统，一直被各类 Linux 发行版所采用。




#### runlevel（运行级别）




SysV init使用run level来描述系统各种可能的状态，通常会有 8 种runlevel，即runlevel 0 到 6 和 S 或者 s。Sysvinit 在启动时检查/etc/inittab文件中是否含有 initdefault 项，这个项用来告诉init系统的默认runlevel是多少。如果没有默认的runlevel，那么用户将进入系统控制台，手动决定进入何种runlevel。




每种 Linux 发行版对运行模式的定义都不太一样，但是大多数发行版的0、1和6级别的定义却相当类同 : 




**0** 关机




**1** 单用户模式，维护模式




**6** 重启




对于CentOS 系统，它自定义了2、3和5级别 : 




run level 3 将系统初始化为命令行界面的 shell ; run level 5 将系统初始化为 GUI 。无论是命令行界面还是 GUI，run level 3 和 5 相对于其他级别而言都是完整的正式的运行状态，系统可以完成用户需要的任务。而模式 1，2 等往往用于系统故障之后的排错和恢复，因为在1、2等维护模式下，大多数额外安装的进程不会被启动，并且内核只会使用最基础的驱动来维持系统的正常运行。runlevel 2和3相比只是少启动了nfs服务。runlevel 4在CentOS中未被使用。




很显然，这些不同的运行级别下系统需要初始化运行的进程和需要进行的初始化准备都是不同的。比如runlevel 3 不需要启动 X 系统。用户只需要指定需要进入哪种模式，sysvinit 将负责执行所有该级别所必须的初始化工作。




可以通过init命令更改当前的runlevel : init N。 N为指定的运行级别。




可以通过**runlevel**或**who -r**命令查看当前的runlevel。




 




#### inittab（sysvinit的配置文件）




sysvinit的配置文件只有一个inittab，首先查看一下CentOS 5中该配置文件的内容。




![](http://118.25.17.78/wp-content/uploads/2018/06/inittab.jpg)




可以发现每行都有固定的格式 :




**id:run level:action:process **




**id** : 用于描述该条目的唯一标识，不能重复。




**run level** : 指定该条目所使用的运行级别。




**action** : 该条目执行的动作。




**process** : 该条目生效时执行的程序。







#### initab中每行的解释




**id:3:initdefault:**




我们发现第一行的id使用的就是id，这个没有特别的意义，所以我们不需要关注它，第二个字段3表示运行级别为3，第三个字段initdefault表示系统的默认运行级别，由于只是定义了默认运行级别，所以第四个字段就没有必要执行程序了，所以留空。所以第一行实际上只是定义了系统的默认运行级别。如果前面runlevel没有指定运行级别，那么会在启动时在终端询问。




**si::sysinit:/etc/rc.d/rc.sysinit**




si只是一个标识符，无需关心。第二个字段留空，表示这个条目在所有运行级别下都会生效。第三个字段为sysinit，表示在系统初始化时会执行后面的process（/etc/rc.d/rc.sysinit）。第四个字段表示在系统初始化时会执行的程序。（该脚本的功能见下面的讲解）




**l0:0:wait:/etc/rc.d/rc 0**




l0是level 0的缩写，无需关心。第二个字段为0，表示这个条目在级别0下生效。第三个字段为wait，表示在进入runlevel 0时就执行后面的process（/etc/rc.d/rc 0）。第四个字段表示在进入runlevel 0时会执行的程序。（该脚本的功能见下面的讲解）




l1~l6也是如此，不再做赘述。




**ca::ctrlaltdel:/sbin/shutdown -t3 -r now**




第二个字段留空，表示这个条目在所有运行级别下都会生效。第三个字段为ctrlaltdel，表示在按下ctrl + alt + delete组合键时会执行后面的process。第四个字段为/sbin/shutdown -t3 -r now，表示3秒钟后重新启动系统。




**pf::powerfail:/sbin/shutdown -f -h +2 "Power Failure; System Shutting Down"**




**pr:12345:powerokwait:/sbin/shutdown -c "Power Restored; Shutdown Cancelled"**




pf表示在断电后进入UPS电源时，计时2分钟后关机。pr表示在2分钟内如果电源恢复了，则取消关机。




**1:2345:respawn:/sbin/mingetty tty1**




第二个字段2345表示这个条目在2345级别下生效。第三个字段为respawn，表示如果后面的process终止了，就将其重新启动。第四个字段为/sbin/mingetty tty1，表示启动一个虚拟终端tty1。




2-6也是如此。




**x:5:respawn:/etc/X11/prefdm -nodaemon**




表示5级别下启动的图形终端。




 




##### /etc/rc.d/rc.sysinit




/etc/rc.d/rc.sysinit为系统初始化脚本，sysvinit所需要完成的系统初始化任务就是由它来完成的。它主要完成以下的初始化工作 : 




**(1)** 设置主机名




**(2)** 设置登录后的欢迎信息




**(3)** 激活udev和selinux




**(4)** 挂载/etc/fstab中定义的文件系统




**(5)** 检测根文件系统，并以读写方式重新挂载根文件系统。（为了安全起见，内核最初是以只读方式挂载的根文件系统）




**(6)** 设置系统时钟（从硬件时钟同步到系统时钟）




**(7)** 激活swap设备（或者swap分区）




**(8)** 根据/etc/sysctl.conf文件设置内核参数




**(9)** 激活lvm及软raid设备




**(10)** 加载额外设备的驱动




**(11)** 初始化完成后的清理操作




 




##### /etc/rc.d/rc N




参数N表示运行级别，不同的运行级别会让rc执行不同的脚本。该脚本用于启动或停止对应运行级别中预先定义的服务，使得不同的运行级别能够启动不同的服务。根据不同的 runlevel，rc 脚本将打开对应该 runlevel 的 rcX.d 目录(X 就是 runlevel)，找到并运行存放在该目录下的所有启动脚本。每个 runlevel X 都有一个这样的目录，目录名为/etc/rc.d/rcX.d。




在这些目录下存放着很多不同的脚本。以K或S开头，并紧跟着两位数字。K（Kill）开头的服务脚本表示停止该服务，S（Start）开头的服务脚本表示启动该服务，K开头的服务脚本先被执行，然后才会执行S开头的服务脚本。两位数字表示启动或停止的优先级，数字越小优先级越高。在/etc/rc.d/rcX.d 目录下的脚本其实都是一些软链接文件，真实的脚本文件存放在/etc/init.d 目录下。




当所有的初始化脚本执行完毕，Sysvinit 会运行/etc/rc.d/rc.local 脚本（一般只有级别3和5才会运行rc.local，因为其他运行级别没有用户的概念）。




rc.local 是 Linux 留给用户进行个性化设置的地方，你可以将那些不方便写成服务脚本但是又希望能够开机执行的命令写在rc.local中。




比如/etc/rc.d/rc 0，它会先执行/etc/rc.d/rc0.d中K开头的脚本，并且根据后面两位数字决定K开头脚本执行的先后顺序（先执行数字小的脚本），这些服务都会被停止。




![](http://118.25.17.78/wp-content/uploads/2018/06/rc0.jpg)




然后执行S开头的脚本，也是根据数字决定执行顺序，这些服务都会被启动。在这里我们注意到，虽然runlevel 0是关机，但是仍然有两个S开头的脚本，一个是S00killall，一个是S01halt，其中S00killall是终止所有没有停止的进程，而halt是关机，所以这两个脚本以S开头也情有可原。




rc0.d下的脚本实际上为init.d下脚本的软链接，使用K或S开头只是为了服务于脚本。




![](http://118.25.17.78/wp-content/uploads/2018/06/llrc0.jpg)




 




而rc脚本决定启动顺序的方式也非常巧妙，观察一下rc脚本的代码就可以发现，它是通过通配符取得服务脚本的 : 




![](http://118.25.17.78/wp-content/uploads/2018/06/killscripts.jpg)




如果使用通配符匹配文件，那么文件会被按照文件名从小到大的顺序依次匹配，所以rc利用了一个很巧妙的方式实现了执行顺序。执行顺序的主要目的是为了解决服务之间的依赖关系，比如我们需要启动网络服务，然后才能够启动apache服务。




 




#### chkconfig




这些软链接文件不可能凭空出现，但也不应该是手动添加的，因为手动操作一个疏忽就可能带来问题。如果你有注意到之前在init.d目录下的脚本，你可能会发现一些有趣的东西。




我们打开init.d目录下的syslog脚本，查看开头的注释内容。




![](http://118.25.17.78/wp-content/uploads/2018/06/syslog.jpg)




我们可以发现这其实就是一个服务脚本，用来控制syslog服务的启动、停止、重启等动作。




注意到第六行的chkconfig，这后面有三个字段，2345 12 88。其中12表示syslog服务的启动顺序号为12,88则表示关闭的顺序号为88，分别对应 S12syslog和K88syslog两个软链接。2345则表示syslog服务只在2345级别启动，也就是说，S12syslog只创建在/etc/rc.d/rc{2,3,4,5}.d目录中，K88syslog只创建在/etc/rc.d/rc{0,1,6}.d目录中。（一般来说，先启动的服务应该后关闭，反之亦然。这也是因为服务之间的依赖关系。所以你会发现启动顺序号和关闭顺序号相加往往接近100。）




但是这一行有什么用呢？事实上，这一行信息就是给chkconfig命令使用的，通过这一行信息，chkconfig命令就会在相应的目录中创建相应的软链接，以实现服务方便地添加和删除。用户只需要会使用Linux的基本操作就能够轻松地控制某个runlevel下服务的启动与否。




chkconfig命令有四个常用选项 :




**chkconfig --list [name]**




列出指定服务在不同runlevel下的启动与否。如果不指定服务，则列出所有服务。




![](http://118.25.17.78/wp-content/uploads/2018/06/chkconfig-syslog.jpg)




  

**chkconfig --add name**




通过init.d目录下的服务脚本名添加服务，首先在init.d目录下要有相应的脚本才能够添加该服务，通过服务脚本中的chkconfig字段在相应的rcN.d目录下创建相应的软链接。




这里创建了一个测试用的脚本，没有实际意义。可以看见chkconfig只是简单地为这个服务添加了几个软链接。




![](http://118.25.17.78/wp-content/uploads/2018/06/testsrv.jpg)




  

**chkconfig --del name**




通过init.d目录下的服务脚本的chkconfig字段删除相应的软链接。




可以发现chkconfig只是简单地删除了这些软链接实现了服务的删除。在使用list的时候可以发现chkconfig提示你testsrv服务脚本已经存在，但是还没有被添加到服务中。




![](http://118.25.17.78/wp-content/uploads/2018/06/testsrv-del.jpg)




  

**chkconfig [--level levels] name <on|off|reset>**




为服务指定在不同runlevel下的启动与否。levels表示运行级别，如果有多个运行级别需要指定，则直接键入多个即可，不要使用空格分隔（比如--level 2345），如果不指定level，则默认为2345。on表示启动，off表示禁用。reset表示根据服务脚本的chkconfig字段重置。




![](http://118.25.17.78/wp-content/uploads/2018/06/testsrv-set.jpg)




 




 




#### sysvinit执行过程的总结




**1、**首先，sysvinit会去读取/etc/inittab配置文件文件，分析这个配置文件中的信息。




**2、**得到配置文件的信息后，sysvinit会设定系统的默认runlevel。




**3、**接着，sysvinit会去执行/etc/rc.d/rc.sysinit脚本以实现重要的系统的初始化。




**4、**完成了以上工作后，sysvinit会执行/etc/rc.d/rc脚本，根据不同的 runlevel，rc 脚本将打开对应该 runlevel 的 rcX.d 目录(X 就是 runlevel)，找到并运行存放在该目录下的所有启动脚本。每个 runlevel X 都有一个这样的目录，目录名为/etc/rc.d/rcX.d。其中S（Start）开头的脚本是需要启动的服务脚本，K（Kill）开头的脚本是需要停止的服务脚本。




**5、**如果级别为3或5，那么在最后会去执行/etc/rc.d/rc.local脚本。




**6、**最后，系统初始化完毕后，sysvinit为用户打开登录终端。




 




### CentOS 6




CentOS 6使用了Upstart替代了传统的sysvinit。Upstart是由Canonical公司前雇员Scott James Remnant所写，并由Ubuntu最早在2006年将其应用在系统中以取代sysvinit。




我们先来看一下sysvinit的工作方式，可以发现sysvinit使用服务脚本和软链接实现服务的启动和停止、运行级别，由于脚本和软链接的创建并不复杂，而且不需要学习新的额外知识或语法，所以sysvinit使用起来不仅简单灵活，而且概念清晰。




但是这就带来一个问题，脚本的执行是串行化的，所以在开机启动服务时，必须等待当前服务脚本执行完成（即当前服务启动完成），才能够执行下一个服务脚本，这就导致sysvinit的运行效率非常低效，在现代的桌面系统中，低效的系统启动是用户无法忍受的。当然串行化的启动也带来一个优点，管理员能够很容易地在启动过程发生问题时排错（因为服务脚本严格按照启动数字的大小顺序执行，一个执行完毕再执行下一个，通过观察kernel dump可以发现在哪个服务启动时出错）。




此外，当Linux内核更新到2.6版本时，内核新增了很多新特性，使得Linux不仅适合用于服务器上，还适合用于桌面机和嵌入式设备上。现在桌面系统的一个特点是硬件繁多，但是接口数量有限，所以一些设备并不是直接连接在接口上，而是在系统启动之后一段时间，需要用到设备时才连接。因此当系统POST时，有一些设备可能并没有连接。在 2.6 内核支持下，一旦新外设连接到系统，内核便可以自动实时地发现它们，并初始化这些设备，进而使用它们。这为便携式设备用户提供了很大的灵活性。




但是这也为sysvinit带来了一些问题，有时候需要初始化的设备在系统初始化时并没有连接，比如打印机，为了使用打印服务，系统必须要启动CUPS等服务，而打印机往往在系统启动时并没有连接到系统，这个时候启动打印服务完全就是浪费。但是不幸的是，对于这种服务，sysvinit并没有能力管理，它只能够一次性启动所有指定的服务，即使打印机没有连接到系统。




还有挂载网络设备的问题，我们在前面讲sysvinit执行过程的时候提到过，/etc/fstab中的设备会在系统初始化时由rc.sysinit脚本完成挂载，而网络服务则是在完成系统初始化后，由rc脚本启动的，所以如果/etc/fstab中有一个网络设备，那么在该网络设备被挂载时，网络是没有的，所以系统显然找不到这个网络设备，所以挂载一定会失败。Sysvinit 采用 netdev 的方式来解决这个问题，即/etc/fstab 发现 netdev 属性挂载点的时候，不尝试挂载它，在网络初始化之后，还有一个专门的 netfs 服务来挂载所有这些网络盘。这是一个不得已的补救方法，给管理员带来不便。部分新手管理员甚至从来也没有听说过 netdev 选项，因此经常成为系统管理的一个陷阱。




针对以上种种问题，ubuntu开发人员便自己开发了一种新的init程序，也就是Upstart。Upstart是一个基于事件驱动的init程序，这允许它以异步方式对生成的事件作出回应。




比如打印机，当打印机连接到系统时，就发生了一个事件，Upstart检测到这个事件后，就会处理这个事件，比如启动打印服务。由于Upstart能够以异步方式工作，所以在启动服务时，能够并行启动多个不相关的服务（实际上大部分的服务之间都是不相关的），这能够明显缩短系统的启动时间。这能够很好地满足桌面系统的需求（良好的热插拔响应和快速的系统启动）。




虽然Upstart的设计理念很好，但是传统的sysvinit是基于运行级别的，而且由于历史原因，Linux系统上的多数软件采用的仍然是sysvinit的服务启动方式，所以Upstart必须能够兼容sysvinit的服务脚本。




实际上这是很容易的，观察下图，在Upstart启动的过程中仍然可以使用rc.sysinit和rc脚本，而sysvinit的核心就是这些服务脚本，所以在使用时只需要替换init程序和配置文件即可，而不需要对服务脚本和系统初始化脚本做任何修改。




![](http://118.25.17.78/wp-content/uploads/2018/06/Upstart.png)




需要注意的是，Upstart程序的配置文件在/etc/init目录下，以.conf结尾，比如rc.conf配置文件就是用于配置rc脚本。但是这个配置文件的语法必须遵循Upstart配置文件的语法格式，所以要使用Upstart，你需要花费一些时间去学习它的语法格式。在CentOS 6系统中，为了兼容sysvinit，仍然保留了/etc/inittab配置文件，但是这个配置文件中只有一项配置，配置系统的默认运行级别。







### 创建ramdisk文件




我们都知道根分区是非常重要的，一旦根分区出现损坏，那么所有其他分区将都无法使用，所以在安装完系统后，我们可能会将根分区放在RAID设备上面，组成RAID 1来保证根分区的安全。这么一放不要紧，但是ramdisk就无法正常驱动根分区了因为ramdisk中没有能够驱动RAID设备的驱动，此时我们就需要根据当前根分区的情况重新制作ramdisk文件。除此之外，如果ramdisk文件损坏，或者是根分区格式化了新的文件系统，我们都需要重新制作ramdisk文件。




mkinitrd和dracut会根据给定的根文件系统所在块设备所需的内核模块（比如IDE，SCSI或RAID）创建ramdisk文件，保证系统能够将根文件系统挂载。




在CentOS 5中，使用的工具是mkinitrd，在CentOS 6和CentOS 7中，使用的工具是dracut。实际上在CentOS 6中同时提供了mkinitrd和dracut，而在CentOS 7中虽然也提供了mkinitrd，但是mkinitrd会去调用dracut，所以最终使用的还是dracut。




#### mkinitrd




**命令用法**




**mkinitrd [OPTION...] [<initrd-image> [<kernel version>]]**




其中initrd-image为需要创建的initrd的名称及路径，kernel-version为指定需要创建的initrd所对应的内核版本，可以通过 uname -r 命令查看内核版本。




如果没有给定kernel version，那么将会使用当前正在运行的内核版本。如果没有指定initrd-image，那么就使用默认的参数/boot/initramfs-<kernel version>.img。




![](http://118.25.17.78/wp-content/uploads/2018/07/mkinitrd.jpg)




首先看一下uname -r的输出，可以发现它输出了当前运行中内核的版本，然后我们通过命令替换$(uname -r)创建initrd镜像，结果被告知文件已存在，所以我们将其移到别的位置，继续创建。创建过程需要探测硬件信息和根文件系统，所以需要稍加等待，创建完毕后，查看/boot目录，可以发现initrd文件已经创建好了。




 




#### dracut




**命令用法**




**dracut [OPTION...] [<image> [<kernel version>]]**




实际上dracut的命令用法与mkinitrd相同，所以两者最大的区别就是命令名称发生了改变。




![](http://118.25.17.78/wp-content/uploads/2018/07/dracut.jpg)




 







## grub legacy




在前面的bootloader中我们以grub legacy为例讲过了grub legacy的工作流程，这里我们简要描述一下grub legacy的工作流程。




**1、首先BIOS读取启动盘中的MBR（grub stage 1）**




**2、然后MBR中的bootloader（grub stage 1）知道grub stage 1.5所在扇区（一般位于MBR之后的扇区），并且将grub stage 1.5加载。**




**3、grub stage 1.5知道grub stage 2所在的位置，并能够识别grub stage 2所在分区的文件系统，所以grub stage 1.5将grub stage 2加载。**




到了stage 2后，其实一个完整的grub就出现了，这个阶段的grub包含了大部分的grub逻辑代码，能够实现提供菜单、内核选择、编辑内核启动参数、init ramdisk选择甚至提供背景图片等更加高级的功能。下面就着重讲一下grub stage 2。







### stage 2




#### stage 2提供的功能




1、提供了一个可供用户选择的菜单，允许用户加载不同的内核或操作系统（菜单可以隐藏，也可以在菜单页面提供背景图片）




2、提供了交互式接口，允许用户使用交互式接口操作grub（e: 编辑模式，可用于编辑菜单项。c: 命令行模式，提供操作grub的交互式接口）




3、提供了保护机制，允许用户为启动内核和编辑菜单项提供认证机制。（即用户在启动某个内核或者编辑某个菜单项之前需要输入密码）




 




#### stage 2识别设备的方式




stage 2将所有设备都识别为(hdX,Y)。其中hd为固定前缀，任何设备都被识别为hd（hardware disk）。X为磁盘编号，第一块被识别的磁盘编号为0，第二块为1，以此类推。Y为分区编号，磁盘的第一个分区编号为0，第二个分区编号为1，以此类推。例如(hd0,0)表示第一块磁盘的第一个分区。




 




#### stage 2配置文件




stage 2的配置文件为 : **/boot/grub/grub.conf**，并且还有一个链接文件/etc/grub.conf也指向这个配置文件。




查看一下配置文件的内容，可以看到如下图所示的输出。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub.conf_.jpg)




**default** : 表示默认启动的内核，0表示下面的第一个title。（title可以有多个，每个title指明了一个内核启动所需的所有信息）




**timeout** : 启动超时时间。如果用户在等待时间内没有对菜单栏进行操作，那么菜单栏在等待5秒后，会自动选择默认启动的内核进行启动。5表示等待5秒钟。




**splashimage** : 菜单栏背景图片。后面的参数（(hd0,0)/grub/splash.xpm.gz）指明了背景图片所在的路径。




**hiddenmenu** : 隐藏菜单栏。如果不对菜单栏进行操作，那么用户将不会看到菜单栏。




**title** : 显示在菜单栏中的菜单项的名称。每个title需要包含root、kernel和initrd。需要注意的是这三个字段都需要缩进。




**root** : 指明内核所在分区。




**kernel** : 指明内核所在的路径，以及给定一些内核启动参数。（如ro : 以只读方式启动内核; single : 以单用户模式启动内核等）




**initrd** : 指明init ramdisk文件所在路径。







此外，配置文件中还可以添加password字段，用于为启动内核和编辑菜单项提供验证机制。




password [--md5] STRING : 如果不加--md5选项，那么直接使用明文密码即可，后面的STRING即为指明密码。如果添加了--md5选项，那么需要在后面指明md5加密后的密码。（注意 : 该选项可以添加在全局配置中，也可以添加在单个title中。添加在全局配置中表示需要对菜单编辑进行验证，添加在单个title中表示启动该title指明的内核时需要进行验证。）




 




生成md5加密的密码可以使用openssl命令，但是如果只是为了生成md5的密码，这个命令就有点小题大做了，我们可以使用grub提供的grub-md5-crypt命令。




直接使用命令，然后输入两边密码即可，下面的输出即为md5加密后的密码。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-md5-crypt.jpg)




得到加密后的密码后，就可以往配置文件中添加password字段。观察下图输出，可以发现除了在全局配置处添加了password字段之外，还在单个title中添加了password字段。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub.conf-2-1.jpg)




重新启动时按任意键即可进入grub菜单，发现现在有两个菜单项了。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-menu.jpg)




如果想要启动第二个内核，则需要输入密码（title内的password指定）。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-boot-password.jpg)




如果想要编辑某个菜单项，则也需要键入p，然后输入密码（全局的password指定），最后敲回车确认。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-edit-password.jpg)




输入密码后，发现下方的提示信息发生变化，此时已经可以编辑菜单项了。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-edit-unlock.jpg)




 




#### stage2 命令行接口




**e : 编辑指定的菜单项**




**c : 进入交互式接口模式**




 




在第一个内核的菜单项上按下e键，进入对该菜单项的编辑界面。可以发现这三个字段实际上就是在配置文件中的title配置的三个字段。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-press-e.jpg)




将光标移动到kernel项，再次键入e即可编辑kernel项的参数。同理也可以编辑root和initrd。在此处的选项的语法与在配置文件中一致。使用esc可以退出当前界面。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-edit-kernel.jpg)




编辑完后在下图界面按b键即可按照当前配置启动操作系统。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-ready-to-boot.jpg)




 




或者我们可以直接使用grub2的命令行接口，按c进入。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-cli.jpg)




在命令提示符下键入help即可获取所有的命令。键入help COMMAND即可获取某一个命令的帮助信息。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-help.jpg)







##### stage2 命令行接口常用命令




**find FILEPATH** : 在所有分区查找指定文件，并打印出包含该文件的设备列表。后面的参数可以使用tab补齐（如果可以补齐的话）。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-find.jpg)




**root** : 指明内核所在分区。




**kernel** : 指明内核文件路径及内核启动参数。如果前面指定了root，则此处可以不指明分区(hdX,Y)。




**initrd** : 指明init ramdisk文件路径。如果前面指定了root，则此处可以不指明分区(hdX,Y)。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub2-root-kernel-initrd.jpg)




**boot** : 启动上述指定的内核。




 




### 安装grub legacy




有时候我们可能会发生grub损坏的情况，这时候我们就需要进入救援模式或者使用另一台计算机为损坏的硬盘修复grub。




**grub-install [--root-directory=DIR] install_device** : 该命令可以为指定设备安装grub，包括stage 1、stage1.5和stage2。因为stage 1和stage1.5位于硬盘扇区上而不存在于文件系统上，stage 2位于文件系统上，所以我们在使用grub-install时，需要指明需要安装grub的硬盘（安装stage 1和stage1.5）和需要安装grub的分区或目录（安装stage 2）。




其中install_device即为指明需要安装grub的硬盘。--root-directory即为指明需要安装grub的目录或分区，如果不指明则默认安装在/目录下。




需要注意的是，创建stage 2时会需要root-directory下存在boot目录，如果不存在的话就会被创建。同时还会在boot目录下创建grub目录，里面保存的是stage 2的文件。（但是并不包含stage 2配置文件和内核及init ramdisk文件。这些文件需要手动创建。）




而boot目录有两种，一种是为boot目录进行单独分区，另一种是直接将boot目录放在根分区，后者要求根分区能够被stage 1.5识别，也就是说根分区不能是lvm这些文件系统。




针对将boot单独分区的，我们在安装grub时就需要注意一些细节。我们知道如果我们将boot目录单独分区，那么就需要在根分区上创建一个boot目录，然后将boot目录作为boot分区的访问入口使用。也就是说，实际上boot目录在根分区上，而boot分区中只有boot目录下的文件而没有boot目录，这一点一定要搞清楚。（比如 : /boot/grub/grub.conf文件，这个文件存在于boot分区上，然而/boot目录存在于根分区上。）所以如果想在一块硬盘的分区上创建stage 2，那么我们必须先在一个目录下手动创建一个boot目录（可以在任意目录下创建，只是用来挂载），然后将硬盘的boot分区挂载到这个boot目录上，然后将root-directory指向这个目录。（比如在/mnt下创建boot目录，然后将/dev/sdb1挂载到/mnt/boot目录）。之所以这么做就是因为grub安装时一定会在root-directory下找boot目录，而单独分区的boot分区上是没有boot目录的，所以我们要提前创建一个boot目录。




针对将boot直接放在根分区上的，就要简单得多，我们直接将这个硬盘的根分区挂载到一个目录，然后将root-directory指向这个目录即可。（比如将/dev/sdb2挂载打/mnt，然后将root-directory指向/mnt）因为根分区上已经有boot目录了，所以grub-install就能找到boot目录。




 




接下来我会演示将boot目录单独分区，然后在硬盘上安装grub。




首先将需要安装grub的磁盘连接至系统，并且将其分区格式化，在这里我将其分成了三个分区，/dev/sdb1为boot分区，/dev/sdb2swap分区，/dev/sdb3为根分区，并且boot分区和根分区为ext2文件系统。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-sdb.jpg)




由于需要安装grub，所以我们需要将boot分区挂载，而且由于boot为单独分区，所以这里我们需要先创建一个boot目录供其挂载。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-1.jpg)




然后我们就可以在这个硬盘中安装grub了。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-2.jpg)




根据命令输出可以发现grub安装成功了，然后我们观察boot分区可以看见里面已经有了一个grub目录，进入到这个目录中就能看见grub stage 2阶段的文件。







至此，grub的安装其实已经结束了，但是为了让这个硬盘能够正常被引导启动，我们还需要做一些额外的步骤。




首先我们需要一个内核文件和一个init ramdisk文件，这里为了方便我们直接将本机的内核和init ramdisk复制过来。注意，内核和init ramdisk直接放在boot分区上，而不是放在grub目录中。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-3.jpg)




然后我们需要为grub stage 2准备一个配置文件grub.conf，放在grub目录下。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-4-3.jpg)




可以看见grub.conf文件的内容很简单，只引导一个内核文件。但是需要注意的是title被我修改过以便后面识别，而kernel字段中的root指向的是/dev/sda3而不是/dev/sdb3，这是因为当这块硬盘移动到另一台主机作为启动盘时，这块硬盘应该是第一个被识别的，所以设备为/dev/sda而不是/dev/sdb。







这里我们已经将grub的准备工作做完了，下面为了能够正常使用这个硬盘，我们还需要在硬盘的根目录中创建一些目录。




首先我们将根分区（/dev/sdb3）挂载至/mnt/sysroot下，然后在里面创建一些POSIX规范中规定的目录。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-5.jpg)




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-6.jpg)




虽然有了目录结构，但是根分区中并没有一个可执行的程序，我们知道内核被引导后启动的第一个用户空间进程是init，由于init还要涉及到runlevel，所以这里我们使用/bin/bash代替init作为第一个启动的进程。




首先我们将/bin/bash移植到根分区中，不仅需要/bin/bash这个程序，还需要/bin/bash依赖的库文件，这里我们使用ldd命令查看依赖的库文件。移植完后我们可以通过chroot命令测试bash程序能否正常运行。（chroot为根切换命令，可以临时将当前系统的根目录切换到指定的目录，这里我们将根目录切换到新硬盘上的/dev/sdb3挂载的目录）




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-7.jpg)




切换根后我们发现bash程序能够正常运行了，于是/bin/bash移植成功，同样的如果你需要其他的程序，也可以这样移植。




然后为了能够让内核在引导时启动/bin/bash,我们需要向内核传递一个启动参数: init=/bin/bash,表示启动的init程序为/bin/bash。所以我们需要编辑grub.conf配置文件。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-8-2.jpg)




上面使用了sync命令保证内存中的修改被写入到了硬盘中，然后我们就可以取出硬盘，将其放到另一台主机上，然后那台主机就使用这块硬盘作为启动盘。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-9-2.jpg)




启动主机，我们进入了grub界面，说明grub stage 2的启动成功了。接下来我们使用它引导内核。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-error.jpg)




**出现了一个内核恐慌（kernel panic），从上面的报错信息中可以看出来，是init ramdisk无法将根分区挂载为ext3文件系统。这就很好理解了，我们之前为了简单直接使用的是原主机的initrd文件，而那个initrd文件是安装程序专门为之前的操作系统所制作的，而之前的操作系统的根分区为ext3文件系统，所以initrd会带有ext3文件系统的驱动。但是我们的新硬盘上的根分区的文件系统是ext2文件系统，然而我们直接拿来的initrd中并没有ext2文件系统驱动而且initrd中会直接将根分区作为ext3文件系统挂载，所以显然会挂载失败。**




在这里解决办法也很简单，ext2文件系统可以直接升级成为ext3文件系统，因为它们之间只相差一个日志系统。我们使用tune2fs -j 命令将根分区（/dev/sdb3）变成ext3文件系统。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-11.jpg)




通过blkid的输出我们可以发现/dev/sdb3已经变成了ext3文件系统了，然后我们将硬盘拿出来重新放到另一台主机中启动。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-10.jpg)




在最后一行我们看见了bash的命令提示符，表示我们已经成功启动了内核和bash程序，可以开始操作我们的系统了。不过由于这里我们只移植了一个bash程序，所以我们基本上无法执行任何操作。




注意 : 如果你使用的是CentOS 6系统，那么在启动过程中可能也会出现kernel panic，类似如下所示的报错，那就很有可能是因为CentOS 6中的selinux，所以在内核启动参数中加上selinux=0即可关闭selinux。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-error2.jpg)




如下所示添加selinux参数，注意不要添加到init后面。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install-selinux.jpg)




 




上面这是为其他硬盘安装grub，那么**接下来我来演示如何为本机的硬盘安装grub**，这个就要简单得多，直接使用grub-install命令就可以搞定。




首先我们模拟硬盘的grub stage 1被破坏的情况，使用dd命令填充硬盘的前446个字节，我们知道硬盘的前446个字节为bootloader，也就是我们这里使用的grub stage 1。为了安全起见这里我们只填充200个字节，这样已经足够破坏bootloader了。（为了避免出现意外，我们需要先备份一下硬盘的MBR）




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-1.jpg)




此时实际上硬盘的bootloader已经损坏了，这里我先不选择重启，而是直接安装grub。（此时如果重启那么结果肯定是启动时死在bootloader阶段，在下面我会介绍救援模式修复）




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-2.jpg)




安装成功，我们重新启动系统试试。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-3.jpg)




进入到了grub stage 2，显然我们的grub修复成功了。







下面**我们来演示在grub被破坏后电脑重启的情况**，在这种情况下我们有两个选择，一个是使用CentOS 光盘中提供的紧急救援模式（rescue mode），或者将硬盘拔出来放到别的主机上修复，这就和上面讲的安装grub步骤一样了，所以这里我们使用CentOS光盘的rescue mode完成grub的修复。




首先我们同样模拟硬盘的grub stage 1被破坏的情况，但是在此处我们直接选择重新启动。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-4.jpg)




果不其然，由于bootloader被破坏，已经无法引导内核了，所以只能尝试从网络引导内核。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-5.jpg)




此时我们插入CentOS 5的光盘，然后重新启动系统（ctrl + alt + del 即可）。然后我们就能够进入CentOS 5的界面。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-6.jpg)




在这个界面下，我们可以看到很多帮助信息，看到倒数第二行的最后一个F5-Rescue，这就表示进入紧急救援模式。于是我们按下F5。（实际上，直接键入linux rescue后回车也可以进入救援模式）




进入救援模式后，选择语言和键盘布局。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-7.jpg)




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-8.jpg)




然后会询问是否启动网络功能。如果你确定可以不通过网络就能够完成修复过程，那么可以选择no，否则请选择yes配置网络。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-9.jpg)




这里告诉你一些关于rescue mode的信息，最重要的就是它会将你的根文件系统挂载到/mnt/sysimage下，你可以通过chroot将根切换到你的根文件系统下。在这里我们选择continue进入rescue mode提供的文件系统。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-10.jpg)




接下来它会尝试寻找你的硬盘中安装的Linux系统，找到后就会有如下提示。告诉你你的根文件系统被挂载到了/mnt/sysimage下，通过chroot /mnt/sysimage可以将系统环境切换到你原本的环境。我们直接敲回车确认。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-11.jpg)




然后我们就进入了命令行模式，这里我们使用chroot /mnt/sysimage命令切换根。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-12.jpg)




切换根后我们就相当于已经进入了自己原本的系统，在这里我们通过fdisk命令查看/dev/sda，发现硬盘分区信息正常，所以我们就使用grub-install命令直接在/dev/sda上安装grub。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-13.jpg)




安装完后，就可以使用exit退出切换的根，然后使用reboot重启操作系统。




然后我们又回到了熟悉的grub stage 2界面。说明我们的grub已经修复成功了。




![](http://118.25.17.78/wp-content/uploads/2018/06/grub-install2-14.jpg)







# 参考文章




[linux 系统启动过程 – Cizixs Writes Here](http://cizixs.com/2015/01/18/linux-boot-process)




[Bios读文件与Grub（bootload）和initrd和内核对文件系统驱动的支持 - CSDN博客](https://blog.csdn.net/ljl1704/article/details/40508401)




[计算机是如何启动的？ - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2013/02/booting.html)




[浅析 Linux 初始化 init 系统，第 1 部分: sysvinit](https://www.ibm.com/developerworks/cn/linux/1407_liuming_init1/index.html)




[Linux启动流程 - 博客 - binsite](https://www.binss.me/blog/boot-process-of-linux/)




[Details of GRUB on the PC](http://www.pixelbeat.org/docs/disk/)




Linux Kernel Development




wikipedia






















































