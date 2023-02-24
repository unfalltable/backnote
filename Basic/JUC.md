---
title: JUC并发编程
toc: content
keywords: [juc]
---



## 基础

### 	创建线程的方式

- **Thread**类，**Runnable**接口
- **Callable**接口，有返回值，返回值通过 **FutureTask** 进行封装
  - 通过get方法得到返回值。
  - 实现Callable实现call()方法
- 实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的
- 通过实现接口来创建线程会好一点，继承Thread的开销太大

### FutureTask

```java
FutureTask<Integer> task = new FutureTask<>(() -> {
    //业务
});
new Thread(task, "线程别名").start();
//获取返回值
Integer result = task.get();
```

## 常用方法

### sleep

- 使当前线程进入阻塞状态，可通过interrupt()打断
- 睡眠结束后线程未必会立刻得到执行
- 不会释放锁

### Wait/Notify

- object的方法，与synchronized一起使用，会释放锁

- 调用 wait() 方法让当前线程进入waitSet中等待

- 通过 notify() / notifyAll() 唤醒等待中的线程

- 防止虚假唤醒

  - ```java
    synchronized(lock){
        while(条件不成立){
            lock.wait();
        }
        //逻辑代码
    }
    //另一个线程
    synchronized(lock){
        lock.notifyAll();
    }
    ```

### yield

- 调度执行其它同优先级的线程，如果没有，则继续运行当前线程

### interrupt

- **interrupt()**  设置打断标记为true
  - 睡眠中的线程被打断不会将标记设置为true
- **isInterrupted()** 判断线程是否被打断  不会清除标记
- **interrupted()**  判断线程是否被打断 会清除标记 即设置为false

### Join

- 在线程中调用另一个线程的 join() 方法，会将当前线程挂起，直到目标线程结束

### Park

- （LockSupport）park  
  - 让当前线程停下来
  - 如果打断标记为false则停止线程 ，如果打断标记为true则不会停止线程

### 两阶段终止模式

### 优先级

- **setPriority()** 为线程设置了新的优先级。默认是5
- **getPriority()** 返回线程的当前优先级。

## 线程状态

### Java API层面

- New - Runnable - Blocker（Waiting、Time_Wating）- Terminated

### 守护线程和非守护线程

- 守护线程就像是守护其他线程一样，其他的完成了他也就结束了
- 非守护线程都运行结束了，即使守护线程还没结束也会直接结束
- setDaemon(true)  将线程设置为守护线程
  - 垃圾回收器（gc）就是一个守护线程
  - Tomcat中的 Acceptor 和 Poller 线程也是守护线程

## Synchronized 锁升级机制

### 轻量级锁

- 这个锁对象虽然是多线程访问的，但是访问的时间是错开的，即没有发生竞争，那么这个锁可以自动优化成轻量级锁

### 偏向锁

- 轻量级锁在没有发生竞争时，每次‘重入时仍要执行CAS操作，耗费时间
- java6引入了偏向锁来优化此过程，只有在第一次CAS将线程id设置到对象头后，之后只要发现这个线程id是自己的，就不用进行CAS了
- 默认是开启偏向锁的，对象创建后，对象头中的markword的第三位是101，这是它的thread，epoch，age都为0，biased_lock为1，后接01
- 正常无偏下锁对象头中是hashcode、age、biased_lock:0 01
  - hashcode在第一次用到是才会赋值

- 偏向锁是延迟的，不会在启动时立即生效，可以通过设置 `-XX:BiasedLockingStartupDelay=0` 来禁用延迟
- -XX:-UseBiasedLocking 禁用偏向锁

#### 撤销偏向锁的情况

- 如果升级为轻量级锁会发生STW
- 如果对象调用了hashcode方法则会撤销偏向锁
- 当有第二个线程访问加锁方法也会撤销偏向锁
- 调用wait/notify 也会撤销偏向锁
- 偏向锁和hashcode是互斥存在的，轻量级锁的hashcode存储在线程栈帧的锁记录中，重量级锁的hashcode存储在Monitor对象中！
- 撤销偏向和重偏向都是批量进行的
- 当撤销偏向锁的次数超过了阈值，整个类的所有对象都会变的不可偏向

