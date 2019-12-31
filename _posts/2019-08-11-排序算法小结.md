---
title: 排序算法小结
description: 排序算法伪码分析和小结
categories:
 - 数据结构
tags:
 - 排序算法
---

### 前言

计算机考研绕不过数据结构，数据结构内容说多不多：也就线性表、数组、字符串、栈和队列、图、树、排序、查找等。今天主要回顾一下排序算法中的相关思想。

### 简单选择算法

```c
void selectsort(int R[],int n)
{
    int temp;
    int i,j,k;
    for(i=0;i<n;i++)
    {
	k=i;
	for(j=i+1;j<n;j++)
	{
	    if(R[j]>R[k];
		K=j;
	}
	temp=R[i];
	R[i]=R[k];
	R[k]=temp;
    }
}
```

观察代码容易发现，两层循环的**执行次数和初始序列无关**，内层每次循环获得无序列表中的最大值的下标，外层循环每次将无序列表的第一个和最大值进行对调。其时间复杂度平均和最坏均为\\(o(n^2)\\)，因为元素对调操作在外循环中，与if判断条件无关

### 起泡排序（冒泡排序）

```c
void BubbleSort(int R[],int n)
{
    int i,j,flag;
    int temp;
    for (i=n-1;i>0;--i)
    {
	flag=0;
	for(j=1;j<i;++j)
	{
            if(R[j-1]>R[j])
	    {
		temp=R[j];
		R[j]=R[j-1];
		R[j-1]=temp;
		flag=1;
	    }
	}
    if(flag==0)
    {
	return;
    }
}
```

观察代码容易发现，冒泡排序也有两个循环，外循环表示趟数，每循环一次，通过内循环的对调操作，将无序中的最大值调换到最后，若没有发生元素对调即表示数列已经有序了。犹豫对调操作在判断条件内，故最好情况只是外循环遍历一次，而内循环不满足条件无需操作，时间复杂度为o(n)，平均和最坏情况皆为\\(o(n^2)\\)。

### 快速排序

```c
Void QuickSort(int R[],int low,int high)
{
    int temp;
    int i=low,j=high;
    if(low<high)
    {
	temp=R[low];//每次都将给定数组的第一位为比较对象。
	while(i<j)//本循环加上R[i]=temp;即为快速排序的一趟
	{
	    while(j>i&&R[j]>=temp)--j;
	    if(i<j)//从右侧找到了小于temp的关键字
	    {
		R[i]=R[j];
		++i;
	    }
	    while(j>i&&R[i]<temp)++i;
	    if(i<j)//从左侧找到了大于temp的关键字
	    {
		R[j]=R[i];
		--j;
	    }
	}
	R[i]=temp;//调换结束，temp放在最终位置
	QuickSort(R,low,i-1);//递归对temp左边进行排序
	QuickSort(R,i+1,high);//递归对temp右边排序
    }
}
```

观察代码易得，程序时间复杂度基本取决于`while(j>i&&R[j]>=temp)--j`;,最好情况和平均情况都为\\(o(nlog_2n)\\),最坏情况为每一趟从右往左都未找到小于temp的值，复杂度为\\(o(n^2)\\)

### 堆排序
```c
/*本函数完成在数组R[low]到R[high]的范围内对在位置low上的结点进行调整*/
void Sift(int R[],int low,int high)
{
    int i=low,j=i*2;
    int temp=R[i];
    while(j<=high)
    {
        if(j<high&&R[j]<R[j+1])
            ++j;                            //j变为2*i+1 
        if(temp<R[j])
        {
            R[i]=R[j];
            i=j;
            j=2*i;
        }
      else
          break;
    }
    R[i]=temp;
}
/*堆排序函数*/
void heapSort(int R[],int n)
{
    int i;
    int temp;
    for(i=n/2;i>1;--i)
        Shift(R,i,n)
    for(i=n;i>=2;—i)//建立初始堆
    {
        temp=R[1];
        R[1]=R[i];
        R[i]=temp;
        Shift(R,1,i-1);//递归继续调整
    }
}
```

堆排序的最优和最坏情况时间复杂度都为\\(o(nlog_2n)\\)，堆排序适合关键字数很多的情况，例如从10000个关键字中选出前10个最小的，关键字数少用堆排序反而效率低。
