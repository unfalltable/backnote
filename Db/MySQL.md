---
title: MySQL
keywords: [mysql, db]
---

##   常用命令

- 启动数据库
  - `mysql -h -P  -u -p`
    - -h主机名
    - -P 端口号
    - -u 后接用户名   （root）
    - -p后接密码        (无)
- 查看表结构
  - desc 表名;
- 查看服务器的版本
  - 登录mysql后使用`select version();`					
  - 未登录mysql使用
    - `mysql --version`
    - `mysql --V `
- 注释
  - #注释文字 
  - -- 注释文字
  - /*注释文字*/
- 查看系统变量
  - `show variables `
- 查看当前事务隔离级别
  - `select @@transaction_isolation`
- 设置当前mysql连接的隔离级别
  - `set transaction isolation level read committed`
- 设置数据库系统的全局的隔离级别
  - `set global transaction isolation level read committed`
- 导入sql文件
  - source 路径


---

## 操作符

- IN / NOT IN		  等于/不等于 列表中的任意一个
  - 相当于     ANY
  - IN 后的列表大小要尽量小，否则性能很差
    - 解决方法：创建临时表，in后接子查询该临时表
- ANY  |  SOME     和子查询返回的某一个值比较
- ALL                    和子查询返回的所有值比较

## DQL(数据查询语言--Data Query Language)

### 基础查询

`select 查询列表 from 表名`

- 起别名
  - 使用as
    - SELECT 100*9 AS 结果;
  - 使用空格
    - SELECT 100*9 结果;
  - 字符串用" "
    - SELECT salary AS "out put" FROM employees;

- 去重
  - DISTINCT
    - SELECT DISTINCT department_id FROM employees;
- +号的作用
  - 运算符
    - 俩个操作数都为数值型，则进行加法运算
    - 其中一方为字符型
      - 字符为数字则进行计算
        - "100" + 50 -> 150
      - 为字符则字符数值为0
        - "John" + 100 -> 100
    - 其中一方为null则都为null
      - null + 50 -> null

### 条件查询

`select 查询列表 from 表名 where 筛选条件 `

- 筛选条件
  - 条件表达式
    - \> \< \= \!= \>= \<=   \<>不等于
  - 逻辑表达式
    - \&& \|| \!
    - and or not 
  - 模糊查询
    - like
      - %任意多个字符，包含0个
      - _任意当个字符
      - \转义字符   也可以使用ESCAPE 转义
        - ESCAPE '$'  ==    \
      - 无法显示null值
    - between and 
      - 包含临界值
    - in(列表1，列表2)
      - 判断值是否属于in列表中
      - 不能使用通配符
    - is null
      - <=>   安全等于,可以搭配null

### 排序查询

`select 查询列表 from 表名 where 筛选条件 order by 排序列表 [asc默认升序/desc降序] `

#### 注意

- order by 后可以用别名
- order by 要放在语句的最后
- 支持多个字段，函数，表达式，别名

---

### 函数

#### 字符函数

- 连接字段
  - CONCAT(''a,'b','c'); -> abc
- 判断是否为空
  - IFNULL(可能为null的值，如果为空返回的值)
  - isnull(可能为null的值)    
    -  如果为空返回1 ,反之为0
  
- 获取字节长度
  - LENGTH(str)
- 大小写
  - UPPER()		大写
  - LOWER()      小写
- 截取字符串
  - substr(str, 索引值开始,长度)		
    - 索引从1开始
- 返回字符串B在字符串A中第一次出现的索引
  - instr(strA,strb)
- 去除前后字符，默认去除空格
  - trim('去除的字符'，str);
- 填充字符串
  - lpad(str,结果长度,填充字符) 			左填充
  - rpad(str,结果长度,填充字符) 			右填充
- 替换字符串中字段
  - replace(str,strA,strB)           将strB替换sttA

#### 数学函数

- 四舍五入
  - round(数值)
    - 3.6 -> 4
  - round(数值，小数点保留位数)
    - 3.1596 -> 3.16   保留小数点后两位
- 向上取整
  - ceil(数值)
    - 3.14 -> 4
- 向下取整
  - floor(数值)
    - 3.14 -> 3
- 截断
  - truncate(数值，保留小数点后位数)					不进行四舍五入
    - 3.1796 ->  3.17			保留两位
