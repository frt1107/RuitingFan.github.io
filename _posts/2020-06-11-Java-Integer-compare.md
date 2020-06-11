---
layout: post
title:  "Java Integer的==和equals()的区别"
date:   2020-06-11 18:10:01 +0800
categories: Java基础
tag: Java

---


* content
{:toc}




# Java Integer的==和equals()的区别

## 问题发现：

来自于在刷剑指offer的一道题的过程，如图所示。

我在判断栈顶元素相等的时候，通过`==`判断，而不是`equals()`，导致解答错误。

![image-20200611175459766](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20200611175459766.png)

**这是因为我的一个误区：**

以前对`==`和`equals()`区别的理解不到位，导致今天在操作`Stack<Integer>`的过程中出问题。

## Integer的==和equal()区别

**`int`和`Integer`的区别：**

在于Java1.5版本及以上的自动装箱、拆箱机制。由于**autoboxing**机制的存在，使得`Integer`和`int`在很多情况下可以自动转换。

但是：`Integer`是引用类型的变量，`int`是基本类型的变量。这导致在判断`Integer`和`int`类型数据相等时，存在不同。

------

**引用类型和基本类型判断相等的区别：**

对基本类型，`==`比较的是其值而不是地址。

对引用类型，`==`和`equals()`都比较的是地址，除非该类重写了`equals()`方法，即`String类`和`Integer类`。

------

因此，一般意义上来说，`Integer`类型变量比较值相等，需要用`equals()`方法。

然而，特殊情况：由于JVM在常量池中缓存了**-128到127之间**的`Integer`类型整数，因此**-128到127之间**的`Integer`类型整数的地址均相同，因此通过`==`比较这些整数即相当于比较其值。

------

**总体来说，保险起见，对于`Integer`类型整数，比较其值，通过`equals()`方法比较总能得到正确的结果。**