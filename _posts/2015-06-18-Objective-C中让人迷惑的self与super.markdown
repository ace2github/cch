---
layout: post
title:  "Objective-C中让人迷惑的self与super"
date:   2015-06-16 20:56:58
categories: iOS-Dev
---
###一、从代码看起

	@interface Person : NSObject
	+ (void)printSelfClass;
	+ (void)printSuperClass;
	- (void)printSuperClass;
	- (void)printSelfClass;
	@end
	
	@implementation Person
	- (instancetype)init {
	    if (self = [super init]) {
	    }
	    return self;
	}
	
	+ (void)printSelfClass {
	    NSLog(@"+[self class]::%@",[self class]);
	    NSLog(@"+[self superclass]::%@",[self superclass]);
	}
	- (void)printSelfClass {
	    NSLog(@"-[self class]::%@",[self class]);
	    NSLog(@"-[self superclass]::%@",[self superclass]);
	}
	
	+ (void)printSuperClass {
	    NSLog(@"+[super class]::%@",[super class]);
	    NSLog(@"+[super superclass]::%@",[super superclass]);
	}
	- (void)printSuperClass {
	    NSLog(@"-[super class]::%@",[super class]);
	    NSLog(@"-[super superclass]::%@",[super superclass]);
	}
	@end
	
结果如下：
	
	1、+[self class]::Person
	2、+[self superclass]::NSObject
	3、-[self class]::Person
	4、-[self superclass]::NSObject
	
	5、+[super class]::Person
	6、+[super superclass]::NSObject
	7、-[super class]::Person
	8、-[super superclass]::NSObject


试着解析： 

*	关于class与superClass方法，class返回当前调用者的class类型，superClass当前调用者的父类的class类型。
	
	NSObject类的类方法：
	
		+ (Class)superclass;	Returns the class object for the receiver’s superclass.
		+ (Class)class	//The class object.
		
	NSObject协议的实例方法：
	
		- (Class)superclass;		//The class object for the receiver’s superclass.
		- (Class)class		//The class object for the receiver’s class.

*	对比1和3的输出，类方法中的self和实例方法中self，输出的结果相同。但代表的意义不同，实例方法中，self代表“对象”，类方法中self代表“类”。

*	奇怪的3和5的输出一样，`[self class]`和`[super class]`是一样的。

###二、为什么self和super是一样的？

请明确的知识点：

*	self调用自己方法，super调用父类方法。
*	self是类，**super是预编译指令**

**super是预编译指令**,那么编译器是如何调用到父类的方法呢？一切从赋予`Objective-C`面向对象能力的`Runtime`说起。
	
###三、self和super的实现原理  
我们知道，`Objective-C`方法的调用的过程，实质上是消息发送过程。

1、调用实例方法，编译器会替换为`objc_msgSend(id self，SEL _cmd, ...)`方法。  
如`[self class];`，当一个对象person调用对应的实例方法class的时候，其中self用对象实例person代替，_cmd是对应的方法名class代替，后面加参数。  

2、调用类方法，`[super class];`编译器会替换为`id objc_msgSendSuper(struct objc_super *super, SEL op, ...)`方法。其中，第一个参数是`objc_super`结构体,结构体的定义如下：  
	
	struct objc_super {
    	id receiver;
	    Class superClass;
	};  

**实际的调用过程**：  
1、构建`objc_super`结构体，**其中`receiver`初始化为`self`**，第二个成员`superClass`指向调用者类的父类。  
2、将方法名`class`传递给`op`，然后传参。  
3、函数的内部，**从`objc_super`指向的`superClass`查找`class`方法；**  
4、找到之后，使用`objc_super->receiver`去调用这个selector.  
5、由于`objc_super->receiver`和`self`是相同的，**子类没有覆写父类class方法**，因此等价于self和super是一样。  


###四、为什么[super class]和[self class]不一样？
如果假设，在所有的继承关系链中，都实现自身的class方法，那么[self class]和[super class]是不一样。

	@interface Person : NSObject
	@end
	
	@interface USPerson : Person
	@end
	
	@implementation Person
	- (Class)class {
	    return NSClassFromString(@"Person");
	}
	...
	@end
	
	@implementation USPerson
	- (id) init {
	    if (self = [super init]) {
	        NSLog(@"-[self class]::%@",[self class]);
	        NSLog(@"-[super class]::%@",[super class]);
	    }
	    
	    return self;
	}
	
	- (Class)class {
	    return NSClassFromString(@"USPerson");
	}
	...
	@end
	
	结果如下：
	-[self class]::USPerson
	-[super class]::Person  

###五、总结：
**当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用 super 时，则从父类的方法列表中开始找，然后调用父类的这个方法。**  


###六、init的标准写法  

	- (instancetype)init {
	    if (self = [super init]) {
	    }
	    return self;
	}

更具上面的规则，`self = [super init]`会去调用父类的`init`方法，为父类分配内存空间，这样一直往下知道NSObject，这就形成`构造链`,继承父类的子类能够正确的初始化。

###Reference To：  
---  
*	[NSObject Class Reference](https://developer.apple.com/library/prerelease/watchos/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/#//apple_ref/occ/clm/NSObject/class)
*	[关于 self 和 super 在oc 中 的疑惑 与 分析](http://www.cnblogs.com/tangbinblog/p/4034890.html)
*	[self和super区别](http://www.cnblogs.com/wustlj/archive/2011/11/07/2239635.html)  
