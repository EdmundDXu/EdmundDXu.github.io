---
author: edmund
comments: true
date: 2018-05-02 05:27:48+00:00
layout: post
link: http://118.25.17.78/blog/2018/05/02/linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bdu%e5%91%bd%e4%bb%a4/
slug: linux%e6%96%87%e4%bb%b6%e7%b3%bb%e7%bb%9f%e7%ae%a1%e7%90%86%e4%b9%8bdu%e5%91%bd%e4%bb%a4
title: Linux文件系统管理之du命令
wordpress_id: 199
categories:
- Linux技术
post_format:
- 日志
tags:
- linux
- 命令
- 文件系统管理
---

# du命令




### **NAME**




du - estimate file space usage





### **SYNOPSIS**




du [OPTION]... [FILE]...
du [OPTION]... --files0-from=F





### **DESCRIPTION**




Summarize disk usage of each FILE, recursively for directories.




概述每个文件的磁盘使用情况，递归的显示每个目录中文件的磁盘使用情况。





### 常用选项




-s, --summarize 只显示给定文件的磁盘使用情况，如果该文件为目录，不对该目录进行递归显示。默认会递归显示目录的磁盘使用情况。




-h, --human-readable 以人类易读的方式显示磁盘空间使用情况。




--inodes 显示给定文件的inode使用情况，而不显示占用的磁盘空间。





### 范例




<blockquote>

> 
> [root@edu data]# du -sh /etc
31M /etc
[root@edu data]# du -s --inodes /etc
2338 /etc
> 
> 
</blockquote>
