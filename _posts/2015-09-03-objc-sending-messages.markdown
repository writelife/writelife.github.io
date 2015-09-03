---
layout: post
category: "objc"
title:  "Objective-C消息机制的原理"
tags: [objc,消息机制]
---
在Objective-C中，message与方法的真正实现是在执行阶段绑定的，而非编译阶段。编译器将消息表达式转换成对消息发送方法的调用,即对objc_msgSend方法的调用.
```
objc_msgSend方法包含两个主要参数：receiver, 选择器即(SEL) selector，如:  
[receiver message]; 将被转换为：objc_msgSend(receiver, selector);

消息包含的参数也被一起传递给objc_msgSend，如:
objc_msgSend(receiver, selector, arg1, arg2, …);
```

objc_msgSend方法完成必要的一切来完成动态绑定
```
1.首先查找选择器指向的子程序(方法实现)。因为不同类对同一方法有不同的实现，所以对方法的真正实现的查找依赖于receiver的类  
2.调用该子程序，并将一系列参数传递过去
3.将该子程序的返回值作为自身的返回值
```
Note: 编译器完成对objc_msgSend方法的转换, 请不要在代码里直接调用objc_msgSend.

消息机制的关键是，编译器为每个类和对象构建数据结构,每个类结构包含2个必要元素  
```
1.一个指向超类的指针
2.一个类调度表, 这个表包含了与之关联的方法选择器的入口地址
```

每个对象都有一个指向所属类的指针isa。通过该指针，对象可以找到它所属的类，也就找到了其全部父类，如下图所示:  

![](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/messaging1.gif)

