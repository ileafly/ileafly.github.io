title: iOS开发小结之H5与OC交互
tags:
  - iOS进阶
categories:
  - iOS
date: 2017-02-23 14:11:00
---

#### 技术背景
现在混合编程越来越多，为了能给用户提供一个完整的体验，我们就需要解决H5页面与原生程序直接的交互问题，闲话少说，我们直接上代码。我也已经将这篇博客相关的[完整Demo](https://github.com/luzhiyongGit/OC_JS_BridgeDemo)上传到了github上，大家也可以直接clone下来查看。

#### 网页端
##### 核心代码
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>demo</title>
    <script>
        function loginAction() {
            var url = "calloc://login?name=leafly&password=123456";
            document.location = url;
        }
    </script>
</head>
<body>
    <button onClick="loginAction()">登录</button>
</body>
</html>
```
##### 原理分析
在以上的h5代码中，我们主要做了两件事：
1. 页面展现
2. 点击事件的JS处理
当用户点击了登录按钮后，就会调用`loginAction()`方法，在这个方法里，我们加载了一个url，我们将这个url分割成三部分`calloc://`、`login?`、`name=leafly&password=123456`，这三部分共同组成了一个完整的请求地址，第一部分我取了一个名字`calloc`来表达这个地址的核心作用是调起oc代码，当然也可以根据实际情况去修改，第二部分表达了这次调起主要是做login操作，第三部分是login需要的参数

#### 客户端
##### 核心代码
```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    NSURL *url = [request URL];
    if ([[url scheme] isEqualToString:@"calloc"]) {
        // JavaScript与Objective-C交互
        if ([[url host] isEqualToString:@"login"]) {
            // 登录操作
            // 得到一个参数字段
            NSDictionary *params = [self getParams:[url query]];
            [self login:params];
            return NO;
        }
    }
    return YES;
}
```
##### 原理分析
在UIWebView里，每次的地址请求都会触发`- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType`代理，而我们就是通过在这个代理方法里拦截请求地址来实现JS调起OC的作用。通过获取一个请求地址的`scheme`、`host`、`query`这三部分信息，来获取我们所需要的信息。


