
---

title: xopen快捷脚本

date: 2019-06-11 10:10:16 +0800

categories: iOS

tags: [教程]

---

自定义xopen快捷脚本，在终端中快速打开项目<br />![xopen.gif](https://cdn.nlark.com/yuque/0/2019/gif/183307/1562028112900-78c7a90a-d135-43da-bda3-a8f633731c6d.gif#align=left&display=inline&height=410&name=xopen.gif&originHeight=410&originWidth=656&size=702849&status=done&width=656)
<a name="dWZuy"></a>
#### 详细教程

1. 创建 `xopen` 文件
1. 编辑 `xopen` 内容

```ruby
#!/usr/bin/env ruby
require 'shellwords'

proj = Dir['*.xcworkspace'].first
proj = Dir['*.xcodeproj'].first unless proj

if proj
   puts "Opening  #{proj}"
   `open #{proj}`
else
  puts "No  xcworkspace|xcproj  file  found"
end
```

3. 将 `xopen` 文件移入 `/usr/local/bin` 目录下
3. 添加权限 `chmod 777 xopen` 
3. 在终端中，cd到项目目录下，执行xopen

