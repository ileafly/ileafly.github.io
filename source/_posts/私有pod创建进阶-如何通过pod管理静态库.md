---
title: 私有pod创建进阶--如何通过pod管理静态库
date: 2016-09-10 15:46:08
tags: CocoaPods
categories: iOS
---

在[搭建私有Pod](http://www.ileafly.com/2016/08/12/%E6%90%AD%E5%BB%BA%E7%A7%81%E6%9C%89Pod/)一文中我们已经总结了如何创建私有的Pod，这次来总结一下如果利用Pod生成和管理静态库。

## 创建并发布Pod项目

首先，我们需要参考[搭建私有Pod](http://www.ileafly.com/2016/08/12/%E6%90%AD%E5%BB%BA%E7%A7%81%E6%9C%89Pod/)创建一个pod项目至**pod lib lint**验证无误。

## 打包类库

我们需要使用一个cocoapods的插件[cocoapods-packager](https://github.com/CocoaPods/cocoapods-packager)来完成类库的打包。当然也可以手动编译打包，只是过程会很繁琐。
  
   1. 安装打包插件
   
        sudo gem install cocoapods-packager
        
   2. 打包
   
        pod package Lib.podspec --library --force
        
   **--library**指定打包成 **.a** 文件，如果不带上将会打包成 **.framework** 文件。 **--force**是指强制覆盖。
   
## 问题解决

打包完成后我们就能在当前工程的目录下找到类库文件，将类库拖拽到项目中集成遇到了编译报错
造成编译通不过的原因是无法找到类库中使用的一些系统方法，这是因为我们没有在`podspec`文件里添加系统库的引用和开放头文件。
    
        s.public_header_files = 'ZYLib/Classes/**/*.h'
            
        s.frameworks = 'UIKit', 'CoreFoundation', 'QuartzCore', 'AssetsLibrary', 'ImageIO', 'Accelerate', 'MobileCoreServices', 'sqlite3', 'libz'


   
## 管理类库

打包完成之后，我们能够在对应项目根目录里找到打包好的文件，我们可以直接将他们拖拽到项目中使用。当然，我们也可以利用Pod来帮我们管理打包好的类库。

1. 在项目根目录中建一个文件夹命名为Release，里面存放一个podspec文件和项目打包成功的.a文件以及.h文件
2. 修改podspec   

            // 指定.h代码源文件地址
            s.source_files = 'Release/**/*.h'
            // 指定.a库地址
            s.vendored_libraries = 'Release/Lib.a'
            
3. 上传Release代码到git master分支，并打上对应tag
4. 发布pod

贴上Demo中的podspec供参考

         Pod::Spec.new do |s|
            s.name             = 'ZYLib'
            s.version          = '0.1.0'
            s.summary          = 'A short description of ZYLib.'

            s.homepage         = 'https://coding.net/u/ileafly/p/ZYLib/git'
            s.license          = { :type => 'MIT', :file => 'LICENSE' }
            s.author           = { 'luzy' => 'luzy@2345.com' }
            s.source           = { :git => 'https://coding.net/u/ileafly/p/ZYLib/git', :tag => s.version.to_s }

            s.ios.deployment_target = '7.0'

            s.source_files = 'Release/**/*'

            s.vendored_frameworks = 'Release/ZYLib.a'
  
            #s.resource_bundles = {
            #  'ZYLib' => ['ZYLib/Assets/*.png']
            #}

            s.public_header_files = 'ZYLib/Classes/**/*.h'
            s.frameworks = 'UIKit', 'CoreFoundation', 'QuartzCore', 'AssetsLibrary', 'ImageIO', 'Accelerate', 'MobileCoreServices', 'sqlite3', 'libz'
            s.dependency 'AFNetworking', '~> 2.3'
         end 

   
## 参考文章

[使用CocoaPods开发并打包静态库](http://www.cnblogs.com/brycezhang/p/4117180.html)
