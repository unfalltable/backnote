---
title: Java
toc: content
keywords: [Java, basic]
---

## 面向对象

### 继承、封装、多态

- 继承
  - 父子类关系 is-a
  - 从已有类派生出新类，派生类有父类的属性和方法，能对其进行扩展，父类中的private属性不会被继承
  - 提高了代码的复用性，可维护性，但耦合度高
- 封装
  - 将数据和操作封装在一起，成为一个不可分割的整体
  - 隐藏了类的内部实现机制，对外暴露访问方法，保护数据
  - 提高了代码的复用性，安全性，减少耦合
- 多态
  - 是类与类之间的关系，有编译期多态和运行期多态
  - 需要继承、封装、重写、父类引用指向子类对象，用父类做参数，具体子类实现时再确定
  - 提高了代码的可扩展性，但不能使用子类特有方法

### 类图

泛化关系

实现关系

聚合关系

组合关系

关联关系

依赖关系

## 关键字

#### transient

- 修饰的字段不参与序列化
- 只能修饰变量
- 当这个变量可以通过其他变量来获得时就可以不参加序列化，即添加transient修饰
  - 例如：三角形的长，宽，面积，则面积可以不参与序列化
  - Logger对象也不需要

#### static

- 修饰的静态变量是不参与序列化的

## 接口

- 接口可以有默认方法

## 方法

- 静态方法
- 抽象方法
- final方法

## 异常

### 自定义异常

- 需要继承RuntimeException
  - 定义错误码和错误消息，还有各种参数的构造函数，包含异常信息，在抛异常的时候传入

## 序列化

- 就是将对象转换成字节序列，实现持久化和网络传输
- 方式
  - 实现Serializable接口
  - 实现Externalizable接口
    - 实现writeExternal()
    - 实现readExternal()
- transient可以防止属性序列化，因为序列化可能会破坏单例

## 集合


### ArrayList

#### 知识点

- 底层是数组，通过复制数组实现增删
  - 增删尾部快，其他位置慢
  - 按索引查询快，按内容查询和LinkedList差不多
- 可以利用cpu缓存，局部性原理
- 涉及到size的修改都会使 modCount++


### LinkedList

#### 知识点

- 实现了 List 接口和 Deque 接口，当作是一个双端队列，或者是栈，但ArrayDeque有着更好的性能
- 底层是双向链表结构
  - 查询首尾元素速度快，其余位置慢
  - 按下标去增删速度快，按内容去遍历增删慢
- 内部维护了First 和 Last ，包含了大量操作首尾元素的方法
- 检索下标时判断 index 靠前还是靠后，决定从后往前，还是从前往后
- index方法会判断对象是否为空，根据空或者非空来找下标
- 占用内存多
- 可以存null
- 如果需要多个线程并发访问，可以先采用`Collections.synchronizedList()`方法对其进行包装

### ArrayDeque

#### 知识点

- Stack 是类，Queue 是接口
- Java不推荐用Stack了，所以想用栈或者队列首先考虑ArrayDeque
- Deque双端队列，既可以当作栈使用，也可以当作队列使用，*ArrayDeque*和*LinkedList*是*Deque*的两个通用实现
- 初始容量 8，扩容 x 2
- 底层是循环数组，维护了一个 head 和 tail，非线程安全的，不可存null
  - head 指向第一个元素
  - tail 指向最后一个元素的后一位

### PriorityQueue

#### 知识点

- 优先队列，实现了Queue 接口，不可存null
- 作用是能保证每次取出的元素都是队列中权值最小的
- 数组中的元素是按层序遍历树排列的
- 初始容量11，扩容
  - 申请一个更大的数组，并将原数组的元素复制过去
  - 原始容量 < 64，则新队列长度 = 原始容量 * 2 + 2
  - 原始容量 > 64，则新队列长度 = 原始容量 +原始容量 >> 1
  
- 如果扩容超过数组的最大长度，则按数组的最大长度进行扩容
- 具体是通过小顶堆，可以用数组实现
  - leftNo = parentNo * 2 + 1
  - rightNo = parentNo * 2 + 2
  - parentNo = ( nodeNo - 1 ) / 2
- 时间复杂度：O(LogN)

### Vector

- 知识点
  - Vector是同步的 单线程速度慢 早期使用Elements遍历集合
  - Elements的返回值是Enumeration\<E\>（向量/枚举）
    - Enumeration接口是迭代器前身
      - hasMoreElements()   判断集合是否有下一个元素
      - nextElement()       取出集合元素

### HashSet

