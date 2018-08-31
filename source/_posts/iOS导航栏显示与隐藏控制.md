title: iOS导航栏显示与隐藏控制
tags:
  - iOS基础
  - ''
categories:
  - iOS
date: 2018-08-22 13:20:00
---

最近打算整理优化一下项目中对导航栏的显示与隐藏控制

## 早期做法

由于前期项目中需要隐藏导航栏的页面不多，早期采用的方案也比较简单，遵循谁要隐藏自己处理的原则，在需要隐藏的页面的`viewWillAppear`执行`[self.navigationController setNavigationBarHidden:YES animated:YES]`，`viewWillDisappear`执行`[self.navigationController setNavigationBarHidden:NO animated:YES]`，前期满足了需求，但是随着需要隐藏的页面越来越多，问题也逐渐暴露

- 需要在`viewWillAppear`和`viewWillDisappear`控制导航栏的地方越来越多，不便于管理和维护
- 容易出现人为疏忽，导致导航栏未按预期展现
- 切换tabBar的情况下，导航栏有一个向上消失的动画

## 升级做法

鉴于早期方案存在的问题，我打算整理一下导航栏的实现方案

我考虑了两个方案

- 放弃使用原生navigationBar，改用自定义view，灵活控制导航栏的展现
- 优化现有导航栏显示隐藏方案

由于项目已经比较大，现在改自定义view成本太高，所以我打算先从现有方案的优化开始

### 优化方案

基于早期的`NavigationController`基类进行扩展

```
viewDidLoad() {
    super.viewDidLoad()
    
    self.delegate = self
}

extension ZYNavigationController: UINavigationControllerDelegate {

    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        // 判断要显示的控制器是否是自己
        let isHidden = viewController.isKind(of: FourthViewController.self) || viewController.isKind(of: FirstViewController.self)
        
        self.setNavigationBarHidden(isHidden, animated: true)
    }
}
```
遵循`UINavigationControllerDelegate`，在`willShow viewController`方法内判断当前viewController是否需要隐藏导航栏，并根据判断结果设置。
这样就能达到，controller在被加载到navigationController内，将要展现前，判断并控制导航栏的显示或隐藏。
这里还有一个问题需要解决，那就是导航栏被隐藏后，手势返回功能就会失效，这里就需要我们支持，可以通过`interactivePopGestureRecognizer`来启动手势返回

```
viewDidLoad() {
    super.viewDidLoad()
    
    self.delegate = self
}

extension ZYNavigationController: UINavigationControllerDelegate {

    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        // 判断要显示的控制器是否是自己
        let isHidden = viewController.isKind(of: FourthViewController.self) || viewController.isKind(of: FirstViewController.self)
        
        self.setNavigationBarHidden(isHidden, animated: true)
        
        if isHidden {
            self.interactivePopGestureRecognizer?.delegate = self
            self.interactivePopGestureRecognizer?.isEnabled = true
        }
    }
}
```