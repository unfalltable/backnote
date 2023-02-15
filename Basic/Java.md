---
title: Java
keywords: [Java, basic]
---

## 面向对象

### 继承、封装、多态

- 继承
  - 父子类关系 is-a
  - 从已有类派生出新类，派生类有父类的属性和方法，能对其进行扩展，父类中的private属性不会被继承
  - 提高了代码的复用性，可维护性，但耦合度高
- 封装
  - 将数据和操作封装在一起，成为一个不可分割的整体
  - 隐藏了类的内部实现机制，对外暴露访问方法，保护数据
  - 提高了代码的复用性，安全性，减少耦合
- 多态
  - 是类与类之间的关系，有编译期多态和运行期多态
  - 需要继承、封装、重写、父类引用指向子类对象，用父类做参数，具体子类实现时再确定
  - 提高了代码的可扩展性，但不能使用子类特有方法

### 类图

泛化关系

实现关系

聚合关系

组合关系

关联关系

依赖关系

## 关键字

#### transient

- 修饰的字段不参与序列化
- 只能修饰变量
- 当这个变量可以通过其他变量来获得时就可以不参加序列化，即添加transient修饰
  - 例如：三角形的长，宽，面积，则面积可以不参与序列化
  - Logger对象也不需要

#### static

- 修饰的静态变量是不参与序列化的

## 接口

- 接口可以有默认方法

## 方法

- 静态方法
- 抽象方法
- final方法

## 异常

### 自定义异常

- 需要继承RuntimeException
  - 定义错误码和错误消息，还有各种参数的构造函数，包含异常信息，在抛异常的时候传入

## 序列化

- 就是将对象转换成字节序列，实现持久化和网络传输
- 方式
  - 实现Serializable接口
  - 实现Externalizable接口
    - 实现writeExternal()
    - 实现readExternal()
- transient可以防止属性序列化，因为序列化可能会破坏单例

## Unsafe

### 知识点

- Unsafe提供了非常底层的方法，包括操作内存，线程调度、CAS、系统、内存屏障、对象操作、Class

- 使Java拥有了类似c语言一样的指针，所以不安全

- LookSupport中的unpark也是调用的unsafe的方法

- Unsafe不能直接调用

  - 通过反射获得

    - ```java
      Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
      theUnsafe.setAccessible(true);
      Unsafe unsafe = theUnsafe.get(null);
      ```

  - Unsafe提供了一个静态方法getUnsafe可以获取Unsafe实例，但是必须使用引导类加载器加载Unsafe类才能使用

    - 命令行中输入：`java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径 `


### 操作

- 操作内存
  - 主要包含堆外内存的分配、拷贝、释放、给定地址值操作等方法
  - 为什么要使用堆外内存
    - 对垃圾回收停顿的改善。由于堆外内存是直接受操作系统管理而不是JVM，所以当我们使用堆外内存时，即可保持较小的堆内内存规模。从而在GC时减少回收停顿对于应用的影响。
    - 提升程序I/O操作的性能。通常在I/O通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存。
  - 应用：
    - DirectByteBuffer
      - 在NIO框架中通常作为缓冲池使用，内部的内存分配逻辑都由Unsafe提供
      - `base = unsafe.allocateMemory(内存大小)` ：分配内存，返回基地址
      - `unsafe.setMemory(base, size, (byte) 0)` ：设置内存，内存初始化
      - `cleaner = Cleaner.create(this, new Deallocator(base, size, cap))` ：跟踪DirectByteBuffer对象的垃圾回收，以实现当DirectByteBuffer被垃圾回收时，分配的堆外内存一起被释放
        - Cleaner继承自虚引用，当DirectByteBuffer对象仅被Cleaner对象引用时，gc时会将DirectByteBuffer对象回收，Cleaner对象会调用clean() 方法来进行堆外内存的释放。
- CAS
  - AtomicInteger 使用 unsafe提供的CAS
  - Unsafe只提供了 compareAndSwapObject、compareAndSwapInt、compareAndSwapLong 方法
  - 赋值阶段源码中都是通过do-while判断（通过反编译查看源码）
  - 这些方法都是在unsafe.cpp中有具体的实现
    - 如果是多处理器，需要添加lock前缀（Lock#信号），内存屏障
    - 这里的lock前缀就是使用了处理器的**总线锁**(最新的处理器都使用**缓存锁**代替总线锁来提高性能)
      - 总线锁：将cpu和内存之间的通信锁住，其他cpu不能操作内存中的数据，开销比较大
      - 缓存锁：使用**缓存锁定**，将原子操作放在cpu缓存中进行
        - 缓存锁定：发生共享内存的锁定时，不会锁内存，而是修改内存地址，通过**缓存一致性**保证原子性
        - 缓存一致性：
          - 如果这块内存被两个以上的处理器缓存，会阻止这块内存区域的同时修改
          - 当处理器对缓存中的数据修改后，会通知其他处理器删除其内部这块内存的缓存，或者从主内存中重新读取
        - 如果操作的数据跨多个缓存行时，处理器会使用总线锁
- 线程调度
  - 线程挂起、恢复、锁机制
  - park、unpark、monitorEnter、monitorExit、tryMonitorEnter
- Class相关
  - 提供Class和它的静态字段的操作相关方法，包含静态字段内存定位、定义类、定义匿名类、检验&确保初始化等
  - 获取属性在内存中的位置
  - 获取数组第一个元素在内存中的偏移地址，还可以获取下标的增量地址
- 系统相关
  - addressSize：返回系统指针的大小。返回值为4（32位系统）或 8（64位系统）
  - pageSize：内存页的大小，此值为2的幂次方
- 内存屏障
  - loadFence：禁止load操作重排序
  - storeFence：禁止store操作重排序
  - fullFence：禁止load、store操作重排序
- 对象操作
  - 主要包含对象成员属性相关操作及非常规的对象实例化方式等相关方法
  - Unsafe的allocateInstance实现对象的实例化，保证在目标类无默认构造函数时，反序列化不够影响

## 集合


### ArrayList

#### 知识点

- 底层是数组，通过复制数组实现增删
  - 增删尾部快，其他位置慢
  - 按索引查询快，按内容查询和LinkedList差不多
- 可以利用cpu缓存，局部性原理
- 涉及到size的修改都会使 modCount++

#### 源码


- 扩容

  - 使用无参构造方法创建时默认初始容量是0，使用add方法添加元素就触发扩容
  - 第一次扩容容量变为10，当存满了会再触发扩容，默认是1.5倍
  - 扩容是创建一个新数组，再将原数组内容拷贝到新数组
  - 扩容后容量 = 扩容前容量 > 1 +扩容前容量 (即1.5倍)
  - 因为每次扩容都比较小，所以要尽量避免频繁的扩容，当预知要保存多少元素时，就在创建时指定ArrayList的初始大小，或者使用ensureCapacity手动扩容

  - ```java
    //数组最大值 = 整型最大值 - 8
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0 : DEFAULT_CAPACITY;
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); //1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //得到最大的数组容量
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // 太大超过int最大值就变成负数了
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
    }
    ```


- add(T e)、addAll(Collection<? extends T> c)
  
  - 检查剩余空间是否需要扩容，modCount++
  - 添加到数组末尾
  - 更新size
  
- add(int index, T element)、addAll(int index, Collection<? extends T> c)

  - 检查下标是否越界
  - 检查剩余空间是否需要扩容，modCount++
  - 原数组的值复制到新数组，插入区间前后的元素复制到新数组不包含插入区间的位置
  - 然后在插入区间插入新增的元素
  - 更新size

