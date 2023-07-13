---
title: "Cocoapods之私有库发布及使用"
date: 2010-07-13T16:18:22+08:00
draft: false
tags: ["Cocoapods"]
categories: ["Cocoapods"]
---

本文以github为例  
1、打开你的github主页，选择 `New repository` .

> a. `Repository name` 填写 `DCSpecs`。  
> b. `Description` 可以不填写。  
> c. 选择 `Private` 或 `Public`。  
> d. 勾选 `Initialize this repository with a README`。  
> e. 点击 `Create repository` 按钮。

2、打开终端，进入目录，执行命令

```text
pod repo add DCSpecs https://github.com/dsencheng/DCSpecs.git
```

3、执行成功后，可以进入目录查看 `~/.cocoapods/repos/` ，是否有`DCSpecs`文件夹。

4、在`DCServices.podspec`文件目录，推送私有库。

```text
pod repo push DCSpecs DCServices.podspec
```

> `DCServices`是`code`。  
> `DCSpecs` 是管理和记录。它会记录`DCServices`的`0.1.0`，`0.1.1` ... `1.5.3`。  
> 写命令时，注意区分`DCServices` 和 `DCSpecs`

5、在项目中引入私有库。打开你开发的项目，如果没有`Podfile`，请先初始化。添加内容

```text
source 'https://cdn.cocoapods.org/'
source 'https://github.com/dsencheng/DCSpecs.git'

platform :ios, ‘9.0’

target “your_target_name”   
    do
        pod 'DCServices'
    end
```

打开终端在当前项目目录执行`pod install`完成!
