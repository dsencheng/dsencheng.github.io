---
title: "树莓派之 FRP 笔记"
date: 2020-06-29T18:11:04+08:00
draft: false
tags: ["RaspberryPi"]
categories: ["RaspberryPi"]
---


[中文文档](https://github.com/fatedier/frp/blob/master/README_zh.md)

#### 1.下载

```text
# 树莓派需要对应的版本
wget https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_linux_amd64.tar.gz
```

#### 2.解压缩

```text
tar -zxvf frp_0.33.0_linux_amd64.tar.gz
```

#### 3.修改文件夹名字

```text
mv frp_0.33.0_linux_amd64 frp
```

#### 3.1 或者复制一份目录及文件

```text
cp frp_0.33.0_linux_amd64 frp
```

#### 4.进入目录开始操作

```text
cd frp
```

# 通过 ssh 访问内网机器

1.将对应版本的 frps 及 frps.ini 放到具有公网 IP 的机器上。  
2.修改 frps.ini 文件

```text
# frps.ini
[common]
bind_port = 7000
```

3.启动 frps

```text
./frps -c ./frps.ini
```

4.将 frpc 及 frpc.ini 放到处于内网环境的机器上。  
5.修改 frpc.ini 文件 ，假设 frps 所在服务器的公网 IP 为 x.x.x.x

```text
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

6.启动 frpc

```text
./frpc -c ./frpc.ini
```

7.通过 ssh 访问内网机器

```text
ssh login_name@x.x.x.x -p 6000
```

# 开机启动frpc服务，同样可以启动frps服务【注意路径和文件名】。

```text
sudo vim /etc/systemd/system/frpc.service
```

按如下修改

```text
[Unit]
Description=frpc daemon
After=syslog.target  network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/sbin/frp/frpc -c /etc/frp/frpc.ini
Restart= always
RestartSec=1min
ExecStop=/usr/bin/killall frpc


[Install]
WantedBy=multi-user.target
```

使用`sudo systemctl enable frpc.service`启用

```text
//启动命令
sudo systemctl start frpc
//停止命令
sudo systemctl stop frpc
//重启命令
sudo systemctl restart frpc
//查看状态
sudo systemctl status frpc
```
