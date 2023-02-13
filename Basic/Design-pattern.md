---
title: 设计模式
keywords: [basic]
---

## 类图

## 设计模式的七大原则

- 单一职责原则
- 接口隔离原则
  - 客户端不应该被迫依赖于它不使用的方法
  - 一个类对另一个类的依赖应该建立在最小的接口上

- 依赖倒转原则
  - 高层模块不依赖低层模块，两者都应该依赖其抽象
  - 对抽象进行编程，而不是对实现进行编程

- 里氏替换原则
  - 子类可以扩展父类的功能，尽量不要重写父类的方法

- 开闭原则（ocp）
  - 对扩展开放，对修改封闭

- 迪米特法则
  - 如果两个实体最好不要直接通信，应该通过第三方通信降低耦合

- 合成复用法则
  - 尽量使用组合或聚合，其次再考虑继承


## 创建型

### 单例模式

#### 饿汉式

- ```java
  public class Hungry {
      public static final Hungry HUNGRY = new Hungry();
      private Hungry(){}
  }
  ```

- ```java
  //枚举  z
  public enum  Hungry {
      INSTANCE;
  }
  ```
  
- ```java
  public class Hungry {
      public static final Hungry HUNGRY;
      static {
          /*一般在这个静态代码块中加载配置*/
          HUNGRY = new Hungry();
      }
      private Hungry(){}
  }
  ```

#### 懒汉式

- ```java
  public class Lazy {
      private static Lazy lazy = null;
      private Lazy(){}
      //锁的粒度比较粗
      public static synchronized Lazy getInstance(){
          if (lazy == null)
              return lazy = new Lazy();
          return lazy;
      }
  }
  ```
  
- ```java
  //双重检查锁
  //如果实现了序列化接口，反序列化时会创建新的对象，需要写一个readResovle方法来避免
  public final class Lazy implements Serializable{
      //防止指令重排和使用到半初始化状态的对象
      volatile private static final Lazy INSTANCE = null;
      private Lazy(){
          //防止反射破坏单例
          synchronized{
              if(INSTANCE != null) {
                  throw new RuntimeException("不能创建多个单例对象");
              }
          }
      }
      public static Lazy getInstance(){
          //提高效率，当有实例了不用再去争抢锁
          if(INSTANCE == null) {
              synchronized (Lazy.class) {
                  //防止阻塞队列的线程重新创建对象
                  if (INSTANCE == null)
                      return INSTANCE = new Lazy();
              }
          }
          return INSTANCE;
      }
      //防止反序列化时返回新的对象
      public Object readResovle(){
          return INSTANCE;
      }
  }
  ```
  
- ```java
  //内部类
  public final class Lazy {
      private Lazy(){}
      private static class inner{
          private static final Lazy LAZY = new Lazy();
      }
      public static Lazy getInstance(){
          return inner.LAZY;
      }
  }
  ```

#### 饿汉模式和懒汉模式的区别

- 饿汉式是线程安全的，懒汉式如果在创建实例对象时不加上synchronized则会导致对对象的访问不是线程安全的。
- 懒汉式是延时加载，在需要的时候才创建对象，
- 饿汉式在jvm虚拟机启动的时候就会创建，
- 饿汉式它是加载类时创建实例，如果是一个工厂模式、缓存了很多实例、那么就得考虑效率问题，因为这个类一加载则把所有实例不管用不用都一块创建了
- 懒汉模式在运行的时候获取对象比较慢，但是加载类的时候比较快,
- 饿汉模式是在运行的时候获取对象较快，加载类的时候慢。

#### 反序列化和反射会破坏单例模式

- 添加readResovle方法避免反序列化产生新对象

  - ```java
    public Object readResovle(){
    	return INSTANCE;
    }
    ```

- 在构造方法中判断是否是第一次创建对象来避免产生新对象

  - 不是第一次创建对象就抛异常

  - ```java
    //防止反射破坏单例
    synchronized{
        if(INSTANCE != null) {
            throw new RuntimeException("不能创建多个单例对象");
        }
    }
    ```

### 工厂模式

以咖啡店为例子

#### 简单工厂模式（不是一种设计模式）

- 将咖啡店类和咖啡类解耦，将具体业务放在工厂类中实现
  - 封装了创建对象的过程，避免了修改咖啡店类代码，更容易扩展
  - 虽然解除了咖啡店类和咖啡类的耦合，但是产生了新的耦合，即工厂类和咖啡类的耦合，修改业务时依然需要修改工厂类代码，违背了开闭原则

- 可以把工厂类中的业务方法定义为静态的，方便调用

#### 工厂方法模式

- 

#### 抽象工厂模式

- 

### 建造者模式

- 用于构造复杂对象
- 包含四个角色
  - 抽象建造者类（Builder）：接口
  - 具体建造者类（ConcreteBuilder）：实现Builder接口
  - 产品类（Product）：要构造的复杂对象
  - 指挥者类（Director）：调用ConcreteBuilder来创建Product复杂对象

## 结构型

### 代理模式

### 组合模式

#### 概念

- 部分整体
- 定义个各层次对象的共有方法和属性，预先定义一些默认的行为
- 定义非叶子节点的行为，存储子节点，组合非叶子节点和叶子节点形成树
- 定义叶子节点下无分支
- 有透明组合模式和安全组合模式
  - 透明组合模式是标准模式
    - 不安全，叶子节点不应该具有添加、删除等功能
  - 安全组合模式
    - 安全，不同的节点提供其对应的方法，实现复杂

## 行为型

### 模板方法

#### 概念

- 大致的实现确定，少部分不确定的，就可以将这部分设计为模板
- 这部分将推迟实现，提高代码复用性，实现反向控制

## 代码设计

### 两阶段终止模式

- 一个线程t1 优雅的停止另一个线程t2，给t2一个料理后事的机会
- 如果直接终止t2会导致t2锁住的资源无法释放，导致死锁