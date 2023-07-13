---
title: "正确设置WKWebView的UserAgent指南"
date: 2019-09-10T17:16:38+08:00
draft: false
tags: ["iOS"]
categories: []
---


开发者在使用`WKWebView`时会遇到需要自定义`UserAgent`的情况，比如web需要区分iOS、Android、WeChat等等。  
网上也有这大量如何修改`UserAgent`的方法，但总是会出现`UserAgent`无效的情况，比如[iOS12 设置 WKWebView 的 UserAgent](https://awesome-tips.gitbook.io/ios/webview/content)

掉进坑里无数次后发现，系统早已准备好，只是我没有发现，擦干菜鸟的泪水。。。

#### 系统需要高于 iOS9

###### WKWebView正确设置、修改UserAgent

```text
WKWebViewConfiguration* webViewConfig = WKWebViewConfiguration.new;
NSLog(@"修改前 = %@",webViewConfig.applicationNameForUserAgent);
//正确设置UserAgent的代码
webViewConfig.applicationNameForUserAgent = @"我是自定义Agent";
NSLog(@"修改后 = %@",webViewConfig.applicationNameForUserAgent);
//正确使用设计的初始化方法，不要使用其他初始化方法。
WKWebView* webView = [[WKWebView alloc] initWithFrame:self.view.frame configuration:webViewConfig];
//注意：下面这句代码是查看，不是设置UserAgent
[webView evaluateJavaScript:@"navigator.userAgent" completionHandler:^(NSString * _Nullable originUserAgent, NSError * _Nullable error) {
    NSLog(@"完整UserAgent = %@",originUserAgent);
}];
```

###### 输出

```text
修改前 = Mobile/16A404
修改后 = 我是自定义Agent
完整UserAgent = Mozilla/5.0 (iPhone; CPU iPhone OS 12_0_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) 我是自定义Agent
```

你可能会好奇 `Mobile/16A404` 是什么？  
我也没有找到具体含义，如果你知道可以告诉我。  
**友情提醒**，`applicationNameForUserAgent` 应该追加，不是直接赋值。
