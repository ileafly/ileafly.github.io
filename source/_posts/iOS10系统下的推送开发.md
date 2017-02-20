---
title: iOS之iOS10系统下的推送开发
date: 2016-11-01 11:39:00
tags: iOS小积累
categories: iOS
---

iOS10系统发布后，推送功能发生了很多变化，作为iOS开发者就需要适配这些变化，这里对于如果结合极光推送适配iOS10系统进行一个简单的总结，方便在对其他项目进行适配时使用。

### 相关配置

1. 开启`Push Notifications`功能
   
   ![功能开关](https://github.com/luzhiyongGit/luzhiyongGit.github.io/tree/hexo/public/images/iOS之iOS10系统下的推送开发_1.png)

2. 添加`UserNotifications.framework`

   ![新增系统库](https://github.com/luzhiyongGit/luzhiyongGit.github.io/tree/hexo/public/images/iOS之iOS10系统下的推送开发_2.png)

### 代码适配

```
// 注册推送通知
// iOS8系统以前用来注册一个远程推送通知，当我们调用这个方法时，我们就在Apple Push Notification service上进行了设备注册
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound];
// iOS8 ~ iOS10 
    UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound categories:nil];
    [[UIApplication sharedApplication] registerUserNotificationSettings:settings];
    [[UIApplication sharedApplication] registerForRemoteNotifications];
    
// iOS10
    UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
    
    [center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionBadge | UNAuthorizationOptionSound) completionHandler:^(BOOL granted, NSError * _Nullable error) {
        if (!error) {
            NSLog(@"request authorization success!");
            [[UIApplication sharedApplication] registerForRemoteNotifications];
        }
    }];

```

```
// 利用极光推送注册推送通知
// iOS10 Before
    
    [JPUSHService registerForRemoteNotificationTypes:(UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert) categories:nil];
    
// iOS10
    
    JPUSHRegisterEntity *entity = [[JPUSHRegisterEntity alloc] init];
    entity.types = UNAuthorizationOptionAlert | UNAuthorizationOptionSound | UNAuthorizationOptionBadge;
    [JPUSHService registerForRemoteNotificationConfig:entity delegate:self];
    
// 注册极光推送
    [JPUSHService setupWithOption:launchOptions appKey:@"" channel:@"" apsForProduction:YES];
```