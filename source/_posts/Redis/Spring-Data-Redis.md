---
title: Spring-Data-Redis
date: 2019-11-11 23:41
tags: 
  - Redis
categories:
  - [Redis]
---

配置
```
    <!-- redis配置 -->
	<!-- jedis pool配置 -->
	<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxTotal" value="${redis.maxTotal}" />
		<property name="maxIdle" value="${redis.maxIdle}" />
		<property name="maxWaitMillis" value="${redis.maxWaitMillis}" />
		<property name="testOnBorrow" value="${redis.testOnBorrow}" />
	</bean>

	<!-- Jedis ConnectionFactory -->
	<bean id='jedisConnectionFactory'
		class='org.springframework.data.redis.connection.jedis.JedisConnectionFactory'>
		<property name="usePool" value="true"></property>
		<property name="hostName" value="${redis.host}" />
		<property name="port" value="${redis.port}" />
		<property name="password" value="${redis.pass}" />
		<property name="timeout" value="${redis.timeout}" />
		<property name="database" value="${redis.default.db}"></property>
		<constructor-arg index="0" ref="jedisPoolConfig" />
	</bean>

    <bean id="stringRedisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory" />
    </bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory" />
        <property name="keySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
        <property name="valueSerializer">
            <bean class="com.xxx.xxx.xxx.redis.serializer.JsonRedisSerializer" />
        </property>

        <property name="hashKeySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>

        <property name="hashValueSerializer">
            <bean class="com.xxx.xxx.xxx.redis.serializer.JsonRedisSerializer" />
        </property>
    </bean>
```


序列化操作类
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.SerializationException;

import com.alibaba.fastjson.JSON;

public class JsonRedisSerializer implements RedisSerializer<Object> {

    private static final String CHARSET_NAME = "UTF-8";

    private static final Logger logger = LoggerFactory.getLogger(JsonRedisSerializer.class);

    @Override
    public byte[] serialize(Object data) throws SerializationException {
        if (data == null) {
            return null;
        }

        try {
            // 先转成String
            String json = JSON.toJSONString(data);
            // 包装
            SerializerObject so = new SerializerObject();
            so.setClassName(data.getClass().getName());
            so.setJsonString(json);
            // 再转
            String serializerStr = JSON.toJSONString(so);

            return serializerStr.getBytes(CHARSET_NAME);
        } catch (Exception e) {
            logger.error("RedisSerializer序列化异常", e);
            throw new SerializationException("RedisSerializer序列化异常");
        }
    }

    @Override
    public Object deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null) {
            return null;
        }
        try {
            String serializerStr = new String(bytes, CHARSET_NAME);
            SerializerObject deSerObj = JSON.parseObject(serializerStr, SerializerObject.class);

            Class<?> t = Class.forName(deSerObj.getClassName());
            return JSON.parseObject(deSerObj.getJsonString(), t);

        } catch (Exception e) {
            logger.error("RedisSerializer反序列化异常", e);
            throw new SerializationException("RedisSerializer反序列化异常");
        }
    }

}

class SerializerObject {
    private String className;
    private String jsonString;

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }

    public String getJsonString() {
        return jsonString;
    }

    public void setJsonString(String jsonString) {
        this.jsonString = jsonString;
    }

}
```

简单例子

```
@Service("redisService")
public class RedisServiceImpl implements RedisService {

    @Resource(name = "stringRedisTemplate")
    private RedisTemplate<String, String> template;

    @Resource(name = "stringRedisTemplate")
    private ListOperations<String, String> listOps;

    @Resource(name = "stringRedisTemplate")
    private HashOperations<String, String, String> hashOps;

    @Resource(name = "stringRedisTemplate")
    private SetOperations<String, String> setOps;

    @Override
    public void delKey(String key) {
        template.delete(key);
    }

    @Override
    public boolean hasKey(String key) {
        return template.hasKey(key);
    }

    @Override
    public Long setAdd(String key, String... values) {
        return setOps.add(key, values);
        // return template.boundSetOps(key).add(values);
    }

    @Override
    public Long setSize(String key) {
        return setOps.size(key);
        // return template.boundSetOps(key).size();
    }

    @Override
    public Long rightPushList(String key, String value) {
        return listOps.rightPush(key, value);
        // return template.boundListOps(key).rightPush(value);
    }

    @Override
    public String leftPopList(String key) {
        return listOps.leftPop(key);
        // return template.boundListOps(key).leftPop();
    }

    @Override
    public Long listSize(String key) {
        return listOps.size(key);
        // return template.boundListOps(key).size();
    }
    ...
    ...
```

----


类|操作
--|--|
ValueOperations	|Redis String/Value 操作
ListOperations	|Redis List 操作
SetOperations	|Redis Set 操作
ZSetOperations	|Redis Sort Set 操作
HashOperations	|Redis Hash 操作

类|约束
--|--
BoundValueOperations|Redis String/Value key 约束
BoundListOperations	|Redis List key 约束
BoundSetOperations	|Redis Set key 约束
BoundZSetOperations	|Redis Sort Set key 约束
BoundHashOperations	|Redis Hash key 约束