- 取余
  - mod()
    -  mod(10,3) -> 1 
- 获取随机数
  - rand()  		返回0-1之间的小数，无限接近1

#### 日期函数

- 返回当前系统日期和时间
  - now()
- 返回当前系统日期
  - curdate()
- 返回当前系统时间
  - curtime()
- 获取指定的部分，年月日时分秒
  - YEAR(时间字段)
  - MONTH()
    - MONTHNAME()		以英文形式返回月
  - day()
- 将日期格式的字符串转换成指定格式的日期
  - STR_TO_DATE('4-9-1999'.'%m-%d-%Y') -> 1999-04-09
- 将日期转换成字符
  - DATE_FORMAT('2018/6/5','%Y年%m月%d日') ->	2018年06月05日
- 计算俩个时间的之间的天数
  - DATEDIFF()

#### 其他函数

- version()			 	        当前数据库服务器版本
- databases()                 当前打开的数据库
- user()                           当前用户
- password('字符')        返回该字符的密码形式
- md5('字符')                 返回该字符的md5加密形式

#### 流程控制函数

- 判断语句
  -  IF(判断,true返回的值,false返回的值)

  - case

    - case(要判断的值)

      when 常量1 then 要显示的值或语句;

    ​	   when 常量2 then 要显示的值或语句;

    ​	   ELSE 以上都不符合要显示的值或语句;

    ​	   END

    - case

      when 判断1 then 要显示的值或语句;

      when 判断2 then 要显示的值或语句;

      ELSE 以上都不符合要显示的值或语句;
    
      END
    
  -  if结构
  
     -  if 条件1 then 语句1；
  
        elseif 条件2 then 语句2；
  
        。。。
  
        【else 语句n;】
  
        end if;

#### 分组函数（统计函数，聚合函数 ）

`select 查询列表 from 表名 where 筛选条件 GROUP BY 子句语法`

- ##### 常用函数
  - sum()  		求和					处理数值型

  - avg()          平均值            处理数值型

  - max()         最大值         

  - min()          最小值               

  - count()       计算个数        

    - COUNT(字段)       统计该字段非空值的个数
    - COUNT(*)	          统计结果集行数，比COUNT(字段)效率高
- COUNT(常量值)   统计结果集行数，比COUNT(字段)效率高 
  
  - 以上都忽略null值,null值不参与运算,都可和distinct搭配使用 
    - count(distinct 字段)
  
- 分组后筛选数据
  
  - HAVING  
    - 对分组筛选后的数据进行操作，用在GROUP BY 后，效果同where
  
- ##### 注意

  - GROUP BY 和 HAVING 后都可接别名
  - 筛选条件能写在where后优先现在where后，效率高
  - 多个字段分组用逗号隔开即可

---

### 连接查询（多表查询）

#### 内连接(支持SQL92,SQL99)

- SQL92 `SELECT 查询列表 FROM 表1,表2`
  - 在where中添加连接条件，结果是两表的交集
  - 自连接 ？

- SQL99 `SELECT 查询列表 FROM 表1 INNER JOIN 表2 ON 连接条件`
  - 在 on 之后添加连接条件
  - 自连接 ？

#### 外连接(支持sql99)

- sql99 `SELECT 查询列表 FROM 表1 JOIN 表2 ON 连接条件`

- 外连接的查询结果是主表中的所有记录，如果表2有和表1匹配的值则显示。没有则为null
- 左外连接、右外连接 `SELECT 查询列表 FROM 表1 left/right JOIN 表2 ON 连接条件`
- 全外连接（MySQL不支持）`SELECT 查询列表 FROM 表1 FULL JOIN 表2 ON 连接条件`
  - 结果集是内连接的结果+表1有表2没有的+表2有表1没有的

#### 交叉连接

- `SELECT 查询列表 FROM 表1 CROSS JOIN 表2 ON 连接条件`

- 就是笛卡尔积

### 子查询

- 出现在其他语句内部的select语句称为子查询或内查询

- 内部有子查询的语句为主查询

#### 分类

##### 按位置

- SELECT后
  - 结果集只能为标量子查询（一行一列）
- FROM后
  - 将子查询的结果充当一张表，必须起别名
