---
layout: post
title: 设计模式之适配器模式Adapter
date: 2014-07-03 18:36
tags:
---

### 1.何为适配器模式（Adapter）
将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作（Adapter有助于两个不兼容的接口一起工作）。
简单的说，就是需要的东西就在面前，但却不能使用，而短时间又无法改造它，于是我们就想办法适配它。适配器主要应用于希望复用一些现存的类，但是接口又与复用环境要求不一致的情况，比如在需要对早期代码复用一些功能等应用上很有实际价值。

### 2.如何实现适配器（Objective-C）
Delegate是Objective-C的语言特性，使用它可以定义适配器模式的实例接口。协议本质上是一系列的非关联与类方法声明（等同于Java中的接口Interface）。如果你想让一个对象与其他对象沟通，但对象的接口互不兼容，你可以定义一个协议，然后对象可以通过协议接口将消息发送到其它对象。

<font color='red'> 注：</font> Delegate的设计并不完全匹配于适配器模式的描述。但它实现了该模式的目标是：允许类用不兼容的接口一起工作。

实现Adapter模式的思路是：  <br/>
第一种称为类适配器，主要通过继承和协议实现，首先需要有定义了客户端要使用的一套行为的协议，然后要用具体的适配器类实现这个协议。适配器类同时也要继承被适配者，它们之间的关系如图：

![](/assets/images/2014/07/Class_Adapter.jpg)  <br>
第二种称为对象适配器，与类适配器不同，对象适配器不继承被适配者，而是组合了一个对它的引用。实现为对象适配器时，它们的关系如图：

![](/assets/images/2014/07/Object_Adapter.jpg)

```
//  Adapter
//  Target.h

#import <Foundation/Foundation.h>
@interface Target : NSObject

@end

@protocol Target <NSObject>

- (void)request;

@end

#import "Target.h"
@implementation Target

@end



//  Adaptee.h

#import <Foundation/Foundation.h>
@interface Adaptee : NSObject
- (void)specificRequest;
@end

#import "Adaptee.h"
@implementation Adaptee
- (void)specificRequest
{
    NSLog(@"Adaptee specificRequest method invoked!");
}
@end


//  Adapter.h

#import <Foundation/Foundation.h>
#import "Adaptee.h"
#import "Target.h"
@interface Adapter : NSObject <Target>
@property Adaptee *adaptee;
-(void)adapteeRequest;
@end

#import "Adapter.h"
@implementation Adapter
-(void)request
{
    NSLog(@"Target Protocal request method invoked!");
}
-(void)adapteeRequest
{
    _adaptee = [[Adaptee alloc] init];
    [_adaptee specificRequest];
}
@end

```

### 3.应用（Cocoa中的Adapter）
待续...

参考：  <br />
[`Adapter_pattern`](http://en.wikipedia.org/wiki/Adapter_pattern)  <br />
[`Cocoa Design Patterns`](https://developer.apple.com/legacy/library/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaDesignPatterns/CocoaDesignPatterns.html#//apple_ref/doc/uid/TP40002974-CH6-SW5)  <br/>
[`iOS Design Patterns`](https://github.com/linktoming/notes-ios/wiki/iOS-Design-Patterns)  <br/>
[`Adapter Design Pattern`](http://www.codeproject.com/Articles/42915/Adapter-Design-Pattern)  <br/>
[`iOS Design Patterns`](http://www.raywenderlich.com/46988/ios-design-patterns)

