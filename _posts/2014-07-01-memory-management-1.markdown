---
layout: post
title: iOS核心技术之：内存管理之一基本概念
date: 2014-07-01 18:30
tags: 
---

### 内存模型

### isa指针
在Objective-C语言的内部，每一个对象都有一个名为isa的指针，指向该对象的类。每一个类描述了一系列它的实例的特点，包括成员变量的列表，成员函数的列表等。每一个对象都可以接受消息，而对象能够接收的消息列表是保存在它所对应的类中。关于isa指针详细内容推荐巧哥的 [`Objective-C对象模型及应用`](http://blog.devtang.com/blog/2013/10/15/objective-c-object-model/) 介绍的非常到位并且包含实际应用场景。

### 堆内存
&nbsp;&nbsp;&nbsp;&nbsp;**堆**（heap）是指内存中的一块区域，应用中的所有对象都会保存在堆中。当应用向某个类发送alloc消息时，系统会从堆中分配出一块内存，其大小足够存放相应对象的全部实例变量。

```
//
//  Person.h
//  MemoryManagement
//
//  Created by 小鹏 on 14/07/04.
//  Copyright (c) 2014年 ROC. All rights reserved.
//

#import <Foundation/Foundation.h>

@interface Person : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic) int age;
@property (nonatomic, readonly) NSDate *birthday;

@end
```

&nbsp;&nbsp;&nbsp;&nbsp;一个对象永远不会直接保存另一个对象，所有的对象在堆中都是独立存在的。如果需要，一个对象可以保存指向其他对象的引用，即可以将其他对象的地址赋给指针实例变量。

&nbsp;&nbsp;&nbsp;&nbsp;Person实例包含5个实例变量，其中3个为指针变量（isa、name、birthday），另一个为int变量（age）。一个int变量会占用4个字节的内存，所以一个Person实例总共会占用16个字节的内存空间，如下图：

<img src='/assets/images/2014/07/Person_In_Memory_Size.png' />

### 栈内存
&nbsp;&nbsp;&nbsp;&nbsp;**栈**（stack）是指内存中的另一块区域，和堆是分开的。为这两类内存区域分别取名堆和栈，是为了能够形象地描述这两个概念。堆，包含了大量无序的对象，需要通过指针来保存这些对象在堆中的地址。栈，则会按后进先出（LIFO）的规则保存一组**帧**（frame）。  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;当程序执行某个方法（或函数）时，会从栈中分配一块内存空间，这块内存空间称为栈帧。栈帧负责保存程序在方法内声明的变量的值。在方法内声明的变量称为**局部变量**（local variable）。  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;当某个应用启动并运行`main`函数时，它的栈帧会被保存在栈的底部。当`main`调用另一个方法（或函数）时，这个方法（或函数）的栈帧会压入栈的顶部。被调用的方法还可以再调用其他方法，依此类推，最终会在栈中形成一个塔状的栈帧序列 。当被调用的方法（或函数）结束时，程序会将其从栈帧中“弹出”并释放。如果同一个方法再次被调用，则应用会创建一个全新的的栈帧，并将其压入栈的顶部。如图：  <br/>
<img src='/assets/images/2014/07/Person_Method_Stack.png' />

### 对象所有权
指针变量暗含了对其所指向的对象的**所有权**（Ownership）。

* 当某个方法（或函数）有一个指向某个对象的局部变量时，可以称该方法拥有（own）该变量所指向的对象。
* 当某个对象有一个指向其他对象的实例变量时，可以称该对象拥有相应指针所指向的对象。 

以下情况会使对象失去拥有方：

* 当程序修改某个指向特定对象的变量，将其指向另一个对象时。
* 当程序将某个指向对象的变量设置为nil时。
* 当程序释放某个指向特定对象的变量时。


