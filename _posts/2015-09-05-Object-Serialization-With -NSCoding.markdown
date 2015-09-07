---
layout: post
category: "objc"
title:  "使用NSKeyedArchiver与NSCoding协议实现自定义模型对象存储"
tags: [objc, NSCoding, runtime, NSKeyedArchiver]
---
最基本的做法

{% highlight objc %}
@interface Foo : NSObject <NSCoding>

@property (nonatomic, assign) NSInteger property1;
@property (nonatomic, assign) BOOL property2;
@property (nonatomic, copy) NSString *property3;

@end

@implementation Foo

- (id)initWithCoder:(NSCoder *)coder
{
if ((self = [super init]))
{
// Decode the property values by key, 
// and assign them to the correct ivars
_property1 = [coder decodeIntegerForKey:@"property1"];
_property2 = [coder decodeBoolForKey:@"property2"];
_property3 = [coder decodeObjectForKey:@"property3"];
}
return self;
}

- (void)encodeWithCoder:(NSCoder *)coder
{
// Encode our ivars using string keys
[coder encodeInteger:_property1 forKey:@"property1"];
[coder encodeBool:_property2 forKey:@"property2"];
[coder encodeObject:_property3 forKey:@"property3"];
}

@end
{% endhighlight %}

更灵巧一点的做法

{% highlight objc %}
@interface Foo : NSObject <NSCoding>

@property (nonatomic, assign) NSInteger property1;
@property (nonatomic, assign) BOOL property2;
@property (nonatomic, copy) NSString *property3;
- (NSArray *)propertyNames;

@end

@implementation Foo

- (NSArray *)propertyNames
{
return @[@"property1", @"property2", @"property3"];
}

- (id)initWithCoder:(NSCoder *)aDecoder
{
if ((self = [self init]))
{
// Loop through the properties
for (NSString *key in [self propertyNames])
{
// Decode the property, and use the KVC setValueForKey: method to set it
id value = [aDecoder decodeObjectForKey:key];
[self setValue:value forKey:key];
}
}
return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder
{
// Loop through the properties
for (NSString *key in [self propertyNames])
{
// Use the KVC valueForKey: method to get the property and then encode it
id value = [self valueForKey:key];
[aCoder encodeObject:value forKey:key];
}
}

@end
{% endhighlight %}

更高级的做法,使用objc的内省机制
只需要重写上面的propertyNames方法,代码如下:

{% highlight objc %}
// Import the Objective-C runtime headers
#import <objc/runtime.h> 

- (NSArray *)propertyNames
{    
// Get the list of properties
unsigned int propertyCount;
objc_property_t *properties = class_copyPropertyList([self class], 
&propertyCount);
NSMutableArray *array = [NSMutableArray arrayWithCapacity:propertyCount];
for (int i = 0; i < propertyCount; i++)
{
// Get property name
objc_property_t property = properties[i];
const char *propertyName = property_getName(property);
NSString *key = @(propertyName);

// Add to array
[array addObject:key];
}

// Remember to free the list because ARC doesn't do that for us
free(properties);

return array;
}

{% endhighlight %}
使用这种方法有个缺点,只能对当前类中定义的属性进行解编码,而不能对从父类继承而来的属性,实例变量进行解编码,让我们进一步改进,重写propertyNames方法,代码如下:

{% highlight objc %}
- (NSArray *)propertyNames
{
// Check for a cached value (we use _cmd as the cache key, 
// which represents @selector(propertyNames))
NSMutableArray *array = objc_getAssociatedObject([self class], _cmd);
if (array)
{
return array;
}

// Loop through our superclasses until we hit NSObject
array = [NSMutableArray array];
Class subclass = [self class];
while (subclass != [NSObject class])
{
unsigned int propertyCount;
objc_property_t *properties = class_copyPropertyList(subclass, 
&propertyCount);
for (int i = 0; i < propertyCount; i++)
{
// Get property name
objc_property_t property = properties[i];
const char *propertyName = property_getName(property);
NSString *key = @(propertyName);

// Check if there is a backing ivar
char *ivar = property_copyAttributeValue(property, "V");
if (ivar)
{
// Check if ivar has KVC-compliant name
NSString *ivarName = @(ivar);
if ([ivarName isEqualToString:key] || 
[ivarName isEqualToString:[@"_" stringByAppendingString:key]])
{
// setValue:forKey: will work
[array addObject:key];
}
free(ivar);
}
}
free(properties);
subclass = [subclass superclass];
}

// Cache and return array
objc_setAssociatedObject([self class], _cmd, array, 
OBJC_ASSOCIATION_RETAIN_NONATOMIC);
return array;
}

{% endhighlight %}
至此,你可以随意NSCoding你自己的类了.
