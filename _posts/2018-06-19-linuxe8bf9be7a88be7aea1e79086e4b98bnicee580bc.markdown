---
author: edmund
comments: true
date: 2018-06-19 10:44:06+00:00
layout: post
link: http://118.25.17.78/blog/2018/06/19/linux%e8%bf%9b%e7%a8%8b%e7%ae%a1%e7%90%86%e4%b9%8bnice%e5%80%bc/
slug: linux%e8%bf%9b%e7%a8%8b%e7%ae%a1%e7%90%86%e4%b9%8bnice%e5%80%bc
title: Linux进程管理之Nice值
wordpress_id: 426
categories:
- Linux随笔
post_format:
- 日志
tags:
- linux
- 命令
- 进程优先级
- 进程管理
---

# [Nice值](https://en.wikipedia.org/wiki/Nice_(Unix))




**Nice值**是类UNIX操作系统中表示静态优先级的数值。每个进程都有自己的静态优先级，优先级越高（数值越小），程序越优先执行。Nice值的范围是-20~+19，拥有Nice值越大的进程的实际优先级越小（即Nice值为+19的进程优先级最小，为-20的进程优先级最大），进程的默认Nice值是从父进程继承而来的，默认进程的Nice值是0。由于Nice值是静态优先级，所以一经设定，就不会再被内核修改，直到被重新设定。Nice值只起干预CPU时间分配的作用，实际中的细节，由动态优先级决定。




至于为什么叫做Nice值，你可以这么理解。Nice的意思是优雅，看到优雅就能想到绅士，而作为一个绅士的进程，越是绅士（nice值越高）就越会谦让其他进程（优先级越低）。




需要注意的是，任何用户都可以修改进程的nice值，但是普通用户只能将nice值调高（优先级降低），只有管理员（root用户）可以随意修改进程的nice值。







# nice命令




nice命令可以以指定的nice值启动一个进程。需要注意的是，nice命令无法修改运行中的进程的nice值。




nice命令的用法非常简单 : **nice -n NICE COMMAND** 。




nice命令使用-n选项指定nice值，使用后面的COMMAND为需要执行的命令。如果不指定nice值，则默认为10。如果不指定命令，则打印当前进程的nice值。



    
    [root@edu ~]# nice -n 5 vmstat 1 &> /dev/null &           #为vmstat进程指定nice值为5并后台启动
    [1] 1195
    [root@edu ~]# jobs             #查看后台运行的vmstat
    [1]+  Running                 nice -n 5 vmstat 1 &>/dev/null &
    [root@edu ~]# ps axo pid,comm,ni | grep vmstat         #使用ps 命令查看vmstat进程的nice值，发现nice值为5）
      1195 vmstat            5
    







# renice命令




renice命令可以修改运行中的进程的nice值。renice的用法类似于nice，有区别的地方是指定进程的方式。




命令用法 : **renice [-n] priority [-gpu] identifier...**




其中priority为指定nice值，而identifier随着选项的不同而改变。




**-g, --pgrp pgid...** : 指定进程组ID（process group ID）。修改进程组的nice值后，进程组下的所有进程的nice值都会被修改。




**-u, --user name_or_uid...** : 指定的用户ID或用户名。修改用户的nice值后，所有属于该用户的进程的nice值都会被修改（进程的实际UID，而不是有效UID，比如使用SUID启动的passwd进程则不属于root用户，而属于启动它的用户）。




**-p, --pid pid...** : 指定进程的PID。（默认选项，如果不指定选项则默认接收进程PID）



    
    [root@edu ~]# renice -n 0 -u root        #修改所以属于root的进程的nice值为0
    0 (user ID) old priority -5, new priority 0
    [root@edu ~]# ps axo pid,comm,ni | grep vmstat   #之前修改的vmstat的nice值也从5变为了0
      1195 vmstat            0
    












