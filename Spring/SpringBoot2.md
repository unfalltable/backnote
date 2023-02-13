---
title: SpringBoot
keywords: [spring, springboot]
---

## SpringBoot2的特点

### 依赖管理

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>

<!--上面的父项目-
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>
```

- spring-boot-dependencies 几乎声明了所有开发中常用依赖的版本号，实现自动版本仲裁
- 可自定义版本号

### 自动配置

- ==@SpringBootApplication== 
- 通过SpringApplication.run() 初始化容器
- 主程序所在的包及其子包都会被默认扫描，并且加载所有的自动配置类
  - xxxxAutoConfiguration
- 各种配置都有默认值，最终都是映射到一个类上
- 配置是按需加载的，按引入场景自动配置，没引入不生效
  - 各种starter
- 所有的配置功能都在Spring-boot-autoconfigure包里面
  - 会默认加载全部场景，但最终会按需配置，依靠条件装配（@Canditional ）实现
- 如果不想使用默认的配置，可以直接@Bean替换底层的组件，到Application.properties中进行相应的修改
- 默认的配置会从xxxxProperties中获得，xxxProperties和Application.properties进行了绑定，application.properties中的修改会覆盖默认配置
- 可以在配置文件中设置debug=true开启自动配置报告，查看配置的详细情况，生效和不生效的

#### 源码分析

- ==@SpringBootApplication==合并了以下三个注解
  - ==@SpringBootConfiguration==
    - ==@configuration== 标记为配置类
  - ==@EnableAutoConfiguration==
    - ==@AutoConfigurationPackage== 自动配置包，将包下所有组件导入
      - ==@Import(AutoConfigurationPackage.Registrar.class)==
        - 利用Registrar批量注册组件
    - ==@Import(AutoConfigurationImportSelector.class)==
      - 利用getAutoConfigurationEntry(annotationMetadata a) 给容器批量导入一些组件
      - 调用getCandidateConfigurations(annotationMetadata, attributes) 获取所有需要导入的容器中的配置类/组件
        - 利用了SpringFactoriesLoader（Spring工厂加载器）的loadFactoryNames() 这个方法返回loadSpringFactories的Map得到所有的组件
          - loadSpringFactories方法会扫描当前系统里面所有的META-INF/spring.factories
  - ==@ComponentScan("包路径")==
    - 包扫描

### 底层注解

- ==@Configuration==标注配置类
  - proxyBeanMethods 代理bean方法
    - true   保持组件单实例，默认true       ------ Full模式
      - 调用对象组件方法会从容器中返回对象
      - 组件有依赖关系就用true
    - false 不是单实例                                    ------ Lite模式
      - 调用对象组件方法会创建对象
      - 如果只是创建组件false比较快
  - ==SpringApplication.run(MainApplication.class,args);==
    - 返回IOC容器
    - ==getBeanDefinitionNames();==
      - 查看容器中组件的名字
- ==@Bean==
  - 向容器注册组件
  - 方法名为组件id
  - 返回类型是组件类型
  - 放回值就是容器中的实例
- ==@Import(类.class)==
  - 向容器中创建组件，默认组件名字是全类名
- ==@Conditional==
  - 有很多子实现
    - ==@ConditionalOnBean==
  - 写在方法上有先后执行
  - 设定条件，按**条件装配**容器
- ==@ImportResource("classpath:路径")==
  - 导入资源（xml）

### 配置绑定

1. ==@Component==+==@Configuration Properties(prefix =  "")==
     - prefix/value 根据前缀筛选
2. ==@EnableConfigurationProperties(类.class)==+==@ConfigurationProperties()==
- ==@Enable...==需要在配置类中写
  
- 开启该类的配置绑定功能，并注册到容器中

## 核心功能

### Web开发

#### 静态资源配置

##### 静态资源文件夹

- resources下的这些目录是静态资源目录

  - static、public、resources、META—INF/resources

- 只要是在这些文件夹下的都可以直接访问资源

  - 原理：静态映射/**
  - 通过根路径/+静态资源名

- 有请求进来时优先找Controller，然后再找静态资源

- 可以为静态资源文件夹加前缀

  - 在配置文件中

  - ```yaml
    spring:
      mvc:
        static-path-pattern: /前缀/**
    ```

    - 加了前缀会影响欢迎页和网页Favicon图标

- 可以改变默认的静态资源路径

  - 在配置文件中

  - ```yaml
    spring:
      resources:
        static-locations: classpath:/资源文件夹名/
    ```

    - 改变后原来默认的资源文件夹都会失效

##### 欢迎页

- 在静态资源文件下写一个 index.html 会作为欢迎页
- 用controller处理 /index 

##### Favicon

- 在静态资源文件下放入一个favicon自动作为网页图标

##### 原理

- 加载WebMvcAutoConfiguration

  - ```java
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnWebApplication(type = Type.SERVLET)
    @ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
    @ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
    @AutoConfigureOrder(-2147483638)
    @AutoConfigureAfter({DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class})
    public class WebMvcAutoConfiguration {
    	//...
    }
    ```

##### 补充

```yaml
spring:
  resources:
    add-mappings: false
```

- add-mappings: false   禁用所有静态资源规则

## 执行流程

1. 调用构造方法创建SpringApplication对象
   - 根据各种来源添加BeanDefinition
   - 推断应用程序类型(servlet, none, reacter)
   - 使用初始化器，监听器
   - 进行主类推断
2. 执行run方法
   1. 得到SpringApplicationRunListeners事件发布器
      - 发布7个事件
        - starting：开始启动
        - environmentPrepared：准备环境
        - contextPrepared：容器创建并调用初始化器
        - contextLoaded：Bean定义加载
        - started：refresh方法执行
        - running：springboot启动完成
   2. 封装启动args
      - 用DefaultApplicationArguments(args)封装

## 开发步骤

1. ### 配置 pom文件

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>
<!--web项目的依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2. ### 配置主程序类

```java
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

- ==@SpringBootApplication==来说明这是个SpringBoot应用

3. ### 编写业务类

```java
//@ResponseBody
//@Controller
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String handle1(){
        return "Hello SpringBoot2!!!";
    }
}
```

- ==@RequestMapping==("/浏览器发送的内容")    
  - 映射浏览器的请求
  - 按浏览器发送的内容调用方法

- ==@ResponseBody==
  - 将方法返回的内容写给浏览器

- ==@RestController==
  - 等于==@ResponseBody==加==@Controller==

4. ### 编写配置信息

   - 创建一个application.properties配置类
   - 即可配置所有需要用到的

5. ### 部署项目

   - 在pom文件中引入插件，把项目打成jar包，可直接运行

   ```xml
   <plugins>
       <plugin>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-maven-plugin</artifactId>
       </plugin>
   </plugins>
   ```

## 整合ssmp

### 实体类层（domain）

- 创建要用到的实体，比如 Book、User 等等

### 数据层（dao）

- 使用mybatis-plus 实现CRUD，然后测试
- 开启mp运行日志
  - ![image-20220121103834129](..//pic//image-20220121103834129.png)
- 配置分页设置
  - 创建一个mp的配置类 MPConfig
    - ![image-20220121104135610](..//pic//image-20220121104135610.png)
      - MybatisPlusInterceptor是MP的拦截器
      - PaginationInnerInterceptor是分页内部拦截器

### 业务层（service）

- 业务层一般写业务方法，例如login，register 等等

- 创建实体类对应的service 接口，例如BookService、UserService 等等
  - 创建要使用到的业务的抽象方法
- 创建对应的实现类，例如 BookServiceImpl 等等
  - ==使用@Autowired自动注入数据层== 
  - 实现Service接口的方法，一般返回值是boolean类型，表示业务执行成功与否
  - 调用数据层的方法实现业务功能
- 测试
- 为了简便以上繁琐的开发，Mp提供了一套快速开发
  - 创建实体类对应的接口继承 ==IService<T>== 类
    - ![image-20220121131707505](..//pic//image-20220121131707505.png)
  - 创建对应的实现类继承 ==ServiceImpl<K, V>== 实现对应的接口
    - ![image-20220121132705036](..//pic//image-20220121132705036.png)

### 表现层（controller）

- 创建controller类，在类上添加@RestController、@RequestMapping("/映射地址")
- 使用rest风格，==注入业务层==

### 消息一致性处理

- 在表现层处理

- 设计一个模型类，把返回的数据封装到这个模型类中，称为前后端协议

## 工具

### lombok

- 简化Bean

#### 配置

- ```
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
  </dependency>
  ```

- ```
  <plugin>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
  </plugin>
  ```

### Yaml

- 简化配置信息，常用xxx.yml
- 适合以数据为中心的配置文件
- 在双引号中支持转义字符
- 可以将配置文件中的数据封装到Environment对象中，加上@Autowired
  - 用getProperty("") 获取对应的信息

- 数据读取
  - 定义一个实体用于存放yml文件中的配置信息
  - 实体内部的成员变量名需要和配置中的一致
  - 在实体上添加@ConfigurationProperties(prefix = "配置名"),并将实体加入到容器中


### 配置提示

- 引入依赖

  - ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-		 processor</artifactId>
    </dependency>
    ```

## 技巧

### 复制模块

![image-20220118141414868](..//pic//image-20220118141414868.png)

## 注意

- 只有在容器中的组件才会拥有SpringBoot提供的功能
- 如果配置类只有一个有参构造器，它的所有参数的值都会从容器中确定

## 运维实用

![image-20220121183317113](..//pic//image-20220121183317113.png)

### 修改端口号

- `java -jar 包名 --server.port=8080`
  - --后面写需要修改的配置，对应yml文件里的配置

### 日志

- 创建日志对象
  - 写一个静态常量
  
    - ```java
      private static final Logger log =                                    LoggerFactory.getLogger(BookController.class)
      ```
  
  - 或者创建一个类去继承
  
    - ```java
      //定义一个有日志对象的类并创建
      public class BaseClass{
          private Class clazz;
          public static Logger log;
          public BaseClass(){
              clazz = this.getClass();
              log = LoggerFactory.getLogger(clazz);
          }
      }
      //继承上面的类后即可使用日志对象
      public class Book extends BaseClass{
          log.info();
      }
      ```
  
  - 或者在想使用日志的类上添加@Slf4j，然后在类中直接使用Log对象
  
    - ```java
      @Slf4j
      public class Book(){
          log.info();
      }
      ```
      
      - 需要同时添加lombok
  
- 配置日志级别
  - ![image-20220121230358127](..//pic//image-20220121230358127.png)

- 日志输出格式
  - ![image-20220122143914895](..//pic//image-20220122143914895.png)
    - %d：日期
    - %m：消息
    - %n：换行
    - %clr()： 颜色
      - 在后面加{} 指定颜色
    - %t： 线程名
    - %c：类名
    - 在%后加数字可以控制长度
      - 正数右对齐，负数左对齐
    - 在%数字后加数字可以内容长度

- 保留日志文件

## 开发篇

### 热部署

![image-20220122173257165](..//pic//image-20220122173257165.png)

- 开启自动启动热部署
  - Bulid -> Compiler -> Bulid project automatically
  - Settings -> Advanced Settings -> Allow auto-make to 

![image-20220122174058004](..//pic//image-20220122174058004.png)

![image-20220122174357110](..//pic//image-20220122174357110.png)

![image-20220122174808490](..//pic//image-20220122174808490.png)

### 高级配置

![image-20220123104038852](..//pic//image-20220123104038852.png)![image-20220123112457178](..//pic//image-20220123112457178.png)

![image-20220123112609439](..//pic//image-20220123112609439.png)![image-20220123133559396](..//pic//image-20220123133559396.png)

### 常用计量单位

![image-20220123143054552](..//pic//image-20220123143054552.png)

### Bean数据校验

![image-20220123144209154](..//pic//image-20220123144209154.png)

![image-20220123144234511](..//pic//image-20220123144234511.png)

![image-20220123144250340](..//pic//image-20220123144250340.png)

### 加载测试专用属性

![image-20220123152043913](..//pic//image-20220123152043913.png)

![image-20220123152440680](..//pic//image-20220123152440680.png)

### 加载测试专用配置

![image-20220123153110123](..//pic//image-20220123153110123.png)

### Web环境模拟测试

![image-20220123154607248](..//pic//image-20220123154607248.png)

![image-20220123155154707](..//pic//image-20220123155154707.png)

![image-20220123155604329](..//pic//image-20220123155604329.png)

![image-20220123161447231](..//pic//image-20220123161447231.png)

![image-20220123161814689](..//pic//image-20220123161814689.png)

![image-20220123162101843](..//pic//image-20220123162101843.png)

### 数据层测试事务回滚

![image-20220123163912285](..//pic//image-20220123163912285.png)

### 测试用例数据设定

![image-20220123164608927](..//pic//image-20220123164608927.png)

### 数据源配置

![image-20220123165910659](..//pic//image-20220123165910659.png)

- 内置了JDBCTemplate

![image-20220123174507600](..//pic//image-20220123174507600.png)

### 读取配置

- @Value

### 整合Redis

![image-20220123183244873](..//pic//image-20220123183244873.png)

![image-20220123183313872](..//pic//image-20220123183313872.png)

![image-20220123184131259](..//pic//image-20220123184131259.png)

![image-20220123184636107](..//pic//image-20220123184636107.png)

![image-20220123184743164](..//pic//image-20220123184743164.png)

### 整合MongoDB

![image-20220123215753371](..//pic//image-20220123215753371.png)

![image-20220123215816660](..//pic//image-20220123215816660.png)

![image-20220123215846300](..//pic//image-20220123215846300.png)

### 整合ElasticSearch

![image-20220124105416259](..//pic//image-20220124105416259.png)

![image-20220124105452145](..//pic//image-20220124105452145.png)

![image-20220124110838151](..//pic//image-20220124110838151.png)

![image-20220124110850992](..//pic//image-20220124110850992.png)

![image-20220124110908512](..//pic//image-20220124110908512.png)

![image-20220124110928433](..//pic//image-20220124110928433.png)

### 默认的缓存方案

![image-20220124120543131](..//pic//image-20220124120543131.png)

![image-20220124120602179](..//pic//image-20220124120602179.png)

![image-20220124120609718](..//pic//image-20220124120609718.png)

### 整合Ehcache

![image-20220124143014510](..//pic//image-20220124143014510.png)

![image-20220124143058876](..//pic//image-20220124143058876.png)

![image-20220124143125540](..//pic//image-20220124143125540.png)

### 整合Redis缓存

![image-20220124143936988](..//pic//image-20220124143936988.png)

![image-20220124144000676](..//pic//image-20220124144000676.png)

![image-20220124144012503](..//pic//image-20220124144012503.png)

## 注解

### @EnableConfigurationProperties

- 启用 @ConfigurationProperties 的功能

### @ConfigurationProperties

- 将.property文件中的键值信息和Bean的属性进行绑定

### @Condition相关的注解

- 条件装配

### @SpringBootApplication

- 入口

### @EnableAutoConfiguration

### @SpringBootConfiguration

## 原理篇

### 自动配置

### 启动流程

- SpringApplication构造方法
  - 加载各种来源的BeanDefinition源
    - spring内置的Bean来源是null
  - 推断应用程序类型（servlet、none、reactive）
    - 使用ClassUtils判断
  - 准备 ApplicationContext 初始化器
    - 从配置文件中读取初始化器
    - 对 ApplicationContext 做一些扩展
  - 准备监听器事件
    - 从配置文件中读取初始化器
    - 容器创建阶段发布的事件都可以监听到
  - 进行主类推断
    - 通过 deduceMainApplicationClass() 方法去推断
- 执行run方法（12步骤7事件）
  1. 得到 SpringApplicationRunListeners 事件发布器
     - 发布 application starting 事件
       - boot开始启动事件、环境信息准备完成事件、容器创建并执行初始化器事件、BeanDefinition加载完毕事件、容器初始化完成事件（Refresh调用完毕）、boot启动完成事件、boot启动失败事件
  2. 封装args为ApplicationArgument对象
  3. 创建environment对象
     - ApplicationEnvironment对象
       - 有系统属性和系统环境变量，优先找系统属性
       - 可以往里面添加资源
  4. 对environment中命名规范统一处理
     - ConfigurationPropertySources.attach(环境对象)
  5. 对environment做扩展增强
     - 使用EnvironmentPostProcessor后处理器做增强
     - 通过事件发布器的发布与响应，添加后处理器
       - `EnvironmentPostProcessorApplicationListener` 监听器
  6. 对environment中以”spring-main”为前缀的Key与容器对象
  7. 做绑定
     - Binder实现属性绑定
  8. 打印banner
  9. 创建Spring容器
  10. 准备容器
      - 应用初始化器做增强
  11. 加载BeanDefinition
  12. 执行Refresh()
  13. 执行Runner
      - 调用所有实现ApplicationRunner和CommandLineRunner接口的Bean
      - 可用于数据预加载

# 测试相关

- Test下的路径需要和容器所在的路径对应
- 









