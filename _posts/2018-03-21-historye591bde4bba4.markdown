---
author: edmund
comments: true
date: 2018-03-21 03:07:21+00:00
layout: post
link: http://118.25.17.78/blog/2018/03/21/history%e5%91%bd%e4%bb%a4/
slug: history%e5%91%bd%e4%bb%a4
title: history命令
wordpress_id: 50
categories:
- Linux技术
post_format:
- 日志
tags:
- bash
- 命令
---

# history命令的作用




history顾名思义就是命令历史，这个命令会自动保存你曾经执行过的所有命令，通过执行history命令，你就能看到你的命令历史记录:




<blockquote>

> 
> [root@node1 ~]# history
> 
> 

> 
> 1 passwd  

2 shutdown -h now  

3 yum list all
> 
> 

> 
> ........
> 
> 

> 
> 480 cat  

481 cats  

482 hash  

483 history
> 
> 
</blockquote>




这些命令历史会在你键入新的命令时实时更新，新的命令会被追加到命令历史的尾部，并且保存在缓存中。而当你调用history -a命令或者正常关闭主机时，缓存中的新添加的命令历史会被追加写入到~/.bash_history文件中。而这个文件会在下次你登录shell时被读取，然后载入缓存中。




因此，你会发现，无论你登录登出多少次，你都能找到以前执行过的命令。







# history命令相关的环境变量




#### **1.HISTSIZE**




因为命令历史保存在缓存中，所以为了避免命令历史无限制的占用内存空间，所以需要限制缓存中命令历史的长度，HISTSIZE这个环境变量就是用来限制命令历史的长度:




<blockquote>

> 
> [root@node1 ~]# echo $HISTSIZE  

1000
> 
> 
</blockquote>




1000表示缓存中最多保存1000条命令历史，如果超过1000，比如有第1001条，那么第一条记录会被覆盖，以此类推，始终保持缓存中只有1000条命令历史。







#### **2.HISTFILESIZE**




除了避免占用内存过大之外，也需要避免~/bash_history这个命令历史文件过大，所以同样的，也需要限制文件的大小,HISTFILESIZE就是用来限制命令历史文件的大小的:




<blockquote>

> 
> [root@node1 ~]# echo $HISTFILESIZE  

1000
> 
> 
</blockquote>




1000表示文件中最多保存1000条命令历史，如果超过1000，同样的最前面的记录会被覆盖。







#### 3.HISTCONTROL




有时候也需要告知shell有哪些命令不需要记录，这时候就可以设置HISTCONTROL环境变量.这个环境变量有三种取值：




1、ignoredups 忽略重复的命令（必须在命令历史中是连续且完全相同的命令）




2、ignorespace 忽略以空白字符开头的命令




3、ignoreboth 同时忽略以上两项 ignoredups+ignorespace







[root@node1 ~]# echo $HISTCONTROL  

ignoredups







# history命令的常用操作




相信你已经发现了,在shell环境下使用方向键↑和↓可以来回切换命令历史，↑表示往前找一条命令，↓表示往后找一条命令，这个功能提供了极大地便捷性。




在观察history命令的输出时，你可以看到每条命令历史记录都有对应的记录号，通过 _! 记录号  _可以直接执行记录号对应的那条记录:




[root@node1 ~]# !482  

hash  

hits command  

1 /usr/sbin/service  

1 /usr/bin/jps  

5 /usr/bin/cat  

2 /usr/bin/ln  

1 /usr/sbin/setenforce  

1 /usr/bin/man




也可以通过_ !! _执行上一条命令:




[root@node1 ~]# !!  

hash  

hits command  

1 /usr/sbin/service  

1 /usr/bin/jps  

5 /usr/bin/cat  

1 /usr/bin/cat  

2 /usr/bin/ln  

1 /usr/sbin/setenforce  

1 /usr/bin/man




还可以通过 _!string_ 来从下往上查找以 string开头的命令并执行:




[root@node1 /]# !cat  

cat /etc/passwd  

root:x:0:0:root:/root:/bin/bash  

bin:x:1:1:bin:/bin:/sbin/nologin  

daemon:x:2:2:daemon:/sbin:/sbin/nologin  

adm:x:3:4:adm:/var/adm:/sbin/nologin




history NUM 可以只查看最近的NUM条命令(NUM为数字)




[root@node1 ~]# history 10  

431 whatis whatis  

432 man whatis  

433 mandb   

434 whatis man  

435 man halt  

436 man poweroff  

437 whatis halt  

438 man poweroff  

439 shutdown -h now  

440 history 10




# history命令的常用选项




<blockquote>

> 
> **history**
> 
> 

> 
> -a               将该session下的缓存中新添加的命令历史记录追加到历史记录文件中
> 
> 

> 
> -d offset     删除指定偏移下的命令历史记录
> 
> 

> 
> -c               删除所有缓存中的命令历史记录
> 
> 

> 
> -w              将缓存中的命令历史记录写入到历史记录文件中
> 
> 
</blockquote>






