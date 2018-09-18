title: 个人博客进阶--Hexo搭建博客
tags:
  - Hexo
categories:
  - 其他
date: 2016-08-25 10:31:00
---

在结合Github、Jekyll搭建好自己的博客后，发现无法方便的更换主题和给文章进行分类，通过调查发现很多精美的博客都是结合Github和[Hexo](https://github.com/hexojs/hexo)搭建的，本文就是分享一下我是如何利用Hexo搭建自己的博客的。

### 安装Hexo

#### 安装Node.js
hexo是一款基于[Node.js](https://nodejs.org/en/)的静态博客框架，所以想要安装Hexo，我们首先需要安装Node.js库。我们可以到Node.js的[官网](https://nodejs.org/en/)下载相应平台的最新版本，一路安装即可。

#### 正式安装Hexo

1.执行安装Hexo命令
        
         sudo npm install -g hexo
        
2.创建blog文件夹并cd到blog文件夹内

3.执行init命令初始化hexo

         hexo init
         
4.生成静态页面

         hexo g
       
5.本地启动服务器，进行文章预览

         hexo server
         
  在浏览器中输入http://localhost:4000 即可预览文章
  
6.将github仓库与blog建立关联

         vi _config.yml
         
         deploy:
            type: git
            repo: https://github.com/luzhiyongGit/luzhiyongGit.github.io
            branch: master
            
   
  执行配置命令
  
         npm install hexo-deployer-git -S
         
7.发布博客

         hexo d
         
  执行完毕后，你会发现你的github仓库内容已经发生了更改，同时打开[https://luzhiyonggit.github.io](https://luzhiyonggit.github.io)，能够看到一篇默认博客
  
### 利用hexo编写博客

1.新建一篇博客

        hexo new 博客名称
        
2.生成博客页面

        hexo g
        
3.发布博客

        hexo d
        
### 博客主题

Hexo有非常多的主题可以选择替换，替换流程也特别方便

1.从[Hexo Themes](https://github.com/hexojs/hexo/wiki/Themes)中找到一个你心仪的主题，本博客使用的是[next](https://github.com/iissnan/hexo-theme-next)

2.在你的blog文件目录下，执行下面的命令

     git clone https://github.com/iissnan/hexo-theme-next.git themes/next
     
3.编辑配置信息

     vi _config.yml
     
     # Extensions
     ## Plugins: http://hexo.io/plugins/
     ## Themes: http://hexo.io/themes/
     theme: next


4.生成博客页面

     hexo g
     
5.发布博客

     hexo d

### 特效配置

    next主题有一个[next wiki](https://github.com/iissnan/hexo-theme-next/wiki/)提供了很多特性配置的方案，我们可以通过参考它完成很多特性配置
     
### 参考链接

[简洁轻便的博客平台: Hexo详解](http://www.tuicool.com/articles/ueI7naV)
