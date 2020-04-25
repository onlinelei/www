---
title: Tomcat + Redis 实现Session共享 
date: 2017-4-26 19:59:35
tags: java
---


Tomcat + Redis 实现Session共享 
-----------------------------



1.Tomcat配置
-----------
1.1引入jar包

	tomcat-redis-session-manager1.2.jar
	jedis-2.8.0.jar
	commons-pool2-2.2.jar

<!--more-->

1.2配置context.xml

> host:redis地址多个以,分割  
> prot:redis端口号  
> database:redis数据库
> maxInactiveInterval:session失效时间

	<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
		<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
				 host="172.16.100.200"
				 port="9736" 
				 database="15"    /> 

2.问题总结
---------

Q:Session失效时间优先级  
A:（1）RedisSessionManager，maxInactiveInterval="60"（秒）  
(2) web.xml=>session-config  <session-timeout>2</session-timeout>(分钟)   
测试结果失效时间为2分钟，web.xml时间会覆盖RedisSessionManager 时间的配置。 


Q:Session 失效时间如何计算。  
A:测试结果每次请求会刷新session 有效时间。登陆后无请求超过失效时间session无效。




1.用户登录后会保存session,再次登陆会创建新的session覆盖原有session (实验用的ent-op-war、exercise 系统)  
2.session 存放redis db 默认为db0 (可修改)
3.存入session的对象需要实现序列化接口











