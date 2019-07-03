
---

title: iOS封装SDK

date: 2019-07-01 13:19:08 +0800

tags: []

---
日常开发中，我们可能需要封装一些SDK或者使用一些别人封装的SDK。这里总结一下SDK的封装流程。

<a name="sH1aJ"></a>
## 基本概念
通常我们遇到的SDK有两种：.framework和.a文件。首先，我们需要弄清楚这两种类型的文件有什么区别。

在讲清楚.framework和.a文件的区别前，我们需要先了解另外两个概念：静态库和动态库。

<a name="MsdOd"></a>
#### 静态库
静态库即是静态链接库，在编译时会直接拷贝一份，复制到目标程序里。静态库的代码就相当于是目标程序的一部分。

<a name="qmzIU"></a>
#### 动态库
动态库在编译时并不会被拷贝到目标程序中，目标程序中只会存储指向动态库的引用，等到App启动时，动态库才会被真正加载进来。

<a name="BfxbV"></a>
#### 静态库 VS 动态库

- 静态库在编译时将代码拷贝进目标程序中，会导致目标程序的体积增加。被多次使用就会在内存中存在多份冗余拷贝。
- 动态库在App冷启动时需要加载动态链接库，进行rebase指针调整和bind符号绑定等工作，会导致App的启动时间增长。由系统动态加载到内存，供App调用，系统只加载一次，多个程序共用，节省内存。
- iOS中静态库的形式：.a 和 .framework
- iOS中动态库的形式：.dylib 和 .framework

<a name="0nmXZ"></a>
#### 苹果的动态库发展史
在iOS 8之前，iOS平台不支持自定义动态库，开发者可以使用的动态库只能是苹果自家的 `UIKit.framework` 、 `Foundation.framework` 等。这种限制的原因是出于安全考虑，因为iOS应用都是运行在沙盒中，不同的程序之间不能共享代码，动态下载代码是苹果明令禁止的，既然没办法发挥动态库的优势，动态库也就没有存在的必要了。<br />在iOS 8之前，也有一些第三方提供的.framework文件存在，但是它们本质上都是静态库，只不过通过一些方法进行了包装，相比较.a文件使用更方便一些。<br />iOS 8/Xcode 6 推出后，iOS平台添加了动态库的支持，支持开发者有条件地创建和使用动态库，这种动态库叫做 `Cocoa Touch Framework` ，但是这种动态framework与系统的framework还是有很大区别的。系统的framework在编译时不需要拷贝进目标程序中，而 `Cocoa Touch Framework` 在打包和提交App时会被放到app bundle中，运行在沙盒里，不同的app就算使用了相同的framework也是会有多份的框架被分别签名、打包和加载，因此苹果又把这种framework称为在[Embedded Framework](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html)。

`Cocoa Touch Framework` 的推出主要是为了解决两个问题：

1. 从iOS 8开始的扩展开发
1. Swift在早期不支持编译为静态库

<a name="RiNwW"></a>
## .a VS .framework

- .a是纯二进制文件 .framework中除了二进制文件还有资源文件
- .a文件不能直接使用，需要引入.h文件配合 .framework文件包含了.h文件和其他文，可以直接使用

<a name="oYQRv"></a>
## 制作教程
关于如何制作.a文件或.framework的教程网上特别多，这里我就不做具体描述。这里主要总结一下几个关键步骤。
<a name="jLvrh"></a>
### 架构
苹果的架构分为两大类：模拟器架构和真机架构

1. 模拟器架构
- i386  32位架构 4S ~ 5
- x86_64 64位架构 5S ~ 现在的机型
2. 真机架构
- armv7 32位架构 3GS ~ 4S
- armv7s 特殊架构 5 ~ 5C （此架构有问题，有的程序变得更快，有的程序变得更慢）
- arm64 64位架构 5S ~ 现在的机型

<a name="sD83I"></a>
#### 合成架构
使用模拟器编译出来的包是模拟器架构，使用真机编译出来的包是真机架构。可以使用 `lipo -info` 查看当前包的架构。<br />真机和模拟器架构合成的好处是调试会非常方便，缺点是体积会变大，一般而言，SDK都需要合成架构方便使用者使用。<br />合成架构的命令：<br />`lipo -create simulator.a device.a -output name.a` <br />合成.framework文件的架构命令也是使用 `lipo -create` 区别点是合成的是.framework文件内部的可执行文件。

<a name="c315M"></a>
### 脚本打包
手动打包虽然能满足我们的需求，但是利用脚本打包会带来几点优势：

1. 提高效率，原本繁琐的打包流程，只需要执行一下脚本就能完成
1. 统一规范，繁琐的操作流程，依赖个人去完成，难免会出现差错，利用脚本可以确保准确性
1. 易于使用，利用脚本打包，即使是新人也可以非常容易的上手，降低沟通成本

