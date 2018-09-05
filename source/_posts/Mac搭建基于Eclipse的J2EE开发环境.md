title: Mac搭建基于Eclipse的J2EE开发环境
tags:
  - 开发环境
categories: 
  - 其他
date: 2017-12-14 19:21:00
---
虽然网上有各种Mac环境下搭建基于Eclipse的J2EE开发环境的博客，但是在实际参考中或多或少存在着一些偏差，经过一段时间的努力，总算是成功搭建完成并跑通了项目，这里做一个总结供大家参考。
## 安装JDK
- 打开终端输入`java -version`判断是否已安装java运行环境
- 如果未安装则下载JDK并进行安装 [下载地址](http://www.oracle.com/technetwork/java/javaee/downloads/java-ee-sdk-7-jdk-7u21-downloads-1956231.html)
## 安装Tomcat
- 在[Apache官网](http://tomcat.apache.org)下载最新的tomcat二进制包
- 将二进制包解压后改名为Tomcat，并复制到你希望存放的目录，一般存放在`/Library`目录下
- 通过终端修改Tomcat的权限 `sudo chmod 755 /Library/Tomcat`
- 执行`/Library/Tomcat/bin/startup.sh`启动Tomcat
- 打开[http://localhost:8080](http://localhost:8080)验证Tomcat是否安装成功

![](http://upload-images.jianshu.io/upload_images/260268-fcc36bece30a4fbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

## 安装Eclipse

- 访问[Eclipse官方首页](http://www.eclipse.org/home/index.php)下载安装包
- 执行`Eclipse installer`，将Eclipse拖入应用程序中
- 打开Eclipse设置工作空间

## 配置Eclipse

- Eclipse 偏好设置
- java Installed JREs
- Add 选择`Standard VM`
- Directory 选择本地安装的JDK的路径
  JDK路径查询命令行
 ```
  ls -l `which java`
 ```
- 选中新添加的JRE
- 右击项目
- 选择`Java Build Path`
- Add 选择`User Library`
- 选择Tomcat
- 配置完成

