---
layout: post
title: 读懂Java集合（三）
categories:
 - Java
tags:
 - Java集合
---

![](https://image.jianger.space/uPic/Map.png)

<!-- more -->

本文承接上文[读懂Java集合（二）](https://jianger.space/java-collections-part-two/)，主要整理总结一下集合框架Map接口的常用实现。

> 💡图中只列出了常见的，省略了部分子接口，本文内容基于JDK 1.8

### Map

存储键值对的集合，它的键和值都是无序的(特别的,LinkedHashMap是有序的)，存储的键不可重复,值可以重复,除了HashTable其他实现类都允许null key和null value.

> 此处的有序指插入顺序,元素的顺序是固定的,同样的代码执行多次结果是一致的



### HashMap

平时使用最为频繁,面试几乎必问的问题,因为它牵涉到了许多知识点,作为开发人员必须非常了解.

![](https://image.jianger.space/uPic/HashMap.png)

对Map接口基于哈希表的一种实现，键值元素可包含`null`，它基于数组+链表+红黑树实现，初始化时可以指定大小和装载因子，未指定默认大小为16，默认装载因子0.75f,**非线程安全**。

#### Cloneable和Serializable作用

HashMap实现了Cloneable和Serializable接口,但是这两个接口是空的,这是为什么呢?这是因为Cloneable和Serializable都是标记型的接口,它们内部都没有方法和属性.实现 Cloneable来表示该对象能被克隆，能使用Object.clone()方法;实现 Serializable来表示该对象能被序列化

#### 所有方法

| 方法                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `public int size()`                                          | 返回Map集合大小                                              |
| `public boolean isEmpty()`                                   | 判断Map元素是否为空                                          |
| `public V get(Object key)`                                   | 根据key获得相应的value值                                     |
| `final Node<K,V> getNode(int hash, Object key)`              | 根据hash和key获得相应的Node对象(静态内部类)                  |
| `public boolean containsKey(Object key)`                     | 判断给定的键key是否存在                                      |
| `public V put(K key, V value)`                               | 添加一对key-value                                            |
| `final V putVal(int hash, K key, V value, boolean onlyIfAbsent,                boolean evict)` | 给定hash和其他信息添加一对key-value                          |
| `final Node<K,V>[] resize()`                                 | `ensureCapacity (int minCapacity)`增加这个的容量 实例，以确保它至少可以保存由最小容量参数指定的元素数 |
| `final void treeifyBin(Node<K,V>[] tab, int hash)`           | `forEach ( Consumer <? super E > action)`对象的每个元素执行给定的操作 直到所有元素都处理完毕，或者操作抛出异常 |
| `public void putAll(Map<? extends K, ? extends V> m)`        | `get (int index)`返回此列表中指定位置的元素                  |
| `public V remove(Object key)`                                | `indexOf ( Object o)`返回此列表中指定元素的第一个匹配项的索引，如果此列表不包含该元素，则返回 -1 |
| `final Node<K,V> removeNode(int hash, Object key, Object value,                            boolean matchValue, boolean movable)` | 指定哈希值、键值和匹配模式移除元素                           |
| `public void clear()`                                        | 清空集合                                                     |
| `public boolean containsValue(Object value)`                 | 判断给定的值value是否存在                                    |
| `public Set<K> keySet()`                                     | 返回集合所有key的Set集合对象                                 |
| `public Collection<V> values()`                              | 返回集合所有value的Collection集合对象                        |
| `public Set<Map.Entry<K,V>> entrySet()`                      | 返回集合对应的Set集合对象                                    |
| `public V getOrDefault(Object key, V defaultValue)`          | 返回指定键对应的值,没找到返回默认值                          |
| `public V putIfAbsent(K key, V value)`                       | 如果给定的键值不存在,将给定键值添加到集合                    |
| `public boolean replace(K key, V oldValue, V newValue)`      | 给定键值和新的值将旧值替换为新值                             |
| `public V replace(K key, V value)`                           | 给定键值将指定的键对应的值替换为新值                         |
| `public V computeIfAbsent(K key,                          Function<? super K, ? extends V> mappingFunction)` | 没搞懂                                                       |
| `public V computeIfPresent(K key,                           BiFunction<? super K, ? super V, ? extends V> remappingFunction)` | 没搞懂                                                       |
| `public V compute(K key,                  BiFunction<? super K, ? super V, ? extends V> remappingFunction)` | 没搞懂                                                       |
| `public V merge(K key, V value,                BiFunction<? super V, ? super V, ? extends V> remappingFunction)` | 如果指定的键尚未与值关联或与 null 关联，则将其与给定的非空值关联 |
| `public void forEach(BiConsumer<? super K, ? super V> action)` | 对此集合中的每个元素执行给定的操作，直到处理完所有元素或该操作引发异常为止 |
| `public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function)` | 将每个元素的值替换为调用给定函数的结果，直到所有元素都得到处理或该函数引发异常 |
| `public Object clone()`                                      | 返回此集合的浅拷贝                                           |
| `private void writeObject(java.io.ObjectOutputStream s)`     | 用于HashMap序列化                                            |
| `private void readObject(java.io.ObjectInputStream s)`       | 用于HashMap序列化                                            |

#### 数据结构

从网上找来一张图，直观展示 HashMap 结构：数组+链表+红黑树

![HashMap](https://segmentfault.com/img/bVcO4D1)

#### HashMap的扩容机制

关键源码

```java
//默认初始化容量    
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
//最大容量,即2^30
static final int MAXIMUM_CAPACITY = 1 << 30;
//默认加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//链表转为红黑树的阈值
static final int TREEIFY_THRESHOLD = 8;
//红黑树转为链表的阈值
static final int UNTREEIFY_THRESHOLD = 6;

static final int MIN_TREEIFY_CAPACITY = 64;
//数组扩容阈值,即出发扩容的大小
int threshold;
//加载因子
final float loadFactor;
//元素数组table,transient表示该数组不序列化
transient Node<K,V>[] table;
//元素对象,静态内部类
static class Node<K,V> implements Map.Entry<K,V> {
  final int hash;
  final K key;
  V value;
  Node<K,V> next;

  Node(int hash, K key, V value, Node<K,V> next) {
    this.hash = hash;
    this.key = key;
    this.value = value;
    this.next = next;
  }

  public final K getKey()        { return key; }
  public final V getValue()      { return value; }
  public final String toString() { return key + "=" + value; }

  public final int hashCode() {
    return Objects.hashCode(key) ^ Objects.hashCode(value);
  }

  public final V setValue(V newValue) {
    V oldValue = value;
    value = newValue;
    return oldValue;
  }

  public final boolean equals(Object o) {
    if (o == this)
      return true;
    if (o instanceof Map.Entry) {
      Map.Entry<?,?> e = (Map.Entry<?,?>)o;
      if (Objects.equals(key, e.getKey()) &&
          Objects.equals(value, e.getValue()))
        return true;
    }
    return false;
  }
}
/** 这个方法在HashMap进行扩容时会调用到,((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
  * @param map 代表要扩容的HashMap
  * @param tab 代表新创建的数组，用来存放旧数组迁移的数据
  * @param index 代表旧数组的索引
  * @param bit 代表旧数组的长度，需要配合使用来做按位与运算
  */
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
  TreeNode<K,V> b = this;
  //设置低位首节点和低位尾节点
  TreeNode<K,V> loHead = null, loTail = null;
  //设置高位首节点和高位尾节点
  TreeNode<K,V> hiHead = null, hiTail = null;
  //定义两个变量lc和hc，初始值为0，后面比较要用，他们的大小决定了红黑树是否要转回链表
  int lc = 0, hc = 0;
  //遍历红黑树
  for (TreeNode<K,V> e = b, next; e != null; e = next) {
    //取e的下一节点赋值给next遍历
    next = (TreeNode<K,V>)e.next;
    //取好e的下一节点后，把它赋值为空，方便GC回收
    e.next = null;
    if ((e.hash & bit) == 0) {
      //
      if ((e.prev = loTail) == null)
        loHead = e;
      else
        loTail.next = e;
      loTail = e;
      ++lc;
    }
    else {
      if ((e.prev = hiTail) == null)
        hiHead = e;
      else
        hiTail.next = e;
      hiTail = e;
      ++hc;
    }
  }

  if (loHead != null) {
    if (lc <= UNTREEIFY_THRESHOLD)
      tab[index] = loHead.untreeify(map);
    else {
      tab[index] = loHead;
      if (hiHead != null) // (else is already treeified)
        loHead.treeify(tab);
    }
  }
  if (hiHead != null) {
    if (hc <= UNTREEIFY_THRESHOLD)
      tab[index + bit] = hiHead.untreeify(map);
    else {
      tab[index + bit] = hiHead;
      if (loHead != null)
        hiHead.treeify(tab);
    }
  }
}
//扩容关键代码
final Node<K,V>[] resize() {
  //旧数组,首次扩容时oldTab=table=null
  Node<K,V>[] oldTab = table;
  //旧容量,首次扩容时oldCap=0
  int oldCap = (oldTab == null) ? 0 : oldTab.length;
  //旧阈值,触发本次扩容操作的阈值,首次扩容且初始化未设置initialCapacity默认为0
  int oldThr = threshold;
  //每次扩容先初始化新容量、新阈值为0
  int newCap, newThr = 0;
  //非首次扩容
  if (oldCap > 0) {
    //扩容前的数组大小如果已经达到最大2^30,不再扩容
    if (oldCap >= MAXIMUM_CAPACITY) {
      threshold = Integer.MAX_VALUE;
      return oldTab;
    }
    //（1）
    //进行翻倍扩容(假如旧的oldCap为8， < DEFAULT_INITIAL_CAPACITY，那么此条件不成立newThr将不会赋值)
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
             oldCap >= DEFAULT_INITIAL_CAPACITY)//扩容后小于最大容量并且旧容量>=16
      newThr = oldThr << 1; // 新阈值为旧阈值的两倍
  }
  //===================首次扩容start=============================
  //（2）
  //oldCap == 0（说明hashmap中的散列表是null）且oldThr > 0 ；下面几种初始化时都会出现oldCap == 0,oldThr > 0
  // 1.public HashMap(int initialCapacity);
  // 2.public HashMap(Map<? extends K, ? extends V> m);并且这个map有数据
  // 3.public HashMap(int initialCapacity, float loadFactor);
  else if (oldThr > 0) // initial capacity was placed in threshold
    newCap = oldThr;
  else { //oldCap == 0, oldThr == 0,使用public HashMap()初始化时
    // 如果没有手动设置initialCapacity，则设为默认值16
    newCap = DEFAULT_INITIAL_CAPACITY;
    //新扩容阈值设为默认12,0.75*16=12
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
  }
  //===================首次扩容end=============================
  //对应上面（1）不成立或者（2）成立的情况
  if (newThr == 0) {
    float ft = (float)newCap * loadFactor;
    newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
              (int)ft : Integer.MAX_VALUE);
  }
  threshold = newThr;
  @SuppressWarnings({"rawtypes","unchecked"})
  //扩容后新数组
  Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
  //扩容后的数组table
  table = newTab;
  if (oldTab != null) {// 对oldTab中所有元素进行rehash。由于每次扩容是2次幂的扩展(指数组长度/桶数量扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置
    for (int j = 0; j < oldCap; ++j) {
      Node<K,V> e;
      if ((e = oldTab[j]) != null) {// 数组j位置的元素不为空，需要该位置上的所有元素进行rehash
        oldTab[j] = null;
        if (e.next == null)// 桶中只有一个元素，则直接rehash
          newTab[e.hash & (newCap - 1)] = e;
        else if (e instanceof TreeNode)// 桶中是树结构
          ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
        //桶中是链表结构（JDK1.7中旧链表迁移新链表的时候，用的是头插法，如果在新表的数组索引位置相同，则链表元素会倒置；但是JDK1.8不会倒置，用的是双指针）
        else {
          // 低位链表：存放在扩容之后的数组下标的位置，与当前数组下标位置一致的元素
          Node<K,V> loHead = null, loTail = null;
          //高位链表：存放在扩容之后的数组下标的位置为当前数组下标位置+ 扩容之前数组长度的元素
          Node<K,V> hiHead = null, hiTail = null;
          Node<K,V> next;
          do {
            next = e.next;
            if ((e.hash & oldCap) == 0) {// 是0的话索引没变，是1的话索引变成“原索引+oldCap”
              if (loTail == null)
                loHead = e;// 总是指向头结点
              else
                loTail.next = e;// 该操作有可能会改变原链表结构
              loTail = e;// 总是指向下一个节点，直到尾节点
            }
            else {
              if (hiTail == null)
                hiHead = e;
              else
                hiTail.next = e;
              hiTail = e;
            }
          } while ((e = next) != null);
          if (loTail != null) {
            loTail.next = null;
            newTab[j] = loHead;// 原索引
          }
          if (hiTail != null) {
            hiTail.next = null;
            newTab[j + oldCap] = hiHead;// 原索引+oldCap
          }
        }
      }
    }
  }
  return newTab;
}
```

可以看到，HashMap扩容分为两个步骤:1.获得新容量、新扩容阈值及新容量的空数组;2.新旧数组元素迁移;

**步骤1又分为首次扩容和不是首次两类情况:**

**首次扩容**

因为四个构造方法都没有对数组`table`进行初始化,在第一次put操作时才会对`table`进行初始化.

- 除了`new HashMap()`,其他构造方法都对`threshold`进行了初始化,此时新容量`newCap=oldThr=threshold`
- 使用`new HashMap()`时`newCap=DEFAULT_INITIAL_CAPACITY=16,threshold=12`.

**非首次**

- 先判断旧的数组容量是否达到最大值,如果达到返回旧数组不进行后续扩容操作
- 扩容后小于最大容量并且旧容量>=16时,新扩容阈值为旧扩容阈值的两倍,需要注意的是,不管条件是否成立,`newCap = oldCap << 1`这一赋值一定会执行.

**步骤2新旧数组元素迁移:**



### LinkedHashMap

![](https://image.jianger.space/uPic/LinkedHashMap.png)



### TreeMap

### SortedMap

### HashTable



### 参考来源

-    [阅读 JDK 源码：HashMap 扩容总结及图解 - SegmentFault 思否](https://segmentfault.com/a/1190000039302830) 
-    [Java面试必问之Hashmap底层实现原理(JDK1.8) - SegmentFault 思否](https://segmentfault.com/a/1190000021928659?utm_source=sf-similar-article) 
-     [HashMap-split()方法源码简读(JDK1.8)_绅士jiejie的博客-CSDN博客_hashmap split](https://blog.csdn.net/weixin_38106322/article/details/104422422) 

