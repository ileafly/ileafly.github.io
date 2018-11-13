
title: fastlane指南
date: 2018-11-08 10:20:01 +0800
tags: [自动化、fastlane]
categories: 教程
---

[fastlane](https://github.com/fastlane/fastlane)是一套使用Ruby写的自动化工具集，为iOS和Android开发者提供了自动化构建工具，可以帮助开发者将App打包、签名、测试、发布、信息整理、提供AppStore等工作连接起来，实现完全自动化的工作流。
[fastlane-doc](https://docs.fastlane.tools) 参考文档。

### <a name="ule4kq"></a>安装
```powershell
# 安装 fastlane
gem install fastlane

# 检查版本 
fastlane --version
```

### <a name="upokfd"></a>配置
```ruby
# 初始化配置
cd 项目目录
## fastlane init期间会有四种配置选择，这里我们选择4 完全自定义，然后一直enter直至完成即可
fastlane init

# fastlane init执行完成后会在当前目录下创建一个fastlane文件夹，包含Appfile和Fastfile文件
## Appfile

app_identifier("bundle  id") # The bundle identifier of your app
apple_id("********@mail.com") # Your Apple email address

# For more information about the Appfile, see:
#     https://docs.fastlane.tools/advanced/#appfile

## Fastline

default_platform(:ios)

platform :ios do
  desc "Description of what the lane does"
  lane :betaDebug do
    # add actions here: https://docs.fastlane.tools/actions
    # pod install
    cocoapods(use_bundle_exec: false)
    # build app
    build_app(workspace: "app.xcworkspace", scheme: "app", export_method: "ad-hoc", output_directory: "./fastlane/package", configuration: "Release")
    # upload pgyer
    pgyer(api_key:  "****************",  user_key:  "***********************")
    # send email
    mailgun(
      postmaster: "postmaster@*************************.mailgun.org",
      apikey: "********************-4412457b-38b62932", 
      to: "tester1@mail.com,tester2@mail.com,tester3@mail.com",
      from: "author <author@mail.com>",
      success: true,  
      message: "app 上传蒲公英成功",
      app_link: "https://www.pgyer.com/****",
      )
  end
end
```

按照上述流程配置好fastlane，执行`fastlane  betaDebug `不出意外你将看到执行成功的信息，在fastlane\package目录下找到ipa文件。
### <a name="h1gtce"></a>上传蒲公英
1. 安装蒲公英插件
```plain
fastlane add_plugin pgyer
```
2. 配置蒲公英账号信息
```ruby
lane :betaDebug do
    # add actions here: https://docs.fastlane.tools/actions
    build_app(workspace: "app.xcworkspace", scheme: "app", export_method: "ad-hoc", output_directory: "./fastlane/package", configuration: "Release")
    pgyer(api_key:  "****************",  user_key:  "***********************")
  end
```
3. 打包上传
```powershell
fastlane betaDebug
```
### <a name="fgh2mb"></a>pod install
fastlane集成了`cocoapods`进行pod管理，在终端里输入`fastlane cocoapods`可以查看相关的API文档。
1. 在Gemfile中添加`gem "cocoapods"`
2. 在Fastfile中添加cocoapods的配置
```ruby
# 这里需要注意，我尝试过按照文档里设置Podfile的路径，发现不能成功执行pod install，后来通过调查使用如下命令可以成功执行，具体原因不明
cocoapods(use_bundle_exec: false)
```
### <a name="ng4arh"></a>邮件通知
fastlane集成了`mailgun` 进行邮件发送，在终端里输入fastlane mailgun可以查看相关的API文档，这里总结一下正确集成mailgun的流程

1. 注册mailgun
 [mailgun](https://www.mailgun.com)是一个开发人员的电子邮件服务，提供了强大的API，每个月可免费发送10000封邮件，而且还可以进行跟踪日志等操作。按照流程注册完后会有一个测试domain，利用测试domain可以发送邮件，不过需要添加收件邮箱的验证，通过添加自定义域名可以使用更强大的功能。
1. 在Gemfile中添加`gem "rest-client"`
2. 在Fastfile中添加mailgun的配置
```ruby
# mailgun 邮件通知    
    mailgun(
      postmaster: "postmaster@****************.mailgun.org",
      apikey: "********************-4412457b-38b62932", 
      to: "tester1@mail.com,tester2@mail.com,tester3@mail.com",
      from: "author <author@mail.com>",
      success: true,  
      message: "app 上传蒲公英成功",
      app_link: "https://www.pgyer.com/****",
      )
```
4. 终端执行fastlane命令，成功配置时你将看到如下信息
```ruby
[10:51:39]: ------------------------------
[10:51:39]: --- Step: default_platform ---
[10:51:39]: ------------------------------
[10:51:39]: Driving the lane 'ios betaDebug' 🚀
[10:51:39]: ---------------------
[10:51:39]: --- Step: mailgun ---
[10:51:39]: ---------------------

+------+-----------------+-------------+
|           fastlane summary           |
+------+-----------------+-------------+
| Step | Action          | Time (in s) |
+------+-----------------+-------------+
| 1    | default_platfo  | 0           |
|      | rm              |             |
| 2    | mailgun         | 1           |
+------+-----------------+-------------+

[10:51:41]: fastlane.tools finished successfully 🎉
```

#### <a name="ewqdsv"></a>mailgun相关注意点
* mailgun会自动读取git提交message放入邮件内容中
* mailgun的apikey配置不正确时会提供401认证错误
* 使用测试域名，配置多个接收邮箱时，有一个邮箱未接收认证，就会导致所有的都失败

### <a name="oxgbns"></a>fastlane进阶
上面的流程只是帮我们简单实现fastlane打包与上传至蒲公英的流程，绝大多数情况下已经够用，不过fastlane还有很多功能可以挖掘

* [x] fastlane 支持pod install
* [x] fastlane 上传蒲公英后邮件通知测试人员
* [ ] fastlane 上传Testflight
* [ ] fastlane 上传AppStore 

### <a name="6yx6gx"></a>使用过程中的一些坑
1. fastlane配置完成后执行一直报错，编译不能通过，通过分析发现因为我装了两个Xcode，默认使用的是9.0版本的command line，而代码是在Xcode 10中编写的，使用了一些swift 4.2的语法，解决方案是在设置里切换command line版本即可。
2. cocoapods的使用其他命令尝试了一直报错，只有上面那个命名能够成功，原因还未知
3. mailgun的集成详细的教程比较少，要多尝试，如果遇到401报错就看看apiKey是否设置正确，另外要注意使用测试域名，接受邮件的邮箱需要先邀请认证，否则会报400错误。


