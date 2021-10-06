### Redis环境安装

**Redis官方是没有windows版本的redis,只有linux版本的。**网上的windows版本都是大牛改编！



原因:Redis的底层是采用Nio中的多路IO复用机制，能够非常好的支持并发，从而保证线程安全问题。

> Nio在不同的操作系统上的实现方式不同
>
> 在我们windows操作系统使用selector实现轮询的时间复杂度是o(n),而且还存在空轮询的情况，效率非常的低，其次是默认对我们轮询的数据有一定的限制，所以支持上万的tcp连接是非常难的。
>
> 在linux系统采用epoll实现事件驱动回调，不会存在空轮询的情况，只针对活跃的socket连接实现主动回调这样在性能上有大大的提升

简单来说，NIO在windows和linux系统的实现方式不同，在windows上性能很低。性能低的原因就是linux的epoll机制。



**Redis线程模型：Redis采用nio和io多路复用原则。**



### 安装步骤

1. 上传redis安装包
2. 解压Redis安装包

> tar  -zxvf redis-6.2.5.tar.gz

3. mkdir/usr/redis
4. 执行安装指令 make install PREFIX=/usr/redis 
5. 启动Redis cd/usr/redis/bin     ./redis-server

	### 修改后台启动

1. 将安装包中的redis.config文件复制到redis的bin目录下

> cp/usr/redis-5.0.6/redis.conf     /usr/redis/bin

2. 修改redis.conf文件

  将后台启动选项daemonize设为yes

![image-20210920123434796](C:\Users\王萌\Desktop\Redis学习.assets\image-20210920123434796.png)

3. 启动redis

> ./redis-server redis.conf   启动时要加上redis.conf



### 设置Redis账号密码

修改redis.conf,搜索requirepass ,修改密码

![image-20210920124616805](C:\Users\王萌\Desktop\Redis学习.assets\image-20210920124616805.png)

改完密码后，再使用redis-cli取值时会提示没有权限

输入指令

> auth  [密码] 登录

### 设置Redis运行ip访问

1. redis默认只支持本机访问，通过修改redis.conf运行外界访问

![image-20210920125504855](C:\Users\王萌\Desktop\Redis学习.assets\image-20210920125504855.png)

2. 开放redis端口号6379

> firewall-cmd --add-port=6379/tcp --permanent



**Redis默认情况下分为16个库，spring整合redis时，默认连接第一个库db0,可以通过配置文件指定**



### Redis数据结构

 **String,Hash,List,Set,Sorted-Set**



**String类型**

>String是redis最基本的类型，一个key对应一个value, String 类型是二进制安全的。意思是
>
>redis的string可以包含任何数据。比如jpg图片或者序列化的对象,Sring类型是Redis最基本的数据类型，
>
>一个键最大能存储**512MB**。

存放一个字符串

```
set name dog
```

取出一个值

```
get name  //"dog"
```



**hash类型**

相当于HashMap

存放几个键值对

```
hmset key1 zhangsan 21 sex 男 length 180
```

取出其中一个键值对

```
hmget key1 zhangsan
```



### List类型

Redis列表是简单的字符串列表,按照插入顺序排序。你可以添加一个元素到列表的头部(左边)或者尾部(右边)。

从左边向列表添加元素

>  LPUSH [列表名称]   【值1】 【值2】  【值3】

从右边(L=left,R=right)

> RPUSH

移除列表第一个元素

> BLPOP key1 [key2] timeout



### Set类型

Redis的Set 是String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。
Redis中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是0(1)。

向集合中添加一个或多个成员

>  SADD mayiktset mayikt mayikt02 mayikt03.

向集合中添加重复元素会被自动去重



### Sorted Set

Redis有序集合和集合一样也是string类型元素的集合，且不允许重复的成员。
不同的是每个元素都会关联一一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
有序集合的成员是唯一的,但分数(score)却可以重复。
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是0(1)。集合中 最大的成员，



### SpringBoot整合Redis

Redis如果存放一个java对象？

直接存放json类型

其他：xxl-sso使用二进制存放



1. 新建springboot项目
2. 引入redis依赖

