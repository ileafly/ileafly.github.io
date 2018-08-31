title: 自动化打包IPA脚本
tags:
  - iOS技巧
  - ''
catgories: iOS
categories:
  - iOS
date: 2016-11-17 09:29:00
---

### 自动化打包IPA原理

当我们用Xcode编译项目时，其实是Xcode执行了`xcodebuild`这条命令。

当我们用Xcode打包项目时，其实是Xcode执行了`xcrun`这条命令。

##### 常用命令

1. xcodebuild -list

   ```shell
   xcodebuild -list [[-project ]|[-workspace ]] [-json]
   ```

2. xcodebuild -project

   ```shell
   xcodebuild [-project] [[-target]...|-alltargets] [-configuration] [-arch]... [-sdk [|]] [-showBuildSettings] [=]... []...
   ```

3. xcodebuild -workspace

   ```shell
   xcodebuild -workspace -scheme [-destination]...[-configuration] [-arch]...[-sdk [|]] [-showBuildSettings] [=]
   ```

4. xcodebuild -exportArchive

   ```shell
   xcodebuild archive -workspace xxx.xcworkspace -scheme xx -archivePath xxx.xcarchive
   ```

   ```shell
   xcodebuild -exportArchive -archivePath xx.xcarchive -exportPath xx -exportFormat ipa
   ```

5. xcrun

   ```shell
   xcrun -sdk iphoneos PackageApplication build/Release -iphoneos/xx.app -o ~/Desktop/xx.ipa
   ```

##### 总结

根据上述命令我们有两种方法来生成我们需要的ipa包。

1. 使用`xcodebuild`命令来编译我们的项目生成app，然后再用`xcrun`将app转成ipa

2. 使用`xcodebuild archive`命令来直接生成我们需要的ipa

   由于在 macos10.12和xcode8的环境下，使用`xcrun`方法会出现一个警告

   `warning: PackageApplication is deprecated, use xcodebuild -exportArchive instead.`

   说明`PackageApplication`已经被弃用了，所以建议选择第二种方法来打包。

##### 项目实践

###### 方案一

```shell
#!/bin/bash

SCHEMENAME=项目的Scheme Name

DATE=`date +%Y%m%d_%H%M`
SOURCEPATH=$( cd "$( dirname %0)" && pwd)
IPAPATH=$SOURCEPATH/AUTOBUILDIPA/$DATE
ARCHIVEPATH=$SOURCEPATH/AUTOBUILDIPA/$DATE/项目名称_$DATE.xcarchive
IPANAME=项目名称_$DATE.ipa

PYG_DOMAIN="http://www.pgyer.com"
PGY_UPLOAD_URL="$PYG_DOMAIN/apiv1/app/upload"
PGY_UKEY="你的蒲公英账号的ukey"
PGY_API_KEY="你的蒲公英账号的api key"

# build 
xcodebuild archive -workspace $SOURCEPATH/项目名称.xcworkspace \
-scheme $SCHEMENAME \
-configuration Release \
-archivePath $ARCHIVEPATH

if [ -e $ARCHIVEPATH ]; then
	echo "xcodebuild Successful"
else 
	echo "error:Build Failed!"
	exit 1
fi

# exportArchive ipa
xcodebuild -exportArchive -archivePath $ARCHIVEPATH \
-exportPath $IPAPATH -exportFormat ipa 

# xcrun .ipa
# xcrun -sdk iphoneos PackageApplication \
#       -v $IPAPATH/Build/Products/Release-iphoneos/$SCHEMENAME.app \
#       -o $IPAPATH/$IPANAME

if [ -e $IPAPATH ]; then
	echo "Create IPA Success!"
	open $IPAPATH
else 
	echo "error: Create IPA failed!"
fi

# upload ipa  上传ipa到蒲公英发布平台
echo "Uploading the ipa file..."
echo $IPAPATH
response=`curl -F "file=@$IPAPATH" -F "uKey=$PGY_UKEY" -F "_api_key=$PGY_API_KEY" $PGY_UPLOAD_URL`
echo $response

# send email 邮件通知同事让他们之间去下载
echo "蒲公英网站上的APP已更新，欢迎更新.下载地址：https://www.pgyer.com/****" | mail -s "ipa更新" ****@163.com,****@163.com,****@163.com,****@163.com
```

###### 方案二

```shell
#!/bin/bash

SCHEMENAME=项目的Scheme Name

DATE=`date +%Y%m%d_%H%M`
SOURCEPATH=$( cd "$( dirname %0)" && pwd)
# IPAPATH=$SOURCEPATH/AUTOBUILDIPA/$DATE/CinemaForiPhoneEnterprise_$DATE.ipa
IPAPATH=$SOURCEPATH/AUTOBUILDIPA/$DATE
ARCHIVEPATH=$SOURCEPATH/AUTOBUILDIPA/$DATE/项目名称_$DATE.xcarchive
IPANAME=项目名称_$DATE.ipa

PYG_DOMAIN="http://www.pgyer.com"
PGY_UPLOAD_URL="$PYG_DOMAIN/apiv1/app/upload"
PGY_UKEY="你的蒲公英账号的ukey"
PGY_API_KEY="你的蒲公英账号的api key"

# build 
xcodebuild -workspace $SOURCEPATH/项目.xcworkspace \
-scheme $SCHEMENAME \
-configuration Release \
clean \
build \
-derivedDataPath $IPAPATH

if [ -e $IPAPATH ]; then
	echo "xcodebuild Successful"
else 
	echo "error:Build Failed!"
	exit 1
fi

# xcrun .ipa
xcrun -sdk iphoneos PackageApplication \
-v $IPAPATH/Build/Products/Release-iphoneos/$SCHEMENAME.app \
-o $IPAPATH/$IPANAME

if [ -e $IPAPATH ]; then
   echo "Create IPA Success!"
    open $IPAPATH
	# upload ipa
   echo "Uploading the ipa file..."
   echo $IPAPATH
      response=`curl -F "file=@$IPAPATH/$IPANAME" -F "uKey=$PGY_UKEY" -F "_api_key=$PGY_API_KEY" $PGY_UPLOAD_URL`
   echo $response

   # send email
   echo "蒲公英网站上的APP已更新，欢迎更新.下载地址：https://www.pgyer.com/****" | mail -s "ipa更新" ****@163.com,****@163.com,****@163.com,****@163.com
else 
	echo "error: Create IPA failed!"
fi
```

写好已经脚本后，将它拷贝到`/usr/local/bin`目录下，添加权限：`chmod 777 xbuild`

执行完以上操作后，就可以在项目目录下直接执行xbuild打包

通过实践发现方案二可以正常使用，方案一打出来的企业版ipa不能正常安装，应该是证书配置问题，还需要调研。

##### 参考链接

[从零开始写个自动打包IPA脚本](http://www.jianshu.com/p/97c97c2ec1ca)

[iOS 自动构建命令——xcodebuild](http://www.jianshu.com/p/3f43370437d2)

[关于 iOS 批量打包的总结](http://ios.jobbole.com/90259/)

[脚本发送邮件的实现方法](http://stackoverflow.com/questions/1822425/sending-mail-from-a-bash-shell-script?rq=1)

