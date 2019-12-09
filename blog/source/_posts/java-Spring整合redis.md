---
title: Spring整合redis
date: 2017-3-28 14:45:38
tags: redis
---
新换环境,又有新东西可以学习了,哈皮! 抽空学习之余看了一下redis,个人对Springmvc的爱是忠贞不渝,所以整理了一下Springmvc整合redis的环境搭建.分享学习.
<!--more-->

pom.xml
-------

	<dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
      <version>1.6.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.8.0</version>
    </dependency>

    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi</artifactId>
      <version>3.15</version>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
      <version>3.15</version>
    </dependency>

redis.properties
----------------
	 # Redis settings
	redis.host=172.16.100.200
	redis.port=9736
	redis.pass=
	redis.dbIndex=0
	
	
	redis.maxIdle=300
	redis.maxTotal=600
	redis.maxWaitMillis=1000
	redis.testOnBorrow=true


spring_redis.xml
----------------



	<!-- scanner redis properties  -->
    <context:property-placeholder location="classpath:redis.properties" ignore-unresolvable="true"/>

    <!-- 连接池配置 -->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}" />
        <property name="maxTotal" value="${redis.maxTotal}" />
        <property name="maxWaitMillis" value="${redis.maxWaitMillis}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
    </bean>

    <!-- 数据库连接配置 -->
    <bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="hostName" value="${redis.host}"/>
        <property name="port" value="${redis.port}"/>
        <property name="password" value="${redis.pass}"/>
        <property name="database" value="${redis.dbIndex}"/>
        <property name="poolConfig" ref="poolConfig"/>
    </bean>

    <!--<bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">-->
        <!--<property name="connectionFactory"   ref="connectionFactory" />-->
    <!--</bean>-->

    <bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />

    <bean id="commonRedisTemplate" class="org.springframework.data.redis.core.RedisTemplate" >
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="keySerializer" ref="stringRedisSerializer" />
        <property name="hashKeySerializer" ref="stringRedisSerializer" />
        <property name="valueSerializer" ref="stringRedisSerializer" />
        <property name="hashValueSerializer" ref="stringRedisSerializer" />
    </bean>

    <bean id="stringRedisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate"
          p:connectionFactory-ref="connectionFactory" />

    <bean name="RedisCacheUtil" class="com.xxx.utils.RedisCacheUtil">
        <property name="redisTemplate" ref="commonRedisTemplate" />
        <property name="stringRedisTemplate" ref="stringRedisTemplate" />
    </bean>

