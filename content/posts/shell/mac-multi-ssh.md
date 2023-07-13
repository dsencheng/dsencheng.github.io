---
title: "Mac设置多个SSH"
date: 2019-09-12T18:02:23+08:00
draft: false
tags: []
categories: []
---


##### 1.前因

个人常用github，公司使用gitee，为了不同环境的使用方便。下文以这两个网站示例设置多个SSH。

##### 2.生成SSH公钥私钥

```text
//1.进入ssh目录
$ cd ~/.ssh 
//2.创建公私钥，输入对应gitee、github邮箱
$ ssh-keygen -t rsa -C "youremail@email.com" 
Generating public/private rsa key pair.
//3.输入区分名字,例如 github_id_rsa 、 gitee_rsa
Enter file in which to save the key (/Users/dsen/.ssh/id_rsa):github_id_rsa
//4.设置密码，我这里为了方便，没有使用密码，直接按回车enter,
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in github_id_rsa.
Your public key has been saved in github_id_rsa.pub.
The key fingerprint is:
SHA256:mvlMQCBQqa1TQjtSduYPIqJbxRJidpqAwj9bR5KQ02E youremail@email.com
The key's randomart image is:
+---[RSA 2048]----+
|+ooo=Eo          |
|====+= .         |
|****. +          |
|*==o=o .         |
|+=.++oo S        |
|+ ..  .=         |
| +    + .        |
|.      +         |
|        o        |
+----[SHA256]-----+

```

##### 3.创建config文件

在 `~/.ssh` 目录创建一个`config`文件

```text
vim config
或
touch config
```

##### 4.config文件写入内容

```text
# github
  Host github.com
  HostName github.com
# github对应的email或者用户名
  User xxxxx@gmail.com
  PreferredAuthentications publickey
# github对应的私钥完整路径,没有pub后缀
  IdentityFile ~/.ssh/github_id_rsa

# gitee
  Host gitee.com
# gitee对应的email
  User xxxx@163.com
  PreferredAuthentications publickey
# gitee对应的私钥完整路径,没有pub后缀
  IdentityFile ~/.ssh/gitee_id_rsa
```

##### 5.配置网站SSH公钥

a.打开网址，填写公钥标题，用来识别。  
[https://github.com/settings/keys](https://github.com/settings/keys)  
[https://gitee.com/profile/sshkeys](https://gitee.com/profile/sshkeys)  
b.填写公钥内容。

```text
//查看公钥，把输出的内容全部填入的网站SSH公钥内容中，ssh-rsa开头，email结尾。
$ cat github_id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDTa570Bo0BptLJKbXxVrMfe7xv6I9dFY7GTh7LQcaaYe9G7miT0HuX+PS1eu4qmYAeY7HmEsHFGOfOkcq+d5CYYLabUFCuW0BtQEgniQuUr9Aumr9R6nhUbyQ+ldFAUkOqmxP3upUSPbilCzfQ2hBpAik2zZ2yeiDTBKXXq5HU1c4W9c3pbTetUYr36QgfBo6XuK8zlt24DwNmjplOasUhiejK68KVBgmsPuIeU71b8aZcdn9oTF0BIjapLQ0n4F180Oexp0PHvojN8k13oglgkb+ljtrPa8tPbb4xd+j0gr3DM8aG+kF704yuTKwKlVIARZpbgdzc4SSaH0+zr4yN youremail@email.com
```

##### 6\. 测试

```text
ssh -T git@github.com
//成功
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.

ssh -T git@gitee.com       
//成功                                                                                                                       
Hi xxx! You've successfully authenticated, but GITEE.COM does not provide shell access.
```

##### 7.错误

```text
$ ssh -T git@gitee.com
Permission denied, please try again
```

> 1.  检查命令、config文件是否拼写错误。2.检查网站是否配置公钥。3.若`~/.ssh`目录存在`known_hosts`文件，打开 `known_hosts` ,删除对应的ip、网址记录。

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

> ssh-keygen -R "you server hostname or ip"

##### 参考链接：

[Mac生成多个ssh并配置不同域名](https://juejin.im/post/5bf6092a6fb9a04a0279fb6a)

[« expect 中处理重定向](15682637412780.html "Previous Post: expect 中处理重定向")

[Mac 终端 terminal 命令行 iTerm2 访问外网 »](15682635419676.html "Next Post: Mac 终端 terminal 命令行 iTerm2 访问外网")
