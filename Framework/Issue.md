---
title: 常见问题
keywords: [issue]
---

## 跨域

- 因为浏览器不能执行其他网站的脚本，它是由浏览器的==同源策略==导致的，是浏览器对js的安全限制

  - 协议、域名、端口都要相同，一个不同即发生跨域

- ==非简单请求==需要先发送一个预检请求，服务器响应允许跨域后才能发送真实请求

  - post请求

- 解决方案

  1. 使用nginx

     - nginx会和其他服务在同一个域，所以不会出现跨域问题

     - 前提是前端项目、网关、后端服务等需要在一个域

  2. 配置当前请求允许跨域，添加响应头

     1. 可以在gateway的配置文件中添加全局过滤器
     2. 也可以编写配置类


## 非spring模块下的类加入spring容器

- 例如将全局异常处理写在了common包下，但是common包不属于spring管理的包，所以不起作用的情况

- 解决方法

  1. 在非spring模块的包里创建一个类，将需要加入容器的类通过@Import导入这个类

     ```java
     @Configuration
     @Import(value = {
             GlobalExceptionHandler.class
     })
     @EnableConfigurationProperties
     public class WebExtendConfiguration {
     }
     ```

  2. 在resource下创建META-INF文件夹，文件夹下创建 spring.factories

     ```properties
     org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.xz.market.common.web.WebExtendConfiguration
     ```

     

