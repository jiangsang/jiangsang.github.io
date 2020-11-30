---
layout: post
title: 两文读懂Java集合（一）
categories:
 - Java
tags:
 - Java集合
---

![](http://image.jianger.space/uPic/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6.png)

<!-- more -->

> 💡：图中只列出了常用的

### 概述

如果数组只含有数量固定的对象，使用数组相对简单， 但若程序有这样的要求：只有运行时才能知道对象的类型或数量而且需要我们把这些对象存在某种容器中。此时数组无法胜任，我们就必须用到集合框架。Java自带的集合框架给了我们一些集合类作为工具。利用这些集合类，我们可以维护一组类型相同的自定义对象。

Java集合框架包括Collection和Map两大类：  
`Collection`：盛放一组单独的元素，比如，一个List（列表）必须按特定的顺序容纳元素，而一个Set（集）不可包含任何重复的元素。  
`Map`：一系列“键－值”对。

> 💡：以下内容基于JDK 1.8



### Collection

```java
public interface Collection<E> extends Iterable<E>
```

Collection是集合层次结构中的根接口，JDK不提供此接口的任何直接实现：它提供更具体的子接口（例如Set和List）的实现。

#### Iterable<E>

看代码可以知道Collection继承了Iterable接口，它主要的功能便是提供了迭代器的接口，Collection继承了迭代器的接口，因此Collection各个实现类都可以通过ForEach()或者Iterator迭代器遍历。

| 方法                                               | 说明                                                         |
| :------------------------------------------------- | :----------------------------------------------------------- |
| `default void forEach(Consumer<? super T> action)` | 为集合中每个元素执行给定的操作，直到所有元素都已处理或该操作引发异常。 |
| `Iterator<T> iterator()`                           | 返回一个迭代器。                                             |
| `default Spliterator<T> spliterator()`             | 一个为了并行遍历元素而设计的可分割迭代器。                   |

##### forEach()和Iterator迭代器

```java
//for循环遍历
for(int i = 0;i < arr.length;i++){
    System.out.println(arr[i]);
}
//forEach遍历方式
for(String str : arr){
    System.out.println(str);
}
//Iterator迭代器遍历
Iterator<String> it = list.iterator();
while (it.hasNext()){
    System.out.println(it.next());
}
```

**重要知识**

- forEach 底层采用iterator实现。 

-  不要在 forEach 循环里进行元素的 remove / add 操作，会报错，原因出在属性`modCount`的修改。
- 需要在List循环中remove就使用Iterator的remove()方法，或者使用listIterator迭代器，或者直接使用for循环遍历
- 如果要并发操作，需要对Iterator对象加锁

##### Iterable与Iterator关系

为什么不直接将Iterator中的hasNext()，next()方法放在Iterable接口中，其他类直接实现呢？
原因是有些集合类可能不止一种遍历方式，实现了Iterable的类可以再实现多个Iterator内部类，例如`LinkedList`中的`ListItr`和`DescendingIterator`两个内部类，就分别实现了双向遍历和逆序遍历。通过返回不同的`Iterator`实现不同的遍历方式，这样更加灵活。如果把两个接口合并，就没法返回不同的`Iterator`实现类了。

#### 所有方法

> 💡：已省略Iterable中含有的接口方法

| 修饰符和类型                                            | 方法和说明                                                   |
| :------------------------------------------------------ | :----------------------------------------------------------- |
| `boolean add(E e)`                                      | 将指定的元素追加到集合的末尾。                               |
| `boolean addAll(Collection<? extends E> c)`             | 将指定集合中的所有元素添加到此集合中。                       |
| `void clear()`                                          | 从此集合中删除所有元素。                                     |
| `boolean contains(Object o)`                            | 如果此集合包含指定的元素，则返回`true`。                     |
| `boolean containsAll(Collection<?> c)`                  | 如果此集合包含指定集合中的所有元素，则返回`true`。           |
| `boolean equals(Object o)`                              | 比较指定对象与此集合的相等性。                               |
| `int hashCode()`                                        | 返回此集合的哈希码值。                                       |
| `boolean isEmpty()`                                     | 如果此集合不包含任何元素，则返回`true`。                     |
| `default Stream<E> parallelStream()`                    | 返回`Stream`与此集合可能平行的源。                           |
| `boolean remove(Object o)`                              | 如果存在给定元素，则从此集合中删除该元素。                   |
| `boolean removeAll(Collection<?> c)`                    | 删除此集合中包含在指定集合里的所有的元素。                   |
| `default boolean removeIf(Predicate<? super E> filter)` | 删除此集合中满足给定谓词的所有元素。                         |
| `boolean retainAll(Collection<?> c)`                    | 仅保留此集合中包含在指定集合中的元素。                       |
| `int size()`                                            | 返回此集合中的元素数。                                       |
| `default Stream<E> stream()`                            | 返回`Stream`以该集合为源的序列。                             |
| `Object[] toArray()`                                    | 返回一个包含此集合中所有元素的数组。                         |
| `<T> T[] toArray(T[] a)`                                | 返回一个包含此集合中所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。 |

注意：从JDK 1.8开始添加的一个新特性便是可以使用默认方法扩展现有接口，默认方法不仅被声明，而且还在接口中定义。以上由`default`修饰符修饰的就是接口的默认方法。



#### List

一种有序集合（也称为序列），这里的**有序**是指所存储的元素都有相对应的有序的下标对应，下标从0开始依次递增。该接口可以精确控制列表中每个元素的插入位置。用户可以通过其整数索引（列表中的位置）访问元素，并在列表中搜索元素 。与Set不同，List通常允许重复的元素。**特别的，接口提供了一个称为ListIterator的特殊的迭代器，它允许元素插入、替换、双向访问以及 Iterator 接口提供的正常操作。**

##### 所有实现类

- [AbstractList](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractList.html)
- [AbstractSequentialList](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractSequentialList.html)
- [ArrayList](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html)
- [AttributeList](https://docs.oracle.com/javase/8/docs/api/javax/management/AttributeList.html)
- [CopyOnWriteArrayList](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CopyOnWriteArrayList.html)
- [LinkedList](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)
- [RoleList](https://docs.oracle.com/javase/8/docs/api/javax/management/relation/RoleList.html)
- [RoleUnresolvedList](https://docs.oracle.com/javase/8/docs/api/javax/management/relation/RoleUnresolvedList.html)
- [Stack](https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html)
-  [Vector](https://docs.oracle.com/javase/8/docs/api/java/util/Vector.html)

##### 独有方法

| 方法                                                 | 说明                                                         |
| :--------------------------------------------------- | :----------------------------------------------------------- |
| `E get(int index)`                                   | 返回此列表中指定位置的元素。                                 |
| `E remove(int index)`                                | 删除此列表中指定位置的元素，并返回在指定位置的元素。         |
| `int indexOf(Object o)`                              | 返回指定元素在此列表中首次出现的索引；如果此列表不包含该元素，则返回-1。 |
| `int lastIndexOf(Object o)`                          | 返回指定元素在此列表中最后一次出现的索引；如果此列表不包含该元素，则返回-1。 |
| `ListIterator<E> listIterator()`                     | 返回此列表中的元素的列表迭代器（按适当顺序）。               |
| `ListIterator<E> listIterator(int index)`            | 从列表中的指定位置开始，以适当的顺序返回在此列表中的元素上的列表迭代器。 |
| `default void replaceAll(UnaryOperator<E> operator)` | 用将该运算符应用于该元素的结果替换此列表中的每个元素。       |
| `E set(int index, E element)`                        | 用指定的元素替换此列表中指定位置的元素（可选操作）。         |
| `default void sort(Comparator<? super E> c)`         | 根据指定的诱导顺序对列表进行排序 [`Comparator`](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html)。 |
| `List<E> subList(int fromIndex, int toIndex)`        | 返回此列表中指定的`fromIndex`（包括）和`toIndex`（不包括）之间的子列表。 |

#### Set

一种**不包含重复元素**的集合，至多包含一个`null`元素，所含接口方法见Collection的方法。

##### 所有实现类

- [AbstractSet](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractSet.html) 
- [ConcurrentHashMap.KeySetView](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.KeySetView.html)
- [ConcurrentSkipListSet](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentSkipListSet.html)
- [CopyOnWriteArraySet](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CopyOnWriteArraySet.html)
- [EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html)
- [HashSet](https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html)
- [JobStateReasons](https://docs.oracle.com/javase/8/docs/api/javax/print/attribute/standard/JobStateReasons.html)
- [LinkedHashSet](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html)
- [TreeSet](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html)

#### Queue

一种用来存放**等待处理元素**的集合，这种场景一般用于缓冲、并发访问。值得注意的是，添加、删除、获取这些操作在该接口中提供了两种形式。

| 操作 | 说明                         | 抛出异常的方法                     | 返回特殊值的方法                        |
| ---- | ---------------------------- | ---------------------------------- | --------------------------------------- |
| 添加 | 添加新元素                   | `boolean add(e)`，添加失败抛出异常 | `boolean offer(e)`，添加失败返回`false` |
| 删除 | 删除队列头部元素并返回新头部 | `E remove()`，队列为空时抛出异常   | `E poll()`,队列为空时返回`null`         |
| 获取 | 获取队列头部元素             | `E element()`，队列为空时抛出异常  | `E peek()`，队列空时返回`null`          |

##### 重要知识

- 虽然实现类LinkedList 没有禁止添加 null，但是一般情况下 Queue 的实现类都不允许添加 null 元素，因为 poll(), peek() 方法在操作失败的时候会返回 null，添加了null以后，操作究竟成功与否不好判断，因为此时成功与失败的返回值都为null。
- Queue 一般都是 FIFO 的，但是也有例外，比如优先队列 priority queue（它的顺序是根据自然排序或者自定义 comparator 的）；再比如 LIFO 的队列（跟栈一样，后来进去的先出去）。
- 不论进入、出去的先后顺序是怎样的，使用 remove()，poll() 方法操作的都是头部的元素；而添加的位置则不一定是在队尾了，不同的 queue 会有不同的插入逻辑。

##### 所有实现类

- [AbstractQueue](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractQueue.html)
- [ArrayBlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ArrayBlockingQueue.html)
- [ArrayDeque](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html)
- [ConcurrentLinkedDeque](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentLinkedDeque.html)
- [ConcurrentLinkedQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentLinkedQueue.html)
- [DelayQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/DelayQueue.html)
- [LinkedBlockingDeque](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedBlockingDeque.html)
- [LinkedBlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedBlockingQueue.html) 
- [LinkedList](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)
- [LinkedTransferQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedTransferQueue.html)
- [PriorityBlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/PriorityBlockingQueue.html)
- [PriorityQueue](https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html)
- [SynchronousQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/SynchronousQueue.html)



### Map

### 拓展知识

#### 抽象类和接口

未完待续

### 参考来源

-   [Java中的Iterable与Iterator详解](https://www.cnblogs.com/litexy/p/9744241.html) - xinyuexy - 博客园
-    [java.util (Java Platform SE 8 )](https://docs.oracle.com/javase/8/docs/api/java/util/package-summary.html) - oracle
-    [Outline of the Collections Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/reference.html) - oracle
-    [foreach循环中为什么不要进行remove/add操作](https://www.cnblogs.com/luyu1993/p/7148765.html)  - 丨核桃牛奶 - 博客园
-    [for 、foreach 、iterator 三种遍历方式的比较](https://www.cnblogs.com/cxuanBlog/p/10927538.html)  - 程序员cxuan - 博客园
-   阿里巴巴Java开发手册



