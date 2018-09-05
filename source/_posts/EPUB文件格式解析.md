title: EPUB文件格式解析
tags:
  - iOS进阶
  - epub
  - reader
categories:
  - iOS
date: 2017-07-20 17:45:00
---

### 背景知识

`EPUB`是Electronic Publication的缩写，是一种电子图书标准，`EPUB`是一个自由的开发标准，属于一种可以自动重新编排的内容，也就是说`EPUB`格式的图书，它的文字内容可以根据阅读设备的特性，以最适于阅读的方式显示。

### 文件组成

一个未经加密处理的epub电子书由以下三部分组成：

1. META-INF（文件夹，里面有一个container.xml文件）
2. OEBPS （文件夹，包含images文件夹、很多xhtml文件、css文件和content.opf文件）
3. mimetype

#### 文件mimetype
每个epub电子书均包含一个mimetype的文件，且内容不变，用以说明epub的文件格式。
```
application/epub+zip
```

#### 文件夹META-INF
META-INF用于存放容器信息，默认情况下此目录只包含一个文件container.xml，文件内容如下：
```
<?xml version="1.0"?>
<container version="1.0" xmlns="urn:oasis:names:tc:opendocument:xmlns:container">
    <rootfiles>
        <rootfile full-path="OEBPS/content.opf" media-type="application/oebps-package+xml"/>
    </rootfiles>
</container>
```
container.xml文件的主要功能用于告诉阅读器，电子书的根文件的路径和打开格式

#### 文件夹OEBPS

OEBPS目录用于存放OPF文档、CSS文档、NCX文档

#### 文件OPF
OPF文档是epub的核心文件，且是一个标准的xml文件，依据OPF规范，此文件的根元素为`<package>`
```
<package xmlns="http://www.idpf.org/2007/opf" version="2.0" unique-identifier="uuid_id">
```
其内容主要由五部分组成：

1. <metadata>
   元数据信息，这个标签里面是书籍的出版信息，由两个子元素组成
   - <dc-title> 标题
   - <dc-creator> 责任人
   - <dc-subject> 主题词、关键词
   - <dc-descributor> 内容描述
   - <dc-date> 日期
   - <dc-type> 类型
   - <dc-publisher> 出版者
   - <dc-contributor> 发行者
   - <dc-format> 格式
   - <dc-identifier> 标识信息
   - <dc-source> 来源信息
   - <dc-language> 语言
   - <dc-relation> 相关资料
   - <dc-coverage> 覆盖范围
   - <dc-rights> 权限描述
   
2. <manifest>
   文件列表，列出书籍出版的所有文件，但是不包括 mimetype、container.xml、content.opf，由一个个子元素构成
   ```
   <item id="" href="" media-type="">
   ```
   - id 文件的id号
   - href 文件的相对路径
   - media-type 文件的媒体类型
   
   ```
   <manifest>
    <item id="ncx" href="toc.ncx" media-type="application/x-dtbncx+xml" />
    <item href="cover.xhtml" id="cover" media-type="application/xhtml+xml"/>
    <item href="copyright.xhtml" id="copyright" media-type="application/xhtml+xml"/>
    <item href="catalog.xhtml" id="catalog" media-type="application/xhtml+xml"/>
    <item href="chap0.xhtml" id="chap0" media-type="application/xhtml+xml"/>
    </manifest>
   
   ```
   
3. <spine toc="ncx">
   
   提供书籍的线性阅读次序，由一个个子元素构成
   ```
   <itemref idref="copyright">
   ```
   - idref 即参考manifest列出的id
   
4. <guide>
   指南，一次列出电子书的特定页面

   ```
   <guide>
    <reference href="cover.xhtml" type="text" title="封面"/>
    <reference href="catalog.xhtml" type="text" title="目录"/>
   </guide>
   ```
5. <tour>
    导读，可以根据读者水平或阅读目的，按一定的次序，选择电子书的部分页面组成导读


#### NCX文件

NCX文件是epub电子书的另一个重要文件，用于制作电子书的目录。.ncx文件中最主要的节点是navMap,navMap节点是由许多navPoint节点组成

```
<?xml version="1.0" encoding="utf-8"?>
<ncx xmlns="http://www.daisy.org/z3986/2005/ncx/" version="2005-1">
    <head>
        <meta content="178_0" name="dtb:uid"/>
        <meta content="2" name="dtb:depth"/>
        <meta content="0" name="dtb:totalPageCount"/>
        <meta content="0" name="dtb:maxPageNumber"/>
    </head>
    <docTitle>
        <text>1984</text>
    </docTitle>
    <docAuthor>
        <text>[英] 乔治·奥威尔</text>
    </docAuthor>
    <navMap>
        <navPoint id="catalog" playOrder="0">
            <navLabel>
                <text>目录</text>
            </navLabel>
            <content src="catalog.xhtml"/>
        </navPoint>
        <navPoint id="chap0" playOrder="1">
            <navLabel>
                <text>前言</text>
            </navLabel>
            <content src="chap0.xhtml"/>
        </navPoint>
        <navPoint id="chap1" playOrder="2">
            <navLabel>
                <text>　　第一部</text>
            </navLabel>
            <content src="chap1.xhtml"/>
        </navPoint>
        <navPoint id="chap2" playOrder="3">
            <navLabel>
                <text>　　第1节</text>
            </navLabel>
            <content src="chap2.xhtml"/>
        </navPoint>
        <navPoint id="chap3" playOrder="4">
            <navLabel>
                <text>　　第2节</text>
            </navLabel>
            <content src="chap3.xhtml"/>
        </navPoint>
        <navPoint id="chap4" playOrder="5">
            <navLabel>
                <text>　　第3节</text>
            </navLabel>
            <content src="chap4.xhtml"/>
        </navPoint>
    </navMap>
</ncx>
```

### 开源项目

目前支持`EPUB`格式的开源阅读器有：

#### iOS 阅读器

- [Reader](https://github.com/GGGHub/Reader)--一款基于CoreText实现的电子书阅读器，支持txt，epub格式
- [FolioReaderKit](https://github.com/FolioReader/FolioReaderKit)--Swift版的ePub阅读器
- [KFEpubKit](https://github.com/ricobeck/KFEpubKit)--支持iOS和OSX的epub阅读器
- [Reader](https://github.com/vfr/Reader)--pdf阅读器

#### Android 阅读器

- [BookReader](https://github.com/JustWayward/BookReader)--一款比较完整的阅读器，支持txt/pdf/epub书籍阅读
- [FBReader](https://github.com/geometer/FBReader)--C++项目，支持多平台的阅读器
- [FolioReader-Android](https://github.com/FolioReader/FolioReader-Android)--FolioReader的android版本

