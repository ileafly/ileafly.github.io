---
title: SDWebImage与YYWebImage加载gif对比
date: 2017-08-16 16:53:07
tags: iOS
---

很多应用为了丰富运营策略都需要支持gif图片，`SDWebImage`和`YYWebImage`已经为我们提供了这方面的支持，我们只需要按照对应的文档进行支持即可。但是在实际使用中发现`SDWebImage`和`YYWebImage`在性能方面存在着较大的差距，这里就给大家进行一个对比

## SDWebImage加载gif

`SDWebImage`将gif资源中的每一张image写入到内存中，通过`animatedImageWithImages`的方式播放动画，这样做的好处是gif轮播时直接从内存中读取资源就好，降低了CPU的使用，以空间换取流畅度，但是这也会导致当同时加载的gif数量增加时内存问题暴露的尤其明显

## YYWebImage加载gif

`YYWebImage`是每次从缓存中读取需要展示的image，这样就不需要为gif的每一张image都开辟空间，每次都是从一份gif资源中读取每一张image，以一定的帧率从缓冲区中解析出当前的image，这必然需要消耗CPU，所以`YYWebImage`相比较`SDWebImage`更消耗CPU，但是这也带来了内存的大幅降低

## 通过Demo对比

[SDWebImage和YYWebImage加载gif图片对比Demo](https://github.com/luzhiyongGit/WebImageDemo)