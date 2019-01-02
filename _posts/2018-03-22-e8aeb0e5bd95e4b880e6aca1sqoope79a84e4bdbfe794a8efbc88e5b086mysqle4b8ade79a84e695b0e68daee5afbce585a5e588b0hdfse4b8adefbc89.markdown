---
author: edmund
comments: true
date: 2018-03-22 02:58:13+00:00
layout: post
link: http://118.25.17.78/blog/2018/03/22/%e8%ae%b0%e5%bd%95%e4%b8%80%e6%ac%a1sqoop%e7%9a%84%e4%bd%bf%e7%94%a8%ef%bc%88%e5%b0%86mysql%e4%b8%ad%e7%9a%84%e6%95%b0%e6%8d%ae%e5%af%bc%e5%85%a5%e5%88%b0hdfs%e4%b8%ad%ef%bc%89/
slug: '%e8%ae%b0%e5%bd%95%e4%b8%80%e6%ac%a1sqoop%e7%9a%84%e4%bd%bf%e7%94%a8%ef%bc%88%e5%b0%86mysql%e4%b8%ad%e7%9a%84%e6%95%b0%e6%8d%ae%e5%af%bc%e5%85%a5%e5%88%b0hdfs%e4%b8%ad%ef%bc%89'
title: 记录一次sqoop的使用（将mysql中的数据导入到HDFS中）
wordpress_id: 69
categories:
- Linux技术
post_format:
- 日志
tags:
- sqoop
- 命令
- 大数据
---

环境：CentOS7，mysql5.5，hadoop2.7.3

实验目的：将mysql的test数据库中的fav表中的数据导入到hadoop的hdfs中

首先保证hdfs正常运行，并且准备好sqoop的软件包，此处使用的是sqoop-1.99.7-bin-hadoop200.tar.gz


#### 安装sqoop


将二进制sqoop解压


<blockquote>[root@node1 ~]# tar xf sqoop-1.99.7-bin-hadoop200.tar.gz</blockquote>


将sqoop解压出来的目录移动到/usr/local目录下重命名为sqoop


<blockquote>[root@node1 ~]# mv sqoop-1.99.7-bin-hadoop200 /usr/local/sqoop/</blockquote>


如果有必要，可以将/usr/local/sqoop/bin 目录添加到PATH变量中


<blockquote>[root@node1 ~]# export PATH=/usr/local/sqoop/bin:$PATH</blockquote>




#### 启动sqoop服务




<blockquote>[root@node1 ~]# sqoop.sh server start
Setting conf dir: /usr/local/sqoop/bin/../conf
Sqoop home directory: /usr/local/sqoop
Starting the Sqoop2 server...
1 [main] INFO org.apache.sqoop.core.SqoopServer - Initializing Sqoop server.
33 [main] INFO org.apache.sqoop.core.PropertiesConfigurationProvider - Starting config file poller thread
Sqoop2 server started.</blockquote>


如果出现包含 “/etc/hadoop/conf/” 的错误信息，则去修改sqoop下conf目录中的sqoop.properties文件，找到org.apache.sqoop.submission.engine.mapreduce.configuration.directory=/etc/hadoop/conf/

将其修改为

org.apache.sqoop.submission.engine.mapreduce.configuration.directory=你的hadoop的配置文件所在目录


#### 启动sqoop客户端


进入sqoop的shell命令行界面


<blockquote>[root@node1 conf]# sqoop.sh client
Setting conf dir: /usr/local/sqoop/bin/../conf
Sqoop home directory: /usr/local/sqoop
Sqoop Shell: Type 'help' or '\h' for help.

sqoop:000></blockquote>





 	
  1. `# 设置交互的命令行打印更多信息，打印的异常信息更多`

 	
  2. `set option --name verbose --value true`

 	
  3. `# 查看连接`

 	
  4. `show version --all`




#### 创建Link


如果需要使用sqoop进行导入导出操作，需要先创建连接。
使用show conncetor命令可以查看sqoop支持的连接器。
而sqoop中默认提供了如下几种连接。

