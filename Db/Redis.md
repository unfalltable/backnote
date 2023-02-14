---
title: Redis
keywords: [redis, db]
---



##  数据类型

### String

| 功能                                    | 命令                                                |
| --------------------------------------- | --------------------------------------------------- |
| 添加/修改数据                           | set key value<br />mset key1 value1 key2 value2 ... |
| 获取数据                                | get key<br />mget key1 key2...                      |
| 删除数据                                | del key                                             |
| 获取字符串长度                          | strlen key                                          |
| 追加消息                                | append key value                                    |
| 增加                                    | incr key<br />inceby / incrbyfloat   key 数值       |
| 减少                                    | decr key<br />decrby key 数值                       |
| 设置有效期                              | setex key seconds 值<br />psetex key millseconds 值 |
| 设置一个值，当redis中没有时才能设置成功 | setnx key 值                                        |

- 数值超过上限会报错
- 0 表示失败，1表示成功
- nil 表示为null
- 数据最大存512MB

### Hash

| 功能                                        | 命令                                                |
| ------------------------------------------- | --------------------------------------------------- |
| 添加 / 修改数据                             | hset key field value                                |
| 获取数据                                    | hget key field<br />hgetall key                     |
| 删除数据                                    | hdel key field                                      |
| 添加 / 修改多个数据                         | hmset key field1 value1...                          |
| 获取多个数据                                | hmget key field1 field2...                          |
| 获取key中字段的数量                         | hlen key                                            |
| 获取key中是否存在指定的字段                 | hexists key field                                   |
| 获取key中所有的字段名或字段值               | hkeys key<br />hvals key                            |
| 设置指定字段的数值数据增加指定范围的值      | hincrby key field 值<br />hincrbyfloat key field 值 |
| 获取一个key中所有的field-value              | hgetall key                                         |
| 添加一个key中的field，前提是这个field不存在 | hsetnx key field 值                                 |

- 一个key对应多个field - value
- 如果filed少，会优化为类数组
- 如果field多，会优化为HashMap
- value只能存字符串
- 一个key可以存2^32^ - 1 个键值对
- hash可以存储少量对象

### List

| 功能                             | 命令                 |
| -------------------------------- | -------------------- |
| 左侧插入元素                     | LPUSH key 值         |
| 移除左侧第一个元素               | LPOP key             |
| 右侧插入元素                     | RPUSH key 值         |
| 移除右侧第一个元素               | RPOP key             |
| 返回一段范围内的所有元素         | LRANGE key start end |
| 阻塞取元素，没元素会等待一段时间 | BLPOP / BRPOP key    |

- 可以看成双向链表

### Set

| 操作               | 命令             |
| ------------------ | ---------------- |
| 添加元素           | Sadd key 值      |
| 移除元素           | Sred key 值      |
| 返回元素个数       | Scard key        |
| 判断元素是否存在   | Sismember key 值 |
| 获取所有元素       | Smembers         |
| 求key1和key2的交集 | Sinter key1 key2 |
| 求key1和key2的差集 | Sdiff key1 key2  |
| 求key1和key2的并集 | Sunion key1 key2 |

- 类似hashSet

### SortedSet

| 操作                       | 命令                      |
| -------------------------- | ------------------------- |
| 添加                       | Zadd key score 成员       |
| 删除                       | Zrem key 成员             |
| 获取分值                   | Zscore key 成员           |
| 获取排名                   | Zrank key 成员            |
| 获取元素个数               | Zcard key                 |
| 统计范围内个数             | Zcount key min max        |
| 指定元素自增               | Zincrby key 自增的值 成员 |
| 排序后，取指定排名范围的值 | Zrange key min max        |
| 排序后，取指定分数范围的值 | ZrangeByScore key min max |
| 差集 / 交集/ 并集          | Zdiff / Zinter / Zuinon   |

- 排序默认是升序的，Zrev为降序
- 底层数据结构是跳表

### Stream消息队列

#### 消费者

- 添加信息
  - `Xadd key [nomkstream] [maxlen | minId [=|~] threshold [limit count]] *|ID field value...` 
    - [nomkstream]：队列不存在则创建
    - [maxlen | minId [=|~] threshold [limit count]]：设置队列的最大消息数量
    - *|ID：消息唯一id，\*代表自动生成
    - field value：要发送的消息
- 读取消息
  - `Xread [Count count] [block milliseconds] streams key... Id...`
    - [Count count]：每次读取的数量
    - [block milliseconds]：没有消息时的阻塞时长
    - streams key：队列名
    - Id：起始id，0代表从第一个消息开始，$代表从最新的消息开始

#### 消费者组

- 创建消费者组
  - `Xgroup create key groupName Id [mkstream]`
    - key：队列名
    - groupName：消费者组名
    - Id：$代表队列中最后一个消息，0代表第一个消息
    - mkstream：队列不存在则创建
