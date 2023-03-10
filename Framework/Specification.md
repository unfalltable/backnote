---
title: 代码规范
toc: content
keywords: [specification]
---

## 概念

### 数据模型

| 值   | 场景                                         |
| ---- | -------------------------------------------- |
| DTO  | 接口入参，即前端传给后端的Json接收对象       |
| VO   | 接口返回对象，即后端传给前端展示数据用的对象 |
| PO   | 数据库表对应的对象，和Entity等同             |
| BO   | 业务对象，即一组PO                           |



## LomboK

- 避免使用@Data注解，使用@Getter、@Setter、@ToString替代
- 尽量自己实现 hashCode 和 equals 方法
- 在使用 Lombok 的情况下
  - boolean类型的变量不能以 is 开头
  - 字段名前两个字母最好为小写，非要有大写的话使用 @TableField 注解指定字段，或者专门为其写 get 和 set 方法

## 返回值处理

- 如果返回的集合没有数据的话就返回空集合，不要返回null

## Spring

- 使用构造注入替代@Autowired注入Bean对象
  - 好处
    - 能够在项目启动时检测出出现循环依赖的模块
    - 可以使用final关键字来修饰依赖字段使其不可变

## 数据库规范

- 如果创建的表在后续可能会新增字段，就在创建表时添加一些扩展字段
- 尽量不要创建外建约束
- 可以通过第三张表来表示两张表的关联关系

## SQL规范

- 使用sum函数是如果可能出现为0的情况，需要使用 IFNULL(SUM(), 0) 约束
- 用String类型接收返回值时要注意null的问题，有可能返回 null 字符串的
- 尽量不要在已经线上正常运行的sql上加东西，最好再写一个sql

## 业务规范

### 当接口出现bug时

- 应该先查询有多少个地方使用了该接口
  - 如果只有一处使用则可以进行修改
  - 如果多出使用必须仔细排查和列出修改后可能出现的情况
    - xml可能存在复用的情况

### 项目上线后，当发现代码有错误或者冗余时，尽量不要修改



