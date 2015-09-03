---
layout: post
category: "objc"
title:  "objc之runtime源码分析"
tags: [objc,runtime]
---
<link rel="stylesheet" href="/path/to/styles/default.css">
<script src="/path/to/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
<pre><code class="Objective C">
/* Types */

/// An opaque type that represents a method in a class definition.  
typedef struct objc_method *Method;//objc_method结构体指针

/// An opaque type that represents an instance variable.  
typedef struct objc_ivar *Ivar;

/// An opaque type that represents a category.  
typedef struct objc_category *Category;

/// An opaque type that represents an Objective-C declared property.  
typedef struct objc_property *objc_property_t;  

</code></pre>





































