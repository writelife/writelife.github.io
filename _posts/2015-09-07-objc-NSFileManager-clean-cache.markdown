---
layout: post
category: "objc"
title:  "ios获取文件夹大小,清空文件夹"
tags: [objc, Foreach, Loop, implicit convert]
---

获取文件夹大小,代码如下:

{% highlight objc linenos%}
- (float)checkSizeAtFolderPath:(NSString *)folderPath{
    float totalSize = 0;
    NSArray *fileList = [[NSFileManager defaultManager]contentsOfDirectoryAtPath:folderPath error:nil];

    for (NSString *fileName in fileList) {
    totalSize += [[[NSFileManager defaultManager]attributesOfItemAtPath:[folderPath stringByAppendingPathComponent:fileName] error:nil] fileSize];
    }

    return totalSize;
}
{% endhighlight %}

清空文件夹,代理如下:

{% highlight objc linenos%}
- (void)cleanDiskAtPath:(NSString *)folderPath{
    NSArray *fileList = [[NSFileManager defaultManager]contentsOfDirectoryAtPath:folderPath error:nil];

    for (NSString *fileName in fileList) {
        [[NSFileManager defaultManager]removeItemAtPath:[folderPath stringByAppendingPathComponent:fileName] error:nil];
    }
}
{% endhighlight %}
