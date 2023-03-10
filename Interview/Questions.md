---
title: 常见面试题
toc: content
keywords: [interview]
---

##    JavaSe

### == 和 equal 的区别

- == 比较的是地址，equal比较的是值
- == 比较快，== 是运算符，equal是Object 的方法
- equal 源码中等同于 == ，但是应用类型重写了equals方法
  - 重写需要满足 自反性、对称性、传递性、一致性、非空性

### HashTable、HashMap、ConcurrentHashMap的区别

|            | HashMap         | HashTable            | ConcurrentHashMap         |
| ---------- | --------------- | -------------------- | ------------------------- |
| 线程安全？ | 不安全          | 安全                 | 安全                      |
|            | 无contains()    | 有contains()         | 有，等价为containsValue() |
|            | 可null          | 不可null             | 不可null                  |
|            | 二次hash        | 一次hash             | 二次hash                  |
|            | 继承AbstractMap | 继承Dictionary       | 继承AbstractMap           |
|            | 初始16          | 初始11               | 初始16                    |
|            | 扩容 X2         | 扩容 X2 + 1          | 扩容 X2                   |
| 创建时机   | put时创建       | 构造时创建           | put时创建                 |
| 并发度     | 无              | 低（synchronized锁） | 高（分段锁）              |

### 创建对象的五种方法

- 使用new关键字
- 使用Class.newInstance
- 使用Constructor.newInstance
- 使用Clone方法
  - 对象必须实现Cloneable接口，重写clone方法，调用父类的clone方法
  - 在Object中这个方法是protected的，重写才能调用
- 使用反序列化
  - 对象需要实现Serializable接口

### 抽象类和接口的区别

- 抽象类
  - 自底向上，对类的抽象，是一种模板设计
  - 有抽象方法的一定是抽象类，可以写具体方法，构造方法
  - public、protected 和 default 这些修饰符
- 接口
  - 自顶向下，对行为的抽象，是一种行为规范
  - 只能有抽象方法，1.8之后可以有具体方法
  - 只能由static final 变量
  - 默认修饰符是 public


### Double类型进行运算时有什么问题，怎么解决

- 除数为0时会出现 NAN、definite 等等
- 使用BigDecimal解决，将double类型转字符串，传入BigDecimal的构造函数，创建对象
- 就可以进行加减乘除了

### 多态的底层原理

- 多态的底层实现是**动态绑定**，即在运行时才把方法调用与方法实现关联起来

### HashMap、LinkedHashMap、TreeMap使用场景

- 涉及到键值的存储优先考虑HashMap，如果需要根据key的顺序取存储键值对则使用TreeMap就更合适一点
  - 因为TreeMap底层是基于红黑树实现的，可以提供顺序访问，但是TreeMap的Key不能为null，而且时间复杂度是O(LogN)

- 对于LinkedHashMap，它的实现是通过为键值对维护一个双向链表
  - 构建一个看重空间占用的资源池时可以选择使用LinkedHashMap，可以实现自动的将不常用的资源释放，例如模拟LRU缓存淘汰策略

### 哪些方法可以减少Hash冲突

- 拉链法
  - 拼接成链表

- 开放寻址法
  - 寻找下一个空闲位置

- 多哈希算法
- 一致性Hash算法

### Fail-Fast和Fail-Safe

- Fail-Fast：一旦发现遍历时有修改，则立刻抛异常，例如ArrayList，vector
  - ArrayList内部的modCount记录了修改次数，当遍历时会获取modCount
  - 遍历过程会比较当前modCount和获取时的值，不一致抛异常
- Fail-Safe：遍历的同时有修改，会有相应的策略，例如CopyOnWriteArrayList

### new String(“aa”)创建了多少个对象

- 一个或两个
  - 如果字符串常量池中有aa则在堆中创建一个引用指向常量池中的aa
  - 如果字符串常量池中没有aa则现在常量池中创建aa，然后在堆中创建一个引用指向常量池中的aa

### final原理

- 分为写final域和读final域
  - 写final域
    - 会要求编译器在写final域之后，插入一个StoreStore屏障
  - 读final域
    - 会要求编译器在读final域之前，插入一个LoadLoad屏障
- 具体处理器具体分析，x86处理器由于有默认的重排序规则，所以不需要屏障

### String、StringBuffer、StriBuilder区别

- String
  - 是final修饰的，不可变
- StringBuilder

  - 迭代（循环）首选

  - 线程不安全的，但是速度快


- StringBuffer

  - 并发首选


    - 线程安全的，源码中有Synchronized

## Spring 

### 单例bean是线程安全的吗

- 不是，spring没有对单例bean进行多线程的封装处理
- 大部分时候bean都是无状态的，某种程度上是安全的
- 有状态的话就需要开发者去保证线程安全了
- 作用域改为prototype就可以保证线程安全了

### Bean初始化和销毁的顺序（先    - ->  后）

- 初始化
  - @PostConstructc  ---->  实现InitializingBean并重写afterPropertiesSet()方法  ---->  @Bean(initMethod = “方法名”)
- 销毁
  - @PreDestroy  ---->  实现DisposableBean并重写destroy()方法  ---->  @Bean(destroyMethod = “方法名”)

### @Autowired失效

- 创建BeanFactoryPostProcessor在创建对象之后则会失效

### Scope失效

