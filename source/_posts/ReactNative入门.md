---
title: ReactNative入门
date: 2016-11-02 09:32:53
tags: ReactNative
categories: ReactNative
---

### 安装

#### Homebrew

[Homebrew](http://brew.sh/)是Mac系统的包管理器，我们要使用它安装NodeJS和一些其他必须的工具。

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

#### Node.JS

使用Homebrew安装[Node.js](https://nodejs.org/en/)

```shell
brew install node
```

安装完成后最好设置npm镜像以加速安装过程

```sh
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```

#### React Native的命令行工具(react-native-cli)

React Native的命令行工具用于执行创建、初始化、更新项目、运行打包服务等任务。

```shell
npm install -g react-native-cli
```

#### Xcode

React Native目前需要Xcode 7.0或更高版本。

### 推荐安装的工具

#### Watchman

[Watchman](https://facebook.github.io/watchman/docs/install.html)是由Facebook提供的监视文件系统变更的工具。安装此工具可以提供开发时的性能

```shell
brew install watchman
```

#### Flow

[Flow](https://www.flowtype.org/)是一个静态的JS类型检查工具。(新手建议跳过，降低学习成本)

```shell
brew install flow
```

#### Sublime Text 3

[Sublime](http://www.sublimetext.com)是一款开源的编辑器，支持插件安装。

##### React Native开发推荐的一些插件

1. ReactJS: 支持React开发，代码提示，高亮显示

2. Emmet: 前端开发必备

3. Terminal: 在sublime中打开终端并定位到当前目录

4. react-native-snippets: react native的代码片段

   ​

#### 测试安装

```shell
react-native init ReactNativeDemo
cd ReactNativeDemo
react-native run-ios
```

在测试过程中遇到一个问题导致编译失败，报错的原因是Watchman没有安装，于是我又尝试用Homebrew安装Watchman，结果在快要完成的时候报错了，报错的原因是未正确安装Command Line Tools,执行`xcode-select --install`，安装完成后再次安装Watchman，并运行通过。

### 参考文章

[ReactNative搭建开发环境](http://reactnative.cn/docs/0.36/getting-started.html#content)

[用Sublime 3作为ReactNative的开发IDE](http://www.jianshu.com/p/2ddfff095e90)

