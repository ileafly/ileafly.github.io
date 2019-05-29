
title: iOS 技巧合集
date: 2017-02-20 11:55:00 +0800
tags: [iOS]
categories: 开发技巧
---
tags: [iOS]<br />categories: 开发技巧<br />date: 2017-02-20 11:55:00

---

<a name="tf2kgf"></a>
#### [](#tf2kgf)一行命令，统计OC项目中每个源代码文件的行数以及总行数
```powershell
$ find . "(" -name "*.m" -or -name "*.mm" -or -name "*.cpp" -or -name "*.h" -or -name "*.rss" ")" -print | xargs wc -l
```

<a name="8wricr"></a>
#### [](#8wricr)复制字符串到剪贴板
```objectivec
UIPasteboard *pasteboard = [UIPasteboard  generalPasteboard];
pasteborad.string = @"你需要复制的字符串";
```

<a name="s16fhl"></a>
#### [](#s16fhl)让子view不响应父view的手势
正常情况下，父view的手势在子view上点击也会响应，想要在子view中屏蔽父view的手势有以下方法：

1. 子view中再次添加手势，拦截掉父view的手势

2. 实现父view的手势代理，在代理中拦截父view手势

```objectivec
- (BOOL)gestureRecognizer:(UIGestureRecognizer  *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
    if ([touch.view  isDescendantOfView:subview]) {
       return NO;
    }
    return YES;
}
```

<a name="nognoh"></a>
#### [](#nognoh)自定义xopen快捷打开工程
利用ruby脚本在终端快速打开工程，ruby脚本内容如下：
```ruby
#!/usr/bin/env ruby
require 'shellwords'

proj = Dir['*.xcworkspace'].first
proj = Dir['*.xcodeproj'].first unless proj

if proj
   puts "Opening  #{proj}"
   `open #{proj}`
else
  puts "No  xcworkspace|xcproj  file  found"
end
```
将脚本文件保存为`xopen`，拷贝到`/usr/local/bin`目录下，添加权限：`chmod 777 xopen`<br />在终端中任何需要打开项目的位置，执行xopen即可。

<a name="fqw2ll"></a>
#### [](#fqw2ll)CUICatalog Invalid asset name supplied
项目中最近在打印`CUICatalog Invalid asset name supplied`，调研发现之所以输出这个log是因为传了一个空字符串来获取image，如果项目这种情况比较多，可以添加一个`symbolic breakpoint`来定位。<br />![](https://cdn.nlark.com/yuque/0/2018/png/183307/1541845301865-3989fad5-4209-460a-8ec8-af444c4a9643.png#width=536)

需要说明一下，$arg3用于模拟器，$r0用于真机，除了传入值是nil的情况，还可能是@""，这种情况的判断规则是`[$arg3  length] == 0`

<a name="nim5dg"></a>
#### [](#nim5dg)iOS导航栏显示与隐藏
项目中可能存在需要特殊隐藏导航栏的页面，通常情况下我们只需要在特殊页面执行`viewWillAppear`时隐藏导航栏，再在`viewWillDisapper`时显示导航栏以确保不对需要正常显示导航栏的页面产生影响。不过在实际项目中，页面之间的情况远比我们想象的复杂，我们以A、B两个页面为例：

- A 显示 B 隐藏  这种情况下我们按照上述操作是没有问题的

- A 隐藏 B 显示  这种情况下我们按照上述操作是没有问题的

- A 隐藏 B 隐藏  这种情况下就会存在连续执行先显示、再隐藏的情况，频繁操作就会存在导航栏异常的情况


为了解决这个问题，最佳方案应该是每个页面根据自身需要执行显示或隐藏导航栏的操作，如果让每个页面无形增加了代码量，而且不利于维护。为了进一步优化，我们可以利用`UINavigationControllerDelegate`做些文章。
```swift
extension ZYNavigationController: UINavigationControllerDelegate {
    func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {
        // 判断要显示的控制器是否是自己
        let isHidden = viewController.isKind(of: FourthViewController.self) || viewController.isKind(of: FirstViewController.self)
        
        self.setNavigationBarHidden(isHidden, animated: true)
    }
}
```

<a name="i5bdgx"></a>
#### [](#i5bdgx)iPhoneX 隐藏底部的Home条
```objectivec
- (BOOL)prefersHomeIndicatorAutoHidden {
      return YES;
}
```
需要注意，Home条并不会立刻消失，而是延迟1~2秒后缓慢消失

<a name="hhsehy"></a>
#### [](#hhsehy)轻松搞定状态栏变化
```swift
// 当设置statusBarHidden时，调用setNeedsStatusBarAppearanceUpdate立即更新状态栏
var isStatusBarHidden: Bool = false {
    didSet {
        self.setNeedsStatusBarAppearanceUpdate()
    }
}
// 状态栏样式 
override var preferredStatusBarStyle: UIStatusBarStyle {
    return .lightContent
}
//  状态栏是否隐藏   
override var prefersStatusBarHidden: Bool {
    return isStatusBarHidden
}
```
<a name="sydfiq"></a>
#### [](#sydfiq)引导用户跳转AppStore设置支付方式
```objectivec
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"https://buy.itunes.apple.com/WebObjects/MZFinance.woa/wa/editAddress?mt=8"]];
```
<a name="rgslgc"></a>
#### [](#rgslgc)自定义代码片段
Xcode可以添加一段代码的快捷方式：

1. 将要添加的代码选中，拖拽到Code Snippets区域

2. 在弹框里对代码片段进行编辑，将想要方便更换的内容以`<#  #>`包裹，对应内容就是可变内容啦

3. 在`Completion  Shortcut`一栏可以填入快捷标记

4. 完成以上步骤后，在需要使用相关代码片段时，只需要输入快捷标记就能快速输入代码片段

<a name="6l0pqp"></a>
#### [](#6l0pqp)腾讯QMUI Team开源的代码片段
腾讯QMUI Team开源了他们经常使用的代码片段，非常全面，下载后存放在**~/Library/Developer/Xcode/UserData/CodeSnippets**目录下即可。<br />[qmui-ios-codesnippets](https://github.com/QMUI/QMUI_iOS_CodeSnippets)
<a name="30i5og"></a>
#### [](#30i5og)Xcode控制台杂乱信息屏蔽
Xcode升级后，运行项目会打印一堆杂乱的log信息，虽然不影响App的功能，但是会对问题定位产生影响。为了解决这个我们可以在scheme里设置`os_activity_mode`为disable。这里有两个注意点：

1. NSLog打印的信息也会被一并屏蔽掉  解决方案：使用printf替代NSLog 

```objectivec
#ifdef DEBUG
#define NSLog(format, ...) printf("\n[%s] %s [第%d行] %s\n", __TIME__, __FUNCTION__, __LINE__, [[NSString stringWithFormat:format, ## __VA_ARGS__] UTF8String]);
#else
#define NSLog(format, ...)
#endif
```

2. swift的print打印信息不受影响




