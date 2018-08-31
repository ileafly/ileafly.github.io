title: iOS复制字符串到剪贴板
tags:
  - iOS技术方案
categories:
  - iOS
date: 2017-12-14 19:22:00
---
最近有一个小功能，需要自动帮用户将一些字符串复制到剪贴板中，其实实现比较简单，因为这个功能相对比较生僻，做一个简单记录

```
UIPasteboard *pasteboard = [UIPasteboard generalPasteboard];
pasteboard.string = @"你需要复制的字符串内容";
```