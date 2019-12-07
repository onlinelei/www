---
title: org.springframework.beans.factory.annotation.Value 导致的依赖问题
date: 2017-2-16 12:18:44
tags: java
---
### 使用效果
自动注入需要使用@Value注解，这个注解的格式#{configProperties['mysql.url']}其中configProperties是我们在appContext.xml中配置的beanId，mysql.url是在properties文件中的配置项。

	@Service("MailSendServiceImpl")
	public class MailSendServiceImpl implements MailSendService{
		
		
		private JavaMailSenderImpl javaMailSender;
		
		@Value("${mail.host}")
		private String host;
		@Value("${mail.username}")
		private String username;
		@Value("${mail.password}")
		private String password;
		@Value("${mail.port}")
		private int port;
		@Value("${mail.smtp.auth}")
		private String auth;
		@Value("${mail.smtp.timeout}")
		private String timeout;
		@Value("${mail.default.from}")
		private String from;
	
		//TODO...
	}
<!--more-->
### 如何实现

在很多情况下我们需要在配置文件中配置一些属性，然后注入到bean中，Spring提供了```org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer```类，可以方便我们使用注解直接注入properties文件中的配置。
#### spring依赖
首先要新建maven项目，并在pom文件中添加spring依赖，如下pom.xml文件：

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	  <modelVersion>4.0.0</modelVersion>
	
	  <groupId>cn.outofmemory</groupId>
	  <artifactId>hellospring.properties.annotation</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
	  <packaging>jar</packaging>
	
	  <name>hellospring.properties.annotation</name>
	  <url>http://maven.apache.org</url>
	
	  <properties>
	    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	    <org.springframework-version>3.0.0.RC2</org.springframework-version>
	  </properties>
	
	  <dependencies>
	    <dependency>
	      <groupId>junit</groupId>
	      <artifactId>junit</artifactId>
	      <version>3.8.1</version>
	      <scope>test</scope>
	    </dependency>              
	    <!-- Spring -->
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-context</artifactId>
	        <version>${org.springframework-version}</version>
	    </dependency>
	  </dependencies>
	</project>

#### spring管理properties
要自动注入properties文件中的配置，需要在spring配置文件中添加org.springframework.beans.factory.config.PropertiesFactoryBean和org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer的实例配置：

如下spring配置文件appContext.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xmlns:context="http://www.springframework.org/schema/context"
	 xsi:schemaLocation="http://www.springframework.org/schema/beans
	 http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	 http://www.springframework.org/schema/context
	 http://www.springframework.org/schema/context/spring-context-3.0.xsd ">
	    <!-- bean annotation driven -->
	    <context:annotation-config />
	    <context:component-scan base-package="cn.outofmemory.hellospring.properties.annotation">
	    </context:component-scan>
	    <bean id="configProperties" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
	        <property name="locations">
	            <list>
	                <value>classpath*:application.properties</value>
	            </list>
	        </property>
	    </bean>
	    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
	        <property name="properties" ref="configProperties" />
	    </bean>    
	</beans>
### properties文件
properties文件的内容如下：

	mysql.url=mysql's url
	mysql.userName=mysqlUser
	mysql.password=mysqlPassword


### 可能出现的问题
maven聚合项目多个之间引用时需要项目中有如上的properties配置文件，当maven项目过多的聚合时使用注解的方式获取配置文件重的参数时，需要在每个项目中添加配置文件，相对比较繁琐
