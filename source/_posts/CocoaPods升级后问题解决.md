title: CocoaPods升级后问题解决
tags:
  - 解决方案
categories:
  - iOS
date: 2016-10-09 17:30:00
---

升级[CocoaPods](https://cocoapods.org/)到`1.0.1`版本遇到了两个问题，其中第一个比较简单，由于新版本强制要求设置target否则会出现错误，所以我们只需要在`Podfile`里按照新规范给每个Target都设置上就行了。而第二个问题让我寻找了很久，先来描述一下第二个问题。

升级完CocoaPods后，在iPhone 6模拟器中编译没有任何问题，但是在iPhone 4或者iPhone 5等模拟器中一直编译失败，通过调研找到了原因，因为iPhone 4或者iPhone 5模拟器是64位模拟器，也就是说在64位模拟器上编译不会通过，这是因为新版本的CocoaPods
