---
title: Linux常用命令
date: 2017-3-31 15:18:39
tags: Linux Redis
---
总结一些平时厂用的linux命令
<!--more-->
linux开启端口号
---------------
centos 7 

    开启端口
     
    firewall-cmd --zone=public --add-port=80/tcp --permanent
     
    命令含义：
     
    --zone #作用域
     
    --add-port=80/tcp  #添加端口，格式为：端口/通讯协议
     
    --permanent   #永久生效，没有此参数重启后失效
     
    重启防火墙
     
    firewall-cmd --reload
    查询端口开放
    firewall-cmd    --query-port=9200/tcp
    
查看端口开放
------------
关于nmap的使用，都可以长篇大写特写，这里不做展开。如下所示，nmap 127.0.0.1 查看本机开放的端口，会扫描所有端口。 当然也可以扫描其它服务器端口。

    [root@DB-Server Server]# nmap 127.0.0.1
     
    Starting Nmap 4.11 ( http://www.insecure.org/nmap/ ) at 2016-06-22 15:46 CST
    Interesting ports on localhost.localdomain (127.0.0.1):
    Not shown: 1674 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    25/tcp   open  smtp
    111/tcp  open  rpcbind
    631/tcp  open  ipp
    1011/tcp open  unknown
    3306/tcp open  mysql
     
    Nmap finished: 1 IP address (1 host up) scanned in 0.089 seconds
    You have new mail in /var/spool/mail/root
    [root@DB-Server Server]# 

删除
----

    关于date命令:
    date -d "-2 day"  #两天前
    date -d "-1 hour "  #一小时前
    复制代码
    关于删除指定时间之前的文件：
    find . -type f -mtime +2 |xargs rm -f  #两天前
    find . -type f -mmin +120 |xargs rm -f  #两小时前

配置环境变量
------------

    vim /etc/profile
    
    添加如下内容：JAVA_HOME根据实际目录来
    JAVA_HOME=/usr/java/jdk1.8.0_60
    CLASSPATH=$JAVA_HOME/lib/
    PATH=$PATH:$JAVA_HOME/bin
    export PATH JAVA_HOME CLASSPATH

	重启机器或执行命令 ：source /etc/profile
