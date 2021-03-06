---
layout: post
title: 字符串模式匹配算法详解
categories:
 - 数据结构
tags:
 -
---

### 前言

考研期间复习到了模式匹配算法，本系列博客旨在强化模式匹配的相关知识，以及方便以后的快速拾起本块知识，故整理以记之。主要包括普通模式匹配算法和KMP算法以及对二者的比较总结。

<!-- more -->
### 简单模式匹配算法

即BF(Brute Force)算法，是一种普通的模式匹配算法。**算法的基本思想**：从主串的第一个位置起和模式串的第一个字符开始比较，如果相等，则继续逐一比较两个字符串的后续字符；否则接着从主串的第二个字符开始，再重新用上一步方法与模式串的字符做比较，依次类推，直到比较完模式串中的所有字符。

**算法的时间复杂度**：最坏情况o(mn)，即前面的匹配每次都是最后一个字符不匹配，只在最后才匹配成功；

**图解实例：**

**模式串：ABCFGDS，主串：CFG**

**代码实现（C/C++）：**

```c
#include<stdio.h>
#include<stdlib.h>
#define MAXSIZE 100
//变长分配结构体定义
typedef struct
{
    char *ch;
    int length;
}Str;
//初始化变长分配的字符串
Str init()
{
    Str s;
    s.length=0;
    s.ch=(char*)malloc(MAXSIZE *sizeof(char));
    return s;
}
//匹配成功返回模式串在主串中的位置下标，反之返回0
int index(Str str,Str substr)
{
    int i=0,j=0,k=i;
    while(i<str.length&&j<substr.length)
    {
        if(str.ch[i]==substr.ch[j])
        {
            ++i;
            ++j;
        }
        else
        {
            j=0;
            i=++k;
        }
    }
    if(j>=substr.length)
        return k;
    else return 0;
}
int main()
{
    Str a=init();
    Str b=init();
    char ch[]="asfbg";
    a.ch=ch;
    a.length=5;
    char ch1[]="fb";
    b.ch=ch1;
    b.length=2;
    printf("%d",index(a,b));
    return 0;
}
```

**BF算法小结：**可以看到BF算法优缺点很明显，优点是简单粗暴、具有广泛适用性；缺点是效率低，每次失配都只往后移一位（理解上的后移）再对比，只适用于解决较小问题，规模大了速度就慢了。那么对于模式匹配而言，是否有更为精巧绝妙的方法呢，接下来需要厘清的就是是更为高效好用的KMP算法。

### KMP算法

Knuth-Morris-Pratt 字符串查找算法，简称为 “KMP算法”，常用于在一个主串S内查找一个模式串P 的出现位置，这个算法由Donald Knuth、Vaughan Pratt、James H. Morris三人于1977年联合发表，故取这3人的姓氏命名此算法。

理解KMP算法，先需要了解几个概念：

**字符串的前缀和后缀：**如果有字符串A，存在A=B+S，其中S是任意的非空字符串，那就称B为A的前缀;若存在A=S+B那就称B为A的后缀。例如有字符串S="bsabs",其前缀包括{"b","bs","bsa","bsab"},其后缀包括{"sabs","abs","bs","s"}**。字符串的前缀集合与后缀集合的交集中最长元素一定在字符串的两端，**如本例的**“bs**a**bs”。需要要注意的是，字符串本身并不是自己的前缀或后缀。**

**KMT（Partial Match Table）部分匹配表：这是KMP算法的核心，一般用next数组保存。**next[j]（０为起始下标）的值是**模式串的前j个字符的前缀集合与后缀集合的交集中最长元素的长度**。例如有模式串P="abcas"，next[0]比较特殊，设置为-1（为什么是-1后面会讲），next[1]的值就是计算字符串"a"的前缀集合与后缀集合的交集中最长元素的长度，．．．易得next[1]＝next[2]＝next[3]＝0，next[4]＝1；**next[j]的一个重要作用在于发生失配时指示模式串j的下一步位置（很重要）。**

接下来是KMP的算法流程：

1. 匹配时操作与暴力方法没有不同，都是接着对比下一字符；
2. 如果出现失配情况，主串i不变，模式串j=next[j]，然后接着比较； 
3. 当j大于模式串的长度时，算法结束。

那么为甚么出现失配时主串i不变，j=next[j]呢？以下图为例（来源网络）：

　　![img](https://images0.cnblogs.com/blog/416010/201308/17084030-82e4b71b85a440c5a636d57503931415.png)

　　主串S="ABCABCDHIJK"，模式串P="ABCABB“，计算到到next[6]={-1,0,0,0,1,2};

　　可以看到当i=j=5时出现了失配，那么接下来j移向何处呢（实际只是j的值变化了）？

　　看对应next[5]=2，j应该移动到下标2也就是第三个字符C处，事实也是如此。

　　![img](https://images0.cnblogs.com/blog/416010/201308/17084037-cc3c34200809414e9421c316ceba2cda.png)

　　仔细观察不难发现，因为此时j的前两位已经匹配了，直接比对第三位字符即可。为什么能够这么做呢，**因为失配时，j要移动的下一个位置next[j],有这样的性质：模式串的前next[j]个字符与j（还未将next[j]赋值给j)之前的最后next[j]个字符一致**。所以总结到最后就是为什么此时模式串的前两位字符一定会相等，我们通过数学解题步骤来理解：

　　已知i=j=5时失配,则S[0~4]=P[0~4];

　　又因为next[5]=2,即P[0~1]=P[3~4]

　　所以S[3~4]=P[0~1]，故所求得证。

　　上述过程是一个实例，将具体数字参数化同样成立。那么这个next数组如何求得呢，下面贴出书籍上普遍的一种（C/C++)：

```c
//变长分配结构体定义字符串
typedef struct
{
    char *ch;
    int length;
}Str;
int getnext(Str substr,int next[])
{
    int j=0,k=0;
    next[0]=-1;
    while(j<substr.length)
    {
        if(k==-1||substr.ch[j]==substr.ch[k])
        {
            next[++j]=++k;
        }
        else
            k=next[k];
    }
    return next;
}
```

我看到next[++i]=++j和j=next[j]这两句一头雾水，怎么来的？为什么这样做？同样通过一个实例讲解（图源网络）：

　![img](https://images0.cnblogs.com/blog/416010/201308/17084327-8a3cdfab03094bfa9e5cace26796cae5.png)![img](https://images0.cnblogs.com/blog/416010/201308/17084342-616036472ab546c082aa991004bb0034.png)

通过观察容易知道，当P[j]==P[k]时，next[j+1]=next[j]+1=k+1;

那如果P[k] != P[j]呢？比如下图所示：

![img](https://images0.cnblogs.com/blog/416010/201308/17122358-fd7e52dd382c4268a8ff52b85bff465d.png) 

像这种情况，如果你从代码上看应该是这一句：k = next[k];为什么是这样子？你看下面应该就明白了。

 ![img](https://images0.cnblogs.com/blog/416010/201308/17122439-e349fed25e974e7886a27d18871ae48a.png)
