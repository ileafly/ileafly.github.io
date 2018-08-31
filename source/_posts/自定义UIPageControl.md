title: 自定义UIPageControl
tags:
  - iOS进阶
  - ''
categories:
  - iOS
date: 2017-04-11 11:46:00
---

近期项目中需要给UIPageControl自定义图片，为了方便使用也为了方便大家，建了一个轮子[ZYPageControl](https://github.com/luzhiyongGit/ZYPageControl)

### 实现思路
1. 继承至UIView创建一个ZYPageControl
2. 为ZYPageControl类添加`currentPage`、`numberOfPages`等PageControl控件常用的属性
3. 添加设置图片的方法
4. 在`- (void)layoutSubviews`方法里 绘制pageControl

### 支持功能
- 设置PageControl的默认图和高亮图
- 灵活设置PageControl间距
- 灵活设置PageControl的大小
- 灵活设置PageControl的位置

### 使用方法
#### 手动集成
1. 下载ZYPageControl的代码
2. 将ZYPageControl类添加到项目中
3. 与UIPageControl用法一样，创建一个ZYPageControl对象并添加到View上
4. 设置你想要的图片作为默认图
5. 设置你想要的PageControl间距

#### Pod集成
等待发布支持~



