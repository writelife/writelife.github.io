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


每个对象都有一个指向所属类的指针isa。通过该指针，对象可以找到它所属的类，也就找到了其全部超类，如下图所示:  
<center>
![](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/messaging1.gif)
</center>

当向一个对象发送消息时，objc\_msgSend方法根据对象的isa指针找到对象的类，然后在类的调度表（dispatch table）中查找selector。如果无法找到selector，objc\_msgSend通过指向超类的指针找到超类，并在超类的调度表（dispatch table）中查找selector，以此类推直到NSObject类。一旦查找到selector，objc\_msgSend方法根据调度表的内存地址调用该实现。 通过这种方式，message与方法的真正实现在执行阶段才绑定。

为了保证消息发送与执行的效率，系统会将全部selector和使用过的方法的内存地址缓存起来。每个类都有一个独立的缓存，缓存包含有当前类自己的 selector以及继承自超类的selector。查找调度表（dispatch table）前，消息发送系统首先检查receiver对象的缓存。
缓存命中的情况下，消息发送（messaging）比直接调用方法（function call）只慢一点点。

下面就显式使用objc_msgSend来验证一下,代码如下:  
{% highlight objc %}
#import <Foundation/Foundation.h>


int main(int argc, const char * argv[]) {

    id obj = objc_msgSend(objc_msgSend([NSNumber class], @selector(alloc)), @selector(initWithInteger:), 123);
    id obj1;
    NSLog(@"obj = %@, obj is a %@, %@", obj, [obj class], obj1);
    return 0;
}
{% endhighlight %}

运行结果:  
obj = 123, obj is a __NSCFNumber, (null)

显然,obj的输出结果与直接使用obj=@123一致

接下来还是分析下message.h的源码吧

{% highlight objc %}
struct objc_super {
/// Specifies an instance of a class.
__unsafe_unretained id receiver;

/// Specifies the particular superclass of the instance to message. 
#if !defined(__cplusplus)  &&  !__OBJC2__
/* For compatibility with old objc-runtime.h header */
__unsafe_unretained Class class;
#else
__unsafe_unretained Class super_class;
#endif
/* super_class is the first class to search */
};
//很明显的objc_super主要包含一个receiver和一个指向超类的指针

/* Struct-returning Messaging Primitives
*
* Use these functions to call methods that return structs on the stack. 
* On some architectures, some structures are returned in registers. 
* Consult your local function call ABI documentation for details.
* 
* These functions must be cast to an appropriate function pointer type 
* before being called. 
*/
#if !OBJC_OLD_DISPATCH_PROTOTYPES
OBJC_EXPORT void objc_msgSend_stret(void /* id self, SEL op, ... */ )
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
OBJC_ARM64_UNAVAILABLE;
OBJC_EXPORT void objc_msgSendSuper_stret(void /* struct objc_super *super, SEL op, ... */ )
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
OBJC_ARM64_UNAVAILABLE;
#else
/** 
* Sends a message with a data-structure return value to an instance of a class.
* 
* @see objc_msgSend
*/
OBJC_EXPORT void objc_msgSend_stret(id self, SEL op, ...)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
OBJC_ARM64_UNAVAILABLE;

/** 
* Sends a message with a data-structure return value to the superclass of an instance of a class.
* 
* @see objc_msgSendSuper
*/
OBJC_EXPORT void objc_msgSendSuper_stret(struct objc_super *super, SEL op, ...)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
OBJC_ARM64_UNAVAILABLE;
#endif


/* Floating-point-returning Messaging Primitives
* 
* Use these functions to call methods that return floating-point values 
* on the stack. 
* Consult your local function call ABI documentation for details.
* 
* arm:    objc_msgSend_fpret not used
* i386:   objc_msgSend_fpret used for `float`, `double`, `long double`.
* x86-64: objc_msgSend_fpret used for `long double`.
*
* arm:    objc_msgSend_fp2ret not used
* i386:   objc_msgSend_fp2ret not used
* x86-64: objc_msgSend_fp2ret used for `_Complex long double`.
*
* These functions must be cast to an appropriate function pointer type 
* before being called. 
*/
#if !OBJC_OLD_DISPATCH_PROTOTYPES

# if defined(__i386__)

OBJC_EXPORT void objc_msgSend_fpret(void /* id self, SEL op, ... */ )
__OSX_AVAILABLE_STARTING(__MAC_10_4, __IPHONE_2_0);

# elif defined(__x86_64__)

OBJC_EXPORT void objc_msgSend_fpret(void /* id self, SEL op, ... */ )
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
OBJC_EXPORT void objc_msgSend_fp2ret(void /* id self, SEL op, ... */ )
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

# endif

