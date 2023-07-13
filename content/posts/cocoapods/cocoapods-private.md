---
title: "Cocoapods之创建私有库"
date: 2019-07-06T15:49:41+08:00
draft: false
tags: ["Cocoapods"]
categories: ["Cocoapods"]
---


**本文以创建名为"DCServices"的私有库为例。**


1. 打开任意终端工具，进入你的工作目录，输入命令 `pod lib create DCServices`，按照提示回答问题，即可完成lib项目的创建工作。

```
~ pod lib create DCServices                                                                                                                         
Cloning `https://github.com/CocoaPods/pod-template.git` into `DCServices`.
Configuring DCServices template.

------------------------------

To get you started we need to ask a few questions, this should only take a minute.

If this is your first time we recommend running through with the guide:
 - https://guides.cocoapods.org/making/using-pod-lib-create.html
 ( hold cmd and click links to open in a browser. )


What platform do you want to use?? [ iOS / macOS ]
ios

What language do you want to use?? [ Swift / ObjC ]
objc

Would you like to include a demo application with your library? [ Yes / No ]
yes

Which testing frameworks will you use? [ Specta / Kiwi / None ]
none

Would you like to do view based testing? [ Yes / No ]
no

What is your class prefix?
DC

Running pod install on your new library.

Analyzing dependencies
Downloading dependencies
Installing DCServices (0.1.0)
Generating Pods project
Integrating client project

[!] Please close any current Xcode sessions and use `DCServices.xcworkspace` for this project from now on.
Pod installation complete! There is 1 dependency from the Podfile and 1 total pod installed.

 Ace! you're ready to go!
 We will start you off by opening your project in Xcode
  open 'DCServices/Example/DCServices.xcworkspace'

To learn more about the template see `https://github.com/CocoaPods/pod-template.git`.
To learn more about creating a new pod, see `https://guides.cocoapods.org/making/making-a-cocoapod`.

```

2. 创建完成后会自动打开Xcode工程，依次展开左侧列表 `Pods > Development Pods > DCServices` ,找到 `ReplaceMe.m` 文件后删除它。

3. 在 `DCServices` 中创建名为 `DCTest` 的 `.h` 和 `.m` 文件，存放路径选择 `...\DCServices\Classes\` ，完成后添加测试方法。
   


**DCTest.h**

```
@interface DCTest : NSObject

- (void)test;

@end
```

**DCTest.m**

``````
@implementation DCTest

-(void)test {
    NSLog(@"test");
}

@end
``````

4. 回到终端，进入 `DCServices/Example` , 执行 `pod install`。

``````
~/GitHub cd DCServices/Example                                                                                                                                  
~/GitHub/DCServices/Example pod install                                                                                                       
Analyzing dependencies
Downloading dependencies
Generating Pods project
Integrating client project
Pod installation complete! There is 1 dependency from the Podfile and 1 total pod installed.
``````

5. 打开 `Example for DCServices` 中的 `DCViewController.m` 文件，导入DCTest.h,并使用测试方法。

``````
#import "DCViewController.h"
#import "DCTest.h";

@interface DCViewController ()

@end

@implementation DCViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    DCTest *obj = [DCTest new];
    [obj test];
}

- (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}
``````
@end

6. 选择模拟器或真机运行，Console窗口中输出 'test'，到此项目创建完成。
