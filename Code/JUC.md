---
title: JUC源码
toc: content
keywords: [juc, code]
---

## Unsafe

### getUnsafe()

```java
@CallerSensitive
public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass(); //getCallerClass方法需要启动类加载的类才可以调用
    if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
        throw new SecurityException("Unsafe");
    } else {
        return theUnsafe;
    }
}
```

### getAndAddInt()

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2); //检验
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

### 核心方法

```java
 //重新分配内存
 public native long reallocateMemory(long address, long bytes);  
   
 //分配内存  
 public native long allocateMemory(long bytes);  
   
 //释放内存  
 public native void freeMemory(long address);  
   
 //在给定的内存块中设置值  
 public native void setMemory(Object o, long offset, long bytes, byte value);  
   
 //从一个内存块拷贝到另一个内存块  
 public native void copyMemory(Object srcBase, long srcOffset, Object destBase, long destOffset, long bytes);  
   
 //获取值，不管java的访问限制，其他有类似的getInt，getDouble，getLong，getChar等等  
 public native Object getObject(Object o, long offset);  
   
 //设置值，不管java的访问限制，其他有类似的putInt,putDouble，putLong，putChar等等  
 public native void putObject(Object o, long offset);  
   
 //从一个给定的内存地址获取本地指针，如果不是allocateMemory方法的，结果将不确定  
 public native long getAddress(long address);  
   
 //存储一个本地指针到一个给定的内存地址,如果地址不是allocateMemory方法的，结果将不确定  
 public native void putAddress(long address, long x);  
   
 //该方法返回给定field的内存地址偏移量，这个值对于给定的filed是唯一的且是固定不变的  
 public native long staticFieldOffset(Field f);  
   
 //报告一个给定的字段的位置，不管这个字段是private，public还是保护类型，和staticFieldBase结合使用  
 public native long objectFieldOffset(Field f);  
   
 //获取一个给定字段的位置  
 public native Object staticFieldBase(Field f);  
   
 //确保给定class被初始化，这往往需要结合基类的静态域（field）  
 public native void ensureClassInitialized(Class c);  
   
 //可以获取数组第一个元素的偏移地址  
 public native int arrayBaseOffset(Class arrayClass);  
   
 //可以获取数组的转换因子，也就是数组中元素的增量地址。将arrayBaseOffset与arrayIndexScale配合使用， 可以定位数组中每个元素在内存中的位置  
 public native int arrayIndexScale(Class arrayClass);  
   
 //获取本机内存的页数，这个值永远都是2的幂次方  
 public native int pageSize();  
   
 //告诉虚拟机定义了一个没有安全检查的类，默认情况下这个类加载器和保护域来着调用者类  
 public native Class defineClass(String name, byte[] b, int off, int len, ClassLoader loader, ProtectionDomain protectionDomain);  
   
 //定义一个类，但是不让它知道类加载器和系统字典  
 public native Class defineAnonymousClass(Class hostClass, byte[] data, Object[] cpPatches);  
   
 //锁定对象，必须是没有被锁的
 public native void monitorEnter(Object o);  
   
 //解锁对象  
 public native void monitorExit(Object o);  
   
 //试图锁定对象，返回true或false是否锁定成功，如果锁定，必须用monitorExit解锁  
 public native boolean tryMonitorEnter(Object o);  
   
 //引发异常，没有通知  
 public native void throwException(Throwable ee);  
   
 //CAS，如果对象偏移量上的值=期待值，更新为x,返回true.否则false.类似的有compareAndSwapInt,compareAndSwapLong,compareAndSwapBoolean,compareAndSwapChar等等。  
 public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object x);  
   
 // 该方法获取对象中offset偏移地址对应的整型field的值,支持volatile load语义。类似的方法有getIntVolatile，getBooleanVolatile等等  
 public native Object getObjectVolatile(Object o, long offset);   
   
 //线程调用该方法，线程将一直阻塞直到超时，或者是中断条件出现。  
 public native void park(boolean isAbsolute, long time);  
   
 //终止挂起的线程，恢复正常.java.util.concurrent包中挂起操作都是在LockSupport类实现的，也正是使用这两个方法
 public native void unpark(Object thread);  
   
 //获取系统在不同时间系统的负载情况  
 public native int getLoadAverage(double[] loadavg, int nelems);  
   
 //创建一个类的实例，不需要调用它的构造函数、初使化代码、各种JVM安全检查以及其它的一些底层的东西。即使构造函数是私有，我们也可以通过这个方法创建它的实例,对于单例模式，简直是噩梦。 
 public native Object allocateInstance(Class cls) throws InstantiationException;  
```

