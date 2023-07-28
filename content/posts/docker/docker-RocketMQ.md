---
title: "折腾Docker之RocketMQ单机部署"
date: 2023-07-27T15:35:33+08:00
draft: false
tags: ["RocketMQ"]
categories: ["Docker"]
---

有不少教程是下载`apache/rocketmq`这个官方镜像来操作，由于本人新手，并且是单机运行，所以使用了官方提供的脚本，自己构建镜像运行。
(好像自己构建镜像更复杂了？😅)

本机环境 `Mac`

## clone rocketmq-docker项目的代码

官方的docker地址
```
git clone https://github.com/apache/rocketmq-docker.git
```

执行命令下载官方的脚本仓库

## rocketmq镜像构建

下载完成后，进入 `nacos-docker/image-build` 目录

```
cd nacos-docker/image-build
```

执行脚本构建镜像，示例为 5.1.3 版本

```
sh build-image.sh 5.1.3 centos
```
上面使用centos系统，最后出来的镜像比较大，可以选择 `alpine`， 小很多
```
sh build-image.sh 5.1.3 alpine
```

## rocketmq-dashboard镜像构建[可选]

```
sh build-image-dashboard.sh 1.0.0 centos
```
目前为止，dashboard的版本和系统是固定参数，传其他版本和系统会报错。

{{< admonition question "404 Not Found" false >}} 
这个问题是在下载`https://dlcdn.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz` 文件时，找不到导致的错误

打开`image-build`目录下的`Dockerfile-centos-dashboard`文件，修改其中的`MAVEN_VERSION=3.6.3`的版本号，改成`3.9.3`
{{< /admonition >}}

## 创建挂载路径

因为系统不同，路径也有所区别，只要保证路径正确存在就行

### NameSrv的挂载路径

```
mkdir -p /root/rocketmq/data/namesrv/logs
mkdir -p /root/rocketmq/data/namesrv/store

```

### Broker的挂载路径

```
mkdir -p /root/rocketmq/data/broker/logs
mkdir -p /root/rocketmq/data/broker/store
mkdir -p /root/rocketmq/etc/broker
```

### 创建Broker配置文件

```
nano /root/rocketmq/etc/broker/broker.conf
```

文件内容如下

```
brokerClusterName = mxsm-docker
brokerName = mxsm-docker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
# Docker环境需要设置成宿主机IP
brokerIP1 = 192.168.43.128

```

### 编写Docker-compose文件

```
version: '3'
services:
  #Service for nameserver
  namesrv:
    image: apache/rocketmq:5.1.3
    container_name: rocketmq-namesrv
    ports:
      - 9876:9876
    environment:
      - JAVA_OPT_EXT=-server -Xms256m -Xmx256m -Xmn256m
    volumes:
      - /root/rocketmq/data/namesrv/logs:/root/logs
      - /root/rocketmq/data/namesrv/store:/home/rocketmq/store
    command: sh mqnamesrv

  #Service for broker
  broker:
    image: apache/rocketmq:5.1.3
    container_name: rocketmq-broker
    links:
      - namesrv
    depends_on:
      - namesrv
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    environment:
      - NAMESRV_ADDR=namesrv:9876
      - JAVA_OPT_EXT=-server -Xms512m -Xmx512m -Xmn256m
    volumes:
      - /root/rocketmq/data/broker/logs:/home/rocketmq/logs
      - /root/rocketmq/data/broker/store:/home/rocketmq/store
      - /root/rocketmq/etc/broker/broker.conf:/home/rocketmq/conf/broker.conf
    command: sh mqbroker -c /home/rocketmq/conf/broker.conf

  # 如果前面没有创建dashboard的镜像，下面删除掉
  #Service for rocketmq-dashboard
  dashboard:
    image: apache/rocketmq-dashboard:1.0.0-centos
    container_name: rocketmq-dashboard
    ports:
      - 8080:8080
    links:
      - namesrv
    depends_on:
      - namesrv
    environment:
      - NAMESRV_ADDR=namesrv:9876
```

运行命令

```
docker-compose -f ./docker-compose.yml up
```

启动成功后，可以访问 `localhost:8080`查看 dashboard。

{{< admonition note "compose文件注意事项">}} 

1. image: apache/rocketmq:5.1.3 镜像名字正确
2. volumes: 挂载路径保证一致
      - /root/rocketmq/data/namesrv/logs:/root/logs


{{< /admonition >}} 

---
参考文章: [RocketMQ Docker部署](https://juejin.cn/post/7045944869642043422)