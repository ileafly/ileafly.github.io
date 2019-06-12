
---

title: 弄透Block

date: 2019-06-11 10:18:48 +0800

categories: iOS

tags: [iOS]

---


<a name="CYyta"></a>
# 目标

- [x] Block是什么？
- [x] 总结Block的使用场景
- [x] 为什么Block属性需要用copy修饰？
- [x] __block修饰后为何就可以修改？
- [x] Block循环引用是怎么产生的？


<a name="d9d97201"></a>
# Block是什么？

> Block：带有自动变量的匿名函数，它是C语言的拓展功能，之所以是扩展，是因为C语言不允许存在这样的匿名函数


1. 匿名函数

匿名函数是指不带函数名称的函数

2. 带有自动变量

这是因为Block拥有捕获外部变量的功能，在Block中访问一个外部的局部变量，Block会持有它的临时状态，自动捕获变量值，外部局部变量的变化不会影响它的状态

```objectivec
int val = 10;
void (^blk)(void) = ^{
  printf("val=%d\n", val);
};
val = 2;
blk(); // 这里输出的值是10，而不是2，因为block在实现时就会对它所在方法中定义的栈变量进行一次只读拷贝
```

3. 为了解决block不能修改自动变量的值，可以使用 `__block` 修饰

```objectivec
__block int val = 10;
void (^blk)(void) = ^{
 printf("val=%d\n", val);
};
val = 2;
blk(); // 这里输出的值是2
```


<a name="e3d6ac67"></a>
# Block的使用场景

1. 声明Block属性 利用Block属性响应事件或传递数据

> UIViewController 需要监听TableView中Cell的某个按钮的点击事件，既可以通过Delegate回调，也可以利用Block回调
> Block回调的思路：
> 声明一个Block属性，注意这里要用copy。
> 利用Block属性进行回调


2. 方法参数为Block 利用Block实现回调

> 以 `[UIView animateWithDuration:animations:]` 为例，animations是一个block对象，利用block实现调用者与UIView之间的数据传递


3. 链式语法

> 链式编程思想：将block作为方法的返回值，且返回值的类型为调用者本身，并将该方法以setter的形式返回，从而实现连续调用


```objectivec
//  CaculateMaker.h
//  ChainBlockTestApp

#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

@interface CaculateMaker : NSObject

@property (nonatomic, assign) CGFloat result;

/*
* 返回类型 CaculateMaker
* 传入参数 CGFloat num
*/
- (CaculateMaker *(^)(CGFloat num))add;

@end


//  CaculateMaker.m
//  ChainBlockTestApp


#import "CaculateMaker.h"

@implementation CaculateMaker

- (CaculateMaker *(^)(CGFloat num))add;{
    return ^CaculateMaker *(CGFloat num){
        _result += num;
        return self;
    };
}

@end

// 使用
CaculateMaker *maker = [[CaculateMaker alloc] init];
maker.add(20).add(30);
```


<a name="fd60c885"></a>
# 为什么Block属性需要用copy修饰？

**因为在MRC情况下如果Block属性不使用copy修饰，在使用中会出现崩溃，在ARC情况下，Block属性使用strong修饰会被默认进行copy，所以ARC情况下，Block属性可以使用strong或copy修饰，不然会出现崩溃。**<br />**<br />为何会有这种现象出现？

Block在内存中的位置分为三种类型：

- **NSGlobalBlock** 是位于全局区的block，它是设置在程序的数据区中。
- **NSStackBlock** 是位于栈区，当超出变量作用域时，栈上的Block以及__block变量都会被销毁。
- **NSMallocBlock** 是位于堆区，在变量作用域结束时不受影响。

这三种类型对应以下三种情况：

1. Block中没有截获自动变量时Block类型是**NSGlobalBlock**
1. Block中截获自动变量时Block类型是**NSStackBlock**
1. 堆中的Block无法直接创建，当对**NSStackBlock**类型的Block进行copy时，会将Block放到堆中，Block类型变为**NSMallocBlock**

当Block类型是**NSStackBlock**时，一旦超出了变量作用域，栈上的Block以及__block变量就会被销毁，从而导致调用Block回调时崩溃。因此，Block属性需要用copy修饰来避免这种情况。

