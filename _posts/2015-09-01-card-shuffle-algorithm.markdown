---
layout: post
category: "算法"
title:  "洗牌算法探究"
tags: [洗牌算法,ios]
---
洗牌算法在我们的日常生活中应用比较广泛,比如我们玩的纸牌游戏,扫雷,以及平常经常用到的行文本打乱工具...

鉴于现在从事ios开发,那就用oc代码实现吧.

代码如下所示:

```
+ (void)defaultShuffleWithArray:(NSMutableArray *)dataSource
{
    int index;
    int value;
    int median;

    if (dataSource.count==0) {
        return;
    }

    for (index = 0; index < dataSource.count; index++) {

        value = arc4random()%dataSource.count;
        median = [dataSource[index]intValue];
        dataSource[index] = dataSource[value];
        dataSource[value] = [NSNumber numberWithInt:median];

    }
}
```