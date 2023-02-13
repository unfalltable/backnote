# 注册中心

## Eureka

### 简介

- 服务提供者启动时会向Eureka注册信息，每30秒发送一次心跳
- Eureka为服务消费者提供服务提供者的信息

### 使用

- Eureka搭建
  - 启动类上加 `@EnableEurekaServer`
  - 在yaml中配置eureka地址
- 服务注册
  - 在yaml中配置eureka地址
- 服务拉取
  - 在yaml中配置eureka地址
  - 修改访问Url，用服务名代替ip和端口
  - 添加 `@LoadBalanced` 启动负载均衡

## Nacos

### 简介

- 基于Java语言实现的
  
- 端口：8848
- 进入 nacos / bin 目录下
  - `startup.cmd -m standalone`  ：启动Nacos
- 服务分级存储模型
  - 服务 - 集群 - 实例
  - 进行地域划分集群
  
- NacosRule负载均衡策略
  - 默认是随机访问
  - 需要在配置文件中配置使用Nacos负载均衡规则
    - 服务调用优先调用本地集群，本地集群不可用时访问其他集群
- 可以在nacos控制面板中设置实例权重
  - 权重设置为0则不处理请求
- 实例可以选择临时实例或者非临时实例
  - `ephemeral: false`

### Nacos和Eureka的区别

- 相同点
  - 都支持服务注册和拉取、心跳检测
- 不同点
  - Nacos主动检测提供者状态，临时实例采用心跳检测，非临时采用主动检测
    - 临时实例不正常会剔除，非临时不会
  - Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
  - Nacos集群采用AP模式，集群中存在非临时实例采用CP模式
  - Eureka采用AP方式

# 配置中心

## Nocos

### 作用

- 同一配置管理
  - 实现热更新
    - 一般配置一些开关或者一些格式
- 服务拉取Nacos配置
  - 新建一个boostrapt.yml 或者 properties 文件
    - 配置Nacos地址，配置文件后缀名，模块名

  - 使用@NacosValue可以获取配置文件信息
- 配置自动刷新，优先使用nacos的配置
  - 接口上添加 @RefreshScope 实现自动刷新
  - 也可以创建一个类专门完成属性的加载
    - 类上添加 @ConfigurationProperties(prefix = “”)、@Component
    - 对应Nacos配置中的前缀
    - 创建一个变量接收这个配置的信息

### 环境隔离NameSpace 多环境配置共享

- Nacos控制面板中可以创建命名空间，在配置文件中配置namespace，内容是命名空间自动生成的id
- 多种配置的优先级
  - 服务名称-开发环境.yaml  > 服务名称.yaml 中 > 本地配置
- 命名空间可以按服务来划分，各个服务使用自己的命名空间
- 可以使用配置多配置文件，将配置信息都放在nacos上

## Consul

# 负载均衡

## Ribbon

- 请求先到Ribbon，然后Ribbon取注册中心找服务地址
- 使用时往Spring容器中添加IRule对应的子类即可（全局），也可以在配置文件中配置（部分微服务）
- 默认是懒加载的，第一次访问时才会创建LoadBalanceClient，请求时间比较长
  - 可以设置饥饿加载，启动时就会创建，可以指定饥饿加载的服务器


## 规则

- RoundRobinRule：轮询
- AvailabilityFilteringRule：忽略短路和并发数过高（可以指定上限）的服务器
- WeightedResponseTimeRule：服务器响应时间越久，服务器的权重越小
- ZoneAvoidanceRule：分区，再对区内轮询
- BestAvailableRule：忽略短路的服务器，选择并发数低的服务器
- RandomRule：随机
- RetryRule：重试机制

## LoadBalance

# 远程调用

## Dubbo

### 简介

- 服务治理框架
- 支持多协议

## Feign

### 简介

- 是一个声明式Http客户端
- 传统的Http远程调用是使用 RestTemplate ，存在一些问题
  - 代码可读性差，编程不统一
  - 参数复杂Url难以维护
