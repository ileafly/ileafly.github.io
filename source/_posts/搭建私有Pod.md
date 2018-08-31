title: 搭建私有Pod
tags:
  - iOS进阶
  - ''
categories:
  - iOS
date: 2016-08-12 10:21:00
---

[Cocoapods](http://cocoapods.org/)是一款非常好用的iOS依赖库管理工具，使用它可以方便的管理和更新项目中所使用到的第三方库。同时我们也可以将公司中的公共组件或者项目中的核心组件交由它去管理。本文就是给大家介绍一下，如何使用Cocoapods创建私有的Pod仓库。

 
### 创建一个私有Pod的步骤：
 
 1. 创建并设置一个私有的 **Spec Repo**
 2. 创建Pod所需管理的项目
 3. 创建Pod所对应的podspec文件
 4. 本地测试配置好的podspec文件是否可用
 5. 向私有的Spec Repo中提交podspec
 6. 在个人项目中的Podfile中增加刚刚制作好的Pod并使用
 7. 更新维护podspec
 
### 创建私有Spec Repo
 
先来说第一步，什么是Spec Repo？它是所有的Pods的一个索引，就是一个容器，所有公开的Pods都在这个里面，我们常用的第三方库都是放在[Github](http://www.github.com)里开源的[Specs](https://github.com/CocoaPods/Specs)中，当我们使用CocoaPods后就会在~/.cocoapods/repos目录下生成一个官方的Spec Repo文件夹master。这个master目录的结构如图：
![Spec Repo](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/Pod/spec_struct.png)

我们首先需要做的就是创建一个类似于master的私有Spec Repo：  
   1. 创建一个Git仓库  
   2. 创建完成之后再终端执行如下命令
      
      pod repo add [Private Repo Name] [Github Https clone URL]
 
### 创建Pod项目工程文件

Cocoapods为我们提供了一个工具，用来创建Pod管理的项目，相关的文档介绍是[using-pod-lib-create](http://guides.cocoapods.org/making/using-pod-lib-create)，下面我们通过创建一个podDemoLibrary
     
     pod lib create podDemoLibrary

这段命令执行完之后，它会问你四个问题：

1.是否需要一个例子工程

2.选择一个测试框架

3.是否基于View测试

4.类的前缀

根据你的实际需求回答这四个问题，通常我们都会选择需要一个例子工程，至于是否需要测试框架或者类的前缀都根据实际请求选择。

回答完问题后，pod会自动帮你创建对应的目录结构，接下来你可以通过终端进入Example文件夹内执行**pod install**，然后打开**podDemoLibrary.workspace**，在Pods工程下面，会有一个Resources文件夹用来存放你的图片资源和一个Classes文件夹用来存放你的代码。至此，pod项目创建已经完成。

### 编辑Pod项目工程文件
pod项目创建完之后你还需要修改它，已达到实现你的功能需求的目的，接下来就是如何编辑Pod文件。

#### 如何添加代码

添加代码的方法同正常的文件创建一样，只是在创建完成之后我们并不能直接使用它，我们需要重新在终端进入Example文件夹内执行**pod install**

#### 如何添加图片资源

想要添加图片资源，首先，我们需要在Assets文件夹内存放图片资源，然后打开**podDemoLibrary.podspec**文件，找到如下代码：

     s.resource_bundles = {
       'podDemoLibrary' => ['podDemoLibrary/Assets/*.png']
     } 

放开注释，重新执行**pod install**，这样你就能在项目中看到Resource文件夹了。

虽然，你已经能看到Resource文件夹，但是，你并不能直接通过
   
      [UIImage imageWithNamed:@"pic"];
      
加载图片。你可以通过一个宏定义方法加载图片，
   
      #define SrcWithName(name) [@"podDemoLibrary.bundle" stringByAppendingPathComponent:name]
      [UIImage imageWithNamed:SrcWithName(@"pic")];


#### 如何添加依赖库

依赖库的添加需要在**podDemoLibrary.podspec**中添加，s.frameworks 对应的是系统库，s.denpendency 对应的是第三方pod库

### 发布Pod项目
1.上传代码到git master
   
      git add .
      git commit -m "Initial Commit"
      git push origin master
      
2.tag标记
     
      git tag -m "first release" 0.1.0
      git push --tags
      
3.验证pod

      pod lib lint
      
4.发布pod

      pod repo push Specs podDemoLibrary.podspec

### 查找Pod项目

      pod search podDemoLibrary
      
如果你能查询到**podDemoLibrary**项目，说明你的pod创建成功了。

### 注意点

在实际使用中遇到了各种零散的问题，这里将遇到的问题做一个总结。

1. 上传了一个错误的tag，需要删除已经上传的tag
    
        git tag -d 0.1.0
        git push origin :refs/tags/0.1.0
      
2. pod lib lint时，发现存在warning
   
        pod repo push Specs podDemoLibrary.podspec --allow-warnings
        
3. 依赖库中有一个framework

        pod repo push Specs podDemoLibrary.podspec --use-libraries
        

### 参考

[使用Cocoapods创建私有podspec](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)


