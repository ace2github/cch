---
layout: post
title:  "关于id与instancetype"
date:   2015-07-16 20:56:58
categories: iOS Dev
---
####1、什么是instancetype  
关键字instancetype是LLVM编译器特性，和Objective-C语法无关。instancetype用来表示Related Result Types(相关返回类型)，让编译器明确知道`类方法`的返回类型。  
也就是说，**instancetype关键字，保证了编译器能够正确推断方法返回值的类型。**一般用于类中**返回自身的实例的实例对象的方法。**


####2、instancetype解决的问题
如下代码：

	@interface Person:NSObject
	- (id)initWithName:(NSString *)name; //1、 initializer，初始化方法
	+ (id)fooWithBar:(NSString *)name;  //2、 class factory，类工厂
	@end

第一行代码`- (id)initWithName:(NSString *)name;`，编译器能够自动的将`id`转化为`instancetype`，因此id是等价于instancetype。
  
第二行代码`+ (id)fooWithBar:(NSString *)name;`则不同，返回类型为`id`，类方法的返回类型，LLVM(或者说Clang)却无法判断。

这会导致以下类似的问题（OS X系统下）：  
`[[NSFileHandle fileHandleWithStandardOutput] writeData:formattedData]`  
在Mac OS X（只在该OS版本）中会报错“Multiple methods named 'writeData:' found with mismatched result, parameter type or attributes.”  
原因是NSFileHandle和NSURLHandle都提供writeData:方法。由于`[NSFileHandle fileHandleWithStandardOutput]` 返回的类型是id，编译器并不确定请求了哪个类的writeData:方法。

####3、instancetype的推荐用法   

*	对于`返回自身类型的类方法`，强烈推荐使用`instancetype`代替`id`,让编译器更加的明确你的意图，避免混淆出错。
			
		+ (id)fooWithBar:(NSString *)name;
		+ (instancetype)fooWithBar:(NSString *)name;      //推荐做法~
		
*	对于`返回自身类型的实例方法`，推荐用`instancetype`代替`id`，让代码风格更加的明确和统一，增强代码可读性。   
	
		- (id)initWithName:(NSString *)name;  //等价于
		- (instancetype)initWithName:(NSString *)name;    //更好的做法~	  

####4、 instancetype和id、NSObject *区别  

*	NSObject是SDK中的最原始基类，封装内存管理等基础功能，NSObject *基类的指针，用户和面向对象中的父类类似。
*	id可以执行任何的对象类型，属于Objective-C语法概念，可以用它实现面向对象OO的多态性；
*	instancetype编译器的关键字，属于LLVM的优化特性。
*	instancetype只能用于函数返回类型，id和NSObject类似可定义变量、参数、属性等。
*	对于init返回，id和instancetype是一样的。


###Reference To：  
---  
*	[ClangClang Language Extensions](http://clang.llvm.org/docs/LanguageExtensions.html#objective-c-features)
*	[Would it be beneficial to begin using instancetype instead of id?](http://stackoverflow.com/questions/8972221/would-it-be-beneficial-to-begin-using-instancetype-instead-of-id)