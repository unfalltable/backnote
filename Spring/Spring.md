---
title: Spring5
toc: content
keywords: [spring]
---

# IOC

## 是什么

- IOC 控制反转，即将对象的创建个调用交给Spring管理
- DI 依赖注入
  - @Autowired、@Resources
- 容器
  - 存储Bean对象，使用map结构存储

## 执行流程

1. 解析各种来源的bean信息，存入BeanDefinition中
2. 依据BeanDefinition中的类定义信息，反射创建其对象，涉及到**Bean的生命周期**

## 原理

1. createBeanFactory创建出容器
2. 循环创建bean对象，先通过getBean，doGetBean从容器中找
3. 找不到就通过createBean,doCreateBean以反射的方式创建对象，一般使用无参构造创建
4. 然后对对象属性进行填充

# AOP

## 是什么

- AOP是IOC流程中的一个扩展点，在bean初始化的后处理器**BeanPostProcessor**
- 在不通过修改源代码的基础上，对其进行功能的添加，例如日志输出，计算时间等等
- 使用到了**装饰器设计模式** 

## 实现

1. 通过ajc编译器实现
   - 需要添加aspec-maven-plugin
   - 原理是直接将增强方法写进类中，不使用代理，不走IOC
2. 通过Agent类加载
   - 需要添加JVM参数 `-Javaagent:maven仓库路径/`
   - 原理也是直接将增强方法写进类中，不使用代理，不走IOC
3. 通过jdk动态代理
   - 代理对象有实现接口则使用JDK代理，创建的代理和代理对象一个等级
4. 通过cglib动态代理
   - 创建的子类代理对象

## 定义

- 切点，要增强的方法
  - @Pointcut()，传入 
    - execution 表达式匹配方法名
    - @Annotation() 表达式匹配注解名
    - 实现StaticMethodMatcherPointcut，重写其matchs(Method method, Class<?> targetClass)
- 通知 ，类型
  - @Before、@After、@AfterReturning、@Around、@AfterThrowing、@After
- 切面，是一个动作
  - @Advice
- 优先级
  - @Order(值)    前置通知值小优先，后置通知值大优先

## 使用步骤

```java
//切点
AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
pointcut.setExpression("execution(* 被代理方法())");
//通知
MethodInterceptor(){} advice = new MethodInterceptor(){
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable{
		//方法增强
        Object result = invocation.proceed();
        return result;
    }
}
//切面
DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, advice);
//代理
ProxyFactory factory = new ProxyFactory();
Target target = new Target();
factory.setTarget(target);
//告诉代理工厂目标类是否实现接口，有实现接口则使用JDK
factory.setInterfaces(target.getClass().getInterface());
//强制使用CGLIB代理
factory.setProxyTargetClass(true);
factory.setAdvisor(advisor);
接口 proxy = (接口) factory.getProxy();
```

## 静态代理

- ajc编译器
- 添加虚拟机参数

## 动态代理

### JDK代理

- 被代理对象需要实现接口

- JDK代理和被代理对象同级，兄弟关系

- JDK代理前16次通过反射调用方法，后续都是

- ```java
  ClassLoader loder = 被代理类.class.getClassLoader();
  //被代理对象
  Target target = new Target();
  接口 proxy = (接口) Proxy.newProxyInstance(loader, new Class[]{接口.class}, new InvacationHandler(){
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throw Throable{
          //执行增强
          //调用被代理对象方法
          Object result = method.invoke(target, args);
          return result;
      }
  });
  ```

### CGLIB代理

- CGLIB代理和被代理对象是父子关系，所以被代理对象不能是final修饰的

- CGLIB可以通过方法代理直接调用被代理方法，不走反射

- 被代理对象的方法是final修饰的话不会有增强效果

