---
title: "制作一个CocoaPods私有库"
date: 2019-07-04T14:15:53+08:00
draft: false
tags: ["Cocoapods"]
categories: ["Cocoapods"]
---


## 创建pod库

 在GitHub上创建一个空仓库，不要初始化。`PrivatePods 是你的私有库名字。如果已经初始化，下面创建lib要在对应的目录里执行。

```
https://github.com/your_github_name/PrivatePods.git
```
 
1. 打开终端，进入到要建立私有库工程的目录，执行下面的命令 。执行命令过程中会询问几个问题，按自己所需回答。

```
pod lib create PrivatePods
```

创建完成后，会自动打开项目。


2. 删除`ReplaceMe.m`文件。
3. 添加一个测试类，写一个测试方法。注意，文件要放到 PrivatePods/PrivatePods/Classes 目录中。
例如：

```
ATest.h
+(void)testMethod;
ATest.m
+(void)testMethod {
    NSLog(@"%s", __func__);
}
如果有已存在源码，文件也放入 /Classes 文件下。
```

4. 打开终端，进入`Example`文件夹，执行`pod install`，安装依赖项。

```
这里需要注意的是，只要新增加类、资源文件或依赖的第三方库都需要重新运行`pod install`来进行更新。
```

5. 在 `XXViewController.m`中引用测试类 `#import "ATest.h"`，运行看是否符合预期结果. `+[ATest testMethod]`

## 提交推送到远端

提交源码到GitHub，并打tag，不使用tag会出现问题。这里尽量使用命令行，一些软件是不支持添加tag的。`0.1.0` 版本号保持和`DCPrivatePods.podspec`的`s.version = '0.1.0'`保持一致。

```
$ git add -A 
$ git commit -m "0.1.0 发布"
$ git tag 0.1.0
$ git push remote origin master --tag

其他命令
$ git remote add origin https://github.com/your_github_name/repo_name.git
$ git pull origin master
$ git push -u origin master
$ git tag push
```

打开终端，执行命令，验证`podspec`文件。

```
pod spec lint   
PrivatePods.podspec passed validation.
# 注意
pod lib lint 是验证pod库。
pod spec lint 是验证podspec文件。
```

直到验证成功，如果有任何的error，都需要解决掉。warn 可以使用下面的命令忽略。

私有库可能出现URL警告，可增加 `--allow-warnings` 忽略警告。

```
- WARN  | url: The URL (https://github.com/your_github_name/repo_name.git) is not reachable. 
```

## 发布私有库
在github上创建一个新的仓库。例如：

```
https://github.com/your_git_name/MyPrivateSpecs.git
```
注意:创建时勾选初始化，空仓库是没有master分支的。会提示如下错误

```
您的配置中指定要合并远程的引用 'refs/heads/master'，
但是没有获取到这个引用。
```


创建完成后执行命令。

```
$ pod repo add MyPrivateSpecs https://github.com/your_github_name/MyPrivateSpecs.git
```

执行成功后，可以进入目录查看 `~./cocoapods/repos/` ，看到`MyPrivateSpecs`文件夹是说明成功了。


这个仓库是管理你的私有库，你可能创建一个或多个代码库，每个代码库又会不断的迭代发布新的版本。那么这个Spec就像记录表一样，记录你的所有代码私有库，以及每个版本。使用时，在你的项目Podfile文件中添加`source 'spec仓库地址'`，然后`pod '私有库的名字'` ,就可以使用了。


发布你的私有库，如果没通过可以尝试加`--allow-warnings`忽略警告.

```
$ pod repo push MyPrivateSpecs DCPrivatePods.podspec
```

在自己项目中引用私有库，执行`pod install`完成!

```
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/your_github_name/MyPrivateSpecs.git'
platform :ios, ‘8.0’

target “your_target_name” do
    pod 'DCPrivatePods'
end
```

私有库依赖另一个私有库时, `lint`、`push`操作可能需要增加`--sources`参数。用法如下：

```
pod lib lint --sources=https://github.com/dsencheng/DCSpecs.git,https://cdn.cocoapods.org/ --use-libraries --allow-warnings --verbose
```

## 问题记录

在私有库中，引入第三方framework报如下错误

```    
ld: framework not found AliRTCSdk
```

解决办法:写完整路径。 

```
s.vendored_frameworks = 'AliModule/Classes/AliRTCSdk.framework'`
```

验证私有库时报错

```
ERROR | [iOS] xcodebuild: Returned an unsuccessful exit code. You can use `--verbose` for more information.
```

解决办法

报错信息


```
Showing All Messages
:-1: Multiple commands produce '/Users/dsen/Library/Developer/Xcode/DerivedData/Ali-dufrvubbsfygbsfskqpkqsmffkhv/Build/Products/Debug-iphonesimulator/Ali/Ali.framework/Info.plist':
1) Target 'Ali' (project 'Pods') has copy command from '/Users/dsen/GitHub/Ali/Ali/Classes/UTDID.framework/Info.plist' to '/Users/dsen/Library/Developer/Xcode/DerivedData/Ali-dufrvubbsfygbsfskqpkqsmffkhv/Build/Products/Debug-iphonesimulator/Ali/Ali.framework/Info.plist'
2) Target 'Ali' (project 'Pods') has process command with output '/Users/dsen/Library/Developer/Xcode/DerivedData/Ali-dufrvubbsfygbsfskqpkqsmffkhv/Build/Products/Debug-iphonesimulator/Ali/Ali.framework/Info.plist'
```


解决办法

```
去掉 Pods &gt; Project &gt; Build Setting &gt; Valid Architectures &gt; x86_64 
```


报错信息 


```
does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE)
```


解决办法


```
在xxx_Example 中修改 enable bitcode = false
```




报错信息


```
no rule to process file
```


解决办法

```
s.source_files = 'AliModule/Classes/**/*'
修改为
s.source_files = 'AliModule/Classes/**/*.{h,m}'
忽略其他类型文件
```





报错信息


```
ERROR | [iOS] file patterns: The `vendored_frameworks` pattern did not match any file.
```


解决办法:暂无





报错信息


```
[!] Unable to find a specification for `DCView` depended upon by `DCLib`

You have either:
 * out-of-date source repos which you can update with `pod repo update` or with `pod install --repo-update`.
 * mistyped the name or version.
 * not added the source repo that hosts the Podspec to your Podfile.
```


解决办法


写错了 `source`,应该写Spec的地址，错写了lib库的地址。




`pod lib lint`出现i386 x86_64模拟器问题



1 加命令参数`pod lib lint --skip-import-validation` 

2 podspec添加参数
`s.pod_target_xcconfig = { 'VALID_ARCHS[sdk=iphonesimulator*]' =&gt; '' }`

`s.pod_target_xcconfig = {
        'ARCHS[sdk=iphonesimulator*]' =&gt; '$(ARCHS_STANDARD_64_BIT)'
    }`


