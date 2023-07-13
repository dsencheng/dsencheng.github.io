---
title: "Cocoapods之私有库代码同步"
date: 2019-07-08T16:11:17+08:00
draft: false
tags: ["Cocoapods"]
categories: ["Cocoapods"]
---


本文项目名`DCServices`,仓库为github。

1. 打开你的github主页，选择 `New repository` .

> a. `Repository name` 填写 `DCServices`。  
> b. `Description` 可以不填写。  
> c. 选择 `Private` 。  
> d. 点击 `Create repository` 按钮。

__注意: 不需要勾选 `Initialize this repository with a README`。__

2. 首次提交代码。之前已经创建好 `DCServices` 私有库，打开终端进入项目目录。

```
~/GitHub/DCServices git add -A  
~/GitHub/DCServices git commit -m "首次提交"                                                                                               
[master 1b2baff] 首次提交
 34 files changed, 1396 insertions(+), 4 deletions(-)
 create mode 100644 DCServices/Classes/DCTest.h
 create mode 100644 DCServices/Classes/DCTest.m
 ...
 ...
create mode 100644 Example/Pods/Target Support Files/Pods-DCServices_Tests/Pods-DCServices_Tests.release.xcconfig
~/GitHub/DCServices git tag 0.1.0                                                                                                              
~/GitHub/DCServices git remote add origin https://github.com/dsencheng/DCServices.git                                                    
~/GitHub/DCServices git push -u origin master --tag                                                                                           
Enumerating objects: 86, done.
Counting objects: 100% (86/86), done.
Delta compression using up to 4 threads
Compressing objects: 100% (77/77), done.
Writing objects: 100% (86/86), 28.19 KiB | 1.23 MiB/s, done.
Total 86 (delta 20), reused 0 (delta 0)
remote: Resolving deltas: 100% (20/20), done.
To https://github.com/dsencheng/DCServices.git
 * [new branch]      master -> master
 * [new tag]         0.1.0 -> 0.1.0
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

3. 这时刷新你的github项目主页,已经发生了变化。接下来创建一些其他文件，比如创建一个 `DCServerKit.h` 文件，文件路径 `...\DCServices\Classes\`。在.h文件里添加之前创建的测试文件 `#import "DCTest.h"`。

4. 打开终端，进入`Example` 目录，执行命令。

```
~/GitHub/DCServices/Example pod install                                                                                              
Analyzing dependencies
Downloading dependencies
Generating Pods project
Integrating client project
Pod installation complete! There is 1 dependency from the Podfile and 1 total pod installed.
```

4、修改 `DCViewController.m` 文件，运行查看结果。

```
#import <DCTest.h> 改为 #import <DCServerKit.h>
```

5、打开 `DCServices.podspec` 文件，增加版本号。版本号的作用，在Podfile文件中， `pod 'xxx', ~> 2.1.0` 可以指定某个版本。

```text
s.version          = '0.1.0'
修改位
s.version          = '0.1.1'
```

6、执行 `pod lib lint` 验证库是否有错误或警告，并修复问题。

7、同步代码，注意`git tag 0.1.1` 命令保持数字和 `s.version = '0.1.1'` 一致。

```text
~/GitHub/DCServices git add -A                                                                                                     
~/GitHub/DCServices git commit -m "0.1.1版本"                                                                                         
[master 29e9131] 0.1.1版本
 5 files changed, 135 insertions(+), 116 deletions(-)
 create mode 100644 DCServices/Classes/DCServerKit.h
~/GitHub/DCServices git tag 0.1.1                                                                                                           
~/GitHub/DCServices git push origin master --tag                                                                                      
Enumerating objects: 28, done.
Counting objects: 100% (28/28), done.
Delta compression using up to 4 threads
Compressing objects: 100% (14/14), done.
Writing objects: 100% (15/15), 2.58 KiB | 879.00 KiB/s, done.
Total 15 (delta 9), reused 0 (delta 0)
remote: Resolving deltas: 100% (9/9), completed with 9 local objects.
To https://github.com/dsencheng/DCServices.git
   1b2baff..29e9131  master -> master
 * [new tag]         0.1.1 -> 0.1.1
```

8、执行 `pod spec lint` 验证spec文件是否有错误，并修复问题。

9、验证成功后，执行 `pod repo push DCSpecs DCServices.podspec` 向管理repo推送更新.

踩坑指南  
1、如果你是私有库，通常都会报警告, 这个警告可以忽略。

```text
WARN  | url: The URL (https://github.com/dsencheng/DCServices) is not reachable.
```

比如

```text
pod spec lint --allow-warnings
pod repo push DCSpecs DCServices.podspec --allow-warnings
```

2、如果私有库中包含静态库，需要增加 `--use-libraries` 参数。 比如

```text
pod repo push DCSpecs DCServices.podspec --use-libraries --allow-warnings
```

3、如果私有库libA依赖私有库modelB，该如何解决呢？

> 第一步，在 `Podfile` 中添加 `source`
> 
> ```text
> source 'https://cdn.cocoapods.org/'
> source 'https://github.com/dsencheng/DCSpecs.git'
> ```
> 
> 第二步，验证时也增加`source`, 参数 `--use-libraries`包含静态库，参数`--allow-warnings`忽略警告，参数`--verbose`输出详细信息。
> 
> ```text
> pod lib lint --sources=https://github.com/dsencheng/DCSpecs.git,https://cdn.cocoapods.org/ --use-libraries --allow-warnings --verbose
> ```

4、强制推送 `--force`
