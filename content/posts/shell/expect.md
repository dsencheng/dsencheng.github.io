---
title: "expect 中处理重定向"
date: 2023-09-13T18:04:32+08:00
draft: false
tags: ["shell"]
categories: []
---

先说结果，下面这个expect脚本会执行错误，其原因是spawn不能响应Linux中的重定向，也就 **2>&1** 这个部分。

```text
  #!/usr/bin/expect -f
  spawn ./shadowsocks.sh 2>&1 | tee shadowsocks.log   //执行错误
```

通过Google在Stack Overflow上找到了解决方法。修改如下：

```text
  #!/usr/bin/expect -f
  spawn bash -c "./shadowsocks.sh 2>&1 | tee shadowsocks.log"
```

[原址链接](https://stackoverflow.com/questions/41046258/bash-command-redirection-in-expect-script-failing-with-permission-denied-spawn)

——————————————————————————

## 下面是啰嗦，可以不看。

在vps上安装ss(翻墙)时，使用root用户登录，运行以下命令：

```text
  wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
  chmod +x shadowsocks.sh
  ./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

在ssh登录时，网上抄了一份expect脚本来自动登录，例如：

#### 注1：如果你想使用，请不要照抄，很可能执行失败

```text
  #!/usr/bin/expect -f
  set user 用户名
  set host IP地址
  set password 密码

  spawn ssh $user@$host
  expect {
      "yes/no" {
          send "yes\r"
          exp_continue
        }
      "password:" {
        send "$password\r" 
      }
  }
 interact
```

之后就想安装ss的3条命令，也可以这样执行啊，然后自己就写了一个expect脚本，保存为 ***ss.exp***

#### 注2：如果你想使用，请不要照抄，很可能执行失败

```text
#!/usr/bin/expect -f
set timeout -1
spawn wget --no-check-certificate  https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

expect "#"
spawn chmod +x shadowsocks.sh
expect "#"
spawn ./shadowsocks.sh 2>&1 | tee shadowsocks.log


expect {
    "*Default password*" {
        send "\n"
        exp_continue
    }
    "*Default port*" {
        send "\n"
        exp_continue
    }
    "*Which cipher you'd select*" {
        send "7\n"
        exp_continue
    }
    "*Press any key to start*" {
        send "\n"
        exp_continue
    }
    "#" {
        send "\r"
    }
}
interact
```

好，写完了，上传到vps上，同样用expect写的上传脚本

```text
#!/usr/bin/expect -f
set pwd 密码

spawn scp 本地文件名 访问地址的用户名@IP地址:/目录
#例：spawn scp ss.exp root@127.0.0.1:/root
expect "password*"
send "$pwd\r"

interact
```

Linux执行安装expect的命令

```text
sudo apt-get install expect
```

安装完成后，执行命令

```text
expect ss.exp //这是刚刚上传的文件
```

报错，会提示

> shadowsocks.sh argument error 什么的,懒得补错误提示了。

找了好久，才发现

```text
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

原来这个命令中的 **2>&1** ，叫重定向，而expect好像不支持Linux中的重定向。  
经过2天的Google后终于找到了文章开头的处理方法。
