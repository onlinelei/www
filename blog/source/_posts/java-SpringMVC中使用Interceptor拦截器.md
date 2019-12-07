---
title: SpringMVC中使用Interceptor拦截器
date: 2016-09-15 17:59:44
tags: java
---

>SpringMVC 中的Interceptor 拦截器也是相当重要和相当有用的，它的主要作用是拦截用户的请求并进行相应的处理。比如通过它来进行权限验证、判断用户是否登陆、登录log记录等  

<!--more-->
SpringMVC 中的Interceptor 拦截请求是通过HandlerInterceptor 来实现的。在SpringMVC 中定义一个Interceptor 非常简单，主要有两种方式，第一种方式是要定义的Interceptor类要实现了Spring 的HandlerInterceptor 接口，或者是这个类继承实现了HandlerInterceptor 接口的类，比如Spring 已经提供的实现了HandlerInterceptor 接口的抽象类HandlerInterceptorAdapter ；第二种方式是实现Spring的WebRequestInterceptor接口，或者是继承实现了WebRequestInterceptor的类。
#### 一.创建拦截器类####
##### 1.实现HandlerInterceptor接口

	package top.suroot.base.interceptor;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	
	import org.apache.log4j.Logger;
	import org.springframework.web.servlet.HandlerInterceptor;
	import org.springframework.web.servlet.ModelAndView;
	
	public class TestInterceptor implements HandlerInterceptor{
	
	    /**
	     * preHandle方法是进行处理器拦截用的，顾名思义，该方法将在Controller处理之前进行调用，SpringMVC中的Interceptor拦截器是链式的，可以同时存在 
	     * 多个Interceptor，然后SpringMVC会根据声明的前后顺序一个接一个的执行，而且所有的Interceptor中的preHandle方法都会在 
	     * Controller方法调用之前调用。SpringMVC的这种Interceptor链式结构也是可以进行中断的，这种中断方式是令preHandle的返 
	     * 回值为false，当preHandle的返回值为false的时候整个请求就结束了。 
	     */
		public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1,
				Object arg2) throws Exception {
			// TODO Auto-generated method stub
			return false;
		}
		/** 
	     * 这个方法只会在当前这个Interceptor的preHandle方法返回值为true的时候才会执行。postHandle是进行处理器拦截用的，它的执行时间是在处理器进行处理之 
	     * 后，也就是在Controller的方法调用之后执行，但是它会在DispatcherServlet进行视图的渲染之前执行，也就是说在这个方法中你可以对ModelAndView进行操 
	     * 作。这个方法的链式结构跟正常访问的方向是相反的，也就是说先声明的Interceptor拦截器该方法反而会后调用，这跟Struts2里面的拦截器的执行过程有点像， 
	     * 只是Struts2里面的intercept方法中要手动的调用ActionInvocation的invoke方法，Struts2中调用ActionInvocation的invoke方法就是调用下一个Interceptor 
	     * 或者是调用action，然后要在Interceptor之前调用的内容都写在调用invoke之前，要在Interceptor之后调用的内容都写在调用invoke方法之后。 
	     */  
		public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1,
				Object arg2, ModelAndView arg3) throws Exception {
			// TODO Auto-generated method stub
			
		}
		/** 
	     * 该方法也是需要当前对应的Interceptor的preHandle方法的返回值为true时才会执行。该方法将在整个请求完成之后，也就是DispatcherServlet渲染了视图执行， 
	     * 这个方法的主要作用是用于清理资源的，当然这个方法也只能在当前这个Interceptor的preHandle方法的返回值为true时才会执行。 
	     */  
		public void afterCompletion(HttpServletRequest arg0,
				HttpServletResponse arg1, Object arg2, Exception arg3)
				throws Exception {
			// TODO Auto-generated method stub
			
		}
	
	}

##### 2.实现WebRequestInterceptor 接口**

	package top.suroot.base.interceptor;
	
	import org.apache.log4j.Logger;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.context.request.WebRequest;
	import org.springframework.web.context.request.WebRequestInterceptor;
	
	public class TestInterceptor implements WebRequestInterceptor {
		
		/** 
	     * 在请求处理之前执行，该方法主要是用于准备资源数据的，然后可以把它们当做请求属性放到WebRequest中 
	     */  
		public void preHandle(WebRequest request) throws Exception {
			request.setAttribute("request", "request", WebRequest.SCOPE_REQUEST);//这个是放到request范围内的，所以只能在当前请求中的request中获取到  
	        request.setAttribute("session", "session", WebRequest.SCOPE_SESSION);//这个是放到session范围内的，如果环境允许的话它只能在局部的隔离的会话中访问，否则就是在普通的当前会话中可以访问  
	        request.setAttribute("globalSession", "globalSession", WebRequest.SCOPE_GLOBAL_SESSION);//如果环境允许的话，它能在全局共享的会话中访问，否则就是在普通的当前会话中访问  
		}
		/** 
	     * 该方法将在Controller执行之后，返回视图之前执行，ModelMap表示请求Controller处理之后返回的Model对象，所以可以在 
	     * 这个方法中修改ModelMap的属性，从而达到改变返回的模型的效果。 
	     */  
		public void postHandle(WebRequest request, ModelMap arg1) throws Exception {
			// TODO Auto-generated method stub
	
		}
		 /** 
	     * 该方法将在整个请求完成之后，也就是说在视图渲染之后进行调用，主要用于进行一些资源的释放 
	     */  
		public void afterCompletion(WebRequest request, Exception arg1)
				throws Exception {
			// TODO Auto-generated method stub
	
		}
	
	}

#### 二.把定义的拦截器类加到SpringMVC的拦截体系中####

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:mvc="http://www.springframework.org/schema/mvc" 
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:p="http://www.springframework.org/schema/p" 
		xmlns:context="http://www.springframework.org/schema/context"
		xmlns:aop="http://www.springframework.org/schema/aop"
		xsi:schemaLocation="http://www.springframework.org/schema/aop 
		http://www.springframework.org/schema/aop/spring-aop-3.1.xsd
		http://www.springframework.org/schema/context 
		http://www.springframework.org/schema/context/spring-context-3.0.xsd 
		http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
		http://www.springframework.org/schema/mvc 
		http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">
	
		 <!--  配置mvc的拦截器 可以配置多个 -->
	    <mvc:interceptors>
	        <mvc:interceptor><!-- 访问拦截器 -->
	            <mvc:mapping path="/**"/><!--  需要被拦截的路径 -->
	            <bean class="top.suroot.base.interceptor.TestInterceptor"/><!--  拦截处理的interceptor bean-->
	        </mvc:interceptor>
	    </mvc:interceptors>
	</beans>

由上面的示例可以看出可以利用mvc:interceptors标签声明一系列的拦截器，然后它们就可以形成一个拦截器链，拦截器的执行顺序是按声明的先后顺序执行的，先声明的拦截器中的preHandle方法会先执行，然而它的postHandle方法和afterCompletion方法却会后执行。
在mvc:interceptors标签下声明interceptor主要有两种方式：


1. 直接定义一个Interceptor实现类的bean对象。使用这种方式声明的Interceptor拦截器将会对所有的请求进行拦截。
2. 使用mvc:interceptor标签进行声明。使用这种方式进行声明的Interceptor可以通过mvc:mapping子标签来定义需要进行拦截的请求路径。 

经过上述两步之后，定义的拦截器就会发生作用对特定的请求进行拦截了。