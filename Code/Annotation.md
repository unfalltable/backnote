---
title: 注解源码
keywords: [annotation, code]
---

## @CallerSensitive

- 该注解标注的方法为危险方法，调用该注解标注的方法的对象必须也需要有该注解，且改对象必须有启动类加载
- 开发者自己写的@CallerSensitive不可被识别即无法调用该危险方法
- 可以通过jvm参数 `-Xbootclasspath/a: path` 伪装为启动类

## @FunctionInterface

- 接口中只有一个抽象方法，即可搭配lamda表达式使用