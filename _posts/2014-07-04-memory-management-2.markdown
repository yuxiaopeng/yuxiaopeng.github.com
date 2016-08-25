---
layout: post
title: iOS核心技术之：内存管理之二手动内存管理
date: 2014-07-04 18:01
tags: 
---
### 1.关于内存管理
应用程序内存管理是：“在程序运行时开辟内存空间、使用内存空间，并在程序完成时释放内存空间的过程”。写得好的程序，会尽可能少占用内存。在Objectiv-C中，内存管理被看做是在很多数据和代码中分配受限内存资源所有权（Ownership）的一种方式。虽然内存管理通常被认为是针对单个对象级别进行的，但实际上我们的任务是管理“对象图”(Object Graph)，你需要确保除了你实际需要的对象之外，内存中没有其它的对象。

<img src='/assets/images/2014/07/memory_management_2x.png' width='600' />


* Objective-C提供了两种内存管理方式：  <br/>
1）Manual Retain-Release（**MRR**），通过跟踪程序所拥有的对象来“显式”地管理内存。这种方式采用了一种称为“引用计数”的模型来实现，该模型由Foundation框架的NSObject类和运行时环境（Runtime Environment）共同提供。  <br/>
2）Automatic Reference Counting（**ARC**），自动引用计数，这种方式采用和MRR相同的引用计数系统，但是在编译时（Compile-time）编译器自动插入适当的内存管理的方法。在新的项目中，强烈建议开发者使用ARC方式管理内存。这样开发者不需要理解内存管理的底层实现。不过，理解底层实现对开发高性能的应用程序很有帮助同时对于开发者的自身技术提高也很有帮助。
* 防止内存泄漏或过早释放的最佳实践：  <br/>
引起内存管理的问题主要分为两类：  <br/>
1）**过早释放**：如果某个对象有至少一个拥有方，就必须保留不能释放。如果释放了某个对象，但是其他对象或方法仍然有指向该对象的指针（准确地说，是指向该对象被释放前的地址），那么应用就有可能在运行时出错。当程序向一个已经不存在的对象发送消息时，就会发生崩溃。这类错误称为过早释放。  <br/>
2）**内存泄漏**：如果某个对象没有拥有方，就应该将其释放掉。没有拥有方的对象是孤立的，程序无法向其发送消息。保留这样的对象只会浪费宝贵的内存空间，导致内存泄漏（memory leak）问题。

&nbsp;&nbsp;&nbsp;&nbsp; 从引用计数的角度考虑内存管理，常常会适得其反，因为开发者会过度关注内存管理的实施细则方面，而忽略了实际目标。所以，作为开发者应该从对象所有权（Ownership）和对象图（Object Graph）的角度进行内存管理。

### 2.内存管理策略

通过引用计数来进行内存管理的基本模型是由NSObject协议（Protocol）中的方法以及标准的方法命名规范来实现的。NSObject还定义了一个dealloc方法，该方法会在对象需要被释放的时候自动调用。

