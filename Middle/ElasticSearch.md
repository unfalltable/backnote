---
title: ElasticSearch
toc: content
keywords: [middle, es]
---

## 定义

Index        -->   数据库

Type         -->   表

Document -->   文档

Field          -->   字段

## 基本使用

- 创建索引

  - ```json
    PUT url:端口号/索引名
    {
        "settings": {
            "index": {
             	"number_of_shards": "2", #分片数
                "number_of_replicas" : "0" #副本数
            }
        }
    }
    ```

- 插入数据

  - ```json
    POST url:端口号/索引名/类型/id
    {
        "settings": {
            "index": {
             	"number_of_shards": "2", #分片数
                "number_of_replicas" : "0" #副本数
            }
        }
    }
    ```

- 修改数据

  - ```json
    PUT url:端口号/索引名/类型/id
    {
        "":  
    }
    ```

    - 通过覆盖实现的更新

    - 每次覆盖更新版本会加一

    - 可以实现局部更新

      - ```json
        PUT url:端口号/索引名/类型/id/_update
        {
            "doc":{
                "": 
            }  
        }
        ```

- 删除数据

  - ```json
    DELETE url:端口号/索引名/类型/id
    ```

- 查询数据

  - ```json
    GET url:端口号/索引名/类型/id
    ```

  - ```json
    GET url:端口号/索引名/类型/_search
    ```

    - 搜索全部数据，默认返回10条

  - ```json
    GET url:端口号/索引名/类型/_search?q=age:20
    ```

    - 添加关键字

## DSL搜索

查询数据时可以携带Json体，请求类型为POST

- 全文搜索

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
            "match": {
                "name": "张三 里斯"
            }
        }
    }
    ```

- 高亮显示

  - ```json
    "highlight": {
        "fields": {
            "name": {}
        }
    }
    ```

- 聚合搜索

  - 桶聚合

    - 类似sql中的分组

    - ```json
      POST url:端口号/索引名/类型/_search
      {
          "aggs": {
              "all_interests": {
                   "terms": {
                      "fields": "age"
          		}
              }
          }
      }
      ```

    - 可以指定size

  - 度量聚合

    - min、avg、max

- 分页搜索

  - 类似sql中的分页
  - size: 结果数
  - from: 跳过数

- 文档

- 美化

  - 在查询语句后添加`?pretty` 可以美化返回的json

- 指定响应字段

  - GET请求，指定返回字段，会返回元数据

    - ```json
      GET url:端口号/索引名/类型/id/?_source=返回的字段
      ```

  - 返回数据，不返回元数据

    - ```json
      GET url:端口号/索引名/类型/id/_source //返回全部数据
      ```

    - ```json
      GET url:端口号/索引名/类型/id/_source?_source=返回的字段
      ```

- 判断文档是否存在

  - ```json
    HEAD url:端口号/索引名/类型/id
    ```

## 批量操作

- 批量查询

  - ```json
    POST url:端口号/索引名/类型/_mget
    {
        "ids": ["",""]
    }
    ```

- 批量增删改

  - ```json
    POST url:端口号/_bulk
    {
        //插入
        {"create": {"_index":"索引名", "_type":"类型","_id":id}}\n
    	{"id":1,"name":"name1","age":18,"sex":"男"}\n
    	//删除
    	{"delete":{"_index":"索引名", "_type":"类型","_id":id}}
    }
    ```

- 映射

  - String 新版不支持了，需要有全文搜索用text，不需要就用keyword

  - 查看映射

    - ```json
      GET url:端口号/索引名/_mapping
      ```

## 结构化查询

- term

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
         	"term": {
                "age": 20
            }   
        }
    }
    ```

    - 用于精确匹配

- terms

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
         	"terms": {
                "tag": ["","",""]
            }   
        }
    }
    ```

    - 类似sql中的in

- range

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
         	"range": {
                "age": {
                    "gte": 20,
                    "lt": 30
                }
            }   
        }
    }
    ```

  - gt、gte、lt、lte

- exists

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
         	"exists": {
                "field": "title"
            }   
        }
    }
    ```

- match

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
         	"match": {
                "tweet": "About Search"
            }   
        }
    }
    ```

- bool

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
         	"bool": {
                "must": {"term": {}}      //相当于and
                "must_not": {"term": {}}  //相当于not
                "should": {"term": {}}    //相当于or
            }   
        }
    }
    ```

- 过滤查询

  - ```json
    POST url:端口号/索引名/类型/_search
    {
        "query": {
         	"bool": {
                "filter": {
                	"term": {
                        "age": 20
                    }
                }
            }   
        }
    }
    ```

    - 查询和过滤的对比
      - 过滤会询问每个文档的字段值是否包含特定值
      - 查询户询问每个文档的字段值与特定值的匹配程度
      - 过滤会缓存结果，高效
      - 查询语句比过滤语句耗时

## 分词

- 内置分词

  - ```json
    POST url:端口号/_analyze
    {
        "analyzer": "分词器",
        "field": "字段",
        "text": ""
    }
    ```

    - 内置分词器有
      - Standard，按单词划分，会转化为小写
      - Simple，按照非单词划分，转小写
      - Whitespace，按空格切分
      - Stop，去除语气词后划分
      - Keyword，不做分词处理

- 中文分词
  - IK分词器

## 概念

- 全文搜索

  - 相关性
    - 评价查询与结果的相关程度，并进行排序
  - 分析
    - 将文本转换为token，目的是为了创建倒排索引，查询倒排索引

- 倒排索引

  - 根据属性值找记录，记录属性值在记录中出现的频率

- 单次搜索

  - 检查字段类型
  - 分析查询字符串
  - 查找匹配文档
  - 为每个文档评分

- 多次搜索

  - `"minimum_should_match" : "40%"`  设置多词之间匹配度
  - `"operator" : "and / or"`  设置多次之间的关系

- 组合搜索

  - ```json
    "query": {
        "bool": {
            "must": {
                "match": {
                    "": ""
                }
            },
            "must_not": {
                "match": {
                    "": ""
                }
            },
            "should": {
                "match": {
                    "": ""
                }
            }
        }
    }
    ```

    - 计算每个文档的相关度评分_score，将所有匹配的must和should语句的分数求和，除于must和should语句的条数
    - should中的内容不是必须匹配的，但是语句中没有must的话，那么至少会匹配一个

- 权重

  - should中通过boost来设置权重

- 短语匹配

  - 词和顺序都要匹配
  - slop：跳过n个词进行匹配



## 分布式文档

计算存储节点位置：`shard = hash(routing) % number_of_primary_shards`

- 文档搜索
  - 查询
    - 客户端发送搜索请求，节点收到后，创建一个from+size大小的空优先队列，然后将请求转发到其他分片
    - 每个分片在本地执行这个请求，将结果存入优先队列
    - 每个分片返回id和队列中的排序值给一开始接收请求的分片，这个分片合并这些值到自己的优先队列中后返回
  - 取回
    - 协调节点辨别出哪个Document需要取回，并且向相关分片发出GET请求
    - 每个分片加载document并且根据需要丰富它们，然后再将document返回协调节点
    - 一旦所有的document都被取回，协调节点会将结果返回给客户端

## 调优