- 读取消息
  - `XreadGroup Group group consumer [Count count] [block milliseconds] [NoAck] streams key... Id...`
    - group：消费者组名
    - consumer ：消费者名
    - [Count count]：每次读取的数量
    - block milliseconds：没有消息时的阻塞时长
    - NoAck：无需手动Ack，自动确认
    - key：队列名
    - Id：获取信息的起始ID
      - “ >”：从下一个未消费的信息开始
      - 下标：从pending-list下标位置开始读取
- 查看pending-list
  - `  Xpending key group id范围 count`

### GEO

| 作用                                                        | 命令                |
| ----------------------------------------------------------- | ------------------- |
| 添加地理空间信息                                            | GEOadd 经度 纬度 值 |
| 计算两点之间距离                                            | GEOdist 点1 点2     |
| 指定点的坐标转换为hash字符串                                | GEOhash             |
| 返回点的坐标                                                | GEOpos              |
| 指定圆心，半径，找到园内点，按照离圆心的距离排序（6.2废弃） | GEOradius 圆心 半径 |
| 指定范围内找点，可以是圆型、矩形，会排序（6.2新增）         | GEOsearch           |
| 与GEOsearch功能一样，但可以把结果存入一个key                | GEOsearchStore      |

- 底层是SortedSet

### BitMap

| 作用                    | 命令        |
| ----------------------- | ----------- |
| 指定位置插入0或1        | setBIT      |
| 获取指定位置值          | getBIT      |
| 统计1的数量             | BITcount    |
| 查、改、增 指定位置的值 | BITfield    |
| 获取数组                | BITfield_ro |
| 结果做位运算            | BITop       |
| 找第一个1 或0 的位置    | BITpos      |

### HyperLogLog

- 用于确定非常大的集合的基数，不需要存储其所有值
- 基于String实现的，当个HLL内存小于16kb
- 测量结果有小于0.81%的误差

## 客户端

### SpringDataRedis序列化的方式

- ​	自定义redisTemplate，修改序列化器

  - ```java
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        //创建Template
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        //设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //设置序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        //key和HashKey采用String序列化
        redisTemplate.setKeySerializer(ReidsSerializer.string());
        redisTemplate.setHashKeySerializer(ReidsSerializer.string());
        //value和HashValue采用Json序列化
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
    ```

- 使用StringRedisTemplate，手动序列化Json

## 雪崩、穿透、击穿

### 缓存穿透

- 缓存穿透是指客户端请求的数据在缓存和数据库中的不存在，这些请求都会打到数据库

#### 解决方法

- 使用缓存空对象解决，可能存在短期不一致性
- 使用布隆过滤器解决，在客户和缓存之间添加
- 增强id的复杂度
- 加强用户权限校验
- 做好热点参数的限流

### 缓存雪崩

- 同一时段大量的缓存key同时失效或者redis宕机，导致大量的请求直接到达数据库，带来巨大压力

- 解决方案：

  - 给不同的缓存key的过期时间（TTL）添加随机值

  - 使用Redis集群
  - 给缓存业务添加降级限流策略
  - 给业务添加多级缓存

### 缓存击穿

- 一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求会打到数据库

#### 解决方法

- 互斥锁
  - 没有额外内存消耗，保证一致性，实现简单，但是性能受影响，可能出现死锁
  - 利用Redis的 SETNX 实现
    - 只有第一个执行SETNX的成功	
- 逻辑过期
  - 性能较好，但不保证一致性，有额外内存消耗，实现复杂

## 消息队列

|              |        List        |       Pubsub       |                   Stream                   |
| ------------ | :----------------: | :----------------: | :----------------------------------------: |
| 持久化       |         √          |         ×          |                     √                      |
| 阻塞读取     |         √          |         √          |                     √                      |
| 消息堆积处理 | 可利用多消费者处理 | 受限于消费者缓冲区 | 受限于队列长度，可使用消费者组加快消费速度 |
| 消息确认     |         ×          |         ×          |                     √                      |
| 消息回溯     |         ×          |         ×          |                     √                      |

## Lua脚本

## 内存淘汰策略

- noeviction：不清除缓存（默认）
- allkeys-lru：对所有key使用LRU算法进行删除
- volat-lru：对所有设置了过期时间的key使用LRU删除
- allkeys-random：对所有key随机删除
- volatile-random：对所有设置了过期时间的key随即删除
- volatile-ttl：删除马上就要过期的key
- allkeys-lfu：对所有key使用LRU算法进行删除
- volatile-lfu：对所有设置了过期时间的key使用LFU删除

## 应用场景

- 短信验证
- 数据缓存
  - 先更新数据库在更新缓存
  - 分布式事务保证原子性
- 全局ID
  - incrby方法，是原子性操作
  - 符号位(1) + 时间戳(31) + 序列号(32)
  - 雪花算法
    - 符号位 + 时间戳 + 机器码 + 序列号
