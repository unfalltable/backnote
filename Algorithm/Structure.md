---
title: 数据结构
keywords: [structure, basic] 
---

## 数组

### 数组实现队列

- 需要两个变量 front 和 rear 
  - front 是队列第一个元素的前一位
  - rear 是最后一个元素的位置
- front == real 表示队列为空
- real = maxSize - 1 表示队列已满
- (rear + maxSize - front) % maxSize  队列中有效数据个数

### 数组实现环形队列

- 需要两个变量 front 和 rear 
  - front 是队列第一个元素
  - rear 是最后一个元素的下一个位置
- front == real 表示队列为空
- (rear+1) % maxSize == front  表示队列已满

## 字符串

### KMP

- 字符串匹配

```java
public int strStr(String haystack, String needle) {
    //
    int[] next = getNext(needle);
    int s1 = 0;
    int s2 = 0;
    while(s1 < haystack.length() && s2 < needle.length()){
        if(haystack.charAt(s1) == needle.charAt(s2)){
            s1++;
            s2++;
        }else if(next[s2] == -1){
            s1++;
        }else{
            s2 = next[s2];
        }
    }
    return s2 == needle.length() ? s1-s2 : -1;
}
```

```java
public int[] getNext(String str){
    if(str.length() == 1) return new int[]{-1};
    int[] res = new int[str.length()];
    res[0] = -1;
    res[1] = 0;
    int i = 2;
    int cn = 0;
    while(i < res.length){
        if(str.charAt(i - 1) == str.charAt(cn)){
            res[i++] = ++cn;
        }else if(cn > 0){
            cn = res[cn];
        }else{
            res[i++] = 0;
        }
    }
    return res;   
}
```

### Manacher

```java
//将字符串变为例如：#a#a#a# 的样子
public static char[] manacherString(String str){
    char[] charArr = str.toCharArray();
    char[] res = new char[str.length() * 2 + 1];
    int index = 0;
    for(int i = 0; i != res.length; i++){
        res[i] = (i & 1) == 0 ? '#' : charArr[index++];
    }
    return res;
}
```

```java
public static int maxLcpsLength(String s){
    if(s == null || s.length() == 0) return 0;
    //s变成 #s#s#s#
    char[] str = manacherString(s);
    //回文半径数组
    int[] pArr = new int[str.length];
    int R = -1; //回文右边界再往右一个位置
    int C = -1; //中心
    int max = Integer.MAX_VALUE;
    for(int i = 0; i != str.length; i++){
        //不用验的区域
        pArr[i] = R > i ? Math.min(pArr[2 * C - i], R - i) : 1;
        //从不用验的区域往外验
        while(i + pArr[i] < str.length && i - pArr[i] > -1){
            if(str[i + pArr[i]] == str[i - pArr[i]]){
                pArr[i]++;
            }else{
                break;
            }
        }
        if(i + pArr[i] > R){
            R = i + pArr[i];
            C = i;
        }
        max = Math.max(max, pArr[i]);
    }
    return max - 1;
}
```

#### 适用题目：

- 最长回文子串
- 滑动窗口

## 栈

### 单调栈

- 找距离当前位置最近大的
- 栈内元素保证从大到小，数据依次进入栈
- 当前元素比栈内元素大时，依次弹出栈内元素
  - 当前元素（右边最近大的）  > 栈内元素 < 栈内元素上一位（左边最近大的） 
- 当所有元素遍历完了栈内还有元素时所有元素依次出栈
  - 无 > 栈内元素 < 栈内元素上一位（左边最近大的） 

#### 适用题目：

- 

## 二叉树

### 遍历

```java
static List<Integer> result = new Arraylist<>();
public void Iterater(TreeNode node){
    if(node == null ) return;
    //前序
    result.add(node.val);
    if(node.left != null) Iterater(node.left);
    if(node.right != null) Iterater(node.right);
    //中序
    if(node.left != null) Iterater(node.left);
    result.add(node.val);
    if(node.right != null) Iterater(node.right);
    //后序
    if(node.left != null) Iterater(node.left);
    if(node.right != null) Iterater(node.right);
    result.add(node.val);
}
```

