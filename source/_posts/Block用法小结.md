title: Block用法小结
tags:
  - iOS进阶
categories:
  - iOS
date: 2017-06-06 16:20:00
---

这篇博客将系统整理一下Block相关的知识

首先，先思考一个问题Block能做些什么？

- Block内部能够读取外部局部变量的值
- 经过`__block`修饰符修饰的变量，Block内部不仅能够读取它的值，而且可以改变这个变量的值
- Block可以用来作为控制器之间的一个通信
- Block可以作为方法的参数

## 进一步深挖block的原理
#### 为什么不加__block修饰符就不能改变Block外部的局部变量?

对于block外的变量引用，block默认是将其复制到其数据结构中实现访问的，对于用__block修饰的外部变量引用，block是复制其引用地址来实现访问的，所以加上__block就能改变block外部的变量

#### Block如何作为控制器之间的一个通信？

举例说明：
1. 页面B的.h文件中声明一个block变量
```
// 方法一

typedef void(^blo)(NSString *str);
@property (nonatomic, copy) blo block;

// 方法二

@property (nonatomic, copy) void (^blo)(NSString *str);

```
2. 页面A中创建B页面，并调用Block变量
```
// 方法一

B *b = [[B alloc] init];

__weak A *weakself = self;
b.block = ^(NSString *str) {
    weakself.str = str;
}

// 方法二

B *b = [[B alloc] init];

__weak A *weakself = self;
b.blo = ^(NSString *str) {
    weakself.str = str;
}

```

3. 在页面B中调用Block变量

```
self.block("string");
```

#### 为何A页面要用__weak修饰？
使用__weak修饰符是为了避免引起循环引用。如果在Block中使用了__strong修饰符修饰的对象，那么当Block从栈复制到堆时，改对象为Block所有，这就容易引起循环引用，从而发生内存泄漏

#### 如何使用Block作为方法的参数

```
- (void)blockAsParameter:(void (^)())block {
    block(); // 执行block内部的代码
}

[self blockAsParameter:^{
    NSLog(@"");
}];
```

#### block的三种类型
1. 不管在ARC还是MRC环境下，block内部如果没有访问外部变量，这个block就是全局block `_NSConcreteGlobalBlock`，存储在内存中的代码区
2. 在MRC下，block内部如果访问外部变量，这个block就是栈block `_NSConcreteStackBlock`，存储在内存中的栈上，当函数返回时会被销毁
3. 在MRC下，block内部访问外部变量，同时对该block做一次copy操作，这个block就是堆block `_NSConcreteMallocBlock`，存储在内存的堆上，当引用计数为 0 时会被销毁
4. 在ARC下，block内部如果访问了外部变量，这个block就是堆block，因为在ARC下，默认对block做一次copy操作

### 总结一些block使用的注意点

1.默认情况下，block内部可以访问外部变量，但是不能修改外部变量，如果想要修改外部变量，可以给外部变量加上`__block`关键字，block是复制其引用地址来实现访问的
2. block作为属性应该用copy修饰，如果我们使用weak、assign来修饰block属性，block访问外部变量时类型就是栈block，而保存在栈中的block在其所在的函数、方法返回时，该block就会被销毁，在其他方法内部调用改block时，就会引发野指针错误。

### 参考文献

[谈Objective-C block的实现](http://blog.devtang.com/2013/07/28/a-look-inside-blocks/)