- WHERE或HAVING后
  - 标量子查询（单行子查询）
    - 子查询返回的值是一行一列的
  - 列子查询（多行子查询）
    - 子查询返回的值是一列多行的
    - IN / ANY / SOME / ALL
  - 行子查询（一行多列）
    - 子查询返回的值是一行多列的
    - (A,B) = (select语句)
- EXISTS后(相关子查询)
  - EXISTS（完整的查询语句）
    - 有结果返回1，没结果返回0

##### 按结果集行列数

- 标量子查询（一行一列）
- 列子查询（一列多行）
- 行子查询（一行多列）
- 表子查询（多行多列）

#### 特点

- 子查询的执行优先于主查询

---

### 分页查询

- `SQL 语句 limit offset,size `
  - offset              起始索引
    - 索引从0开始
  - size                 要显示的个数

#### 注意

- limit最后执行
- 索引从0开始可以省略
- page 显示的页数  size每一页显示的条目
  - limit (page-1)*size,size  

### 联合查询

`语句A union 语句B union...`

将多条查询语句的结果合并成一个表

#### 注意

- 要求多条查询语句查询的字段个数要一致 
- 要求多条查询语句查询的每一个字段的类型和顺序最好一致
- 默认情况会自动去重，不想去重可以使用union ALL

---

## DML(数据操纵语言--Data Manipulation Language)

### insert

#### 方式一

`insert into 表名(字段名...) values(值...);`

- 注意
  - 插入的值的类型要与字段的类型一致或兼容
  - 可以为空的字段可以不写也可以写null
  - 可以省略字段名，但所有字段都得有值
  - 支持多行插入，用逗号分隔
  - 支持子查询

#### 方式二

`insert into 表名 set 字段名=值...`

- 注意
  - 不支持插入多行
  - 不支持子查询

---

### update

#### 修改单表

`update 表名 set 字段名=新值,字段名=新值... where 筛选条件`

#### 修改多表

`update 表1 别名 inner/left/right join 表2 别名 on 连接条件 set 字段名=值。。。 where 筛选条件`

---

### delete

#### 方式一

##### 单表删除

`delete from 表名 where 筛选条件`

##### 多表删除

`delete 要删除的表的别名 from 表1 别名 inner/left/right join 表2 别名 on 连接条件 where 筛选条件`

##### 注意

- 删除后自增长列从删除前断点开始
- 有返回值（能显示删除了几条）
- 删除后能回滚

#### 方式二

`truncate table 表名`

##### 注意

- 只能删除整张表，不能使用where
- 删除后自增长列从1开始
- 没返回值（不能显示删除了几条）
- 删除后不能回滚

---

## DDL(数据定义语言--Data Define Language)

### 库的管理

#### 创建库

`create database (if no exists) 库名`

#### 修改库

- 修改库的字符集
  - `alter database 库名 character set 编码格式`

#### 删除库

`drop database (if is exists) 库名`

#### 注意

- 创建数据库时默认字符集是utf-8

### 表的管理

#### 创建表

`create table 表名(`

​					`字段名 字段类型[(长度)] [约束],`

​					`字段名 字段类型[(长度)] [约束],`

​					`...) `

#### 修改表

- 修改字段名
  - `alter table 表名 change column 旧字段名 新字段名 新字段类型`
- 修改字段的类型
  - `alter table 表名 modify column 字段名 新字段类型`
- 添加字段
  - `alter table 表名 add column 新字段名 新字段类型`
- 删除字段
  - `alter table 表名 drop column 字段名`
- 修改表名
  - `alter table 表名 rename to 新表名`
- 添加约束
  - 添加列级约束
    - `alter table 表名 modify column 字段名 字段类型 新约束`
  - 添加表级约束
    - `alter table 表名 add [constraint 约束名] 约束类型(字段名) [外键的引用]  `

---

#### 删除表

`drop table (if is exists) 表名`

#### 复制表

##### 复制表的结构

`create table 复制表 like 原始表`

##### 复制表的部分结构

`create table 复制表 select 字段1，字段2 from 原始表 where 0`

##### 复制表的结构和所有数据

`create table 复制表 select * from 原始表`

##### 复制表的结构和部分数据

`create table 复制表 select 字段1，字段2 from 原始表 where 筛选条件`

---

## 数据类型

### 整形

- tinyint, smallint, mediumint, int/integer, bigint
  - int unsigned    无符号整形

##### 注意

- zerefill    0填充，会成无符号整形
- 如果插入值超出范围就会插入临界值