// !OBJC_OLD_DISPATCH_PROTOTYPES
#else
// OBJC_OLD_DISPATCH_PROTOTYPES
# if defined(__i386__)

/** 
* Sends a message with a floating-point return value to an instance of a class.
* 
* @see objc_msgSend
* @note On the i386 platform, the ABI for functions returning a floating-point value is
*  incompatible with that for functions returning an integral type. On the i386 platform, therefore, 
*  you must use \c objc_msgSend_fpret for functions returning non-integral type. For \c float or 
*  \c long \c double return types, cast the function to an appropriate function pointer type first.
*/
OBJC_EXPORT double objc_msgSend_fpret(id self, SEL op, ...)
__OSX_AVAILABLE_STARTING(__MAC_10_4, __IPHONE_2_0);

/* Use objc_msgSendSuper() for fp-returning messages to super. */
/* See also objc_msgSendv_fpret() below. */

# elif defined(__x86_64__)
/** 
* Sends a message with a floating-point return value to an instance of a class.
* 
* @see objc_msgSend
*/
OBJC_EXPORT long double objc_msgSend_fpret(id self, SEL op, ...)
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

#  if __STDC_VERSION__ >= 199901L
OBJC_EXPORT _Complex long double objc_msgSend_fp2ret(id self, SEL op, ...)
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
#  else
OBJC_EXPORT void objc_msgSend_fp2ret(id self, SEL op, ...)
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
#  endif

/* Use objc_msgSendSuper() for fp-returning messages to super. */
/* See also objc_msgSendv_fpret() below. */

# endif

// OBJC_OLD_DISPATCH_PROTOTYPES
#endif


/* Direct Method Invocation Primitives
* Use these functions to call the implementation of a given Method.
* This is faster than calling method_getImplementation() and method_getName().
*
* The receiver must not be nil.
*
* These functions must be cast to an appropriate function pointer type 
* before being called. 
*/
#if !OBJC_OLD_DISPATCH_PROTOTYPES
OBJC_EXPORT void method_invoke(void /* id receiver, Method m, ... */ ) 
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
OBJC_EXPORT void method_invoke_stret(void /* id receiver, Method m, ... */ ) 
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0)
OBJC_ARM64_UNAVAILABLE;
#else
OBJC_EXPORT id method_invoke(id receiver, Method m, ...) 
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
OBJC_EXPORT void method_invoke_stret(id receiver, Method m, ...) 
__OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0)
OBJC_ARM64_UNAVAILABLE;
#endif

/* Message Forwarding Primitives
* Use these functions to forward a message as if the receiver did not 
* respond to it. 
*
* The receiver must not be nil.
* 
* class_getMethodImplementation() may return (IMP)_objc_msgForward.
* class_getMethodImplementation_stret() may return (IMP)_objc_msgForward_stret
* 
* These functions must be cast to an appropriate function pointer type 
* before being called. 
*
* Before Mac OS X 10.6, _objc_msgForward must not be called directly 
* but may be compared to other IMP values.
*/
#if !OBJC_OLD_DISPATCH_PROTOTYPES
OBJC_EXPORT void _objc_msgForward(void /* id receiver, SEL sel, ... */ ) 
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
OBJC_EXPORT void _objc_msgForward_stret(void /* id receiver, SEL sel, ... */ ) 
__OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_0)
OBJC_ARM64_UNAVAILABLE;
#else
OBJC_EXPORT id _objc_msgForward(id receiver, SEL sel, ...) 
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
OBJC_EXPORT void _objc_msgForward_stret(id receiver, SEL sel, ...) 
__OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_0)
OBJC_ARM64_UNAVAILABLE;
#endif


/* Variable-argument Messaging Primitives
*
* Use these functions to call methods with a list of arguments, such 
* as the one passed to forward:: .
*
* The contents of the argument list are architecture-specific. 
* Consult your local function call ABI documentation for details.
* 
* These functions must be cast to an appropriate function pointer type 
* before being called, except for objc_msgSendv_stret() which must not 
* be cast to a struct-returning type.
*/

typedef void* marg_list;

OBJC_EXPORT id objc_msgSendv(id self, SEL op, size_t arg_size, marg_list arg_frame) OBJC2_UNAVAILABLE;
OBJC_EXPORT void objc_msgSendv_stret(void *stretAddr, id self, SEL op, size_t arg_size, marg_list arg_frame) OBJC2_UNAVAILABLE;
/* Note that objc_msgSendv_stret() does not return a structure type, 
* and should not be cast to do so. This is unlike objc_msgSend_stret() 
* and objc_msgSendSuper_stret().
*/
#if defined(__i386__)
OBJC_EXPORT double objc_msgSendv_fpret(id self, SEL op, unsigned arg_size, marg_list arg_frame) OBJC2_UNAVAILABLE;
#endif
{% endhighlight %}






































