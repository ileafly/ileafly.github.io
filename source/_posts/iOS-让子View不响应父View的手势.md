---
title: iOS--让子View不响应父View的手势
date: 2016-10-24 13:57:13
tags: iOS
categories: iOS
---

在项目中遇到一个问题，父View上添加的手势事件在子View中也会响应，通过搜索后找到了解决方案，接下来作一个小总结，方便以后遇到相同问题时查找。
解决思路：手势事件有一个代理方法，`- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch`，我们就是要通过这个代理方法屏蔽掉子View上的事件响应。

    - (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
    if ([touch.view isDescendantOfView:你想屏蔽掉手势的子视图]) {
     return NO;
     }
     return YES;
    } 

