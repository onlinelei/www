---
title: Spring中注解方式Aop实现
date: 2016-07-25 17:59:44
tags: java
---

>在软件业，AOP为Aspect Oriented Programming的缩写，意为：[面向切面编程](http://baike.baidu.com/view/1865230.htm)，通过[预编译](http://baike.baidu.com/view/176610.htm)方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是[OOP](http://baike.baidu.com/view/63596.htm)的延续，是软件开发中的一个热点，也是[Spring](http://baike.baidu.com/view/23023.htm)框架中的一个重要内容，是[函数式编程](http://baike.baidu.com/view/1711147.htm)的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的[耦合度](http://baike.baidu.com/view/1599212.htm)降低，提高程序的可重用性，同时提高了开发的效率。  
<!--more-->
##### 1.首先需要自定义注解类#####


	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.TYPE,ElementType.METHOD})
	public @interface DynamicDataSourceAnnotation {
		//dataSource 自定义注解的参数
		String dataSource() default DataSourceContextHolder.DATA_SOURCE_A;
	}

##### 2.自定义aop切面类#####

	@Aspect
	@Component
	@Order(1) 
	public class DynamicDataSourceAspect {
		
		
		@Before("@annotation(top.suroot.base.aop.DynamicDataSourceAnnotation)") //前置通知
		public void testBefore(){
			
			System.out.println("方法执行前执行！");
		} 
		@After("@annotation(top.suroot.base.aop.DynamicDataSourceAnnotation)")   //后置通知
		public void testAfter(JoinPoint point){
			
			System.out.println("方法执行后执行！");
		}
	
	
	}

##### 3.在方法中使用注解#####

	@RequestMapping("annotation")
	@DynamicDataSourceAnnotation(dataSource = DataSourceContextHolder.DATA_SOURCE_B)
	public String myAnnotation(HttpServletRequest request,HttpServletResponse response){
		System.out.println("自定义注解");
		return "hello";
	}

##### 4.在spring配置文件中配置好切入点的注解扫描#####

	<aop:config>
		<aop:pointcut id="transactionPointcut" expression="execution(* top.suroot.*.service..*Impl.*(..))" />
		<aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice" order="2"/>
	</aop:config>
