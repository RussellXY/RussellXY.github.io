---
layout: post
title: 使用YapDatabase替代FMDB
date: 2015-05-11
comments: true
categories: iOS
tags: 
- YapDatabase
keywords: iOS YapDatabase FMDB SQLite
description: 使用YapDatabase替代FMDB
published: false
---


最近在研究一个开源项目[ChatSecure-iOS](https://github.com/ChatSecure/ChatSecure-iOS)时发现了一个不错的用于本地数据持久化的第三方库:[YapDatabase](https://github.com/yapstudios/YapDatabase),一番调研后决定集成到自己正在做的项目中，实践后发现比起之前用FMDB时需要写大量的sql语句来建表、增删查改等等方便太多了，不光如此**YapDatabase**还提供了好几个非常好用的功能(后面会一一叙述).

##什么是YapDatabase
**YapDatabase**由两个主要特点组成:

+ 一个面向**iOS**和**Mac**的collection/key/value形式的数据存储工具，它是通过**SQLite**实现的.
+ 一个能提供诸如二级索引、全文检索等高级功能的插件架构.

它包括如下功能:

+ **并发**.你可以同时的在其他线程对数据库做出修改时读取数据。因此你永远不用担心会阻塞主线程，同时你可以很轻松的在一个后台线程里向数据库写数据。当然,你也可以在多个线程中同时的从数据库读取数据。
+ **内置缓存**.你可以在数据库中缓存对象，当然sqlite可以进行缓存，但是sqlite缓存的是字节流，而你可以使用yapdatabse来直接缓存对象，这样内置的缓存对象特性可以使你省去了序列化对象的过程，你可以更快的处理对象。
+ **集合**.有时候单独的key是不够的，这时collection和key的组合更好用。不用担心,**YapDatabase**提供了很简便的集合用法。
+ **视图**.有使用你需要对数据过滤、分组、排序，没有问题，YapDatabase 有view特性，你再也不用写复杂的sql语句，你可以使用Objective-C代码。 YapDatabase 还可以更新它们，你可以使用这一特性得到一些视图表。
+ **索引**.使用索引使你快速定位一些反复访问的数据，索引使你更快速的找到数据。
+ **全文检索**.建立在sqilte的FTS 上（由谷歌贡献），使用最小的代价快速的找到你需要的数据。
+ **关系**.你可以在对象上建立关系，建立级联删除规则。
+ **扩展**.不仅仅是key-value，还具备很多扩展的数据结构，你可以扩展自己的数据结构。
+ **Objective-C**.使用Objective-C的api意味着你可以即刻上手。

##初次使用
开始使用YapDatabase来实现一个简单的存储操作
{% highlight ruby %}
// 创建或者打开指定路径的数据库文件
YapDatabase *database = [[YapDatabase alloc] initWithPath:dataPath];

// 获得一个数据库的connection
YapDatabaseConnection *connection = [database newConnection];

YourObject *object1 = [YourObject new];
NSString *key = @"id of this object";

// 存储一个对象
[connection readWriteWithBlock:^(YapDatabaseReadWriteTransaction *transaction){
    [transaction setObject:object1 forKey:key inCollection:NSStringFromClass([YourObject class])];
}];

// 读取这个对象
[connection readWithBlock:^(YapDatabaseReadTransaction *transaction){
    NSLog(@"YourObject:%@",[transaction objectForKey:key inCollection:NSStringFromClass(YourObject class)]);
}];
{% endhighlight %}

一个很好理解**YapDatabase**的方法是把它当作一个存储dictionaries的dictionary，区别仅仅是它会把所有的值都存储在磁盘中。
你可以在一个transaction block中做很多事情，例如：在数据库中插入对象、枚举所有对象。

##存储对象
通过**YapDatabase**你可以存储任何类型的对象，不过为了能够存储对象到磁盘中你必须实现这个对象的序列化,一个常用的方式是遵守NSCoding协议，然后实现该协议中的两个序列化和反序列化的方法，详情可阅读apple[官方文档](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/Archiving/Articles/codingobjects.html#//apple_ref/doc/uid/20000948-BCIHBJDE)
{% highlight ruby %}
@interface MyObject : NSObject <NSCoding>
@end

@implementation MyObject
{
    NSString *myString;
    UIColor *myColor;
    MyWhatever *myWhatever;
    float myFloat;
}

- (id)init // Normal init method
{
    if ((self = [super init])) {
        // ...
    }
    return self;
}

- (id)initWithCoder:(NSCoder *)decoder // NSCoding deserialization
{
    if ((self = [super init])) {
        myString = [decoder decodeObjectForKey:@"myString"];
        myColor = [decoder decodeObjectForKey:@"myColor"];
        myWhatever = [decoder decodeObjectForKey:@"myWhatever"];
        myFloat = [decoder decodeFloatForKey:@"myFloat"];
    }
    return self;
}

- (void)encodeWithCoder:(NSCoder *)encoder // NSCoding serialization
{
    [encoder encodeObject:myString forKey:@"myString"];
    [encoder encodeObject:myColor forKey:@"myColor"];
    [encoder encodeObject:myWhatever forKey:@"myWhatever"];
    [encoder encodeFloat:myFloat forKey:@"myFloat"];
}
{% endhighlight %}
