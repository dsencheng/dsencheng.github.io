---
title: "Cocoapods之pod命令"
date: 2019-11-26T16:24:10+08:00
draft: false
tags: ["Cocoapods"]
categories: ["Cocoapods"]
---

`pod init`  
初始化

`pod install`  
安装/更新库

`pod lib lint`  
验证lib合法性

`pod lib lint --verbose --use-libraries --allow-warnings -- sources='私有仓库repo地址,https://github.com/CocoaPods/Specs'`  
验证多仓库lib

`pod spec lint`  
验证spec合法性

`pod spec lint --verbose --use-libraries --allow-warnings -- sources='私有仓库repo地址,https://github.com/CocoaPods/Specs'`  
验证多仓库spec

`pod repo push DCSpecs name.podspec --use-libraries --allow-warnings`  
向DCSpecs推送DCServices库

`pod repo push MCLib MCAppKit.podspec --verbose --use-libraries --allow-warnings`  
强制推送

以上命令均可选参数，  
`--use-libraries`使用静态库  
`--allow-warnings`允许警告