### 小数

- 浮点型
  - float(M,D)
  - double(M,D)

- 定点型
  - dec(M,D)
  - dectmal(M,D)
    - M默认是10
    - D默认是0

#### 特点

- M和D
  - M是整数+小数个数
  - D是小数个数
  - 如果超出M则插入临界值
  - 超出D则四舍五入
- dectmal的精度较高

### 字符型

- char
  - 固定长度，比较耗费空间，但效率高一些
- varchar
  - 可变长度，比较节省空间，但效率要低一些
- text
  - bigtext
- blob(保存较大的二级制文件)
- binary(保存较短的二级制文件)
- varbinary(保存较短的二级制文件)
- enum
- set

### 日期型

- date       日期
- time       时间
- datetime       日期+时间      范围大
- timestamp（时间戳）    日期+时间      范围小    受时区影响 

#### 注意

- set time_zone='+9:00'      地区切换成东九区

## 约束

- not null                 非空        字段不能为空
- default                  默认值    该字段有默认值
- primary key          主键        该字段具有唯一性，非空
- unique                   唯一       该字段具有唯一性，可以为空
- check                     检查       mysql中不支持          
- foreign key            外键        限制俩个表的关系，保证该字段的值和另一个表的值对应
  - 后接          references 表名(字段名) 
  - 在从表设置外键关系
  - 从表的外键字段和主表的关联字段的类型要一致或兼容
  - 主表的关联字段必须是一个key(一般是主键或唯一)
- [constraint  约束名] 约束类型(字段名)

### 删除外键的方法

- #### 方法一：级联删除

  - on delete cascade

- #### 方法二：级联置空

  - on delete set null

### 注意

- `show index from 表名`         查看表中所有的索引，包括主键，外键，唯一
- 主键不允许为空，唯一可以为空
- 插入数据时先插入主表，删除数据先删除从表

## 变量

### 系统变量

- 查看所有系统变量

  - `show global | [session] variables`  
- 查看满足条件的系统变量

  - `show global | [session] variables like ''`
- 查看指定的系统变量的值

  - `select @@global | [session] 系统变量名`
- 为系统变量赋值

  - `set global | [session] 系统变量名 = 值 `
  - `set @@global | [session].系统变量名 = 值`

#### 全局变量

- 查看所有全局变量
  - `show global variables` 
- 查看满足条件的全局变量
  - `show global variables like ''`
- 查看指定的全局变量的值
  - `select @@global.全局变量名`
- 为全局变量赋值
  - `set @@global.全局变量名 = 值`
- 注意
  - 必须拥有super权限才能为系统变量赋值，作用域为整个服务器

#### 会话变量

- 查看所有会话变量
  - `show session variables` 
- 查看满足条件的会话变量
  - `show session variables like ''`
- 查看指定的会话变量的值
  - `select @@session .会话变量名`
- 为会话变量赋值
  - `set @@session .会话变量名 = 值`
  - `set session 会话变量名 = 值 `
- 注意
  - 作用域为当前的会话/连接

### 自定义变量

#### 用户变量

- 声明并初始化
  - `set @用户变量名=值:`
  - `set @用户变量名:=值:`
  - `select @用户变量名:=值:`
- 赋值
  - `set @用户变量名=值:`
  - `set @用户变量名:=值:`
  - `select @用户变量名:=值:`
  - `select 字段 into 用户变量名 from 表`
- 使用
  - `select @用户变量名`

#### 局部变量

- 声明
  - `declare 变量名 类型`
  - `declare 变量名 类型 default 值`
- 赋值
  - `set @局部变量名=值:`
  - `set @局部变量名:=值:`
  - `select @局部变量名:=值:`
  - `select 字段 into 局部变量名 from 表`
- 使用
  - `select @局部变量名`
- 注意
  - 定义只能放在begin-end中的第一句

### 游标

- 声名
  - ​	`declare 游标名称 cursor for 查询语句`
- 打开
  - `open 游标名称`
- 获取游标记录
  - `fetch 游标名称 into 变量[变量]`
- 关闭游标
  - `close 游标名称`

### 条件处理

![image-20220320155557326](C:\Users\BDA\Documents\Note\pic\image-20220320155557326.png)

## 事务

### 特性（ACID）

- 原子性(Atomicity)
  - 事务是一个不可分割的工作单位，要么都发生，要么都不发生