- set(int index, T element)

  - 检查下标是否越界
  - 保存老值，新值插入index位置
  - 返回老值

- get(int index)

  - 检查下标是否越界
  - 返回指定下标的值，需要注意类型转换

- remove(int index)

  - 检查下标是否越界，modCount++
  - 保存老值
  - 数组从index下标之后的元素都往前移（通过复制）
  - 为了让GC起作用，最后一个位置会显式赋`null`值
  - 返回老值

- remove(Object o)

  - 删除第一个满足`o.equals(elementData[index])`的元素

-  trimToSize() 将底层数组的容量调整为当前列表保存的实际元素的大小

  - modCount++
  - 判断 size 是否 小于数组大小

    - 如果size == 0 则为空数组
    - size != 0 则 将数组复制到一个size = 数组长度的新数组

- indexOf(Object o)、lastIndexOf(Object o)

  - 判断 o 是否为 null
  - 遍历数组找下标

    - o 为 null 则用 == 判断
    - o 不为 null 则用 equals 判断

  - 找到返回下标，找不到返回-1

- Fail-Fast机制

  - 通过记录modCount参数，迭代器遍历的时候遇到并发的修改时会报错


### LinkedList

#### 知识点

- 实现了 List 接口和 Deque 接口，当作是一个双端队列，或者是栈，但ArrayDeque有着更好的性能
- 底层是双向链表结构
  - 查询首尾元素速度快，其余位置慢
  - 按下标去增删速度快，按内容去遍历增删慢
- 内部维护了First 和 Last ，包含了大量操作首尾元素的方法
- 检索下标时判断 index 靠前还是靠后，决定从后往前，还是从前往后
- index方法会判断对象是否为空，根据空或者非空来找下标
- 占用内存多
- 可以存null
- 如果需要多个线程并发访问，可以先采用`Collections.synchronizedList()`方法对其进行包装

#### 源码

- getFirst() 、getLast()
  - 定义个一个常量来接收 first 或者 last
  - 如果指向 null，抛`NoSuchElementException`异常
  - 有值就返回

- get(int index)
  - 检查下标是否合法，是否越界
  - 返回 node(index).item
- set(int index, T element)
  - 判断下标是否合法，是否越界
  - Node x = node(index) 
  - T oldVal = x.item
  - x.item = element
  - return  oldVal 
- remove(Object o)
  - 判断 o 是否为 null
  - 遍历链表找值符合的节点

    - o 为 null 则用 == 判断
    - o 不为 null 则用 equals 判断
  - 找到则用 unlink() 删除节点，成功返回true

- remove(int index)
  - 检查下标是否越界
  - 用 unlink( node(index) ) 删除节点

- removeFirst()、removeLast()
  - 定义一个常来接收 first / last
  - first / last 为null 抛NoSuchElementException异常
  - 不为空则调用 unlinkFirst() / unlinkLast() 

- unlink(Node<E> x)
  - 定义三个常量保存节点的值 element、前驱节点 prev、后继节点 next
  - 先处理前驱，在处理后继，注意防止空指针异常
    - 判断 prev 是否为null
      - 为 null 说明是第一个节点要删除，直接 first = next
      - 不为 null，则 prev.next = next，x.prev = null
    - 判断 next 是否为null
      - 为 null 则说明是最后一个节点要删除，直接 last = prev
      - 不为 null，则 next.prev = prev，x.next = null
  - 将x的值置为 null。更新 size，modCount++
  - 返回被删除的值 element
- unlinkFirst(Node<E> f)
  - 定义两个常量保存节点的值 element，后继节点 next
  - 将 f 的值和后继节点置为 null
  - 然后 first = next
  - 判断 next 是否等于 null
    - 为 null，则 last = null
    - 不为 null，则 next.prev = null

  - size--
  - modCount++
  - 返回被删除的值 element

- unlinkLast(Node<E> l)
  - 定义两个常量保存节点的值 element，前驱节点 prev
  - 将 l 的值和前驱节点置为 null
  - 然后 last = prev
  - 判断 prev 是否等于 null
    - 为 null，则 first = null
    - 不为 null，则 prev.next = null
  - size--
  - modCount++
  - 返回被删除的值 element
- add(T e)
  - 调用 linkLast(e)
  - 返回 true
- add(int index, T element)
  - 判断下标是否合法，是否越界
  - 如果 下标 = size，说明是在链表最后插入，调用 linkLast(element)
  - 下标 != size，则调用 linkBefore(element, node(index))
- linkLast(T e)
  - 定义两个常量保存 尾节点 l，新节点 newNode（根据 e 创建）
  - last = newNode
  - 判断 l 是否为 null
    - 为 null，说明链表中无节点，所以 first = newNode
    - 不为 null，说明链表中有节点，所以 l.next = newNode
  - size++
  - modCount++
- node(int index) 
  - 判断 index 在链表前半段还是后半段，通过比较 size >> 1
    - 前半段：从前往后遍历找
      - x = first
      - for x = x.next
    - 后半段：从后往前遍历找
      - x = last
      - for x = x.prev

- addAll(Collection<? extends T> c)

  - 调用addAll(size, c) 实现

- addAll(int index, Collection<? extends T> c)

  - 判断下标是否合法
  - 将集合c转化为Object数组 a
  - 得到 a的长度 numNew，判断是否等于0，等于0返回false
  - 创建两个节点指针 pred、succ
  - 判断 index 是否等于 size
    - 等于，相当于在链表最后添加，则 succ = null，pred = last
    - 不等于，则 succ = node(index)，pred = succ.prev

  - 遍历 for (Object o : a) 
    - T e = (T) o      强制类型转换
    - 根据值和前驱 pred 创建节点 newNode 
    - 判断 pred 是否为 null
      - 为 null，则 first = newNode
      - 不为 null，pred.next = newNode
    - pred = newNode
  - 判断 succ 是否为 null
    - 为 null，则 last = pred
    - 不为 null，则 pred.next = succ，succ.prev =  pred 
  - size += numNew
  - modCount++
  - return true

- clear()

  - 遍历
    - 先取得当前节点的next节点
    - 然后当前节点的所有属性置为 null
    - x = next
  - first = last = null
  - size = 0
  - modCount++ 

- indexOf(Object o)

  - 定义一个index = 0
  - 判断 o 是否为 null 
    - 为 null，遍历时走 == 判断
    - 不为 null，遍历时走 equals判断
  - 找到返回index，找不到返回 -1

- lastIndexOf(Object o)

  - 定义一个index = size
  - 从后往前遍历，也是先判断 o 是否为 null 走不同的判断逻辑
  - 找到返回index，找不到返回 -1

- peek()、peekFirst()、peekLast() 

  - 定义一个常量 f 接收 first
  - 如果 f 为 null 返回 null，不为 null 返回 f.item

- poll()、pollFirst()、pollLast()

  - 定义一个常量 f 接收 first
  - 如果 f 为 null 返回 null，不为 null 返回 unlinkFirst(f) 

- removeLastOccurrence(Object o)

  - 判断 o 是否为 null 
    - 为 null，遍历时走 == 判断
    - 不为 null，遍历时走 equals判断

  - 找到用 unlink(x) 并返回true，找不到返回 false

- element()

  - 调用 getFirst()

- remove()

  - 调用 removeFirst()

- offer(T e)

  - add(e)

- offerLast(T e)

  - 调用 addLast(e)
  - 返回 true

