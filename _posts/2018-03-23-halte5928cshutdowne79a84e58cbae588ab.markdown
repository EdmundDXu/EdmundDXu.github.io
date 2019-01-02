---
author: edmund
comments: true
date: 2018-03-23 04:23:33+00:00
layout: post
link: http://118.25.17.78/blog/2018/03/23/halt%e5%92%8cshutdown%e7%9a%84%e5%8c%ba%e5%88%ab/
slug: halt%e5%92%8cshutdown%e7%9a%84%e5%8c%ba%e5%88%ab
title: halt和shutdown的区别
wordpress_id: 88
categories:
- Linux随笔
post_format:
- 日志
tags:
- 命令
---

<blockquote>以下内容摘自[what is the difference between halt and shutdown commands](https://unix.stackexchange.com/questions/8690/what-is-the-difference-between-halt-and-shutdown-commands)

Generally, one uses the [`shutdown` command](http://linux.die.net/man/8/shutdown). It allows a time delay and warning message before shutdown or reboot, which is important for system administration of multiuser shell servers; it can provide the users with advance notice of the downtime.

As such, the shutdown command has to be used like this to halt/switch off the computer immediately (on Linux and FreeBSD at least):

>     
>     <code>shutdown -h now
>     </code>
> 
> 
Or to reboot it with a custom, 30 minute advance warning:

>     
>     <code>shutdown -r +30 "Planned software upgrades"
>     </code>
> 
> 
After the delay, `shutdown` tells [`init`](http://linux.die.net/man/8/init) to change to runlevel 0 (halt) or 6 (reboot). (Note that omitting `-h` or `-r` will cause the system to go into single-user mode (runlevel 1), which kills most system processes but does not actually halt the system; it still allows the administrator to remain logged in as root.)

Once system processes have been killed and filesystems have been unmounted, the system halts/powers off or reboots automatically. This is done using the [`halt` or `reboot` command](http://linux.die.net/man/8/halt), which syncs changes to disks and then performs the actual halt/power off or reboot.

On Linux, if `halt` or `reboot` is run when the system has not already started the shutdown process, it will invoke the `shutdown` command automatically rather than directly performing its intended action. However, [on systems such as FreeBSD](http://www.gsp.com/cgi-bin/man.cgi?section=8&topic=halt), these commands first log the action in [`wtmp`](http://www.gsp.com/cgi-bin/man.cgi?section=5&topic=wtmp) and then will _immediately_ perform the halt/reboot themselves, without first killing processes or unmounting filesystems.</blockquote>


简单地说，在大部分linux发行版 中，halt或者reboot命令在被调用时，如果先前没有执行过shutdown命令，且运行级别不处于0或6的时候，会先去调用shutdown命令，执行安全关机操作。也就是说，会先关闭所有进程，卸载所有文件系统，将内存中的内容同步到磁盘中，然后继续执行halt或reboot进行关机或者重启操作，并且会自动切断电源(poweroff)。
