---
title: MongoDB
toc: content
keywords: [mongodb, db] 
---

## 简介

- 结构松散，使用bson结构
- 是一个分布式文件存储的数据库、C++编写
- 最像关系型数据库的非关系型数据库，介于非关系和关系型之间
- 高并发读写、海量数据高效率存储和访问、高扩展性和高可用性

- 应用场景（海量数据，写入频繁，价值低，事务要求不高）

  - 社交：朋友圈信息，附近的人，地点等
  - 游戏：装备、积分
  - 物流：实时订单信息，
  - 物联网：智能设备信息，日志信息
  - 视频直播：点赞互动等

- | MySQL       | MongoDB            |
  | ----------- | ------------------ |
  | database    | database           |
  | table       | collection    集合 |
  | row         | document   文档    |
  | colunm      | field              |
  | index       | index              |
  | table joins |                    |
  | primary key | primary key        |

## 操作

- 查看所有的数据库
  - `show dbs`
- 使用指定数据库
  - `use 数据库名`
- 创建数据库
  - `use 数据库名`
  - 数据库自动创建，进行插入数据即可自动创建

- 删除数据库
  - `db.dropDatabase()`
- 删除表（集合）
  - `db.表名.drop()`
- 插入数据
  - `db.表名.insert({id:1, username:'张三'})`
  - `db.表名.save({id:1, username:'张三'})`

- 更新
  - `db.表名.update(查询条件，变更的内容，[upesrt不存在数据是否插入], [multi 是否更新多条数据]，[writeConcern抛出异常的级别])`
  - `db.表名.update({id:1}, {age:25})` 这样写数据会被删除只剩age
  - `db.表名.update({id:1}, {$set:{sex:1}})` 更新不存在的字段，会新增字段
  - 更新不存在的数据默认不会新增数据

- 删除

  - `db.表名.remve([查询条件], [justOne 默认为true，删除一个，false删除所有匹配的], [writeConcern抛出异常的级别])`

  - 官方推荐使用`db.表名.deleteOne()` 和 `db.表名.deleteMany()`

- 查询
  - `db.表名.find([查询条件]，[fields 指定返回的字段])`
  - 后接pretty()美化返回结果
  - 条件
    - 等于（:）、小于（\$lt:）、小于等于（\$lte:）、大于（\$gt:）、大于等于（\$gte:）、不等于（\$ne:）
  - 分页
    - limit()
    - skip()
    - sort()

## 索引

- 查看索引
  - `getIndexes()`

- 创建索引
  - `createIndex()`
- 删除索引
  - `dropIndex()`
  - `dropIndexex()`      删除除了_id之外的索引
- 创建联合索引
  - `createIndex(多个fields)`
- 查看索引大小
  - `totalIndexSize()`

## 执行计划

- 语句后接 explain()

## JavaAPI

### 配置

```java
public static void main(Stringp[] args){
    //建立连接
    MongoClient mongoClient = MongoClients.create("mongodb://127.0.0.1:27017");
    //选择数据库
    MongoDatabase database = mongoClient.getDatabase("数据库名");
    //选择表 
    MongoCollection<Document> collection =  database.getCollection("表名");
    //操作
    collection.find().limit(10).forEach((Consumer<? super Document>) document ->{
        //输出
    });
    //关闭连接，释放资源
    mongoClient.close();
}
```

### 查询

```java
public void Query(){
    //查询age <= 50 且 id >= 100 的用户信息，id倒序输出，返回id，age
    this.collection.find(
    	Filters.and(
        	Filters.lte("age", 50),
            Filters.gte("id", 100)
        )
    ).sort(//排序
    	Sorts.descending("id")
    ).projection(//查询的字段
    	projections.fields(
        	projections.include("id","age"),
            projections.excludeId()
        )
    ).forEach((Consumer<? super Document>) document ->{
        //输出
    });
}
```

### 插入

```java
public void Insert(){
    Document document = new Document();
    document.append("id", 1001);
    document.append("name", "张三");
    document.append("age", 100);
    this.colection.insertOne(document);
}
```

### 修改

```java
public void Update(){
    						   //查询条件         //更新的字段
	this.collection.updateOne(eq("id", 1001), Updates.set("age", 30));
}
```

### 删除

```java
public void Delete(){
    						    //查询条件  
	this.collection.deleteMany(eq("id", 1001));
}
```
