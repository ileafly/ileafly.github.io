
---

title: xopen快捷脚本

date: 2019-06-11 10:10:16 +0800

categories: iOS

tags: [iOS]

---

自定义xopen快捷脚本，在终端中快速打开项目

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

