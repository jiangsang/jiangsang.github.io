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

> 此处的有序指插入与取出的顺序是一致的

### HashMap

平时使用最为频繁,面试几乎必问的问题,因为它牵涉到了许多知识点,作为开发人员必须非常了解.后续我还得在专门整理一篇详细的HashMap源码分析,还好多自己不是很理解😂.

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

从网上找来一张图，直观展示 HashMap 结构：数组+链表+红黑树,不难发现,数组内的元素和链表节点都是Node<K,V>对象

![HashMap](https://image.jianger.space/uPic/bVcO4D1.png)

#### HashMap的扩容机制

首先需要明白的是为什么需要扩容呢.直接固定长度,然后有冲突使用链表或者红黑树不是也可以吗.

的确可以这么处理,但是这样做的话当元素很多的时候哈希冲突导致的链化会影响查询效率,而扩容操作可以缓解这一问题.

**扩容时机**

肯定是在put操作时需要进行扩容,分为两种情况:

1. HashMap 中 put 入第一个元素，初始化数组 table。
2. HashMap 中的元素数量大于阈值 threshold。

关键源码resize()

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
  //table指向扩容后的数组
  table = newTab;
  if (oldTab != null) {// 对oldTab中所有元素进行rehash。由于每次扩容是2次幂的扩展(指数组长度/桶数量扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置
    for (int j = 0; j < oldCap; ++j) {
      Node<K,V> e;
      // 数组j位置的元素不为空，对该位置上的所有元素进行rehash
      if ((e = oldTab[j]) != null) {
        oldTab[j] = null;
        // 桶中只有一个元素，则直接rehash
        if (e.next == null)
          //hash & (length - 1)等同hash % length,hash寻址算法
          newTab[e.hash & (newCap - 1)] = e;
        else if (e instanceof TreeNode)// 桶中是树结构
          ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
        //桶中是链表结构（JDK1.7中旧链表迁移新链表的时候，用的是头插法，如果在新表的数组索引位置相同，则链表元素会倒置；但是JDK1.8不会倒置，用的是双指针）
        else {
          // 低位链表：存放在扩容之后的数组下标的位置，与当前数组下标位置一致的元素
          Node<K,V> loHead = null, loTail = null;
          //高位链表：存放在扩容之后的数组下标的位置为当前数组下标位置+ 扩容之前数组长度的元素
          Node<K,V> hiHead = null, hiTail = null;
          //当前链表的一个元素
          Node<K,V> next;
          do {
            next = e.next;
            // 是0的话索引没变，是1的话索引变成“原索引+oldCap”
            if ((e.hash & oldCap) == 0) {
              // 低位链表的链尾为空则链头指向头结点
              if (loTail == null)
                loHead = e;
              //否则链尾的下一个节点指向当前节点
              else
                loTail.next = e;
              // 最后再把链尾节点指向最后一个节点
              loTail = e;
            }
            //高位链表操作同理
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

可以看到，HashMap扩容分为两个步骤:1.计算获得新容量、新扩容阈值及新容量的空数组;2.新旧数组元素迁移,这里才是真正的扩容操作;

##### 计算新容量、新阈值

**首次扩容**
因为四个构造方法都没有对数组`table`进行初始化,在第一次put操作时才会对`table`进行初始化.
- 除了`new HashMap()`,其他构造方法都对`threshold`进行了初始化,此时新容量`newCap=oldThr=threshold`,此时新容量不一定等于new一个HashMap指定的容量`initialCapacity`,而是与`tableSizeFor(int cap)`方法有关,它总是2的幂次方
- 使用`new HashMap()`时`newCap=DEFAULT_INITIAL_CAPACITY=16,threshold=12`.

**非首次**
- 先判断旧的数组容量是否达到最大值,如果达到返回旧数组不进行后续扩容操作
- 扩容后小于最大容量并且旧容量>=16时,新扩容阈值为旧扩容阈值的两倍,需要注意的是,不管条件是否成立,`newCap = oldCap << 1`这一赋值一定会执行,即此时扩容是肯定的,此时扩容阈值如下:

    ```java
    float ft = (float)newCap * loadFactor;
    newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
              (int)ft : Integer.MAX_VALUE);
    ```

##### 数组迁移
这里牵涉到hash寻址算法、高低位拆分扩容和与(&)操作等,其中高低位拆分扩容是关键.以前没看过源码的时候以为扩容就是单纯的数组扩容,然后后面多出的空位可以添加新的元素,原来不仅如此还需要对链表进行拆分并将它们重新索引到新的下标.

**高低位拆分扩容**
进行高低位拆分是为了降低原链表的长度,缓解哈希冲突导致的查询效率的下降.那么为什么根据`(e.hash & oldCap) 等于0或1`就可以将原先的一条链表拆分成两条,然后存储到不同数组索引下面,这点还没弄明白...


### LinkedHashMap

![](https://image.jianger.space/uPic/LinkedHashMap.png)

LinkedHashMap继承自HashMap,除了重写部分方法,其他都是直接使用的HashMap的方法.相较HashMap的区别就是多了一个双向链表,**并且它是有序的,因为它的遍历是对双向链表的遍历**,因此如果需要有序的HashMap可以使用LinkedHashMap

#### 数据结构
![LinkedHashMap结构](https://image.jianger.space/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDUwNTg2,size_16,color_FFFFFF,t_70.png)

LinkedHashMap=HashMap+双向链表,每次putVal()操作时,如果添加成功都会把当前Node对象加入到双向链表中去  
LinkedHashMap的有序其实包括2种:  

1. 插入顺序：很好理解，就是按插入顺序储存,当accessOrder = false时成立。
2. 访问顺序:当accessOrder = true时,被访问的元素会放到链表的尾端，其他元素顺序不变。

#### 关键源码

```java
 //头指针，指向第一个node
transient LinkedHashMap.Entry<K,V> head;

//尾指针，指向最后一个node
transient LinkedHashMap.Entry<K,V> tail;

//false为插入顺序
//true为访问顺序,每次get操作后需要将对应节点重新放到链表的尾部
final boolean accessOrder;
static class Entry<K,V> extends HashMap.Node<K,V> {
  //双向指针
  Entry<K,V> before, after;
  Entry(int hash, K key, V value, Node<K,V> next) {
    super(hash, key, value, next);
  }
}
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
  return false;
}
//将新增节点放到队列尾部,返回新增的节点
//对HashMap中newNode方法的重写
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
  LinkedHashMap.Entry<K,V> p =
    new LinkedHashMap.Entry<K,V>(hash, key, value, e);
  linkNodeLast(p);
  return p;
}
//尾插法插入p
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
  //last指向原尾节点
  LinkedHashMap.Entry<K,V> last = tail;
  //尾插法,故首先将尾指针指向新节点p
  tail = p;
  //尾指针为空,此时链表是空的,头指针也指向p
  if (last == null)
    head = p;
  else {
    //正常的尾插操作,p的前驱指向原尾节点
    p.before = last;
    //原尾节点的后继指向p
    last.after = p;
  }
}
// unlink
void afterNodeRemoval(Node<K,V> e) {
  LinkedHashMap.Entry<K,V> p =
    (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
  p.before = p.after = null;
  if (b == null)
    head = a;
  else
    b.after = a;
  if (a == null)
    tail = b;
  else
    a.before = b;
}
// possibly remove eldest
void afterNodeInsertion(boolean evict) {
  LinkedHashMap.Entry<K,V> first;
  //removeEldestEntry(first)默认返回false，所以afterNodeInsertion这个方法其实并不会执行
  if (evict && (first = head) != null && removeEldestEntry(first)) {
    K key = first.key;
    removeNode(hash(key), key, null, false, true);
  }
}
// hashmap的putVal方法，最后如果不是新增节点，而是对已经存在节点进行修改，调用afterNodeAccess
// 如果是新增节点，会调用linkedhashmap的newNode方法，里面将新增节点放到队列尾部
// linkedhashmap的get方法，最后如果为访问顺序，调用afterNodeAccess
void afterNodeAccess(Node<K,V> e) {
  LinkedHashMap.Entry<K,V> last;
  // accessOrder = true 时 访问节点后才需要置于尾端
// 如果e本身就在尾端，那就不需要操作
  if (accessOrder && (last = tail) != e) {
    //p指向e,b指向p的前驱,a指向p的后继
    LinkedHashMap.Entry<K,V> p =
      (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    //p的后继置null
    p.after = null;
    //b为null表示p是头节点,将head头指针指向a即p的后继
    if (b == null)
      head = a;
    //p不是头节点,将b的后继指向a
    else
      b.after = a;
    //p的后继a不为空,p不是尾节点,a的前驱指向b
    if (a != null)
      a.before = b;
    else
      last = b;
    if (last == null)
      head = p;
    else {
      p.before = last;
      last.after = p;
    }
    tail = p;
    ++modCount;
  }
}
```

LinkedHashMap并没有重写`putVal()`方法,而是重写了`afterNodeInsertion()`、`afterNodeAccess()`和`afterNodeRemoval()`方法,这三个方法在HashMap中是空的.

#### 访问顺序的遍历问题

**所有集合在迭代器模式中遍历输出时是不允许修改集合结构的**，也就是说LinkedHashMap在迭代遍历时同其他集合一样无法使用remove、put这种改变链表长度的方法

由于访问顺序的特殊性,每次有get操作获取元素时,被访问的元素会放到链表的尾端,因此LinkedHashMap在这种模式下迭代遍历时不能使用get方法,这样会改变了链表的结构，会抛出`ConcurrentModificationException`异常.

![image-20210604151205225](https://image.jianger.space/uPic/image-20210604151205225.png)

### TreeMap

![TreeMap](https://image.jianger.space/uPic/TreeMap.png)

可以看到TreeMap与其他实现类相比多出了一个实现的接口`NavigableMap`,并且它继承自`SortedMap`接口.**TreeMap不允许null key(未自定义比较器并对null处理),允许重复的null value**,由于其底层数据结构是一颗平衡二叉查找树(红黑树),因此它是有序的(**默认根据key自然排序**),同样它是非线程安全的.  

#### 数据结构

TreeMap的底层数据结构是红黑树,红黑树是一种平衡二叉搜索树.

```java
private static final boolean RED   = false;
private static final boolean BLACK = true;
static final class Entry<K,V> implements Map.Entry<K,V> {
  K key;
  V value;
  Entry<K,V> left;
  Entry<K,V> right;
  Entry<K,V> parent;
  boolean color = BLACK;
}
```

#### 添加节点

```java
private final Comparator<? super K> comparator;

private transient Entry<K,V> root;

private transient int size = 0;

private transient int modCount = 0;

@SuppressWarnings("unchecked")
final int compare(Object k1, Object k2) {
  return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
    : comparator.compare((K)k1, (K)k2);
}
//数据插入
public V put(K key, V value) {
  //t指向根节点
  Entry<K,V> t = root;
  //首次插入根节点是null
  if (t == null) {
    //判断是否传入了比较器或者key实现了Comparable接口
    compare(key, key); // type (and possibly null) check
		//根节点赋值
    root = new Entry<>(key, value, null);
    size = 1;
    modCount++;
    return null;
  }
  int cmp;
  Entry<K,V> parent;
  // split comparator and comparable paths
  Comparator<? super K> cpr = comparator;
  //new TreeMap的时候传入了自定义比较器
  if (cpr != null) {
    //从根节点开始循环遍历
    do {
      parent = t;
      //key和t.key比较
      cmp = cpr.compare(key, t.key);
      //key小于t的key,t指向左子树的根节点
      if (cmp < 0)
        t = t.left;
      //key大于t的key,t指向右子树的根节点
      else if (cmp > 0)
        t = t.right;
      else
        //找到相同key的节点,覆盖value值
        return t.setValue(value);
    } while (t != null);
  }
  //new TreeMap的时候没传入了自定义比较器
  else {
    //key为null时抛出异常
    if (key == null)
      throw new NullPointerException();
    @SuppressWarnings("unchecked")
    //实现了Comparable接口的类进行强转,未实现的会抛出异常
    Comparable<? super K> k = (Comparable<? super K>) key;
    do {
      parent = t;
      cmp = k.compareTo(t.key);
      //key小于t的key,t指向左子树的根节点
      if (cmp < 0)
        t = t.left;
      //key大于t的key,t指向右子树的根节点
      else if (cmp > 0)
        t = t.right;
      else
        //找到相同key的节点,覆盖value值
        return t.setValue(value);
    } while (t != null);
  }
  //遍历完之后如果没有遇到相同key的情况
  //构造一个 父指针指向最后一个遍历到的节点 的新节点
  Entry<K,V> e = new Entry<>(key, value, parent);
  //再根据新节点与父节点的比较给父节点的左指针或者右指针赋值
  if (cmp < 0)
    parent.left = e;
  else
    parent.right = e;
  //最后进行红黑树的调整
  fixAfterInsertion(e);
  size++;
  modCount++;
  return null;
}
```

根据源码可以看出,put操作分为插入数据和调整红黑树两步.
##### 插入数据
分为两种情况:
- 一种是树为空,直接把新节点赋值给根节点就行
- 另一种是树非空,则需要循环遍历树中的节点,将新插入的key与其比较,比它小往左子树走,比它大往右子树走,若相等则覆盖原先的value值;遍历完成后若没有相同key的情况,将新节点插入到最后遍历到的那个节点的左边或者右边.不难发现,新节点的一定是插入到原先叶子节点上的.
- 特别的,在进行元素的比较时有两种情况:
  1. 传入了自定义比较器,使用比较器重写的compare方法
  2. 未传入比较器时先判断key是否为null,是的话直接抛出异常,否则将key强转成Comparable对象,未实现Comparable接口的对象会抛出异常.

##### 调整红黑树

```java
//获取节点颜色
private static <K,V> boolean colorOf(Entry<K,V> p) {
  return (p == null ? BLACK : p.color);
}
//父节点
private static <K,V> Entry<K,V> parentOf(Entry<K,V> p) {
  return (p == null ? null: p.parent);
}
//设置节点颜色
private static <K,V> void setColor(Entry<K,V> p, boolean c) {
  if (p != null)
    p.color = c;
}
//左节点
private static <K,V> Entry<K,V> leftOf(Entry<K,V> p) {
  return (p == null) ? null: p.left;
}
//右节点
private static <K,V> Entry<K,V> rightOf(Entry<K,V> p) {
  return (p == null) ? null: p.right;
}

//左旋转
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

//右旋转
private void rotateRight(Entry<K,V> p) {
  if (p != null) {
    Entry<K,V> l = p.left;
    p.left = l.right;
    if (l.right != null) l.right.parent = p;
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

/** From CLR */
private void fixAfterInsertion(Entry<K,V> x) {
  x.color = RED;
	//x不为null不为根节点并且x的父节点为红色就一直从下往上遍历调整
  while (x != null && x != root && x.parent.color == RED) {
    //x插入左子节点
    if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
      //获取x的叔父节点
      Entry<K,V> y = rightOf(parentOf(parentOf(x)));
      //如果叔父节点是红色
      if (colorOf(y) == RED) {
        //x的父节点着色为黑色
        setColor(parentOf(x), BLACK);
        //x的叔父节点着色为黑色
        setColor(y, BLACK);
        //x的祖父节点着色为红色
        setColor(parentOf(parentOf(x)), RED);
        //x赋值为祖父节点
        x = parentOf(parentOf(x));
        //如果叔父节点是黑色
      } else {
        //判断x是否插入左子节点的右边
        if (x == rightOf(parentOf(x))) {
          x = parentOf(x);
          //对x的父节点进行一次左旋
          rotateLeft(x);
        }
        //父节点着色为黑
        setColor(parentOf(x), BLACK);
        //祖父节点着色为红
        setColor(parentOf(parentOf(x)), RED);
        //对祖父节点进行一次右旋
        rotateRight(parentOf(parentOf(x)));
      }
      //x插入右子节点,流程同上
    } else 
      Entry<K,V> y = leftOf(parentOf(parentOf(x)));
      if (colorOf(y) == RED) {
        setColor(parentOf(x), BLACK);
        setColor(y, BLACK);
        setColor(parentOf(parentOf(x)), RED);
        x = parentOf(parentOf(x));
      } else {
        if (x == leftOf(parentOf(x))) {
          x = parentOf(x);
          rotateRight(x);
        }
        setColor(parentOf(x), BLACK);
        setColor(parentOf(parentOf(x)), RED);
        rotateLeft(parentOf(parentOf(x)));
      }
    }
  }
  root.color = BLACK;
}
```



#### 自然排序
Comparable接口强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序  
例如String实现了Comparable接口,可以看到它会逐个字符比较,负数表示小于,正数表示大于,0为相等. 如果key是自定义的类,既没有实现Comparable接口,也没有传入比较器,TreeMap是无法添加的.

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence{
  
  public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
      char c1 = v1[k];
      char c2 = v2[k];
      if (c1 != c2) {
        return c1 - c2;
      }
      k++;
    }
    return len1 - len2;
  }
}
```

#### 定制排序

那么如果不想使用上面那种排序比较方式呢,比如我想小于的时候返回正数,反之返回负数,如果是自己写的类还好,随便怎么实现Comparable接口,像String这种写好的类呢,方法也是有的,定义一个Comparator比较器,声明新TreeMap时将这个定制器作为参数传入即可.

```java
Comparator comparator = (o1, o2) -> {
  if(o1 instanceof String && o1 instanceof String) {
    String p = (String)o1;
    String p2 = (String)o2;
    //逆序
    return -p.compareTo(p2);
  }
  throw new RuntimeException("输入的参数不符合要求");
};

TreeMap treeMap = new TreeMap(comparator);
```



### HashTable

![Hashtable](https://image.jianger.space/uPic/Hashtable.png)

### 参考来源

-    [阅读 JDK 源码：HashMap 扩容总结及图解 - SegmentFault 思否](https://segmentfault.com/a/1190000039302830) 
-    [Java面试必问之Hashmap底层实现原理(JDK1.8) - SegmentFault 思否](https://segmentfault.com/a/1190000021928659?utm_source=sf-similar-article) 
-     [HashMap-split()方法源码简读(JDK1.8)_绅士jiejie的博客-CSDN博客_hashmap split](https://blog.csdn.net/weixin_38106322/article/details/104422422) 
-      [Java 8系列之重新认识HashMap - 知乎](https://zhuanlan.zhihu.com/p/21673805) 

