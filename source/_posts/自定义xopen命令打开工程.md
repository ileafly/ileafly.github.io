title: 自定义xopen命令打开工程
tags:
  - iOS技巧
  - ''
categories:
  - iOS
date: 2016-10-08 16:15:00
---

通常我们在终端中对代码进行版本控制或者使用CocoaPods命令安装完依赖库后，都需要手动去打开工程。对于追求快捷和逼格的我们来说怎样才能进一步提升我们的逼格呢？

今天就跟大家分享一个通过ruby脚本来帮助我们在终端中直接打开工程的方法。

如果大家足够细心的话应该能注意到CocoaPods管理的项目后缀是`projectname.xcworkspace`而普通工程的后缀是`projectname.xcodeproj`,下面我们就通过ruby脚本来判断后缀，帮助我们优先打开`projectname.xcworkspace`。

       #!/usr/bin/env ruby
 
       require 'shellwords'
 
       proj = Dir['*.xcworkspace'].first
       proj = Dir['*.xcodeproj'].first unless proj
 
       if proj
         puts "Opening #{proj}"
         `open #{proj}`
       else
         puts "No xcworkspace|xcproj file found"
       end
       
将以上内容命名为`xopen`保存，并将文件拷贝到`/usr/local/bin`目录下，添加权限：`chmod 777 xopen`

执行完以上操作后你就可以方便的使用`xopen`命令来打开工程啦。