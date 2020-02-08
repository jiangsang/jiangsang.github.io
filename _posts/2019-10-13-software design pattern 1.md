---
layout: post
title: 设计模式学习笔记（一）
categories:
 - Java
 - 笔记
tags:
 - 设计模式
---

软件工程中，设计模式是对软件设计中普遍存在的各种问题，所提出的解决方案。该术语是由埃里希·伽玛等人从建筑设计领域引入到计算机科学的。

### 为什么需要设计模式

假设以下情景：

- 当一个项目开发完，如果客户提出新功能，怎么办?
- 如果项目开发完后，原来程序员离职，你接手维护该项目怎么办？

显然，设计模式对于软件工程来说为项目的编程开发提供的方案和标准，根据项目情况，在项目之初就计划好采用的模式，就像建筑之初就画好设计图，使得项目开发和维护秩序井然，极大的提高了项目开发的复用性、拓展性、稳定性。这是软件工程发展到一定程度的必然要求。

<!-- more -->

### 哪里会用到设计模式

---

设计模式究竟在软件开发过程的哪个阶段得到运用呢？

主要在两个方面：

- 功能模块中，使用到设计模式和数据结构
- 框架使用中，框架源码中使用到多种设计模式，例如SpringMVC、Mybatis



### 设计模式的六大原则

---

#### 单一职责原则

对类来说，即一个类应该只负责一项职责，各行其是，以便提高类的可读性和可读性。

以交通工具为例：汽车类就负责汽车运行的职责（方法），而飞机类就飞机运行的职责  



#### 开闭原则（OCP原则）

1. 开闭原则是编程中最基础、最重要的设计原则
2. **一个软件实体如类，模块和函数应该对拓展开放（对提供方），对修改关闭（对使用方）。用抽象构建框架，用实现拓展细节。**
3. 当软件需要变化时，尽量通过拓展软件实体的行为来实现变化，而不是通过修改原有代码来实现变化。
4. 编程中遵循其他原则，以及使用设计模式的目的就是遵循开闭原则。



#### 里氏替换原则

如此命名是由于这是一位姓里的提出来的。。

定义：如果对每个类型为T1的对象o1，都有类型为T2的对象o2，使得以T1定义的所有程序P在所有的对象o1都代换成o2时，程序P的行为没有发生变化，那么类型T2时类型T1的子类型。换句话说，所有引用基类的地方必须能透明地使用其子类的对象。



#### 依赖倒置原则

1. 高层模块不应该低层模块，二者都应该依赖其抽象
2. **抽象不应该依赖细节，细节应该依赖抽象**
3. 依赖倒置的中心思想是面向接口编程

Java中，抽象指的是接口和抽象类，细节就是具体的实现类。使用接口的目的就是制定好规范，不涉及具体的操作，把展现细节交给它们的实现类去完成。这样可以为之后项目的拓展开发提供了便利。

**依赖关系传递的三种方式：**

- 接口传递
- 构造方法传递
- setter方式传递

**注意事项和细节：**

1. 低层模块尽量都要有抽象类或接口，或者两者都有，程序稳定性更好。
2. **变量的声明类型尽量是抽象类或接口，利于程序拓展和优化。**
3. 继承时遵循里氏替换原则  



#### 接口隔离原则

客户端不应该依赖它不需要的接口，即一个类对另一个类的依赖应该建立在最小的接口上。

例如一个类只依赖接口中的两个方法（共三个），那么该接口应该改成两个，类依赖的两个方法构成一个接口，另外的方法构成另一个接口。类所依赖的接口应该是最小的，没有额外的，这样即是接口隔离。



#### 迪米特原则（最少知道原则）

1. 一个对象应该对其他对象保持最少的了解。
2. 类与类关系越密切，耦合度越大
3. 一个类对自己依赖的类知道的越少越好。也就是说，对于被依赖的类不管多么复杂，都尽量将逻辑封装在类的内部。对外除了提供的public方法，不对外泄露任何信息。

注意事项和细节：

- 迪米特法则的核心是降低类之间的耦合
- 但是不是由于每个类都减少了不必要的依赖，因此迪米特法则只是要求降低类间耦合关系，并非要求完全没有依赖。



### 设计原则的核心思想

- 找出应用中可能需要变化之处，把它们独立出来，不要和那些不需要变化的代码混合在一起。
- 针对接口编程，而不是针对实现编程。
- 为了交互对象之间的**松耦合设计**而努力