### 重量级锁（管程（Monitor锁)））

- 结构
  - 内部有 WaitSet、EntryList、Owner
- 流程
  - 一开始时，Monitor中的Owner为null
  - 当线程获取重量级锁时，Monitor中的Owner会指向当前线程
  - 当线程的锁未释放时，其他线程想获取锁就会进入EntryList中阻塞等待
    - 会先进行自旋，自旋一定次数后失败在进入队列等待
    - jdk 1.6 后自旋是自适应的，之前的线程自旋成功，会认为当前自旋成功率高会多自旋几次，反之则不自旋或少自旋
  - 当线程释放锁后，会唤醒EntryList中等待的线程来竞争锁，是非公平的

## JUC锁优化

### 锁膨胀

- 如果在尝试加轻量级锁的过程中，CAS操作无法成功，可能的一种情况是其他线程为此对象加上了轻量级锁，即有竞争，这是锁会膨胀成重量级

- 此时会申请一个monitor锁，让对象指向monitor地址
- monitor锁的Owner指向当前运行线程，新的进程会进入EntryList阻塞

### 自旋优化

- 在重量级锁发生竞争时，会通过自旋来进行优化，即发现对象已经上锁了会等待一段时间然后再次检查对象，如果对象释放了锁则可以获得对象锁，可以避免当前线程进入阻塞队列
- 有一定的次数，也有自适应的自旋

### 锁粗化

- 同一个线程的多次细化的加锁会优化成一个大的整体加锁，减少频繁的加锁释放锁

### 锁消除

- JIT即时编译器默认锁消除
- 当JIT判断锁对象不会被共享时会优化掉锁，即锁消除

## 对象

- 对象头（以32位虚拟机为例）

  - 普通对象（64bit）
    - MarkWord（32bit）+ KlassWord（32bit）
  - 数组对象（96bit）
    - MarkWord（32bit）+ KlassWord（32bit）+ ArrayLength（32bit）

- MarkWord

  - |                             结构                             |   状态   |
    | :----------------------------------------------------------: | :------: |
    |     hashcode:       \| age:     \| biased_lock: 0 \| 01      | 普通状态 |
    | thread:    \| epoch:     \| age:     \| biased_lock: 1 \| 01 | 偏向状态 |
    |   ptr_to_lock_record:                               \| 00    | 轻量级锁 |
    |       ptr_to_heavyweight_monitor:               \| 10        | 重量级锁 |
    |                            \| 11                             |  GC状态  |

## 线程安全

- 多个线程访问同一个共享变量就会出现线程安全问题
- private 和 final可以一定程度上提供线程安全
- 线程安全的方法组合起来就不能保证线程安全了

### 线程安全程度

- 不可变
  - final、String、枚举、Number部分子类（Long、Double、BigInteger、BigDecimal）、使用`Collections.unmodifiableXXX()` 包装的集合
- 绝对线程安全
  - 调用者都不需要任何额外的同步措施
- 相对线程安全
  - 有一定的安全，但有意外的情况，例如顺序的连续调用
  - 通过一些同步方法来保证安全
- 线程兼容
  - 对象本身不安全，需要通过一些同步方法来保证安全
- 线程对立
  - 即使使用了同步方法也无法保证线程安全

### 临界区

- 多线程读写同一个共享资源，这个共享资源就称为临界区

### 竟态条件

- 多个线程同时在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竟态条件

## JUC并发工具类

### 简介

- 工具类的api是原子性操作，即线程安全的
- 但是调用同一个工具类同的api则不一定是线程安全的

### 原子变量

- AtomicBoolean
- AtomicInteger
  - 底层使用了Unsafe的方法

- AtomicLong

- AtomicReference 原子引用
  - `AtomicReference<T> ref = new AtomicReference();`
  - `ref.set(对象);`
