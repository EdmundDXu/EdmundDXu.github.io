---
author: edmund
comments: true
date: 2018-07-04 06:44:09+00:00
layout: post
link: http://118.25.17.78/blog/2018/07/04/linux%e5%86%85%e6%a0%b8%e7%ae%a1%e7%90%86%e4%b9%8b%e4%bc%aa%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9fprocfs%e5%92%8csysfs/
slug: linux%e5%86%85%e6%a0%b8%e7%ae%a1%e7%90%86%e4%b9%8b%e4%bc%aa%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9fprocfs%e5%92%8csysfs
title: Linux内核管理之伪文件系统procfs和sysfs
wordpress_id: 540
categories:
- Linux随笔
post_format:
- 日志
tags:
- kernel
- linux
- procfs
- sysctl
- sysfs
- udev
---

# 前言




通过上节（[Linux内核管理之常用内核模块管理工具](http://118.25.17.78/blog/2018/07/03/linux内核管理之常用内核模块管理工具/)）我们知道，Linux内核的管理实际上分为对Linux kernel的管理和对Linux kernel object的管理。而上节降了对kernel object的管理，这里我们就来讲讲对Linux Kernel的管理。




在Linux中，内存空间分为内核空间和用户空间，而且除了特权指令外，用户一般是无法访问内核空间的，但是管理员往往会需要通过配置内核的一些参数来实现调优操作或者开启内核功能，这个时候该怎么办？




这个时候我们想想之前讲过的进程管理，进程虽然可能运行在用户空间，但是显然进程信息还是保存在内核空间中（如PCB），所以我们如果需要进行进程管理，我们就需要在用户空间访问内核空间，这就显得既麻烦又不安全。




但是我们之前明明有通过ps命令很轻松地获取进程数据信息，而且所有用户都可以执行这个命令。实际上ps命令并没有访问内核空间，而是访问的procfs，在procfs中获取进程相关信息。同样的，管理员也是通过操作procfs中的文件实现对内核参数的配置。







# procfs




那到底什么是procfs呢？process filesystem（procfs）是一个特殊的文件系统，它以类文件的层级结构展示了进程信息和其他的系统信息，并且为动态地访问内核中的数据提供了一个更加方便标准的方法，比起直接访问内核内存，procfs要来的更加安全。




实际上，procfs是一个伪文件系统（[pseudo file system](https://en.wikipedia.org/wiki/Pseudo_file_system) ），这个文件系统通常被挂载到 /proc 目录。由于procfs 不是一个真正的文件系统，它也就不占用存储空间，只是占用有限的内存。




我们在用户空间所看到的/proc目录实际上并不包含真实文件（所以你会发现/proc目录中的文件大小都是0），它在内核启动前是空的，在内核启动后，内核会将系统运行时信息（例如系统内存，安装的设备，硬件配置等）和进程信息输出成一个文件系统的格式保存在内存中，然后将内存中的procfs映射到/proc目录，所以在系统启动后，用户就会发现/proc目录下有很多文件和目录。实际上，相当多的系统信息监控程序（如ps、vmstat）都只是读取procfs中的文件然后输出而已。




而对于内核参数，内核将其输出成了一个文件，每个内核参数都对应procfs中的一个文件，你可以通过对文件的读/写实现读取/更改内核参数。




总的来说，**用户可以使用procfs 来访问 Linux 内核的内容。****这个伪文件系统在内核空间和用户空间之间打开了一个通信窗口，使得用户能够通过对procfs中文件的读写实现对内核中数据的读写。**




 




用文字来描述终归显得有些苍白，下面我会结合实际的操作来进一步描述procfs。（更多关于procfs中内容的描述可以查看proc的man page）




首先我们知道procfs一般被挂载到/proc目录下，所以我们先查看一下/proc目录的大小。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-du.jpg)




可以发现/proc目录的大小是0，但是这里面确实有许多的文件，这就是因为proc目录只是对内存中一段空间的映射，并不是实际存在的。




下面我们查看一下/proc目录的内容。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc.jpg)




在这个目录下我们可以看见许多以数字命名的目录，实际上这里的每一个数字都是一个PID，你可以在进程表中找到和这些数字一一对应的进程。也就是说，内核将每一个进程信息输出成了一个目录，目录以进程的PID命名。接下来我们查看一下877的内容。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877.jpg)







在里面我们可以看见很多进程相关的信息，下面我就挑几个常用的文件讲一下它们的作用 : 




**cmdline**: 该进程启动时所使用的命令。我们可以看见877进程其实就是sshd进程。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-cmdline.jpg)




**cwd**: cwd实际上是一个符号链接，指向进程的当前工作目录。我们发现sshd进程的工作目录是根目录。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-cwd.jpg)




**environ**: 进程使用到的环境变量的值。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-environ.jpg)




**exe**: exe实际上是一个符号链接，指向进程的可执行文件。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-exe.jpg)




**fd**: 进程所使用的所有文件描述符（file description）。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-fd.jpg)




