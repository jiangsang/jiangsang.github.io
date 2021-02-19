---
layout: post
title: 读懂Java集合（二）
categories:
 - Java
tags:
 - Java集合
---

![](https://image.jianger.space/uPic/Java常用集合.png)

<!-- more -->

本文承接上文[读懂Java集合（一）](https://jianger.space/java-collections/)，主要整理总结一下集合框架List接口的常用实现。

> 💡图中只列出了常见的，省略了部分子接口，本文内容基于JDK 1.8

### 

List翻译过来即是列表、目录之意，它是**有序、可重复的**，其逻辑结构分为线性和非线性，线性的有ArrayList数组列表，非线性的有LinkedList链表。因此ArrayList适合查询，删除和增加操作复杂度较高，而链表恰恰相反，适用于删除和增加操作较多的场景，它只需要变动删除元素前后两个元素。

> 此处的有序通常指插入顺序,元素的顺序是固定的

### ArrayList

![ArrayList](https://image.jianger.space/uPic/ArrayList.png)

对List接口可调整大小的数组实现，运行操作包含所有元素（或对象），包括`null`，它基于Object[]数组实现，提供了操作用于内部存储列表的数组大小的方法，初始化时可以指定数组大小，未指定默认大小为10，**但不是线性安全的**。

#### 所有方法

| 修饰符和类型         | 方法和描述                                                   |
| :------------------- | :----------------------------------------------------------- |
| `boolean`            | `add ( E e)`将指定的元素添加到列表的末尾                     |
| `void`               | `add (int index, E element)`将指定的元素插入到此列表中的指定位置 |
| `boolean`            | `addAll ( Collection <? extends E > c)`将指定集合中的所有元素按指定集合的 Iterator 返回的顺序追加到此列表的末尾 |
| `boolean`            | `addAll (int index, Collection <? extends E > c)`从指定位置开始，将指定集合中的所有元素插入到此列表中 |
| `void`               | `clear ()`从此列表中移除所有元素                             |
| `Object`             | `clone ()`返回此操作的浅拷贝实例                             |
| `boolean`            | `contains ( Object o)` 如果这个列表包含指定的元素，返回True，否则False |
| `void`               | `ensureCapacity (int minCapacity)`增加这个的容量 实例，以确保它至少可以保存由最小容量参数指定的元素数 |
| `void`               | `forEach ( Consumer <? super E > action)`对象的每个元素执行给定的操作 直到所有元素都处理完毕，或者操作抛出异常 |
| `E`                  | `get (int index)`返回此列表中指定位置的元素                  |
| `int`                | `indexOf ( Object o)`返回此列表中指定元素的第一个匹配项的索引，如果此列表不包含该元素，则返回 -1 |
| `boolean`            | `isEmpty ()`如果这个列表为空，返回True                       |
| `Iterator < E >`     | `iterator ()`以正确的顺序返回对列表中元素的迭代器            |
| `int`                | `lastIndexOf ( Object o)`返回此列表中指定元素的最后一个匹配项的索引，如果此列表不包含该元素，则返回 -1 |
| `ListIterator < E >` | `listIterator ()`返回对列表中元素的列表迭代器(以正确顺序)    |
| `ListIterator < E >` | `listIterator (int index)`从列表中的指定位置开始，返回对列表中元素的列表迭代器(以正确顺序) |
| `E`                  | `remove (int index)`移除此列表中指定位置的元素               |
| `boolean`            | `remove ( Object o)`从列表中移除指定元素的第一个匹配项(如果存在) |
| `boolean`            | `removeAll ( Collection <?> c)`从此列表中移除指定集合中包含的所有元素 |
| `boolean`            | `removeIf ( Predicate <? super E > filter)`移除此集合中满足给定断言的所有元素 |
| `protected void`     | `removeRange (int fromIndex, int toIndex)`从此列表中移除索引在 fromIndex (包含)和 toIndex (不包含)之间的所有元素。 |
| `void`               | `replaceAll ( UnaryOperator < E > operator)`将此列表中的每个元素替换为对该元素应用运算符的结果 |
| `boolean`            | `retainAll ( Collection <?> c)`仅保留列表中包含在指定集合中的元素 |
| `E`                  | `set (int index, E element)`将列表中指定位置的元素替换为指定的元素 |
| `int`                | `size ()`返回此列表中的元素数                                |
| `void`               | `sort ( Comparator <? super E > c)`对象导出的顺序排序此列表[ `Comparator `](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html). |
| `Spliterator < E >`  | `spliterator ()`对此列表中的元素创建后期绑定和快速失败拼接器 |
| `List < E >`         | `subList (int fromIndex, int toIndex)`返回索引在fromIndex (包含)到toIndex (不包含)之间的视图。 |
| `Object []`          | `toArray ()`返回一个数组，该数组以正确的顺序(从第一个元素到最后一个元素)包含此列表中的所有元素 |
| `<T> T[]`            | `toArray (T[] a)`返回一个数组，该数组以正确的顺序(从第一个元素到最后一个元素)包含列表中的所有元素; 返回数组的运行时类型是指定数组的运行时类型 |
| `void`               | `trimToSize ()`调整这个的实例的容量为列表的当前大小          |

使用add方法可以将元素添加到数组列表中，可以指定位置或者添加到末尾，最终，数组的全部空间有可能被用尽。这就显现出数组列表的操作魅力：如果调用add且内部数组已经满了，数组列表就将自动地创建一个更大的数组，并将所有的对象从较小的数组中拷贝到较大的数组中。那么，具体是怎么实现的呢？这里就涉及到ArrayList的扩容机制了.

#### ArrayList的扩容机制

先上源码

```java
private int size;
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
private static int calculateCapacity(Object[] elementData, int minCapacity) {
  if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
    return Math.max(DEFAULT_CAPACITY, minCapacity);
  }
  return minCapacity;
}
private void ensureCapacityInternal(int minCapacity) {
  ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
private void ensureExplicitCapacity(int minCapacity) {
  modCount++;

  // overflow-conscious code
  if (minCapacity - elementData.length > 0)
    grow(minCapacity);
}
private void grow(int minCapacity) {
  // overflow-conscious code
  int oldCapacity = elementData.length;
  int newCapacity = oldCapacity + (oldCapacity >> 1);
  if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
  if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
  // minCapacity is usually close to size, so this is a win:
  elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
  if (minCapacity < 0) // overflow
    throw new OutOfMemoryError();
  return (minCapacity > MAX_ARRAY_SIZE) ?
    Integer.MAX_VALUE :
  MAX_ARRAY_SIZE;
}
```

可以看到，ArrayList在进行增加元素操作之前，会先对数组的容量进行判断，超过就会越界添加失败。

流程是这样的：

1. 首先计算出所需最小容量，一般就是给定的，如果是首次添加元素，判断默认容量和给定容量谁更大
2. 如果所需最小容量大于当前数组容量，对数组进行扩容，新的容量`newCapacity = oldCapacity + (oldCapacity >> 1)`，>>为右位移符号，相当于`newCapacity = oldCapacity *1.5`
3. 复制原数组，使得复制后的数组容量为新的容量newCapacity，那些没有值的位置会填充元素的默认值

### LinkedList

![LinkedList](https://image.jianger.space/uPic/LinkedList.png)

对List接口的一种双向链表的实现，跟ArrayList一样，元素也是有序和可重复的，可以包含`null`,**并且不是线程安全的**.如果需要使其线程安全,可以通过使用`List list = Collections.synchronizedList(new LinkedList(...))`的方式实现

**关键数据结构**

```java
private static class Node<E> {
  E item;
  Node<E> next;
  Node<E> prev;

  Node(Node<E> prev, E element, Node<E> next) {
    this.item = element;
    this.next = next;
    this.prev = prev;
  }
}
```

存储元素过程中，无需像 ArrayList 那样进行扩容,但是需要额外的空间存储前序和后序结点的引用,可以看到,LinkedList类内部有一个内部类Node,包括指向下一个元素的 next 和 指向上一个元素的prev.LinkedList 在链表头部和尾部插入效率比较高，但在指定位置进行插入时，需要从头开始遍历,效率一般.

#### 内部类

内部类（inner class）是定义在另一个类中的类。

为什么需要使用内部类呢？其主要原因有以下三点：

- 内部类方法可以访问该类定义所在的作用域中的数据，包括私有的数据。

- 内部类可以对同一个包中的其他类隐藏起来。

- 当想要定义一个回调函数且不想编写大量代码时，使用匿名（anonymous）内部类比较便捷。

内部类分为四类:匿名内部类、静态内部类、局部内部类、成员内部类

### 参考来源

-   阿里巴巴Java开发手册
-   Java核心技术第10版

