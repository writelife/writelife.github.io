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
```
 







