- ```java
  //被代理对象
  Target target = new Target();
  Target proxy = (Target) Enhancer.create(Target.class, new MethInteceptor(){
      @Override
      public Object intercept(Object p, Method method, Object[] args, MethodProxy methodProxy) throw Throwable{
          //执行增强
          
          //调用被代理对象方法（反射）
          Object result = method.invoke(target, args);
          //调用被代理对象方法（直接调用）(需要被代理对象)（Spring使用）
          Object result = methodProxy.invoke(target, args);
          //调用被代理对象方法（直接调用）(不需要被代理对象)
          Object result = methodProxy.invokeSuper(p, args);
          
          return result;
      }
  });
  ```

## AspectJ

- 性能更好更易用

### 使用步骤

- 在增强类上添加@AspectJ 和 @Component
- 写切点和通知即可

## 代理对象的创建

- 创建的时机
  - 一般是在初始化之后创建
  - 出现循环依赖时，则会在依赖注入之前创建，并暂存二级缓存中

# 容器

## 创建

- 通过Refresh()创建容器（12个方法）
  - ApplicationContext的准备工作（1）
    - prepareRefresh
      - 创建Environment对象
        - 提供键值信息：虚拟机信息、操作系统信息、自定义文件信息
        - 为@Value注入信息
  - 准备和完善BeanFactory（2-6）
    - obtainFreshBeanFactory
      - 获取或创建BeanFactory
      - 通过BeanDefinition把对象信息存放在map中
      - 创建需要的对象
    - prepareBeanFactory
      - 初始化BeanFactory中的成员变量
        - beanExpressionResolver  解析Spel表达式
        - propertyEditorRegistrars   注册类型转换器，解析#{}
        - resolvableDependencies   管理特殊bean，beanFactory和ApplicationContext，进行依赖注入
        - beanPostProcessors         后处理器集合
    - postProcessBeanFactory
      - web环境下的ApplicationContext要利用它注册新的scope，完善Web下的BeanFactory
      - 留给子类来做扩展，体现了模板方法设计模式
      - 使用的少
    - invokeBeanFactoryPostProcessors
      - 对BeanFactory做扩展
      - ConfigurationClassPostProcessor
        - 解析@Configuration、@Bean、@Impor、@PropertySource
    - registerBeanPostProcessors
      - 创建并加入更多的后处理器，主要看BeanDefinitionMap中对象是否实现了PostProcessor接口
      - 加入beanPostProcessors集合中
  - 完善ApplicationContext（7-12）
    - initMessageSource
      - 创建MessageSource
      - 实现国际化功能
    - initApplicationEventMulticaster
      - 创建事件广播器
    - onRefresh
      - 空实现，留给子类实现
      - SpringBoot中的子类可以在这里准备内嵌的web容器
    - registerListeners
      - 创建事件监听器
    - finishBeanFactoryInitialization
      - 初始化BeanFactory中的成员变量
        - conversionService 和 propertyEditor 一起作为转换机制
        - embeddedValueResolvers 内嵌解析${}
        - singletonObject   创建beanDefinitionMap中保存的对象
    - finishRefresh
      - 创建生命周期处理器

## 接口

### BeanFactory

- spring的核心容器
- 提供getBean()

#### 实现类

##### DefaultListableBeanFactory

- 提供控制反转、基本的依赖注入、Bean生命周期的各个功能

- 默认无其他后处理器，解析不了注解，需要添加

  - `AnnotationConfigUtils.registerAnnotationConfigProcessors(容器对象);`

  - 添加一些Spring内置的后处理器，但是并未启用

    - 启动后处理器

      - ```java
        容器对象.getBeanOfType(BeanFactoryPostProcessor.class)
            ,values().stream().forEach(beanFactoryPostProcessor -> {
            beanFactoryPostProcessor.postProcessBeanFactory(容器对象);
        })
        ```

- 容器中的Bean对象在使用时才会创建
  - 可以提前一次性创建
    - `容器对象.preInstantiateSingletos()`
  - 默认不会解析 ${}、#{}

### ApplicationContext

- 间接继承了BeanFactory，功能更多，国际化、匹配资源、发布事件、环境信息等

#### 实现类 

##### ClassPathXmlApplicationContext、FIleSystemXmlApplicationContext

- 创建DefaultListableBeanFactory

- 通过XmlBeanDefinitionReader中的loadBeanDefinitions()

