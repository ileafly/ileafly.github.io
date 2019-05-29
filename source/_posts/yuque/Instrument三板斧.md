
title: Instrument三板斧
date: 2018-11-21 11:55:00 +0800
tags: [iOS]
categories: 开发技巧
---
tags: [iOS]<br />categories: 开发技巧<br />date: 2018-11-21 11:55:00

---

作为开发，不仅仅要会开发功能，还需要关注产品的性能，苹果为我们提供了强大的性能检测功能Instrument。这里总结一下Instrument的三个常用功能。

<a name="u7avsg"></a>
#### [](#u7avsg)CPU性能
Instrument有一个**Timer Profiler**选项，选择**Timer Profiler**模块运行你想要检测的应用即可得到它的CPU性能的结果：

![](https://cdn.nlark.com/yuque/0/2019/png/183307/1548062488952-32b82364-c834-417d-be0e-5c6c065f6e53.png#width=747)

- 可以在时间轴里选择一段时间查看该段时间更为细节的CPU性能

- 可以在Call Tree里选择调用栈的展现形式，更加方便问题定位

- 如果CPU过高会导致手机耗电甚至发热，列表滚动中CPU过高也有可能导致掉帧


<a name="bysosy"></a>
#### [](#bysosy)图形性能
图形性能方面最关注的应该就是「帧率」了，也就是我们常说的FPS。在Instrument中使用**Core Animation** + **Time Profiler**来评估图形性能。<br />除了观察时间轴上的FPS情况，也可以在控制台中选择Core Animation FPS Measurements来观察FPS的情况，在屏幕滑动时，FPS越高表示性能越好，帧率过低则意味着屏幕可能会出现卡顿。<br />![](https://cdn.nlark.com/yuque/0/2019/png/183307/1548063371611-be58f216-99e0-49fa-9920-e631f3771c5a.png#width=747)

除了FPS还有很多选项可以检测图形性能，在Xcode 9之前这些选项在右下角的区域（如上图），Xcode 9之后这些选择在Xcode的Debug菜单中<br />![](https://cdn.nlark.com/yuque/0/2019/png/183307/1548064256231-96434508-1f01-4d2b-b52e-cbfba03a1ddf.png#width=747)

下面着重介绍一下这些选项的作用：

- Color Blended Layers 这个选项基于渲染程度对屏幕中的混合区域进行绿到红的高亮显示，越红表示性能越差，会对帧率等指标造成较大的影响。红色通常是由于多个半透明的图层叠加引起

- Color Hits Green and Misses Red 当开启栅格化时，耗时的图片绘制会被缓存，当做一个简单的扁平图片来呈现，如果页面的其他区块使用缓存直接命中，就显示绿色，反之就显示红色。红色越多，性能越差，因为栅格化生成缓存的过程是有开销的，如果缓存能被大量命中和有效使用，则总体会降低开销，反之则意味着频繁生成新的缓存，这会让性能问题雪上加霜

- Color Copied Images 对于GPU不支持的色彩格式的图片只能由CPU处理，这样的图片会标为蓝色，蓝色越多性能越差

- Color Immediately 通常以每毫秒10次的频率更新图层调试颜色，对某些效果来说可能太慢了，这个选项可以用来设置每帧都更新

- Color Misaligned Images 这个选项用来检查图片是否被缩放以及像素是否对齐，被缩放的图片会被标记为黄色，像素不对齐则会标注为紫色。黄色、紫色越多，性能越差

- Color Offscreen-Rendered Yellow 这个选项会把离屏渲染的图层显示为黄色，黄色越多，性能越差，这些显示为黄色的图层很可能需要用 shadowPath 或者 shouldRasterize来优化

- Color OpenGL Fast Path Blue 这个选项会把任何直接使用OpenGL绘制的图层显示为蓝色。蓝色越多，性能越好

- Flash Update Regions 这个选项会把重绘的内容显示为黄色，不该出现的黄色越多，性能越差。


<a name="2ekfyu"></a>
#### [](#2ekfyu)内存性能

- 全局内存使用情况：从全局的角度监测应用的内存使用情况，捕捉非预期的或大幅度的内存增长

- 内存泄漏：未被程序引用，同时也不能被使用或释放的内存

- 废弃内存：被程序引用，但是没有任何作用的内存

- 僵尸对象：对应的内存已经被释放并且不再会使用到，但是程序依然指向它的引用


![](https://cdn.nlark.com/yuque/0/2019/png/183307/1548068035204-9b36b29a-e575-4149-a2db-f9500e7fbf32.png#width=747)
<a name="uxuyny"></a>
#### [](#uxuyny)注意点

- 如果使用Instrument查看调用栈时看到的都是地址而不是函数名，这就需要在项目的Build Settings - Debug Information Format的Debug 和 Release里设置为**DWAF with dSYM File**，这样才能将对应的堆栈信息符号化显示



