title: NS_UNAVAILABLE和NS_DESIGNATED是什么
tags:
  - iOS基础
  - ''
categories:
  - iOS
date: 2017-12-14 19:15:00
---
经常我们在阅读一些开源项目的源码时会发现有时候作者会在声明的方法后面添加`NS_UNAVAILABLE`、`NS_EDSIGNATED_INITIALIZER`这样的关键字。这表示什么意思呢？

其实，这两个关键字是苹果提供给我们来约束定义方式，使得接口更加清晰的。

下面就来跟大家解释一下这两个字段的具体作用：

## NS_UNAVAILABLE

`NS_UNAVAILABLE`的作用是直接禁止此方法的使用，一般常见用法如下：
```
+ (instancetype)new NS_UNAVAILABLE;
+ (instancetype)init NS_UNAVAILABLE;
```
这样就不能直接使用这两种创建实例的方法，而需要使用开源库自定义的初始化方法，从而确保了创建实例的统一性

## NS_EDSIGNATED_INITIALIZER

`NS_EDSIGNATED_INITIALIZER`的作用是指定构造器，通常来说指定初始化函数对一个类来说非常重要，参数也会比较多，为了适应不同的初始化需求就有了便利初始化函数。

```
- (instancetype)initWithName:(NSString *)name NS_EDSIGNATED_INITIALIZER;
```
声明指定构造器函数的一个目的是帮助规范代码，找到程序中潜在问题。

在Objective-C中，指定初始化函数的使用有一些规范：

- 子类如果有指定初始化函数，那么指定初始化函数实现时必须调用它的直接父类的指定初始化函数
- 如果子类有指定初始化函数，那么便利初始化函数必须调用自己的其他初始化函数，不能调用super的初始化函数。
