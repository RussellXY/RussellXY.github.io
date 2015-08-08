---
layout: post
title: 理解Objective-C中的消息传递机制
date: 2014-10-12
comments: true
categories: iOS
tags: 
- Objective-C - objc_msgSend
keywords: iOS Objective-C selector message
description: 理解Objective-C中的消息传递机制
published: true
---

在Objective-C中调用一个类的对象的方法不叫方法调用，而叫给这个对象传递消息。消息有"名称"(name)或"选择子"(selector)，可以接受参数，而且可能还有返回值。

C语言中的函数调用方式叫"静态绑定"(static binding),意思是说，在编译期就能决定运行时所调用的函数。比如下列代码:

{% highlight ruby %}

void printHello() {
	printf("Hello, world!\n");
}
void printfGoodbye() {
	printf("Goodbye, world!\n");
}
void doTheThing(int type) {
	if (type == 0) {
		printfHello();
	} else {
		printfGoodbye();
	}
	return 0;
}

{% endhighlight %}

如果不考虑"内联"(inline)，那么编译器在编译代码的时候就已经确认程序中有printHello和printGoodbye这两个函数了，于是会直接生成调用这些函数的指令。此时函数地址实际上是硬编码在指令之中的。若是将上面那段代码写成下面这样，会如何呢？

{% highlight ruby %}

void printHello() {
	printf("Hello, world!\n");
}
void printGoodbye() {
	printf("Goodbye, world!\n");
}
void doTheThing(int type) {
	void (*func)();
	if (type == 0) {
		func = printHello;
	}
	else {
		func = printGoodbye;
	}
	func();
	return 0;
}

{% endhighlight %}

这时就得使用"动态绑定"(dynamic binding)，因为所要调用的函数只有到运行期才能确定。编译器在这种情况下生成的指令与刚才那个例子不同，在第一个例子中，if与else语句里都有函数调用指令。而在第二个例子中，只有一个函数调用指令，不过待调用的函数地址无法硬编码在指令之中，而是要在运行期读取出来。

在Objective-C中，如果向某对象传递消息，那就会使用动态绑定机制来决定需要调用的方法。在底层，所有方法都是普通的C语言函数，然而对象收到消息之后，究竟该调用哪个方法则完全于运行期决定，甚至可以在程序运行时改变，这些特性使得Objective-C称为一门真正的动态语言。
给对象发送消息可以这样来写:

```
id returnValue = [someObject messageName:parameter];
```
在本例中，someObject叫做"接受者"(receiver)，messageName叫做"选择子"(selector)。选择子与参数和起来称为"消息"(message)。编译器看到此消息后，将其转换为一条标准的C语言函数调用，所调用的函数是消息传递机制中的核心函数，叫做objc_msgSend，其"原型"(prototype)如下：

```
void objc_msgSend(id self, SEL cmd, ...)
```
这是一个“参数个数可变的函数”(variadic function)，能接受两个或两个以上的参数。第一个参数代表接受者，第二个参数代表选择子(SEL是选择子的类型)，后续参数就是消息中的那些参数，其顺序不变。选择子指的就是方法的名字。"选择子"与"方法"这两个词经常交替使用。编译器会把刚才那个例子中的消息转换为如下函数

```
id returnValue = objc_msgSend(someObject,@selector(messageName),parameter);
```
objc_msgSend函数会依据接受者与选择子的类型来调用适当的方法。为了完成此操作，该方法需要在接受者所属的类中搜寻其"方法列表"(list of methods)，如果能找到与选择子名称相符的方法，就跳至其实现代码。若找不到，那就沿着继承体系继续向上查找，等找到合适的方法之后再跳转。如果最终还是找不到相符的方法，那就执行"消息转发"(message forwarding)操作。

