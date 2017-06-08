---
title: 深入浅出Texture--高性能界面的解决方案
date: 2017-06-01 19:08:54
tags: iOS进阶
categories: iOS
---

[Texture](https://github.com/texturegroup/texture/)是由Facebook开源的[AsyncDisplayKit](https://github.com/facebookarchive/AsyncDisplayKit)演变而来

### 特性
Texture的基础单元是一个个node，ASDisplayNode是基于CALayer的一层抽象。不同于UIView对象，ASDisplayNode是线程安全的，它不仅仅可以用于主线程而且可以在辅线程中使用。

使用Texture可以让我们在辅线程中进行图片的解析、文本的适应、约束等等UI操作，从而能够保证主线程及时对用户的操作做出反应

### 常用类
- `ASViewController`: 基于`UIViewController`封装的子类，能够方便的为我们提供node的管理
- `ASNavigationController`: 可用于替代`UINavigationController`，遵循`ASVisibility`协议
- `ASTabBarController`: 可用于替代`UITabBarController`，遵循`ASVisiblity`协议
- `ASCollectionNode`、`ASTableNode`:等同于`UICollectionView`、`UITableView`
- `ASPagerNode`:基于`ASCollectionNode`封装的子类，能够像`UIPageViewController`一样给我们提供强大的滑动操作体验
- `ASCellNode`:等同于`UITableViewCell`、`UICollectionViewCell`，需要强调的是它返回的是一个Block(ASCellNodeBlock)。
- `ASScrollNode`: 代替`UIScrollView`
- `ASEditableTextNode`: 代替`UITextView`
- `ASTextNode`: 代替`UILabel`
- `ASImageNode`: 代替`UIImage`
- `ASNetworkImageNode`: 可以自动加载图片并进行内存管理，而且还支持逐步加载Jpeg和动态gif图片
- `ASMultiplexImageNode`:
- `ASVideoNode`: 代替`AVPlayerLayer`
- `ASVideoPlayerNode`: 代替`UIMoviePlayer`
- `ASControlNode`: 代替`UIControl`
- `ASButtonNode`: 代替`UIButton`
- `ASMapNode`: 代替`MKMapView`
- `ASStackLayoutSpec`: 堆放布局规则
- `ASOverlayLayoutSpec`: 覆盖布局规则
- `ASRelativeLayoutSpec`: 相对布局规则
- `ASInsetLayoutSpec`: 插入布局规则
- `ASBackgroundLayoutSpec`: 背景布局规则
- `ASCenterLayoutSpec`: 中心布局规则
- `ASAbsoluteLayoutSpec`: 绝对布局规则
- `ASRatioLayoutSpec`: 比例布局规则


### 层级结构

```
graph TB
ASDisplayNode-->ASTableNode
ASDisplayNode-->ASCollectionNode
ASDisplayNode-->ASCellNode
ASDisplayNode-->ASScrollNode
ASDisplayNode-->ASEditableTextNode
ASDisplayNode-->ASControlNode
ASCollectionNode-->ASPagerNode
ASControlNode-->ASButtonNode
ASControlNode-->ASTextNode
ASControlNode-->ASMapNode
ASControlNode-->ASImageNode
ASImageNode-->ASNetworkImageNode
ASImageNode-->ASMultiplexImageNode
ASNetworkImageNode-->ASVideoNode
```

### 批抓取进行无限滚动

```
// 将leadingScreensForBatching 设置为1.0表示当滚动到还剩下一个全屏时就开始抓取新的一批数据
self.tableNode.view.leadingScreensForBatching = 1.0; 
```

```
- (BOOL)shouldBatchFetchForTableNode:(ASTableNode *)tableNode {
    return YES;
}
```

### ASDK的容器

当在项目中替换使用AsyncDisplayKit的时候，应该把nodes节点，当做一个子节点添加到一个容器类里，这些容器类负责告诉所包含的节点，他们现在是什么状态，以便尽可能有效的加载数据和渲染，经常犯的一个错误是把node节点直接添加到一个现有的view中，这样做会导致节点在渲染的时候会闪烁一下。

Texture为我们提供了以下这些节点容器：
- `ASCollectionNode`
- `ASPagerNode`
- `ASTableNode`
- `ASViewController`
- `ASNavigationController`
- `ASTabBarController`

#### 使用节点容器的好处
节点容器能够自动为它的节点们管理高效的预加载，这就意味着节点容器中所有的节点的尺寸约束、数据请求、图片解析、渲染都将是异步进行

### 核心机制

Texture模拟了Core Animation的机制，所有针对ASNode的修改和提交，总有些任务是必需放在主线程执行的，当这种任务出现的时候，ASNode就会把任务用`ASAsyncTransaction(Group)`封装并提交到一个全局的容器去。Texture在RunLoop中注册了一个Observer，监视的事件和CA一样，但是优先级比CA低。当RunLoop进入休眠前，CA处理完事件后，ASDK就会执行该loop内提交的所有任务

利用这个机制，Texture可以在合适的机会把异步、并发的操作同步到主线程去，并且获得不错的性能

### 用法

1. 利用ASTableNode替代UITableView创建一个高性能的列表页面

2. 利用ASLayoutSpec替代AutoLayout创建高效的页面约束


### 疑问点
- 为何使用`ASCellNode`不需要担心cell重用问题？

### 参考地址

[AsyncDisplaykit2.0 使用](http://www.bijishequ.com/detail/227995?p=)--主要讲解了LayoutSpec相关的知识


