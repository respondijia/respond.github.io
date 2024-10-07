# Spring Data Redis
java 操作 redis

Spring Data：通用的数据访问框架，定义了一组 增删改查 的接口

还可以操作：mysql、redis、jpa

使用方式如下：

1）引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.6.4</version>
</dependency>
```
2）配置 Redis 地址

```xml
spring:
## redis 配置
  redis:
    host: localhost
    port: 6379
    password:
    database: 1
```
## 使用默认的序列化
```java
@SpringBootTest
public class redisTest {

    @Resource
    private RedisTemplate redisTemplate;

    @Test
    void redisTest(){
        ValueOperations valueOperations = redisTemplate.opsForValue();
        valueOperations.set("name","respond");
        valueOperations.set("age",1);
    }
}
```
最后存储在redis中的k-v如下：
```
\xac\xed\x00\x05t\x00\x03age:{
    "fields": [
        {
            "value": 1
        }
    ],
    "annotations": [],
    "className": "java.lang.Integer",
    "serialVersionUid": 1360826667806852920
}
\xac\xed\x00\x05t\x00\x04name:\xac\xed\x00\x05t\x00\x07respond
```
可以看出默认的序列化有问题，这时候我们就要自己定义序列化
## 自定义序列化
为了防止写入 Redis 的数据乱码、浪费空间等，可以自定义序列化器。示例代码如下：
```java
package com.respond.jiaren.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;

@Configuration
public class RedisTemplateConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // redis连接工厂
        redisTemplate.setConnectionFactory(connectionFactory);
        // 自定义redis序列化
        redisTemplate.setKeySerializer(RedisSerializer.string());
        return redisTemplate;
    }
}
```
这是存储的k-v如下：
```
age:{
    "fields": [
        {
            "value": 1
        }
    ],
    "annotations": [],
    "className": "java.lang.Integer",
    "serialVersionUid": 1360826667806852920
}
name:respond
```
这时候序列化存储正常