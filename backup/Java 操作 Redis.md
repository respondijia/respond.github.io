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

# Redisson
Redisson 是一个 java 操作 Redis 的客户端， 提供了大量的分布式数据集来简化对 Redis 的操作和使用，可以让开发者像使用本地集合一样使用 Redis，完全感知不到 Redis 的存在。
2 种引入方式
spring boot starter 引入（不推荐，版本迭代太快，容易冲突）：https://github.com/redisson/redisson/tree/master/redisson-spring-boot-starter
直接引入：https://github.com/redisson/redisson#quick-start
```xml
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.37.0</version>
</dependency>
```
创建redisson客户端
```java
@Configuration
@ConfigurationProperties(prefix = "spring.redis")
@Data
public class RedissonConfig {

    private String host;

    private String port;

    @Bean
    public RedissonClient redissonClient(){
        // 1.配置redis redis://127.0.0.1:6379
        Config config = new Config();
        String redis = String.format("redis://%s:%s", host, port);
        config.useSingleServer().setAddress(redis).setDatabase(2);
        // 2.创建RedissonClient
        RedissonClient redisson = Redisson.create(config);
        return redisson;
    }
}
```
然后创建测试类进行测试，示例代码如下：
```java
@SpringBootTest
public class redissonTest {

    @Resource
    private RedissonClient redissonClient;

    @Test
    void test(){
        RList<String> list = redissonClient.getList("test-list");
        list.add("respond");
        System.out.println("test-list = " + list.get(0));
        //list.remove(0);
    }
}
```
输出结果如下：
```
test-list = respond
```
结果显示正确写入。
## 分布式锁保证定时任务不重复执行
```java
void testWatchDog() {
    RLock lock = redissonClient.getLock("yupao:precachejob:docache:lock");
    try {
        // 只有一个线程能获取到锁
        if (lock.tryLock(0, -1, TimeUnit.MILLISECONDS)) {
            // todo 实际要执行的方法
            doSomeThings();
            System.out.println("getLock: " + Thread.currentThread().getId());
        }
    } catch (InterruptedException e) {
        System.out.println(e.getMessage());
    } finally {
        // 只能释放自己的锁
        if (lock.isHeldByCurrentThread()) {
            System.out.println("unLock: " + Thread.currentThread().getId());
            lock.unlock();
        }
    }
}
```
### 注意：
#### 1.waitTime 设置为 0，只抢一次，抢不到就放弃
#### 2.注意释放锁要写在 finally 中