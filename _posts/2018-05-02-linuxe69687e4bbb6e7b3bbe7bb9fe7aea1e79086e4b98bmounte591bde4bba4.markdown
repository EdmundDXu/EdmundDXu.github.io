---
author: edmund
comments: true
date: 2018-05-02 03:04:33+00:00
layout: post
link: http://118.25.17.78/blog/2018/05/02/linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bmount%e5%91%bd%e4%bb%a4/
slug: linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bmount%e5%91%bd%e4%bb%a4
title: Linux文件系统管理之mount命令
wordpress_id: 187
categories:
- Linux随笔
post_format:
- 日志
tags:
- 命令
- 文件系统管理
---

# 挂载




在Linux文件系统管理中有一个很重要的动作叫做挂载(mount)，挂载指的是将指定的设备与文件系统建立起关联关系，使得文件系统上的某个目录成为指定设备的访问入口的一个动作，而在Linux上能够完成这一个操作的命令就是**mount**命令。与挂载（mount）相对应的还有卸载（unmount）动作，也就是解除这种关联关系，使得目录仍然是一个目录，而不是作为设备的访问入口。能够完成卸载动作的命令是**umount**命令。




上面提到的与设备建立起关联关系的目录即为挂载点（**mount point**）。





## mount命令




**NAME**
mount - mount a filesystem




**SYNOPSIS**
mount [-lhV]




mount -a [-fFnrsvw] [-t vfstype] [-O optlist]




mount [-fnrsvw] [-t vfstype] [-o options] **device** **dir**




如果直接使用mount命令不加任何选项和参数，表示显示所有记录在/etc/mtab中的文件系统。(/etc/mtab中记录的文件系统为没有使用静默方式挂载的文件系统)





<blockquote>

> 
> [root@edu ~]# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=489496k,nr_inodes=122374,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
> 
> 
</blockquote>




**device**: 指明需要挂载的设备，可以使用以下几种方式指定设备。




（1）设备文件所在路径: 如/etc/sda1。




（2）UUID 设备的通用唯一识别码。




（3）设备文件的卷标（label）。




（4）伪文件系统的名称。




**dir**: 指明设备需要挂载到哪个目录下，需要给定该目录所在的路径。





### 常用选项:




**-a**: 自动挂载所有记录在/etc/fstab中的文件系统。




**-n**: 静默方式挂载。在挂载时不将文件系统信息写入/etc/mtab，但是仍然可以通过cat /proc/mounts查看此类文件系统的挂载情况。




**-r**: 以只读方式挂载文件系统。




**-w**: 以读写方式挂载文件系统。




**-L, --label label**:  挂载卷标（label）所指定的文件系统。




**-U, --uuid uuid**: 挂载UUID所指定的文件系统。




**-t**: 指定挂载的文件系统类型。




**-B, --bind**: 将文件系统树中的某个目录（subtree）再次挂载到别处，这意味着可以在多处访问到同一个目录的内容（但是不包括目录下的子挂载点）。见下面对-B和-R选项的详解。




**-R, --rbind**: 将文件系统树中的某个目录（subtree）及目录下的所有子挂载点再次挂载到别处，这意味着可以在多处访问到同一个目录的内容。见下面对-B和-R选项的详解。




**-o, --options options**: **options**为一些用逗号分隔开的选项，具体选项如下:




**async** 此选项会使得文件系统工作在异步模式下（异步模式：对该文件系统的写请求在内存中完成后，随即就会告知操作系统写请求已完成，然后内存中的修改会在随后某个时间点真正写入磁盘中，该模式工作效率高，但是存在写操作丢失的安全隐患。）




**sync** 此选项会使得文件系统工作在同步模式下（同步模式：对该文件系统的写请求在内存中完成后，还需要将修改写入到磁盘中，写入结束后才告知操作系统写请求已完成，该模式工作效率低，但是操作较为安全。）




**atime** 每次对文件的访问操作都会修改文件的atime时间戳。