- 一致性(Consistency)
  - 事务必须使数据库从一个一致性状态变换到另外一个一致性状态
- 隔离性(Isolation)
  - 一个事务的执行不能被其他事务干扰
- 持久性(Durability)
  - 事务一旦被提交，对数据库的改变就是永久性的

### 创建事务  

- 隐式事务
  - 没有明显的开启和结束的标记
  - 如insert，update，delete

- 显式事务
  - 有明显的开启和结束的标记
  - 需要先设置自动提交功能为禁用
    - set autocommit=0;
      - 当前会话有效
  - 开启事务
    - start transaction          可选
  - 编写sql语句
  - 结束事务
    - commit            提交事务
    - rollback           回滚事务

### 可能出现的问题

- 脏读
  - T1读取了T2还没有提交的事务
- 不可重复读
  - T1俩次读取T2同一字段的值不同
- 幻读
  - T1多次读取T2同一字段返回的数据行数不同，因为T2做了插入操作

### 隔离级别

- oracle支持两种隔离级别
  - read commited            默认
  - serializable
- Mysql支持四种隔离级别
  - repeatable read              可重复读    mysql默认
    - 多次读一个字段值值相同，禁止其他事务对这个字段进行更新
    - 可避免脏读和不可重复读，仍存在幻读
  - read uncommitted         读未提交数据
    - 允许事务读取其他事务未提交的数据
    - 脏读，不可重复读，幻读都不可避免
  - read commited               读已提交的数据
    - 允许事务读取其他事务已提交的数据
    - 可避免脏读，但不可重复读和幻读不可避免
  - serializable                      串行化                       最高级别
    - 事务操作期间加锁，其他事务无法操作
    - 脏读，不可重复读，幻读都可避免
    - 性能十分低下

![image-20220318132259385](C:\Users\BDA\Documents\Note\pic\image-20220318132259385.png)

### SavePoint

- 搭配rollback to 使用

### delete和truncate在事务使用时的区别

- delete支持回滚，truncate不支持回滚

## 视图

### 视图的好处

- 重用sql语句
- 简化sql操作，屏蔽查询细节
- 保护数据，提高安全性
- 临时的表

### 视图的创建

`create view 视图名 as 查询语句`

- 虚拟表，和普通表一样使用
- 相当于给查询起别名

### 视图的修改

`create or replace view  视图名 as 查询语句`

`alter view 视图名 as 查询语句`

### 删除视图

`drop view 视图名，视图名,...`

### 查看视图

`desc 视图名`

`show create view 视图名`

### 视图的更新

- 视图插入数据原表也会插入
- 视图插入数据需要原表和视图都要有相同的字段
- 视图可以增删改，原始表会受到影响，一般很少会这样用
- 视图一般会添加权限，只可读不可写
- 具备以下特点的视图不允许更新
  - 包含分组函数，distict，group by，having，union/union all
  - 常量视图
    - select 常量
      - select 'john' name；
  - Select中包含子查询
  - 连接（join）
    - 可以更改数据
  - from后是一个不能更新的视图
  - where子句的子查询引用了from子句中的表

### 视图的检查

- MySQl允许在一个视图上再创建一个视图，为了保证两个视图的一致性，提供了两个属性检查方式

- `创建包含视图的视图语句 + with cascaded check option`
  - 级联，对其依赖的视图或表也有约束，需要同时满足两条语句的规则


- `创建包含视图的视图语句 + with local check option`
  - 仅检查当前语句


## 存储过程

### 作用

- 减少网络交互提升效率

### 用法

```sql
create procedure 存储过程名(参数列表)
begin
	合法的sql语句
end
```

- 参数列表包含三部分：参数模式，参数名，参数类型
  - 参数模式
    - in	   作为输入，需要传值
    - out    作为输出，即返回值
    - inout 既是输入，也是输出
- delimiter 结束标记
  - 修改结束标记为指定的结束符号
- 调用
  - `call 存储过程名(实参列表)`
- 删除
  - `drop procedure 存储过程名`
- 查看存储过程的信息
  - `show create procedure 存储过程名`
- 注意
  - 如果方法体只有一句话，可以省略begin end语句
  - 方法体中每条sql结尾都需要加分号

## 存储函数

```sql
create function 函数名(参数列表) returns 返回类型
begin
	函数体
end
```

