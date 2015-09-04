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
```
 







