**noatime** 对文件的访问操作不会修改文件的atime时间戳（文件访问频繁的服务器常用选项，因为atime的修改是文件系统层面的修改，需要发生I/O请求，非常影响性能）




**auto** 该文件系统可以通过-a选项被自动挂载。




**noauto** 该文件系统不能通过-a选项被自动挂载。




**dev** 将字符设备或块设备文件视为设备文件，即可以在该文件系统上挂载设备。




**nodev** 不将字符设备或块设备文件视为设备文件，即不允许在该文件系统上挂载设备。




**diratime** 每次对目录的访问操作都会修改目录的atime时间戳。




**nodiratime** 每次对目录的访问操作都不会修改目录的atime时间戳。




**exec** 允许可执行文件在该文件系统上运行为进程。




**noexec** 不允许可执行文件在该文件系统上运行为进程。




**suid** 允许SUID或SGID生效。




**nosuid** 不允许SUID或SGID生效。




**remount** 试图重新挂载文件系统，一般用于修改文件系统的挂载选项，该操作并不会改变设备或者挂载点。




**ro** 将文件系统挂载为只读文件系统。




**rw** 将文件系统挂载为读写文件系统。




**user** 允许普通用户挂载该文件系统。




**nouser** 不允许普通用户挂载该文件系统。




**defaults** 使用默认选项 **rw, suid, dev, exec, auto, nouser, and async.**





### 范例




将/dev/sda7重新挂载为ro,noatime。





<blockquote>

> 
> [root@edu ~]# mount /dev/sda7 /opt/
[root@edu ~]# mount | grep /opt
/dev/sda7 on /opt type btrfs (rw,relatime,seclabel,space_cache,subvolid=5,subvol=/)
[root@edu ~]# mount -o remount,ro,noatime /dev/sda7
[root@edu ~]# mount | grep /opt
/dev/sda7 on /opt type btrfs (ro,noatime,seclabel,space_cache,subvolid=5,subvol=/)
> 
> 
</blockquote>




### 对-B选项和-R选项的详细解释




在这里我给出了两个目录test和test1。





<blockquote>

> 
> [root@edu ~]# ls -l
total 4
drwxr-xr-x. 1 root root 28 May 2 09:03 test
drwxr-xr-x. 1 root root 20 May 2 09:03 test1
> 
> 
</blockquote>




在test目录中有一个子目录test2和一个文件test.file，在test1目录中有一个文件test1.file。





<blockquote>

> 
> [root@edu ~]# ls test
test2 test.file
[root@edu ~]# ls test1
test1.file
> 
> 
</blockquote>




在test2目录中有一个文件test2.nomount。





<blockquote>

> 
> [root@edu ~]# ls test/test2
test2.nomount
> 
> 
</blockquote>




现在我将/dev/sda7挂载到test2目录上，并且在test2中创建一个文件test2.mount。





<blockquote>

> 
> [root@edu ~]# mount /dev/sda7 test/test2
[root@edu ~]# ls test/test2
test2.mount
> 
> 
</blockquote>




然后我使用-B选项将test目录共享给test1目录。





<blockquote>

> 
> [root@edu ~]# mount -B test test1
[root@edu ~]# ls test
test2 test.file
[root@edu ~]# ls test1
test2 test.file
> 
> 
</blockquote>




可以发现，现在两个目录共享同一个内容，但是当我们查看两个目录下的test2目录，就会发现其实并不完全相同。





<blockquote>

> 
> [root@edu ~]# ls test/test2
test2.mount
[root@edu ~]# ls test1/test2/
test2.nomount
> 
> 
</blockquote>




test目录下的test2目录中是挂载后的文件系统，而test1目录下的test2目录是挂载前的文件系统，如果这时候想要将挂载后的文件系统也一并共享过来，就需要使用-R选项。





<blockquote>

> 
> [root@edu ~]# umount test1
[root@edu ~]# mount -R test test1
[root@edu ~]# ls test/test2
test2.mount
[root@edu ~]# ls test1/test2
test2.mount
> 
> 
</blockquote>