- push(T e)

  - 调用 addFirst(e)

- pop()

  - 调用 removeFirst()

- removeFirstOccurrence(Object o)

  - 调用 remove(o)

### ArrayDeque

#### 知识点

- Stack 是类，Queue 是接口
- Java不推荐用Stack了，所以想用栈或者队列首先考虑ArrayDeque
- Deque双端队列，既可以当作栈使用，也可以当作队列使用，*ArrayDeque*和*LinkedList*是*Deque*的两个通用实现
- 初始容量 8，扩容 x 2
- 底层是循环数组，维护了一个 head 和 tail，非线程安全的，不可存null
  - head 指向第一个元素
  - tail 指向最后一个元素的后一位

#### 源码

- addFirst(T e)
  - 判断 e 是否为null，为 null 就抛异常
  - 就是在 head 之前插入，插入后head-1
    - elements [ head = (head - 1) & (elements.length - 1) ] = e
  - 判断是否需要扩容， 如果 head == tail 就需要扩容
- addLast(T e)
  - 判断 e 是否为null，为 null 就抛异常
  - 就是在 tail 的位置插入，插入后 tail+1
    - elements[tail] = e
  - 判断下标是否越界，越界就要扩容
    - ( tail = ( tail + 1 ) & ( elements.length - 1) ) == head，为 true 就要扩容
- doubleCapacity()
  - 只有在 head == tail 的时候才需要扩容，源码中用 assert 判断，不满足条件则抛AssertionError异常
  - 保存 head 的值（下标） p，数组的长度 n
  - 计算 head 右边元素的个数 r = n - p
  - 计算扩容后的数组大小 newCapacity = n << 1，如果 newCapacity < 0 则抛IllegalStateException异常 （就是太大了，超出in最大值就会变负数）
  - 根据 newCapacity 创建一个新数组
  - 将旧数组的值复制到新数组 a，分两步复制
    - 先复制 head 右边的数据到新数组
    - 再复制 head 左边的数据到新数组
  - 将 elements 替换为新数组 a
  - head = 0，tail = n
- pollFirst()
  - 取得 head 位置的值 result
  - 判断是否为 null，为 null 则说明数组为 null ，直接返回 null
  - 将 head 位置的值置为 null
  - 更新 head
    - head = (head + 1) & (elements.length - 1)
  - 返回 result
- pollLast()
  - 获得 tial 前一位的值 t
    - t = (tail - 1) & (elements.length - 1)
  - 取得 t 位置上的值 result
  - 判断  result  是否为 null，为 null 则说明数组为 null ，直接返回 null
  - 将 t 位置上的值置为 null
  - 更新 tail
    - tail = t
  - 返回 result
- peekFirst()
  - 返回 elements[head] 的值
- peekLast()
  - 返回 elements[(tail - 1) & (elements.length - 1)] 的值

### PriorityQueue

#### 知识点

- 优先队列，实现了Queue 接口，不可存null
- 作用是能保证每次取出的元素都是队列中权值最小的
- 数组中的元素是按层序遍历树排列的
- 初始容量11，扩容
  - 申请一个更大的数组，并将原数组的元素复制过去
  - 原始容量 < 64，则新队列长度 = 原始容量 * 2 + 2
  - 原始容量 > 64，则新队列长度 = 原始容量 +原始容量 >> 1
  
- 如果扩容超过数组的最大长度，则按数组的最大长度进行扩容
- 具体是通过小顶堆，可以用数组实现
  - leftNo = parentNo * 2 + 1
  - rightNo = parentNo * 2 + 2
  - parentNo = ( nodeNo - 1 ) / 2
- 时间复杂度：O(LogN)

#### 源码

- add(T e)、offer(T e)

  - ```java
    public boolean offer(T e) {
        if (e == null)//不允许放入null元素
            throw new NullPointerException();
        modCount++;
        int i = size;
        if (i >= queue.length)
            grow(i + 1);//自动扩容
        size = i + 1;
        if (i == 0)//队列原来为空，这是插入的第一个元素
            queue[0] = e;
        else
            siftUp(i, e);//调整
        return true;
    }
    ```

- siftUp(int k, T x)

  - ```java
    private void siftUp(int k, T x) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;//parentNo = (nodeNo-1)/2
            Object e = queue[parent];//取得父节点的值e
            if (comparator.compare(x, (T) e) >= 0)//调用比较器的比较方法
                break;// x 大于 e 则不用交换了，直接退出
            queue[k] = e;
            k = parent;
        }
        queue[k] = x;
    }
    ```

- element()、peek()

  - 如果 size =  0，返回 null
  - 返回 (T) queue[ 0 ]

- remove()、poll()

  - ```java
    public T poll() {
        if (size == 0)
            return null;
        int s = --size;
        modCount++;
        T result = (T) queue[0];//0下标处的那个元素就是最小的那个
        T x = (T) queue[s];
        queue[s] = null;
        if (s != 0)
            siftDown(0, x);//调整
        return result;
    }
    ```

- siftDown(int k, T x)

  - 得到 size 的一半 half

  - 循环 k < half 
    - 得到左孩子的下标  child = (k << 1) + 1
    - 得到左孩子下标对应的值 c
    - 得到右孩子的下标 right = child + 1
    - 如果 right < size 并且 c > queue[right]，则 c = queue[child = right]，就是找左右孩子小的那一个
    - 如果 x < c 退出循环
    - queue[k] = c，因为 x > c
    - 交换后从 k = child
    
  - queue[ k ] = x

  - ```java
    private void siftDown(int k, T x) {
        int half = size >>> 1;
        while (k < half) {
        	//首先找到左右孩子中较小的那个，记录到c里，并用child记录其下标
            int child = (k << 1) + 1;//leftNo = parentNo*2+1
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                comparator.compare((T) c, (T) queue[right]) > 0)
                c = queue[child = right];
            if (comparator.compare(x, (T) c) <= 0)
                break;
            queue[k] = c;//然后用c取代原来的值
            k = child;
        }
        queue[k] = x;
    }
    ```

- remove(Object o)

  - 遍历数组找到符合值的下标 i = indexOf(o)
  
  - 如果 i = -1 ，返回 false
  
  - 得到数组最后一个元素的下标 s = --size
  
  - 如果 s == i，说明要删除的是最后一个元素，则 queue[i] = null
  
  - s != i
    - 取出数组最后一个元素的值 moved
    - queue[ s ] = null
    - siftDown(i, moved)
  
  - 返回 true
  
  - ```java
    public boolean remove(Object o) {
    	//通过遍历数组的方式找到第一个满足o.equals(queue[i])元素的下标
        int i = indexOf(o);
        if (i == -1)
            return false;
        int s = --size;
        if (s == i) //情况1
            queue[i] = null;
        else {
            T moved = (T) queue[s];
            queue[s] = null;
            siftDown(i, moved);//情况2
            ......
        }
        return true;
    }
    ```

### Vector

- 知识点
  - Vector是同步的 单线程速度慢 早期使用Elements遍历集合
  - Elements的返回值是Enumeration<E>（向量/枚举）
    - Enumeration接口是迭代器前身
      - hasMoreElements()   判断集合是否有下一个元素
      - nextElement()       取出集合元素

### HashSet

