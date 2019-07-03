
---

title: iOS高效开发

date: 2019-06-28 13:36:06 +0800

categories: iOS

tags: [总结]

---

工欲善其事必先利其器，合理的利用工具，提升开发效率，不仅仅帮助我们节省时间，关键是能帮我们从一些重复、低效的工作中抽离出来，专注于有挑战，有深度的问题，不断提升自己。这里总结一些我在日常开发中提升开发效率的一些技巧，如果您有更好的提升效率的方法也请不吝赐教。

<a name="RxXxI"></a>
## xopen快捷脚本
自定义xopen快捷脚本，在终端中快速打开项目<br />![](https://cdn.nlark.com/yuque/0/2019/gif/183307/1562028112900-78c7a90a-d135-43da-bda3-a8f633731c6d.gif#align=left&display=inline&height=410&originHeight=410&originWidth=656&status=done&width=656)

详细步骤：

1. 创建一个xopen文件 文件内容如下：
```ruby
#!/usr/bin/env ruby
require 'shellwords'

proj = Dir['*.xcworkspace'].first
proj = Dir['*.xcodeproj'].first unless proj

if proj
   puts "Opening  #{proj}"
   `open #{proj}`
else
  puts "No  xcworkspace|xcproj  file  found"
end
```

2. 将 `xopen` 文件移入 `/usr/local/bin` 目录下 并执行chmod 777添加读写权限

<a name="6OWDd"></a>
## 代码片段
Xcode为我们提供了一些代码片段，但是不够全面，不过Xcode支持我们自己添加代码片段，通过添加常用的代码片段，可以极大的节省开发效率。<br />腾讯的QMUI团队开源了一份他们总结的代码片段库_[qmui-ios-codesnippets](https://github.com/QMUI/QMUI_iOS_CodeSnippets)，_集成他们的代码片段库就能满足日常的开发需求，提升coding的效率。

<a name="nm4ba"></a>
## fastlane
[fastlane]()是一套ruby编写的持续集成工具集。通过fastlane可以实现自动打包、发布等工作。<br />原先我都是利用Xcode提供的 `xcodebuild` 命令自定义了一个 `xpublish` 脚本来进行打包，详细的配置过程可以参考：[iOS--两套自动打包脚本](https://juejin.im/post/5be2e07fe51d454d5c7c2b96)，不过当我发现有 `fastlane` 这个神器后果断放弃了原来使用的脚本，主要原因当然还是 `fastlane` 更加全面和强大。 `fastlane` 的集成过程比较简单，网上有很多资料，可以参考[小团队的自动化发布--Fastlane带来的全自动化发布](https://whlsxl.github.io/fastlane1/)。<br />因为我们项目已经使用Jenkins进行持续集成，日常使用 `fastlane` 并不多，主要会在偶尔打单独的测试包或审核包时才会使用，这里简单总结一下我的fastlane配置。

```ruby
default_platform(:ios)

platform :ios do
  desc "send ipa to pgyer"
  lane :pgyer do
    # 执行pod install 需要在Gemfile里配置cocoapods
    cocoapods(
      clean: true,
      podfile: "./Podfile"
    )
    # 打包项目
    build_app(workspace: "****.xcworkspace", scheme: "****", export_method: "ad-hoc", output_directory: "./fastlane/package", configuration: "Release")
    # 上传蒲公英 需要先安装蒲公英插件
    pgyer(api_key: "********", user_key: "**********")
    # 上传完成后 发送消息通知 避免忘记
    notification(subtitle: "Finished Uploading", message: "upload success")
  end
end
```

有几点说明：

1. 支持cocoapods需要在Gemfile里配置一下

```ruby
# 配置cocoapods，并指定版本
gem 'cocoapods', '1.7.1'
```


2. 蒲公英的插件安装命令

`fastlane add_plugin pgyer` 

安装完后的Gemfile:
```ruby
source "https://rubygems.org"

gem "fastlane"

# 配置cocoapods，并指定版本
gem 'cocoapods', '1.7.1'

plugins_path = File.join(File.dirname(__FILE__), 'fastlane', 'Pluginfile')
eval_gemfile(plugins_path) if File.exist?(plugins_path)
```

更多关于 `fastlane` 的功能可以查看[官方文档中的Actions](https://docs.fastlane.tools/actions/)。

<a name="wtjSf"></a>
## Jenkins
[Jenkins]()是一款开源的CI工具，利用Jenkins可以通过规范化的操作流程避免一些低级错误，将开发人员从简单、繁琐的工作中释放出来。关于Jenkins的配置教程网上也是有很多，[Jenkins 持续集成使用教程](https://juejin.im/post/5ad6beff6fb9a028c06b5889)就比较详细。

<a name="OTH2U"></a>
## SwitchHosts
[SwitchHosts]()是一个开源的用于hosts管理和切换的软件。<br />对于绝大多数公司肯定存在测试环境和线上环境，通常我们需要在测试环境验证无误后才会发布到正式环境，这就需要客户端在开发阶段能够请求不同的服务器IP，利用 `SwitchHosts` 和 `Charles` 就能实现不修改客户端代码的情况下访问不同的服务器IP。这样做就能避免了客户端打包时需要根据需求切换访问地址，之前就有公司误将测试环境的App发布给用户的情况。

