
title: ZYRouterManager--Swift
date: 2018-11-21 14:12:41 +0800
tags: []
categories: 
---
# <a name="9pf8aw"></a>ZYRouterManager--Swift
封装一个简易、灵活的路由中间件，适合小型项目使用--Swift版

## <a name="mzlzvo"></a>目的

随着项目越来越大，页面之间的跳转逻辑也越来越复杂，加上有很多h5页面也存在着与原生页面跳转的逻辑，为了统一管理页面的跳转逻辑，封装了一个路由中间件的类

## <a name="ck62gs"></a>预期

提供一个类方法，解析传入的路径，跳转到相应的界面

## <a name="yg84rf"></a>实现方案

###### <a name="p1oqks"></a>提供一个统一的路由中间件入口

提供了一个唯一对外开放的类方法，需要传入跳转的scheme。

```swift
/// 路由中间件入口方法
///
/// - Parameter schema: 需要跳转的scheme
/// - Returns: 是否能跳转
class func routerWithSchema(_ schema: String) -> Bool {
    guard schema.count > 0 else {
        return false
    }
    if let urlString = schema.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) {
        if let url = URL(string: urlString) {
            return ZYRouterManager.analysisScheme(url)
        }
    }
        
    return false
}
```

###### <a name="5n19nl"></a>分析传入的scheme

分析传入的scheme，通过对比host判断跳转路径，通过解析请求参数，将传递的参数赋值给相对应的controller。

```swift
/// 分析传入的scheme
///
/// - Parameter url: 跳转的URL对象
/// - Returns: 是否能跳转
private class func analysisScheme(_ url: URL) -> Bool {
    let routePattern = url.absoluteString
    if let components = URLComponents(string: routePattern), let scheme = components.scheme {
            
        guard scheme == RootScheme else {
            return false
        }
            
        let host = components.host
            
        var queryParams = Dictionary<String, Any>()
            
        if let queryItems = components.queryItems {
            for item in queryItems {
                if item.value == nil {
                    continue
                }
                queryParams[item.name] = item.value
            }
        }
            
        switch host {
        case RouterPath.detailPath.rawValue:
            do {
                // 跳转到详情页
                let vc = DetailViewController()
                let currentVC = ZYRouterManager.getCurrentVC()
                if let productId = queryParams["productId"] as? String {
                    vc.productId = productId
                }
                currentVC?.navigationController?.pushViewController(vc, animated: true)
            }
            break
        case RouterPath.searchPath.rawValue:
            do {
                // 跳转到搜索页
                let vc = SearchViewController()
                let currentVC = ZYRouterManager.getCurrentVC()
                currentVC?.navigationController?.pushViewController(vc, animated: true)
            }
            break
        default: break
                
        }
    }
    return false
}
```

###### <a name="xy09yi"></a>获取当前控制器

为了方便使用者调用，`ZYRouterManager`内部实现了获取当前控制器的流程，大体思路是通过rootViewController一步一步向下查找，一直找到`UIViewController`对象，作为当前控制器返回

```swift
/// 获取当前控制器
///
/// - Returns: 当前控制器
private class func getCurrentVC() -> UIViewController? {
    if let rootVC = UIApplication.shared.keyWindow?.rootViewController {
        return ZYRouterManager.findCurrentVC(rootVC)
    }
    return nil
}
    
/// 在rootVC中查找currentVC
///
/// - Parameter parentVC: 父ViewController
/// - Returns: 查找到的ViewController
private class func findCurrentVC(_ parentVC: UIViewController) -> UIViewController? {
    if parentVC.isKind(of: UINavigationController.self), let nav = parentVC as? UINavigationController {
        // rootVC是UINavigationController
        if let vc = nav.childViewControllers.last {
          return ZYRouterManager.findCurrentVC(vc)
        }
        return nil
    } else if parentVC.isKind(of: UITabBarController.self), let tabBar = parentVC as? UITabBarController {
        // rootVC是UITabBarController
        if let vc = tabBar.selectedViewController {
            return ZYRouterManager.findCurrentVC(vc)
        }
        return nil
    } else if let presentedVC = parentVC.presentedViewController {
        return ZYRouterManager.findCurrentVC(presentedVC)
    }
    return parentVC
}
```

## <a name="7882cc"></a>思路扩展

这里路由控制器的实现还是比较简单的，主要就是通过对比host判断加载对应的controller。比较适合中小型项目的使用，如果项目是组件化搭建的，应用场景相对更复杂，目前相关比较好的库是[JLRoutes](https://github.com/joeldev/JLRoutes)。
[JLRoutes](https://github.com/joeldev/JLRoutes)的主要实现思路是在启动时注册一组跳转map，当传递了对应的scheme后就会触发相应的block方法，在block中实现具体的跳转逻辑。

