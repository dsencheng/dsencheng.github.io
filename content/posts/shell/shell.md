---
title: "Shell 笔记"
date: 2020-06-17T18:09:33+08:00
draft: false
tags: ["shell"]
categories: []
---

获取podspec版本号

```text
VersionString=`grep -E 's.version.*=' xxx.podspec`
VersionString=${VersionString##*=}
VersionNumber=`tr -d "'" <<<"$VersionString"`
echo "current version is ${VersionNumber}"
```

经过几天的Jenkins折腾，借助管道，获取版本号的命令现在仅需要一行.  
首先使用`grep`命令，读取文件内容并过滤到`s.version`这一行；然后使用`tr`命令去掉单引号、字母、空格、等号；最后使用`sed`命令，替换第一个点。

```text
grep -E 's.version.*=' xxx/xxx.podspec | tr -d "'a-z= " | sed "s/\.//1"

3.3.3
```

Jenkins iOS修改版本号脚本示例。  
iOS这里使用了不同的文件来区分环境，比较简单

```text
sed -i '' -e "s/CFBundleVersion.*/CFBundleVersion=2.33/g" dev.xcconfig
```

Jenkins安卓修改版本号脚本示例。  
安卓的不同环境是放在同一个文件里做区分，逻辑比较复杂。

```text
#获取关键字行数    
rows=$(sed -n -e '/isDev/=' build.gradle)
#获取替换行数差值
di=$(grep -n -A 6 'isDev{' build.gradle | sed -n -e '/versionCode/=')
#计算指定行数 
target=`expr $rows + $di - 1`
#替换指定行的内容
sed -n "${target} s/versionCode .*/versionCode 99/p" build.gradle
```

#### Sed命令

+   替换原始文件内容。  
    Mac系统不可以直接使用`-i`，改成`-i '' -e`的组合方式。  
    搜索`APP_Version`开头(包含)之后的任意字符,到换行符前。  
    替换为`APP=88.88`。  
    修改的文件`aa.txt`。批量替换可改为`*.txt`。
    
    ```text
    sed -i '' -e 's/APP_Version.*/APP=88.88/g' aa.txt
    ```
    
+   打印匹配内容的行号
    
    ```text
    sed -n -e '/isPro/=' build.gradle
    ```
    
+   正则截取/替换内容  
    1.假设变量`branch=branches/v0.0.9E2`，输出`v0.0.9E2`。
    
    ```text
    abc=`echo $branch | sed 's/.*\///g'`
    echo $abc
    ```
    

#### SCP命令

远程复制文件

```text
scp zzjs@172.16.77.210:~/Desktop/a.file ~/Desktop
scp ~/Desktop/a.file zzjs@172.16.77.210:~/Desktop
```

#### TR命令

删除数字

```text
echo "hello 123 world 45345" |tr -d '0-9'
```

保留数子

```text
echo "hello 123 world 45345" |tr -dc  '0-9'
```