- 读取Xml中的Bean

##### AnnotationConfigApplicationContext

- 通过配置类创建容器

- 会自动添加常用的后处理器

##### AnnotationConfigServletWebServerApplicationContext

- 借助了内嵌的Tomcat
- 需要提供一些Bean
  - ServletWebServerFactory
  - DispatcherServlet
  - DispatchServletRegistrationBean
- 非必须Bean
  - Controller（web.servlet.mvc）
    - 处理Request、Response
    - 可以指定访问的url


# Bean

## 作用域（Scope）

- singleton
- prototype
- request
- session
- application

## 生命周期（doGetBean）

- 处理名称，检查缓存
  - 解析别名，解析特殊符号，查询三级缓存看是否有创建好的Bean

- 处理父子容器
  - 如果有父容器，回去父容器是否有创建好的Bean
    - 父子容器的bean名称可以重复
    - 优先找子容器Bean，子容器没有再找父容器

- dependsOn
  - 判断是否用使用dependsOn指定Bean的创建顺序

- 按不同的Scope创建Bean
- 实例化
  - 通过构造函数创建
  - 反射生成对象，堆中申请空间，给成员变量赋默认值
- 初始化
  - Bean属性注入
    - 
  - Bean功能扩展
    - BeanPostProcessor
      - 
- 使用，通过容器对象调用getBean()方法获取Bean对象
- 销毁

## 注入属性

- `<property name="" value="" />`  需要有对应的set方法
  - `<![CDATA[  此处写特殊值   ]]>`  注入特殊值
- `<constructor-arg name="" value=""></constructor-arg>` 需要有对应的有参构造方法
- @Autowired、@Qualifer、@Resource(name = "")、@Value(value = "")

## 加载方式

- XML中 \<Bean id="" class="" />
- @Component、@service、@Controller、@Repository、@Import、@Bean
- @ComponentScan(“路径”)、@Configuration
- @ImportResource(“xml名”)
- @Configuration(proxyBeanMethods =)
- FactoryBean\<T>
- 容器对象.registerBean()
- 容器对象.registerSingleton(“Bean名”, Bean对象)    不会走bean的创建、依赖注入、初始化过程
- 实现 ImportSelector、ImportBeanDefinitionRegister、BeanDefinitionRegistryPostProcessor 接口，实现对应的方法
- 构造注入

## 加载控制

- @Conditional()
- 根据一些条件决定是否加载Bean

## BeanDefinition

- 获取BeanDefinition

  - ```java
    BeanDefinitionBuilder.genericBeanDefinition(类.class)
        .setScope("singleton")  //设置作用域
        .getBeanDefinition()
    ```


- 向容器中注册Bean
  - `容器对象.registerBeanDefinition("Bean名", BeanDefinition)` 

# 后处理器

### 简介

- 作用时机
  - 实例化前后-依赖注入阶段-初始化前后-销毁前  进行扩展


### Bean后处理器

#### AutowiredAnnotationBeanPostProcessor

- 解析@Autowired、@Value

#### CommonAnnotationBeanPostProcessor

- 解析@Resource、@PostConstruct、@PostDestroy

#### ConfigurationPropertiesBindingPostProcessor

- 解析@ConfigurationProperties

#### AnnotationAwareAspectJAutoProxyCreator

- 解析AOP相关注解
- 获取所有切面

  - 通过findEligibleAdvisors(目标类) 找到所有作用于目标类的切面，会将高级切面转化为低级切面

- 创建代理

  - 通过wrapIfNecessary(目标类对象)，需要判断findEligibleAdvisors()返回的集合是否为空

### BeanFactory后处理器

- ConfigurationClassPostProcessor
  - 解析@ComponentScan、@Bean、@import、@ImportResource
- MapperScannerConfigurer
  - 解析@MapperScan
- internalConfigurationAnnotationProcessor

  - 解析@Configuration、@Bean


### 后处理器执行顺序

- 加入处理器的顺序决定执行的顺序
- 可以通过比较器控制加入的顺序
  - `容器对象.getDependencyComparator()` 
