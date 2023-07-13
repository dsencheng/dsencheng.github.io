---
title: "Jenkins基本安装与使用"
date: 2019-08-13T17:14:11+08:00
draft: false
tags: ["Jenkins"]
categories: []
---

# 准备工作

1.  开发环境，如Java、Ruby、MacOS等。
2.  开发工具，如Xcode、Android Studio、IntelliJ IDEA等。
3.  证书管理，如推送证书、商店证书、手机配置文件等。
4.  其他依赖工具，如Git、Fastlane、Cocoapods、Maven等。

**建议：准备完成后，创建新项目或使用已有项目，打包一次验证是否可以成功打包。**

##### 本文使用的**操作系统为MacOS**。

# Jenkins安装

#### [Jenkins中文安装文档](https://www.jenkins.io/zh/doc/book/installing/)

1、**推荐安装方式**，安装完成自动运行。**不推荐PKG方式安装**

```text
brew install jenkins-lts

# 启动/停止/重启

brew services start jenkins-lts
brew services stop jenkins-lts
brew services restart jenkins-lts
```

2、**不推荐**。简单方式，下载jenkins-lts.war，cd到文件所在目录，使用命令启动

```text
java -jar jenkins-lts.war
```

3、浏览器访问: [http://localhost:8080](http://localhost:8080) 按引导教程完成配置。若局域网内无法访问，参考下面的问题记录。

# Jenkins使用

1.  如果是英文，建议先安装中文插件  
    登录Jenkins后，依次点击左侧的“齿轮(Manage Jenkins)”，然后找到右侧“拼图(Manage Plugins)”，继续点击第二个选项卡“Available”，在搜索框输入“Localization: Chinese (Simplified)”，在插件前面的多选框里打钩，然后点击下面的“Install without restart”按钮，等待下载安装完成后重启，如果没有重启，可以手动重启Jenkins。
    
2.  创建一个FreeStyle的任务。  
    选择“新建任务”，输入任务名称，选择“构建自由风格的软件项目”，点击“确定”。
    
3.  新任务的配置工作。  
    “描述”简要说明下项目。  
    “源码地址”填写好Git或SVN的代码仓库地址，同时需要提前在“凭据”中，添加访问代码仓库的账号密码。SVN可能需要安装插件“Subversion”。  
    “构建”，“增加构建步骤”，“执行shell”，添加项目的构建脚本，基本的配置工作就算完成了。  
    **任务配置介绍**
    

```text
1.General
    项目描述，设置参数化构建，文件存留天数、数量等。
2.源码管理
    git、svn
3.构建触发器
    提供webhook，轮询，定时等触发构建条件，便于打包自动化。
4.构建环境
    设置构建的参数、打包证书(需要插件)等。
5.构建
    提供shell、插件等多种构建方式，可以执行多个构建方式，并且有顺序。比如先执行shell，然后执行插件构建。
    注:我这里是调用fastlane来完成。fastlane的工作内容就是打包，Jenkins这里就是提供了一个网页的壳。
6.构建后操作
    提供通知，邮件，推送等构建完成后任务。
```

1.  build 立即构建。

# Jenkins插件

安装完成后建议重启  
汉化插件：Localization: Chinese (Simplified)  
证书管理插件：Keychains and Provisioning Profiles Management  
Xcode打包插件：Xcode integration

# 添加节点

1.首先添加目标节点的登录账户。进入“凭据”，“添加凭据”，"类型"选择“Username with password”，“用户名”填写目标节点的SSH登录账户名，“密码”填写账户对应的密码。

2.进入“系统管理” - “节点管理” - ”新建节点“ ，填写节点名称，点击”确定“。  
”远程工作目录“这里填写目标节点的项目目录，比如`/Users/your_login_name/.jenkins`。  
”启动方式“，选择”launch agents via ssh“，如果没有该选项，需要安装”SSH Build Agents“ 插件。  
”主机“填写ip地址。  
”Credentials“选择刚刚添加的账户。  
“Host Key Verification Strategy”选择“Non verifying Verification Strategy”。  
  
点击保存完成。

附：如果在Jenkins构建过程中出现命令执行错误，但在终端可以执行命令，就要思考一下是否需要在“环境变量”中添加使用命令的环境。  
例如使用Cocoapods，需要在配置节点，“节点属性”，“环境变量”添加键值。  
如果仍然报错，依次打开“系统管理”，“系统设置”，“全局属性”，“环境变量”，添加同样的键值。  
键：PATH  
值：/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/your\_login\_name/.rvm/bin:/usr/local/bin/pod

# 问题记录

#### 问题：无法更新插件

管理Jenkins > 插件管理 > 高级  
把`URL`里的地址`https`改为`http`

* * *

#### 问题：局域网内，无法通过ip+port访问

关闭Jenkins服务，打开文件夹 `/usr/local/Cellar/jenkins/` 或者 `/usr/local/Cellar/jenkins-lts/` ,找到`homebrew.mxcl.jenkins.pist`这个文件.

```text
--httpListenAddress=127.0.0.1
修改为
--httpListenAddress=0.0.0.0
```

重启Jenkins，局域网其他用户就可以访问了。

* * *

#### 问题：fastlane: command not found

参照安装方法添加路径`sudo gem install fastlane -n /usr/local/bin`，在Jenkins > 节点管理 > 配置(右侧⚙图标) > 设置环境变量。  
查看当前环境变量,

```text
echo $PATH
```

* * *

### 问题： git fetch 443

> 使用私有库

```text
Started by user dsen
Running as SYSTEM
Building in workspace /Users/Shared/Jenkins/Home/workspace/PrivateTest
using credential 380f80e8-7d4d-4bfd-8505-e739d1f1c86f
 > git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://gitee.com/gdyh21/PrivateTest.git # timeout=10
Fetching upstream changes from https://gitee.com/gdyh21/PrivateTest.git
 > git --version # timeout=10
using GIT_ASKPASS to set credentials 
 > git fetch --tags --force --progress https://gitee.com/gdyh21/PrivateTest.git +refs/heads/*:refs/remotes/origin/* # timeout=2
ERROR: Timeout after 2 minutes
ERROR: Error fetching remote repo 'origin'
hudson.plugins.git.GitException: Failed to fetch from https://gitee.com/gdyh21/PrivateTest.git
    at hudson.plugins.git.GitSCM.fetchFrom(GitSCM.java:894)
    at hudson.plugins.git.GitSCM.retrieveChanges(GitSCM.java:1161)
    at hudson.plugins.git.GitSCM.checkout(GitSCM.java:1192)
    at hudson.scm.SCM.checkout(SCM.java:504)
    at hudson.model.AbstractProject.checkout(AbstractProject.java:1208)
    at hudson.model.AbstractBuild$AbstractBuildExecution.defaultCheckout(AbstractBuild.java:574)
    at jenkins.scm.SCMCheckoutStrategy.checkout(SCMCheckoutStrategy.java:86)
    at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:499)
    at hudson.model.Run.execute(Run.java:1815)
    at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:43)
    at hudson.model.ResourceController.execute(ResourceController.java:97)
    at hudson.model.Executor.run(Executor.java:429)
Caused by: hudson.plugins.git.GitException: Command "git fetch --tags --force --progress https://gitee.com/gdyh21/PrivateTest.git +refs/heads/*:refs/remotes/origin/*" returned status code 143:
stdout: 
stderr: 
    at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandIn(CliGitAPIImpl.java:2042)
    at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:1761)
    at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.access$400(CliGitAPIImpl.java:72)
    at org.jenkinsci.plugins.gitclient.CliGitAPIImpl$1.execute(CliGitAPIImpl.java:442)
    at hudson.plugins.git.GitSCM.fetchFrom(GitSCM.java:892)
    ... 11 more
ERROR: Error fetching remote repo 'origin'
Finished: FAILURE
```

PKG卸载  
'/Library/Application Support/Jenkins/Uninstall.command'

* * *

### 问题：errSecInternalComponent

报错内容如下，\*号为主机名或工程名。

```text
/Users/***/Library/Developer/Xcode/DerivedData/***-hhnneuyrewxlcchalpeniwciweuw/Build/Products/Release-iphoneos/***.framework: errSecInternalComponent
Command /usr/bin/codesign failed with exit code 1

** BUILD FAILED **
```

这个问题大部分原因是访问keychain失败造成的。

当前场景是一台Windows主机通过Jenkins添加一台Mac Mini节点，iOS的打包任务绑定在Mac节点。Mac本身有Jenkins可以正常打包，但执行Windows主机的Jenkins打包任务就会报错。

`Jenkins(windows) -> Jenkins Node(Mac Mini)`  
这是因为Windows通过SSH访问Mac，打包任务没有Keychains的访问权限。

经过google之后，找到了解决办法。  
1.首先打开Mac Mini的命令行，输入 `security list-keychains`，查看当前已有的keychain。下面是示例：

```text
"/Users/****/Library/Keychains/login.keychain-db"
"/Users/****/JenkinsTmp/login.keychain"
"/Library/Keychains/System.keychain"
```

2.打开`钥匙串(Keychains)`，依次选择左侧的`登录(login)`，我的`证书(My Certificates)`，选择你的证书，点左侧三角展开证书，双击`证书key`，然后选择`访问控制(Access Control)`，勾选`允许所有应用程序访问此项目(Allow all applications to access this item)`，点击`存储更改(Save Changes)`，输入登录密码，完成。

3.在Jenkins构建之前，解锁keychain，然后打包。

`security unlock-keychain -p <login password> <keychain path>`

以下为示例。  
**123456替换成登录密码!**

```text
cd YourProject
pod install
# ssh登录需要解锁keychain，可以依次尝试 security list-keychains 命令列出的路径。
#security unlock-keychain -p 123456 login.keychain
#security unlock-keychain -p 123456 ~/Library/Keychains/login.keychain-db
security unlock-keychain -p 123456 ~/JenkinsTmp/login.keychain
# 执行打包
xcodebuild -workspace 'YourProject.xcworkspace' -scheme YourTarget
```

如果还是有问题，建议重启。

参考链接：  
[Xcode errSecInternalComponent](https://stackoverflow.com/questions/24023639/xcode-command-usr-bin-codesign-failed-with-exit-code-1-errsecinternalcomponen)  
[Unable to unlock the keychain](https://stackoverflow.com/questions/35517496/unable-to-unlock-the-keychain)

# 打包报错 error: exportArchive: Code signing "xxxx" failed.

```text
Error Domain=IDEDistributionPipelineErrorDomain Code=0 "Code signing "xxxx" failed." UserInfo={NSLocalizedDescription=Code signing "xxxx" failed., NSLocalizedRecoverySuggestion=View distribution logs for more information.}
```

需要解锁kenchain `security unlock-keychain -p <login password>`