- 自定义Fegin的配置
  - Fegin运行自定义的配置来覆盖默认配置
  - 配置：
    - fegin.Longger.Level：修改日志等级
      - NONE：没任何日志
      - BASIC：一次请求的请求时间，结束时间，耗时时间等基本信息
      - HEADERS：基本信息 + 请求头信息
      - FULL：基本信息 + 请求头信息 + 请求体 + 响应体

    - fegin.codec.Decoder：响应结果的解析器，将Json 转换为 Java对象
    - fegin.codec.Encoder：请求参数编码
    - fegin.Contract：支持的注解格式，规定支持的注解，默认是SpringMVC的注解
    - fegin.Retryer：失败重试机制，默认是不重试

- 使用
  - 配置文件
  - 加入Spring容器，创建对应的配置类
    - 全局配置：@EnableFeignClient(defaultConfiguration = 配置类)
    - 局部配置：@FeignClient(value = “作用的服务”, configuration = 配置类)


### 性能优化

- 底层的客户端，修改为支持连接池的客户端（导包 - 配置）
  - 默认是 URLConnection，不支持连接池
  - Apache HttpClient：支持连接池
  - OKHttp：支持连接池
- 日志级别最好设置为Basic

### 最佳实践

- 继承
  - 创建一个接口写这个方法声明，再让Client和Controller去继承和实现
  - 缺点
    - 对SpringMVC不起作用，这个接口的参数列表中的映射不会被继承
    - 服务紧耦合
- 抽取
  - 将方法声明，使用到的Pojo，默认配置等抽取为一个模块
  - 缺点
    - 会将所有方法引入，多余了一些

## OpenFeign

# 网关

## Gateway

- 基于WebFlux实现d
- 拦截所有客户端请求，进行身份认证和权限校验
- 进行服务路由，负载均衡
- 请求限流
- 路由断言工厂 （predicates）
  - 配置写的字符串会被 Predicate Factory 读取并处理，转变为路由判断的条件
  - spring 提供了11个断言工厂
    - After、Before、Between、Cookie、Header、Host、Method、Path、Query、RemoteAddr、Weight
- 过滤器 （filters）
  - 当前路由过滤器
  - 默认过滤器
    - Default-filter

  - 全局过滤器
    - GlobalFilter 接口

  - 执行顺序
    - 指定Order来决定执行顺序，值相同时，先执行默认，然后局部，然后全局 

# 队列

## AMQP

# 安全监控

## Sentinel

- 雪崩问题
  - 超时处理：一定时间没响应就返回错误信息
  - 舱壁模式（线程隔离）：限定每个业务能使用的线程数，避免tomcat资源耗尽
  - 熔断降级：断路器统计业务执行的异常比例。超出阈值则熔断该业务，拦截访问该业务的一切请求
  - 流量控制：限制业务访问的QPS，避免服务流量突增发生故障

## 流量控制

## 雪崩问题

### 隔离

- 线程池隔离
- 信号量隔离（mo'r

### 降级

## 授权规则

- 通过过滤器给请求添加一个特殊标识的头信息，携带这个信息的请求才可以正常访问

## 规则持久化

## 线程隔离

- 线程池
  - 支持主动超时、支持异步调用
  - 线程开销大
  - 支持低扇出
- 信号量（默认实现）
  - 轻量级，开销小
  - 无主动超时和异步调用
  - 高频调用、高扇出

## 熔断降级

- 断路器
  - closed：服务正常调用，统计服务异常比例，达到阈值则熔断，进入open状态
  - open：熔断状态，禁止目标服务调用，熔断持续一段时间后进入half-open状态
  - half-open：尝试请求服务，如果能正常访问则恢复到closed状态
- 熔断策略
  - 慢调用：响应时间慢，一点数量的请求慢则熔断，可以设置响应时间阈值
  - 异常比例：在一定请求中抛异常的请求数量超过一定比例则熔断
  - 异常数：异常请求达到设置的异常数则熔断