* 引用计数系统机制  <br/> 
&nbsp;&nbsp;&nbsp;&nbsp; 可以用开关房间的灯为例来说明引用计数的机制，此处引自《Pro Multithreading and Memory Management for iOS and OS X with ARC， Grand Central Dispatch， and Blocks》一书。  <br/> 
&nbsp;&nbsp;&nbsp;&nbsp; 当上班进入办公室时需要照明，此时需要开灯；而当下班离开办公室时不需要照明，此时需要关灯，如图1-1所示：  <br/>
<img src='/assets/images/2014/07/reference_counting_1.png' width='600px'/>  <br/>
&nbsp;&nbsp;&nbsp;&nbsp; 假设办公室里的照明设备只有一个。上班进入办公室的人需要照明，所以要把灯打开。而对于下班离开办公室的人来说，已经不需要照明了，所以要把灯关掉。若是很多人上下班，每个人都开灯或是关灯，那么办公室的情况又将如何呢？最早下班离开的人如果关了灯，那就会像图 1-2 那样，办公室里还没走的所有人都将处于一片黑暗之中。  <br/>
<img src='/assets/images/2014/07/reference_counting_2.png' width='600'/>  <br/>
解决这一问题的办法是使办公室在还有至少1人的情况下保持开灯状态，而在无人时保持关灯状态。  <br/>
(1)最早进入办公室的人开灯。  <br/>
(2)之后进入办公室的人，需要照明。  <br/>
(3)下班离开办公室的人，不需要照明。  <br/>
(4)最后离开办公室的人关灯(此时已无人需要照明)。  <br/>
为判断是否还有人在办公室里，这里导入计数功能来计算“需要照明的人数”。下面让我们
来看看这一功能是如何运作的吧。  <br/>
(1)第一个人进入办公室，“需要照明的人数”加 1。计数值从 0 变成了 1，因此要开灯。  <br/> 
(2)之后每当有人进入办公室，“需要照明的人数”就加 1。如计数值从 1 变成 2。   <br/>
(3)每当有人下班离开办公室，“需要照明的人数”就减 1。如计数值从 2 变成 1。   <br/>
(4)最后一个人下班离开办公室时，“需要照明的人数”减 1。计数值从 1 变成了 0，因此要关灯。  <br/>
这样就能在不需要照明的时候保持关灯状态。办公室中仅有的照明设备得到了很好的管理，如图1-3所示。  <br/>
<img src="/assets/images/2014/07/reference_counting_3.png" width="600px" />  <br/>
&nbsp;&nbsp;&nbsp;&nbsp; 在Objective-C中，“对象”相当于办公室的照明设备。在现实世界中办公室的照明设备只有一个，但在Objective-C的世界里，虽然计算机资源有限，但一台计算机可以同时处理好几个对象。  <br/>
&nbsp;&nbsp;&nbsp;&nbsp; 此外，“对象的使用环境”相当于上班进入办公室的人。虽然这里的“环境”有时也指在运 行中的程序代码、变量、变量作用域、对象等，但在概念上就是使用对象的环境。上班进入办公 室的人对办公室照明设备发出的动作，与Objective-C中的对应关系则如表1-1所示。  <br/> <img src="/assets/images/2014/07/reference_counting_table_1.png" width="600px" />  <br/> &nbsp;&nbsp;&nbsp;&nbsp; 使用计数功能计算需要照明的人数，使办公室的照明得到了很好的管理。同样，使用引用计 数功能，对象也就能够得到很好的管理，这就是 Objective-C 的内存管理。如图1-4所示。  <br/>
<img src='/assets/images/2014/07/reference_counting_4.png' width='600'/>  <br/>

* 基本内存管理策略  <br/>
内存管理的模型，是基于对象的所有权。只要一个对象有所有者，它就会在内存中一直存在；如果这个对象没有了所有者，那么系统就会自动将其销毁。为了更清晰地描述何时拥有一个对象、何时释放一个对象的所有权，Cocoa遵循下面的策略：  <br/>
1）**自己生成的对象，自己所持有。**  <br/>
如果使用下面名称作为开头的方法来创建对象，那么你将拥有这个对象：`alloc`、`new`、`copy`、 `mutableCopy`。比如，alloc方法、newObject方法、或者mutableCopy等方法。  <br/>
2）**非自己生成的对象，自己也能持有。**  <br/>
如果你在一个方法体中，得到了一个对象，那么这个对象在本方法内部是一直都有效的。而且你还可以在本方法中将这个对象作为返回值返回给方法的调用者。在下面两种状况下，你需要用retain获得一个对象的所有权:在实现存取方法(getter and setter)或`init`方法时，希望将一个自己所持有的对象作为成员变量（property）来存储时；防止对象被其他操作释放掉，从而变为无效的对象。  <br/>
3）**自己持有的对象不再需要时释放。**  <br/>
通过向对象发送`release`消息或者`autorelease`消息来放弃一个对象的所有权。用Cocoa的术语说，所谓放弃所有权，就是`release`一个对象的引用。  <br/>
4）**自己正在使用的对象，不要释放。**  <br/>
5）**非自己持有的对象无法释放。**


例:

```
{
    Person *aPerson = [[Person alloc] init];
    // ...
    NSString *name = aPerson.fullName;
    // ...
    [aPerson release];
}
```
&nbsp;&nbsp;&nbsp;&nbsp; Person对象通过`alloc`方法创建，因此当不再使用该对象时，需要通过向这个Person对象发送`release`消息来释放。Person的name因为不是使用获取对象所有权的方法来得到的，所以也不必`release`。请注意，这里用的是`release`而非`autorelease`方法。 
### 3.内存管理实践

