---
title: "Cocoapods之podspec文件"
date: 2019-07-17T16:21:07+08:00
draft: false
tags: ["Cocoapods"]
categories: ["Cocoapods"]
---


```text
#名字
s.name             = 'Chat2Love'
#版本号，更新发布内容最好更新版本号
s.version          = '0.1.0'
#简介说明
s.summary          = 'A short description of Chat2Love.'
#详细说明
s.description      = ''
#主页，通常是github项目地址
s.homepage         = 'https://github.com/dsencheng/Chat2Love'
#屏幕快照链接
# s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
#协议
s.license          = { :type => 'MIT', :file => 'LICENSE' }
#作者
s.author           = { 'dsencheng' => 'dsencheng@gmail.com' }
#源码地址
s.source           = { :git => 'https://github.com/dsencheng/Chat2Love.git', :tag => s.version.to_s }
#社交信息
# s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'
#开发版本
s.ios.deployment_target = '8.0'
#暴露的文件，举例：`/*.h` 仅暴露.h文件。 `/*.{h,m} ` 仅暴露.h和.m文件
s.source_files = 'Chat2Love/Classes/**/*'
#资源文件
# s.resource_bundles = {
#   'Chat2Love' => ['Chat2Love/Assets/*.png']
# }
#暂未用到
# s.public_header_files = 'Pod/Classes/**/*.h'
#需要使用到的系统库
# s.frameworks = 'UIKit', 'MapKit'
# 需要依赖的三方库，支持指定版本,不支持指定地址。可在podfile文件中指定地址。
# s.dependency 'AFNetworking', '~> 2.3'
#静态库
#s.static_framework = true
# 依赖的系统静态库
#s.library = 'c++', 'resolv'
# build setting中的配置
#s.ios.pod_target_xcconfig = { 'OTHER_LDFLAGS' => '-lObjC', 'ENABLE_BITCODE' => false, }
# 子库,会有不同的文件夹
#  s.subspec 'Sub' do | b |
#      b.source_files = 'Chat2Love/Classes/**/*'
#   end
# 引用其他的库，不建议样写，pod中引用framework文件会报错。
# s.ios.vendored_frameworks = 'Ali/Classes/AliRTCSdk.framework','Ali/Classes/UTDID.framework'
# 引用其他的库，建议写法
#  s.subspec 'AliRTC' do | sub_rtc |
#      sub_rtc.ios.vendored_frameworks = 'Ali/Classes/AliRTCSdk.framework','Ali/Classes/UTDID.framework'
#   end

```
