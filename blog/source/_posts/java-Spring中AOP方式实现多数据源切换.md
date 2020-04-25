---
title: Spring中AOP方式实现多数据源切换
date: 2016-07-26 16:50:44
tags: java
---

>[spring](http://lib.csdn.net/base/17)动态配置多数据源，即在大型应用中对数据进行切分，并且采用多个[数据库](http://lib.csdn.net/base/14)实例进行管理，这样可以有效提高系统的水平伸缩性。而这样的方案就会不同于常见的单一数据实例的方案，这就要程序在运行时根据当时的请求及系统状态来动态的决定将数据存储在哪个数据库实例中，以及从哪个数据库提取数据。
       Spring2.x以后的版本中采用Proxy模式，就是我们在方案中实现一个虚拟的数据源，并且用它来封装数据源选择逻辑，这样就可以有效地将数据源选择逻辑从Client中分离出来。Client提供选择所需的上下文（因为这是Client所知道的），由虚拟的DataSource根据Client提供的上下文来实现数据源的选择。

***具体的实现就是，虚拟的DataSource仅需继承AbstractRoutingDataSource实现determineCurrentLookupKey（）在其中封装数据源的选择逻辑。***
 
### 1.动态配置多数据源
数据源的名称常量和一个获得和设置上下文环境的类，主要负责改变上下文数据源的名称

```java
	public class DataSourceContextHolder {
	
		public static final String DATA_SOURCE_A = "dataSourceA";
		public static final String DATA_SOURCE_B = "dataSourceB";
		public static final String DATA_SOURCE_C = "dataSourceC";
		public static final String DATA_SOURCE_D = "dataSourceD";
		// 线程本地环境
		private static final ThreadLocal<String> contextHolder = new ThreadLocal<String>();
	
		// 设置数据源类型
		public static void setDbType(String dbType) {
			// System.out.println("此时切换的数据源为："+dbType);
			contextHolder.set(dbType);
		}
		// 获取数据源类型 
		public static String getDbType() {
			return (contextHolder.get());
		}
		// 清除数据源类型 
		public static void clearDbType() {
			contextHolder.remove();
		}
	}
```

### 2.建立[动态数据](http://baike.baidu.com/view/8740809.htm)源类，
注意，这个类必须继承AbstractRoutingDataSource，且实现方法determineCurrentLookupKey，该方法返回一个Object，一般是返回字符串：

```java
	public class DynamicDataSource extends AbstractRoutingDataSource {  
	    @Override  
	    protected Object determineCurrentLookupKey() {  
	    	System.out.println("此时获取到的数据源为："+DataSourceContextHolder.getDbType());
	        return DataSourceContextHolder.getDbType();  
	    }  
	}
 ```

***为了更好的理解为什么会切换数据源，我们来看一下`AbstractRoutingDataSource.java`源码,源码中确定数据源部分主要内容如下***

```java
    /** 
     * Retrieve the current target DataSource. Determines the 
     * {@link #determineCurrentLookupKey() current lookup key}, performs 
     * a lookup in the {@link #setTargetDataSources targetDataSources} map, 
     * falls back to the specified 
     * {@link #setDefaultTargetDataSource default target DataSource} if necessary. 
     * @see #determineCurrentLookupKey() 
     */  
    protected DataSource determineTargetDataSource() {  
        Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");  
        Object lookupKey = determineCurrentLookupKey();  
        DataSource dataSource = this.resolvedDataSources.get(lookupKey);  
        if (dataSource == null && (this.lenientFallback || lookupKey == null)) {  
            dataSource = this.resolvedDefaultDataSource;  
        }  
        if (dataSource == null) {  
            throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");  
        }  
        return dataSource;  
    }
```

上面这段源码的重点在于`determineCurrentLookupKey()`方法，这是`AbstractRoutingDataSource`类中的一个抽象方法，而它的返回值是你所要用的数据源`dataSource`的key值，有了这个key值，`resolvedDataSource`（这是个map,由配置文件中设置好后存入的）就从中取出对应的`DataSource`，如果找不到，就用配置默认的数据源。
看完源码，应该有点启发了吧，没错！你要扩展`AbstractRoutingDataSource`类，并重写其中的`determineCurrentLookupKey()`方法，来实现数据源的切换：  

### 3.编写spring的配置文件配置多个数据源

```xml
	<!-- 配置数据源  dataSourceA-->
	<bean name="dataSourceA"  class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
		<property name="url" value="${jdbc_url}" />
		<property name="username" value="${jdbc_username}" />
		<property name="password" value="${jdbc_password}" />

		<!-- 初始化连接大小 -->
		<property name="initialSize" value="20" />
		<!-- 连接池最大使用连接数量 -->
		<property name="maxActive" value="500" />
		<!-- 连接池最大空闲 -->
		<property name="maxIdle" value="20" />
		<!-- 连接池最小空闲 -->
		<property name="minIdle" value="0" />
		<!-- 获取连接最大等待时间 -->
		<property name="maxWait" value="60000" />
		
		<property name="validationQuery" value="${validationQuery}" />
		<property name="testOnBorrow" value="true" />
		<property name="testOnReturn" value="false" />
		<property name="testWhileIdle" value="true" />
		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="25200000" />
		<!-- 打开removeAbandoned功能 -->
		<property name="removeAbandoned" value="true" />
		<!-- 1800秒，也就是30分钟 -->
		<property name="removeAbandonedTimeout" value="1800" />
		<!-- 关闭abanded连接时输出错误日志 -->
		<property name="logAbandoned" value="true" />
		
		<!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
	    <property name="poolPreparedStatements" value="true" />
	    <property name="maxPoolPreparedStatementPerConnectionSize" value="1000" />
		
		<!-- 监控数据库 -->
		<!--<property name="filters" value="stat,log4j"/>-->
		<property name="filters" value="stat" /> 
	</bean>


	<!-- 配置数据源    dataSourceB    -->
	<bean name="dataSourceB"  class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
		<property name="url" value="${jdbc_new_url}" />
		<property name="username" value="${jdbc_new_username}" />
		<property name="password" value="${jdbc_new_password}" />
		<property name="initialSize" value="10" />
		<property name="maxActive" value="100" />
		<property name="maxIdle" value="10" />
		<property name="minIdle" value="0" />
		<property name="maxWait" value="10000" />
		<property name="validationQuery" value="${validationQuery3}" />
		<property name="testOnBorrow" value="false" />
		<property name="testOnReturn" value="false" />
		<property name="testWhileIdle" value="true" />
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<property name="minEvictableIdleTimeMillis" value="25200000" />
		<property name="removeAbandoned" value="true" />
		<property name="removeAbandonedTimeout" value="1800" />
		<property name="logAbandoned" value="true" />
		<property name="filters" value="stat" /> 
	</bean> 

	
	 <!-- 配置数据源    dataSourceC  -->
	 <bean name="dataSourceC"  class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
		<property name="url" value="${jdbc_mysql_url}" />
		<property name="username" value="${jdbc_mysql_username}" />
		<property name="password" value="${jdbc_mysql_password}" />
		<property name="initialSize" value="10" />
		<property name="maxActive" value="100" />
		<property name="maxIdle" value="10" />
		<property name="minIdle" value="0" />
		<property name="maxWait" value="60000" />
		<property name="validationQuery" value="${validationQuery2}" />
		<property name="testOnBorrow" value="false" />
		<property name="testOnReturn" value="false" />
		<property name="testWhileIdle" value="true" />
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<property name="minEvictableIdleTimeMillis" value="25200000" />
		<property name="removeAbandoned" value="true" />
		<property name="removeAbandonedTimeout" value="1800" />
		<property name="logAbandoned" value="true" />
		<property name="filters" value="stat" /> 
	</bean> 
 
	<!-- 多数据源配置 -->
	 <bean id="dataSource" class="top.suroot.base.datasource.DynamicDataSource">  
	    <!-- 默认使用dataSourceA的数据源 -->
	    <property name="defaultTargetDataSource" ref="dataSourceA"></property>
	    <property name="targetDataSources">  
	        <map key-type="java.lang.String">  
	            <entry value-ref="dataSourceA" key="dataSourceA"></entry>  
	            <entry value-ref="dataSourceB" key="dataSourceB"></entry>
	            <entry value-ref="dataSourceC" key="dataSourceC"></entry>
				<!--<entry value-ref="dataSourceD" key="dataSourceD"></entry>-->
	        </map>  
	    </property>  
   </bean>  
 ```

在这个配置中第一个property属性配置目标数据源，`<map key-type="java.lang.String">`中的`key-type`必须要和[静态](http://baike.baidu.com/view/612026.htm)键值对照类`DataSourceConst`中的值的类型相 同；`<entry key="User" value-ref="userDataSource"/>`中key的值必须要和静态键值对照类中的值相同，如果有多个值，可以配置多个`< entry>`标签。第二个`property`属性配置默认的数据源。  

### 4.动态切换是数据源 

```java
	DataSourceContextHolder.setDbType(DataSourceContextHolder.DATA_SOURCE_B);
```

-----
以上讲的是怎么动态切换数据源，下面要说下自定义注解、aop方式自动切换数据源 ，感兴趣的可以继续往下看看 

### 5.配置自定义注解

```java
	@Retention(RetentionPolicy.RUNTIME)
	@Target({ElementType.TYPE,ElementType.METHOD})
	public @interface DynamicDataSourceAnnotation {
		//dataSource 自定义注解的参数
		String dataSource() default DataSourceContextHolder.DATA_SOURCE_A;
	}
 ```

### 6.配置切面类  

```java
	@Aspect
	@Component
	@Order(1) 
	public class DynamicDataSourceAspect {
		
		
		@Before("@annotation(top.suroot.base.aop.DynamicDataSourceAnnotation)") //前置通知
		public void testBefore(JoinPoint point){
			//获得当前访问的class
			Class<?> className = point.getTarget().getClass();
			DynamicDataSourceAnnotation dataSourceAnnotation = className.getAnnotation(DynamicDataSourceAnnotation.class);
			if (dataSourceAnnotation != null ) {
				//获得访问的方法名
				String methodName = point.getSignature().getName();
				//得到方法的参数的类型
				Class[] argClass = ((MethodSignature)point.getSignature()).getParameterTypes();
				String dataSource = DataSourceContextHolder.DATA_SOURCE_A;
				try {
					Method method = className.getMethod(methodName, argClass);
					if (method.isAnnotationPresent(DynamicDataSourceAnnotation.class)) {
						DynamicDataSourceAnnotation annotation = method.getAnnotation(DynamicDataSourceAnnotation.class);
						dataSource = annotation.dataSource();
						System.out.println("DataSource Aop ====> "+dataSource);
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				DataSourceContextHolder.setDbType(dataSource);
			}
			
		}
	
		@After("@annotation(top.suroot.base.aop.DynamicDataSourceAnnotation)")   //后置通知
		public void testAfter(JoinPoint point){
			//获得当前访问的class
			Class<?> className = point.getTarget().getClass();
			DynamicDataSourceAnnotation dataSourceAnnotation = className.getAnnotation(DynamicDataSourceAnnotation.class);
			if (dataSourceAnnotation != null ) {
				//获得访问的方法名
				String methodName = point.getSignature().getName();
				//得到方法的参数的类型
				Class[] argClass = ((MethodSignature)point.getSignature()).getParameterTypes();
				String dataSource = DataSourceContextHolder.DATA_SOURCE_A;
				try {
					Method method = className.getMethod(methodName, argClass);
					if (method.isAnnotationPresent(DynamicDataSourceAnnotation.class)) {
						DynamicDataSourceAnnotation annotation = method.getAnnotation(DynamicDataSourceAnnotation.class);
						dataSource = annotation.dataSource();
					}
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				if(dataSource != null && !DataSourceContextHolder.DATA_SOURCE_A.equals(dataSource)) DataSourceContextHolder.clearDbType();
			}
		}
	}
```

### 7.在切入点添加自定义的注解

```java
	@Service("baseService")
	@DynamicDataSourceAnnotation
	public class BaseServiceImpl implements BaseService {
		
		@DynamicDataSourceAnnotation(dataSource = DataSourceContextHolder.DATA_SOURCE_B)
		public void changeDataSource() {
			System.out.println("切换数据源serviceImple");
		}
	}
```

### 8.当然注解扫描、和aop代理一定要在配置文件中配好
```java
	<!-- 自动扫描(bean注入) -->
	<context:component-scan base-package="top.suroot.*" />
	<!-- AOP自动代理功能 -->
	<aop:aspectj-autoproxy proxy-target-class="true"/>
```

大功告成，当调用`changeDataSource`方法的时候会进入切面类中切换数据源，方法调用完毕会把数据源切换回来。