果然，现在test目录和test1目录已经完全相同。





## umount命令




** umount [options] <source> | <directory>**




**source**: 指明卸载的设备。




**directory**: 指明卸载的挂载点。




有时候会遇到无法卸载该设备的情况。这时候可以通过fuser命令查看哪个用户在使用该设备，然后酌情解决该问题。





<blockquote>

> 
> [root@edu /]# mount /dev/sda7 /data/
[root@edu /]# cd /data
[root@edu data]# umount /data
umount: /data: target is busy.
(In some cases useful info about processes that use
the device is found by lsof(8) or fuser(1))
[root@edu data]# fuser -v /data
USER PID ACCESS COMMAND
/data: root kernel mount /data
root 1012 ..c.. bash
> 
> 
</blockquote>




此时使用**fuser -km /data**即可将该进程强制结束。





## 文件系统挂载相关的配置文件/etc/fstab




上面讲到的mount命令挂载的文件系统都只对这次开机有效，一旦重启，就需要再次挂载。只有写在/etc/fstab中的文件系统才能够在开机时被自动挂载。




Linux内核在被boot loader加载进来后，会执行一个初始化脚本，这个脚本会调用mount命令读取/etc/fstab中的内容，然后将定义在/etc/fstab中的文件系统自动挂载到定义好的挂载点上。




当然，你也可以通过mount -a命令来自动挂载所有定义在/etc/fstab中的文件系统。




NAME




fstab - static information about the filesystems




SYNOPSIS




/etc/fstab




DESCRIPTION




The file fstab contains descriptive information about the various file systems.
fstab is only read by programs, and not written; it is the duty of the system
administrator to properly create and maintain this file. Each filesystem is
described on a separate line; fields on each line are separated by tabs or spa
ces. Lines starting with '#' are comments, blank lines are ignored. The order
of records in fstab is important because fsck(8), mount(8), and umount(8)
sequentially iterate through fstab doing their thing.




/etc/fstab中的每一行都描述了一个文件系统，每一行中的每个字段都用空格或者制表符（tab）分隔。#开头的行为注释信息，空白行会被忽略。





**第一个字段（指定的文件系统或伪文件系统）**




可以是本地的块设备文件或者是远程的文件系统，比如NFS。




本地的块设备文件可以通过设备文件路径、设备标签（Label）、设备的UUID、伪文件系统的名称来指定。远程的文件系统，如NFS，可以通过<host>:<dir>的形式来指明文件系统。







**第二个字段（文件系统的挂载点）**




指定文件系统需要挂载的路径，如果是swap分区，可以指定为swap。




**第三个字段（文件系统的类型）**




指定需要挂载的文件系统的类型。可以通过/proc/filesystems查看当前内核支持的文件系统。







**第四个字段（文件系统的挂载选项）**




指定文件系统的挂载选项，每个选项之间使用逗号分隔。具体挂载选项可以查看mount(8)，对于swap分区可以查看swapon(8)。







**第五个字段（文件系统的转储/备份频率）**




指明文件系统是否需要备份以及备份的频率。0表示不需要备份，1表示每天备份，以此类推。（该字段一般都置为0，因为现在对数据的备份一般都有专门的工具支持，对目录或文件系统的备份也会使用定时计划来完成。）




**第六个字段（文件系统的自检次序）**




指明文件系统在开机后是否需要自检（调用fsck命令），如果需要自检，那么指明每个文件系统的自检次序。0表示不自检，1表示优先自检，一般rootfs都需要置为1，而其他文件系统都置为2。一般来说，自己挂载的文件系统都置为0.





### 范例




一般来说，如果需要挂载一个光盘或者是镜像文件，可以通过mount -r /dev/sr0命令来进行手动挂载，或者通过将其定义在/etc/fstab中自动挂载。





<blockquote>

> 
> /dev/sr0/        media           iso9660            defaults,ro         0 0
> 
> 
</blockquote>



