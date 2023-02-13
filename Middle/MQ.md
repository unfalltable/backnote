---
title: MQ消息队列
keywords: [middle]
---



# 协议

- MOM（Message Oriented Middleware 面向消息中间件）
  - PO  面向过程
  - OO 面向对象
  - AO  面向切面

## JMS（Java Message Service）

- Java消息服务协议，是一种MOM的设计实现
- 标准的生产、发送、接收消息
- ActiveMQ是该协议的典型实现

## STOMP（Streaming Text Orientated Message Protocol）

- 面向文本流的消息协议，是一种MOM的设计实现
- 可互操作的连接格式，允许客户端与任意的STOMP消息代理进行交互
- ActiveMQ是该协议的典型实现，RabbitMQ可通过插件实现

## AMQP（Advanced Message Queuing Protocol）

- 高级消息队列协议，是一种MOM的设计实现
- 是一个应用层协议，是一种标准，不受客户端/中间件/语言的限制
- RabbitMQ是该协议的典型实现

## MQTT（Message Queuing Telemetry Transport）

- 消息队列遥测传输，即时通信协议，是一种二进制协议，也是一种MOM的设计实现
- 用于服务器和低功耗LOT设备间的通信，支持所有平台，可以把所有联网物品和外部连接起来
- RabbitMQ可通过插件实现

# RocketMQ

## 基础

- Producer
  - 消息生产者
  - Producer Group
    - 一类Producer集合，发送一类消息，发送逻辑一致
- Consumer
  - 消息消费者，后台异步消费
  - Push Consumer
    - 服务端向消费者端推送消息
  - Pull Consumer
    - 消费者端向服务端定时拉取消息
  - Consumer Group
    - 一类Consumer集合，消费一类消息，消费逻辑一致
- NameServer
  - 组织协调者
  - 收集Broker的工作情况
- Broker
  - RocketMQ的核心，负责消息的发送，接收，高可用等（是真正干活的）
  - 需要定时发送自身情况到NameServer，10秒发送一次，超过2分钟没发，NameServer会认为该Broker失效
- Topic
  - 在Broker中
  - 不同类型的消息以不同的Topic进行区分
  - MessageQueue
    - 消息队列，用于存储消息

## 创建Topic

```java
public static void CreateTopic(){
    //设置nameserver的地址
    producer.setNameservAddr("ip地址:端口号");
    //启动生产者
    producer.start();
    //创建topic
    producer.createTopic("broker名称", "topic名称", 队列数量);
}
```

## 生产者

