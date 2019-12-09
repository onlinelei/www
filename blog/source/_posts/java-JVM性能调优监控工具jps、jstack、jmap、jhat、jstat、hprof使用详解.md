---
title: JVM性能调优监控工具jps、jstack、jmap、jhat、jstat、hprof使用详解
date: 2017-4-26 19:59:35
tags: java
---

>JDK本身提供了很多方便的JVM性能调优监控工具，除了集成式的VisualVM和jConsole外，还有jps、jstack、jmap、jhat、jstat、hprof等小巧的工具，本博客希望能起抛砖引玉之用，让大家能开始对JVM性能调优的常用工具有所了解。  

现实企业级Java开发中，有时候我们会碰到下面这些问题：

* OutOfMemoryError，内存不足

* 内存泄露

* 线程死锁

* 锁争用（Lock Contention）

* Java进程消耗CPU过高

* ......  

这些问题在日常开发中可能被很多人忽视（比如有的人遇到上面的问题只是重启服务器或者调大内存，而不会深究问题根源），但能够理解并解决这些问题是Java程序员进阶的必备要求。本文将对一些常用的JVM性能调优监控工具进行介绍，希望能起抛砖引玉之用。本文参考了网上很多资料，难以一一列举，在此对这些资料的作者表示感谢！关于JVM性能调优相关的资料，请参考文末。

<!--more-->

###A、 jps(Java Virtual Machine Process Status Tool)
  jps主要用来输出JVM中运行的进程状态信息。语法格式如下：

	jps [options] [hostid]   

如果不指定hostid就默认为当前主机或服务器。

命令行参数选项说明如下：

	-q 不输出类名、Jar名和传入main方法的参数
	-m 输出传入main方法的参数
	-l 输出main类或Jar的全限名
	-v 输出传入JVM的参数

比如下面：

	root@ubuntu:/# jps -m -l
	2458 org.artifactory.standalone.main.Main /usr/local/artifactory-2.2.5/etc/jetty.xml
	29920 com.sun.tools.hat.Main -port 9998 /tmp/dump.dat
	3149 org.apache.catalina.startup.Bootstrap start
	30972 sun.tools.jps.Jps -m -l
	8247 org.apache.catalina.startup.Bootstrap start
	25687 com.sun.tools.hat.Main -port 9999 dump.dat
	21711 mrf-center.jar