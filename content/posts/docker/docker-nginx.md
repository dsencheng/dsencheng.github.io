---
title: "折腾Docker之Nginx"
date: 2021-01-31T18:28:31+08:00
draft: false
tags: ["Docker"]
categories: ["Docker"]
---


在创建Gitlab时，加了一个参数`--hostname gitlab.yourdomain.com`，而现在用`http://192.168.0.X:10080`的这种方式。显然这种方式并不方便。  
这时候就需要请出Nginx了。  
执行命令`docker pull nginx`，拉取最新镜像。  
拉取完成后执行命令`docker images`

```text
REPOSITORY         TAG           IMAGE ID       CREATED       SIZE
gitlab/gitlab-ce   13.8.1-ce.0   404c168c204f   5 days ago    2.15GB
nginx              latest        f6d0b4767a6c   2 weeks ago   133MB
```

可以看到宿主机有两个镜像。  
执行命令`docker ps`

```text
CONTAINER ID   IMAGE                          COMMAND             CREATED        STATUS                  PORTS                                                                  NAMES
14f86e7bba8f   gitlab/gitlab-ce:13.8.1-ce.0   "/assets/wrapper"   23 hours ago   Up 23 hours (healthy)   0.0.0.0:10022->22/tcp, 0.0.0.0:10080->80/tcp, 0.0.0.0:10443->443/tcp   gitlab
```

当前有一个名字为`gitlab`的容器正在运行。  
尝试创建一个Nginx容器。  
执行命令

```text
docker run -d \
-p 80:80 -p 443:443 \
--name nginx \
--restart=always \
--privileged=true \
-v $HOME/nginx/html:/usr/share/nginx/html \ #可以去掉
-v $HOME/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $HOME/nginx/conf/default.conf:/etc/nginx/conf.d/default.conf \
-v $HOME/nginx/logs:/var/log/nginx \
nginx
```

#### 错误:

`not a directory: unknown: Are you trying to mount a directory onto a file (or vice-versa)? Check if the specified host path exists and is the expected type.`

出现这个问题是`-v xxx:xxx` 卷映射的问题，`default.conf` 这种**是文件映射，不是目录**。  
处理方法是先创建一次Nginx容器，把容器内地文件复制出来。然后删除容器，再次重新创建容器，添加卷映射。  
处理步骤

```text
1.创建一个默认容器
docker run -p 80:80 --name mynginx -d nginx

2.查看容器ID
docker ps -a
回显
CONTAINER ID   IMAGE  COMMAND                  CREATED          STATUS                PORTS                                                                  NAMES
9fa05ef56aa2   nginx  "/docker-entrypoint.…"   20 minutes ago   Up 20 minutes         0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp                               otc-nginx

3.复制文件出来，可以使用命令 `mkdir -p ~/nginx/{conf,html,logs}` 提前创建好目录
docker cp 容器ID:/etc/nginx/nginx.conf $HOME/nginx/conf/nginx.conf

4.删除创建的临时容器
docker rm 容器ID

```

参考[Docker安装nginx以及负载均衡](https://www.cnblogs.com/xishaohui/p/8871994.html)

解决错误之后，重新创建容器，完成后打开浏览器，访问宿主机，如`http://192.168.0.X`。  
此时页面显示如下内容，说明成功。

```text
Welcome to nginx!

If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```

Nginx已经启动成功，接下来就是添加配置文件，把请求转给gitlab容器。  
假设宿主机的IP是192.168.7.10，宿主机已经启动gitlab和nginx两个docker容器。  
宿主机10080端口映射gitlab的80端口。

1.本机增加host映射

```text
127.0.0.1    localhost
::1         localhost
# 新增映射
192.168.7.10 gitlab.yourdomain.com
```

2.增加Nginx配置，重启Nginx容器。

```text
server {
    listen       80;
    listen  [::]:80;
    server_name  gitlab.yourdomain.com;

    location / {
        proxy_set_header Host $proxy_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # 这里有个坑，nginx容器内的网段和宿主机网段不一样，不要使用localhost或者127.0.0.1。
        # 如果你是安装的nginx，不是采用docker，那么nginx和宿主机是一个网段。
        proxy_pass    http://192.168.7.10:10080;
    }

}

```

3.顺利的话，现在访问`gitlab.yourdomain.com`就可以显示正确的网页了。

这里有个套娃逻辑。  
本机增加host映射，访问`gitlab.yourdomain.com`  
⬇️  
发送到`192.168.7.10`(宿主机)的80端口  
⬇️  
宿主机的80端口已经映射nginx容器的80端口  
请求发送给nginx容器的80端口  
⬇️  
nginx容器收到`gitlab.yourdomain.com`请求  
⬇️  
解析配置文件`proxy_pass http://192.168.7.10:10080;`  
意思是把请求转给`192.168.7.10`(宿主机)的10080端口  
⬇️  
宿主机的10080端口已经映射gitlab容器的80端口  
请求发给gitlab的80端口  
⬇️  
gitlab的80端口收到请求，返回网页。

在写nginx配置时，需要清楚容器和宿主机的网络关系，在照搬一些文章的nginx配置文件时，`proxy_pass http://localhost:10080;`和`proxy_pass http://127.0.0.1:10080;`使我走进了一个很大的误区。nginx容器内的`localhost`是指nginx容器内的Linux网络，并不是指宿主机(我的宿主机是Mac)。
