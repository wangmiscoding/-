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

2. 关闭防火墙

> systemctl stop firewalld

