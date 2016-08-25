---
layout: post
title: 设计模式之观察者模式Observer
date: 2014-07-07 19:30
tags:
---

### 1.何为观察者模式（Observer）

**观察者模式**（又被称为发布/订阅模式Publish-Subscribe或依赖模式Dependents）是软件设计模式的一种。即定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。在此种模式中，一个目标对象管理所有相依赖于它的观察者对象，并且在它本身的状态改变时主动发出通知。这通常通过调用各观察者所提供的方法来实现。此种模式通常被用来实现分布式事件处理系统。此模式也是MVC模式的关键组成部分。在许多编程中都用到了观察者模式，包括几乎所有的GUI程序都用到了此模式，例如：Mac、iOS、Win操作系统的图形用户界面。

观察者模式可能引起内存泄露（`memory leaks`），著名的流失监听者泄露问题([`Lapsed listener problem`](http://en.wikipedia.org/wiki/Lapsed_listener_problem))此总是经常发生在具备垃圾回收特性的编程中，因为此模式需要基于明确的注册和注销主题对象来实现，并且主题对象持有观察者对象的强引用。而观察者对象也可以称为监听者，主题对象持有所有监听者对象的强引用，导致监听者对象内存无法被释放，造成内存泄露。此问题可以通过将主题对象所持有的强引用改为弱引用的方式来防止内存泄露。

减少对象之间的耦合有利于系统的复用，但是同时设计师需要使这些低耦合度的对象之间能够维持行动的协调一致，保证高度的协作。观察者模式是满足这一要求的各种设计方案中最重要的一种。



### 2.如何实现观察者（Objective-C）
实现思路：主题（Subject）持有所有的观察者列表，而观察者（Observer）监听主题对象。主题对象状态发生变化时，会通知所有持有的观察者对象，使它们能够做出响应。结构如下图：

<img src='/assets/images/2014/07/Observer_Pattern.png' />

创建Observer观察者协议：

```
//
//  Observer.h
//  Observer
//
//  Created by 小鹏 on 14/7/7.
//  Copyright (c) 2014年 ROC. All rights reserved.
//
#import <Foundation/Foundation.h>

@protocol Observer <NSObject>

-(void) notify:(NSString *)message;

@end
```
创建Subject主题协议及实现：

```
//
//  Subject.h
//  Observer
//
//  Created by 小鹏 on 14/7/7.
//  Copyright (c) 2014年 ROC. All rights reserved.
//

#import <Foundation/Foundation.h>
#import "Observer.h"

@protocol Subject <NSObject>

-(void) addObserver:(id) observer;
-(void) removeObserver:(id) observer;
-(void) notifyObservers:(NSString *) message;

@end

@interface Subject : NSObject <Subject>

@property (nonatomic, strong) NSMutableSet *observerCollection;

@end

//
//  Subject.m
//  Observer
//
//  Created by 小鹏 on 14/7/7.
//  Copyright (c) 2014年 ROC. All rights reserved.
//

#import "Subject.h"

@implementation Subject

-(NSMutableSet *) observerCollection
{
    if (_observerCollection == nil)
        _observerCollection = [[NSMutableSet alloc] init];
    
    return _observerCollection;
}

-(void) addObserver:(id)observer
{
    [self.observerCollection addObject:observer];
}

-(void) removeObserver:(id)observer
{
    [self.observerCollection removeObject:observer];
}

-(void) notifyObservers:(NSString *)message
{
    for (id<Observer> observer in self.observerCollection) {
        [observer notify:message];
    }
}

@end
```
创建Observer实现：

```
//
//  ConcreteObserverA.m
//  Observer
//
//  Created by 小鹏 on 14/7/7.
//  Copyright (c) 2014年 ROC. All rights reserved.
//

#import "ConcreteObserverA.h"

@implementation ConcreteObserverA

-(void) notify:(NSString *)message
{
    NSLog(@"Class is %@,message is : %@", NSStringFromClass([self class]), message);
}

@end

//
//  ConcreteObserverB.m
//  Observer
//
//  Created by 小鹏 on 14/7/7.
//  Copyright (c) 2014年 ROC. All rights reserved.
//

#import "ConcreteObserverB.h"

@implementation ConcreteObserverB

-(void) notify:(NSString *)message
{
    NSLog(@"Class is %@ ,message is : %@", NSStringFromClass([self class]), message);
}

@end
```
main方法中测试代码：

```
//
//  main.m
//  Observer
//
//  Created by 小鹏 on 14/7/7.
//  Copyright (c) 2014年 ROC. All rights reserved.
//

#import <Foundation/Foundation.h>
#import "Subject.h"
#import "ConcreteObserverA.h"
#import "ConcreteObserverB.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        Subject *subject = [[Subject alloc] init];
        ConcreteObserverA *observerA = [[ConcreteObserverA alloc] init];
        ConcreteObserverB *observerB = [[ConcreteObserverB alloc] init];
        
        [subject addObserver:observerA];
        [subject addObserver:observerB];
        
        [subject notifyObservers:@"notify event!"];
    }
    return 0;
}
```
打印结果：

```
2014-07-07 11:13:30.869 Observer[18209:1539966] Class is ConcreteObserverA,message is : notify event!
2014-07-07 11:13:30.871 Observer[18209:1539966] Class is ConcreteObserverB,message is : notify event!
```

### 3.应用（Cocoa中的Observer）
iOS作为主要的GUI系统，大量应用了观察者模式，其中的事件响应机制就是观察者的深度应用和实现。Cocoa框架中也不乏观察者模式应用的例子。通知（NSNotification & NSNotificationCenter）和 KVO（Key-Value Observing）是对观察者模式的实践。

* NSNotification & NSNotificationCenter  <br/>
Cocoa通知机制，遵循广播模型（broadcast model），通过注册成为通知中心的观察者对象，当事件发生时观察者会收到消息并做出响应。这种机制使主题与观察者以一种松耦合的方式通信。主题与观察者在通信时无需了解对方的实现。通过发送消息可以达到代码的彻底解耦。  <br/>
 <img src='/assets/images/2014/07/notificationcenter.jpg' />  <br/>
任何对象都可以观察通知，但要做到这一点，该对象必须注册，以接收通知。在注册时，必须指定一个选择器，以确定由通知传送所调用的方法；方法签名必须只有一个参数：通知对象。注册后，观察者也可以指定发布对象。通知机制中涉及两个重要的类：  <br/>
1）NSNotification: 这是消息的载体，通过它可以把消息内容传递给观察者。  <br/>
2）NSNotificationCenter: 实现NSNotificationCenter的原理是一个观察者模式，通过调用静态方法defaultCenter就可以获取这个通知中心的对象，而NSNotificationCenter同时是一个单例模式，这个通知中心的对象会一直存在于一个应用的整个生命周期中。一个NSNotificationCenter可以有许多的通知消息NSNotification，对于每一个NSNotification可以有很多的观察者Observer来接收通知。  <br/>
注册成为观察者：

```
[[NSNotificationCenter defaultCenter] addObserver:self 
										 selector:@selector(downloadImage:) 
										 	 name:@"BLDownloadImageNotification" 
										   object:nil];
```
发送消息：

``` 
[[NSNotificationCenter defaultCenter] postNotificationName:@"BLDownloadImageNotification"
                                                    object:self                                                 
                                                  userInfo:@{@"imageView":coverImage, @"coverUrl":albumCover}];
```
dealloc中移除观察者：

```
- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

* KVO  <br/>
Cocoa中提供一种称为键值观察的机制，对象可以通过KVO得到其他对象特定属性的变更通知。这种机制在MVC模式的场景中尤为重要。因为它让**View对象可以经由Controller对象观察Model对象的变更**。  <br/>
这一机制基于NSKeyValueObserving非正式协议，Cocoa通过这个协议为所有遵循协议的对象提供了一种自动化的属性观察能力。要实现自动观察，参与KVO的对象要符合KVC的要求，并且需要符合KVC的存取方法。KVC基于有关非正式协议，通过存取对象属性实现自动观察。  <br/>
Notifications和KVO者是Cocoa对观察者模式的实践。尽管两者依赖同样的发布-订阅关系，但是它们是为不同的解决方案而设计的。两者的主要差别如下表：

	 Notifications                  |  KVO                        
	 ------------------------------ | --------------------------- 
	 一个中心对象为所有观察者提供变更通知 | 被观察的对象直接向观察者发送通知  
	 主要从广义上关注程序事件           | 绑定于特定对象属性的值          
	 通知有开销，即使你不使用它们。每次发布一个通知时，它必须对每一个观察系统中进行检查，即使没有人观察该对象（即使没有人正在观察的任何东西）| 如果没有实际观察到的任何对象，则零开销 


参考：  <br/>
[`Observer Pattern`](http://en.wikipedia.org/wiki/Observer_pattern)  <br/>
[`流失监听者泄露`](http://jolestar.com/1278227820000/)  <br/>
[`iOS Design Patterns`](http://www.raywenderlich.com/46988/ios-design-patterns)  <br/>
[`Key-Value Observer`](http://nshipster.com/key-value-observing/)  <br/>
[`Patterns in Objective-C: Observer Pattern`](http://www.numbergrinder.com/2008/12/patterns-in-objective-c-observer-pattern/)  <br/>
[`Notification Programming Topics`](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Notifications/Introduction/introNotifications.html) <br/>
[`Key-Value Coding Programming Guide`](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.hbr/>
[`Key-Value Observing Programming Guide`](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)