- 知识点

  - 只能通过迭代器和增强for循环遍历集合

  - 不同步，不可存储重复元素，无序的集合，底层是一个哈希表结构(查询速度非常快)

  - 存储自定义类型的元素
    - 因为Set集合保证元素唯一，防止出现相同元素必须重写hashCode()和equals()方法
  - 对HashMap进行了一个封装，适配器模式

  - 内部都是调用的hashmap 的方法
  - key放存储的值，value存储的一个静态常量

### HashMap

#### 知识点

- 无序，存取元素顺序可能不一致

- hashmap是懒惰创建数组的，第一次调用put方法执行putVal时才创建数组

- 默认初始长度为16

- hashMap的 key 和 value 可以为null

- 底层数据结构
  - 1.7是数组+链表
  - 1.8是数组+链表 or 红黑树


- 扩容
  - 扩容2倍
    - 1.7是大于等于阈值且插入时的位置不为空时扩容
      - 扩容后的链表会以头插法插入（扩容死链出现的原因）
        - a-b-c 会变成 c-b-a
        - ?
    - 1.8是大于阈值就扩容，不改变原链表或红黑树的顺序
  - 扩容是创建一个新的数组，将旧数组的值移动到新数组，通过改变引用，而不是创建新的元素
  
- 存储自定义类型的键值

  - Map集合中要保证Key是唯一的，所以必须重写hashcode()和equals()方法
    - 重写hashcode()是为了对象的key在整个hashmap中有更好的分布，提高查询性能
    - 重写equals()是为了防止两个对象的hash值一样，通过比较值来区分

- 存储的对象一定是不能变的，对象的属性发生改变会影响hash值，就会导致找不到对象，除非hash方法中不包含此改变对象的属性

- put过程

  - put 之前会遍历一次 map，看是否已经存在这对 键值
  - 插入空位置时将key和value封装为entry对象（1.8后是node对象）插入该位置
    - 不为空时
      - jdk7是头插法插入到当前节点下链表的头部
      - jdk8后是判断当前位置是红黑树还是链表
        - 红黑树：
          - 将key和value封装为一个红黑树节点并插入，这个过程会判断是否存在key值
        - 链表：
          - 将key和value封装为一个node对象用尾插法插入链表，这个过程会判断是否存在key值和统计当前节点个数，如果大于8个节点则将该链表转换为红黑树
            - 如果hashmap长度小于64则会先执行扩容，因为扩容也可以缩小链表长度，扩容还是超过8个则再转换为红黑树

- 插入时是先插入然后再判断是否超过阈值，超过则扩容
- 1.7是先插入在扩容，1.8是先扩容在插入

#### 源码 1.8

- put( K key, V value )
  - ```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    ```
  
- putVal( int hash, K key, V value, boolean onlyIfAbsent, boolean evict )
  - 判断 p 是否和要插入的 key 相等，这个 p 是这个位置上的第一个元素
    - 相等则用 e 接收这个节点（保留）
    - 不相等判断 p 是不是树节点
      - 是树节点，调用红黑树的插入 ( ( TreeNode<K,V> ) p ).putTreeVal( this, tab, hash, key, value )，并用 e 接收返回值
      - 是链表节点
        - 遍历链表节点
          - 用 e 接收 p.next，判断 e 是否为null
            - 为 null 直接插入p.next
            - 如果链表长度大于等于8，则需要转化为红黑树，调用 treeifyBin(tab, hash)，跳出循环
          - 如果在该链表中找到了相等的 key (== 或 equals)，跳出循环
          - p = e
    - 如果 e != null，说明存在旧值的key与要插入的key相等
      - 取得 e.value 为 oldValue
      - 覆盖旧值 e.value = value
      - 返回旧值 oldValue
  
  - ++modCount
  
  - 如果因为插值 size > 扩容阈值，就执行扩容 resize()
  
  - 返回 null
  
  - ```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        //定义一个 tab 接收数组，p 接收下标的第一个元素，n 接收数组长度，i 接收下标
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 第一次 put 值的时候，会触发扩容，0 -> 16
        if ((tab = table) == null || (n = tab.length) == 0)  n = (tab = resize()).length;
        // pqu如果p为 null，说明该位置还没有值，直接插入即可
        if ((p = tab[i = (n - 1) & hash]) == null) tab[i] = newNode(hash, key, value, null);
        // 数组该位置有数据
        else {
            //局部变量e储存插入的位置，k储存p的
            Node<K,V> e; K k;
            // 首先，判断该位置的第一个数据和我们要插入的数据，key 是不是"相等"，如果是，取出这个节点
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果该节点是代表红黑树的节点，调用红黑树的插值方法，本文不展开说红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 到这里，说明数组该位置上是一个链表
                for (int binCount = 0; ; ++binCount) {
                    // 插入到链表的最后面(Java7 是插入到链表的最前面)
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // TREEIFY_THRESHOLD 为 8，所以，如果新插入的值是链表中的第 8 个
                        // 会触发下面的 treeifyBin，也就是将链表转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 如果在该链表中找到了"相等"的 key(== 或 equals)
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        // 此时 break，那么 e 为链表中[与要插入的新值的 key "相等"]的 node
                        break;
                    p = e;
                }
            }
            // e!=null 说明存在旧值的key与要插入的key"相等"
            // 对于我们分析的put操作，下面这个 if 其实就是进行 "值覆盖"，然后返回旧值
            if (e != null) {
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 如果 HashMap 由于新插入这个值导致 size 已经超过了阈值，需要进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
    ```
  
- resize()

  - 定义一个节点数组保存原始数据 oldTab

  - 定义一个 oldCap 保存旧数组容量，oldThr 保存旧阈值，定义 newCap, newThr 为 0

  - ```java
    if (oldCap > 0) { // 对应数组扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 将数组大小扩大一倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 将阈值扩大一倍
            newThr = oldThr << 1; 
    } else if (oldThr > 0) // 使用指定容量的构造函数创建时，因为没put所以 oldCap<=0
        newCap = oldThr;
    else {// 使用无参构造函数创建并第一次put的时候
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //如果走了上面第一个if 中的 if 和 第一个 if 的else if 时
    //newThr是没有值的，所以需要赋值，需要判断是哪一种情况
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    ```

  - 创建新数组并转移数据

    - ```java
      //用新的数组大小初始化新的数组
      Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
      table = newTab; // 如果是初始化数组，到这里就结束了，返回 newTab 即可
      // 开始遍历原数组，进行数据迁移。
      if (oldTab != null) {
          for (int j = 0; j < oldCap; ++j) {
              Node<K,V> e;
              if ((e = oldTab[j]) != null) {
                  oldTab[j] = null;
                  // 如果该数组位置上只有单个元素，那就简单了，简单迁移这个元素就可以了
                  if (e.next == null)
                      newTab[e.hash & (newCap - 1)] = e;
                  // 如果是红黑树，具体我们就不展开了
                  else if (e instanceof TreeNode)
                      ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                  else { 
                      // 这块是处理链表的情况，
                      // 需要将此链表拆成两个链表，放到新的数组中，并且保留原来的先后顺序
                      // loHead、loTail 对应一条链表，hiHead、hiTail 对应另一条链表，代码还是比较简单的
                      Node<K,V> loHead = null, loTail = null;
                      Node<K,V> hiHead = null, hiTail = null;
                      Node<K,V> next;
                      do {
                          next = e.next;
                          if ((e.hash & oldCap) == 0) {
                              if (loTail == null)
                                  loHead = e;
                              else
                                  loTail.next = e;
                              loTail = e;
                          }
                          else {
                              if (hiTail == null)
                                  hiHead = e;
                              else
                                  hiTail.next = e;
                              hiTail = e;
                          }
                      } 
                      while ((e = next) != null);
                      if (loTail != null) {
                          loTail.next = null;
                          // 第一条链表
                          newTab[j] = loHead;
                      }
                      if (hiTail != null) {
                          hiTail.next = null;
                          // 第二条链表的新的位置是 j + oldCap，这个很好理解
                          newTab[j + oldCap] = hiHead;
                      }
                  }
              }
          }
      }
      return newTab;
      ```

