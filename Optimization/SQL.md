---
title: SQL性能优化
toc: content
keywords: [optimization, sql]
---

## SQL性能分析

- 慢查询日志

  - 默认没有开启，需要在 `/etc/my.cnf` 中配置以下内容

    - `show_query_log = 1` 开启慢查询日志

    - `long_query_time = 2` SQL查询超过2秒的为慢查询

  - 重启mysql，在`/var/lib/mysql/localhost-slow.log` 中查看日志
- SQL执行频率

  - `show session / global status like 'Com_'` 
- 查看SQL耗时情况
  - 每一条SQL耗时：`show profiles` 
  - 指定SQL各阶段耗时：`show profile for query {id}` 
  - 指定SQL的CPU使用情况：`show profile cpu for query {id}` 

- 查看SQL执行计划
  - `explain sql语句` 
    - id：sql序列号，id值越大越先执行
    - select_type：查询的类型
      - simple：简单单表查询
      - primary：主查询，即外层的查询
      - union：联合查询后的查询
      - subquery：select/where后的子查询

    - type：连接类型（null、system、const、eq_ref、ref、range、index、all）
      - 如果使用的是主键索引或唯一索引一般是const

    - possible_key：可能使用到的索引
    - key：实际使用到的索引
    - key_len：索引中使用的字节数（最大可能长度），越短越好
    - rows：必须要执行的查询的行数，innodb引擎中是一个估值
    - filtered：返回结果行数占需读取行数的百分比，越大越好


## SQL优化

- 如果存储介质是机械硬盘，可以设置MRR开启顺序存储，这个功能会将插入的值按id排好序存到buffer中，顺序写入磁盘，提高插入效率，因为机械硬盘需要一个磁盘寻址的过程会影响效率
- 插入大批量数据时使用 load 指令
- 手动提交事务
- 插入时按主键顺序插入
- MySQL5.6 之后支持索引下推，能减少回表
- 分页查询时使用 覆盖索引 + 子查询（想查询的id）
- 排序优化
  - 在创建索引是可以指定索引排序规则，符合规则的sql语句效率高
  - 需要注意各字段的升降序，尽量一致，尽量覆盖索引
  - 可以适当的增大排序缓冲区的大小
    - `sort_buffer_size` 默认是256k
- Join 优化
  - 小表 join 大表
  - 连接字段需要是索引字段
  - 左右连接才有优化，内连接的话由mysql自行判断顺序
  - 增大 join buffer 的大小
  - 减少不必要的查询字段，可以缓存更多的数据
  - 大表 join 大表的话可以为大表建立分区
  - 算法
    - NLJ算法：双重for，连接字段为非索引就是用这个算法
    - BNLJ算法：把 join 的驱动表放到了内存 buffer 中，减少了循环次数
    - INLJ算法：连接字段为索引字段用这个算法，内层表的连接索引字段进行匹配，减少内层表的循环次数