- 知识点

  - 只能通过迭代器和增强for循环遍历集合

  - 不同步，不可存储重复元素，无序的集合，底层是一个哈希表结构(查询速度非常快)

  - 存储自定义类型的元素
    - 因为Set集合保证元素唯一，防止出现相同元素必须重写hashCode()和equals()方法
  - 对HashMap进行了一个封装，适配器模式

  - 内部都是调用的hashmap 的方法
  - key放存储的值，value存储的一个静态常量

### HashMap

#### 知识点

- 无序，存取元素顺序可能不一致

- hashmap是懒惰创建数组的，第一次调用put方法执行putVal时才创建数组

- 默认初始长度为16

- hashMap的 key 和 value 可以为null

- 底层数据结构
  - 1.7是数组+链表
  - 1.8是数组+链表 or 红黑树


- 扩容
  - 扩容2倍
    - 1.7是大于等于阈值且插入时的位置不为空时扩容
      - 扩容后的链表会以头插法插入（扩容死链出现的原因）
        - a-b-c 会变成 c-b-a
        - ?
    - 1.8是大于阈值就扩容，不改变原链表或红黑树的顺序
  - 扩容是创建一个新的数组，将旧数组的值移动到新数组，通过改变引用，而不是创建新的元素
  
- 存储自定义类型的键值

  - Map集合中要保证Key是唯一的，所以必须重写hashcode()和equals()方法
    - 重写hashcode()是为了对象的key在整个hashmap中有更好的分布，提高查询性能
    - 重写equals()是为了防止两个对象的hash值一样，通过比较值来区分

- 存储的对象一定是不能变的，对象的属性发生改变会影响hash值，就会导致找不到对象，除非hash方法中不包含此改变对象的属性

- put过程

  - put 之前会遍历一次 map，看是否已经存在这对 键值
  - 插入空位置时将key和value封装为entry对象（1.8后是node对象）插入该位置
    - 不为空时
      - jdk7是头插法插入到当前节点下链表的头部
      - jdk8后是判断当前位置是红黑树还是链表
        - 红黑树：
          - 将key和value封装为一个红黑树节点并插入，这个过程会判断是否存在key值
        - 链表：
          - 将key和value封装为一个node对象用尾插法插入链表，这个过程会判断是否存在key值和统计当前节点个数，如果大于8个节点则将该链表转换为红黑树
            - 如果hashmap长度小于64则会先执行扩容，因为扩容也可以缩小链表长度，扩容还是超过8个则再转换为红黑树

- 插入时是先插入然后再判断是否超过阈值，超过则扩容
- 1.7是先插入在扩容，1.8是先扩容在插入

### LinkedHashSet

- 知识点
  - 底层是一个哈希表（数组+链表 or 数组+红黑树+链表(记录元素的存取顺序))
  - 存储的有序的集合，但是也是不允许重复的

### LinkedHashMap

- 有序，存取元素顺序一致，可以存 null
- 底层是哈希表+双向链表（存储顺序）
- 该双向链表的迭代顺序就是`entry`的插入顺序
- 非线程安全的，想要安全可以 `Collections.synchronizedMap()` 

### HashTable

- 知识点
  - 底层是哈希表，单线程，安全，同步，速度慢
  - 初始容量是11，每次扩容是：容量 × 2 + 1
  - 容量是质数，不需要二次hash就要就好的散列分布
  - 不能存null键，null值，空指针异常
  - Hashtable和vector集合在jdk1.2之后被（HashMap,ArrayList）取代了但Hashtable的子类Properties依然活跃
  - Properties集合是唯一一个和IO流相结合的集合


### TreeMap（红黑树）

#### 知识点

- 是一个双列集合，底层由红黑树构成
- 键不能重复，重复会覆盖
- 元素会按从大到小排列
- 不是线程安全的，可以通过 `Collections.synchronizedSortedMap(new TreeMap(...))` 使其线程安全

### WeakHashMap

#### 知识点

- key是弱引用，gc时回收

## 泛型

### 泛型类/接口

- `class<T>`
  - 类的内部可以使用类的泛型
  - 实例化没有指明类的泛型，默认为Object
  - 子类继承了指明泛型类型的父类（泛型类），实例化时不需要指定泛型类型
  - 静态方法中不能使用类的泛型
  - 异常中不能用泛型

### 泛型方法

- public \<T\> T method(T t){};

## 反射（Reflect）

### 含义

- 将类的各个组成部分封装为其他对象，这就是反射机制

### 内部原理

### 功能

1. #### 获取Class对象

   * Class.forName("全类名")		
     * 将字节码文件加载进内存，返回Class对象

     * 多用于配置文件

   * 类名.class                                
     * 通过类名获取

     * 多用于参数的传递

   * 对象.getClass()
     * getClass()方法在Object中定义的
     * 多用于对象的获取字节码的方式