- AtomicMarkableReference
- AtomicStampedReference 版本号原子引用
  - `AtomicStampedReference<T> ref = new AtomicStampedReference();`
  - `int stamp = ref.getStamp();`

- AtomicIntegerArray
- AtomicReferenceFieldUpdater
- AtomicIntegerFieldUpdater
- AtomicLongFieldUpdater

### LongAdder

- 累加器

- 用一个原子整形来模拟锁机制


### ReentrantLock

- 基于AQS，内部有一个sync抽象静态内部类，它有两个实现类
  - NonfairSync 非公平同步器
    - CAS 判断state，0->1，失败执行acquire(1)->tryAcquire(1)

    - 失败返回false，进入队列，自旋

  - FailSync 公平同步器
    - 自旋获取锁，会判断在自己之前是否有线程（hasQueuedPredecessors方法）

    - 加入队列执行 addWaiter(Node.Exclusive) 自旋执行 acquireQueued()
      - 获取当前当前节点的前驱

      - 若为head且state = 0，把当前节点设置为head

- 可重入锁
  - `reentrantLock.lock()` 获取锁
  - `reentrantLock.unlock()` 释放锁

- 可打断锁
  - lock.lockInterruptibly()
  - 用 lock.interrupt() 打断阻塞
- 锁超时
  - lock.tryLock()    尝试获得锁
    - 可添加超时时间
  - 可打断
- 公平锁
  - 默认是不公平的
  - tryLock(超时时间，时间单位) 会根据是否开启公平锁执行相应的逻辑
  - tryLock() 默认是非公平实现
- 条件变量（多个）
  - synchronized中也有条件变量，就是重量级锁中的waitSet休息室，当线程阻塞时进入waitSet等待
  - ReentrantLock支持多条件变量，有多个waitSet，唤醒时按waitSet来唤醒

### ReentrantReadWriteLock

- 当读操作远远高于写操作时，这时候使用读写锁可以让读读并发，提高性能
- 类似数据库的意向共享锁
- 提供一个数据容器类，内部分别使用读锁保护数据的read方法，写锁保护write方法

- 读锁不支持==条件变量==
- 持有读锁时去获取写锁，会导致获取写锁永久等待
- 读写锁用的是同一个==Sycn同步器==，所以等待队列，state也是同一个
- 写锁占state的低16位，读锁是state高16位

### StampedLock

- jdk8加入，为了进一步优化读性能，在使用读锁和写锁时配合戳使用

  - ```java
    //读锁
    long stamp = lock.readLock();
    lock.unlockRead(stamp);
    //写锁
    long stamp = lock.writeLock();
    lock.unlockWrite(stamp);
    ```

- 乐观读，tryOptimistiicRead()方法，读取完毕后需要做一次戳校验，通过则表示这期间没有写操作，数据可以安全使用，如果没通过需要重新读取数据。

  - ```java
    long stamp = lock.tryOptimisticRead();
    //检验
    if(!lock.validate(stamp)){
        //不通过，锁升级
    }
    ```


- 不支持条件变量，不可重入

### Semaphore(信号量)

- 用来限制访问共享资源的线程上限，限流

- ```java
  Semaphore s = new Semaphore(3);
  //获得信号量
  s.acquire();
  ```

  - 最多只有三个线程可以执行，其他线程会被阻塞


### CountdownLatch

- 用来进行线程同步协作，等待所有线程完成倒计时，其中构造函数用来初始化等待计数值，await()用来等待计数归零，countDown()用来让计数减一

### CyclicBarrier	

- 循环栅栏，用来进行线程协作，等待线程满足某个计数，构造时设置计数个数，每个线程执行到某个需要同步的时刻，调用await方法进行等待，当等待的线程数满足计数个数时继续执行

- 栅栏数和线程数要一致

### ConcurrentHashMap

- 弱一致性
  - 求大小时
  - 读取时
  - 使用迭代器遍历时如果其他线程进行了修改，遍历的是旧数据

#### 结构

