---
author: edmund
comments: true
date: 2018-05-02 05:48:01+00:00
layout: post
link: http://118.25.17.78/blog/2018/05/02/linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bdf%e5%91%bd%e4%bb%a4/
slug: linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bdf%e5%91%bd%e4%bb%a4
title: Linux文件系统管理之df命令
wordpress_id: 207
categories:
- Linux技术
post_format:
- 日志
tags:
- linux
- 命令
- 文件系统管理
---

# df命令


**NAME**


df - report file system disk space usage


**SYNOPSIS**


df [OPTION]... [FILE]...


**DESCRIPTION**


df displays the amount of disk space available on the file system containing each file name argument. 




If no file name is given, the space available on all currently mounted file systems is shown. 




Disk space is shown in 1K blocks by default, unless the environment variable POSIXLY_CORRECT is set, in which case 512-byte blocks are used.




df命令显示指定文件系统的磁盘空间使用情况，如果不给定参数的话，就显示所有已挂载文件系统的磁盘空间使用情况。




## 常用选项




** -h, --human-readable** 以人类易读的方式显示磁盘空间使用情况。




** -i, --inodes** 显示文件系统的inode使用情况而不是数据块使用情况。




**-P, --portability** 以**POSIX**兼容的方式输出信息。如果不使用此选项，在一行内容过长时，会自动换行显示，导致文本处理命令无法正常处理。使用该选项可以保证内容不会自动换行，而始终在一行内显示。








## 范例




![](http://118.25.17.78/wp-content/uploads/2018/05/df.jpg)



