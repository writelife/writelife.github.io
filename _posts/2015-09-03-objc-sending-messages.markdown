---
layout: post
category: "objc"
title:  "Objective-C消息机制的原理"
tags: [objc,消息机制]
---
在Objective-C中，message与方法的真正实现是在执行阶段绑定的，而非编译阶段。编译器将消息表达式转换成对消息发送方法的调用,即对objc_msgSend方法的调用.  

objc\_msgSend方法包含两个主要参数：receiver, 选择器即(SEL) selector，如:  
[receiver message]; 将被转换为：objc\_msgSend(receiver, selector);  

消息包含的参数也被一起传递给objc\_msgSend，如:  
objc\_msgSend(receiver, selector, arg1, arg2, …);  


objc\_msgSend方法完成必要的一切来完成动态绑定  

1.首先查找选择器指向的子程序(方法实现)。因为不同类对同一方法有不同的实现，所以对方法的真正实现的查找依赖于receiver的类    
2.调用该子程序，并将一系列参数传递过去  
3.将该子程序的返回值作为自身的返回值  
  
Note: 编译器完成对objc\_msgSend方法的转换, 请不要在代码里直接调用objc\_msgSend.  
  
消息机制的关键是，编译器为每个类和对象构建数据结构,每个类结构包含2个必要元素    
  
1.一个指向超类的指针  
2.一个类调度表, 这个表包含了与之关联的方法选择器的入口地址  


每个对象都有一个指向所属类的指针isa。通过该指针，对象可以找到它所属的类，也就找到了其全部父类，如下图所示:  
<center>
![](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/messaging1.gif)
</center>

当向一个对象发送消息时，objc_msgSend方法根据对象的isa指针找到对象的类，然后在类的调度表（dispatch table）中查找selector。如果无法找到selector，objc_msgSend通过指向父类的指针找到父类，并在父类的调度表（dispatch table）中查找selector，以此类推直到NSObject类。一旦查找到selector，objc_msgSend方法根据调度表的内存地址调用该实现。 通过这种方式，message与方法的真正实现在执行阶段才绑定。

为了保证消息发送与执行的效率，系统会将全部selector和使用过的方法的内存地址缓存起来。每个类都有一个独立的缓存，缓存包含有当前类自己的 selector以及继承自父类的selector。查找调度表（dispatch table）前，消息发送系统首先检查receiver对象的缓存。
缓存命中的情况下，消息发送（messaging）比直接调用方法（function call）只慢一点点。

下面就显式使用objc_msgSend来验证一下,代码如下:  
![#import &ltFoundation/Foundation.h&gt]

```
int main(int argc, const char * argv[]) {

id obj = objc_msgSend(objc_msgSend([NSNumber class], @selector(alloc)), @selector(initWithInteger:), 123);
id obj1;
NSLog(@"obj = %@, obj is a %@, %@", obj, [obj class], obj1);
return 0;
}
```

运行结果:  
obj = 123, obj is a __NSCFNumber, (null)
```









































