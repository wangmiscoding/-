### 消息确认机制

分为三个角色

1. 生产者

   确保生产者投递到MQ服务器成功

   方式1：	Ack消息确认机制  同步或异步的形式

   ```java
   
    //启用生产者确认
               channel.confirmSelect();
   
   
               channel.basicPublish(exchange, "com", null, message.getBytes());
   
               //消息是否投递成功,同步阻塞等待结果
               boolean sendResult = channel.waitForConfirms();
   ```

   方式2： 事务消息

   ```java
       //开启事务消息
               channel.txSelect();
               
               channel.basicPublish(exchange, "com", null, message.getBytes());
               
               System.out.println("消息发送成功");
   
               //提交事务消息
               channel.txCommit();
           } catch (IOException  | TimeoutException e) {
               //事务消息回滚
               if(channel!=null) {
                   channel.txRollback();
               }
               e.printStackTrace();
           } 
   ```

   

2. 消费者

   必须要将消息消费成功之后，才会将该消息从mq服务器端中移除。

   在kafka中的情况下：

   ​	不管是消费成功还是消费失败，该消息都不会立即从mq服务器端移除。

```java
 Channel finalChannel = channel;
                channel.basicConsume(queueName, false, new DefaultConsumer(finalChannel){  // true自动签收   设置false 手动应答  手动应答不会自动将队列中的消息删除
                    @Override
                    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                        System.out.println(queueName + "收到消息是" + new String(body, StandardCharsets.UTF_8));
                        //消费完成后  消费者通知给mq服务器删除该消息
                        finalChannel.basicAck(envelope.getDeliveryTag(),false);

                    }
                } );
```



1. Mq服务器端 ：在默认情况下  都会对队列中的消息实现持久化，持久化到硬盘。



###  生产者如何获取消息消费状况

​	**根据业务决定**

每隔2s时间根据全局Id查询数据库中，是否存在(存在说明已被消费)

### 死信队列原理

死信队列和普通队列区别不是很大

普通和死信队列都有自己独立的交换机和路由key、队列和消费者。

区别：

1. 生产者投递消息先投递到我们普通交换机中，普通交换机再讲消息投递到普通队列
2. 如果生产者投递消息到普通队列中，普通队列发现该消息一直没有被消费者消费的情况下，这时会将消息转移到死信交换机中。

### 消息幂等问题

#### 消息自动重试机制

