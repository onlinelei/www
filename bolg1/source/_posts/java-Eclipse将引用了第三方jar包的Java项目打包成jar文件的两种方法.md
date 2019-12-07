---
title: Eclipse将引用了第三方jar包的Java项目打包成jar文件的两种方法
date: 2016-09-13 17:59:44
tags: java
---

>如果想把自己的项目打成一个jar文件用cmd在有java运行环境的机器上执行，那么就可以将自己的项目打成jar包，但是如果项目中使用到了第三方的jar包怎么把这些jar文件也一起打包呢。   
<!--more-->
### 方案一：用Eclipse自带的Export功能

####步骤1：准备主清单文件 “MANIFEST.MF”
由于是打包引用了第三方jar包的Java项目，故需要自定义配置文件MANIFEST.MF，在该项目下建立文件MANIFEST.MF，
内容如下：  
>Manifest-Version: 1.0 
Class-Path: lib/commons-codec.jar lib/commons-httpclient-3.1.jar  
lib/commons-logging-1.1.jar lib/log4j-1.2.16.jar lib/jackson-all-1.8.5.jar 
Main-Class: main.KillCheatFans

第一行是MAINIFEST的版本，第二行Class-Path就指定了外来jar包的位置，第三行指定我们要执行的MAIN java文件。

这里要注意几点：   
>1、Class-Path: 和Main-Class: 后边都有一个空格，必须加上，否则会打包失败，错误提示为：Invalid header field； 
2、假设我们的项目打包后为KillCheatFans.jar，那么按照上面的定义，应该在 KillCheatFans.jar的同层目录下建立一个lib文件夹（即lib文件和打包的jar文件
在同一个目录下），并将相关的jar包放在里面。否则将会出现“Exception in thread "main" java.lang.NoClassDefFoundError”的错误； 
3、Main-Class后面是类的全地址，比如你的主文件是KillCheatFans.java，文件里打包为package com.main; 那么这里就写com.main.KillCheatFans，
不要加.java后缀，主文件地址写错将会出现“找不到或无法加载主类”的错误； 
4、写完Main-Class后一定要回车（即最后一行是空白行），让光标到下一行，这样你生成的jar包才能找到你的主class去运行，   

否则将会出现“jar中没有主清单属性”的错误。

###步骤2：右击Java工程选择Export—>选择JAR file—>Next

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-8f30bfe639c516f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###步骤3：选择要打包的文件，不需要的文件不必打包，减小打包后的jar文件大小，并进行选项配置如下

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-7687b4f47f4034e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里有几个选项：
>* Export generated class files and resources 表示只导出生成的.class文件和其他资源文件 
* Export all output folders for checked projects 表示导出选中项目的所有文件夹 
* Export java source file and resouces 表示导出的jar包中将包含你的源代码 *.java，如果你不想泄漏源代码，那么就不要选这项了 
* Export refactorings for checked projects 把一些重构的信息文件也包含进去
###步骤4：选择我们在第一步中自定义的配置文件路径，这一步很重要，不能采用默认选项

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-5600aaee85622983.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里解释一下配置项：
>* Generate the manifest file：是系统帮我们自动生成MANIFEST.MF文件，如果你的项目没有引用其他class-path，那可以选择这一项。 
* Use existing mainfest from workspace：这是可以选择我们自定义的.MF文件，格式如上所写，引用了第三方包时选用。 
* Seal content：要封装整个jar或者指定的包packet。 
* Main class：这里可以选择你的程序入口，将来打包出来的jar就是你这个入口类的执行结果。   

最后Finish，即生成了我们要的jar文件。

### 方案二：安装Eclipse打包插件Fat Jar
方案一对于含有较多第三方jar文件或含有第三方图片资源等就显得不合适，太繁琐。这时可以使用一个打包的插件—Fat Jar。     Fat Jar Eclipse Plug-In是一个可以将Eclipse Java Project的所有资源打包进一个可执行jar文件的小工具，可以方便的完成各种打包任务，我们经常会来打jar包，但是eclipse自带的打包jar似乎不太够用，Fat Jar是eclipse的一个插件，特别是Fat Jar可以打成可执行Jar包，并且在图片等其他资源、引用外包方面使用起来更方便。
###安装方法：
####1. Eclipse在线更新方法
Help > Install New Software > Add， 
name：Fat Jar 
location：http://kurucz-grafika.de/fatjar 
####2. Eclipse插件手动安装方法下载地址：
http://downloads.sourceforge.net/fjep/net.sf.fjep.fatjar_0.0.27.zip?modtime=1195824818&big_mirror=0 
将解压出的plugins中的文件复制到eclipse安装目录中的plugins目录下，然后重启eclipse即可。 
####使用方法：**
#####步骤1：右击工程项目选择Buile Fat Jar

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-478bbae521be3aef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####步骤2：配置jar文件存放目录，主Main文件等，如下图

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-1ae0832aa3c31b2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####步骤3：选择所要用到的第三方jar包

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-75e1b5bb88e3c252.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后Finish，即生成了我们要的jar文件，十分方便。

### 运行该jar文件有两种方式：
#####1. 在命令行下运行命令java -jar 你的jar文件名称，比如我的执行如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-a484009906b19c44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果在jar中有一些System.out.prinln语句（如上执行结果），运行后不想在控制台输出而是保存在文件中方便以后查看，可以用一下命令：java -jar KillCheatFans.jar > log.txt （这时命令行窗口不会有任何输出）输出信息会被打印到log.txt中，当然log.txt自动生成，并位于和KillCheatFans.jar一个目录中。
#####2. 新建一个批处理文件，如start.bat，内容为：java -jar KillCheatFans.jar，放在jar文件同一目录下即可，以后点击自动运行即可，更加方便。

>原文地址：http://www.cnblogs.com/lanxuezaipiao/p/3291641.html