RedisCacheUtils.java
--------------------
	
	import com.alibaba.fastjson.JSON;
	import com.alibaba.fastjson.JSONObject;
	import org.apache.commons.lang3.StringUtils;
	import org.springframework.dao.DataAccessException;
	import org.springframework.data.redis.connection.RedisConnection;
	import org.springframework.data.redis.core.BoundHashOperations;
	import org.springframework.data.redis.core.RedisCallback;
	import org.springframework.data.redis.core.RedisTemplate;
	import org.springframework.data.redis.core.StringRedisTemplate;
	import org.springframework.stereotype.Repository;
	
	import javax.annotation.Resource;
	import java.io.*;
	import java.util.Collection;
	import java.util.Map;
	import java.util.Set;
	import java.util.concurrent.TimeUnit;
	
	@Repository
	public class RedisCacheUtil {
	
		private static RedisTemplate<String, Object> redisTemplate;
	
	    private static StringRedisTemplate stringRedisTemplate;
	
	
	    public static Object get(String key) {
	        Object value = redisTemplate.opsForValue().get(key);
	        return value;
	    }
	
	    public static void set(String key, Object obj) {
	        if (obj == null) {
	            return;
	        }
	        redisTemplate.opsForValue().set(key, obj);
	    }
	
		public static Collection<String> keys(String pattern) {
			return redisTemplate.keys(pattern);
		}
	
		public static void delete(String key) {
			redisTemplate.delete(key);
		}
	
		public static void delete(Collection<String> keys) {
			redisTemplate.delete(keys);
		}
	
	    /**
	     * 取得缓存（int型）
	     * @param key
	     * @return
	     */
	    public static Integer getInt(String key){
	        String value = stringRedisTemplate.boundValueOps(key).get();
	        if(StringUtils.isNotBlank(value)){
	            return Integer.valueOf(value);
	        }
	        return null;
	    }
	
	    /**
	     * 取得缓存（字符串类型）
	     * @param key
	     * @return
	     */
	    public static String getStr(String key){
	        return stringRedisTemplate.boundValueOps(key).get();
	    }
	
		/**
		 * 获取缓存<br>
		 * 注：基本数据类型(Character除外)，请直接使用get(String key, Class<T> clazz)取值
		 * @param key
		 * @return
		 */
		public static Object getObj(String key){
			return redisTemplate.boundValueOps(key).get();
		}
	
		/**
		 * 获取缓存<br>
		 * 注：java 8种基本类型的数据请直接使用get(String key, Class<T> clazz)取值
		 * @param key
		 * @param retain    是否保留
		 * @return
		 */
		public static Object getObj(String key, boolean retain){
			Object obj = redisTemplate.boundValueOps(key).get();
			if(!retain){
				redisTemplate.delete(key);
			}
			return obj;
		}
	
		/**
		 * 获取缓存<br>
		 * 注：该方法暂不支持Character数据类型
		 * @param key   key
		 * @param clazz 类型
		 * @return
		 */
		@SuppressWarnings("unchecked")
		public static <T> T get(String key, Class<T> clazz) {
			return (T)redisTemplate.boundValueOps(key).get();
		}
	
		/**
		 * 将value对象以JSON格式写入缓存
		 * @param key
		 * @param value
		 * @param timeout 失效时间(秒)
		 */
		public static void setJson(String key,Object value,long timeout){
			stringRedisTemplate.opsForValue().set(key, JSONObject.toJSONString(value));
			if(timeout > 0){
				stringRedisTemplate.expire(key, timeout, TimeUnit.SECONDS);
			}
		}
	
		/**
		 * 更新key对象field的值
		 * @param key   缓存key
		 * @param field 缓存对象field
		 * @param value 缓存对象field值
		 */
		public static void setJsonField(String key, String field, String value){
			JSONObject obj = JSON.parseObject(stringRedisTemplate.boundValueOps(key).get());
			obj.put(field, value);
			stringRedisTemplate.opsForValue().set(key, obj.toJSONString());
		}
	
	
		/**
		 * 递减操作
		 * @param key
		 * @param by
		 * @return
		 */
		public static double decr(String key, double by){
			return redisTemplate.opsForValue().increment(key, -by);
		}
	
		/**
		 * 递增操作
		 * @param key
		 * @param by
		 * @return
		 */
		public static double incr(String key, double by){
			return redisTemplate.opsForValue().increment(key, by);
		}
	
		/**
		 * 获取double类型值
		 * @param key
		 * @return
		 */
		public static double getDouble(String key) {
			String value = stringRedisTemplate.boundValueOps(key).get();
			if(StringUtils.isNotBlank(value)){
				return Double.valueOf(value);
			}
			return 0d;
		}
	
		/**
		 * 设置double类型值
		 * @param key
		 * @param value
		 * @param time 失效时间(秒)
		 */
		public static void setDouble(String key, double value, long timeout) {
			stringRedisTemplate.opsForValue().set(key, String.valueOf(value));
			if(timeout > 0){
				stringRedisTemplate.expire(key, timeout, TimeUnit.SECONDS);
			}
		}
	
		/**
		 * 设置double类型值
		 * @param key
		 * @param value
		 * @param timeout 失效时间(秒)
		 */
		public static void setInt(String key, int value, long timeout) {
			stringRedisTemplate.opsForValue().set(key, String.valueOf(value));
			if(timeout > 0){
				stringRedisTemplate.expire(key, timeout, TimeUnit.SECONDS);
			}
		}
	
		/**
		 * 将map写入缓存
		 * @param key
		 * @param map
		 * @param timeout 失效时间(秒)
		 */
		public static <T> void setMap(String key, Map<String, T> map, long timeout){
			redisTemplate.opsForHash().putAll(key, map);
		}
	
		/**
		 * 向key对应的map中添加缓存对象
		 * @param key
		 * @param map
		 */
		public static <T> void addMap(String key, Map<String, T> map){
			redisTemplate.opsForHash().putAll(key, map);
		}
	
		/**
		 * 向key对应的map中添加缓存对象
		 * @param key   cache对象key
		 * @param field map对应的key
		 * @param value     值
		 */
		public static void addMap(String key, String field, String value){
			redisTemplate.opsForHash().put(key, field, value);
		}
	
		/**
		 * 向key对应的map中添加缓存对象
		 * @param key   cache对象key
		 * @param field map对应的key
		 * @param obj   对象
		 */
		public static <T> void addMap(String key, String field, T obj){
			redisTemplate.opsForHash().put(key, field, obj);
		}
	
		/**
		 * 获取map缓存
		 * @param key
		 * @param clazz
		 * @return
		 */
		public static <T> Map<String, T> mget(String key, Class<T> clazz){
			BoundHashOperations<String, String, T> boundHashOperations = redisTemplate.boundHashOps(key);
			return boundHashOperations.entries();
		}
	
	
		/**
		 * 获取map缓存中的某个对象
		 * @param key
		 * @param field
		 * @param clazz
		 * @return
		 */
		@SuppressWarnings("unchecked")
		public static <T> T getMapField(String key, String field, Class<T> clazz){
			return (T)redisTemplate.boundHashOps(key).get(field);
		}
	
		/**
		 * 删除map中的某个对象
		 * @author lh
		 * @date 2016年8月10日
		 * @param key   map对应的key
		 * @param field map中该对象的key
		 */
		public void delMapField(String key, String... field){
			BoundHashOperations<String, String, ?> boundHashOperations = redisTemplate.boundHashOps(key);
			boundHashOperations.delete(field);
		}
	
		/**
		 * 指定缓存的失效时间
		 *
		 * @author FangJun
		 * @date 2016年8月14日
		 * @param key 缓存KEY
		 * @param time 失效时间(秒)
		 */
		public static void expire(String key, long timeout) {
			if(timeout > 0){
				redisTemplate.expire(key, timeout, TimeUnit.SECONDS);
			}
		}
	
		/**
		 * 添加set
		 * @param key
		 * @param value
		 */
		public static void sadd(String key, String... value) {
			redisTemplate.boundSetOps(key).add(value);
		}
	
		/**
		 * 删除set集合中的对象
		 * @param key
		 * @param value
		 */
		public static void srem(String key, String... value) {
			redisTemplate.boundSetOps(key).remove(value);
		}
	
		/**
		 * set重命名
		 * @param oldkey
		 * @param newkey
		 */
		public static void srename(String oldkey, String newkey){
			redisTemplate.boundSetOps(oldkey).rename(newkey);
		}
	
		/**
		 * key是否存在
		 */
		public static boolean exists(String key) {
			return redisTemplate.hasKey(key);
		}
	
		public static void setTime(String key, Object obj, Long timeout,
				TimeUnit unit) {
			if (obj == null) {
				return;
			}
	
			if (timeout != null) {
				redisTemplate.opsForValue().set(key, obj, timeout, unit);
			}
			else {
				redisTemplate.opsForValue().set(key, obj);
			}
		}
	
		public RedisTemplate<String, Object> getRedisTemplate() {
			return redisTemplate;
		}
	
		public void setRedisTemplate(RedisTemplate<String, Object> redisTemplate) {
			this.redisTemplate = redisTemplate;
		}
	
		public static StringRedisTemplate getStringRedisTemplate() {
			return stringRedisTemplate;
		}
	
		public static void setStringRedisTemplate(StringRedisTemplate stringRedisTemplate) {
			RedisCacheUtil.stringRedisTemplate = stringRedisTemplate;
		}
	}
