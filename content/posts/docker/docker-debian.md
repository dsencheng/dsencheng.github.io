---
title: "折腾Docker之Debian系统安装frps"
date: 2023-07-14T13:29:42+08:00
draft: false
tags: ["Docker"]
categories: ["Docker"]
---

现有一台基于KVM的VPS，现在想安装frps映射到家里的路由器或树莓派上。

## 设置存储库

首先通过SSH登录到VPS上，因为直接用的root账号，下面的命令会省掉sodu

{{< admonition tip >}}
如果命令提示权限问题，请按照官方教程补充 `sodu`
{{< /admonition >}}

[官方Debian安装Docker教程](https://docs.docker.com/engine/install/debian/#install-using-the-repository)

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

## 安装Docker

1. 更新包索引:
   ```
   apt-get update
   ```
2. 安装 Docker Engine、containerd 和 Docker Compose
   
   ```
   apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
3. 通过运行映像验证 Docker 引擎安装是否成功
   ```
   docker run hello-world
   ```
4. 出现下面的内容，就表示安装成功了
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
    `/etc/frp/frps.ini` 文件路径需要和启动命令中的第一个路径保持一致
    表示把宿主机的文件映射到Docker容器的文件
    frps会依赖并读取这个配置文件
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
   # 设置http及https协议下代理端口（非重要）
   vhost_http_port = 7080
   vhost_https_port = 7081
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
5. 访问frps面板，打开浏览器，输入`IP:dashboard_port`，如`8.8.8.8:7001`，就能看到frps的web管理页面了。
   

## 附加项：安装Docker Web管理

对于新手来说，各种命令不熟悉，docker运行状态不了解，如果有一个web可视化管理，就舒服多了。

[在 Linux 上安装 Portainer CE with Docker](https://docs.portainer.io/start/install-ce/server/docker/linux)

我这里使用 `Portainer CE` 作为web管理页面。

首先，创建用于存储数据库的卷(volume):
```
docker volume create portainer_data
```

然后，下载安装 `Portainer` 服务容器:
```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
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