- 1.8（数组 + 链表）
- 1.7（segment段 + HashEntry数组 + 链表）

#### 构造方法

- 1.8
  - 懒惰初始化，第一次put数据时才会创建数组结构
  - 无参构造创建时，数组初始容量是16
  - 初始化时传了容量大小时，如果大于等于16 × 0.75 则扩容至下一个容量
    - 指定容量为12， 12 >= 16 × 0.75 则创建的数组大小为32
    - 指定容量为11， 11 < 16 × 0.75 则创建的数组大小为16
- 1.7
  - 初始化时就会创建segment数组和HashEntry数组的0号元素
    - 这个小数组作为其他数组的创建提供原型

### put

- 1.8
  - 尾插法
  - 会锁住链表头
- 1.7
  - 头插法

#### 初始化table方法

- size计算实际发生在put，remove改变集合元素的操作之中
  - 没有竞争发生，向baseCount累加计数
  - 有竞争发生，新建counterCells，向其中的一个cell累加计数
    - counterCells初始有两个cell
    - 如果计数竞争比较激烈，会创建新的cell来累加计数

- jdk8的构造方法实现了table的懒惰初始化，仅计算了table的大小，以后第一次调用时才会真正创建
  - jdk7一开始就创建segment数组，

#### 扩容

- 1.8 
  - 元素个数 >= 四分之三时扩容，与负载因子无关
  - 以链表为单位，从后往前迁移数据，处理完当前位置会在这个位置打上标记（forwardingNode）
  - 可以多个线程帮忙扩容，一个线程扩容16个位置
- 1.7
  - segement不能扩容，扩容的是HashEntry数组
  - 元素个数 > 容量 × 负载因子 时扩容，大小变为2倍

#### computeIfAbsent()

- 保证get()，put()组合使用的原子性
- 可以配合累加器LongAdder使用

### LinkedBlockingQueue

- 初始化链表

  - `last = head = new Node<E>(null);`

- **Dummy**节点用来占位，item为null

- 第一个节点入队

  - `last = last.next = node;`

- 出队：

  - ```java
    Node<E> h = head;
    Node<E> first = h.next;
    h.next = h;
    head = first;
    E x = first.item;
    first.item = null;
    return x;
    ```

- 加锁分析
  - 用了两把锁，同一时刻可以允许两个线程同时（生产者消费者）执行
- 线程安全分析（包括dummy节点）
  - 当节点总数大于2时，putLock保证的是last节点的线程安全，takeLock保证的是head节点的线程安全，两把锁保证入队和出队没用竞争
  - 当节点总数等于2时，仍然是两把锁锁两个对象，不会竞争
  - 当节点总数等于1时，take线程会被notEmpty条件阻塞，有竞争会阻塞

- 性能分析（与ArrayBlockingQueue比较）
  - Linked支持有界，Array强制有界
  - Linked是懒惰的，而Array需要提前初始化Node数组
  - Linked每次入队会生成新Node，而Array的Node是提前创建好的
  - Linked两把锁，Array一把锁

### ConcurrentLinkedQueue

- 两把锁，允许两个线程同时执行
- dummy节点用来占位，避免两把锁竞争
- 锁使用cas实现

### ThreadLocal

#### 简介

- 如果您需要使用强引用来持有 `ThreadLocal` 变量，可以考虑使用 `ThreadLocal` 的子类 `InheritableThreadLocal`，它的实现方式是通过强引用来实现的。

#### 结构

- 每个ThreadLocal会对每一个线程创建一个ThreadLocalMap，第一次使用才会创建
- 这个map的key是ThreadLocal实例本身，value是想要线程隔离的Object
- 初始容量是16，负载因子是2/3, >=阈值时扩容（两倍）
- 开放寻址法解决hash冲突
- key设置了弱引用，gc时会清除
  - get时遇到key为null值，会将其value置为null
  - set时遇到key为null值，会替换key-value，还会删除改位置一定范围内的key为null的键值，范围和map中的元素个数有关