```objectivec
- (void)click:(id)sender {
        TestClass *test = [[TestClass alloc] init];
        
        __block int a = 1;
        // 弱引用，block类型是__NSStackBlock__  当TestClass执行回调时必崩 EXC_BAD_ACCESS
        test.weakBlock = ^() {
            NSLog(@"ok");
            a = 2;
        };
        
        // block类型是__NSStackBlock__  当TestClass执行回调时必崩 EXC_BAD_ACCESS
        test.assignBlock = ^() {
            NSLog(@"ok");
            a = 3;
        };
        
        // block类型是__MallocBlock__ 正常执行
        test.copyBlock = ^() {
            NSLog(@"ok");
            a = 4;
        };
        
        // block类型是__MallocBlock__ 正常执行
        test.strongBlock = ^() {
            NSLog(@"ok");
            a = 5;
        };
        
        NSLog(@"copy property: %@", test.copyBlock);
        NSLog(@"assign property: %@", test.assignBlock);
        NSLog(@"weak property: %@", test.weakBlock);
        NSLog(@"strong property: %@", test.strongBlock);
        
        
        [test start];
    }
```


<a name="7a61fc63"></a>
# __block修饰后为何就可以修改局部变量？

首先，需要弄明白一个概念，static声明的静态局部变量是可以在block内进行修改的，为何会有这种区别呢？<br />因为，静态局部变量存在于应用程序的整个生命周期，而非静态局部变量仅存在于一个局部上下文中，绝大多数情况下，block都是延后执行的，这就有可能出现非静态局部变量被回收的情况，为了避免这个问题，苹果在设计block时，block中非静态局部变量是值传递，这也解释了为何block会持有变量的临时状态，后续再修改，block中的变量值也不再改变。

加上__block修饰的局部变量，被block捕获时，就不再是传递局部变量的值了，而是变成了一个结构体实例。比如：<br />定义一个 `__block int a = 10;`  会变成 `__Block_byref_a_0 *a;` <br />`__Block_byref_a_0` 的结构体如下所示：

```objectivec
struct __Block_byref_a_0 {
 void *__isa;
 __Block_byref_a_0 *__forwarding; // forwarding指针
 int __flags;
 int __size;
 void (*__Block_byref_id_object_copy)(void*, void*);
 void (*__Block_byref_id_object_dispose)(void*);
 int a; // 原变量同类型变量
};
```

结构体中有一个**forwarding指针，此指针指向转换后变量本身，结构体中也有一个原变量一样类型的变量。此后代码中涉及到原变量的地方，都会转换成新变量->forwarding->原变量同类型变量**

如果在block中直接修改变量的值，实质的过程是新变量->__forwarding->原变量同类型变量，最终修改的其实是结构体中原变量同类型变量，很明显这个结构体内的变量已经不属于block的外部变量了，所以能在block内修改。

这个新变量也是非静态局部变量，所以如果没有copy，block执行时，新变量有可能已经被栈回收

总结：<br />**block修饰的变量转换成了结构体，结构体内有一个forwarding指针和一个与原变量相同类型的成员变量，forwarding指针指向结构体内的成员变量。无论在block内外，都是通过forwarding来访问的。**


<a name="0eccb7a1"></a>
# Block的循环引用是怎么产生的？

我们先看一段block导致循环引用的代码：

```objectivec
TestClass *test = [[TestClass alloc] init]; 

test.copyBlock = ^() {
    NSLog(@"ok: %d", test.result);
};
```

当我们写完这段代码后，Xcode就会提醒我们，这段代码存在循环引用，事实上也确实存在循环引用。接下来我们就来分析一下为什么会产生循环引用。<br />test的属性block强引用了SecondViewController中的block，SecondViewController中的block又强引用了test的属性result，从而导致了循环引用。

并非所有的block都存在循环引用，下面列举一些常见的block使用的示例：

```objectivec
// self-->requestModel-->block-->self 
[self.requestModel requestData:^(NSData *data) {
    self.name = @"leafly";
}];

// 虽然存在引用环，但是通过主动释放requestModel打破了循环
[self.requestModel requestData:^(NSData *data) {
    self.name = @"leafly";
    self.requestModel = nil;
}];

// t-->block-->self 不存在循环引用
Test *t = [[Test alloc] init];

[t requestData:^(NSData *data) {
    self.name = @"leafly";
}];

// AFNetworking-->block-->self 不存在循环引用
[AFNetworking requestData:^(NSData *data) {
    self.name = @"lealfy";
}];
```


