---
title: "折腾Docker之Gitlab"
date: 2021-01-31T18:26:35+08:00
draft: false
tags: ["Docker"]
categories: ["Docker"]
---

### 三、安装Gitlab

##### 1、打开终端，搜索gitlab.

```text
> docker search gitlab
```

##### 2、拉取镜像

```text
> docker pull gitlab/gitlab-ce
```

##### 3.创建容器

创建之前，建议打开`Docker Desktop` - `Preferences` - `Resources`  
`Advanced` 页签把 CPU 改成 4， Memory 改成 4GB。  
`File sharing` 全部删除，然后添加 `/Users` 或 `/Users/用户名`，以减少容器访问磁盘。  
终端执行命令：

```text
docker run --detach \
--hostname gitlab.interotc.com \
--name gitlab \
--restart always \
--publish 10443:443 --publish 10080:80 --publish 10022:22 \
--volume $HOME/gitlab/config:/etc/gitlab \
--volume $HOME/gitlab/logs:/var/log/gitlab \
--volume $HOME/gitlab/data:/var/opt/gitlab \
--privileged=true \
gitlab/gitlab-ce:13.8.1-ce.0
```

###### 命令参数解释

`--detach` 后台运行  
`--hostname gitlab.interotc.com` 容器的主机名或域名，即192.168.10.10或者 gitlab.example.com  
`--name gitlab` 容器的名字  
`--restart always` 设置重启方式，always 代表一直开启，服务器开机后也会自动开启的  
`--publish 10080:80` 把本机端口映射到容器端口，即访问宿主机10080端口为容器的80端口  
`--volume $HOME/gitlab/config:/etc/gitlab` 宿主机目录映射到容器目录  
`--privileged=true` 提升容器的`root`用户拥有真正root权限，否则只是普通用户权限  
`gitlab/gitlab-ce:13.8.1-ce.0` 这里我指定了镜像的tag版本，如果你是最新的版本，可以改为`gitlab/gitlab-ce`

创建成功后会显示一串字符，例如

```text
> docker run --detach \                                                                                                                          
--hostname gitlab.interotc.com \
--name gitlab \
--restart always \
--publish 10443:443 --publish 10080:80 --publish 10022:22 \
--volume $HOME/gitlab/config:/etc/gitlab \
--volume $HOME/gitlab/logs:/var/log/gitlab \
--volume $HOME/gitlab/data:/var/opt/gitlab \
--privileged=true \
gitlab/gitlab-ce:13.8.1-ce.0
af8fc3a2499f6ce47d52a64871dd2027bfcb3e17c7ee23fc64026c6f236e09ab
```

执行命令  
`docker ps` 或者 `docker container ls`

```text
> docker ps                                                                                                                                      
CONTAINER ID   IMAGE                          COMMAND             CREATED              STATUS                                 PORTS                                                                  NAMES
af8fc3a2499f   gitlab/gitlab-ce:13.8.1-ce.0   "/assets/wrapper"   About a minute ago   Up About a minute (health: starting)   0.0.0.0:10022->22/tcp, 0.0.0.0:10080->80/tcp, 0.0.0.0:10443->443/tcp   gitlab
```

`STATUS` 会有下面几种状态

```text
`health: starting`  说明容器正在启动中，还没有启动完成。
`healthy`  说明容器启动完成，一切正常。
`unhealthy`  说明容器启动完成，但是不健康，有问题。建议执行命令重启容器`docker restart gitlab`。
```

**大坑**:如果状态一直是`unhealthy`，在更改CPU=4 , Memory=4GB的情况下，状态变为`healthy`。恢复CPU=2 , Memory=2GB时，状态又变回`unhealthy`。说明gitlab很消耗性能。  
**深坑**：在启动容器后， com.docker.hyperkit 进程出现高达 100%、200%的CPU使用率，在提高CPU数量，删除`File sharing`条目之后得到缓解。

##### 4、访问gitlab

本机打开浏览器访问`http://127.0.0.1:10080`，会显示`Change your password`，这是第一次访问，修改管理员账户密码。更改完成后，就进入了登录页面，输入`root`和刚改的密码，就可以进入gitlab的控制台了。  
局域网内其他用户可以通过`http://宿主机ip:10080`来访问。

#### 5.设置语言

成功登录后，可以设置当前用户的语言为中文。点击右上角的头像，选择`Settings`，左侧找到`Preferences`，页面向下滚动就可以看到`Localization`了，下拉选择中文，最后点击`Save changes`，刷新页面就可以显示成中文了。如图  
![](media/16120859165616/16134863305633.jpg)  
![](media/16120859165616/16134864292402.jpg)

#### 6.创建用户

在使用`root`用户登录之后，点击导航栏的扳手图标，找到左侧`用户`，点击右侧按钮`新用户`，之后填写好必填项，初始密码填写12345678，点击最下面的`创建用户`按钮就可以了。  
![](media/16120859165616/16134870337206.jpg)

#### 7.创建一个项目