```java
<!--redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

3. 导入依赖后，Ioc容器中已经存在StringRedisTemplate模板api，直接使用模板api

   ```java
   /**
    * @author wangm
    * @since 2021/9/20
    * 注意事项：对我们的redis的key设置一个有效期
    */
   public class RedisUtils {
   
       /**
        * redis模板
        */
       private static StringRedisTemplate stringRedisTemplate;
   
   
       public static void setString(String key, String value) {
           setString(key, value, 1000L);
       }
   
       public static void setString(String key, String value, Long timeout) {
           stringRedisTemplate.opsForValue().set(key, value);
           if (null != timeout) {
               stringRedisTemplate.expire(key, timeout, TimeUnit.SECONDS);
           }
       }
   
       public static String get(String key) {
           return stringRedisTemplate.opsForValue().get(key);
       }
   
       public static void init(StringRedisTemplate template) {
           stringRedisTemplate = template;
       }
   }
   ```

   **注意：向redis中添加值要设置一个过期时长**

4. 使用string数据类型，将json字符串存进redis

```java
@GetMapping("add")
public String add(User user) {
    String userString = JSONObject.toJSONString(user);
    RedisUtils.setString("user", userString);
    return "成功";
}
```

### Redis持久化

新建RedisTemplateUtils类，注入RedisTemplate<String,Object> redisTemplate变量。

```java
public class RedisTemplateUtils {
    
    private static RedisTemplate<String,Object> redisTemplate;
    

    public static void setObject(String key, Object value) {
        setObject(key, value, 1000L);
    }

    public static void setObject(String key, Object value, Long timeout) {
        redisTemplate.opsForValue().set(key, value);
        if (null != timeout) {
            redisTemplate.expire(key, timeout, TimeUnit.SECONDS);
        }
    }

    public static Object get(String key) {
        return redisTemplate.opsForValue().get(key);
    }


    public static void init(RedisTemplate template) {
        redisTemplate = template;
    }
}
```



存进redis的值要实现序列化接口，存入redis的结果是二进制

### springboot整合Redis注解版本

启动类加上注解@EnableCaching

controller上加注解@Cacheable

```java
/**
 * cacheNames:文件夹
 * key:真实的key
 * @return
 */
@GetMapping("all")
@Cacheable(cacheNames = "member",key = "'allUser'")
public List<User> list(){
    return userMapper.list();
}
```

**注意**:key值一定要加单引号''



### Mysql与Reids的数据不同步的问题如何解决？

方式1：直接清理Redis的缓存，重新查询数据库即可

方式2：直接采用mq订阅mysql binlog日志文件增量同步到Redis中。

方式3：使用alibaba的canal

### Redis持久化机制

Redis因为某种原因宕机之后，数据是不会丢失的。

原理就是持久化

大部分缓存框架都会有基本功能淘汰策略，持久机制

**Redis的持久化机制有两种：AOF,RDB（默认）**

全量同步与增量同步：

1. 每天定时（避开高峰期）或者采用一种周期的实现将数据拷贝到另外的一个地方。

> 频率不是很大，可能会产生数据丢失，假设900s以内，如果我们队redis做过10次写操作 将所有数据写入到硬盘中。

2. 增量同步采用行为操作对数据实现同步

> 频率比较高、对服务器同步的压力也非常大，保证数据不丢失

### RDB与AOF实现持久化的区别

RDB采用定时（全量）持久化机制，但是服务器因为某种原因宕机后可能数据会丢失，AOF是基于数据日志操作实现的持久化，所有AOF采用增量同步的方案。



​	Redis默认开启RDB存储。



**RDB实现持久化**

![image-20211006073446108](C:\Users\王萌\Desktop\Redis学习.assets\image-20211006073446108.png)

![image-20211006073225814](C:\Users\王萌\Desktop\Redis学习.assets\image-20211006073225814.png)



**AOF实现持久化**

在Redis的配置文件中存在三种同步方式，它分别是：

appendfsync always   #每次有数据修改发生时都会写入AOF文件，能够实时的保证数据的安全性。

appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。

appendfsync no             #从不同步。高效但是数据不会被持久化。

修改redis.conf将appendonly no改为yes

![image-20211006080419476](C:\Users\王萌\Desktop\Redis学习.assets\image-20211006080419476.png)



always方式每次写请求都产生数据同步，会执行大量io操作，影响效率。

every方式引入了缓冲区，每秒从缓冲区读取数据，提供了效率，建议使用every方式。可能会丢失1s内的数据，但是同步效率高。

S
