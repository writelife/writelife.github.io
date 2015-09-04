---
layout: post
category: "objc"
title:  "objc之objc数据结构分析"
tags: [objc,runtime]
---
```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

struct objc_class {
Class isa;
} OBJC2_UNAVAILABLE;

/// Represents an instance of a class.
struct objc_object {
Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
//可以看出id就是一个objc_object结构体指针,所以使用id的时候前面不用再加*

/// An opaque type that represents a method selector.
typedef struct objc_selector *SEL;
//objc_selector结构体的详细定义没有在头文件中找到。方法的selector用于表示运行时方法的名字。

#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id (*IMP)(id, SEL, ...); 
#endif
//默认OBJC_OLD_DISPATCH_PROTOTYPES为No,使用typedef void (*IMP)(void /* id, SEL, ... */ );

再来看看对Nil, nil的定义
#ifndef Nil
# if __has_feature(cxx_nullptr)
#   define Nil nullptr
# else
#   define Nil __DARWIN_NULL
# endif
#endif

#ifndef nil
# if __has_feature(cxx_nullptr)
#   define nil nullptr
# else
#   define nil __DARWIN_NULL
# endif
#endif

_types.h中有对__DARWIN_NULL的定义
#define __DARWIN_NULL ((void *)0)

可以看出Nil本质上是：(void *)0
用于表示指向Objective-C类（Class）类型的指针为空

nil本质上也是：(void *)0
用于表示指向Objective-C对象的指针为空

#if ! (defined(__OBJC_GC__)  ||  __has_feature(objc_arc))
#define __strong /* empty */
#endif

#if !__has_feature(objc_arc)
#define __unsafe_unretained /* empty */
#define __autoreleasing /* empty */
#endif

在MRC下使用__strong, __unsafe_unretained, __autoreleasing

```
 







