- 因为后处理器中都会实现一个getOrder方法，实现了Order接口
  - Order值越小优先级越高

# 内置功能

- 不用后处理器解析就能使用

## Aware

- BeanNameAware：注入Bean的名字
- BeanFactoryAware ：注入BeanFactory容器   
- ApplicationContextAware：
  - 接口
  - 注入ApplicationContext容器

- EmbeddedValueResolverAware：解析 $ { }

## InitializingBean

- 容器初始化接口

# 事务

## 是什么

- 事务一般加在Service层
- 底层使用AOP
- @Transaction
  - propagation传播属性
  - isolation隔离级别
  - rollbackFor
  - noRollbackFor


## 传播行为（7种）

A中调用B

- Requirded：A有B就用A的，没有就创建
- Required_New：不管A有没有，B创建新的
- Supports：A有就用A的，没有就不用
- Not_Supports：A有就挂起，B不用事务
- Mandatory：A没有就抛异常
- Never：A有就抛异常
- Nested：A里有，B在A里的事务执行，A里没有就在里面创建

## 回滚

- 事务是由AOP来实现的，首先生成代理对象，通过TransactionInterceptor，调用invoke实现具体逻辑
  1. 解析方法上事务的相关属性，判断是否开启新事务
  2. 获取数据库连接，关闭自动提交，开启事务
  3. 执行具体的sql逻辑
     - 成功：通过commitTransactionAfterReturning提交，通过doCommit实现
     - 失败：通过commitTransactionAfterThrowing执行回滚，通过doRollBack实现
  4. 之后清除相关的事务信息，cleanupTransactionInfo

## 失效及解决方法

- 抛出检查异常
  - 可以在注解上添加 `rollbackFor = Exception.class` 解决

- 捕获了检查异常
  - 需要抛出才能回滚，并且需要加`rollbackFor = Exception.class` 
  - 也可以使用 `TransactionInterceptor.currentTransactionStatus().setRollbackOnly()` 

- 自定义切面捕获了异常没抛出
  - 不抛出的话，也可以通过改变切面执行优先级解决，添加@Order

- 事务需要加载public修饰的方法上，否则不起作用
  - 可以将添加 AnnotationTransactionAttributeSource( false ) Bean  **不推荐** 

- 父子容器中父容器开启了事务，子容器没有配置事务并且调用的子容器的Bean的方法
  - 需要加载父容器中的Bean

- 在事务方法中直接调用其他事务方法，因为直接调用不是用到代理对象调用，其实是用的this
  - 自己注入自己
  - 设置@EnableTransactionManagement(exposeProxy = true)，然后使用AopContext.currentProxy()获取代理对象，然后执行方法

- 多线程下可能出现失效
  - 需要加锁在事务的完整的过程，或者在数据库加行锁

- mysql没开启事务

# 其他注解

## @Nullable

## @Indexed

- 在编译时将标注了该注解的Bean，会生成/META-INF/spring.components文件并加入其中
- 扫描包时会先查这个文件并加载其中的Bean
- 没有再走包sao'miao

## @Order

- 数字越小的优先级越大

## @Lazy

- 标注类上是延迟创建
- 标注方法参数或成员变量上，

## @primary

- 优先注入该Bean

## @Qualifier

- 不注入该Bean

# WebFlux

- 异步非阻塞的框架、响应式编程，功能和SpringMVC类似，基于Netty
- SpringWebFlux + Reactor + Netty
- 请求和响应是 ServerReuquest 和 ServerResponse
- 观察者模式 Oberver（Flow取代）
- 主要使用Flux 和 Mono
  - Flux
    - 声明多个数据流
    - 可传入数组（fromArray）、集合（fromItertr）、流（fromStream）

  - Mono
    - 声明0个或1个数据流

- 可以发送信号
  - 错误信号

- 需要订阅后才能发出数据流
  - subscribe()

- 操作符
  - map 映射
  - flatMap 将元素映射成流


## 执行流程

- 与SpringMVC基本一致，使用到的组件不同
- 使用的是DispatchHandler

