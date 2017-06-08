---
title: iOS 11 开发者需要知道的改变
date: 2017-06-08 17:18:08
tags: iOS11
categories: iOS
---

苹果在`WWDC 2017`披露了许多的iOS 11的新特性，作为iOS开发者有哪些改变是我们需要知道的呢？

## iOS 11的发布时间
在正式发布iOS 11之前，苹果通常会先提供一个可供开发者安装的版本，然后再进过几个版本公开发布的beta版，最终才会发布一个正式版本的iOS 11系统
- 开发者beta版本： 已经发布
- 公开发布的beta版本：六月底
- 最终正式版本：秋季

## iOS 11支持哪些设备
- iPhone 5S, 6, 6 Plus, 6S Plus, SE, 7, 7 Plus
- iPad Air and Air 2, iPad Mini 2, 3, 4, 5代 iPad，所有的 iPad Pro
- 6代 iPod Touch

## iOS 新增了哪些框架

### Core ML

iOS 11中苹果新增了一个机器学习框架[Core ML](https://developer.apple.com/machine-learning/)，同时Apple也提供了[一系列的工具](https://developer.apple.com/documentation/coreml/converting_trained_models_to_core_ml)用来将各类机器学习的模型转换为Core ML可以理解的形式，从而帮助开发者轻松的在APP里使用前人训练出来的模型。

### ARKit

[ARKit](https://developer.apple.com/documentation/arkit)帮助开发者更容易的在项目中使用AR功能，扩展了应用和游戏的应用场景，三年前我在开发带有AR功能的App时，相关的核心功能还需要像高通购买SDK才能实现。现在有了ARKit普通开发者也能够开发AR相关的功能了

## 其他值得注意的变更

1. 在iOS 11系统中，在视频播放界面调节音量时将不会出现音量提示框，一个比之前更小的音量滑动条将会出现在屏幕的右上角
![](https://resource.feng.com/resource/h061/h84/img201706072211540.jpg)
2. 由于AppStore的改版，原先在应用内设置的跳转评分的链接将不会正常加载到评分界面
3. 用户能够在AppStore中的产品页面里直接进行应用内购买，App必须支持新的`SKPaymentTransactionObserver`方法来支持AppStore内的应用内购买功能
4. 开发者能够在后台回复用户的评论了，再也不用愁无法跟AppStore里的用户进行沟通了
5. 由于我们在iTunes Connect后台只能提供一套产品信息，类如应用名称、图标、截图等，这就要求我们考虑到不同版本AppStore界面间的兼容
6. 新的Navigation title设计，iOS 11系统中大多app都采用了这个新的设计，放大了导航栏的标题字体。虽然个人感觉有点丑，但是如果想要采用这项设计的话，只需要设置`navigation bar`的`prefersLargeTitles`即可，当然前提是你的导航栏是用的原生的`navigation bar`
7. 提供了[FileProvider](https://developer.apple.com/documentation/fileprovider)功能让App可以获取用户设备或云端上的文件
8. iOS 11将不再支持32位的app，如果想要让程序运行在iOS 11设备上，进行64位的重新编译是必须步骤

## 参考

[开发者所需要知道的 iOS 11 SDK 新特性](https://onevcat.com/2017/06/ios-11-sdk/)

[AppStore官方说明](https://developer.apple.com/app-store/product-page/)