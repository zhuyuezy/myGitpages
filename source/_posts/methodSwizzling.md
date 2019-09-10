---
title: Hook之 Method Swizzling
date: 2018-05-18 16:32:00
tags: 
- Objective-C 
- hook 
categories: 
- objective-c基础
---




Method Swizzling利用了Objective-C的runtime特性，使得我们能动态替换方法的实现，实现hook。
<!-- more -->

# runtime
从敲下`NSlog(@"hello world!")`开始，就不断看到runtime这个词。对于之前没接触过动态语言的学习者来说，runtime听起来陌生又晦涩。

Objective-C不同于C语言，它是一门动态语言，这就意味着它将C语言等静态语言在编译和链接时所做的一些工作留到了运行时再处理。比如，在C语言中，编译时就可以确定真正调用哪些函数；而Objective-C并不能在编译时确定，只有在运行时才会由名称找到对应方法。

因为这种特性，Objective-C需要一种机制在运行时来处理编译后的代码，这种机制就是runtime。Runtime从实质来讲，就是一套用C语言和汇编写成的底层API，用于处理编译后的代码。

# method
![methodStruct](/images/methodSwizzling/1.png "method结构体")
method_name 表示的是方法的名称，用于唯一标识某个方法；
method_types 表示的是方法的返回值和参数类型；
method_imp 是一个函数指针，指向方法的实现。
方法的名称 name 和方法的实现 imp 是一一对应的。而且可以发现方法的名称和参数类型是分离的。
在我们调用一个方法时

	[someObject messageName:paremeter]
在runtime底层会转换成

	Objc_msgSend(someObject, @selector(messageName), paremeter)

调用方法其实是向一个对象发送消息，而查找方法实现的唯一依据是selector后接的方法名称。所以，利用这一特性，我们就可以实现在运行时更改selector对应的方法实现，也就实现了HOOK。

# method swizzling
## 原理
![methodIMP](/images/methodSwizzling/2.png "类和方法列表")
![changeIMP](/images/methodSwizzling/3.png "替换")
>图源:念茜

每个类都维护着一个方法列表，里面存放着selector的名字与方法实现间的映射关系。Method Swizzling就是改换了这种对应关系。

可以调用三种方法来实现method swizzling：
`method_exchangeImplementations` 交换2个方法中的IMP（IMPlication）；  
`class_replaceMethod` 修改类；  
利用 `method_setImplementation `直接设置某个方法的IMP。  
这些方法的声明都写在`runtime.h`中。

## 1.method_exchangeImplementations

 ![1](/images/methodSwizzling/4.png)

`method_exchangeImplementations`通过交换两个方法的IMP实现hook的，通过  
`method_exchangeImplementations(A_Method, B_Method);`  
可以实现如下图的交换。
![1](/images/methodSwizzling/5.png)

## 2.method_setImplementation

 ![1](/images/methodSwizzling/method_setImplementation.png)

`method_setImplementation`直接设置更改某个方法的IMP。  
如下图，假设有方法A，我们想要把它的实现替换为方法B的实现，但又不想改变方法B的实现，就可以使用`method_setImplementation`，直接把A的IMP设置为B的IMP。

![1](/images/methodSwizzling/6.png)


## 3.class_replaceMethod

 ![1](/images/methodSwizzling/class_rep.png)

`class_replaceMethod`其实相当于`class_addMethod`和`method_setImplementation`的一个结合。

当调用
`class_replaceMethod(cls, name,  imp,  types) `
时，将把属于`cls`类的`name`方法实现指向`imp`。  
如果`cls`类原本没有`name`方法，就相当于调用`class_addMethod`给`cls`类增添了一个方法及其实现，这个方法的返回类型由`types`给定。  
如果`cls`类原本就有`name`方法，就相当于调用`method_setImplementation`更改了`name`方法的实现为`imp`，返回类型由替换的实现`imp`决定，给定`types`将被忽略。

## 参考


[Objective-C的hook方案（一）: Method Swizzling](https://blog.csdn.net/yiyaaixuexi/article/details/9374411)  
[Hook 原理之 Method Swizzling](https://amywushu.github.io/2017/03/01/%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86-Hook-%E5%8E%9F%E7%90%86%E4%B9%8B-Method-Swizzling.html)
