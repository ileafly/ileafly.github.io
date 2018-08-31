title: UIWebView加载本地html
tags:
  - iOS基础
  - ''
categories:
  - iOS
date: 2017-02-16 14:19:00
---

UIWebView除了能加载网页地址外还可以加载本地html，加载的方式主要有两种

#### 读取本地html内容，加载内容
```
NSString *mainBundlePath = [[NSBundle mainBundle] bundlePath];
NSURL *baseURL = [NSURL URLWithString:mainBundlePath];
NSString *htmlPath = [NSString stringWithFormat:@"%@/index.html", mainBundlePath];
NSString *htmlContent = [NSString stringWithContentsOfFile:htmlPath encoding:NSUTF8StringEncoding error:nil];
[_webView loadHTMLString:htmlString baseURL:baseURL];
```
#### 通过本地html地址加载内容
```
NSString *mainBundlePath = [[NSBundle mainBundle] bundlePath];
NSURL *baseURL = [NSURL URLWithString:mainBundlePath];
NSURLRequest *request = [NSURLRequest requestWithURL:baseURL cachePolicy:NSURLRequestReloadIgnoringCacheData timeoutInterval:(NSTimeInterval)10.0 ];
[_webView loadRequest:request];

```
有些情况下，我们不仅要能加载本地html，还需要给本地html传递参数，这种情况下我们只能通过方法二来进行参数传递
```
NSString *mainBundlePath = [[NSBundle mainBundle] bundlePath];
NSURL *baseURL = [NSURL URLWithString:mainBundlePath];
NSString *queryString = [NSString stringWithFormat:@"%@/index.html?key=value", mainBundlePath];
NSURL *queryURL = [NSURL URLWithString:queryString];
NSURLRequest *request = [NSURLRequest requestWithURL:queryURL cachePolicy:NSURLRequestReloadIgnoringCacheData timeoutInterval:(NSTimeInterval)10.0 ];
[_webView loadRequest:request];
```