- get(Object key)

  - ```java
    return getNode(hash(key), key) == null ? null : e.value
    ```

- getNode(int hash, Object key)

  - ```java
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //表不为空，长度大于0，下标处有值
        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
            // 判断第一个节点是不是就是需要的
            if (first.hash == hash &&  ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //大于一个节点
            if ((e = first.next) != null) {
                // 判断是否是红黑树
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 链表遍历
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                }while ((e = e.next) != null);
            }
        }
        return null;
    }
    ```

#### 题目

- HashMap用红黑树？为什么不用AVL树 

  - 用树结构主要是用来避免DoS攻击，防止链表超长时性能下降

  - 红黑树插入删除快，AVL慢，因为AVL旋转次数更多，平衡和调试更难，但AVL读取快，适合读取密集型任务


  - HashMap红黑树何时会退化？

    - 当因为扩容时导致树的节点少于6时会退化为链表
    
    - 当删除树的节点时，主要看root，root.left，root.right，root.left.left
      - 如果删除前以上节点还存在则不会退化为链表，反之退化


  - HashMap树化阈值为什么为8？
    - hash值如果足够随机，则在hash表内按泊松分布，在负载因子为0.75的情况下，链表长度超过8的概率为 6*10^-7^ 


  - 索引如何计算？
    - 计算对象的hashCode()，再调用HashMap的hash()方法进行第二次哈希运算，最后 & (容量 - 1) 得到索引
      - 二次哈希值 = 原始哈希code ^ 原始哈希code>>>16
      - 注意这里的容量必须是2的n次幂


  - 已经有hashCode方法为什么还要提供hash方法？
    - 为了使哈希分布更加均匀，减少哈希碰撞


  - 容量为何是2的n次幂？
    - 在HashMap中进行二次hash时可以用按位与运算代替掉取模运算，从而提升一定的计算性能


  - 扩容时如何确定新位置？

    - 1.8才有的优化
      - 在扩容时会将元素的二次hash值 和 未扩容前的原始容量 进行按位与（**&**）运算
        - 如果结果为0，表示这些元素扩容后位置不会发生改变
        - 如果结果不为0，则表示这些元素将移动到新的位置
        
        - 新的位置 = 原始位置 + 原始容量
      
      - 会把结果为0的和结果不为0的串成两条链表，批量操作


  - 如果容量不是2的n次幂呢？

    - 容量是2的n次幂存在一个问题，当要存储的元素奇偶分布不均时，哈希分布就不那么均匀了

    - 把容量设置为一个质数可以解决这个问题

    - 设置为质数可以在第一次hash时就能有不错的散列分布

    - 如果想要好的性能则选2的n次幂，想要更好的散列分布则选质数


  - 扩容因子为什么默认是0.75？ 

    - 在空间占用和查询时间之间取得较好的平衡
    
    - 大于这个值，空间节省了，但是链表较长影响性能
    
    - 小于这个值，冲突减少了，但链表扩容频繁，空间占用多


  - 多线程下hashMap存在什么问题？

       - jdk7有扩容死链问题
         - 是因为1.7扩容后的链表会以头插法插入
         - 当多个线程同时进行扩容时，一个线程扩容后次序改变了，另一个线程再扩容导致死链
       
        - jdk8有数据丢失问题
          - 当多个线程同时往map的同一个位置存放数据时，容易出现覆盖丢失数据

### LinkedHashSet

- 知识点
  - 底层是一个哈希表（数组+链表 or 数组+红黑树+链表(记录元素的存取顺序))
  - 存储的有序的集合，但是也是不允许重复的

### LinkedHashMap

- 有序，存取元素顺序一致，可以存 null
- 底层是哈希表+双向链表（存储顺序）
- 该双向链表的迭代顺序就是`entry`的插入顺序
- 非线程安全的，想要安全可以 `Collections.synchronizedMap()` 

#### 源码

- get() 与hashmap的get方法几乎一致

- addEntry(int hash, K key, V value, int bucketIndex)

  - ```java
    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);// 自动扩容，并重新哈希
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = hash & (table.length-1);// hash%table.length
        }
        // 1.在冲突链表头部插入新的entry
        HashMap.Entry<K,V> old = table[bucketIndex];
        Entry<K,V> e = new Entry<>(hash, key, value, old);
        table[bucketIndex] = e;
        // 2.在双向链表的尾部插入新的entry
        e.addBefore(header);
        size++;
    }
    ```

  - ```java
    private void addBefore(Entry<K,V> existingEntry) {
        after  = existingEntry;
        before = existingEntry.before;
        before.after = this;
        after.before = this;
    }
    ```

- removeEntryForKey(Object key)

  - ```java
    final Entry<K,V> removeEntryForKey(Object key) {
    	......
    	int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);// hash&(table.length-1)
        Entry<K,V> prev = table[i];// 得到冲突链表
        Entry<K,V> e = prev;
        while (e != null) {// 遍历冲突链表
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {// 找到要删除的entry
                modCount++; size--;
                // 1. 将e从对应bucket的冲突链表中删除
                if (prev == e) table[i] = next;
                else prev.next = next;
                // 2. 将e从双向链表中删除
                e.before.after = e.after;
                e.after.before = e.before;
                return e;
            }
            prev = e; e = next;
        }
        return e;
    }
    ```

### HashTable

- 知识点
  - 底层是哈希表，单线程，安全，同步，速度慢
  - 初始容量是11，每次扩容是：容量 × 2 + 1
  - 容量是质数，不需要二次hash就要就好的散列分布
  - 不能存null键，null值，空指针异常
  - Hashtable和vector集合在jdk1.2之后被（HashMap,ArrayList）取代了但Hashtable的子类Properties依然活跃
  - Properties集合是唯一一个和IO流相结合的集合


### TreeMap（红黑树）

#### 知识点

- 是一个双列集合，底层由红黑树构成
- 键不能重复，重复会覆盖
- 元素会按从大到小排列
- 不是线程安全的，可以通过 `Collections.synchronizedSortedMap(new TreeMap(...))` 使其线程安全

#### 源码

- 左旋

  - ```java
    /*
    		p                         pr(r)
    	   / \                        / \
    	 pl   pr(r)       ==>        p   rr
    	      / \                   / \
    	    rl   rr               pl  rl
    	
    */
    private void rotateLeft(Entry<K,V> p) {
        if (p != null) {
            Entry<K,V> r = p.right;
            p.right = r.left;
            if (r.left != null)
                r.left.parent = p;
            r.parent = p.parent;
            if (p.parent == null)
                root = r;
            else if (p.parent.left == p)
                p.parent.left = r;
            else
                p.parent.right = r;
            r.left = p;
            p.parent = r;
        }
    }
    ```

