### MyCat

##### 分片配置

- 编辑schema.xml

  - ```xml
    <mycat:schema x,lns:mycat="http://io.mycat/">
    	<schema>
        	<table name="TB_ORDER" dataNode="dn1,dn2,dn3" rule="fen'pian" />
        </schema>
    </mycat:schema>
    ```

  - 

![image-20220321151919616](C:\Users\BDA\Documents\Note\pic\image-20220321151919616.png)

![image-20220321152025319](C:\Users\BDA\Documents\Note\pic\image-20220321152025319.png)

![image-20220321152140077](C:\Users\BDA\Documents\Note\pic\image-20220321152140077.png)

![image-20220321154733648](C:\Users\BDA\Documents\Note\pic\image-20220321154733648.png)

![image-20220321154646037](C:\Users\BDA\Documents\Note\pic\image-20220321154646037.png)

##### 垂直分片

![image-20220321162053378](C:\Users\BDA\Documents\Note\pic\image-20220321162053378.png)

![image-20220321162120396](C:\Users\BDA\Documents\Note\pic\image-20220321162120396.png)

![image-20220321162200323](C:\Users\BDA\Documents\Note\pic\image-20220321162200323.png)

![image-20220321161932413](C:\Users\BDA\Documents\Note\pic\image-20220321161932413.png)

##### 水平分片

![image-20220321162511985](C:\Users\BDA\Documents\Note\pic\image-20220321162511985.png)

##### 分片规则

- 范围分片
- 取模分片
- 一致性hash
  - 针对主键不是纯数字类型
- 枚举
- 应用指定
- 固定分片hash
  - id值取二进制低10为 & 1111111111
  - ![image-20220321163541472](C:\Users\BDA\Documents\Note\pic\image-20220321163541472.png)
- 字符串hash解析
  - 截取子字符串进行hash计算 & 1023
- 按天分片
- 按自然月分片

##### 管理

- 登录9066的管理端口
  - ![image-20220321164806894](C:\Users\BDA\Documents\Note\pic\image-20220321164806894.png)
- 安装Mycat-eye
  - 需要安装zookeeper

## 读写分离

### 一主一从

![image-20220321173444449](C:\Users\BDA\Documents\Note\pic\image-20220321173444449.png)

![image-20220321173432083](C:\Users\BDA\Documents\Note\pic\image-20220321173432083.png)

#### 双主双从

![image-20220321174336949](C:\Users\BDA\Documents\Note\pic\image-20220321174336949.png)

![image-20220321174727809](C:\Users\BDA\Documents\Note\pic\image-20220321174727809.png)

![image-20220321174741039](C:\Users\BDA\Documents\Note\pic\image-20220321174741039.png)

![image-20220321174824361](C:\Users\BDA\Documents\Note\pic\image-20220321174824361.png)

![image-20220321174847139](C:\Users\BDA\Documents\Note\pic\image-20220321174847139.png)

![image-20220321174902338](C:\Users\BDA\Documents\Note\pic\image-20220321174902338.png)

![image-20220321174934332](C:\Users\BDA\Documents\Note\pic\image-20220321174934332.png)

![image-20220321175012876](C:\Users\BDA\Documents\Note\pic\image-20220321175012876.png)

![image-20220321175107081](C:\Users\BDA\Documents\Note\pic\image-20220321175107081.png)



