title: Xcode统计整个项目代码行数
tags:
  - iOS技巧
  - ''
categories:
  - iOS
date: 2017-02-20 11:55:00
---

一行命令，统计出项目中每个源代码文件的行数以及总行数
```
find . "(" -name "*.m" -or -name "*.mm" -or -name "*.cpp" -or -name "*.h" -or -name "*.rss" ")" -print | xargs wc -l
```
