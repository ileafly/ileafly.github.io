title: iOS的引用计数与循环引用
tags:
  - iOS基础
categories:
  - iOS
date: 2018-08-22 13:36:00
---
## 什么是引用计数？

引用计数是一个简单而有效的管理对象生命周期的方式。

- 当我们创建一个新对象时，它的引用计数为1
- 当有一个新的指针指向这个对象时，我们将引用计数加1
- 当某个指针不再指向这个对象时，我们将引用计数减1
- 当对象的引用计数为0时，说明这个对象不再被任何指针指向了，就可以将对象销毁，回收内存

![memory-ref-count.png.jpeg](http://upload-images.jianshu.io/upload_images/1479547-30385df644b8cd5a.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 引用计数的运用场景

引用计数真正派上用场的场景是在面向对象的程序设计架构中，用于对象之间传递和共享数据。

以对象M为例，哪些对象需要长时间使用这个对象，就把它的引用计数加1，使用完之后再把引用计数减1.所有对象都遵守这个规则的话，对象的生命周期管理就可以完全交给引用计数了。

## ARC下的常见内存管理问题

#### 循环引用问题 (Reference Cycle)

引用计数这种管理内存的方式虽然简单，但是有一个比较大的瑕疵，`它不能很好的解决循环引用问题`

1. 什么是循环引用问题？

对象A和对象B，相互引用了对方作为自己的成员变量，只有当自己销毁时，才会将成员变量的引用计数减1，这就导致了A的销毁依赖于B的销毁，同样B的销毁依赖于A的销毁，这样就造成了循环引用问题。

![memory-cycle-1.png.jpeg](http://upload-images.jianshu.io/upload_images/1479547-121197b1f2cebf08.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不仅仅只在两个对象中存在循环引用问题，多个对象依次持有对方，形成一个环状，也会造成循环引用问题。

![memory-cycle-2.png](http://upload-images.jianshu.io/upload_images/1479547-1943ef26038fcb19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 如何解决循环引用问题？

###### 主动断开循环引用

在合理的位置主动断开环中的一个引用，使得对象得以回收。
![memory-cycle-3.png](http://upload-images.jianshu.io/upload_images/1479547-f73d7d5934c88e1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
主动断开循环引用这种方式常见于各种与block相关的代码逻辑中。

例如：网络请求的回调block是被持有的，如果这个block中又存在对于View Controller的引用，就容易产生循环引用。
- Controller 持有网络请求对象
- 网络请求对象持有了回调的block
- 回调的block里面使用了self，所以持有了Controller

```
// View Controller

- (void)sendRequest {
// Controller 持有网络请求对象
    Request *request = [Request alloc] initWithPath:@"https://www.baidu.com"];
    [request startWithCompletionBlockWithSuccess:^(Block block) {
    // 回调的block里面使用了self，所以持有了Controller
        [self reloadView];
    } failure:^(NSError *error) {
        [self toast];
    }
}

// Request

// 网络请求对象持有了回调block
- (void)startWithCompletionBlockWithSuccess:(void (^)(Block block))success failure:(void (^)(NSError *error) {
    self.successCompletionBlock = success;
    self.failureCompletionBlock = failure;
}

```

解决这个问题的办法就是在网络请求结束后，主动释放对block的持有，打破循环引用

```
- (void)clearCompletionBlock {
    self.successCompletionBlock = nil;
    self.failureCompletionBlock = nil;
}
```

##### 使用弱引用

主动断开循环引用需要程序员能够准确发现循环引用，并知道什么时机断开循环引用，所以这种解决方法并不常见，更常见的办法是使用弱引用

弱引用虽然持有对象，但是不增加引用计数，这样就避免了循环引用的产生。例如delegate模式中的weak声明。View Controller的delegate成员变量通常是一个弱引用，以避免两个View Controller互相引用对方造成循环引用。

![memory-cycle-4.png.jpeg](http://upload-images.jianshu.io/upload_images/1479547-05e4202ef56b249c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 弱引用的实现原理

系统对于每一个有弱引用的对象，都维护一个表来记录它所有的弱引用的指针地址。当一个对象的引用计数为0时，系统就通过这张表，找到所有的弱引用指针，将他们置为nil。`弱引用的使用是有额外的开销的，虽然这个开销很小，但是如果肯定不需要使用弱引用特性时，就不应该盲目使用弱引用`

##### Block如何避免循环引用

- 当block本身不被self持有，而被别的对象持有，同时不产生循环引用的时候，就不需要weakself，最常见的代码就是UIView的动画代码

```
[UIView animateWithDuration:0.2 animations:^{  
  self.alpha = 1;
}];
```
- 通过weakSelf和strongSelf解决循环引用

```
__weak __typeof (self)weakSelf = self;

_observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"Change" object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification * _Nonnull note) {
    __strong __typeof(weakSelf)strongSelf = weakSelf;
    if (strongSelf) {
        [strongSelf reload];
    }
}
```
1. 在block之前定义对self的弱引用weakSelf,因为是弱引用，所以self被释放时weakSelf会变成nil
2. 在block中引用该弱引用，考虑到多线程情况，通过强引用strongSelf来引用该弱引用，如果self不为nil，就会retain self，以防在block内部使用过程中self被释放
3. 在block块中使用该强引用strongSelf，注意对strongSelf进行nil检测，因为多线程在弱引用weakSelf对强引用strongSelf赋值时，弱引用weakSelf可能已经为nil了
4. 强引用strongSelf在block作用域结束之后，自动释放

- weakSelf为什么需要strongSelf配合使用

在block块中先写一个strongSelf，是为了避免在block的执行过程中，突然出现self被释放的情况。

##### __weak与__block的区别

`__weak`可以避免循环引用，`__block`不能避免循环引用，`__block`的作用是提升变量的作用域


## 总结

- 引用计数是一种简单高效的管理对象生命周期的方法
- 引用计数无法避免循环引用
- 循环引用的解决分为弱引用（事先规避）和主动断开循环引用（事后补救）两个方案
- `__weak` 可以避免循环引用，但是其存在外部对象释放后，block内部也访问不到这个对象的问题，所以我们通过在block内部声明一个 `__strong`的变量来指向weakObj，使得外部对象既能在block内部保持，又能避免循环引用的问题
- `__block`无法避免循环引用的问题，它的作用是提升了变量的作用域，在block内外访问的都是同一个对象，如果想要在block中改变变量的值，就需要在变量声明的时候加上`__block`修饰符