- 右旋

  - ```java
    /*
    		p                         pl(l)
    	   /  \                        / \
       (l)pl   pr       ==>          ll   p
    	/ \                              / \
      ll   lr                           lr  pr
    	
    */
    private void rotateRight(Entry<K,V> p) {
        if (p != null) {
            Entry<K,V> l = p.left;
            p.left = l.right;
            if (l.right != null) 
                l.right.parent = p;
            l.parent = p.parent;
            if (p.parent == null)
                root = l;
            else if (p.parent.right == p)
                p.parent.right = l;
            else p.parent.left = l;
            l.right = p;
            p.parent = l;
        }
    }
    ```

- 寻找节点后继

  - ```JAVA
    static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
        if (t == null){
            return null;
        }
        // 1. t的右子树不空，则t的后继是其右子树中最小的那个元素，即最左的元素
        else if (t.right != null) {
            Entry<K,V> p = t.right;
            while (p.left != null)
                p = p.left;
            return p;
        }
        // 2. t的右孩子为空，则t的后继是其第一个向左走的祖先，即parent.right = t的
        else {
            Entry<K,V> p = t.parent;
            Entry<K,V> ch = t;
            while (p != null && ch == p.right) {
                ch = p;
                p = p.parent;
            }
            return p;
        }
    }
    ```

- get

  - ```java
    final Entry<K,V> getEntry(Object key) {
        ......
        if (key == null)//不允许key值为null
            throw new NullPointerException();
        Comparable<? super K> k = (Comparable<? super K>) key;//使用元素的自然顺序
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)//向左找
                p = p.left;
            else if (cmp > 0)//向右找
                p = p.right;
            else
                return p;
        }
        return null;
    }
    ```

- put()

  - ```java
    public V put(K key, V value) {
    	......
        Entry<K,V> t = this.root;
        //如果根节点为null，当前节点直接作为根节点
        if(t == null) this.root = new Entry<>(key, value, parent);
        int cmp;
        Entry<K,V> parent; //定义一个父亲节点
        if (key == null)  throw new NullPointerException();
        Comparable<? super K> k = (Comparable<? super K>) key;//使用元素的自然顺序
        do {
            parent = t;
            cmp = k.compareTo(t.key); //与当前值比较大小
            if (cmp < 0) t = t.left;//向左找 插入的值小于当前值
            else if (cmp > 0) t = t.right;//向右找 插入的值大于当前值
            else return t.setValue(value); //当前值等于插入值
        } while (t != null); //非叶子节点
        //到达叶子节点
        Entry<K,V> e = new Entry<>(key, value, parent);//创建并插入新的entry
        if (cmp < 0) parent.left = e;
        else parent.right = e;
        fixAfterInsertion(e);//调整（旋转和变色）
        size++;
        return null;
    }
    ```
    
    
    
  - ```java
    //调整插入的情况
    /*         左三               右三                     3树左                     3树右
    			a（黑）         a（黑）                    a（黑）                  a（黑）
    	   	   /                 \                       /   \                     /   \
    	 	 b（红）               b（红）            b（红） c（红）            b（红）  c（红）
        	/                       \                 /                                   \
       	   x（红）                    x（红）        x（红）                                x（红）
    */
    private void fixAfterInsertion(Entry<K,V> x) {
        x.color = RED;//新插入的节点都是红色的
        //父节点为红色时需要调整
        while (x != null && x != root && x.parent.color == RED) {
            //x的父节点是爷爷节点的左节点（处理左三和3树左的情况）
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                //取出父亲节点的父亲节点的右节点（叔叔节点）
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));
                //判断是否是3树左，如果叔叔节点为空是false，不为空则是3树左的情况
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);              //x的父亲设置为黑色
                    setColor(y, BLACK);                        //叔叔节点设置为黑色
                    setColor(parentOf(parentOf(x)), RED);      //爷爷节点设置为红色
                    x = parentOf(parentOf(x));                 //x指向爷爷节点，如果爷爷节点的父节点也是红接着处理
                } 
                //左3
                else {
                    //x是父节点的右节点（非完全左3），需要先左旋为左3，再右旋
                    if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);                       
                        rotateLeft(x);                        //在旋转时会改回父子间的指向
                    }
                    //x是父节点的左节点
                    setColor(parentOf(x), BLACK);             //x的父亲设置为黑色
                    setColor(parentOf(parentOf(x)), RED);     //爷爷节点设置为红色
                    rotateRight(parentOf(parentOf(x)));       //右旋 
                }
            } 
            //x的父节点是爷爷节点的右节点（处理右三和3树右的情况）
            else {
                //取出父亲节点的父亲节点的左节点（叔叔节点）
                Entry<K,V> y = leftOf(parentOf(parentOf(x)));
                //判断是否是3树右，如果叔叔节点为空是false，不为空则是3树右的情况
                if (colorOf(y) == RED) {
                    setColor(parentOf(x), BLACK);              //x的父亲设置为黑色
                    setColor(y, BLACK);                        //叔叔节点设置为黑色
                    setColor(parentOf(parentOf(x)), RED);      //爷爷节点设置为红色
                    x = parentOf(parentOf(x));                 //x指向爷爷节点，如果爷爷节点的父节点也是红接着处理
                } 
                //右3
                else {
                    //x是父节点的左节点（非完全右3），需要先右旋为右3，再左旋
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);                       
                        rotateRight(x);                        //在旋转时会改回父子间的指向
                    }
                    setColor(parentOf(x), BLACK);              //x的父亲设置为黑色
                    setColor(parentOf(parentOf(x)), RED);      //爷爷节点设置为红色
                    rotateLeft(parentOf(parentOf(x)));         //左旋 
                }
            }
        }
        root.color = BLACK;			//根节点必须为黑色
    }
    ```

