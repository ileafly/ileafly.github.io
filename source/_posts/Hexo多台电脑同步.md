---
title: Hexo多台电脑同步
date: 2016-10-24 15:00:08
tags: 博客, Hexo
categories: 博客
---

使用Hexo搭建个人博客已经有一段时间了，在使用中发现一个问题，github上存放的并非是原始的博客文件而是经过hexo处理的html文件，这就导致无法在两台电脑上协同工作，多么悲伤的事情！
难道我需要单独创建一个git项目来存放? 通过调研找到了一个方便快捷的管理源码的方法。

### Hexo多台电脑同步方案

1. 创建两个分支：master和hexo
2. 设置hexo为默认分支，[github](https://www.github.com)的setting里可以设置
3. git clone 拷贝仓库
4. 进入仓库初始化hexo
5. 配置hexo `_config.yml` 可参考[个人博客进阶--Hexo搭建博客](http://www.ileafly.com/2016/08/25/%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E8%BF%9B%E9%98%B6-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)
6. 依次执行 `git add .` `git commit -m "message"` `git push origin hexo` 提交相关文件

这样，在你的博客项目里就存在两个分支，master分支用来存放生成的今天网页，hexo分支用来存放网站的原始文件

### 新电脑如何同步Hexo

1. git clone 拷贝仓库
2. 已安装hexo则直接使用，未安装Hexo只需要安装Hexo即可

### 待尝试的同步方法
有大神写了一个同步的脚本，只需要执行`hexo b`就可以达到同步的目的，还没有尝试，[hexo-git-backup](https://github.com/coneycode/hexo-git-backup)

### 参考地址
[hexo多台电脑同步](https://www.zhihu.com/question/21193762)
