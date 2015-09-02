---
layout: post
category: "ios"
title:  "ios之获取最新ua"
tags: [html解析,ios]
---
鉴于不少人使用User Agents Switcher时,得手动添加UA比较繁琐,这里就用ios环境下写个自动获取最新ua的程序,由于ios沙盒路径的限制,生成的
文件必须到沙盒下取,在cocoa下使用好些...

这里直接使用了TFHpple包进行html解析,代码如下:
```
#import "ViewController.h"
#import <TFHpple.h>

@interface ViewController ()
@property(nonatomic, strong)NSArray *uaArray;
@end

@implementation ViewController
@synthesize uaArray;

- (void)viewDidLoad {
[super viewDidLoad];

[self myUAXMLCreate];
}

- (void)htmlParser{

NSString *htmlString = [NSString stringWithContentsOfURL:[NSURL URLWithString:@"https://techblog.willshouse.com/2012/01/03/most-common-user-agents/"] encoding:NSUTF8StringEncoding error:nil];
NSData *htmlData = [htmlString dataUsingEncoding:NSUTF8StringEncoding];
TFHpple *xpathParser = [[TFHpple alloc]initWithHTMLData:htmlData];
NSArray *elements = [xpathParser searchWithXPathQuery:@"//textarea"];
TFHppleElement *element = [elements objectAtIndex:0];
NSString *uaText = [[element firstTextChild]content];
NSLog(@"%@", uaText);

uaArray = [uaText componentsSeparatedByString:@"\n" ];

}

- (void)myUAXMLCreate{
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *saveDirectory = [paths objectAtIndex:0];
NSString *saveFileName = @"ua.xml";
NSString *filePath = [saveDirectory stringByAppendingPathComponent:saveFileName];

NSMutableString *xmlString = [[NSMutableString alloc]initWithString:@"<useragentswitcher>"];

[xmlString appendString:@"<folder description=\"UA-Top\">"];

[self htmlParser];
int index = 0;

for (NSString *str in uaArray) {

NSString *tempStr = [NSString stringWithFormat:@"<useragent description=\"%d\" badge=\"%d\" useragent=\"%@\" appcodename=\"\" appname=\"\" appversion=\"\" platform=\"\" vendor=\"\" vendorsub=\"\" />", index, index, str];
[xmlString appendString:tempStr];
index++;

}

[xmlString appendString:@"</folder>"];
[xmlString appendString:@"</useragentswitcher>"];

NSLog(@"filePath=%@", filePath);
[xmlString writeToFile:filePath atomically:YES encoding:NSUTF8StringEncoding error:nil];

}

```