- remove()

  - ```java
    /*
    	删除的情况：
    		1.删除节点是叶子节点，直接删除
    		2.删除节点有左孩子或者右孩子，删除节点后让其左孩子或右孩子指向向删除节点的父节点
    		3.删除节点有左右孩子节点，需要找到当前节点的后继节点，替代当前节点                    
    */
    private void deleteEntry(Entry<K,V> p) {
        modCount++;
        size--;
        //情况3，将其转换成情况2来处理
        if (p.left != null && p.right != null) 
            Entry<K,V> s = successor(p);
            p.key = s.key;
            p.value = s.value;
            p = s; //p指向它的后继节点
        }
    	//情况2
    	//有左孩子取左孩子，没有就取右（因为我们找的是后继节点，是不会有左孩子的，但源码为了代码健壮性）
        Entry<K,V> replacement = (p.left != null ? p.left : p.right);
    	//有右孩子
        if (replacement != null) {
            //左孩子或右孩子指向向删除节点的父节点
            replacement.parent = p.parent;
            if (p.parent == null) //父节点为空说明是根节点，直接替换
                root = replacement;
            else if (p == p.parent.left) //是父节点的左孩子，重新修改指向
                p.parent.left  = replacement;
            else						//是父节点的右孩子，重新修改指向
                p.parent.right = replacement;
            p.left = p.right = p.parent = null; //被删除的节点的引用全部置为null，gc处理
            if (p.color == BLACK)			 //如果删除节点的颜色为黑色需要进行调整
                fixAfterDeletion(replacement);// 调整
        } 
    	//无左右孩子且父节点为null，即删除的节点是根节点
    	else if (p.parent == null) {
            root = null;	      
        } 
    	//情况1
    	else {
            if (p.color == BLACK)
                fixAfterDeletion(p);// 调整
            if (p.parent != null) {
                if (p == p.parent.left)
                    p.parent.left = null;
                else if (p == p.parent.right)
                    p.parent.right = null;
                p.parent = null;
            }
        }
    }
    ```
    
  - ```java
    /*
    	情况1：被删除的节点有左右孩子的节点，需要通过改变节点指向和变色就可以解决的
    	情况2：被删除的节点无左右孩子的节点，用父亲节点替代，再用一个兄弟节点的子节点替代父亲节点
    		情况2.1：兄弟节点有一个孩子，旋转成有右孩子的情况，在进行左旋
    		情况2.2：兄弟节点有两个孩子，直接进行左旋
    	情况3：被删除的节点无左右孩子的节点，用父亲节点替代，但是兄弟节点没有子节点可以替代父亲节点	
    */
    private void fixAfterDeletion(Entry<K,V> x) {
        //情况2，3
        while (x != root && colorOf(x) == BLACK) {
            //x是左孩子的情况
            if (x == leftOf(parentOf(x))) {
                //找到x的兄弟节点
                Entry<K,V> sib = rightOf(parentOf(x));
                //判断sib是否是x的兄弟节点，只有sib是黑色时才是兄弟节点，红色需要调整
                if (colorOf(sib) == RED) {
                    setColor(sib, BLACK);                   // 兄弟变黑
                    setColor(parentOf(x), RED);             // 兄弟父亲变红
                    rotateLeft(parentOf(x));                // 左旋，将左孩子变成右孩子
                    sib = rightOf(parentOf(x));             // 修改找到真正的兄弟节点
                }
                //情况3，兄弟节点无左右孩子节点
                if (colorOf(leftOf(sib))  == BLACK && colorOf(rightOf(sib)) == BLACK) {
                    setColor(sib, RED);                     // 设置兄弟节点为红色
                    x = parentOf(x);                        // 删除的节点指向x的父亲，递归条件
                } 
                //情况2，兄弟节点有一个或俩个子节点
                else {
                    //没有右孩子，需要右旋为有左孩子的情况
                    if (colorOf(rightOf(sib)) == BLACK) {
                        setColor(leftOf(sib), BLACK);       // 将兄弟节点的左节点变黑
                        setColor(sib, RED);                 // 将兄弟节点变红
                        rotateRight(sib);                   // 右旋
                        sib = rightOf(parentOf(x));         // 兄弟节点重新指向为父亲节点的右节点
                    }
                    setColor(sib, colorOf(parentOf(x)));    // 兄弟节点要变成父亲节点的颜色
                    setColor(parentOf(x), BLACK);           // 父亲节点变黑
                    setColor(rightOf(sib), BLACK);          // 兄弟节点的右孩子变黑
                    rotateLeft(parentOf(x));                // 左旋
                    x = root;                               // 结束循环
                }
            } 
            //x是右孩子的情况，不会出现这种可能
            else { 
                Entry<K,V> sib = leftOf(parentOf(x));
                if (colorOf(sib) == RED) {
                    setColor(sib, BLACK);                   // 
                    setColor(parentOf(x), RED);             // 
                    rotateRight(parentOf(x));               // 
                    sib = leftOf(parentOf(x));              // 
                }
                if (colorOf(rightOf(sib)) == BLACK &&
                    colorOf(leftOf(sib)) == BLACK) {
                    setColor(sib, RED);                     // 
                    x = parentOf(x);                        // 
                } else {
                    if (colorOf(leftOf(sib)) == BLACK) {
                        setColor(rightOf(sib), BLACK);      // 
                        setColor(sib, RED);                 // 
                        rotateLeft(sib);                    // 
                        sib = leftOf(parentOf(x));          // 
                    }
                    setColor(sib, colorOf(parentOf(x)));    // 
                    setColor(parentOf(x), BLACK);           // 
                    setColor(leftOf(sib), BLACK);           // 
                    rotateRight(parentOf(x));               // 
                    x = root;                               // 
                }
            }
        }
        //情况1，变黑，指向在前面已经修改完毕了
        setColor(x, BLACK);
    }
    ```

### WeakHashMap

#### 知识点

- key是弱引用，gc时回收

## 函数式接口

### 	Runnabl

### Comparator

### Supplier<T>		

- 作用：返回一个指定类型的数据（生产型接口）

- 方法：T get()

### Consumer<T>		

- 作用：用于消费数据（消费型接口）

- 方法

  - accept(T t)

  - andThen()			可以把俩个Consumer接口组合在一起
    - consumer1.andThen(consumer2).accept()			
      - consumer1会先执行


### Predicate<T>

- 作用：对数据进行判断

- 方法：

  - test(T t)
    - 返回值是Boolean


  - and()		与
    - `pre1.and(pre2).test(str);					//都为真返回true`


  - or()			或
    - `pre1.or(pre2).test(str);					//一个为真返回true`


  - negate()		取反
    - `pre.negate().test(str)						//真为假，假为真`


### Function<T,R>

- 作用：将一个类型的数据转换为另一个类型的数据

- 方法

  - R apply(T t)		把数据类型T转换为R类型

  - andThen			 将俩个function接口进行组合操作 

    - ```java
      //放回值取决于第二个接口的放回值类型
      public static String method2(String s, Function<String,Integer> fun1,Function<Integer,String> fun2){
              return fun1.andThen(fun2).apply(s);
      }
      ```


## Stream流

### 思想

​	关注的是做什么，而不是怎么做，相当于流水线，只能使用一次 

### 获取流的方法

​	1.Collection集合都可以调用默认方法Stream()来获取流

```java
//list转Stream	List<T>
Stream<T> stream = list.stream();
//set转Stream	Set<T>
Stream<T> stream = set.stream();
//map转Stream	Map<T,R>
Stream<T> stream = map.keySet().stream();
Stream<R> stream = map.values().stream();
Stream<Map.Entry<T, R>> stream = map.entrySet().stream();
```

​	2.Stream接口的静态方法of()可以获取对应的流

```java
//数组转Stream
Stream.of(数组)
```

### 常用方法

#### reduce() 

- 实现聚合计算，求和等等
- Identity : 定义一个元素代表是归并操作的初始值，如果Stream 是空的，也是Stream 的默认结果
- Accumulator: 定义一个带两个参数的函数，第一个参数是上个归并函数的返回值，第二个是Strem 中下一个元素。
- Combiner: 调用一个函数来组合归并操作的结果，当归并是并行执行或者当累加器的函数和累加器的实现类型不匹配时才会调用此函数。

#### mapToXXX()

- 映射为对应的类型，有Int、Long、Double
- 后可接求大小、平均值、求和等等

#### orElse / orElseGet

- orElse(null)表示如果一个都没找到返回null。【orElse()中可以塞默认值。如果找不到就会返回orElse中你自己设置的默认值。】
- orElseGet(null)表示如果一个都没找到返回null。【orElseGet()中可以塞默认值。如果找不到就会返回orElseGet中你自己设置的默认值。】
- 区别就是在使用方法时，即使没有值，也会执行 orElse 内的方法， 而 orElseGet则不会

## 泛型

### 泛型类/接口

- `class<T>`
  - 类的内部可以使用类的泛型
  - 实例化没有指明类的泛型，默认为Object
  - 子类继承了指明泛型类型的父类（泛型类），实例化时不需要指定泛型类型
  - 静态方法中不能使用类的泛型
  - 异常中不能用泛型

### 泛型方法

- public <T> T method(T t){};

## 反射（Reflect）

### 含义

- 将类的各个组成部分封装为其他对象，这就是反射机制

### 内部原理

### 功能