![](http://118.25.17.78/wp-content/uploads/2018/03/sqoop-connector.jpg)


#### 创建mysql-link


本例实现mysql-->hdfs的数据导入操作，所以需要创建一个mysql的link和hdfs的link。
**注意：在创建mysql-link的时候需要注意:**


<blockquote>

> 
> The MySQL JDBC Driver was removed from Sqoop distribution in order to ensure that the default distribution is fully Apache license compliant.
> 
> 

> 
> 

> 
> Sqoop发行版为了遵循apache协议，所以将MySQL JDBC Driver从sqoop中移除。所以为了创建mysql-link，我们需要手动下载一个mysql-connector，并且将其放入到sqoop的库文件目录中。
> 
> </blockquote>




此处我使用的是：mysql-connector-java-5.1.7-bin.jar

将其放到sqoop安装目录下的 server/lib下

[root@node1 sqoop]# ll server/lib/ | grep mysql
-rw-r--r--. 1 root root 709922 Mar 21 06:53 mysql-connector-java-5.1.7-bin.jar

下面就可以开始创建mysql-link，过程如下


<blockquote>sqoop:000> create link -c generic-jdbc-connector
Creating link for connector with name generic-jdbc-connector
Please fill following values to create new link object
Name: mysql-link

Database connection

Driver class: com.mysql.jdbc.Driver
Connection String: jdbc:mysql://10.60.72.28/test
Username: root
Password: ******
Fetch Size: [回车]
Connection Properties:
There are currently 0 values in the map:
entry# protocol=tcp
There are currently 1 values in the map:
protocol = tcp
entry#[回车]

SQL Dialect

Identifier enclose: [此处需要输入一个空格后回车]
New link was successfully created with validation status OK and name mysql-link</blockquote>


注意，除了Identifier enclose项需要输入一个空格，然后回车之外，其他留空的项都是直接回车。


#### 创建hdfs-link


创建HDFS的link的配置就比较简单，配置HDFS访问地址和hadoop配置文件目录路径即可


<blockquote>sqoop:000> create link -c hdfs-connector
Creating link for connector with name hdfs-connector
Please fill following values to create new link object
Name: hdfs-link

HDFS cluster

URI: hdfs://node1:9000
Conf directory: /usr/local/hadoop/etc/hadoop
Additional configs::
There are currently 0 values in the map:
entry#
New link was successfully created with validation status OK and name hdfs-link</blockquote>




#### 创建job


sqoop:000> create job -f mysql-link -t hdfs-link

即：from mysql to hdfs


<blockquote>sqoop:000> create job -f mysql-link -t hdfs-link
Creating job for links with from name mysql-link and to name hdfs-link
Please fill following values to create new job object
Name: m2h

Database source

Schema name: test(数据库名)
Table name: fav(数据表名)
SQL statement:
Column names:
There are currently 0 values in the list:
element#
Partition column:
Partition column nullable:
Boundary query:

Incremental read

Check column:
Last value:

Target configuration

Override null value:
Null value:
File format:
0 : TEXT_FILE
1 : SEQUENCE_FILE
2 : PARQUET_FILE
Choose: 0(选择文本文件) 
Compression codec:
0 : NONE
1 : DEFAULT
2 : DEFLATE
3 : GZIP
4 : BZIP2
5 : LZO
6 : LZ4
7 : SNAPPY
8 : CUSTOM
Choose: 0(选择NONE) 
Custom codec:
Output directory: /myhdfs/mysql（这里输入HDFS文件的目录，需要是空目录） 
Append mode:

Throttling resources

Extractors: 2（这里是参考官网填的2）
Loaders: 2（这里是参考官网填的2）

Classpath configuration

Extra mapper jars:
There are currently 0 values in the list:
element#
New job was successfully created with validation status OK and name m2h

> 
> #### 启动job
> 
> 
</blockquote>


如果上面的job创建时指定错了参数，可以使用


<blockquote>sqoop:000> update job -name m2h</blockquote>


修改参数。


<blockquote>sqoop:000> start job -name m2h</blockquote>





### 期间遇到的问题及解决方案




#### Exception: java.lang.Throwable Message: User: root is not allowed to impersonate root


错误原因：
该错误是因为在安装sqoop时，在hadoop的core-site.xml配置文件中配置的用户权限错误。
按照官网的配置如下。其中hadoop.proxyuser.sqoop2.hosts中的sqoop2是用户的意思，同理hadoop.proxyuser.sqoop2.groups中的sqoop2是用户组的意思。



 	
  1. `<property>`

 	
  2. `<name>hadoop.proxyuser.sqoop2.hosts</name>`

 	
  3. `<value>*</value>`

 	
  4. `</property>`

 	
  5. `<property>`

 	
  6. `<name>hadoop.proxyuser.sqoop2.groups</name>`

 	
  7. `<value>*</value>`

 	
  8. `</property>`


**解决方案：**
将sqoop2改为root即可，改完后如下：



 	
  1. `<property>`

 	
  2. `<name>hadoop.proxyuser.root.hosts</name>`

 	
  3. `<value>*</value>`

 	
  4. `</property>`

 	
  5. `<property>`

 	
  6. `<name>hadoop.proxyuser.root.groups</name>`

 	
  7. `<value>*</value>`

 	
  8. `</property>`


注意：改完后请重启hadoop服务和sqoop服务。


#### Exception: org.apache.sqoop.common.SqoopException Message: GENERIC_HDFS_CONNECTOR_0007:Invalid input/output directory - Unexpected exception




输入输出目录有问题，检查HDFS中是否存在相应目录即可，如果有存在相应目录，检查一下当前用户是否有权限操作该目录，如果没有，则自己创建一个新的目录。
