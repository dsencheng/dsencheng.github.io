---
title: "折腾Docker之Jenkins"
date: 2021-02-04T18:29:44+08:00
draft: false
tags: ["Docker"]
categories: ["Docker"]
---

#### 安装

1.拉取镜像  
`docker pull jenkins/jenkins`  
注意: `docker search jenkins` 显示的`jenkins`,已经废弃，需要使用`jenkins/jenkins`。

2.运行容器，首次启动需要几分钟。

```text
docker run -d \
--name jenkins \
--restart=always \
--privileged=true \
-v $HOME/jenkins_home:/var/jenkins_home \
-p 8080:8080 \
jenkins/jenkins
```

3.打开浏览器，输入`127.0.0.1:8080`或者宿主机的IP地址，首次访问，需要`解锁 Jenkins`。根据提示，打开`/var/jenkins_home/secrets/initialAdminPassword`文件来查看初始化密码。在创建容器时，已经做了目录映射`-v $HOME/jenkins_home:/var/jenkins_home`，所以打开宿主机的`$HOME/jenkins_home/secrets/initialAdminPassword`密码文件查看就可以了。

4.输入初始化密码后，点击`继续`，在`自定义Jenkins`页面，选择`安装推荐的插件`，等待插件安装完成。如果你对Jenkins比较了解，也可以选择手动安装插件。  
ps:安装插件出奇的顺利，以前通过其他方式安装，总是会有插件安装失败。而且页面自动中文，不是英文的。

5.插件安装完成后，`创建第一个管理员用户`，填写用户名，密码，邮箱，点击`保存并完成`。

6.`实例配置`页面不做修改，直接`保存并完成`，最后点击`开始使用Jenkins`。好了，到这里就完成了，开始使用吧。

**注：docker里的Jenkins是Linux环境，不是Mac环境。**

##### 更新Jenkins

```text
# 进入容器
> docker exec -it -u root jenkins bash

# 进入jenkins.war目录
> cd /usr/share/jenkins/

# 下载新包，因为当前目录已有正在使用的jenkins.war文件，新的文件命名为jenkins.war.1
> wget http://mirrors.jenkins.io/war/latest/jenkins.war
# 上面的地址较慢，可以在Jenkins的升级提示里复制新的下载链接，比如下面的这个链接。
> wget https://updates.jenkins.io/download/war/2.280/jenkins.war
# 或者手动下载，替换容器内的文件。

# 更改新旧文件名
> ls
jenkins.war  jenkins.war.1  ref
> mv jenkins.war jenkins.war.backup
> ls 
jenkins.war.1  jenkins.war.backup  ref
> mv jenkins.war.1 jenkins.war
> ls
jenkins.war  jenkins.war.backup  ref

# 退出容器
> exit

# 重启容器
> docker restart jenkins
```

#### 添加节点

iOS任务需要Mac环境，而Docker里的Jenkins是Linux环境，所以还需要添加宿主机为节点。我的宿主机是Mac mini，这里的套娃逻辑如下:

```text
Mac > Docker > Jenkins容器(Linux) > 新建节点 > Mac
```

打开Jenkins的`系统管理` > `节点管理` > `新建节点`。  
填写`节点名称`，这个名称在任务配置中指定节点需要填写。  
`远程工作目录`，填写项目存放路径，例如`/Users/dsen/.jenkins`。  
`用法`，选择`只允许绑定到这台机器的job`。  
`启动方式`，选择`Launch agents via SSH`，`主机`填写IP，如`192.168.1.45`，不需要添加前缀。`Credentials`添加节点的登录账户和密码，也就是这台Mac电脑的登录账户。`Host Key Verification Strategy`选择`Non verifying Verifiycation Strategy`。

`节点属性` 勾选 `环境变量`，新增键值，键输入 `PATH`, 值输入`/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/dsen/.rvm/bin:/usr/local/bin/pod`。如果不知道这个值，可以打开节点的终端，输入`echo $PATH`获得。

#### 新建任务

新建一个任务，名字随便输入，选择`构建一个自由风格的软件项目`，点击确定。在项目的配置里，`构建` > `增加构建步骤` > `执行shell`，填写`uname -a`，点击保存。返回项目页面后，点击`立即构建`，构建完成后，可以在构建历史查看`控制台输出`。  
如果是`Linux 67c7f669fe0b 4.19.121-linuxkit #1 SMP Tue Dec 1 17:50:32 UTC 2020 x86_64 GNU/Linux`，说明当前任务是在Linux系统执行的。  
打开项目的设置，勾选`限制项目的运行节点`，输入之前添加的节点名称，保存之后，再次构建，查看`控制台输出`，如果是`Darwin localhost 19.6.0 Darwin Kernel Version 19.6.0: Sun Jul 5 00:43:10 PDT 2020; root:xnu-6153.141.1~9/RELEASE_X86_64 x86_64`，说明当前任务是在Mac系统执行的。
