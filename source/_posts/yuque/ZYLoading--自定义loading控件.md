
title: ZYLoading--自定义loading控件
date: 2017-12-14 19:20:00 +0800
tags: [iOS]
categories: 组件
---
date: 2017-12-14 19:20:00<br />tags: [iOS]<br />categories: 组件

---

移动端项目的开发离不开loading控件，通常为了能快速在项目中实现loading效果我们有几个主流的开源库可以选择：<br />[MBProgressHUD](https://github.com/jdg/MBProgressHUD)、[SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)等

然后，为了能让整体项目的loading效果显得更加贴切我就想创建一个loading控件，希望此控件能够比较方便的开启、停止loading效果，而且能易于集成和更换logo。

为了达到这个目的，我创建了一个名为[ZYLoading](https://github.com/luzhiyongGit/ZYLoading.git)的控件，下面就为大家分享一下我这个控件的原理以及使用方法。

<a name="6gngga"></a>
## [](#6gngga)原理分析

此控件的核心思想是利用runtime机制给分类增加成员属性，通过给UIView扩展开启、停止loading的方法，从而实现任何UIView的实例都能方便的开启、停止loading动画

```
#import "UIView+ZYLoadingView.h"

#import <objc/runtime.h>

static char LoadingViewKey;

@implementation UIView (ZYLoadingView)

#pragma mark - Setter

// 将创建的ZYLoadingView实例关联到分类
- (void)setLoadingView:(ZYLoadingView *)loadingView {
    [self willChangeValueForKey:@"LoadingViewKey"];
    objc_setAssociatedObject(self, &LoadingViewKey, loadingView, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    [self didChangeValueForKey:@"LoadingViewKey"];
}

// 获取关联的ZYLoadingView
- (ZYLoadingView *)loadingView {
    return objc_getAssociatedObject(self, &LoadingViewKey);
}

// 开启动画
- (void)beginLoading {
    if (!self.loadingView) {
        self.loadingView = [[ZYLoadingView alloc] initWithFrame:self.bounds];
    }
    
    [self addSubview:self.loadingView];
    
    [self.loadingView startAnimation];
}

// 停止动画
- (void)endLoading {
    if (self.loadingView) {
        [self.loadingView stopAnimation];
    }
}

@end
```

<a name="onp8ux"></a>
## [](#onp8ux)使用方法

<a name="yz1gkc"></a>
##### [](#yz1gkc)通过一组图片组合成动画

```objectivec
// 通过枚举选择图片组合动画
ZYLoadingConfigInstance.loadingType = ZYLoadingAnimateImages;
// 图片名称
ZYLoadingConfigInstance.animateImageName = @"zy_loading_";
// 图片尺寸
ZYLoadingConfigInstance.loopImageSize = CGSizeMake(37, 13);
// 动画过渡时长
ZYLoadingConfigInstance.duration = 1.f;
```

<a name="qckoog"></a>
##### [](#qckoog)通过一张图旋转形成动画

```objectivec
// 通过枚举选择通过旋转图片展现loading动画
ZYLoadingConfigInstance.loadingType = ZYLoadingLoopImage;
// 图片名称
ZYLoadingConfigInstance.loopImage = [UIImage imageNamed:@"loading_circle"];
// 图片尺寸    
ZYLoadingConfigInstance.loopImageSize = CGSizeMake(60, 60);
// 动画过渡时长    
ZYLoadingConfigInstance.duration = 0.25f;
```

<a name="1htvcz"></a>
##### [](#1htvcz)通过一张图片旋转，另一张图片渐隐渐显组合成动画

```objectivec
// 通过枚举选择通过旋转图片展现loading动画    ZYLoadingConfigInstance.loadingType = ZYLoadingLoopImage;
// 图片名称
ZYLoadingConfigInstance.loopImage = [UIImage imageNamed:@"loading_circle"];
// 图片尺寸    
ZYLoadingConfigInstance.loopImageSize = CGSizeMake(60, 60);
// logo图片名称
ZYLoadingConfigInstance.logoImage = [UIImage imageNamed:@"loading_zhangyu"];
// logo图片尺寸
ZYLoadingConfigInstance.logoImageSize = CGSizeMake(40, 40);
// 动画过渡时长
ZYLoadingConfigInstance.duration = 0.25f;
```

<a name="1v21fg"></a>
##### [](#1v21fg)开启、停止动画

```objectivec
// 开启动画
[self.view beginLoading];

// 停止动画
[self.view endLoading];
```

你也可以直接参考github上的[ZYLoading](https://github.com/luzhiyongGit/ZYLoading.git)