### 调用

- `select 函数名(参数列表)`

### 查看

- `show create function 函数名`

### 删除

- `drop function 函数名` 

### 注意

- 有且仅有一个返回值
- 参数列表包含二部分：参数名，参数类型
  - 参数列表只能是int类型


## 循环结构

### 分类

- while

  - ```sql
    while 循环条件 do
    	循环体;
    end while;
    ```

- loop

  - ```sql
    循环名:loop
    	循环体;
    	iterate / leave 循环名
    end loop;
    ```
    
    - iterate
      - 相当于continue;
    - leave
      - 相当于break; 

- repeat

  - ```sql
    repeat
    	循环体;
    until 结束循环的条件
    end repeat;
    ```

## 触发器（Trigger）

- Insert触发器
- Update触发器
- Delete触发器

![image-20220320160434561](C:\Users\BDA\Documents\Note\pic\image-20220320160434561.png)

![image-20220320160526307](C:\Users\BDA\Documents\Note\pic\image-20220320160526307.png)

## InnoDB引擎

### 逻辑存储结构

- 表空间：存记录、索引
- Segment段：数据段（叶），索引段（非叶）、回滚段
- Exten区：1M，64页，会申请4-5个区
- Page页：16kb，最小单元，操作都是以页为单位的
- Row行：数据

### 内存架构

- BufferPoll 缓冲池
  - 缓存的是经常操作的真实数据，执行增删改时会先操作缓存中的数据（没数据的话会先从磁盘加载并缓存），之后按一定时间频率刷进磁盘
    - freePage：未使用的页
    - cleanPage：有数据但未修改的页
    - dirtyPage：数据被修改的页
- ChangeBuffer 更改缓冲区
  - 针对非唯一 二级索引
  - 增删改时BufferPoll中没数据，先记录操作
  - 8.0之后引入的change Buffer，以前是insert buffer 
  
- AdaptiveHashIndex 自适应哈希索引
  - 用于优化BufferPoll数据的查询
  - Innodb会监控表上索引页的查询，发现走hash更快时会建立hash索引
- LogBuffer 日志缓冲区
  - 保存redo log、undo log

### 磁盘结构

![image-20220320234541137](C:\Users\BDA\Documents\Note\pic\image-20220320234541137.png)

![image-20220320234634719](C:\Users\BDA\Documents\Note\pic\image-20220320234634719.png)

![image-20220320234739425](C:\Users\BDA\Documents\Note\pic\image-20220320234739425.png)

![image-20220320235020735](C:\Users\BDA\Documents\Note\pic\image-20220320235020735.png)

### 事务原理

- 通过两个日志，保证原子性，一致性，持久性
- 通过MVCC+锁，保证隔离性

#### redo log

- 记录的是数据的修改，保证持久性
- 由 **redo log buffer** 和 **redo log file** 组成，前者在内存中，后者在磁盘中
- 执行数据变更操作时，数据会先加载到内存中，然后在内存中更新，同时写入 redo log buffer 再由 redo log buffer写入redo log file，内存中的数据也会更新到数据库中
- 事务未提交时，redo log中的记录处于prepare状态
- 事务提交后，会将记录修改为commit状态，然后把所有修改数据的记录刷新到磁盘中（redo log file 中）
- redo log 记录的是一定时间内的修改操作，不会存储历史变更，当数据刷新到磁盘时，redo log 中的数据就失效了

#### undo log

- 记录的是数据被修改前的信息，保证原子性
- undo log 是逻辑日志
  - 当delete一条记录时，undo log会记录一条对应的插入数据
  - 当update一条记录时，undo log会记录一条相反的update数据
- 发生回滚时执行undo log即可
- undo log 在事务执行时产生，事务提交时，undolog不会立即删除，因为还用于MVCC（快照读）
  - 如果是insert，产生的undolog日志只在回滚时需要，在事务提交后可被立即删除

- undo log 采用段的方式进行管理和记录，存放在**rollback segment**回滚段中，内部包含1024个undo log segment

#### MVCC（多版本并发控制）

- 当前读
  - 读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对当前读取的记录加锁，意向锁都是一种当前读
