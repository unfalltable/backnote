---
title: Spring源码
toc: content
keywords: [spring, code]u
---

## IOC



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

### @Autowired

### @Rsource

### @EnableTransactionManagement

### @EnableAspectJAutoProxy

### @Configuration

## 接口

### BeanFactory

```java
```

### ApplicationContext

```java

```