- 推荐用remove删除，然后设置为static final 的，这样Gc就不会删除了，避免了内存泄漏

#### 方法

- `protected T initialValue()` ：返回当前线程据变量的初始值
- `public void set(T value)` ：将局部变量和当前线程绑定
- `pubilc T get()` ：获取当前线程绑定的局部变量
- `public void remove()` ：移除当前线程绑定的局部变量

#### 扩容

- 元素个数大于等于当前容量的2/3时扩容（向下取整），大小变为2倍

#### set方法

- 判断当前线程ThreadLocalMap是否为空，为空则创建
- 以ThreadLocal自己作为key，资源对象作为value插入到map中

- 采用的是开放寻址法解决hash冲突

#### get方法

- 获取当前线程的ThreadLocalMap，以ThreadLocal自身为key查找对应的value，返回的是entry
- 如果entry不为空则返回entry.value
- ThreadLocalMap为空或enrty为空，通过initialValue方法获取value的初始值，将ThreadLocal自身和初始值的value拼接成entry，插入ThreadLocalMap

#### remove方法

- 获取当前线程的ThreadLocalMap，不为空则移除对应的entey

#### initialvalue

- 给map中没有的对象赋初始值

### CopyOnWriteArrayList

- **CopyOnWriteArraySet**是它的马甲
- 底层实现采用写入时拷贝的思想，增删改会将数组拷贝一份，在拷贝的数据上更改，不影响其他线程的并发读，读写分离

---

## AQS

- AbstractQueueSynchronizer
- 是阻塞式锁和相关同步器工具的框架
- 用state属性来表示资源的状态（分独占模式和共享模式），子类需要定义如何维护这个状态，控制如何获取锁和释放锁
  - getState：获取state状态
  - setState：设置state状态
  - compareAndSetState：乐观锁机制设置state状态

- 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源
- 提供了基于FIFO的等待队列（双向链表），类似于Monitor的EntryList
- 条件变量来实现等待，唤醒机制，支持多个条件变量。类似于Monitor的WaitSet
- 用于自定义锁，创建一个类继承AbstractQueueSynchronizer，实现以下方法即可，不可重入
  - tryAcquire
  - tryRelease
  - tryAcquireShared
  - tryReleaseShared
  - isHeldExclusively

## 线程池(Executor)

`![20180823094933478](..//pic//20180823094933478.png)`

### ExecutorService

- 是Java中对线程池的实现，用于创建线程池

  - execute(Runnable r)
  - submit(Callable\<T> c)
    - 接收的是一个Callable的实现，返回一个future对象，用get()方法获得结果
    - future.get()方法会产生阻塞，有异常返回异常信息，保护性暂停模式
  - invokeAny(...)
    - 接收的是一个Callable的集合
    - 可添加超时时间
    - 执行这个方法不会返回Future，但是会返回所有Callable任务中最先执行完的任务的执行结果，其他任务都取消掉
  - invokeAll(...)
    - 接收的是一个Callable的集合
    - 可添加超时时间
    - 执行之后会返回一个Future的List，其中对应着每个Callable任务执行后的Future对象

- 使用完ExecutorService后需要关闭
  - ExecutorService.shutdown()
    - 不会立即关闭，但是它不再接收新的任务，直到当前所有线程执行完成才会关闭，所有在shutdown()执行之前提交的任务都会被执行
    
  - ExecutorService.shutdownNow()
    - 打断所有正在执行的任务和被提交还没有执行的任务。但是它并不对正在执行的任务做任何保证，可能它们都会停止，也有可能执行完成
    
    - 会返回阻塞队列中的任务集合

### Executors(工厂类)