1. #### 获取Class对象

   * Class.forName("全类名")		将字节码文件加载进内存，返回Class对象

     ​						多用于配置文件

   * 类名.class                                通过类名获取 

     ​						多用于参数的传递

   * 对象.getClass()                        getClass()方法在Object中定义的

     ​						多用于对象的获取字节码的方式

2. #### 获取成员

   * 获取成员变量
     * Fleld[] getFields()											获取public修饰的成员变量
     * field getField(String name)                    获取指定名称的public修饰的成员变量
     * Field[] getDecLaredFields()                    获取所有的成员变量
     * Field getDeclaredField(String name)     获取指定名称的成员变量
   * 获取构造方法
     * Constructor<T>[] getConstructors()																		
     * Constructor<T> getConstructor(类<?>...  parameterTypes)       
     * Constructor<T>[] getDeclaredConstructors()
     * Constructor<T>[] getDeclaredConstructors(类<?>...  parameterTypes)

   * 获取成员方法
     * Method[] getMethods()
     * Method getMethods(String name,类<?>...  parameterTypes)
     * Method[] getDeclaredMethods()
     * Method getDeclaredMethods(String name,类<?>...  parameterTypes)
   * 获取类名
     * String getName()

### 成员

 * #### Field：成员变量

   - void set(Object obj,Object value)			设置值
   - get(Object obj)												 获取值
   
* #### Constructor：构造方法

  - T newInstance(Object... initargs)			创建对象

* #### Method：成员方法

  - Object invoke(Object obj,) 				执行方法
- String getName()									获取方法名	

---

### 注意

1. 获取访问受限的成员时需要使用暴力反射

   ​			setAccessible(true)		忽略安全检查

2. 创建无参构造可以使用Class对象的方法

   ​			newInstance()				创建无参构造方法

3. 加载配置文件，配置文件以properties结尾

   ​			classLoader.getResourceAsStream("pro.Properties");读取配置文件，

   ​																	返回InputStream流

   ​			通过类名.class.getClassLoader()获取ClassLoader对象

## 枚举（Enum）

### 要求：

* 类的对象的数量有限，确定的
* 需要定义一组常量时最好使用枚举类
* 如果枚举类中只有一个对象，则可以作为单例模式的实现方式

### 定义：

1. 5.0之前的自定义
   * private final 成员变量
   * private 构造方法
   * public static final 创建对象
2. 使用enum关键字定义
   * 需要将对象定义在开头，对象之间用逗号分隔，最后用分号结束

### 常用方法

	* values()									返回枚举类型的对象数组
	* valueOf(String str)            可以把字符串转为对应的枚举对象
	* toString()                          返回当前枚举类对象常量的名称

---

## 注解（Annotation）	

### 定义

​	也叫元数据，出现在1.5之后的新特性，

### 作用

 1. 编写文档

    javadoc   java提供的工具，可以将java文件抽取为doc文档

    ​	java文件需要以ANSI格式编码才不会乱码

 2. 代码分析

 3. 编译检查

### JDK中预定义的注解

   * @Override							检测该注解标注的方法是否覆盖重写父类方法
   * @Deprecated	               该注解标注的内容表示以过时
     @suppressWarnings       压制警告

### 自定义注解

- 本质上就是一个接口，可以定义成员方法，成员方法的返回值有要求

  - 要求：基本数据类型,String,枚举,注解,以上类型的数组，使用时需要赋值

    例如：

    ​          int value();//如果方法只有一个且名字为value，那么使用时可以直接写值 

    ​									使用时：@MyAnno(10)

    ​          String name() **default** "张三"			//有默认值，使用时可以不赋值

    ​			enumTest emum1();								//枚举类型

    ​									使用时 emum1 = enumTest.P1

    ​			MyAnno2 anno2();									//注解类型

    ​									使用时	anno2 = @MyAnno2

    ​			String strs[]												//字符数组 

​						   

- 格式:			

​			public @interface 注解名称{}

- 注意：

### 元注解

- 作用
  - 用于描述注解
- 常用
  - @Target						描述注解能够作用的位置
    - ElementType.TYPE     	       该注解只可作用于类上
    - ElementType.METHOD     	该注解只可作用于类上
    - ElementType.FIELD     	     该注解只可作用于类上
  - @Retention              描述注解被保留的阶段
    - RetentionPolicy.SOURCE          该注解保留到资源时状态
    - RetentionPolicy.CLASS             该注解保留到class时状态
    - RetentionPolicy.RUNTIME        该注解保留到运行时状态，能被JVM读取
  - @Documented         描述注解是否被抽取到api文档
  - @Inherited               描述注解是否被子类继承

### 使用注解的步骤

1. 获取注解定义的位置的class对象
2. Class.**getAnnotation**()							//获取注解的实现类的对象
3. 调用注解中的方法

### 其他

* javap 								java提供的反编译工具

---

## 动态代理

![image-20210219132858763](..//pic//image-20210219132858763.png)

​	实现类又称被代理类

### 实现

 - 创建动态代理类

```java
//代理工厂类
class ProxyFactory {
	//传入实现类(被代理类)对象，返回代理类对象
    //Object obj 被代理类对象
    public static Object getProxyInstance(Object obj){
    	//创建InvocationHandler实现类对象
        MyInvocationHandler handler = new MyInvocationHandler(obj);
        //返回代理类对象
        //obj.getClass().getClassLoader()获取被代理类的类加载器
        //obj.getClass().getInterfaces() 获取被代理类实现的接口
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(),
        obj.getClass().getInterfaces(),handler);
    }

}
//InvocationHandler实现类
class MyInvocationHandler implements InvocationHandler{
	//被代理类对象
    private Object obj;
    //获取被代理类对象
    MyInvocationHandler(Object obj){
        this.obj = obj;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(obj,args);
    }
}
```

---

## API

### Runtime

- 概述

  - 封装了运行的环境，每个Java应用程序都有一个Runtime实例
  - 一般不能实例化Rumtime对象，可以通过getRuntime()获取当前对象的引用
  - 拥有一个静态初始化对象--饿汉模式(单例模式的一种)
    - `private static Runtime currentRuntime = new Runtime();`

  - 常用方法

    - availableProcessors()								返回当前可用的虚拟机数量

    - exec()                                                    执行其他程序，例如打开记事本

      - `r.exec("notepad")`

    - 内存管理

      - totalMemory())						返回当前对象的堆内存有多大

      - freeMemory()                      返回当前对象的堆内存还剩多少

      - gc()                                      根据需要运行无用单元收集器先于

        ​														虚拟机自动回收对象

### Process

### Optional

- 为了防止空指针异常

#### 方法

- Optional.of(T t)
  - t不能为null
- Optional.empty()
  - 创建一个空的Optional实例
- Optional.ofNullable(T t)
  - t可以为空
- Optional对象.orElse(T t)
  - Optional对象为空则返回t

## 其他

### rt.jar、dt.jar、tools.jar

- rt.jar
  - 代表runtime JAR，包含Core Java API的所有类，包含所有核心Java 运行环境的已编译calss文件
  - 位于JRE的lib目录下，包含了JVM信任的class文件，JVM加载时不会对其他class文件进行严格的安全检查
  - 是一个类似zip的压缩文件，精确称为 Java archive（Jar），存储了Java class文件和程序所需的全部资源。它还可以包含==mainfest==文件，还可以包含==Main-Class条目==，变成可执行JAR
- dt.jar
  - 是系统用来编译一个类的时候用到的，即执行javac的时候用到。
- tools.jar
  - dt.jar是关于运行环境的类库，主要是swing包。
