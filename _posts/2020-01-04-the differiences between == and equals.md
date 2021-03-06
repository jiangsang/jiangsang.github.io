---
layout: post
title: ==和equals的比较
categories:
 - Java
tags:
 - Java

---

==和equals在Java中的使用频率颇高，自己之前使用总是搞混，因此有必要做一个总结。

比较之前，先瞧瞧以下两个概念：

**值类型（基本数据类型）**：值类型的变量存储的是值本身

四类八种：

- 整型：int、short、long、byte
- 浮点型：float、double
- 字符型：char
- 布尔型:boolean

<!-- more -->


**引用类型**：引用类型的变量存储所指向对象的内存地址

除了四类八种的基本数据类型之外，所有的类型都是引用类型（数组、字符串、类、接口等）。



### ==

它的作用是判断两个对象的值是不是相等。具体来看，如果两个对象是值类型（基本数据类型），==比较的就是它们的内容；如果两个对象是引用类型，==比较的就是它们的内存地址。



### equals() 

它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

- 情况1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

通过以下代码加深理解

```java
public class test {

	public static void main(String[] args) {
		String a = new String("ab"); //a为一个引用，指向对象内容为ab
		String b = new String("ab"); //b为另一个引用，指向对象内容同a
		String aa = "ab";
		String bb = "ab";
		if (aa == bb) // true 
			System.out.println("aa==bb"); 
		if (a == b) // false，string类型比较的内存地址，不输出
			System.out.println("a==b"); 
		if (a.equals(b)) // true 
			System.out.println("aEQb"); 
		if (42 == 42.0) { // true 
			System.out.println("true"); 
			}
		}
}

```

![](https://article-1300776923.cos.ap-chengdu.myqcloud.com/equals%26%3D.PNG)

> 注意： 每次new String("ab")都创建了一个新的对象；String 中的 equals 方法是被重写过的，因为 object 类中的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
>
> 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。所以aa==bb为true