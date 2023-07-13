---
title: "Fastlane"
date: 2020-06-13T18:06:18+08:00
draft: false
tags: []
categories: []
---

[**Fastlane官方文档**](https://docs.fastlane.tools/)

1、安装

```text
sudo gem install fastlane -NV

错误-1: 
You don't have write permissions into the /usr/bin directory.
解法方法：
sudo gem install fastlane -n /usr/local/bin
```

2、初始化,根据情况选择。可以选择自定义，稍后自己编辑文件。

```text
fastlane init
```

3、执行

```text
fastlane custom

fastlane有多种执行方式，可以直接使用actions，custom是自定义的lane
```

#### Fastlane文件示例

**Appfile**

```text
app_identifier "com.xxx.opensdk" 
apple_id "dsencheng@gmail.com" 
team_id "xxxxxxxx"

```

**Fastfile** [文档链接](https://docs.fastlane.tools/actions/)

```text
default_platform(:ios)

platform :ios do
  desc "fastlane 打包测试"
  lane :AppStore do
    cocoapods
    increment_build_number
    scan
    gym
    deliver
    testflight
  end
end
```

#### 插件

```text
蒲公英fastlane插件
fastlane add_plugin pgyer

Facebook测试脚本
brew install xctool
```

参考文章  
[iOS 持续交付之 Fastlane](https://juejin.im/post/5a7b10bb6fb9a0636263bfd5#heading-14)  
[使用 Fastlane 上传 App 到蒲公英](https://www.pgyer.com/doc/view/fastlane)

##### 问题记录

```text
increment_build_number 报错
```

> 解决方法: build settings > Current project version 设置为整数

  

* * *

```text
$ bundle exec fastlane beta update_description:"大多数更新说明文字描述"

[!] Could not find action, lane or variable 'option'. Check out the documentation for more details: https://docs.fastlane.tools/actions
```

> 这个问题很广泛，解决方法也不同。
> 
> ##### 情况一,不熟悉`Ruby`语法。
> 
> 执行`fastlane`时，需要传递更新说明参数，但编写`Fastfile`时丢失参数`option`，导致报错。
> 
> ```text
> # 示例
> default_platform(:ios)
> 
> platform :ios do
>   desc "打包测试"
>   # lane :beta do 
>   # 这里需要补代码 |options| ，默认是没有的
>   lane :beta do |options|
>     #更新说明
>     update_description = option[:update_description]
>   end
> end
> ```

* * *

20200630更新

**Xcode管理Provisioning Profile文件，打包上传到蒲公英后，出现部分设备下载后黑色图标，点击提示‘无法安装’。**

> 增加打包参数 `export_xcargs: "-allowProvisioningUpdates"`  
> [fastlane文档链接](https://docs.fastlane.tools/codesigning/xcode-project/)
> 
> ```text
> # 示例
> platform :ios do
>   desc "Description of what the lane does"
>   lane :sdkdemo do
>   desc "开始打包"
>   gym(
>     scheme: "SDKDemo",
>     output_name: "SDKDemo.ipa",
>     clean: true,
>     export_method: "ad-hoc",
>     export_xcargs: "-allowProvisioningUpdates",
>     )
>   desc "上传到蒲公英"
>   pgyer(api_key: "xxx", user_key: "xxx")
>   end
> end
> ```
