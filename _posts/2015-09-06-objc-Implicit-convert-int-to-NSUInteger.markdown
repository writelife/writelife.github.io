---
layout: post
category: "objc"
title:  "objc之for循环中的隐式转换"
tags: [objc, Foreach, Loop, implicit convert]
---

对于以下代码:

{% highlight objc linenos%}
for (int i = -1; i < 3; i++) {
    NSLog(@"%d", i);
}
{% endhighlight %}
毫无疑问的,大家都知道会输出结果, -1, 0 ,1, 2

那么对于以下代码,大家就得注意一下:

{% highlight objc linenos%}
NSArray *Array = @[@1, @2, @3];
for (int i = -1; i < Array.count; i++) {
    NSLog(@"%d", i);
}
{% endhighlight %}
结果是不会有任何输出,原因是数组个数为无符号整数, 对初始化语句赋值 i = -1时,内部做了隐式转换,把int的-1转换成NSUInteger类型,变成一个很大的数.