title: iOS项目添加自定义字体
tags:
  - iOS基础
  - ''
categories:
  - iOS
date: 2016-11-28 11:23:00
---

有时候系统的字体无法满足我们的要求，这时就需要我们添加自定义字体，下面就来总结一下添加自定义字体的流程和注意点。

####添加字体
首先，我们需要将需要自定义的字体文件添加到我们的项目中
![添加字体](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/iOS项目添加自定义字体/iOS项目添加自定义字体_1.png)
####配置plist
添加完成后就需要配置plist文件告诉项目你添加了自定义字体
![配置plist](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/iOS项目添加自定义字体/iOS项目添加自定义字体_2.png)
####打印字体名称
```
// 打印字体
        let familyNames: Array = UIFont.familyNames()
        
        for familyName in familyNames {
            let fontNames = UIFont.fontNamesForFamilyName(familyName)
            
            for fontName in fontNames {
                print("Font:"+fontName)
            }
        }
```
![打印字体名称](https://github.com/luzhiyongGit/luzhiyongGit.github.io/raw/hexo/images/iOS项目添加自定义字体/iOS项目添加自定义字体_3.png)
####使用新字体
打印完所有的字体名称后，我们需要找到新字体的名称，这个比较考验眼力，需要说明的是导入的字体名称与你的字体文件名不存在必然关联，所以找起来比较麻烦，本项目中导入的字体名称是`Woxxdrgnxpjndtijvopiqfvieua` `Rlwnvqnxbaojotrussvqutrmalm` `Gjfhtoyvcitsqewokxubciqwsnn` `Daipwzrbqhqhabefqqcwoentynp`。
调用方法：
```
[UIFont fontWithName:@"Woxxdrgnxpjndtijvopiqfvieua" size:9];
```