- 生产者需要在构造时创建或加入分组
- 信息需要设置topic、broke名，可以添加tags，队列数量
- 发送消息
  - 同步：send(message)
    - message需要设置topic、tags、消息内容（字节数组）
  - 异步：send(message, new SendCallback(){  //重写两个方法 })

## 消费者

- 订阅topic，设置接收的信息类型（可以根据tags选择）
- 创建监听器（并发 / 顺序）
  - 消息过滤器(需要开启)
    - 消费者接收消息是可以进行筛选
      - MessageSelector.bySql(“”)
    - 生产者发送信息时可为信息设置一些属性
      - message.putUserProperty(name, value)

## 顺序信息

- 生产者需要将信息存入同一个topic中同一个队列
- 消费者需要按顺序从同一条队列中取消息
  - 监听器选择MessageListenerOrderly

## 分布式事务消息

![image-20220725004549050](RocketMQ/image-20220725004549050.png)

- 生产者
  - 使用TransactionMQProducer，设置NamesrvAddr，设置事务监听器（TransactionListener），需要传入事务监听器实现类
    - 实现类需要实现executeLocalTransaction（业务实现）和checkLocalTransaction（回查）
  - 事务回查
    - 在executeLocalTransaction中需要设置各个阶段事务的状态，存储在一个Map集合中，事务id为key
    - 在checkLocalTransaction中，从Map中根据事务id从Map中获取事务状态返回

### 使用Spring

- 实现`RocketMQLocalTransactionListener` 接口

- 添加`@RocketMQTransactionListener(txProducerGroup = "生产者组名")`

- 保存事务的状态

  - ```java
    public class XXX implements RocketMQLocalTransactionListener{
        private static Map<String, RocketMQLocalTransactionState> STATE_MAP = new HashMap<>();
        
        @Override
        public RocketMQLocalTransactionState executeLocalTransaction(Message message, Object o){
            
        }
    }
    ```

- ![image-20220726225310037](RocketMQ/image-20220726225310037.png)

- 生产者![image-20220726225617808](RocketMQ/image-20220726225617808.png)

- 消费者![image-20220726225741449](RocketMQ/image-20220726225741449.png)

### 应用场景

- 订单超时取消

## 消息推送

### 消费者推送模式

- push模式
  - 服务端有消息就推送给客户端，需要建立长连接，客户端多时消耗资源
- pull模式
  - 客户端轮询服务端，有消息就拉取，存在实时性问题
- 长轮询
  - RocketMQ使用长轮询解决了push和pull的问题，即长连接+轮询

### 消息模式

- 集群（默认）
  - 某个消费者可以接收
- 广播
  - 每个消费者都可以接收
- 通过 `setMessageModel` 设置

### 重复消息

- 消费端处理业务保证幂等性
- 每条消息都有唯一的编号，判断消息是否处理过即可
- RocketMQ不保证消息不重复，需要在业务端处理

## 数据存储

- 零拷贝（mmap+write） 
- 文件系统采用Linux Ext4
- 往磁盘写入是顺序写入的，由ConsumeQueue和CommitLog配合
  - CommitLog 真正存储数据的文件
  - ConsumeQueue 存储消息在CommitLog 中的位置信息

##  刷盘

- 同步刷盘
  - 写入磁盘后再返回成功
- 异步刷盘
  - 写入内存后就返回成功，内存积累一定信息后批量刷盘

## 重试策略

### 生产者

- 设置重试次数
  - `setRetryATimesWhenSendFaild(次数)`
- 在发送消息时指定超时时间
  - `send(message, 超时时间)`
- 并不是所有的异常都会触发重试

### 消费者

- 异常失败
  - 每次重试的时间间隔不同，不能修改
  - 重新获取的消息id是不同的，但是内容是相同的
- 超时失败
  - RQ服务器会不断重试发送，直到发送成功

## 集群	

### 集群模式

- 单master
  - 风险最大
- 多master
  - 全是master
  - 一个机器宕机，未消费的信息不可订阅，实时性受到影响
- 多master多slave
  - 异步复制
    - master和slave之间有短暂消息延迟，毫秒级，速度快，磁盘损坏丢失少量信息
  - 同步双写
    - master和slave同时写，速度慢，消息无延迟，高可用

### 集群搭建

- ![image-20220726152658071](RocketMQ/image-20220726152658071.png)
- ![image-20220726152811139](RocketMQ/image-20220726152811139.png)![image-20220726152822328](RocketMQ/image-20220726152822328.png)
- ![image-20220726153353385](RocketMQ/image-20220726153353385.png)
- ![image-20220726153428809](RocketMQ/image-20220726153428809.png)![image-20220726153436277](RocketMQ/image-20220726153436277.png)
- ![image-20220726153450253](RocketMQ/image-20220726153450253.png)

### 两阶段提交

- 消息发送到master后，将消息设置为uncommitted，当大部分slave节点同步完成后，设置为committed

## SpringBoot整合RokcetMQ

![image-20220726154035749](RocketMQ/image-20220726154035749.png)

![image-20220726154120005](RocketMQ/image-20220726154120005.png)

### 生产者

![image-20220726154438110](RocketMQ/image-20220726154438110.png)

使用同步发送或者异步发送才能获取发送消息后的结果信息

### 消费者

![image-20220726154819917](RocketMQ/image-20220726154819917.png)

# RabbitMQ

## 手动事务消息

- 提供了一套api实现
- 会导致Channel阻塞，造成吞吐量下降

## 生产者确认机制（Publisher Confirm）

- 新版本扩展
- 类似RocketMQ的事务消息基本一致

# Kafka
