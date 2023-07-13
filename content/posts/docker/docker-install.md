---
title: "折腾Docker之安装"
date: 2021-01-28T18:24:46+08:00
draft: false
tags: ["Docker"]
categories: ["Docker"]
---


### 一. 安装 Docker

下载 [Docker Desktop](https://www.docker.com/products/docker-desktop) 按提示进行安装。

安装完成后，打开`Docker Desktop`，选择`Preferences`，然后选择`Docker Engine`，加入下面国内镜像源，点击`Apply & Restart`。

```text
{
    "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://reg-mirror.qiniu.com",
    "https://mirror.ccs.tencentyun.com"
  ]
}
```

### 二. 安装 Kubernetes(可选)

**注：如果是简单使用Docker，可跳过此步骤。**

因为中国大陆网络限制的原因，直接`Enable Kubernetes`，会一直卡在`starting`的状态。

##### 深坑：Kubernetes is starting

**参考：[k8s-for-docker-desktop](https://github.com/AliyunContainerService/k8s-for-docker-desktop) 解决方案。**

1、首先克隆仓库到本地。  
2、终端打开`k8s-for-docker-desktop`目录，执行`./load_images.sh` 命令，等待完成。  
3、打开`Docker Desktop`，选择`Preferences`，打开左侧`Kubernetes`,勾选`Enable Kubernetes`,点击`Apply & Restart`。等待完成。如果有弹窗，点击`Install`。

### 一. 初步使用 Docker

1.搜索镜像  
`docker search 关键字`  
示例

```text
> docker search hello
NAME                                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
hello-world                                Hello World! (an example of minimal Dockeriz…   1379      [OK]
kitematic/hello-world-nginx                A light-weight nginx container that demonstr…   148
tutum/hello-world                          Image to test docker deployments. Has Apache…   78                   [OK]
nginxdemos/hello                           NGINX webserver that serves a simple page co…   65                   [OK]
```

2.拉取镜像  
`docker pull 关键字`  
示例

```text
> docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

3.查看镜像  
`docker images` 或 `docker image ls`  
示例

```text
> docker images
REPOSITORY         TAG           IMAGE ID       CREATED         SIZE
jenkins/jenkins    latest        0d3e7552f52a   7 days ago      721MB
gitlab/gitlab-ce   13.8.1-ce.0   404c168c204f   2 weeks ago     2.15GB
nginx              latest        f6d0b4767a6c   4 weeks ago     133MB
hello-world        latest        bf756fb1ae65   13 months ago   13.3kB
```

4.删除镜像  
`docker image rm 镜像ID或镜像名` 或 `docker rmi 镜像ID或镜像名`  
示例

```text
# 删除镜像
> docker image rm hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b
Deleted: sha256:9c27e219663c25e0f28493790cc0b88bc973ba3b1686355f221c38a36978ac63
# 查看镜像
> docker images
REPOSITORY         TAG           IMAGE ID       CREATED       SIZE
jenkins/jenkins    latest        0d3e7552f52a   7 days ago    721MB
gitlab/gitlab-ce   13.8.1-ce.0   404c168c204f   2 weeks ago   2.15GB
nginx              latest        f6d0b4767a6c   4 weeks ago   133MB
# 拉取镜像
> docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
# 查看镜像
> docker images
REPOSITORY         TAG           IMAGE ID       CREATED         SIZE
jenkins/jenkins    latest        0d3e7552f52a   7 days ago      721MB
gitlab/gitlab-ce   13.8.1-ce.0   404c168c204f   2 weeks ago     2.15GB
nginx              latest        f6d0b4767a6c   4 weeks ago     133MB
hello-world        latest        bf756fb1ae65   13 months ago   13.3kB
# 删除镜像
> docker rmi bf756fb1ae65
Untagged: hello-world:latest
Untagged: hello-world@sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b
Deleted: sha256:9c27e219663c25e0f28493790cc0b88bc973ba3b1686355f221c38a36978ac63
# 查看镜像
> docker images
REPOSITORY         TAG           IMAGE ID       CREATED       SIZE
jenkins/jenkins    latest        0d3e7552f52a   7 days ago    721MB
gitlab/gitlab-ce   13.8.1-ce.0   404c168c204f   2 weeks ago   2.15GB
nginx              latest        f6d0b4767a6c   4 weeks ago   133MB

```

5.删除重复的镜像  
`docker rmi 镜像名:TAG`

6.删除容器,删除之前需要停止容器，否则会报错

```text
> docker rm gitlab                                                                                                                               
Error response from daemon: You cannot remove a running container 381f7be56483d7ae3ee2362c3c7e4d041d6dc5e08c368de65807f383c253c3f2. Stop the container before attempting removal or force remove

> docker stop gitlab  
gitlab

```

7.更新镜像  
先拉取最新镜像，完成后查看镜像，其中一个`TAG`为`<none>`，把这个镜像删除即可。  
如果有容器正在使用旧的镜像，删除是会报错的。需要先停止并删除容器，在删除镜像，使用新的镜像创建新的容器。

**踩坑：**

```text
 > docker search gitlab
 Error response from daemon: Get https://index.docker.io/v1/search?q=gitlab&n=25: net/http: TLS handshake timeout
```

> 如果已经配置`registry-mirrors`为国内镜像，仍然出现上面报错。尝试如下步骤:**1.重启Docker 2.关闭终端 3.重新打开终端**。
