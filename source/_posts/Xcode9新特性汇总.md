title: Xcode9新特性汇总
tags:
  - iOS基础
  - ''
categories:
  - iOS
date: 2017-12-14 19:19:00
---
新版的Xcode 9正式发布了，今天我也将Xcode进行了升级。这次的Xcode更新给我们带来了不少的新特性，这里我进行一个简单的汇总。

## Main Thread Checker

Xcode 9现在会自动检测UI操作是否在主线程了，一旦代码运行到在非主线程操作UI时就会警告提示，相关代码会高亮，特别方便定位

![Xcode9非主线程操作UI2.png](http://upload-images.jianshu.io/upload_images/1479547-ae5b514cae24218c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Xcode9非主线程操作UI.png](http://upload-images.jianshu.io/upload_images/1479547-667852563144f730.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以前遇到这种在非主线程操作UI的情况，问题定位相对比较难，需要排查才能找到，现在Xcode 9 给我们提供了非常方便的支持

## Swift Language Version 支持 4.0和3.2

Xcode 9使用Swift 4编译器，同时支持切换到Swift 3.2，开发者可以根据项目需要选择Swift语言版本

![Xcode9swift语言选择.png](http://upload-images.jianshu.io/upload_images/1479547-22e7d4510d174243.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 项目文件路径与本地文件路径自动保持统一

在以前，为了保证项目文件路径和文件系统中的路径保持一致，我们通常是先在本地路径创建文件夹，然后添加到项目中，现在Xcode 9 将项目文件和文件系统进行了统一，我们可以直接在项目中创建文件或者直接拖拽改变文件位置时，也会相应的改变此文件在文件系统中的位置

## Refactor功能改进

`Refactor`是Xcode一直都提供的一个功能，不过在Xcode 9中进行了优化，Xcode 9将相关代码直接铺在代码编辑器里面，只要滚动编辑框，就可以轻松的看到即将影响到的所有地方，非常的清晰

![xcode9refactor.png](http://upload-images.jianshu.io/upload_images/1479547-32674d7c60b80bf5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Named Color

Xcode 9支持在xcassets里添加颜色，这样就可以直接在代码或Storyboard里引用这个颜色了，这就非常有利于项目主题颜色的更换

![named-colors.png](http://upload-images.jianshu.io/upload_images/1479547-4fcc16fe299d6ba4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 模拟器升级

Xcode 9模拟器又改回了之前的拟物化，而且现在支持多个模拟器同时运行，这就对多屏调试方便了很多

![xcode9simulator.png](http://upload-images.jianshu.io/upload_images/1479547-b9e5bf3b94af0fd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 全新的构建系统

Xcode 9提供了一个全新的构建系统，这次的构建系统完全使用Swift语言写成，基于Apple的[llbuild]( https://github.com/apple/swift-llbuild )引擎，新的构建系统默认是不开启的，我们可以通过File -> Project Settings 或 File -> Workspace Settings 来切换构建系统

![xcode9build.png](http://upload-images.jianshu.io/upload_images/1479547-58d40d4c79ce3d68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 深度集成Github

Xcode 9针对Github做了定制化的集成，在Xcode的Preference -> Account 可以添加github账号，这样就能看到完整的项目记录和分支情况

![xcode9github.png](http://upload-images.jianshu.io/upload_images/1479547-7cab79d2f8ebe53d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