点击导航栏的`Gitlab`图标，会显示首页的项目，点击右侧`新建项目`，选择`创建空白项目`，也可以选择使用模板创建。  
![](media/16120859165616/16134870881325.jpg)

项目名称填入`demo`，点击`新建项目`按钮完成。

![](media/16120859165616/16134872391466.jpg)

创建完成后，还需要添加SSH，才可以访问仓库。  
![](media/16120859165616/16134874595200.jpg)

点击`添加SSH密钥`，在`密钥`输入框中，填写密钥。如果不知道如何添加，可以点击页面中的`生成一个`链接，会跳转到帮助页面，有文档教你如何创建密钥。  
下面以Mac系统为例，创建一个`ED25519`密钥。  
打开命令行，输入`ssh-keygen -t ed25519 -C "admin@example.com"`回车，记得替换一下邮箱。

![](media/16120859165616/16134888347946.jpg)

**注意，输入文件名字时，如果没有指定完整路径，文件会存放在当前用户的目录，即`/Users/xxx`**

查看一下你的密钥对存放的目录

```text
ls ~/.ssh/ #这是默认目录，可以替换指定的存储目录
gitlab_root     gitlab_root.pub
```

使用`cat`命令，查看并复制公钥的内容。复制ssh开头的这行内容。

```text
> cat ~/.ssh/gitlab_root.pub                                                                                                                     
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKAxw5whGEq+Itaui0vH0Jic4lb45kg7W7xwVA0f4fy2 admin@example.com
```

复制后粘贴到gitlab中的密钥输入框内，点击`添加密钥`。  
![](media/16120859165616/16134894177816.jpg)  
![](media/16120859165616/16134895554352.jpg)

然后回到`demo`项目的主页，按照`命令行指引`，这就完成项目的创建了。

## 踩坑

坑:  
`Permission denied (publickey,password,keyboard-interactive).`

[参考:Permission denied (publickey,keyboard-interactive)](https://stackoverflow.com/questions/1556056/permission-denied-publickey-keyboard-interactive)  
[参考:SSH Permission denied (publickey,password,keyboard-interactive)](https://github.com/lqshow/notes/issues/26)  
[参考:Mac下SSH免密登录Permission denied (publickey,password,keyboard-interactive).](https://www.cnblogs.com/feiquan/p/13693329.html)

问题的原因应该是SSH端口的问题。  
gitlab容器创建时，增加了 10022:22 的映射关系，即宿主机 10022 端口映射到容器的 22 端口。在使用 `git clone git@gitlab.interotc.com:zzjs/demo.git` 命令时会报上面的错误。  
同时又因为本地有多个git环境，不同的账号。  
不断的google之后，处理如下：  
1.创建config文件，来解决不同环境和不同账号的问题。  
2.创建authorized\_keys文件，来解决免密登录。

```text
# 进入ssh目录
> cd ~/.ssh
# 查看文件，我这里的文件比较多
> ls
gitee_id_rsa      github_id_rsa       gitlab_zzjs         known_hosts
config            gitee_id_rsa.pub    github_id_rsa.pub   gitlab_zzjs.pub


```

可以看到已经存在一个 `config` 文件了，如果没有这个文件，使用 `touch config` 创建一个就好了。  
在`config`文件中添加下面的内容

```text
Host gitlab.interotc.com
HostName gitlab.interotc.com
User zzjs@example.com
Port 10022
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitlab_zzjs
```

当前目录是没有 `authorized_keys` 文件的，这里使用管道的方式创建文件。

```text
>  cat gitlab_zzjs.pub >> authorized_keys
>  chmod 600 authorized_keys
>  cat authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKEY0aS5EkIHAs3PPT2Gv1qjbp14S084qn5t6ryt1O/W zzjs@example.com
```

这两步完成后来测试一下

```text
> ssh -T git@gitlab.interotc.com
The authenticity of host '[gitlab.interotc.com]:10022 ([x.x.x.x]:10022)' can't be established.
ECDSA key fingerprint is SHA256:zX6tCKkYn+/77AMxBN0OWr1uoo43jKCJqp+2bGt4PVY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[gitlab.interotc.com]:10022' (ECDSA) to the list of known hosts.
Welcome to GitLab, @zzjs!
```

再次测试，并`git clone`，成功！

```text
> ssh -T git@gitlab.interotc.com
Welcome to GitLab, @zzjs!
> git clone git@gitlab.interotc.com:zzjs/demo.git
Cloning into 'demo'...
warning: You appear to have cloned an empty repository.
```

虽然解决了问题，但对于本地来说设置和操作过多，后续尝试使用Nginx去解决这个问题。

坑:  
`Pushing to http://gitlab.interotc.com/zzjs/demo.git   remote: You are not allowed to push code to this project.   fatal: unable to access 'http://gitlab.interotc.com/zzjs/demo.git/': The requested URL returned error: 403`

这是使用Sourcetree，在推送时的报错。暂时较忙，也没有启用https，在新建克隆时，把http的请求改成git的请求后，问题不在出现。