- 单例对象中注入多例对象，多例会失效
  - 可以添加@Lazy解决（原理是代理）
  - 可以在@Scope(value = “prototype”, proxyMode = ScopedProxyMode.TARGET_CLASS) 
  - 使用ObjectFactory<多例对象>解决
  - 使用容器对象的getBean方法获取多例对象

### spring如何处理线程并发问题

![image-20220406164345074](..//pic//image-20220406164345074.png)

### @Component、Controller、Service、Repository的区别 

![image-20220406164522780](..//pic//image-20220406164522780.png)

### @Autowired和@Resource的区别

![image-20220406164653753](..//pic//image-20220406164653753.png)

### 怎么解决循环依赖

- A依赖B，B依赖A
- 在初始化中填充属性时发生循环依赖问题
- 三级缓存，提前暴露对象，aop 
  - 一级缓存：singletonObjects
    - 存成品对象
  - 二级缓存：earlySingletonObjects
    - 存半成品对象
  - 三级缓存：singletonFactories
    - 存lambda表达式，value值是ObjectFactory


### 如果只有一级缓存，能解决循环依赖吗

- 可以，即半成平对象和成品对象都放到一个map中，需要在value上打标签，每次获取对象时要取出value判断，很麻烦

### 如果只有一级和二级缓存，能解决循环依赖吗

- 可以，前提是整个循环引用过程中没有使用到aop

### 为什么使用aop之后就需要使用三级缓存

- 因为在同一时刻会同时存在原始对象和代理对象，而属性在赋值调用和对外暴露的时候，没办法确定是原始对象还是代理对象，所以我需要在第一次对外暴露的时候，通过一个lambda表达式。类似于回调机制的东西，来把最终版本对象返回回去

### 缓存的放置时间和删除时间

- 三级缓存：createBeanInstances之后放置
- 二级缓存：第一次从三级缓存中确定对象是代理对象还是普通对象的时候放置，同时删除三级缓存
- 一级缓存：生成完整对象之后放到一级缓存，删除二三级缓存

### BeanFactory 和 ApplicationContext的区别

- 都是接口，用于创建容器
- BeanFactory
  - 实现类实现了很多功能
    - DefaultListableBeanFactory
  - 延迟加载Bean对象，spring内部使用的多
- ApplicationContext
  - 内部间接调用Benfactory，扩展了国际化、匹配资源、发布事件、环境信息等功能
  - 实现类
    - ClassPathXmlApplicationContext：
    - FIleSystemXmlApplicationContext
    - AnnotationConfigApplicationContext
    - AnnotationConfigServletWebServerApplicationContext
  - 一开始就创建好Bean对象，效率高，体验好

### BeanFactory 和 FactoryBean有什么区别

- ![image-20220316172333995](..//pic//image-20220316172333995.png)

### Spring中用到的设计模式

- 单例模式：单例Bean并不是实现了单例模式
- 原型模式：bean指定为prototype
- 工厂模式：BeanFactory
- 工厂方法模式：FactoryBean
- 模板模式：postProcessBeanFactory、onRefresh、initPropertyValue
- 策略模式：XmlBeanDefinitionReader、PropertiesBeanDefinitionReader
- 观察者模式：listner、event、multicast
- 适配器模式：HandlerAdapter
- 装饰器模式：BeanWrapper
- 责任链模式：aop会生成拦截器链
- 代理模式：aop动态代理
- 建造器模式：Builder结尾的对象（BeanDefinitionBuilder）
- 组合模式： 

### Spring隔离级别

- 五种，第一种是和数据库一致，其余四种与数据库的隔离级别一样

### 事务失效的场景和原因

- 事务的原理是AOP，失效就是AOP没起作用
  - 发生自调用，在类里使用了this调用类中方法（this一般是省略的），因为此时this指向的不是代理类，是被代理对象本身
- @Transaction只能作用与public修饰的方法
  - 如果一定要在非public上用事务，就要用AspectJ

- 数据库不支持事务
- 没有被Spring管理
- 出现检测异常导致事务没有回滚

### 为什么JDK动态代理必须是接口

- 因为Java是单继承的，生成的代理对象会继承一个Proxy类，所以必须实现一个接口

### JDK和CGLIB的区别

- JDK代理和被代理对象同级，兄弟关系
- CGLIB代理和被代理对象是父子关系，所以被代理对象不能是final修饰的
- JDK代理前16次通过反射调用方法，后续都是直接调用
- CGLIB可以通过方法代理直接调用被代理方法



### 什么是Spring Profiles

![image-20220406165205311](..//pic//image-20220406165205311.png)

## SpringMVC

<a href="./SpringMVC.md#SpringMVC工作流程">SpringMVC工作流程</a>  

### SpringMVC常用的注解

- @RequestMapping 建立请求路径和控制器方法之间的关系
- @rest 
  - @RequestBody    处理请求体中的json数据转换为java对象
  - @ResponseBody    把控制器返回的java对象转化为json数据
  - @ResponseStatus    控制响应的状态码
  - @RestController    = @Controller + @ResponseBody    
- 统一处理
  - @ControllerAdvice  
  - @ExceptionHandler
  - @RestControllerAdvice
- 参数
  - @RequestHeader    获取请求头中的参数信息
  - @CookieValue    获取cookie的值
  - @PathVariable    获取请求路径中的参数值
  - @RequestParam    获取请求参数中的参数值（？后面的值）
- Ajax
  - @CrossOrigin    解决ajax跨域问题，往响应头上加特殊的头

### SpringMVC如何解决GET / POST请求中文乱码？

- 使用过滤器

## SpringBoot

### 介绍一下SpringBoot，优点

- SpringBoot是基于Spring创建的一个上层应用框架，简化了配置，效率高
- 有内置的tomcat，可以独立运行
- 约定大于配置理念，最大的简化配置
- 自动配置，整合第三方服务方便

### SpringBoot的核心注解

- @SpringBootApplication：是一个组合注解，包含了7个注解
  - 4个元注解
  - @ComponentScan：扫描当前包及其子包下@Component修饰的java类
  - @SpringBootConfiguration：内部组合了@Configuration
  - @EnableAutoConfiguration：打开自动装配

### SpringBoot如何解决跨域问题

![image-20220406165240221](..//pic//image-20220406165240221.png)

## Mybatis

### 当实体类中的属性名和表中的字段名不一样怎么办？

- 写sql语句时起别名

- 在Mybatis全局配置文件中开启驼峰命名规则

  - ```xml
    <settings>
    	<setting 
            name="mapUnderscoreToCamelCase" 
            value="true"/>   
    </settings>
    ```
  
- 在Mapper映射文件中使用resultMap自定义映射规则

### #{}和${}的区别

- ${}是properties文件种的变量占位符，可以用于标签属性值和sql内部，属于静态文本替换
- {}是sql的参数占位符，mybatis会将sql中的#{}替换为？，在sql执行时会用PreparedStatement按kottt';oi
- '''''                                                     lo序给sql中的？设置参数值，可以有效的防止sql注入

## JVM

### 快速跳转

<a href="./JVM.md#类加载过程">对象的创建过程(类加载过程)</a> 

### 触发老年代GC的时机

- 老年代可用内存小于新生代全部对象的大小，如果没开启空间担保参数，会直接触发Full GC，所以一般空间担保参数都会打开

- 老年代可用内存小于历次新生代GC后进入老年代的平均对象大小，此时会提前Full GC

- 新生代Minor GC后的存活对象大于Survivor，那么就会进入老年代，此时老年代内存不足full gc

- 就是“-XX:CMSInitiatingOccupancyFaction”参数

  如果老年代可用内存大于历次新生代GC后进入老年代的对象平均大小，但是老年代已经使用的内存空间超过了这个参数指定的比例，也会自动触发Full GC。

### 对象直接进入老年代的情况

- 大对象直接进入老年代
  - 需要设置-XX:PretenureSizeThreshold，只在Serial和ParNew收集器下有效
  - 单位是字节

- 对象动态年龄判断
  - 当Survivor区内的对象总大小超过50%（可以设置），那么这些对象中年龄最大的都会直接进入老年代，对象动态年龄判断一般在minor gc后触发

### new Object() 在内存中占用多少字节

- 16字节

- 总的来说会是8的次方字节，默认是开启了指针压缩

- 普通对象
  - (8 + 4) + 对象成员变量大小 + 对齐区
- 数组对象
  - (8 + 4 + 4) + 对象成员变量大小 + 对齐区 

### 为什么hotspot不使用c++对象代表java对象

### Class对象是在堆还是在方法区

### Class.forName

## 多线程（JUC）

### Synchronized

|      | 自旋锁                                                       | 锁消除                           | 锁粗化                       | 偏向锁                                    |
| ---- | ------------------------------------------------------------ | -------------------------------- | ---------------------------- | ----------------------------------------- |
| 简介 | 循环获取锁<br />一定次数<br />得不到锁就挂起<br />有自适应的自旋锁 | JIT判断是否消除<br />不共享<br / | 多个细粒度的锁升级为一个大锁 | 同线程不用CAS判断<br />默认开启，延迟生效 |
|      |                                                              |                                  |                              |                                           |
|      |                                                              |                                  |                              |                                           |

|      | 轻量级锁                           | 重量级锁 |      |      |      |
| ---- | ---------------------------------- | -------- | ---- | ---- | ---- |
| 简介 | 多线程交错访问<br />优化为轻量级锁 |          |      |      |      |
|      |                                    |          |      |      |      |
|      |                                    |          |      |      |      |



### 成员变量和静态变量是否是线程安全的？

- 没有共享则安全
- 如果共享了
  - 如果只有读操作，则线程安全
  - 如果有写操作，则需要考虑线程安全

### 局部变量是否线程安全？

- 局部变量是线程安全的
- 但是引用局部变量的对象则有可能出现线程安全问题
  - 如果该引用对象没有逃离方法的作用域，则线程安全
  - 逃离了则需要保证线程安全

### 多线程解决什么问题？本质是什么？

- 由于cpu、缓存、io设备之间的速度有极大的差异，为了合理使用cpu的高性能，平衡三者的差异，主要体现在
  - cpu增加了缓存，均衡cpu与内存（导致可见性问题）
  - 操作系统增加了进程线程，**分时复用**，均衡cpu与IO（导致原子性问题）
  - 编译程序优化指令执行次序，**重排序**（导致有序性问题）

### run()执行线程和start()执行线程区别

- 如果直接调用线程对象的run()方法，系统把线程对象当成一个普通对象，而run()方法也是一个普通方法，而不是线程执行体

### sleep 和 wait 的区别

- 都是让当前线程暂时放弃CPU使用权进入阻塞状态
- 都可以通过打断唤醒，会抛异常
- 方法归属不同
  - sleep是Thread的静态方法
  - wait是每个对象都有的方法
- 锁特性不同
  - wait需要先获取对象锁
  - 执行wait后会释放对象锁
  - sleep如果在Synchronize中执行，不会释放对象锁

### Lock和Synchronized区别

- Synchronized是关键字，用C++实现的
  - Lock是接口，Lock需要手动释放，两者都是可重入的
- Lock提供了获取等待状态、可公平锁、可打断、可超时、多条件变量（多等待队列）
  - Synchronized是非公平锁
- 在没有竞争时，Synchronized做了很多优化，如偏向锁，轻量级锁，性能不错
  - 竞争激烈时，Lock通常会提供更好的性能

### Volatile能否保证线程安全

### 什么是多线程的上下文切换

### 什么是线程池，为什么要用线程池

### 无界队列有什么坏处

### Volatile

1. **说一下Volatile**
   - 保证了共享变量的可见性和有序性

2. **Volatile的可见性和禁止指令重排是怎么实现的**
   - volatile修饰的属性每次线程使用时都会从内存中读取，不会加载进cpu缓存
   - 禁止指令重排是加了写屏障

### HashTable

### <a href="./JUC.md#ConcurrentHashMap">ConcurrentHashMap</a> 

1. **Segment片段和HashEntry数组的大小？**
   - 1.7
     - segment数组默认大小16，这个容量初始化后就不能改变了，不是懒惰初始化
     - 并发度是ConcurrentHashmap的长度（segment数组的大小）
     - 每个片段下的数组大小 = 容量 / 并发度
       - 最小值为2，数组的元素数量超过阈值（0.75）就会扩容
   - 1.8
     - 没有segments数组
     - HashEntry数组的大小
       - 无参构造创建时，数组初始容量是16
       - 初始化时传了容量大小时，判断是否超过容量×负载因子
         - 大于则创建下一个阶段的大小
2. **索引是怎么计算的？**
   - 1.7
     - segment数组索引
       - 并发度为16（2^4^）时，看二次hash值的高四位
       - 并发度为32（2^5^）时，看二次hash值的高五位

     - 数组索引
       - 数组的大小为2（2^1^）时，看二次hash的低一位
       - 数组的大小为4（2^2^）时，看二次hash的低二位

### ThreadLocal

1. **ThreadLocal和Synchronized的区别**
   - 思路不同
     - Synchronized 多个线程访问共享资源
     - ThreadLocal 多个线程使用独自的共享资源（副本）
2. **索引的计算**
   - 累加1640531527
3. **ThreadLocalMap中的key为什么要设置为弱引用？**
   - 源码中entry继承了WeakReference 但是只有key设置了弱引用
   - 当线程长时间运行时会占用内存，gc时会将key释放掉
   - **释放值的内存** 
     - 当执行get时遇到key为null的，会将value置为null，当前ThreadLocal为key插入
     - 当执行set时遇到key为null时，会将key-value替换掉，并删除邻近一定范围的key为null的键值对（**启发式扫描**），删除的范围和map中元素个数有关
     - 推荐使用remove来释放值的内存，因为一般开发中会将ThreadLocal定义为静态final变量，为强引用，所以gc不会清理key

## 计算机网络

### 快速跳转

#### <a href="./计算机网络.md#TCP/UDP">TCP/UDP</a> <a href="./计算机网络.md#状态码">状态码</a> <a href="./计算机网络.md#三次握手">三次握手</a> <a href="./计算机网络.md#四次挥手">四次挥手</a> 

### ping命令的底层是什么协议

- ICMP协议

### http的请求响应

### http1.0和1.1的区别

### http1.1建立连接后，服务端怎么区分发过来的请求

### Session的4、7层负载均衡

### 浏览器接收到一个url到最后展示页面经历的过程

- DNS解析：将域名解析为IP地址
- TCP连接：TCP <a href="./计算机网络.md#TCP三次握手">三次握手</a> 
- 发送HTTP请求
- 服务器处理请求并返回HTTP报文
- 浏览器解析渲染页面
- 断开连接：TCP <a href="./计算机网络.md#TCP四次挥手">四次挥手</a> 

### http和https的区别

- http是明文传输，没有状态，https是加密的安全传输，有身份认证
- http端口是80，https是443

### https加密过程

- HTTPS协议是由SSL/TLS+HTTP协议构建的

- 采用了非对称加密+对称加密
- 服务端将公钥A发送给客户端，客户端生成一个密钥X，用公钥A加密后发送给服务端，服务端用密钥A解锁后获得密钥X，这样双方就都拥有密钥X了，且别人无法知道它。之后双方所有数据都通过密钥X加密解密即可

### time-wait的作用

- time-wait开始的时间为tcp四次挥手中主动关闭连接方发送完最后一次挥手，也就是ACK=1的信号结束后，主动关闭连接方所处的状态。

  然后time-wait的的持续时间为2MSL. MSL是Maximum Segment Lifetime,译为“报文最大生存时间”，可为30s，1min或2min。2msl就是2倍的这个时间。一般根据实际的网络情况进行确定。

- **为什么要持续这么长的时间呢？**

  - 为了保证客户端发送的最后一个ack报文段能够到达服务器。因为这最后一个ack确认包可能会丢失，然后服务器就会超时重传第三次挥手的fin信息报，然后客户端再重传一次第四次挥手的ack报文。如果没有这2msl，客户端发送完最后一个ack数据报后直接关闭连接，那么就接收不到服务器超时重传的fin信息报，那么服务器就不能按正常步骤进入close状态。那么就会耗费服务器的资源

  - 在第四次挥手后，经过2msl的时间足以让本次连接产生的所有报文段都从网络中消失，这样下一次新的连接中就肯定不会出现旧连接的报文段了。

### DNS解析错误，怎么排查

- 使用nslookup命令
- ping命令

## 操作系统

### 快速跳转

<a href="./操作系统.md#拥塞控制">拥塞控制</a>  <a href="./操作系统.md#系统中断">系统中断</a>  <a href="./操作系统.md#系统调度">系统调度</a>  <a href="./操作系统.md#进程、线程、协程">进程、线程、协程 </a> 

### Java的线程和操作系统的线程有区别吗？

- java线程分为守护线程和用户线程
- 在window上线程是一样的，在Unix上采用Pthread实现
- jdk1.2之前是jvm自己有一套线程管理机制，即和操作系统上的不一样
- jdk1.2之后采用了操作系统原生的内核级线程，将线程的调度交给了操作系统内核
- 总的来说，java的线程就是操作系统的线程，Java线程与操作系统线程一对一映射，依赖于操作系统的具体实现

### 线程切换要保存哪些上下文？

- 当前线程Id、线程状态、堆栈和寄存器状态等信息
- 寄存器主要包括SP PC EAX等寄存器，其主要功能如下：
  - SP:堆栈指针，指向当前栈的栈顶地址
  - PC:程序计数器，存储下一条将要执行的指令
  - EAX:累加寄存器，用于加法乘法的缺省寄存器

### 寄存器有哪些？

- 累加寄存器、基址寄存器、计数寄存器、数据寄存器、变址寄存器、指针寄存器
- 段寄存器
- 指令指针寄存器
- 标志寄存器

### 线程和进程的区别

- 线程属于进程，可以有多线程
- 线程挂了，进程也挂，其他进程不受影响
- 进程是系统资源调度的最小单位，线程是CPU调度的最小单位
- 进程开销比线程开销大
- 进程在执行时拥有独立的内存单元，多个线程共享进程的内存
- 进程切换时需要刷新TLB并获取新的地址空间，然后切换硬件上下文和内核栈，线程切换时只需要切换硬件上下文和内核栈。
- 通信方式不一样

### 线程与协程的区别

- 协程执行效率极高。协程直接操作栈基本没有内核切换的开销，所以上下文的切换非常快，切换开销比线程更小。
- 协程不需要多线程的锁机制，因为多个协程从属于一个线程，不存在同时写变量冲突，效率比线程高。
- 一个线程可以有多个协程。

### 僵尸进程和孤儿进程

- 

### cpu的调度算法有哪些

### 优先级调度算法的具体实现方式

### 如何控制多线程优先级的顺序

### 用户态和内核态的区别

### 什么是虚拟内存	 

### 虚拟存储是怎么实现的

### 32位操作系统的最大虚拟内存空间是多少

### 分页和分段的区别

## <a href="./BIO和NIO">BIO/NIO</a> 

1. **BIO有什么缺陷**
   - accept获取socket和读写会阻塞当前线程，如果1w个客户端就需要服务端1w个线程去支持，线程频繁上下文切换增加cpu负载


3. **针对C10K这样的需求，NIO靠什么解决**
   - 选择器selector可以做到一个线程监听多个socket
4. **多路复用操作系统函数select()的工作原理**

   - 准备需要监听的socket集合，用一个rset保存socketId

   - 调用select函数会涉及到用户态和内核态的切换

   - 还需要复制rset集合到内核态

   - 循环监听rset中记录的socket，有就绪的就将其对应的rset位置位

   - 然后把内核态的rset复制到用户态中去
5. **select()默认监听socket的数量为什么是1024**
   - 实际监听小于1024，是一个位图
6. **select()第一遍O(n)未发现就绪的socket，如果后续某一个socket就绪后。select是如何感知的？是不停的轮询吗？**

   - 第一遍未发现就绪的socket后，会将当前线程保留到每个需要监听的socket的等待队列中

   - 然后当前线程从工作队列中移除

   - 这时客户端向服务端发送了数据，数据通过网线到网卡，网卡的DMA将数据写进内存，数据传输结束后会触发网络传输完毕中断程序，cpu会将当前正在运行的线程挂起然后执行这个中断程序的逻辑

   - 根据内存中的数据包分析出是哪个socket的数据，而且tcp/ip协议传输的数据包是有端口号的，这样就能找到对应的socket实例

   - 然后数据导入到socket的读缓冲区，然后会检查socket的等待队列，有等待的队列就将其加入cpu工作队列

   - 然后select函数又会执行了，就能发现就绪的socket了
7. **poll()和select()主要区别是什么？**

   - 参数不一样

   - select用的是bitmap，长度是1024

   - poll使用的数组，长度很大
     - 数组中的结构是pollfd，里面是fd、events、revents
8. **为什么会有epoll这个技术，产生的背景是？**
   - 为了解决select和poll的缺陷
     - 用户态和内核态频繁切换
     - 不能反映是哪个socket就绪了，每次需要遍历判断
9. **epoll的工作原理是？**
   - epoll_create 在内核态创建一个eventpoll对象
10. **eventpoll对象的就绪列表数据是如何维护的呢？**

    - 需要先描述一遍将socket加入epoll对象监听列表的过程

    - 当有socket进入就绪队列后，会检查epoll对象的等待队列

    - 如果等待队列中有进程，则将其加入cpu工作队列
11. **epoll怎么得知哪些socket是就绪的？**
    - epoll_wait中会传入一个epoll_events 并将其置为1，然后拷贝到数组指针中，然后将这个数组返回给用户空间
12. **epoll_wait可以设置为非阻塞吗？**
    - 默认是阻塞的
    - 调用时可以传入参数来设置
      - 设置为0就是非阻塞的
13. **eventpoll对象中存放需要检查的socket信息是采用的什么数据结构？为什么？**
    - 红黑树，因为经常有增删改，稳定的O(logN)
14. **epoll往监听事件列表中添加一个新事件过程？**

    - 内核程序会把当前eventpoll对象追加到socket的等待队列中

    - socket接收完客户端发送的数据后触发中断程序

    - 根据内存中的数据找到对应的socket的读缓冲区，发现等待队列中的是一个eventpoll对象引用

    - 然后会将当前socket的引用加入到epoll对象的监听列表末尾

## 数据结构和算法

### 如何判断一棵树是线索二叉树

- 前序遍历如果是升序的则是线索二叉树

### 如何判断一棵树是完全二叉树

- 层序遍历
  - 当出现有节点有右孩子没有左孩子，false
  - 当出现有节点有左无右，其后续必须都是叶子节点，否则false

### 如何判断一棵树是满二叉树

### 如何判断一棵树是平衡二叉树

### 二叉树、红黑树、AVL树有什么区别

### 洗牌算法

- 将最后一个数和前面任意 n-1 个数中的一个数进行交换（也可以不换），然后倒数第二个数和前面任意 n-2 个数中的一个数进行交换，如此往复直到最后一个元素，就完成了洗牌，该算法保证了每个元素在每个位置的概率都是相等的。

- ```java
  for (int i = n - 1; i >= 0; i--) {
      //每次随机出0-i之间的下标
      swap(arr[i], arr[rand() % (i + 1)]);    
  }
  ```

- 时间复杂度：O(N)

### 快排归并，区别，手撕时间复杂度推导 

## Mysql

### 快速跳转

#### <a href="./MySql.md#索引">索引</a> 

#### <a href="./MySql.md#隔离级别">事务隔离级别</a> 

#### <a href="./MySql.md#存储引擎">存储引擎</a> 

#### ACID

#### MVCC

#### 性能优化

#### 存储过程

#### 备份

### 查询用户最近登录的时间

- `select userId, max(add_time) from test group by userId;`
- `SELECT * from (SELECT * from test ORDER BY add_time)  GROUP BY userId`

### InnoDB和MyISAM的区别 ？

- InnoDB支持事务、外键、行级锁，存储对事务，数据完整性要求较高的核心数据，擅长事务处理，具有奔溃恢复特性
- MyISAM存储非核心事务

### InnoDB如何解决幻读问题？

- 引入了间隙锁和临键锁
  - 临键锁机制相当于间隙锁和行锁锁，是mysql默认的行锁算法

### 一条sql的执行过程

- 首先客户端发起sql查询
- sql到达服务器先经过连接器，进行身份、权限等校验
- 然后再经过分析器，进行词法分析和语法分析
  - mysql8.0之前会先去查询缓存中找
- 在经过优化器，以优化器认为最优的执行方案去执行
- 然后交给执行器，会先进行权限校验，权限通过就会去调用存储引擎的接口，进行具体的执行

### B+tree的能存放的数据有多少？

- 假设一行数据的大小为1k，一个结点可以存放16行这样的数据，InnoDB的指针占6字节，主键假设为bigint，占用8字节
- 高度2：即一个主键，第二层都是叶子结点
  - n * 8 + (n + 1) * 6 = 16 * 1024 
  - n = 1170
  - 1171 * 16 = 18736
- 高度3：
  - 1171 * 1171 * 16 = 21939856

### 查询计划

### 定位慢查询

### B数和B+树的区别

- b数：叶子结点和非叶子结点都保存数据，这样导致一页中存储的键值减少，指针也减少，树的层级更高，性能就低了
- B+树：

### redo log 两阶段提交过程

### MySQL三种log日志以及作用

- binlog
  - 记录了所有DDL、DML语句，不会记录读业务
- redo log
  - 
- undo log

### 开启事务的完整过程

### 数据库三范式是什么？

- 第一范式：属性不可再分割
- 第二范式：有主键，而且其他字段要依赖主键
- 第三范式：消除传递依赖

### 悲观锁和乐观锁

- 悲观锁：比较适合写入操作比较频繁的场景，如果出现大量的读取操作，每次读取的时候都会进行加锁，这样会增加大量的锁的开销，降低了系统的吞吐量。

- 乐观锁：比较适合读取操作比较频繁的场景，如果出现大量的写入操作，数据发生冲突的可能性就会增大，为了保证数据的一致性，应用层需要不断的重新获取数据，这样会增加大量的查询操作，降低了系统的吞吐量。

## Redis

### 快速跳转

<a href="./Redis.md#内存淘汰策略">淘汰策略</a> <a href="./Redis.md#雪崩、穿透、击穿">雪崩、穿透、击穿</a> 

### 说说redis两种<a href="./Redis.md#持久化">持久化</a>的方式和特点 

|                | RDB                                      | AOF                                     |
| -------------- | ---------------------------------------- | --------------------------------------- |
| 持久化方式     | 定时对整个内存做快照                     | 记录每一次执行的命令                    |
| 数据完整性     | 不完整，俩次备份之间会丢失数据           | 相对完整，取决于刷盘策略                |
| 文件大小       | 有压缩，体积小                           | 有压缩，体积大                          |
| 宕机恢复速度   | 快                                       | 慢                                      |
| 数据恢复优先级 | 低，完整性不如aof                        | 高，数据更完整                          |
| 系统资源占用   | 高，大量cpu和内存消耗                    | 低，主要是磁盘io，但aof重写会有大量消耗 |
| 使用场景       | 能容忍一定的数据丢失，追求更快的启动速度 | 对数据完整性和安全性高的场景            |

### Redis为什么快

- 数据全部存储在内存中，操作效率高
- 单线程处理，避免了不必要的上下文切换
- IO多路复用技术，可以处理高并发的连接请求

## MQ

### 有什么用？使用场景？

- 优点

  - 异步：异步下单，提高系统的响应速度和吞吐量

  - 解耦：减少服务之间的影响，提高系统的稳定性和扩展性

  - 削锋：以稳定的系统资源应对突发的流量冲击

- 缺点

  - 系统可用性降低，MQ宕机，整个业务都会影响
  - 系统的复杂度提高
  - 一致性问题

### 如何设计一个MQ

- 首先需要设计一个先进先出的数据结构，高效，可扩展
- 扩展为分布式队列，分布式集群管理
- 基于Topic定制消息路由策略，发送者路由策略，消费者与对列的对应关系，消费者路由策略
- 实现高效的网络通信，Http
- 规划日志文件，实现文件高效读写，零拷贝，顺序写，服务重启快速恢复
- 死信队列，延迟队列、事务消息

### ActiveMQ、RabbitMQ、Kafka、RocketMQ的选择

|          | RabbitMQ                               | RocketMQ                         | Kafka                        |
| -------- | -------------------------------------- | -------------------------------- | ---------------------------- |
| 优点     | 消息可靠性高，功能全面，性能好，高并发 | 高吞吐、高性能、高可用、功能齐全 | 吞吐量大，性能好，集群高可用 |
| 缺点     | 消息堆积影响性能，erlang不好定制       | 生态不成熟、只支持Java           | 数据丢失，功能单一           |
| 吞吐量   | 万级                                   | 十万级                           | 百万级                       |
| 速度     | 微妙级                                 | 毫秒级                           | 毫秒级                       |
| 使用场景 | 大部分场景                             | 全场景                           | 日志分析，大数据采集         |

### 如何保证消息可靠传输

- 消息不能重复
  - 消费者端实现幂等性
  - 每个消息带一个有业务标识的ID，来进行幂等判断，或者使用全局唯一ID
  - RocketMQ可以给每个消息分配了一个MessaheID，但是不太好
- 消息不能丢失
  - 丢失的情况
    - 主从间信息同步时
      - 同步同步、两阶段提交（RocketMQ）
      - 镜像集群（RabbitMQ）
  
    - 生产者发送消息给消费者时
      - 消息发送+回调（kafka、RocketMQ、RabbitMQ）
      - 事务消息机制（RocketMQ）
      - 手动事务机制、Publisher Comfirm（类似RocketMQ事务消息机制）（RabbitMQ）
  
    - 消费者获取消息时
      - 保证消费者真正消费之后才删除消息，消费端ACK
      - 不采用异步消费即可
  
    - 持久化过程
      - 保证Broker确实收到并**持久化**消息，ACK机制	
      - 同步刷盘（RocketMQ）
      - 持久化队列、Quorum（RabbitMQ）
  

### 如何保证消息的顺序消费（RocketMQ）

- 乱序的原因

  - 一个topic中有多个队列，消息会因为负载均衡保存在不同的队列中
  - 多个消费者取消息时，无法保证消息顺序被消费

- 解决方法

  - 生产者

    - 保证一类消息都放在一个队列上

    - 在Producer发消息时注入了一个**消息队列选择器（MessageQueueSelector）**对象

  - 消费者
  
    - 消费者一次消费整个队列中的消息
  
    - Consumer需要注册消息监听，传入MessageQueueOrderly对象
  
    - ```java
      consumer.registerMessageListener(new MessageQueueOrderly(){
          @Overide
          public ConsumerOrderlyStatus comsumeMessage(List<MessageExt> msgs, ConsumerOrderlyContext context){
              
          }
      });
      ```


- RabbitMq：要保证目标exchange只对应一个队列，并且一个队列只对应一个消费者
- Kafka：生产者通过定制partition分配规则，将消息分配到同一个partition，Topic下只dui'y

### 消息积压

- 熔断隔离、灰度发布、优化线程池配置、消息转发
- 临时紧急扩容了，具体操作步骤和思路如下：

  - 先修复 consumer 的问题，确保其恢复消费速度，然后将现有 consumer 都停掉。
  - 新建一个 topic，partition 是原来的 10 倍，临时建立好原先 10 倍的 队列 数量。
  - 然后写一个临时的分发数据的 consumer 程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的 10 倍数量的队列中。
  - 接着临时征用 10 倍的机器来部署 consumer，每一批 consumer 消费一个临时 queue 的数据。这种做法相当于是临时将 queue 资源和 consumer 资源扩大 10 倍，以正常的 10 倍速度来消费数据。
  - 等快速消费完积压数据之后，得恢复原先部署的架构，重新用原先的 consumer 机器来消费消息

## ES

## ZK

## Dubbo

### Dubbo和SpringCloud的区别

- Dubbo主要解决的是服务调用的问题，SpringCloud主要是一个大而全的框架
- Dubbo的服务调用性能比SpringCloud高

## 分布式

### 快速跳转

分布式事务

### RPC调用和传统HTTP调用的区别

## Nginx

### 负载均衡策略

- 轮询（默认）
  - 每个请求按时间顺序分配到不同的服务器，如果服务器down会自动剔除
- 权重（weight）
  - 默认是1，权重越高被分配的客户端越多
- 最少连接数（Least）
  - 传入的请求是根据每台服务器当前所打开的连接数来分配的。即活跃连接数最少的服务器会自动接收下一个传入的请求
- ip_hash
  - 每个请求按照访问ip的hash结果进行分配，使每个访客固定访问一个服务器，可以解决session的问题
- fair
  - 按后端的响应时间来分配，响应时间短的优先分配

## Docker

### Docker和传统虚拟机的区别

## 场景题

### 现有1T的数据，内存只有1G，该怎么对他们进行排序

- 申请1g可以创建的接近2的次方的大根堆，遍历1t的数据，记录小的记录，输出到结果文件上

### 现有40亿条数据，内存只有1G，找出现次数最多的数

- 将这40亿条数据用hash算法算出其hash值，然后模上100
- 将取模后值相等的数放在同一个文件下
- 再用hash表将这100给文件进行排序找出出现次数最大的数
- 这样得到100个各个文件出现次数最大的数
- 再排序这100个数得到出现次数最大的数

### 1000W数据入库，实时返回成功条数

- 分片读取到内存，对数据进行检验，利用多线程批量入库

### 40亿个无符号整数，找出在0 ~ 2^32^ - 1(4294967295) 范围中没出现的数

1. **最多使用1GB内存**
   - 使用一个  2^32^ / 8大小的  byte数组约500MB
   - 遍历40亿个数，出现的为1，没出现的为0
2. **最多使用3kb内存，但只要找到一个没出现的数即可**
   - 3kb如果用来创建整形数组，最大可以创建768大小的数组
   - 我们创建一个512大小的数组即可
   - 遍历40亿个数，将每个数除于 2^32^ / 512（8388608）
   - 然后将对应的数组下标++，最后不够8388608的区间就是没出现的所在区间
   - 在不够的区间再分成512份继续以上操作
3. **最多使用有限的变量，找到一个没出现的数**
   - 将40亿数据二分，哪边没满  2^32^ / 2 的继续二分

### 一个包含100亿个url的大文件，假设每个url占64B，找出其中重复的url

### 某搜索公司每天的用户搜索是海量的，求top100热词

- hash分流成小文件，各个小文件自身构建大顶堆，拿出各个文件大顶堆堆顶，组成全局大顶堆，弹出堆顶，把堆顶下的再加入全局大顶堆

### 如何排查Linux服务器CPU使用率过高？

- 先查看tomcat日志
- 使用top命令查cpu占用高的进程
- 找出该进程中占用cpu高的线程
- 使用jvm工具排查该线程信息

### 位运算问题

1. **不做比较求出a和b中大的一个** 
   - a - b >> 31 & 1
     - 是1则a < b
     - 是0则a >= b
2. **判断一个32位的数是不是2的次幂？**
   - 判断 a & ~a + 1 是不是等于1，其余所有位置都是0
   - 判断 x & x - 1 是不是等于0 
3. **判断一个32位的数是不是4的次幂？**
   - 判断 x & x-1 是不是等于0 并且 x & 0x55555555（...10101010） 不等于0
4. **不使用算术运算符，实现a和b的加减乘除**
   - 加法
     - 求 a ^ b 
     - 再求 a & b >> 1
     - 循环以上过程直到 a & b >> 1 等于0，a ^ b 就是相加的答案
   - 减法
     - b = b的相反数 （~b + 1 即补码） 再调用加法的过程
   - 乘法
     - 如果 b & 1 == 1, res加上a，a << 1, b >>> 1
     - 直到b等于0结束，res就是结果
   - 除法
     - b左移到a能减的最左位置（找位置时用a右移来找比较安全）

### **订单超时取消**

- DelayQueue延迟队列
- 时间轮算法

- 数据库定时任务（如Quartz）

- Redis ZSet 实现

- MQ 延时队列实现

### **Token的生成方式**

- 使用JWT生成
  - Header、payload、signature
- 用设备号/设备mac地址作为Token(推荐)
  - 缺点是客户端需要带设备号/mac地址作为参数传递，而且服务器端还需要保存
  - 优点是客户端不需重新登录，只要登录一次以后一直可以使用
- 用session值作为Token
  - 缺点就是当session过期后，客户端必须重新登录才能进行访问数据
  - 优点是方便，实现简单

## 业务题



## 算法

### 排序

- 选择