既然脚本打包有这么多优点，接下来就总结一下实现脚本打包的过程：

<a name="kNnmW"></a>
#### 脚本打包思路

1. 利用 `xcodebuild` 分别打包模拟器架构和真机架构
1. 利用 `lipo -create` 合并模拟器和真机架构
1. 如果是framework的合并，需要将合并了的二进制可执行文件复制到framework中
1. 复制文件到指定目录下，并打开文件夹

<a name="ARByx"></a>
#### 详细脚本
```bash
# 项目名
POD_PROJECT_NAME=${PROJECT_NAME}
CONFIGURATION=Release
#自定义的用来存放最后合并的framework
UNIVERSAL_OUTPUTFOLDER=${SRCROOT}/Products/${CONFIGURATION}-universal

WORKSPACE_NAME=${PROJECT_NAME}.xcworkspace
SCHEME_NAME=${PROJECT_NAME}

#clean build是先清除原来的build
xcodebuild -workspace ${WORKSPACE_NAME} -scheme ${SCHEME_NAME} -sdk iphonesimulator -configuration "${CONFIGURATION}" clean build
xcodebuild -workspace ${WORKSPACE_NAME} -scheme ${SCHEME_NAME} -sdk iphoneos -configuration "${CONFIGURATION}" clean build
echo "${WORKSPACE_NAME}"
echo "${PROJECT_NAME}"

#先移除原来的
rm -rf "${UNIVERSAL_OUTPUTFOLDER}"
mkdir -p "${UNIVERSAL_OUTPUTFOLDER}"

#清空build文件夹
rm -rf "${BUILD_DIR}"

# 生成OS库
xcodebuild build -workspace ${WORKSPACE_NAME} -scheme ${POD_PROJECT_NAME} ONLY_ACTIVE_ARCH=NO -configuration ${CONFIGURATION} -sdk iphoneos BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}"

# 生成模拟器i386库
xcodebuild build -workspace ${WORKSPACE_NAME} -scheme ${POD_PROJECT_NAME} -configuration ${CONFIGURATION} -sdk iphonesimulator -arch i386 BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}"

# 重命名i386库
mv ${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${POD_PROJECT_NAME}/${POD_PROJECT_NAME}.framework ${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${POD_PROJECT_NAME}/${POD_PROJECT_NAME}i386.framework

# 生成模拟器x86_64库
xcodebuild build -workspace ${WORKSPACE_NAME} -scheme ${POD_PROJECT_NAME} -configuration ${CONFIGURATION} -sdk iphonesimulator -arch x86_64 BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}"

#合并模拟器i386和x86_64架构
lipo -create  "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}/${PROJECT_NAME}i386.framework/${PROJECT_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}/${PROJECT_NAME}.framework/${PROJECT_NAME}" -output "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}/${PROJECT_NAME}.framework/${PROJECT_NAME}"

#因为framework的合并,lipo只是合并了最后的二进制可执行文件,所以其它的需要我们自己复制过来
#拷贝文件到指定的目录
cp -R "${BUILD_DIR}/${CONFIGURATION}-iphoneos/${PROJECT_NAME}/${PROJECT_NAME}.framework" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework"

#合并模拟器（i386/x86_64）和真机（armv7/arm64）的架构
lipo -create  "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}/${PROJECT_NAME}.framework/${PROJECT_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphoneos/${PROJECT_NAME}/${PROJECT_NAME}.framework/${PROJECT_NAME}" -output "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/${PROJECT_NAME}"

open "${UNIVERSAL_OUTPUTFOLDER}"
```

<a name="spveF"></a>
#### 利用Aggregate提供快捷方法

1. 在当前项目下新建Aggregate Target

![](https://cdn.nlark.com/yuque/0/2019/png/183307/1561969431794-4718bee3-b5cc-4391-bd6b-4db925cb9816.png#align=left&display=inline&height=721&originHeight=721&originWidth=1000&size=0&status=done&width=1000)

2. 添加Run Script

![](https://cdn.nlark.com/yuque/0/2019/png/183307/1561969454208-d3c888d5-6adf-4f05-9832-244d0ff4be5a.png#align=left&display=inline&height=343&originHeight=343&originWidth=1000&size=0&status=done&width=1000)

3. 在Run Script Phases输入脚本内容

![](https://cdn.nlark.com/yuque/0/2019/png/183307/1561969490482-5f40c393-5660-4601-a4ec-0b1d2b1c3a4b.png#align=left&display=inline&height=509&originHeight=509&originWidth=1000&size=0&status=done&width=1000)

4. 编译Aggregate Target 完成脚本打包