```java
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> result = new LinkedList<>();
    Stack<TreeNode> st = new Stack<>();
    if (root != null) st.push(root);
    while (!st.empty()) {
        TreeNode node = st.pop();
        if (node != null) {
            //根据前中后序变换位置即可
            //前序：右左中 中序：右中左 后序：中右左
            st.push(node);// 添加中节点
            st.push(null); // 中节点访问过，但是还没有处理，加入空节点做为标记。
            
            if (node.right!=null) st.push(node.right);  // 添加右节点
            if (node.left!=null) st.push(node.left);    // 添加左节点
            
        } else { // 只有遇到空节点的时候，才将下一个节点放进结果集
            node = st.pop();    // 重新取出栈中元素
            result.add(node.val); // 加入到结果集
        }
    }
    return result;
}
```

### B-tree

- 以一个5阶B树为例
  - 一个节点最多有4个key，5个分支
  - 当插入一个节点使当前节点大于4时，该节点会把当前节点的中间值向上分裂

### B+tree

- 所有数据都保存在叶子节点，叶子节点之间形成单向链表
- mysql中的B+tree叶子节点之间是双向环形链表
- 非叶子节点作为索引，不存储数据

- 以一个5阶B+树为例
  - 一个节点最多有4个key，5个分支
  - 当插入一个节点使当前节点大于4时，该节点会把当前节点的中间值向上分裂
  - 分裂后叶子节点会保留分裂出去的节点值

### 红黑树

- 每条路径上的黑色节点数要一致（黑色平衡）
- 根节点是黑色的
- 新增的节点都是红色的

### 字典树（前缀树）

### Morris遍历(线索二叉树)

- 记录当前节点（cur）
  - 有左孩子
    - 左孩子的最右孩子指向null，改指向当前节点，cur = cur.left
    - 左孩子的最右孩子指向当前节点，改指向null，cur = cur.right

  - 无左孩子，cur = cur.right

- 时间复杂度：O(n)，空间复杂度：O(1)

## 哈希

### 布隆过滤器

- 解决缓存穿透问题
- 用一个bitmap存放，多个hash算法计算

### 布谷鸟过滤器

- 在布隆过滤器的基础上实现了删除功能（鸡肋，不完善）

## 图

### 并查集

- 把元素进行封装，用一个指针指向其父元素，初始化指向自己
- 创建一个元素对应表
  - 给定 a 可以找到封装过后的a

- 创建一个父元素对应表
  - 给定封装过后的 a 可以找到其封装过后的父元素  	

- 创建代表元素下有多少元素的表
  - 给定封装过后的 a 可以知道其下有多少元素


```java
//封装
class Element<V>{
    public V value;
    public Element(V value){this.value = value;}
}
//创建并查集
class UnionFindSet<V>{
    public HashMap<V, Element<V>> elementMap;
    public HashMap<Element<V>, Element<V>> fatherMap;
    public HashMap<Element<V>, Integer> sizeMap;
    //初始化操作
    public UnionFindSet(List<V> list){
        elementMap = new HashMap<>();
        fatherMap = new HashMap<>();
        sizeMap = new HashMap<>();
        for(V value : list){
            Element<V> element = new Element<V>(value);
            elementMap.put(value, element);
            fatherMap.put(element, element);
            sizeMap.put(element, 1);
        }
    }

    //防止找到父元素路径太长，每次查找都让路径上的元素直接z
    private Element<V> findHead(Element<V> element){
        Stack<Element<V>> path = new Stack<>();
        while(element != fatherMap.get(elementMap)){
            path.push(element);
            element = fatherMap.get(element);
        }
        while(!path.isEmpty()){
            fatherMap.put(path.pop(), element);
        }
        return element;
    }

    public boolean isSameSet(V a, V b){
        if(elementMap.containsKey(a) && elementMap.containsKey(b)){
            return findHead(elementMap.get(a)) == findHead(elementMap.get(b));
        }
        return false;
    }

    public void union(V a, V b){
        if(elementMap.containsKey(a) && elementMap.containsKey(b)){
            Element<V> af = findHead(elementMap.get(a));
            Element<V> bf = findHead(elementMap.get(b));
            if(af != bf){
                Element<V> big = sizeMap.get(af) >= sizeMap.get(af) ? af : bf;
                Element<V> small = big == af ? bf : af;
                fatherMap.put(small, big);
                sizeMap.put(big, sizeMap.get(af) + sizeMap.get(bf));
                sizeMap.remove(small);
            }
        }
    }
}
```

#### 适用题目：

- 岛屿问题