- 快照读
  - 简单的select（不加锁）是快照读，读取的记录的可见版本，有可能是历史数据，不加锁，是非阻塞读，不同的事务隔离级别，快照读有不同的实现
    - ReadCommitted：每次select都是快照读（每次数据都可能不同）
    - Repeatable Read：只有第一次select是快照读（后续的数据和第一次相同）
    - Serializable：快照读退化为当前读，每次都一样且加锁


##### 特点

- 维护一个数据的多个版本，使得读写操作没有冲突
- MVCC的具体实现需要依赖记录中的**三个隐式字段**，还有undo log版本链，readView
  - DB_TRX_ID：最近修改事务的ID，记录增删改记录的事务ID
  - DB_ROLL_PTR：回滚指针，指向这条记录的上一个版本，用于配合undo log
  - DB_ROW_ID：隐藏主键，如果表结构没有指定主键，将会生成隐藏主键
- **undo log**版本链
  - 不同事务或相同事务对同一条记录进行修改，会导致该记录的undolog生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录

- **readview**快照读
  - 是SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的未提交的事务id，包含了四个核心字段
    - m_ids：当前活跃的事务ID集合
    - min_trx_id：最小活跃事务ID
    - max_trx_id：预分配事务ID，当前最大事务ID+1
    - creator_trx_id：readview创建者的事务ID

  - 版本链数据访问规则（trx_id 表示当前事务ID）
    - **trx_id** == creator_trx_id：数据是当前事务更改的，可以访问
    - **trx_id** < min_trx_id：数据已经提交了，可以访问
    - **trx_id** > max_trx_id：该事务是在ReadView生成后才开启的，可以访问
    - min_trx_id <= **trx_id** <= max_trx_id：数据已经提交，trx_id不在m_ids中可以访问

  - 不同的隔离级别，生成ReadView的时机不同
    - READ COMMITTED：在事务中每一次执行快照读时生成ReadView
    - REOEATABLE READ：仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView


## 索引

### 索引结构

- 不同的存储引擎实现的索引结构不同
  - **B+Tree索引**：大部分引擎都支持
  - **Hash索引**：**Memory引擎支持**
    - 索引速度快，效率高，但不支持范围索引
    - 无法利用索引排序
  - R-tree(空间索引)：**MyISAM引擎支持**
    - 的一个特殊索引类型，主要用于地理空间类型
  - full-text(全文索引)：**elasticSearch**，**InnoDB**(5.6之后)、**MyISAM**支持

### 索引分类

- **主键索引** 
  - 只能有一个
- **唯一索引** 
  - 该索引字段的值需要不重复
- **常规索引** 
  - 该索引字段的值可能出现重复

- **全文索引** 
  - 该索引字段的值在每个位置都可能出现


### InnoDB中的索引

- **聚集索引**
  - 必须有，而且只有一个，一般是主键索引，没有就选唯一索引，再没有InnoDB会自动生成一个rowid作为隐藏的聚集索引
  - 索引的叶子结点保存了行数据
- **二级索引**
  - 可以有多个
  - 索引的叶子结点存放的是对应的字段值

### 索引语法

- 创建索引
  - `create index 索引名 on 表名(字段名)` 
  - `alter table 表名 add index 索引名(字段)` 
- 查看索引
  - `show index from 索引名`
- 删除索引
  - `drop index 字段名 on 表名`
  - `alter table 表名 drop index 索引名` 
- 查看表的索引情况
  - `show index from 表名` 
  - `show create table 表名` 
- 修改索引
  - 只能通过删除索引再添加索引

## 锁机制

### 全局锁

- 加锁
  - `flush 数据据库名 with read lock`
- 在备份时默认会加锁，我们可以加上 --single-transaction来完成不加锁备份
  - `mysqldump --single-transaction -uroot -p 数据库名>文件名.sql`


### 表级锁

#### 读写锁

##### 表共享读锁

- `load tables 表名... read`
- `unlock tables`

##### 表独占写锁

- `load tables 表名... write`
- `unlock tables`

#### 元数据锁（MDL）

- 由系统自动控制，在访问一张表时会自动加上，MDL锁主要是维护元数据的数据一致性，在表上有活动事务时，不可以对表进行写入操作
- 对表进行增删改查时，加MDL读锁（共享）
- 对表结构进行修改时，加MDL写锁（排他）
- 查看元数据锁
  - `select object_type, object_schema, object_name, lock_type, lock_duration from performance_schema.metadata_locks;` 

