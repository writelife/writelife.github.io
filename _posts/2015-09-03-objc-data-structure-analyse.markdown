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

/** 
* Returns the name of the method specified by a given selector.返回一个指定选择器的名字
* 
* @param sel A pointer of type \c SEL. Pass the selector whose name you wish to determine.
* 
* @return A C string indicating the name of the selector. 返回值类型为C的字符串
*/
OBJC_EXPORT const char *sel_getName(SEL sel)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

在objc-api.h查看对OBJC_EXPORT的定义
#define OBJC_VISIBLE  __attribute__((visibility("default"))) //动态库中的函数可见
#define OBJC_EXPORT  OBJC_EXTERN OBJC_VISIBLE
#define OBJC_EXTERN extern

/** 
* Registers a method with the Objective-C runtime system, maps the method 
* name to a selector, and returns the selector value. 在oc运行时注册一个方法,将方法名映射到一个选择器,并且返回选择器的值
* 
* @param str A pointer to a C string. Pass the name of the method you wish to register. 参数str为一个指向C字符串的指针
* 
* @return A pointer of type SEL specifying the selector for the named method.返回值类型为SEL类型
* 
* @note You must register a method name with the Objective-C runtime system to obtain the
*  method’s selector before you can add the method to a class definition. If the method name
*  has already been registered, this function simply returns the selector.必须在加入类定义之前,先在运行时注册方法名来获得方法选择器,如果方法名已经存在,则返回此方法选择器
*/
OBJC_EXPORT SEL sel_registerName(const char *str)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
* Returns the class name of a given object.
* 
* @param obj An Objective-C object.
* 
* @return The name of the class of which \e obj is an instance.
*/
OBJC_EXPORT const char *object_getClassName(id obj)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
* Returns a pointer to any extra bytes allocated with an instance given object.
* 
* @param obj An Objective-C object.
* 
* @return A pointer to any extra bytes allocated with \e obj. If \e obj was
*   not allocated with any extra bytes, then dereferencing the returned pointer is undefined.
* 
* @note This function returns a pointer to any extra bytes allocated with the instance
*  (as specified by \c class_createInstance with extraBytes>0). This memory follows the
*  object's ordinary ivars, but may not be adjacent to the last ivar.
* @note The returned pointer is guaranteed to be pointer-size aligned, even if the area following
*  the object's last ivar is less aligned than that. Alignment greater than pointer-size is never
*  guaranteed, even if the area following the object's last ivar is more aligned than that.
* @note In a garbage-collected environment, the memory is scanned conservatively.
*/
OBJC_EXPORT void *object_getIndexedIvars(id obj)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
//当你创建一个 Objective-C对象时，runtime会在实例变量存储区域后面再分配一点额外的空间。这么做的目的是什么呢？你可以获取这块空间起始指针（用 object_getIndexedIvars），然后就可以索引实例变量（ivars）

/** 
* Identifies a selector as being valid or invalid.
* 
* @param sel The selector you want to identify.
* 
* @return YES if selector is valid and has a function implementation, NO otherwise. 
* 
* @warning On some platforms, an invalid reference (to invalid memory addresses) can cause
*  a crash. 
*/
OBJC_EXPORT BOOL sel_isMapped(SEL sel)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
//识别一个选择器是否有效,已经做了映射,在往类定义中添加方法之前,必须先把方法名映射到选择器

/** 
* Registers a method name with the Objective-C runtime system.
* 
* @param str A pointer to a C string. Pass the name of the method you wish to register.
* 
* @return A pointer of type SEL specifying the selector for the named method.
* 
* @note The implementation of this method is identical to the implementation of \c sel_registerName.
* @note Prior to OS X version 10.0, this method tried to find the selector mapped to the given name
*  and returned \c NULL if the selector was not found. This was changed for safety, because it was
*  observed that many of the callers of this function did not check the return value for \c NULL.
*/
OBJC_EXPORT SEL sel_getUid(const char *str)
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);
//sel_getUid与sel_registerName的实现完全相同


```
 







































