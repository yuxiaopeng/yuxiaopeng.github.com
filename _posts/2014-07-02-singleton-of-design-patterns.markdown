---
layout: post
title: 设计模式之单例模式Singleton
date: 2014-07-02 17:28
tags:
---
![](/assets/images/2014/07/singleton.jpg)

### 1.何为单例模式（Singleton）
保证一个类仅有一个实例，并提供一个访问它的全局访问点。
一个全局变量使得一个对象可以被访问，但它不能防止实例化多个对象。一个更好的办法是，让类自身负责保存它的唯一实例。这个类可以保证没有其他实例可以被创建（通过截取创建新对象的请求），并且它可以提供一个访问该实例的方法。这就是Singleton模式。

### 2.如何实现单例（Objective-C）
实现单例模式的思路是：一个类能返回对象一个引用(永远是同一个)和一个获得该实例的方法（必须是静态方法）；当我们调用这个方法时，如果类持有的引用不为空就返回这个引用，如果类保持的引用为空就创建该类的实例并将实例的引用赋予该类保持的引用；同时我们还将该类的构造函数定义为私有方法，这样其他处的代码就无法通过调用该类的构造函数来实例化该类的对象，只有通过该类提供的静态方法来得到该类的唯一实例。

在Objective-C中要实现一个单例需要以下步骤：

* 在单例类中声明一个单例类的静态实例，并初始化为nil；
* 在单例类中实现一个类工厂方法（方法名类似“sharedInstance” 或者“sharedManager”），当唯一实例为nil时首先创建再返回这个唯一实例，如果唯一实例已经创建则直接返回这个唯一实例；
* 覆盖 `allocWithZone:` 方法，以防止复制生成实例副本，而只生成这个共享的唯一实例；
* 覆盖 `release`， `retain`， `retainCount`， `autorelease` 内存管理方法，其中在retain和autorelease什么都不做只是返回自己，release的时候什么都不做，将retainCount设为NSUInteger的极大值。

```
/* Singleton.h */
#import <Foundation/Foundation>

@interface Singleton : NSObject
+ (Singleton *)instance;
@end

/* Singleton.m */
#import "Singleton.h"
static Singleton *instance = nil;

@implementation Singleton

+ (Singleton *)instance {
    if (!instance) {
        instance = [[super allocWithZone:NULL] init];
    }
    return instance;
}

+ (id)allocWithZone:(NSZone *)zone {
    return [self instance];
}

- (id)copyWithZone:(NSZone *)zone {
    return self;
}

- (id)init {
    if (instance) {
        return instance;
    }
    self = [super init];
    return self;
}

- (id)retain {
    return self;
}

- (oneway void)release {
    // Do nothing
}

- (id)autorelease {
    return self;
}

- (NSUInteger)retainCount {
    return NSUIntegerMax;
}

@end
```

这是一种很标准的Singleton实现，不过这种实现并不是线程安全的。

<font color='red'>注：</font> 单例模式在多线程的应用场合下必须小心使用。如果当唯一实例尚未创建时，有两个线程同时调用创建方法，那么它们同时没有检测到唯一实例的存在，从而同时各自创建了一个实例，这样就有两个实例被构造出来，从而违反了单例模式中实例唯一的原则。 解决这个问题的办法是为指示类是否已经实例化的变量提供一个互斥锁(虽然这样会降低效率)。

加入线程安全的单例实现：

```
/* Singleton.h */
#import <Foundation/Foundation>

@interface Singleton : NSObject
+ (Singleton *)instance;
@end

/* Singleton.m */
#import "Singleton.h"
static Singleton *instance = nil;

@implementation Singleton

+ (Singleton *)instance {
    @synchronized(self) {
    	if (!instance) {
        	instance = [[super allocWithZone:NULL] init];
    	}
    }
    return instance;
}

+ (id)allocWithZone:(NSZone *)zone {
	 @synchronized(self) {  
     	return [self instance];
     }
}

- (id)copyWithZone:(NSZone *)zone {
    return self;
}

- (id)init {
    if (instance) {
        return instance;
    }
    self = [super init];
    return self;
}

- (id)retain {
    return self;
}

- (oneway void)release {
    // Do nothing
}

- (id)autorelease {
    return self;
}

- (NSUInteger)retainCount {
    return NSUIntegerMax;
}

@end
```
这是一种线程安全的单例实现，但以上两种实现方式都是手动管理内存的方式，iOS5之后ARC技术被引入并普遍被使用，下面给出结合GCD实现ARC单例：

```
#import <Foundation/Foundation.h>

@interface Singleton : NSObject

+(instancetype) sharedInstance;

@end

#import "MySingleton.h"

@implementation Singleton

+(instancetype) sharedInstance {
    static dispatch_once_t pred;
    static id shared = nil;
    dispatch_once(&pred, ^{
        shared = [[super alloc] initUniqueInstance];
    });
    return shared;
}

-(instancetype) initUniqueInstance {
    return [super init];
}

@end
```
使用block `dispatch_once`，正如其名称所暗示的，在应用程序生命周期内，这方法只执行一次，这将确保仅创建一个实例，这也是快速和线程安全的：不需要检查`synchronized`或`nil`。以上就是ARC下结合GCD的一个单例的代码。

再进一步，还可以写一个方便调用的宏，如下：

```
+ (id)sharedInstance
{
  DEFINE_SHARED_INSTANCE_USING_BLOCK(^{
    return [[self alloc] init];
  });
}
```

### 3.应用（Cocoa中的Singleton）
在Cocoa框架中，有许多单例模式的应用，例如：NSFileManager，NSWorkspace。而UIKit框架中，同样不乏单例模式的应用，例如：UIApplication和UIAccelerometer。按照惯例，这些单例类都是通过形如sharedClassType的工厂方法返回一个共享实例。Cocoa框架的例子是`sharedFileManager`，`sharedColorPanel`和`sharedWorkspace`。我们经常使用的`[NSUserDefaults standardUserDefaults]`，`[UIScreen mainScreen]` 也是单例的两个应用。


参考：  <br/>
[`A note on Objective-C singletons`](http://lukeredpath.co.uk/blog/2011/07/01/a-note-on-objective-c-singletons/)  <br/>
[`Singleton of Cocoa Core Competencies`](https://developer.apple.com/library/mac/documentation/General/Conceptual/DevPedia-CocoaCore/Singleton.html)  <br/>
[`Creating a Singleton Instance`](https://developer.apple.com/legacy/library/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaObjects/CocoaObjects.html#//apple_ref/doc/uid/TP40002974-CH4-SW32)  <br/>
[`Singletons in Objective-C`](http://www.galloway.me.uk/tutorials/singleton-classes/)  <br/>
[`Singleton with ARC`](http://stackoverflow.com/questions/7997594/singleton-with-arc)  <br/>
[`单例模式`](http://zh.wikipedia.org/zh/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F)