## AtomicInteger

```java
//自增自减都是通过这个方法实现，
unsafe.getAndAddInt(this, valueOffset, 1 / -1);
```

```java
//复杂运算
public final int updateAndGet(IntUnaryOperator updateFunction) {
    int prev, next;
    do {
        prev = get();
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSet(prev, next));
    return next;
}
```

- IntUnaryOperator是一个函数式接口，使用lambda传值

## ConcurrentHashMap

### 常量

### 成员变量

```java
//默认为0，初始化时为-1，当扩容时为 - (1 + 扩容线程数)，当初始化或扩容后，作为下一次扩容的阈值
private transient volatile int sizeCtl;
//内部类，整个ConcurrentHashMap就是一个Node[]
static class Node<K,V> implements Map.Entry<K,V> {...}
//扩容时某个bin迁移完毕，用ForwardingNode作为旧table bin的头结点
static final class ForwardingNode<K,V> extends Node<K,V> {...}
//用在compute以及computeIfAbsent时，用来占位，计算完成后替换为普通Node
static final class ReservationNode<K,V> extends Node<K,V> {...}
//作为树的头结点，存储root和first，加了读写锁
static final class TreeBin<K,V> extends Node<K,V> {...}
//作为treebin的节点，存储parent，left，right
static final class TreeNode<K,V> extends Node<K,V> {...}
//哈希表
transient volatile Node<K,V>[] table;
//扩容时的新哈希表
private transient volatile Node<K,V>[] nextTable;
```

### 成员方法

```java
//获取Node[]中第 i 个Node
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
//cas修改Node[] 中第 i 个Node的值，c为旧值，v为新值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
//直接修改Node[] 中第 i 个Node的值，v为新值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

### 构造方法

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel){
    if(!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0){
        throw new IllegalArgumentException();
    }
    if(initialCapacity < concurrencyLevel){
        initialCapacity = concurrencyLevel;
    }
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```

### get

```java
public V get(Object key){
    ConcurrentHashMap.Node[] tab;
    ConcurrentHashMap.Node e;
    int n;
    //spread方法能确保返回结果是正数
    int h = spread(key.hashCode());
    if((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null){
        int eh;
        Object ek;
        //如果头结点已经是要查找的key
        if((eh = e.hash) == h){
            if((ek = e.key) == key || (ek != null && key.equals(ek))) return e.val;
        }
        //hash为负数表示该bin在扩容中或是treebin，这是调用find方法来查找
        else if(eh < 0){
            ConcurrentHashMap.Node p;
            return (p = e.find(h, key)) != null ? p.val : null;
        }
        while((e = e.next) != null){
            if(e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

### put

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

### 初始化表initTable()

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

### 扩容

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```

## LongAdder

### 成员变量

```java
//累加单元数组，懒惰初始化，24字节
transient volatile Cell[] cells;
//基础值，如果没有竞争，则用cas累加这个域
transient volatile long base;
//在cells创建或扩容时，置位1，表示加锁
transient volatile int cellsBusy;
```

```java
//防止缓存行 伪共享
@sun.misc.Contended
static final class Cell{
    volatile long value;
    //构造方法
    Cell(long x){ 
        value = x; 
    } 
    //用cas方式进行累加，prev表示旧值，next表示新值
    final boolean cas(long prev, long next){
        //使用UNSAFE提供的cas
        return UNSAFE.compareAndSwapLong(this, valueOffset, prev, next);
    }
}
```

- 一个==缓存行==(64字节)可以存下2个Cell，但是会有问题
  - 当有两个线程同时修改同一个缓存行中的cell时，一方的修改会导致另一方失败
  - ==@sun.misc.Contended==可以解决这个问题，它通过在对象的前后各加上128字节的padding，从而让cpu将对象预读至不同的缓存行，所以不会造成失效

## AQS