* 使用访问器方法使得内存管理变得简单<br/>
如果一个类有一个对象类型的成员变量，在使用这个对象的时候，必须保证被设为值的这个对象没有被`dealloc`。因此，必须在`set`值的时候获得该对象的所有权。并放弃正在使用对象的所有权。<br/>有时，这些听起来很老套和繁琐，但如果统一使用访问器方法来实现，内存管理有问题的几率将大大减少。如果在代码中对实例变量到处使用`retain`和`release`，几乎可以肯定的说：这是错误的。
来看一个需要设置值的Counter对象：

```
@interface Counter : NSObject

@property (nonatomic， retain) NSNumber *count;

@end;
```
&nbsp;&nbsp;&nbsp;&nbsp;这里的属性count定义了两个存取器方法。通常而言，这些方法由编译器来合成，但如果你知道这些方法的实现，对理解问题还是会有帮助的。  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;在`get`方法中，只是返回实例变量，所以不必`retain`和`release`:

```
- (NSNumber *)count {
    return _count;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;在`set`方法中，必须需要考虑的是:新的值可能随时被`dealloc`。因此必须通过发送`retain`消息来取得对新值的所有权，进而保证不会`dealloc`。你还必须对旧值发送`release`消息。（在Objective-c中，对一个 nil发送消息是没问题。因此就算_count还没有值，也不会出错。）你必须在调用`[newCount retain]`之后再(对旧值)发送`release`，因为如果出现意外将会被`dealloc`。

```
- (void)setCount:(NSNumber *)newCount {
    [newCount retain];
    [_count release];
    // Make the new assignment.
    _count = newCount;
}
```

* 使用访问器方法来设置属性值  <br/>
假如要实现一个计数器的reset方法。有几个可行的做法，第一种做法就是调用`alloc`来新建一个NSNumber 类型的变量，然后再对应发送一个`release`消息。

```
- (void)reset {
    NSNumber *zero = [[NSNumber alloc] initWithInteger:0];
    [self setCount:zero];
    [zero release];
}
```
&nbsp;&nbsp;&nbsp;&nbsp;第二种做法是使用convenience constructor来创建一个新的NSNumber对象。这种情况下不需要`release`和`retain`。

```
- (void)reset {
    NSNumber *zero = [NSNumber numberWithInteger:0];
    [self setCount:zero];
}
```
&nbsp;&nbsp;&nbsp;&nbsp;请注意，上面这些都用到了`set`方法。
下面的做法，对于简单的情况而言，肯定是没问题的。但是，因为它的实现绕开了`set`方法， 那么在特定情况下(比如当你忘记了`retain`或者`release`;再比如，当实例变量的内存管理发生了变化。)会导致内存泄露:

```
- (void)reset {
    NSNumber *zero = [[NSNumber alloc] initWithInteger:0];
    [_count release];
    _count = zero;
}
```
还需要注意的是，如果你使用了Key-Value Observing，这样来修改变量是不适合于KVO编程的。

* 在初始化方法和dealloc方法中不要使用访问器方法  <br/>
初始化和`dealloc`方法中是唯一不应该使用存取方法的地方，使用一个number对象来将counter初始化为0，这样来实现`init`方法即可：

```
- init {
    self = [super init];
    if (self) {
        _count = [[NSNumber alloc] initWithInteger:0];
    }
    return self;
}
```
如果不是counter这个实例变量初始化为固定值，你需要实现一个`initWithCount:`方法：

```
- initWithCount:(NSNumber *)startingCount {
    self = [super init];
    if (self) {
        _count = [startingCount copy];
    }
    return self;
}