#### 意向锁

##### 意向共享锁

- `select ... lock in share mode`

##### 意向排他锁

- `insert / update / delete / select ... for update` 

![image-20220320175841051](C:\Users\BDA\Documents\Note\pic\image-20220320175841051.png)

### 行级锁

#### 行锁

- 共享锁S：
  - 允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁

- 排他锁X：
  - 允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁


![image-20220320181113652](C:\Users\BDA\Documents\Note\pic\image-20220320181113652.png)

![image-20220320182348192](C:\Users\BDA\Documents\Note\pic\image-20220320182348192.png)

#### 间隙锁

- 

![image-20220320183111394](C:\Users\BDA\Documents\Note\pic\image-20220320183111394.png)

![image-20220320183120778](C:\Users\BDA\Documents\Note\pic\image-20220320183120778.png)

#### 临键锁（next-key lock）

- 间隙锁 + 行锁

## Mysql工具

![image-20220321130500940](C:\Users\BDA\Documents\Note\pic\image-20220321130500940.png)

![image-20220321130746523](C:\Users\BDA\Documents\Note\pic\image-20220321130746523.png)

![image-20220321131301287](C:\Users\BDA\Documents\Note\pic\image-20220321131301287.png)

![image-20220321132826897](C:\Users\BDA\Documents\Note\pic\image-20220321132826897.png)

![image-20220321132929087](C:\Users\BDA\Documents\Note\pic\image-20220321132929087.png)

![image-20220321133539942](C:\Users\BDA\Documents\Note\pic\image-20220321133539942.png)

![image-20220321134719464](C:\Users\BDA\Documents\Note\pic\image-20220321134719464.png)

## 日志

### 错误日志

- 进入log下输入
  - `tail -f` 查看实时日志

### 二进制日志（binlog）

- 记录了所有DDL、DML语句
- 用于数据恢复、主从复制
- 在事务提交的时候将记录刷新到磁盘
- 日志格式
  - STATEMENT：记录的是sql语句，修改也会记录
  - ROW：记录的是每一行的数据变更（默认）
  - MIXED：混合了以上两种，默认是STATEMENT，某些特殊情况是ROW
- 日志删除
  - 由于每天的binlog数据巨大，需要及时清除
  - `reset master`         删除全部日志，编号从01开始
  - `purge master logs to 'binlog.编号'`    删除编号之前的所有日志
  - `purge master logs before 'yyy-mm-dd hh24:mi:ss'`    删除这个 时间点之前产生的所有日志
- 查看binlog是否打开
  - `show variables like '%log_bin%'`

- 主要用作**主从复制**和**数据恢复** 


### 查询日志

- 记录所有操作，默认关闭

### 慢查询日志

- 记录执行时间超过long_query_time的sql语句，默认关闭

## 主从复制

- Master主库在事务提交时，会把数据变更记录在Binlog中
- 从库读取主库的Binlog，写入到从库的中继日志Relay Log
- 从库重做RelayLog中的事件，将改变反映它自己的数据

### 步骤

- 主库配置

  - 修改`/etc/my.cnf` 

    - ```yaml
      #mysql服务ID，保证整个集群环境中为唯一，取值范围：1 - 2^32 - 1,默认唯一
      server-id=1
      #是否只读，1代表只读，0代表读写
      read-only=0
      #忽略的数据，指不需要同步的数据库
      #binglog-ignore-db=mysql
      #指定同步的数据库
      #binlog-do-db=db01
      ```

  - 重启mysql

    - `systemctl restart mysqld`

  - 登录mysql，创建远程连接的账号，并授予主从复制权限

    - ```yaml
      #创建itcast用户，并设置密码，该用户可在任意主机连接该MySQL
      CREATE USER 'itcast'@'%' IDENTIFIED WITH mysql_native_password BY 'Root@123456';
      #为'itcast@%'用户分配主从复制权限'
      GRANT REPLICATION SLAVE ON *.* TO 'itcast'@'%';
      ```

- 从库配置

  - 修改配置文件/etc/my.cnf

    - ```yaml
      #服务ID，保证整个集群环境中唯一
      server-id=2
      #是否只读，1代表只读
      read-only=1
      ```

  - 重启mysql

    - `systemctl restart mysqld`

  - 登录mysql，设置主库配置

  - 开启同步

  - 查看主从状态