- **newCachedThreadPool**
  - ![image-20220317174010061](..//pic//image-20220317174010061.png)
  - 创建一个全部都是救急线程的线程池，大小为Integer最大值，存活60s
  - 队列采用了**SynchronizedQueue**，这个队列没有容量，线程来取是做交换
  - **适用于任务数密集，耗时较短的任务**

- **newFixedThreadPool**
  - ![image-20220317173041361](..//pic//image-20220317173041361.png)
  - 创建一个固定大小的线程池，**全都是核心线程**，无需超时时间，**阻塞队列是无界**的，可以放任意数量的任务
  - **适用于任务量已知，相对耗时的任务**

- **newScheduledThreadPool**
  - 创建一个固定大小的线程池，支持定时及周期性任务执行。
  - schedule()  方法创建线程，延迟执行
  - scheduleAtFixedRate()  以固定的时间执行线程
  - scheduleWithFixedDelay()  每个任务直接间隔固定时间
- **newSingleThreadExecutor**
  - ![image-20220317174618710](..//pic//image-20220317174618710.png)
  - 线程数固定为1，多个任务排队执行，阻塞队列无界
  - 返回的是FinalizableDelegatedExecutorService,只对外暴露了自定义的方法，不能调用其他方法，运用到装饰器模式
  - 保证始终有一个线程在执行


### ThreadPoolExecutor(实现类)

![image-20220317165730143](..//pic//image-20220317165730143.png)

- 用int的高3位来表示线程池的状态，低29为表示线程数量

- ExecutorService的实现类,继承AbstractExecutorService

  ```java
  //构造方法
  public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize,
                            long keepAliveTime, TimeUnit unit,
                            BlockingQueue<Runnable> workQueue，
      					  ThreadFactory threadFactory,
                            RejectedExecutionHandler handler)
  ```
  
  - **corePoolSize**   线程池维护线程的最少数量（核心线程数）
    - 线程数量小于核心线程数
      - 即使有空闲线程，也要创建新线程来处理新任务
    - 线程数量等于核心线程数，并且阻塞队列未满
      - 将新任务放入阻塞队列
    - 核心线程数小于最大线程数，并且阻塞队列满了
      - 创建新线程来处理新任务,救急线程
    - 核心线程数等于最大线程数缓，并且阻塞队列满了
      - 通过handler所指定的策略来处理新任务
  - **maximumPoolSize** 最大线程数目
  - **keepAliveTime** 救急线程的生存时间
  - **unit** 时间单位
  - **workQueue** 阻塞队列
    - 是一个**BlockingQueue**，默认是**LinkedBlockingQueue\<Runnable>**
  - **threadFactory** 线程工厂
    - 起名字
  - **handler**   拒绝策略 （都满了）
    - AbortPolicy()（系统默认）
      - 抛出RejectedExecutionException异常
    - CallerRunsPolicy()
      - 让调用者执行任务
    - DiscardOldestPolicy()
      - 抛弃较早的任务
    - DiscardPolicy()
      - 抛弃当前的任务
        

### ScheduledThreadPoolExecutor(实现类)

- ExecutorService的实现类，继承ScheduledExecutorService

### ScheduledExecutorService(接口)

- 继承了ExecutorService
- 方法
  - schedule (Runnable task, long delay, TimeUnit timeunit)
    - 在delay延迟后运行线程task
    - 无返回值，不能知道线程的执行结果
  - schedule (Callable task, long delay, TimeUnit timeunit)
    - 在delay延迟后运行线程task
    - 有返回值
  - scheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit timeunit)
    - 周期性的调度task
    - period指的两个任务开始执行的时间间隔
    - task第一次执行的延迟根据 initialDelay 确定，之后每一次间隔 period
    - 不会出现一个以上任务同时执行
  - scheduleWithFixedDelay (Runnable task, long initialDelay, long period, TimeUnit timeunit)
    - period指的当前任务的**结束执行时间**到下个任务的**开始执行时间**

### AbstractExecutorService(接口)

### Fork / Join

- 体现的是一种**分治思想**，把递归交给线程做
- 默认会创建与cpu核心数大小相同的线程池

- 需要继承**RecursiveTask**\<V>（有返回结果）或**RecursiveAction**（无返回结果）
  - 覆盖重写**compute**()方法，实现任务拆分
- 创建**ForkJoinPoll**线程池，执行**RecursiveTask**实现类对象



