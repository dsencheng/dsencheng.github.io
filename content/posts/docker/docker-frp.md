---
title: "折腾Docker之frp内网穿透"
date: 2023-07-14T13:29:42+08:00
draft: false
tags: ["Docker"]
categories: ["Docker"]
---

现有一台基于KVM的VPS，是`Debian`系统，现在想安装frp映射到家里的路由器或树莓派上。

用来从公司访问家里的树莓派，网络如下:

公司 -> VPS -> 家 -> 树莓派

假设VPS的IP地址是 `11.11.11.11`，域名是`hellofrp.top`，并且已经把`@`、`*`记录解析到IP。

![dns](/images/dns_a.png "A记录")

## VPS安装Docker

[官方Debian安装Docker教程](https://docs.docker.com/engine/install/debian/#install-using-the-repository)

首先通过SSH登录到VPS上，因为直接用的root账号，下面的命令会省掉sodu。

{{< admonition tip >}}
如果命令提示权限问题，请按照官方教程补充 `sodu`
{{< /admonition >}}


1. 更新软件包索引并安装软件包以允许使用 基于 HTTPS 的存储库：

    ```
    apt-get update
    apt-get install ca-certificates curl gnupg
    ```
2. 添加 Docker 的官方 GPG 密钥：
   ```
   install -m 0755 -d /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   chmod a+r /etc/apt/keyrings/docker.gpg
   ```
3. 使用以下命令设置存储库：
   ```
   echo \
   "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
   "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
   tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```
4. 安装 Docker Engine、containerd 和 Docker Compose
   
   ```
   apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
5. 通过运行映像验证 Docker 引擎安装是否成功
   ```
   docker run hello-world
   ```
6. 出现下面的内容，就表示安装成功了
    ```
    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
    3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

    For more examples and ideas, visit:
    https://docs.docker.com/get-started/
    ```


{{< admonition tip >}}
如果是自己玩，官方提供了一个安装脚本，可以省略1、2、3、4步骤。

先更新系统程序
```
apt-get update
apt-get upgrade
```
然后执行官方脚本
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
{{< /admonition >}}


## 安装Frps

1. 首先拉取一个Frp的镜像，本文基于 `snowdreamtech/frps` ,你也可以去 https://hub.docker.com/ 搜索适合的镜像
   ```
   docker pull snowdreamtech/frps
   ```
   {{< admonition tip >}}
   这个步骤可以省略
   {{< /admonition >}}
2. 创建并填写 `frps.ini` 配置。[Frp官方文档](https://gofrp.org/)
   ```
   mkdir /etc/frp
   nano /etc/frp/frps.ini
   ```
   {{< admonition tip >}}
    `/etc/frp/frps.ini` 文件路径需要和`docker`启动命令中的第一个路径保持一致
    示例： `-v /etc/frp/frps.ini:/etc/frp/frps.ini`
    表示把宿主机的文件映射到Docker容器的文件
    容器会依赖并读取这个配置文件
    {{< /admonition >}}

    `frps.ini`文件内容示例:
    ```
   [common]
   # 监听端口, 默认7000
   bind_port = 7000
   # 可选面板端口
   dashboard_port = 7001
   # 登录面板账号设置
   dashboard_user = admin
   dashboard_pwd = admin
   # 设置http及https协议下代理端口
   vhost_http_port = 7080
   vhost_https_port = 7081
   # 鉴权用，可以认为是个密码，客户端需要配置一样的
   token = frp1234567890
   # 绑定域名，客户端可设置子域名来连接
   subdomain_host = hellofrp.top
    ```
3. 启动Frps服务端
   ```
   docker run --restart=always --network host -d -v /etc/frp/frps.ini:/etc/frp/frps.ini --name frps snowdreamtech/frps
   ```
   成功后，会返回一条容器ID记录。

4. 查看容器
    ```
    docker ps -a
    ```
5. 访问frps面板，打开浏览器，输入`11.11.11.11:7001` 或 `hellofrp.com:7001`，就能看到frps的web管理页面了。
   

## 测试

VPS上的frps启动之后，我先公司电脑做一个测试。

大致的网络请求过程如下：

本机 -> VPS -> FRP -> 本机

### 启动本地web

刚好`Hugo`可以启动一个本地web服务，本博客就是使用的`Hugo`，使用`hugo server`启动本地博客预览，终端会输出访问地址，我当前测试是 `http://localhost:1313/`。浏览器可以正常访问。

### 下载frp

这里是用的frp二进制程序，可以打开[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)页面下载。

下载完成后解压缩，目录大致如下

```
- frp_0.51.2_darwin_amd64
    - frpc_full.ini
    - frpc.ini
    - frpc
    - frps_full.ini
    - frps.ini
    - frps
    - LICENSE
```

删除`frpc.ini`的示例配置，使用下面内容，覆盖`frpc.ini`文件

```
[common]
# 你的服务器ip地址或域名
server_addr = 11.11.11.11
server_port = 7000
# 鉴权用，可以认为是个密码，和服务端保持一致
token = frp1234567890
# 客户端管理界面，建议填写localhost，就是本机
admin_addr = localhost
admin_port = 7400
admin_user = admin
admin_pwd = admin

[blog]
type = http
# 客户端暴露的接口
local_port = 1313
# 如果你有单独的域名，配置下面这个
# custom_domains = hellofrp.top
# 我域名绑了很多服务，所以这里使用二级域名，访问示例: http://blog.hellofrp.top
subdomain = blog
```

写好配置文件后，在你下载的frp目录下，启动`frpc`。

```
./frpc -c frpc.ini
```

## 验证

浏览器访问`http://blog.hellofrp.top:7080`，成功就能看到和`http://localhost:1313/`一样的页面了。

{{< admonition note >}}
开始我用Docker启动的frpc，搞了半天也不对，frps能看到有个http连接上线了，但就是无法访问本机，而且本机的`http://localhost:7400/`也打不开。

后来想想，好像把客户端搞错了，用Docker启动frpc，应该是这个容器，而不是本机(宿主机)。

然后直接运行frpc的程序，本机客户端管理页面`http://localhost:7400/`可以打开了，并且通过子域名`blog.yourdomain.com`也能访问到我本机上的`http://localhost:1313/`博客

等我搞明白了，在来更新

```
docker run --restart=always --network host -d -v /你的文件路径/frpc.ini:/etc/frp/frpc.ini --name frpc snowdreamtech/frpc
```

{{< /admonition >}}

## 连接失败

出现无法连接上时，排查 `http://localhost:7400/` 是否可以打开。
   
建议配置客户端管理页面。
   
打不开管理页面，说明客户端配置有错误，或者本地网络连接有问题。

如果连接错误，客户端管理页面也有对应的错误日志，方便排查。

我遇见过的几个问题：

1. 没有配置二级域名A记录，域名管理用的是`Cloudflare`，上面没配置 `@` 记录，导致`www.hellofrp.top`浏览器无法访问到VPS。
2. 客户端是路由器openwrt上的frpc页面工具，服务器地址填写 `www.hellofrp.top` 和 `hellofrp.top` 是有区别的。
3. 子域名填写前缀就行。比如`wrt`，那么浏览器访问填写`wrt.hellofrp.top`，不要在子域名填写完整域名。`subdomain = wrt.hellofrp.top`，出现这种配置是错误的。
![frps](/images/frp_wrt.png "OpenWrt")
## 附图

服务端
![frps](/images/frp_s.png "服务端")

客户端
![frpc](/images/frp_c.png "客户端")


## 附加项：安装Docker Web管理

对于新手来说，各种命令不熟悉，docker运行状态不了解，如果有一个web可视化管理，就舒服多了。

[在 Linux 上安装 Portainer CE with Docker](https://docs.portainer.io/start/install-ce/server/docker/linux)

我这里使用 `Portainer CE` 作为web管理页面。

首先，创建用于存储数据库的卷(volume):
```
docker volume create portainer_data
```

运行 `Portainer` 服务容器，`9000`是`http`访问端口，`9443`是`https`访问端口。官方教程中还开放了`8000`端口，这里不使用。
```
docker run -d -p 9000:9000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
`Portainer` 安装完毕，可以使用 `docker ps` 检查容器是否已经启动:

```
root@server:~# docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED       STATUS      PORTS                                                                                  NAMES             
de5b28eb2fa9   portainer/portainer-ce:latest  "/portainer"             2 weeks ago   Up 9 days   0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   portainer
```

现在通过Web浏览器打开地址 `https://localhost:9443` ，如果你安装在远程主机，把`localhost`替换为对应的IP地址。

首次访问，需要初始化一个管理员账号。

登录后，点击左侧 `Home`，然后在点击 `local`，就能进入到 `Dashboard`，查看本机中的镜像、容器等信息了。

