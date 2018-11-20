
title: iOS自动化打包脚本
date: 2017-04-20 11:55:00 +0800
tags: [iOS]
categories: 开发技巧
---

<span data-type="color" style="color:#F5222D">注意：</span>此方案打包OC项目没有问题，打包swift pod的项目，在export时会失败，目前还没找到解决方案，建议使用fastlane进行打包。[fastlane指南](http://www.ileafly.com/2018/11/08/yuque/fastlane指南/)

iOS自动打包主要使用`xcodebuild`，在终端输入`xcodebuid --help`可以查看`xcodebuild`的参数。

#### <a name="opaffn"></a>xcodebuild核心语法
##### <a name="80i4av"></a>无workspace的工程
```powershell
xcodebuild [-project  name.xcodeproj][[-target targetname]...|-alltargets][-configuration  configurationname][-sdk  [sdkfullpath  |  sdkname]][action  ...][buildsetting=value  ...][-userdefault=value  ...]

xcodebuild [-project  name.xcodeproj] -scheme schemename [[-destination  destinationspecifier]...][-destination-timeout  value][-configuration  configurationname][-sdk  [sdkfullpath  |  sdkname][action  ...][buildsetting =value  ...][-userdefault=value  ...]
```
##### <a name="sh64gz"></a>有workspace的工程
```powershell
xcodebuild -workspace name.xcworkspace -scheme schemename [[-destination  destinationspecifier]...][-destination-timeout  value][-configuration  configurationname][-sdk  [sdkfullpath  |  sdkname]][action  ...][buidsetting=value  ...][-userdefault=value  ...]
```

## <a name="z4urwc"></a>具体shell脚本
#### <a name="chqbxy"></a>无workspace的工程
```powershell
#author by leafly

#工程名字(Target名字)
Project_Name="Target名字，系统默认和工程名字一样"
#配置环境，Release或者Debug
Configuration="Release"

Date=`date +%Y%m%d_%H%M`
SourcePath=$( cd "$( dirname %0)" && pwd)

#AdHoc版本的Bundle ID
AdHocBundleID="com.xxx"
#AppStore版本的Bundle ID
AppStoreBundleID="com.xxx"
#enterprise的Bundle ID
EnterpriseBundleID="com.xxx"

# ADHOC
#证书名#描述文件
ADHOCCODE_SIGN_IDENTITY="iPhone Distribution: xxxx"
ADHOCPROVISIONING_PROFILE_NAME="xxxx-xxxx-xxxx-xxxx"

#AppStore证书名#描述文件
APPSTORECODE_SIGN_IDENTITY="iPhone Distribution: xxxx"
APPSTOREROVISIONING_PROFILE_NAME="xxxx-xxxx-xxxx-xxxx"

#企业(enterprise)证书名#描述文件
ENTERPRISECODE_SIGN_IDENTITY="iPhone Distribution: xxxxx"
ENTERPRISEROVISIONING_PROFILE_NAME="xxxx-xxxx-xxxx-xxxx"

#加载各个版本的plist文件
ADHOCExportOptionsPlist=./ADHOCExportOptionsPlist.plist
AppStoreExportOptionsPlist=./AppStoreExportOptionsPlist.plist
EnterpriseExportOptionsPlist=./EnterpriseExportOptionsPlist.plist

ADHOCExportOptionsPlist=${ADHOCExportOptionsPlist}
AppStoreExportOptionsPlist=${AppStoreExportOptionsPlist}
EnterpriseExportOptionsPlist=${EnterpriseExportOptionsPlist}

echo "~~~~~~~~~~~~选择打包方式(输入序号)~~~~~~~~~~~~~~~"
echo "  1 appstore"
echo "  2 adhoc"
echo "  3 enterprise"

# 读取用户输入并存到变量里
read parameter
sleep 0.5
method="$parameter"

# 判读用户是否有输入
if [ -n "$method" ]
then

#clean下
xcodebuild clean -xcodeproj ./$Project_Name/$Project_Name.xcodeproj -configuration $Configuration -alltargets

    if [ "$method" = "1" ]
    then

#appstore脚本
xcodebuild -project $Project_Name.xcodeproj -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-appstore.xcarchive clean archive build  CODE_SIGN_IDENTITY="${APPSTORECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${APPSTOREROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AppStoreBundleID}"
xcodebuild -exportArchive -archivePath build/$Project_Name-appstore.xcarchive -exportOptionsPlist $AppStoreExportOptionsPlist -exportPath ~/Desktop/$Project_Name-appstore.ipa
    elif [ "$method" = "2" ]
    then
#adhoc脚本
xcodebuild -project $Project_Name.xcodeproj -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-adhoc.xcarchive clean archive build CODE_SIGN_IDENTITY="${ADHOCCODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ADHOCPROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AdHocBundleID}"
xcodebuild -exportArchive -archivePath build/$Project_Name-adhoc.xcarchive -exportOptionsPlist $ADHOCExportOptionsPlist -exportPath ~/Desktop/$Project_Name-adhoc.ipa
    elif [ "$method" = "3" ]
    then
#企业打包脚本
xcodebuild -project $Project_Name.xcodeproj -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-enterprise.xcarchive clean archive build CODE_SIGN_IDENTITY="${ENTERPRISECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ENTERPRISEROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${EnterpriseBundleID}"
xcodebuild -exportArchive -archivePath build/$Project_Name-enterprise.xcarchive -exportOptionsPlist $EnterpriseExportOptionsPlist -exportPath ~/Desktop/$Project_Name-enterprise.ipa
else
    echo "参数无效...."
    exit 1
    fi
fi

#upload to pgyer
if [ -e $IpaPath ]; then
  #statements
  echo "Uploading the ipa file..."
  echo $IpaPath
      response=`curl -F "file=@$IpaPath/$IpaName" -F "uKey=$PGY_UKEY" -F "_api_key=$PGY_API_KEY" $PGY_UPLOAD_URL`
  echo response

  # send email
  echo "蒲公英网站上的APP已更新，欢迎更新.下载地址：https://www.pgyer.com/xxxx" | mail -s "测试包更新成功" lu.zhiyong@sohu.com
else
  echo "创建ipa失败"
fi 
```

#### <a name="uxwfdr"></a>有workspace的工程
```powershell
#author by leafly

#蒲公英账号信息
PYG_DOMAIN="http://www.pgyer.com"
PGY_UPLOAD_URL="https://qiniu-storage.pgyer.com/apiv1/app/upload"
PGY_UKEY="196be923698226391e474dabefd2243d"
PGY_API_KEY="906192e72b27666926ba256da0f574dc"

#工程名字(Target名字)
Project_Name="Target名字，系统默认和工程名字一样"
#workspace的名字
Workspace_Name="WorkSpace名字"
#配置环境，Release或者Debug,默认release
Configuration="Release"

#路径配置
Date=`date +%Y%m%d_%H%M`
SourcePath=$( cd "$( dirname %0)" && pwd)
IpaPath=$SourcePath/AutoBuildIpa/$Date
ArchivePath=$SourcePath/AutoBuildIpa/$Date/$Project_Name_$Date.xcarchive
IpaName=$Project_Name.ipa

#AdHoc版本的Bundle ID
AdHocBundleID="com.xxxx"
#AppStore版本的Bundle ID
AppStoreBundleID="com.xxxx"
#enterprise的Bundle ID
EnterpriseBundleID="com.xxxx"

# ADHOC证书名#描述文件
ADHOCCODE_SIGN_IDENTITY="iPhone Distribution: xxxxx"
ADHOCPROVISIONING_PROFILE_NAME="xxxxx-xxxx-xxx-xxxx"

#AppStore证书名#描述文件
APPSTORECODE_SIGN_IDENTITY="iPhone Distribution: xxxxx"
APPSTOREROVISIONING_PROFILE_NAME="xxxxx-xxxx-xxx-xxxx"

#企业(enterprise)证书名#描述文件
ENTERPRISECODE_SIGN_IDENTITY="iPhone Distribution: xxxx"
ENTERPRISEROVISIONING_PROFILE_NAME="xxxxx-xxxx-xxx-xxxx"

#加载各个版本的plist文件
ADHOCExportOptionsPlist=./ADHOCExportOptionsPlist.plist
AppStoreExportOptionsPlist=./AppStoreExportOptionsPlist.plist
EnterpriseExportOptionsPlist=./EnterpriseExportOptionsPlist.plist

ADHOCExportOptionsPlist=${ADHOCExportOptionsPlist}
AppStoreExportOptionsPlist=${AppStoreExportOptionsPlist}
EnterpriseExportOptionsPlist=${EnterpriseExportOptionsPlist}

echo "~~~~~~~~~~~~选择打包方式(输入序号)~~~~~~~~~~~~~~~"
echo "  1 adHoc"
echo "  2 AppStore"
echo "  3 Enterprise"

# 读取用户输入并存到变量里
read parameter
sleep 0.5
method="$parameter"

# pod install

pod install

# 判读用户是否有输入
if [ -n "$method" ]
then
    if [ "$method" = "1" ]
    then
#adhoc脚本
xcodebuild -workspace $Workspace_Name.xcworkspace -scheme $Project_Name -configuration $Configuration -archivePath $ArchivePath clean archive build CODE_SIGN_IDENTITY="${ADHOCCODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ADHOCPROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AdHocBundleID}"
xcodebuild  -exportArchive -archivePath $ArchivePath -exportOptionsPlist ${ADHOCExportOptionsPlist} -exportPath $IpaPath

    elif [ "$method" = "2" ]
    then
#appstore脚本
xcodebuild -workspace $Workspace_Name.xcworkspace -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-appstore.xcarchive archive build CODE_SIGN_IDENTITY="${APPSTORECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${APPSTOREROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AppStoreBundleID}"
xcodebuild  -exportArchive -archivePath build/$Project_Name-appstore.xcarchive -exportOptionsPlist ${AppStoreExportOptionsPlist} -exportPath ~/Desktop/$Project_Name-appstore.ipa

    elif [ "$method" = "3" ]
    then
#企业打包脚本
xcodebuild -workspace $Workspace_Name.xcworkspace -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-enterprise.xcarchive archive build CODE_SIGN_IDENTITY="${ENTERPRISECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ENTERPRISEROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${EnterpriseBundleID}"
xcodebuild  -exportArchive -archivePath build/$Project_Name-enterprise.xcarchive -exportOptionsPlist ${EnterpriseExportOptionsPlist} -exportPath ~/Desktop/$Project_Name-enterprise.ipa
    else
    echo "参数无效...."
    exit 1
    fi
fi

#upload to pgyer
if [ -e $IpaPath ]; then
  #statements
  echo "Uploading the ipa file..."
  echo $IpaPath
      response=`curl -F "file=@$IpaPath/$IpaName" -F "uKey=$PGY_UKEY" -F "_api_key=$PGY_API_KEY" $PGY_UPLOAD_URL`
  echo response

  # send email
  echo "蒲公英网站上的APP已更新，欢迎更新.下载地址：https://www.pgyer.com/xxxx" | mail -s "测试包更新成功" lu.zhiyong@sohu.com
 else
  echo "创建ipa失败"
 fi 
```

### <a name="bkeyhl"></a>plist文件
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>provisioningProfiles</key>
	<dict>
		<key>bundle  ID</key>
		<string>xxxxxxxx-xxxxxx-xxxxxxxxx</string>
	</dict>
	<key>method</key>
	<string>ad-hoc</string>
</dict>
</plist>
```

### <a name="2afoai"></a>用法
将shell脚本命名为`xpublish`，将文件存放到`/usr/local/bin`目录下，`chmod 777 xpublish`修改`xpublish`的权限。进入项目目录，执行`xpublish`即可。