2. #### 获取成员

   * 获取成员变量
     * Fleld[] getFields()											
       * 获取public修饰的成员变量
     * field getField(String name)                    
       * 获取指定名称的public修饰的成员变量
     * Field[] getDecLaredFields()                   
       * 获取所有的成员变量
     * Field getDeclaredField(String name)     
       * 获取指定名称的成员变量
   * 获取构造方法
     * Constructor\<T>[] getConstructors(）
     * Constructor\<T> getConstructor(类<?>...  pFarameterTypes)
     * Constructor\<T>[] getDeclaredConstructors()
     * Constructor\<T>[] getDeclaredConstructors(类<?>...  parameterTypes)
   
   * 获取成员方法
     * Method[] getMethods()
     * Method getMethods(String name,类<?>...  parameterTypes)
     * Method[] getDeclaredMethods()
     * Method getDeclaredMethods(String name,类<?>...  parameterTypes)
   * 获取类名
     * String getName()

### 成员

 * #### Field：成员变量

   - void set(Object obj,Object value)
     - 设置值
   - get(Object obj)
     - 获取值
   
* #### Constructor：构造方法

  - T newInstance(Object... initargs)
    - 创建对象

* #### Method：成员方法

  - Object invoke(Object obj,)
    - 执行方法
- String getName()
  - 获取方法名	


### 注意

1. 获取访问受限的成员时需要使用暴力反射
   - setAccessible(true)		

   - 忽略安全检查

2. 创建无参构造可以使用Class对象的方法
   - newInstance()				

   - 创建无参构造方法

3. 加载配置文件，配置文件以properties结尾
   - classLoader.getResourceAsStream("pro.Properties");读取配置文件，
   - 返回InputStream流
   - 通过类名.class.getClassLoader()获取ClassLoader对象


## 枚举（Enum）

### 要求：

* 类的对象的数量有限，确定的
* 需要定义一组常量时最好使用枚举类
* 如果枚举类中只有一个对象，则可以作为单例模式的实现方式

### 定义：

1. 5.0之前的自定义
   * private final 成员变量
   * private 构造方法
   * public static final 创建对象
2. 使用enum关键字定义
   * 需要将对象定义在开头，对象之间用逗号分隔，最后用分号结束

### 常用方法

```java
values()					   返回枚举类型的对象数组
valueOf(String str)            可以把字符串转为对应的枚举对象
toString()                     返回当前枚举类对象常量的名称
```

## 注解（Annotation）	

### 定义

​	也叫元数据，出现在1.5之后的新特性，

### 作用

  1. 编写文档
    - javadoc   
    - java提供的工具，可以将java文件抽取为doc文档
    - java文件需要以ANSI格式编码才不会乱码

  2. 代码分析
  3. 编译检查

### JDK中预定义的注解

   * @Override
     * 检测该注解标注的方法是否覆盖重写父类方法

   * @Deprecated
     * 该注解标注的内容表示以过时

   * @suppressWarnings
     * 压制警告


### 自定义注解

- 本质上就是一个接口，可以定义成员方法，成员方法的返回值有要求

  - 要求：基本数据类型,String,枚举,注解,以上类型的数组，使用时需要赋值
- 例如：
    - int value();//如果方法只有一个且名字为value，那么使用时可以直接写值  
  - 使用时：@MyAnno(10)
    - String name() **default** "张三"//有默认值，使用时可以不赋值
  - enumTest emum1();//枚举类型
    - 使用时 emum1 = enumTest.P1
  - MyAnno2 anno2();//注解类型
    - 使用时anno2 = @MyAnno2
  - String strs[]//字符数组 			   

- 格式: public @interface 注解名称{}

### 元注解

- 作用
  - 用于描述注解
- 常用
  - @Target						描述注解能够作用的位置
    - ElementType.TYPE     	       该注解只可作用于类上
    - ElementType.METHOD     	该注解只可作用于类上
    - ElementType.FIELD     	     该注解只可作用于类上
  - @Retention              描述注解被保留的阶段
    - RetentionPolicy.SOURCE          该注解保留到资源时状态
    - RetentionPolicy.CLASS             该注解保留到class时状态
    - RetentionPolicy.RUNTIME        该注解保留到运行时状态，能被JVM读取
  - @Documented         描述注解是否被抽取到api文档
  - @Inherited               描述注解是否被子类继承

### 使用注解的步骤

1. 获取注解定义的位置的class对象
2. Class.**getAnnotation**()							//获取注解的实现类的对象
3. 调用注解中的方法

### 其他

* javap 								java提供的反编译工具