```
由于Counter类有一个对象类型的实例变量，所以必须实现`dealloc`方法。这个方法中必须释放实例变量的所有权，并且最后调用基类的`dealloc`方法：

```
- (void)dealloc {
    [_count release];
    [super dealloc];
}
```
* 使用弱引用来避免retain循环<br/>
`retain`操作创建了一个对这个对象的强引用。只有这个对象的所有强引用都被释放后，这个对象才能够被销毁。这就有可能形成这样一个问题，就是我们常说的`retain`循环：当两个对象互相包含一个执行对方的强引用（可能不是直接引用对方，可能是通过其他对象形成的一条强引用链）。<br/>
图一举例说明了一个潜在的`retain`循环。Document对象中包含每一页的Page对象的引用。而每个Page对象又包含一个属性，这个属性用来跟踪自己是属于哪个Document对象。如果Document对象拥有Page对象的强引用，而Page对象也拥有对Document对象的强引用，这两个对象就再也不能被释放了。Document对象的引用计数只有当Page对象是否后才能清零，而Page对象也只有在Document对象被销毁后才能被释放。<br/>
图1：  <br/>
<img src='/assets/images/2014/07/retaincycles_2x.png' width='300' /> <br/> 
解决这个问题的途径是使用弱引用。弱引用是指一种非拥有的对象，源对象只拥有指向目的对象的引用，但并没有对这个引用执行retain操作。<br/>
为了保证对象图的完整，必须要有强引用（如果整个关系图中只有弱引用，page对象或paragraph对象就没有所有者，那么就会自动被销毁）。 Cocoa建立了一个规约：父对象拥有对子对象的强引用，子对象拥有对父对象的 弱引用。<br/>
因此，在图一中，Document对象拥有对Page对象的强引用（需要执行`retain`操作），而Page对象拥有对Document对象的弱引用（不要执行`retain`操作）。<br/>
在Cocoa中有很多弱引用的例子，包括`table data sources`，`outline view items`，`notification observers`，还有其他如`targets`与`delegates`。<br/>
在向一个弱引用的对象发送消息时，必须小心。如果在这个对象被销毁后向其发送消息，程序会崩溃。你必须明确知道什么情况下这个对象是有效的。在大多数情况下，弱引用对象为了防止循环引用，我们只关心引用它的对象，当它将要销毁的时候，需要通知引用它的对象，这个对象已经被销毁了。举例来说，当你在`notification center`注册了一个对象，消息中心保存了对这个对象的弱引用，当有消息时来时，会通知这个对象。当这个对象被销毁后，你也必须在消息中心将其注销，以防止消息中心将消息发送到已经已经不存在的对象上。同样，当一个代理（`delegate`）对象被销毁时，你必须在引用该代理的对象中使用`setDelegate:`消息，将其设置为`nil`。一般来说，这些消息都应在对象的`dealloc`方法中被调用。

* 避免销毁正在使用的对象<br/>
Cocoa的内存管理原则规定，一般返回的对象需要在其调用函数的作用域范围内一直有效。并且，当需要将这个返回的对象作为当前方法的返回值返回时，也不用担心它会因为离开作用域被释放掉。对你的程序来说，`getter`方法返回实例变量的缓存值或计算值是都没有关系。真正重要的是，当你需要使用这个对象时，对象仍然是有效的。<br/>
偶尔有些例外，主要是下面两种情况：<br>
1）当一个对象被基本集合类删除时：

```
heisenObject = [array objectAtIndex:n];
[array removeObjectAtIndex:n];
// heisenObject could now be invalid.
```
&nbsp;&nbsp;&nbsp;&nbsp;当对象从基本集合类移除，集合类发送一个release(而不是autorelease)消息。如果集合类是对象的唯一拥有者，被移除的对象(例子中heisenObject)就被立即销毁。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;2）当父对象被销毁时：

```
id parent = <#create a parent object#>;
// ...
heisenObject = [parent child] ;
[parent release]; // Or， for example: self.parent = nil;
// heisenObject could now be invalid.
```
&nbsp;&nbsp;&nbsp;&nbsp;某些情况，开发者从其他对象获得一个对象，直接或见解释放父对象。release父对象导致被销毁，父对象又是子对象的唯一拥有者，子对象将同时被销毁。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;防止这些情况，当开发者收到heisenObject时retain，使用完release掉，比如：

```
heisenObject = [[array objectAtIndex:n] retain];
[array removeObjectAtIndex:n];
// Use heisenObject...
[heisenObject release];
```

* 不要对稀缺资源进行dealloc <br/>
不要在`dealloc`函数中管理`file descriptor`、`network connections`、`buffer`和`caches`这些资源。通常，开发者不应设计带有dealloc这样的类。由于程序的bug或程序崩溃等，dealloc可能延迟调用。<br/>
如果你的类中有控制稀有资源的实例变量，你应当将你的类设计成：当你不再需要这些资源，通知该实例变量清理该资源，并发送`release`方法，然后系统自动调用`dealloc`方法，来清理资源，这样即便系统没有适时调用`dealloc`方法，你也不会因为没有清理资源而造成问题。<br/>
如果你想依赖`dealloc`方法来进行系统资源的内存管理，可能会出现如下问题：<br/>
1）对象图的顺序依赖性可能遭到破坏<br/>
对象图的销毁在内在机制上是无序的，如果一个对象被意外`autorelease`而不是`release`掉了，对象销毁顺序就会发生变化，从而导致无法预期的错误。<br>
2）稀有资源未回收<br/>
内存泄漏造成的Bug应该是固定的，但它们通常不会立即致命。如果稀有资源没有在应该释放的时候释放，就可能导致严重的问题。比如你的应用程序耗尽了系统文件描述符，用户就可能再也无法保存任何数据了。<br/>
3）在错误的线程上执行清理逻辑 <br/>
如果一个对象在非预期的时间被自动释放了，它可能在任意一个线程的自动释放资源池中被销毁。这种情况很容易就会造成严重的错误。
* 集合拥有它所包含的对象<br/>
当你添加一个对象到集合(如，`array`，`dictionary`和`set`)，集合获得对象的所有权。对象被移除的时候或者集合本身`release`的时候，集合会释放对象的所有权。如果要建一个存放数字的数组，可以按下面方法来做：

```
NSMutableArray *array = <#Get a mutable array#>;
NSUInteger i;
// ...
for (i = 0; i < 10; i++) {
    NSNumber *convenienceNumber = [NSNumber numberWithInteger:i];
    [array addObject:convenienceNumber];
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这种情况，开发者没有调用`alloc`，所以无需掉用`release`。也没有必要`retain`新的`convenienceNumber`

```
NSMutableArray *array = <#Get a mutable array#>;
NSUInteger i;
// ...
for (i = 0; i < 10; i++) {
    NSNumber *allocedNumber = [[NSNumber alloc] initWithInteger:i];
    [array addObject:allocedNumber];
    [allocedNumber release];
}
```
&nbsp;&nbsp;&nbsp;&nbsp;在这个例子中，我们在`for`循环内部向`allocedNumber`发送了与`alloc`相对应的`release`消息。因为 `Array`的`addObject:`方法实际上对这个对象做了`retain`处理，那么这个对象(`allocedNumber`)不 会因此而被`dealloc`。
为了理解这条规则，把你自己想象成集合类的设计者。你必须要保证，在你设计 的集合类中的所有对象都不会平白无故的被释放，因此你必须在添加它们时执行`retain`操作。当你要删除这个对象时，为了平衡`retain`操作，需要对其执行`release`，并且在你的集合类的`dealloc`方法中，应当释放你还拥有的成员对象。

* 对象所有权策略是基于引用计数实现的 <br/>
所有权策略是通过引用计数来实现的，通常称之为“retain count”。每个对象都有一个 retain count。<br/>当新建一个对象时，它的retain计数为1;<br/>发送`retain`消息给一个对象时，它的retain计数加1;<br/>发送`release`消息给一个对象时，它的retain计数减1;<br/>发送`autorelease`消息，它的retain计数将在未来某个时候减1;<br/>如果retain计数是0，就会被`dealloc`。<br/><font color='red'> 重要: </font> 不要显式调用对象的`retainCount`，这个数值有时候会造成对你的误导：实际上你不知道有些系统框架的对象会对你关注的那个对象进行`retain`。在调试内存问题的时候，你只需要遵守所有权规则就行了。

### 4.工具
* Xcode有一个方便的自带工具Clang Static Analyzer
打开项目，选择菜单 `Product` > `Analyze` 或者使用快捷键 **⇧⌘B** ，打开Analyzer工具。其他详细使用方法参考：<br/>[`Automated Static Code Analysis with Xcode 5.1 and Jenkins`](http://blog.manbolo.com/2014/04/15/automated-static-code-analysis-with-xcode-5.1-and-jenkins)<br/>
[`Running the analyzer within Xcode`](http://clang-analyzer.llvm.org/xcode.html)
* 还可以使用Instruments来跟踪引用计数事件并寻找 [`Collecting Data on Your App`](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/GatheringDatafortheFirstTime/GatheringDatafortheFirstTime.html)


参考：
[`Advanced Memory Management Programming Guide`](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html)

