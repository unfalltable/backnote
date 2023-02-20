---
title: Spring源码
toc: content
keywords: [spring, code]u
---

## 后处理器

### AutowiredAnnotationBeanPostProcessor

### CommonAnnotationBeanPostProcessor

### ConfigurationPropertiesBindingPostProcessor

### ConfigurationClassPostProcessor

### MapperScannerConfigurer

## 注解

### @ComponentScan

- 先查找类上是否有该注解
- 获取包名路径，转换为路径
- 读取路径下的类
- 判断是否有@ComponentScan注解
  - 通过获取元数据判断是否直接或间接加了
- 获取BeanDefinition获取bean定义
- 向容器注册Bean

### @Bean

#### 简介

- 只能标注方法
- @Bean标注的方法不支持重载，只有参数最多的方法会执行

#### 源码

```java
//获取类的元信息
CachingMetadataReaderFactory Metadata = new CachingMetadataReaderFactory();
Metadata.getMetaReader();
//获取方法的元信息
```

```java
//获取Bean定义
BeanDefinitionBuilder beanDefinition = new BeanDefinitionBuilder();
beanDefinition.setFactoryMethodOnBean(方法名, 工厂名).getBeanDefinition();
//设置自动装配解析方法参数
context.getDefaultListableBeanFactory().registerBeanDefinition(方法名, Bean定义)
```

### @Mapper

#### 简介

- 扫描mapper包，获取其元信息，判断是否是注解
- 获取Bean定义，传入MapperFactoryBean类作为参数

#### 源码

### @Autowired

- Spring提供，先ByType后ByName，对象必须存在

### @Rsource

- 有JDK提供
- ByType和ByName，可以指定Name

### @EnableTransactionManagement

- @Import(TransactionManagementConfigurationSelector.class)
  - 切面
  - 拦截器
  - 注解解析

### @EnableAspectJAutoProxy

- @Import(AspectJAutoProxyRegister.class)
  - 

### @Configuration

- 被标注的类相当于一个工厂类，类中@Bean标注的方法相当于工厂方法
- 会给标注的类生成代理对象，目的是保证bean的单例特性

## 接口

### BeanFactory

### ApplicationContext

