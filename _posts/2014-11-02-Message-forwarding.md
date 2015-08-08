---
layout: post
title: 通过消息转发机制实现给@dynamic属性添加存取方法
date: 2014-11-02
comments: true
categories: iOS
tags: 
- Objective-C - message forwarding - dynamic property
keywords: iOS Objective-C selector message-forwarding
description: 通过消息转发机制实现给@dynamic属性添加存取方法
published: true
---


对于iOS开发者来说**unrecognized selector sent to instance**应该是见过无数次了，这段异常信息是由NSObject的"doesNotRecognizeSelector:"方法所抛出的，其实在这个消息发送给NSObject之前，运行时系统还做了一件事情来让对象处理它所无法响应的消息，这就是"消息转发"(message forwarding)机制，开发者可以在这个过程中告诉对象应该如何处理未知消息。

消息转发分为两个阶段。第一个阶段先询问接受者所属的类，看其是否能动态添加方法以处理当前这个未知的选择子，这叫做"动态方法解析"(dynamic method resolution)。
对象在收到无法响应的消息后，首先将调用其所属类的下列类方法：

```
+ (BOOL)resolveInstanceMethod:(SEL)selector
```
该方法的参数就是那个未知的选择子，其返回值为Boolean类型，表示这个类是否能新增一个实例方法用于处理此选择子。假如尚未实现的方法不是实例方法而是类方法，那么运行时系统就会调用另外一个方法：`resolveClassMethod:`。
使用这种办法的前提是：相关方法的实现代码已经写好，只等着运行的时候动态加入到类里面。

第二阶段涉及"完整的消息转发机制"(full forwarding mechanism)。如果运行期时统已经把第一阶段执行完了，那么接受者自己就无法再以动态新增方法的手段来响应包含该选择子的消息了。此时，运行时系统会请求接受者能否把这条消息转给其他接受者来处理，并调用该对象的这个方法：

```
- (id)forwardingTargetForSelector:(SEL)selector
```
方法参数代表未知选择子，若当前接受者能找到其他能处理此选择子的对象则将其返回，若做不到则返回nil。

如果未知选择子并没有交给其他接收者处理，那么接下来要做的就是启用完整的消息转发机制。首先创建NSInvocation对象，把与尚未处理的那条消息有关的全部细节都封装于其中。此对象包含选择子、目标(target)及参数。在触发NSInvocation对象时，"消息派发系统"(message-dispatch system)会把消息指派给目标对象。
此步骤会调用下列方法来转发消息:

```
- (void)forwardInvocation:(NSInvocation *)invocation
```
实现此方法时，若发现某调用操作不应由本类处理，则需要调用超类的同名方法。这样的话，继承体系中的每个类都有机会处理此消息，直到NSObject，然后该方法就会调用`doesNotRecognizeSelector:`以抛出异常，就像一开始说的那样。

图示消息转发机制全流程：

![图片加载失败](./../../../../../assets/images/message-forwarding-flow.png "消息转发机制全流程")


###使用动态方法解析来实现@dynamic属性

假设要编写一个类似"字典"的对象，它里面可以容纳其他对象，只不过开发者要直接通过属性来存取其中的数据。这个类的设计思路是：由开发者来添加属性定义，并将其声明为@dynamic，而类则会自动处理相关属性值的存放与获取操作。下面是该类的接口定义：

{% highlight ruby %}

@interface XYAutoDictionary : NSObject

@property (nonatomic,strong) NSString *name;
@property (nonatomic,strong) NSNumber *number;
@property (nonatomic,strong) id customObject;
@property (nonatomic,strong) NSDate *date;

@end

{% endhighlight %}

在类的内部，每个属性的值还是会存放在字典里，所以我们要将属性声明为@dynamic，这样的话，编译器就不会为其自动生成实例变量和存取方法。

{% highlight ruby %}

@interface XYAutoDictionary ()

@property (nonatomic,strong) NSMutableDictionary *propertyDic;

@end

@implementation XYAutoDictionary

@dynamic name,number,customObject,date;

- (instancetype)init {
    if (self = [super init]) {
        _propertyDic = [NSMutableDictionary dictionary];
    }
    
    return self;
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selString = NSStringFromSelector(sel);
    if ([selString hasPrefix:@"set"]) {
        class_addMethod([XYAutoDictionary class], sel, (IMP)xy_autoDictionarySetter, "v@:@");
    }
    else {
        class_addMethod([XYAutoDictionary class], sel, (IMP)xy_autoDictionaryGetter, "@@:");
    }
    return YES;
}

id xy_autoDictionaryGetter(id self,SEL _cmd) {
    XYAutoDictionary *typedSelf = (XYAutoDictionary *)self;
    NSString *key = NSStringFromSelector(_cmd);
    return [typedSelf.propertyDic objectForKey:key];
}

void xy_autoDictionarySetter(id self, SEL _cmd, id value) {
    XYAutoDictionary *typedSelf = (XYAutoDictionary *)self;
    NSMutableString *key = [NSStringFromSelector(_cmd) mutableCopy];
    
    // 去掉setXXX:后面冒号
    [key deleteCharactersInRange:NSMakeRange(key.length - 1, 1)];
    // 去掉set
    [key deleteCharactersInRange:NSMakeRange(0, 3)];
    
    // 首字母小写
    NSString *lowercaseFirstChar = [[key substringToIndex:1] lowercaseString];
    [key replaceCharactersInRange:NSMakeRange(0, 1) withString:lowercaseFirstChar];
    if (value) {
        [typedSelf.propertyDic setObject:value forKey:key];
    }
    else {
        [typedSelf.propertyDic removeObjectForKey:key];
    }
}

@end

{% endhighlight %}

然后在main方法中实验一下

{% highlight ruby %}

XYAutoDictionary *autoDictionary = [[XYAutoDictionary alloc] init];
autoDictionary.name = @"russell";
NSLog(@"Name:%@",autoDictionary.name);

// output: Name:russell

{% endhighlight %}



