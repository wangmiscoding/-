### 什么是Zookeeper

> Zookeeper是一个开源的分布式协同服务系统。设计目标是将那些复杂且容易出错的分布式协同服务封装起来，抽象出一个高效可靠的原语集，并以一系列简单的接口提供给用户使用。

### 典型应用场景：

* 配置管理
* DNS服务
* 组成员管理
* 分布式锁

**Zookeeper适用于存储和协同相关的关键数据，不适合用于大数据量存储。**

### ZooKeeper数据模型

数据模型是层次模型。层次模型常见于文件系统。

1. 文件系统的树形结构便于表达数据之间的层次关系。
2. 文件系统的树形结构便于为不同的应用分配独立的命名空间。

ZooKeeper的层次模型称作data tree。Data tree的每个节点叫做znode。不同于文件系统，每个节点都可以保存数据。每个节点都有一个版本。版本从0开始计数。

### data tree接口

ZooKeeper对外提供一个用来访问data tree的简化文件系统API:

* 使用UNIX风格的路径名来定位znode，例如/A/X表示znode A的子节点X。
*  znode的数据只支持全量写入和读取，没有像通用文件系统那样支持部分写入和读取。data tree的所有API都是 wait-free的。正在执行中的API调用不会影响其他API的完成。
*  data tree的API都是对文件系统的wait-free操作，不直接提供锁这样的分布式协同机制。但是data tree的API非常强大，可以用来实现多种分布式协同机制。

### znode分类

一个znode可以是持久性的，也可以是临时性的：

1. 持久性的znode (PERSISTENT):这样的znode在创建之后即使发生ZooKeeper集群宕机或者client宕
   机也不会丢失。

2. 临时性的znode (EPHEMERAL): client 宕机或者client 在指定的 timeout时间内没有给ZooKeeper集
   群发消息，这样的znode就会消失。

   >  znode也可以是顺序性的。每一个顺序性的node关联一个唯一的单调递增整数。这个单调递增整数是
   > znode名字的后缀。如果上面两种znode具备顺序性，又有以下两种znode :

3. 持久顺序性的znode(PERSISTENT_SEQUENTIAL): znode除了具备持久性znode的特点之外，znode的名字具备顺序性。

4. 临时顺序性的znode(EPHEMERA_SEQUENTIAL): znode除了具备临时性znode的特点之外，znode的名字具备顺序性。

Zookeeper主要有以上4中znode。



### ZooKeeper入门

修改配置文件zookeeper/conf/zoo_sample.cfg  

dataDir=/data/zookeeper

* 使用zkServer.sh start启动Zookeeper服务。
* 检查ZooKeeper日志是否有出错信息。
* 检查Zookeeper数据文件。
* 检查Zookeeper是否在2181端口上监听。



使用zkCli创建节点

> create /app1
>
> create /app2
>
> create /app1/p_1
>
> create /app1/p_2

查看所有节点

> ls -R /

实现分布式锁

> create -e  /lock   创建临时节点
>
> stat -w /lock 等待锁释放