**maps**: 进程的可执行程序和使用到的库文件的内存映射表（在我的 [linux进程管理之进程管理工具2](http://118.25.17.78/blog/2018/06/15/linux%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86%E4%B9%8B%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B72/#pmap) 中的pmap中有详细解释）。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-maps.jpg)




**mem**: 进程拥有的内存空间。




**root**: 进程的根目录。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-root.jpg)




**status**: 便于阅读的进程状态信息（还有不便阅读的stat和statm）。




![](http://118.25.17.78/wp-content/uploads/2018/07/proc-877-status.jpg)




除了这些数字目录之外，我们还看见了许多其他的目录和文件，这些文件大多与进程无关，是一些cpu信息、内存信息以及设备信息。下面我们来看一些常用的文件。




**/proc/cmdline** 启动内核时的命令及内核参数。




**/proc/cpuinfo** 查看cpu信息。




**/proc/filesystems** 查看内核支持的文件系统。




**/proc/ioports** 查看当前正在使用的IO端口。




**/proc/kmsg** 内核输出的信息。也可以在syslog中查看。




**/proc/loadavg** 系统的平均负载情况。




**/proc/meminfo** 内存使用情况，包括物理内存和交换分区。（free命令读取的就是该文件）




**/proc/modules** 内核当前加载的模块。（lsmod命令读取的就是该文件，但是输出更加易读）




**/proc/mounts** 查看已经被挂载的文件系统。




**/proc/net** 网络协议相关信息。




**/proc/swaps** 交换分区使用情况。




**/proc/version** 内核版本号。




**/proc/sys** sys目录包含了大量的文件和子目录，每个文件都指代一个内核变量，通过对这些文件的读写，我们可以实现对内核变量的读写，而且不需要对内核进行重新编译，甚至不需要重启操作系统。在我们修改内核变量前一定要先查阅过文档和源代码，保证无误了之后再对内核变量做修改，虽然修改内核变量有时候可以调优系统，但是同样的也会使得系统崩溃，而系统一旦崩溃我们除了重启别无他法。而修改内核变量的方法也很简单，因为它们被输出成了一个个文件，所以我们只需要使用echo命令将输出重定向到文件即可。同时我们也可以使用专用的命令sysctl实现对内核变量的修改。（只有root用户才能修改内核变量）







# sysfs




在上面我们讲了procfs，我们发现procfs中有各种各样的信息。但是让我们回到最初的procfs，实际上看名字也可以知道，procfs（process filesystem）在最初是用来访问进程信息的，因为以前访问进程信息非常麻烦，而且所有类Unix系统都应该遵循一切皆文件（everything is a file）的哲学思想，所以就将进程信息输出成为一个个文件并且挂载到/proc目录。后来人们发现这个方法非常有用，于是开始把各种不方便用户访问的信息放到里面（比如cpu信息、内存使用情况、设备信息）。这样做虽然方便了用户访问内核空间中的信息，但是却使得procfs中的结构混乱，包含了许多不是进程的信息，要知道procfs设计出来本是为了访问进程信息的。于是后来人们有添加了一个新的伪文件系统，命名为sys，用于保存系统信息。




但是和所有新生的工具一样，由于老一代的工具已经被大量应用程序所使用，一旦将老一代工具直接从系统上移除，将会为这些应用程序带来巨大的问题。procfs已经被众多的应用程序作为系统信息访问入口，所以我们不能轻易将procfs中的一些系统信息直接移到sysfs中，我们只能在sysfs中也保存一份，并且梳理一下sysfs的文件系统使其保持整洁。这就是为什么至今procfs中仍然有许多的非进程信息。




现在我们知道sysfs是什么了，实际上sysfs就是procfs的一部分，只不过这一部分不便于归类到procfs中，所以将其独立出来作为一个个体，就有了sysfs。




sysfs中主要保存的是硬件设备的信息以及驱动信息，以设备树的形式向用户空间提供直观的设备和驱动信息。之所以提到用户空间是因为内核是不需要用到sysfs的，内核本身就能够探测硬件，然后就能利用驱动去操作硬件，但是用户空间想要操作硬件就很麻烦了，回想一下用户是如何操作硬件的。没错，是通过/dev目录下的设备文件，那问题来了，设备文件是由谁创建的呢？




在Linux内核2.4之前，设备文件是提前创建好的，也就是说，在/dev目录下保存了成千上万个设备文件，对应市场上的大量常见设备，而我们实际在使用的设备文件却只有几十个，这就造成了大量的浪费。于是在Linux内核2.4之后，就引入了**devfs**，devfs 是一个虚拟的文件系统，挂载在`/dev`目录下。可以动态地为设备在 `/dev`下创建或删除相应的设备文件，只生成存在设备的节点。




但是在Linux2.6以后，由于devfs存在种种缺陷，devfs 被 **sysfs + udev** 所取代。sysfs + udev 在设计哲学和现实中的易用性都比 devfs 更优，自此 sysfs + udev 的组合走上 主线，直至目前(4.9.40)，依然作为 Linux 的设备管理手段。




所以在sysfs中保存着设备和驱动信息，而udev则在系统启动时扫描 sysfs ，然后根据规则创建相应的设备文件。而之后如果出现设备的热拔插事件，也会由udev负责动态地创建或删除相应的设备文件。规则一般位于 /etc/udev/rules.d/ 和 /lib/udev/rules.d/ 目录下。




 




# 修改内核变量




在前面我们提到了/proc/sys子目录，在这个目录下有大量的文件和目录，保存着众多的Linux内核变量，这些内核变量决定着内核的工作方式以及工作特性，还有内核子功能的开启与否，所以对于系统调优来说，熟悉内核变量是非常重要的，而且也是必须的，在修改任何一个内核变量之前你必须对这个内核变量有足够的了解，并且清楚地知道你在做什么。而修改内核变量通常有两种方法 : 使用输出重定向和sysctl命令。




## echo




由于内核变量在procfs中被输出成了一个个的文件，所以我们可以通过echo命令将输出重定向至目标文件，即可完成内核变量的修改，而且此修改是实时生效的。




**echo "VALUE" > /proc/sys/KERNEL/PARAMETER**




其中VALUE是为内核变量指定的值，KERNEL/PARAMETER为内核变量所在文件路径。




比如 /proc/sys/kernel/hostname 这个内核变量，它代表当前系统的主机名，也就是通过hostname看到的值，你可以直接修改这个内核变量来修改主机名。




![](http://118.25.17.78/wp-content/uploads/2018/07/echo-kernel-variables.jpg)







## sysctl




sysctl是一个用于实时修改位于/proc/sys目录下的内核变量的工具，它既可以读取内核变量的值，也可以修改之。sysctl修改的内核变量也是实时生效，同时sysctl也有自己的配置文件/etc/sysctl.conf 。




但是我们知道实时生效的内核变量都是保存在内核空间中的，一旦系统重启，这些变量就会丢失。不知道你是否还记得Linux系统的启动流程，在init为系统做初始化操作时，它会加载/etc/sysctl.conf文件，根据文件内容设置内核参数，所以写在/etc/sysctl.conf配置文件中的内核变量是永久有效的，但是修改配置文件无法立即生效。




**命令用法**




**sysctl [options] [variable[=value]] [...]**




**variable** 内核变量，和输出重定向所不同的是，这里的内核变量使用的是相对路径，比如 kernel/hostname ，因为sysctl管理的是/proc/sys目录下的内核变量，所以前面的路径（/proc/sys）就被省略了。除了使用 /作为分隔符外，也可以使用 . 作为分隔符，比如 kernel.hostname 。




**variable=value** 为指定的内核变量赋值。




 




**常用选项**




**-n, --values** 只打印内核变量的值。




**-e, --ignore** 如果指定的内核变量不存在，则忽略报错。




**-N, --names** 只打印内核变量的名称。




**-w, --write** 在使用 variable=value 为内核变量赋值时，必须使用-w选项。 




**-p[FILE], --load[=FILE]** 加载sysctl的配置文件，如果没有指定配置文件路径（FILE），则默认读取/etc/sysctl.conf。在修改完sysctl配置文件后，往往会使用该选项使修改立即生效。




**-a, --all** 显示所有当前可用的内核变量。




 




下面我们尝试修改一下/proc/sys/net/ipv4/ip_forward内核变量的值。这个内核变量表示核心转发功能，给定值为0表示关闭内核的核心转发功能，给定值为1表示开启内核的核心转发功能。核心转发功能在系统上有多块网卡时会有影响。如果你开启了核心转发功能，那么当一块网卡收到一个目标ip不是自己，但是是另一块网卡所在网络时，就会将数据包转发给另一块网卡，否则就直接丢弃数据包。一般情况下，核心转发功能都是关闭的，需要我们手动开启。




![](http://118.25.17.78/wp-content/uploads/2018/07/ip_forward-1.jpg)




还有一个常用的内核变量是/proc/sys/vm/drop_caches，将这个变量置为1表示清空所有缓存，包括buffer和cache，置为0则表示不清空。




接着我们通过修改配置文件，并且将其加载来修改一下hostname。




![](http://118.25.17.78/wp-content/uploads/2018/07/etc.sysctl.hostname.jpg)




首先我们通过输出追加重定向将kernel.hostname=edu.edmundx.com追加至/etc/sysctl.conf文件中，然后使用hostname查看，发现果然hostname没有修改。然后我们使用sysctl -p手动加载配置文件，再次查看hostname，发现hostname已经被修改。







# 参考




[What is the difference between /proc and /sys? - Quora](https://www.quora.com/What-is-the-difference-between-proc-and-sys)




[linux - What is the difference between procfs and sysfs? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/4884/what-is-the-difference-between-procfs-and-sysfs)




[Linux Filesystem Hierarchy: Chapter 1. Linux Filesystem Hierarchy - /proc](http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)




[sysfs、udev 和 它们背后的 Linux 统一设备模型 - 博客 - binsite](https://www.binss.me/blog/sysfs-udev-and-Linux-Unified-Device-Model/)




Wikipedia










































