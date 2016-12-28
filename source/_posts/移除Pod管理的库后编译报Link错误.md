---
title: 移除Pod管理的库后编译报Link错误
date: 2016-10-24 13:59:00
tags: CocoaPods
categries: iOS
---

在将项目中用pod管理的一个库移除后编译居然报错，查看了一下报错信息，编译时还会link被删除的那个库，但是无法link成功，不论是重新执行`pod install`还是清除缓存，clean项目，都无法解决这个问题。
这是因为虽然我们将库已经从pod管理中删除，但是我们的项目还在link这个库，我们需要去除掉这个link，具体移除步骤如下：

1. 点击项目工程文件进入Build Setting
2. 找到Other Linker Flags一栏
3. 删除到相关库的link
4. 重新编译即可

