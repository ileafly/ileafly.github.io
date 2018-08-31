title: runtime机制
tags:
  - iOS进阶
categories:
  - iOS
date: 2016-12-27 15:46:00
---
### runtime实现的机制是什么？
runtime是一套比较底层的纯C语言API，属于一个C语言库，包含了很多底层的C语言API。平时我们编写的OC代码在程序运行过程时，最终都是转换成了runtime的C语言
```
  OC: [[User alloc] init]
  runtime: objc_msgSend(objc_msgSend("User", "alloc"), "init")
```

### runtime能用来干什么？
1. 在程序运行过程中，动态的创建一个类
2. 在程序运行过程中，动态的为某个类添加属性、方法，修改属性值、方法
3. 遍历一个类的所有成员变量、所有方法

### runtime相关知识
1. 相关头文件
   ```
     <objc/runtime.h>
     <objc/message.h>
   ```
2. 相关函数
   ```
    objc_msgSend: 给对象发送消息
    class_copyMethodList: 获得某个类内部的所有方法
    class_copyIvarList: 获得某个类内部的所有成员变量
    class_getInstanceMethod: 获得某个实例方法
    class_getClassMethod: 获得某个类方法
    method_exchangeImplementations: 交换两个方法的具体实现
    objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy) 关联对象
   ```

### runtime用法介绍
1. 发送消息
   OC方法调用的本质就是发送消息，消息机制的原理就是对象根据方法编号SEL去映射表中查找对应的方法实现
   ```
   Person *person = [[Person alloc] init];
   [person eat];

   // 本质：让对象发送消息
   objc_msgSend(p, @selector(eat));

   [Person eat];

   // 本质：让类对象发送消息
   objc_msgSend([Person class], @selector(eat));
   ```
2. 交换方法
   ```
   + (void)load {
       // 交换吃和读这两个方法
       Method m1 = class_getInstanceMethod(self, @selector(eat));
       Method m2 = class_getInstanceMethod(self, @selector(read));
    
       method_exchangeImplementations(m1, m2);
    }

#pragma mark - Methods

    - (void)eat {
       NSLog(@"我非常喜欢美食");
    }

    - (void)read {
       NSLog(@"我非常喜欢名著");
    }

   ```
3. 动态添加方法
   ```
   // 动态给Person类中添加drink方法
      class_addMethod([person class], @selector(drink), (IMP)drinkWater, "v@:");
   
   // 调用drink方法         
      if ([person respondsToSelector:@selector(drink)]) {
            [person performSelector:@selector(drink)];
      }

   // 编写drinkWater的实现
   // 注意点：1、void 前面没有+、- 号 因为是C语言代码  2、必须有两个指定参数(id self, SEL _cmd)
      void drinkWater(id self, SEL _cmd) {
          NSLog(@"我喜欢喝饮料");
      }
   ```
4. 在方法上添加额外功能
   ```
   // 以UIButton为类，扩展点击事件

   @implementation UIButton (Hook)

   + (void)load {
       static dispatch_once_t onceToken;
       dispatch_once(&onceToken, ^{
          Class selfClass = [self class];

          SEL oriSEL = @selector(sendAction:to:forEvent:);
          Method oriMethod = class_getInstanceMethod(selfClass, oriSEL);

          SEL cusSEL = @selector(mySendAction:to:forEvent:);
          Method cusMethod = class_getInstanceMethod(selfClass, cusSEL);

          BOOL addSucc = class_addMethod(selfClass, oriSEL, method_getImplementation(cusMethod), method_getTypeEncoding(cusMethod));

          if (addSucc) {
             class_replaceMethod(selfClass, cusSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
          } else {             
            method_exchangeImplementations(oriMethod, cusMethod);
          }
       })
   }

   - (void)mySendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event {
     // 自定义事件
     [self doSomething];

     [self mySendAction:action to:target forEvent:event];
   }
   ```

5. 关联
   ```
    void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
    // 参数说明：
    // object: 与谁关联
    // key: 唯一键
    // value: 关联所设置的值
    // policy: 内存管理策略

    id objc_getAssociatedObject(id object, const void *key)
    // 参数说明：
    // object: 与谁关联
    // key: 唯一键

    void objec_remoteAssociatedObjects(id object)
    // 参数说明：
    // object: 与谁关联
   ```