- 分布式锁
  - setnx方法，超时释放，分布式id冲突
  - Redission实现
- 消息队列
  - XGroup，消息分流、表示、确认、阻塞读取、可回溯
  - 使用Lua脚本添加信息，保证原子性
  - 消息发送后进入pending-list，XACK后移除
- 点赞
  - SortedSet
- 关注
  - Set，查看共同关注
- 推送
  - 有推，拉，推拉结合
  - 为用户建立收件箱，实现分页查询，SortedSet实现
- 附近商铺
  - GeoLocation，底层是SortedSet
- 签到
  - BitMap，String实现

## 持久化

### RDB

- Redis数据备份文件，也被叫做Redis数据快照
- RDB把内存中的所有数据记录到磁盘中，当Redis故障重启后，从磁盘读取RDB
- Redis停机时会保存一次RDB，也可以设置单位时间修改量阈值，超过阈值就保存
- redis.conf文件中可以对RDB进行配置
- RDB执行间隔长，俩次RDB之间写入数据有丢失的风险
- 获得子线程，往磁盘写RDB都比较耗时

#### 存储的流程

- 得到一个子线程，共享内存空间
- 读取内存数据写入新的RDB文件
- 用新RDB文件替换旧的

### AOF

- Redis追加文件，redis的每一个写命令都会记录在aof中
- 默认是关闭的，需要在配置文件中开启，也可以配置相关配置
  - Always：同步刷盘，可靠性高，不丢数据，性能影响大
  - everysec：每秒刷盘，性能适中，最多丢失1s数据
  - no：操作系统控制，性能最好，可靠性较差，可能丢失大量数据
- AOF文件比RDB文件大的多
- aof会记录对一个key的多次写操作，但其实只有最后一次写有意义，可以通过**bgrewriteaof**命令，让aof执行重写，用最少的命令达到同样的效果
- 也可以设置阈值触发重写aof

## Redission

### 可重入原理

- 使用hash结构来记录获取锁的线程，value记录的是重入的次数
- 内部还是实现的lua脚本

### 可重试原理

利用信号量和PubSub功能实现

- 使用带参的构造函数获取锁，参数是重试的等待时间和时间单位
- 底层会给一个默认的超时时间30秒（**看门狗时间**）
- 当获取锁失败时会返回一个锁的剩余有效期 (**pttl**) (剩余的超时时间)
- 用等待时间减去获取锁失败这个过程消耗的时间
  - 小于等于0，返回false
  - 大于0，进入等待通知状态（**订阅状态**），当有线程释放锁时会通知其他等待的线程
- 当等待通知的时间超过了等待时间时，取消订阅状态，返回false
- 当在等待的期间获得了通知
  - 用等待时间减去等待通知这个过程消耗的时间
    - 小于等于0，返回false
    - 大于0，进入一个循环，尝试获取锁，成功返回
      - 失败进入一个等待**信号量**的状态，ttl时间到或者等待时间到时尝试获取锁
        - 判断等待时间，超时false，不超时进入下一轮循环
- 代码中经常涉及到对剩余时间的计算，比较严谨

### 不超时原理

- 使用带参的构造函数获取锁，参数是重试的等待时间、超时时间和时间单位 
- 不设置超时时间才能实现锁不超时（**看门狗机制**）

- 当获取锁成功时会调用**scheduleExpirationRenewal()**方法
  - 方法中会创建一个**ExpirationEntry**对象，用一个**ConcurrentHashMAp**存储，key是当前锁的名称，value是**ExpirationEntry**对象，一个锁对应一个entry，重入不覆盖
- 执行**renewExpiration**方法刷新超时时间
  - 方法中会创建一个**Timeout**定时线程，只要当前线程持有锁，每隔**看门狗时间**/3 秒就会调用**renewExpirationAsync**异步刷新一次超时时间
- 会把线程id作为key，定时刷新的线程作为value存入一个map中
- 当锁释放时，执行**cancelExpirationRenewal**
  - 根据锁id取出map中的定时刷新线程然后取消掉
  - 然后把entry删除
- 避免阻塞导致锁超时释放

### 主从一致（**连锁**）

- Redission不分主从关系，会向所有redis节点加锁，这些节点可以有从节点
- 通过**redissionClient.getMultiLock(锁对象1,锁对象2...)**创建连锁
  - 内部会把这些锁放在一个集合中
    - 所有操作都需要集合中每一个锁都成功才能成功

- 在获取每一个锁时如果超时会释放所有以获得的锁
- 如果设置了超时时间，在获取所有锁之后会一次性给所有锁刷新超时时间
- 没设置超时时间会使用**看门狗机制**

## 分布式缓存

### 主从集群

### 分片集群

## 多级缓存

### JVM进程缓存

#### Caffeine

### Nginx本地缓存

#### Lua

### OpenResty

## 最佳实践

### 批处理

