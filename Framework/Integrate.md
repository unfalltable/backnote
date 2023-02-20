---
title: 组件集成
toc: content
keywords: [integrate]
---

## 版本

```xml
<properties>
    <spring-cloud.version>2021.0.4</spring-cloud.version>
    <spring-cloud-alibaba.version>2021.0.4.0</spring-cloud-alibaba.version>
    <spring-boot.version>2.7.4</spring-boot.version>
    <myself.prod.version>${project.version}</myself.prod.version>
</properties>
```

## 权限集成

## MybatisPlus集成

### 分页插件

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    // 3.4 之后
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    // 指定数据源
    PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
    // 设置分页最大数限时 -1 不限制
    paginationInnerInterceptor.setMaxLimit(500L);
    interceptor.addInnerInterceptor(paginationInnerInterceptor);
    // 添加防止全表更新与删除插件 （防止恶意语句执行 如 update set 没有where）
    interceptor.addInnerInterceptor(new BlockAttackInnerInterceptor());
    return interceptor;
}
```

### 填充器

```java
@Bean
public MetaObjectHandler getMybatisObjectHandler(ApplicationContext context) {
    return new MetaObjectHandler() {
        private static final String CREATE_BY = "createBy";
        private static final String UPDATE_BY = "updateBy";

        private static final String CREATE_TIME = "createTime";
        private static final String UPDATE_TIME = "updateTime";

        @Override
        public void insertFill(MetaObject metaObject) {
            fill(metaObject, CREATE_BY, CREATE_TIME);
            fill(metaObject, UPDATE_BY, UPDATE_TIME);
        }

        @Override
        public void updateFill(MetaObject metaObject) {
            fill(metaObject, UPDATE_BY, UPDATE_TIME);
        }

        private void fill(MetaObject metaObject, final String userFiled, final String time) {
            try {
                Object user = metaObject.getValue(userFiled);
                if(ObjectUtil.isNull(user)) {
                    setFieldValByName(userFiled, this.getUserId(), metaObject);
                }
            } catch (Exception ignored){}
            setFieldValByName(time, new Date(), metaObject);
        }

        /** 获取当前操作的用户 account */
        private Serializable getUserId() {
            try {
                AccountUser user = context.getBean(AccountUser.class);
                return user.getAccount();
            } catch (Exception ignored) {
                try {
                    Class<?> cls = Class.forName("com.bda.huijun.common.security.util.SecurityUtils");
                    Method method = ReflectUtil.getMethod(cls, "getAccount");
                    return (Serializable) method.invoke(null);
                } catch (Exception e) {}
                log.warn("accountUser not exist!");
            }
            return "";
        }
    };
}
```

## SpringBoot集成

## Nacos集成

### 注册中心

1. 添加依赖

```xml
<!-- nacos 服务发现 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

- 需要移除ribbon依赖

2. 添加服务注册发现注解 @EnableDiscoveryClient, 高版本可以不用加
3. 配置

```yml
spring:
  application:
    name: @artifactId@ #服务名
  cloud:
    nacos:
      discovery: #服务发现
        server-addr: 127.0.0.1:8848 #nacos地址
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yml
        import-check:
          enabled: true
  # SpringCloud 2021版本之后，需要用以下方式导入nacos的配置文件
  profiles:
    active: dev
  config:
    import: nacos:${spring.application.name}-${spring.profiles.active}.yml
```

4. 修改nacos配置文件 application.properties

```properties
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=123456
```

5. 创建nacos数据库，导入需要的表和数据
6. 打开网站
   - http://127.0.0.1:8848/nacos/index.html
   - 账号密码：nacos / nacos

### 配置中心

1. 添加依赖

```xml
<!-- nacos 服务配置中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!--配置中心不生效的话添加boot--> <!--高版本需要-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

2. 创建bootstrap.properties文件，配置服务名和配置中心地址

```yml
spring:
  application:
    name: market-member
  cloud:
      config:
        server-addr: 127.0.0.1:8848 #配置中心地址
        namespace: e63b0d76-dc0d-4521-a81c-2168c578e773 #命名空间
        group: dev #配置分组
        file-extension: yml
        ext-config: #加载多配置，配置上云管理
          - data-id: datasource.yml
            group: dev
            refresh: true
```

3. 到nacos配置中心添加
   - Data ID：模块名.yml
   - 配置格式：yml
4. 设置动态获取配置
   - @RefreshScope：在需要获取nacos配置的接口上添加，不需要重启服务就可以获取到
   - @Value：取值

## OpenFeign集成

1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. 创建远程调用接口，添加@FeignClient注解

```java
@FeignClient(contextId = "远程调用的接口",
        value = "接口路径",
        fallback = HjImServiceFallBack.class,
        configuration = SentinelFeignConfig.class
)
public interface FeignService{
    
}
```

3. 在需要远程调用的模块的启动类上添加@EnableFeignClients

```java
@EnableFeignClients(basePackages = "com.xz.market.common.openfeign.feign")
@SpringBootApplication
public class XXXApplication {
    public static void main(String[] args) {
        SpringApplication.run(XXXApplication.class, args);
    }
}
```

4. 注意

   - 如果使用了高版本的cloud的话需要修改依赖

     ```xml
     <!--feign模块的pom中添加-->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-loadbalancer</artifactId>
     </dependency>
     <!--nacos模块的pom中修改-->
     <dependency>
         <groupId>com.alibaba.cloud</groupId>
         <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
         <exclusions>
             <exclusion>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
             </exclusion>
         </exclusions>
     </dependency>
     ```

     - 老版本cloud使用的是ribbon，新版本使用的是loadbalancer

## Gateway集成

1. 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

- 需要排除web依赖
- 需要引入nacos依赖

2. 配置

```yml
spring: 
  cloud: 
    gateway:
      #路由转发断言、过滤、路径转换等等
      routes:      
        - id: admin_route
          uri: lb://renren-fast
          predicates:
            - Path=/admin/**
          filters:
            - RewritePath=/admin/(?<segment>.*), /renren-fast/$\(segment)
      #全局跨域配置
      globalcors:
        add-to-simple-url-handler-mapping: true
        cors-configurations:
          '[/**]':
            allowedOriginPatterns: "*"
            allowedMethods: "*"
            allowedHeaders: "*"
            allowCredentials: true
            maxAge: 360000 
```

## Netty集成

1. 依赖

```xml
<dependency>
	<groupId>io.netty</groupId>
    <artifacId>netty-all</artifacId>
    <version>4.1.39.Fin</version>
</dependency>
```

