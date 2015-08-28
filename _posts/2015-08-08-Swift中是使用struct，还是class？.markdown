---
layout: post
title:  "Swift中是使用Struct，还是Class？"
date:   2015-08-08 12:56:58
categories: Swift
---
###Swift中是使用Struct，还是Class？   
概述：大学时代学习C++的时候，记得有这么一个问题，使用结构体还是类呢？那时候的老师，给的答案是结构体适用于纯数据的封装，类适用于数据的封装并且定义一套行为准备？并没有更加深入的思考，这两者的差异。  
Swift语言同样引入Struct和Class这两种数据封装语法，并且据说整个Swift中，只有4个类，其他的都是使用结构体或者枚举实现。不经要问，为什么？Struct比Class更好？或者其他什么原因？   

####一、Struct和Class的差别  
*	1、Struct是值类型，Class是引用类型  
	值类型和引用类型的主要差异在于赋值或者函数传参的时候行为不同，值类型内存拷贝，引用类型引用的拷贝指向同一块对象内存。也就是说，值变量与内存是一对一的关系，而引用则是一块内存可以对应n变量。   
*	2、Struct不支持继承，Class支持继承  
	应该说Class的特性是为了满足`面向对象编程`的特性，从Class的角度来说，Swift是一门`面向对象编程`的语言。


####二、一些建议
*	1、默认使用Struct，因为使用Stuct构建系统，能降低复杂度。如果当Struct变得非常大，并且需要继承特性的时候，使用Class。
*	2、Struct更加的安全，即使在多线程的情况下，无缺陷(bug free)。
*	

------
###References： 
[Should I use a Swift struct or a class?](http://faq.sealedabstract.com/structs_or_classes/)  
[A Warm Welcome to Structs and Value Types](https://www.objc.io/issues/16-swift/swift-classes-vs-structs/)  
[why-choose-struct-over-class](http://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24232845#24232845)