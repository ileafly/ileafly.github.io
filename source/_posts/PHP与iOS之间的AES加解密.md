title: PHP与iOS之间的AES加解密
date: 2017-03-06 18:12:13
tags: Secret
---

## 前言
在项目开发过程中，为了保证传输数据的安全性，我们经常要对传输的内容进行加密处理，以增加别人破解的成本。常用的加密算法有很多，今天我们先围绕`AES`加密算法进行一个使用总结

### AES算法介绍
`AES`是高级加密标准(Advanced Encryption Standard)的缩写，在密码学中又被称为Rijndael加密法，如果想对`AES`的背景有更多的了解可以移步到[维基百科-高级加密标准](https://zh.wikipedia.org/wiki/高级加密标准)

`AES`加密时需要统一四个参数：
- 密钥长度 (Key Size)
- 加密模式 (Cipher Mode)
- 填充方式 (Padding)
- 初始向量 (Initialization Vector)

由于前后端开发所使用的语言不统一，导致经常出现前后端之间互相不能解密的情况出现，其实，无论什么语言系统，`AES`的算法总是相同的，导致结果不一致的原因在于**上述的四个参数不一致**，下面就来了解一下这四个参数的含义

#### 密钥长度
`AES`算法下，key的长度有三种：128、192、256 bits，三种不同密钥长度就需要我们传入的key传入不同长度的字符串，例如我们选择AES-128,那我们定的key需要是长度为16的字符串

#### 加密模式
`AES`属于块加密，块加密中有CBC、ECB、CTR、OFB、CFB等几种工作模式，为了保持前后端统一，我们选择ECB模式

#### 填充方式
由于块加密只能对特定长度的数据块进行加密，因此CBC、ECB模式需要在最后一数据块加密前进行数据填充

#### 初始向量
使用除ECB以外的其他加密模式均需要传入一个初始向量，其大小与Block Size相等

## 代码实现
### PHP端代码实现
```
<?php
/*
 * 定义类cryptAES 专用于AES加解密
 * 初始化时传入密钥长度、加密Key、初始向量、加密模式四个字段
 */
class cryptAES
{
    public $iv = null;
    public $key = null;
    public $bit = 128;
    private $cipher;

    public function __construct($bit, $key, $iv, $mode)
    {
        if(empty($bit) || empty($key) || empty($iv) || empty($mode))
        {
            return NULL;
        }

        $this->bit = $bit;
        $this->key = $key;
        $this->iv = $iv;
        $this->mode = $mode;

        switch($this->bit)
        {
            case 192 : $this->cipher = MCRYPT_RIJNDAEL_192; break;
            case 256 : $this->cipher = MCRYPT_RIJNDAEL_256; break;
            default : $this->cipher = MCRYPT_RIJNDAEL_128;
        }
        switch($this->mode)
        {
            case 'ecb' : $this->mode = MCRYPT_MODE_ECB; break;
            case 'cfb' : $this->mode = MCRYPT_MODE_CFB; break;
            case 'ofb' : $this->mode = MCRYPT_MODE_OFB; break;
            case 'nofb' : $this->mode = MCRYPT_MODE_NOFB; break;
            default : $this->mode = MCRYPT_MODE_CBC;
        }
    }

        /*
         * 加密数据并返回
         */
    public function encrypt($data)
    {
        $data = base64_encode(mcrypt_encrypt($this->cipher, $this->key, $data, $this->mode, $this->iv));
        return $data;
    }

        /*
         * 解密数据并返回
         */
    public function decrypt($data)
    {
        $data = mcrypt_decrypt($this->cipher, $this->key, base64_decode($data), $this->mode, $this->iv);
        $data = rtrim(rtrim($data), "\x00..\x1F");
        return $data;
    }
}
```

### iOS端代码实现
```
// NSString+AESSecurity.h
@interface NSString (AESSecurity)

+ (NSString *)encrypyAES:(NSString *)content key:(NSString *)key;

+ (NSString *)descryptAES:(NSString *)content key:(NSString *)key;

@end

// NSString+AESSecurity.m
#import "NSString+AESSecurity.h"

#import <CommonCrypto/CommonCrypto.h>

// 初始向量
NSString *const kInitVector = @"0123456789";

// 密钥长度
size_t const kKeySize = kCCKeySizeAES128;

@implementation NSString (AESSecurity)

+ (NSString *)encrypyAES:(NSString *)content key:(NSString *)key {
    
    NSData *contentData = [content dataUsingEncoding:NSUTF8StringEncoding];
    NSUInteger dataLength = contentData.length;
    
    // 为结束符'\\0' +1
    char keyPtr[kKeySize + 1];
    memset(keyPtr, 0, sizeof(keyPtr));
    [key getCString:keyPtr maxLength:sizeof(keyPtr) encoding:NSUTF8StringEncoding];
    
    size_t encryptSize = dataLength + kCCBlockSizeAES128;
    void *encryptedBytes = malloc(encryptSize);
    size_t actualOutSize = 0;
    
    NSData *initVector = [kInitVector dataUsingEncoding:NSUTF8StringEncoding];
    
    CCCryptorStatus cryptStatus = CCCrypt(kCCEncrypt,
                                          kCCAlgorithmAES128,
                                          kCCOptionPKCS7Padding | kCCOptionECBMode, // 加密模式
                                          keyPtr,
                                          kKeySize,
                                          initVector.bytes,
                                          contentData.bytes,
                                          dataLength,
                                          encryptedBytes,
                                          encryptSize,
                                          &actualOutSize);
    
    if (cryptStatus == kCCSuccess) {
        // 对加密后的数据进行 base64 编码
        return [[NSData dataWithBytesNoCopy:encryptedBytes length:actualOutSize] base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithLineFeed];
    }
    free(encryptedBytes);
    return nil;
}

+ (NSString *)descryptAES:(NSString *)content key:(NSString *)key {//
    // 把 base64 String 转换成 NSData
    NSData *contentData = [[NSData alloc] initWithBase64EncodedString:content options:NSDataBase64DecodingIgnoreUnknownCharacters];
    NSUInteger dataLength = contentData.length;
    
    char keyPtr[kKeySize + 1];
    memset(keyPtr, 0, sizeof(keyPtr));
    [key getCString:keyPtr maxLength:sizeof(keyPtr) encoding:NSUTF8StringEncoding];
    
    size_t decryptSize = dataLength + kCCBlockSizeAES128;
    void *decryptedBytes = malloc(decryptSize);
    size_t actualOutSize = 0;
    
    NSData *initVector = [kInitVector dataUsingEncoding:NSUTF8StringEncoding];
    
    CCCryptorStatus cryptStatus = CCCrypt(kCCDecrypt,
                                          kCCAlgorithmAES,
                                          kCCOptionPKCS7Padding | kCCOptionECBMode, // 加密模式
                                          keyPtr,
                                          kKeySize,
                                          initVector.bytes,
                                          contentData.bytes,
                                          dataLength,
                                          decryptedBytes,
                                          decryptSize,
                                          &actualOutSize);
    
    if (cryptStatus == kCCSuccess) {
        return [[NSString alloc] initWithData:[NSData dataWithBytesNoCopy:decryptedBytes length:actualOutSize] encoding:NSUTF8StringEncoding];
    }
    
    free(decryptedBytes);
    return nil;
}

@end

```

### 注意点

在iOS上，字符串经过加解密后可能会在数据中添加一些[操作符](http://baike.baidu.com/view/1112575.htm)这会导致我们想进一步处理解密后的字符串时会处理失败，例如，当我们想将解密后的json字符串转成字典时，可能会抛出`Garbage at End`的错误，解决方案如下：

1. 将字符串中的所有控制符替换成空字符
```
NSString *newStr = [oldStr stringByTrimmingCharactersInSet:[NSCharacterSet controlCharacterSet]];
```
2. 将处理后的字符串进行json序列化操作
```
NSError *err = nil;
            
NSData *jsondata = [str dataUsingEncoding:NSUTF8StringEncoding];
            
NSArray *arr = [NSJSONSerialization JSONObjectWithData:jsondata options:NSJSONReadingMutableLeaves error:&err];
```
---
附上相关模块的代码

[AESSecurity-PHP](https://github.com/luzhiyongGit/AESSecurity-PHP)

[AESSecurity-iOS](https://github.com/luzhiyongGit/AESSecurity-iOS)

欢迎star和fork~


