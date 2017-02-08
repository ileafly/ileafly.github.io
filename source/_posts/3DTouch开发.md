---
title: 3DTouch开发
date: 2016-12-30 16:15:53
tags: iOS, 3DTouch
categories: iOS
---

### 准备工作
开发3DTouch功能需要做好以下准备工作：
    1. iPhone6s或以上的设备
    2. iOS9或以上的系统
    3. Xcode7或以上的开发环境

### 主屏幕按压应用图标展示快捷选项
应用最多可以有4个快捷选项标签，iOS9为我们提供了两种方式来开发按压应用图片展示快捷选项的功能
1. 静态标签
   静态标签是我们通过项目的plist文件进行配置的，我们需要在plist里手动添加一个属性`UIApplicationShortcutItems`，类型为数组，每个元素即表示一个快捷选项标签
   每个快捷选项标签由以下参数构成：
   `UIApplicationShortcutItemTitle` 标签标题 (必填)
   `UIApplicationShortcutItemType` 标签的唯一标识 (必填)
   `UIApplicationShortcutItemIconType` 使用系统图片的类型
   `UIApplicationShortcutItemIconFile` 使用项目中的图片作为标签图标
   `UIApplicationShortcutItemSubtitle` 标签副标题
   `UIApplicationShortcutItemUserInfo` 字典信息，用于传值

2. 动态标签
   静态标签添加比较方便，但是太不灵活，还好苹果给我们提供了接口来配置快捷选项，下面是添加快捷选项的代码：
   ```
   func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        
        self.createShortcutItems()
        
        return true
    }

   // 创建应用图标上的3D Touch快捷选项
    func createShortcutItems() {
        if #available(iOS 9.0, *) {
            let icon: UIApplicationShortcutIcon = UIApplicationShortcutIcon.init(type: .share)
            
            let item: UIApplicationShortcutItem = UIApplicationShortcutItem.init(type: "com.ileafly.3DTouchDemo.share", localizedTitle: "分享", localizedSubtitle: nil, icon: icon, userInfo: nil)
            
            UIApplication.shared.shortcutItems = [item]
        } else {
            // Fallback on earlier versions
        }
    }
   ```

3. 点击快捷选项标签进入应用的响应

  ```
  @available(iOS 9.0, *) // 只有9系统才能调用此方法
    func application(_ application: UIApplication, performActionFor shortcutItem: UIApplicationShortcutItem, completionHandler: @escaping (Bool) -> Void) {
        print(shortcutItem);
        
        if let userInfo = shortcutItem.userInfo {
            print(userInfo);
        }
    }

  ```
  
  #### 参考文章
  
  [iOS 3D touch 开发(一) Home Screen Quick Actions](http://liuyanwei.jumppo.com/2016/03/21/iOS-3DTouch-1.html)

