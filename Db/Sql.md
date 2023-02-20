---
title: SQL共通
toc: content
keywords: [sql, db]
---



# MySQL

## 函数

### Count()

- 

### DATE_FORMAT()

- 时间格式：' %Y-%m-%d %h:%i:%s '

- 当天：`DATE_FORMAT(now(), '%Y-%m-%d')`

### CAST

- 格式：cast(值 as 类型)

- 数据类型转换

  | 值            | 描述                              |
  | ------------- | --------------------------------- |
  | DATETIME      | 转换成'YYYY-MM-DD HH：MM：SS'格式 |
  | TIME          | 转换成'HH：MM：SS'格式            |
  | CHAR          | 转换成CHAR(固定长度的字符串)格式  |
  | SIGNED        | 转换成INT(有符号的整数)格式       |
  | UNSIGNED      | 转换成INT(无符号的整数)格式       |
  | DECIMAL	、 | 转换成FLOAT(浮点数)格式           |
  | BINARY        | 转换成二进制格式                  |

### WITH

- 将子查询抽取出来并起一个别名
- 在mysql8.0之前无效，旧版本用temporary table

# Oracle

## 函数

### CAST

- 格式：cast(值 as 类型)

- 数据类型转换

  | 值            | 描述 |
  | ------------- | ---- |
  | DATE          |      |
  | INTEGER       |      |
  | DOUBLE        |      |
  | number        |      |
  | varchar2(100) |      |
  | timestamp     |      |

  

