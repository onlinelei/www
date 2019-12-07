---
title: java中Logger.getLogger
date: 2016-07-20 17:59:44
tags: java
---


>作为一名Java 程序员，最熟悉的、使用最多的调用恐怕莫过于System.out.print("");。当你没有调试工具而要跟踪一个变量的值得时候；当你需要显示捕获的Exception、Error的时候；当你想知道程序在运行的时候究竟发生了什么的时候，通常的做法就是调用System.out.print把他们在终端、控制台上打印出来。这种方式对于输出信息的分类、格式化及永久保存带来诸多不便。虽然我们可以把它写入一个文件然后进行分析，但是这要需要编写额外的程序代码，其成本不可忽视！而由此给目标系统本身增加的复杂程度不可避免的使开发、调试陷入一个深深的迷潭。  
<!--more-->
**有个简单的方法实现日志的记录**

	public class TestInterceptor implements HandlerInterceptor{
		//Logger 对象可以写在任何类中，我这里写在拦截器里面了
		static private final Logger  log = Logger.getLogger(TestInterceptor.class);
		//前置处理
		public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1,
				Object arg2) throws Exception {
			// TODO Auto-generated method stub
			log.info("Test Log info...."); //这里可以写入任何你想写到log里面的内容
			log.error("log error!");
			return true;
		}
	}
 
当然你要配置好你的log4j文件 然后去你的log文件里面找你输出的内容。