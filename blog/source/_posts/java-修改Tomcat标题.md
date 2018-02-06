---
title: 修改Tomcat标题
date: 2016-09-09 17:59:44
tags: java
---

由于在用的tomcat服务启动后长时间不重启就会有问题，所以决定要做个计划任务，每天定时重启来解决。这一点在linux上实现起来非常简单，直接在crontab里加个任务就行了。对于多个tomcat，我们也可以利用ps、grep、awk等程序加以区分。而在windows服务器上启动多个tomcat时，默认Tomcat启动后的CMD窗口的标题名都是TOMCAT，而在服务器上部署多个Tomcat时，很难分出那个tomcat是那个项目的。
<!--more-->
而如果想通过计划任务定期重启就比较麻烦。不过这两有两种方法可以解决该问题：一种是通过增加成服务，对服务加上不同的描述。这样就可以通过tasklist /svc|find "string"进行区分了；另外一种方法是通过更改tomcat的title来识别。所以重点推荐这种方法。因为通过该方法，也比较方便查看和区分tomcat程序。
具体修改方法如下：
进入tomcat的bin目录，打开catalina.bat 。找到下面的内容：

	if not "%OS%" == "Windows_NT" goto noTitle 
	set _EXECJAVA=start "TOMCAT" %_RUNJAVA%

其中的“TOMCAT ”就是默认的title名称。这里建议改成服务名+所占端口的方式。这样看起来一目了然。因为这个更改不会影响到服务程序的启动。所以具体怎么改，随个人喜欢了！