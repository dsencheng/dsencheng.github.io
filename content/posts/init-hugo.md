---
title: "使用Hugo和GitHub Page制作个人博客"
date: 2023-07-12T09:59:36+08:00
draft: false
---

## 安装Hugo
先放上
[官方文档](https://gohugo.io/getting-started/quick-start/)

本人使用的Mac系统，并且已经装有 `homebrew`，所以直接使用终端安装。

打开`终端`，输入命令
```shell
brew install hugo
```

安装完成后，查看一下`hugo`的版本号
```
 brew version
```
`hugo v0.115.2+extended darwin/amd64 BuildDate=unknown`

注意，通过`brew`安装的hugo版本有可能比官网低，建议比对一下。如果不一致，可以升级。命令如下

`brew upgrade hugo`

现在就可以初始化一个站点，来预览效果了
```
# 初始化站点，名字叫 quickstart
hugo new site quickstart
# 进入 quickstart 目录
cd quickstart
# 初始化仓库管理，如果没有git，建议安装VSCode，本文下面也会用到
git init
# 下载一个主题
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
# 配置主题
echo "theme = 'ananke'" >> hugo.toml
# 启动hugo
hugo server
```

`hugo`启动成功后湖出现

`Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)`
`Press Ctrl+C to stop`

然后打开`http://localhost:1313/`，就可以预览网页了。

### 问题: page not found

如果预览出现 `page not found` ,那应该是主题配置出错了，网上的大部分教程，都会告诉你把主题配置在`config.toml`文件，对比了官方文档才发现，应该是配置在`hugo.toml`文件里。

如果修改后仍出现该问题，建议按照官方文档操作一遍。

#### 建议安装VSCode，微软提供的跨平台代码编写工具，不仅限于写代码。并且VSCode有大量插件，后续再编写文章，仓库同步操作时，只需要这一个软件就可以了。

![截图](/images/iShot_2023-07-12_13.30.02.png)

接下来就可以写第一篇文章了。
```
# 创建文件 content/posts/my-first.md
hugo new posts/my-first.md
```

写入点内容，就可以看到自己的文章预览了。

而且可以在服务启动过程中做修改，可以实时预览效果。

如果需要引用本地图片，像上面的例子一样，可以创建存放图片的目录。

`hugo`会生成`public`目录，所以图片最终要引用这个目录下的相对路径。

网络教程一： 在项目根目录创建 `static/images` 存放图片。

网络教程二： 在 `content` 创建 `images` 目录存放图片。

上面两种方式只是让你找一个位置存放图片。

引用图片则是另外一个路径。

执行 `hugo` 命令，查看`public`目录下，你的图片相对路径，文章中使用该相对路径就行了。

比如目录结构如下:
```
- public
    - images
        - iShot_2023-07-12_13.30.02.png
    - posts
    ....
```

使用方式，不需要带 `public` 路径。

`![图片描述](/images/iShot_2023-07-12_13.30.02.png)`

这样就可以在文章中插入图片了。


## 使用GitHub Actions

很多教程中，会告知你使用 `hugo` 命令，把生成的 `public` 上传到GitHub上，或者其他的服务器上，这样就可以通过网络访问你的网站了。

如果你使用亚马逊云、GitHub、GitLab等，都是有更好的方式。

本文是部署在GitHub上的，上链接

[托管在GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

打开你的GitHub对应仓库，依次找到 `Settings` -> `Pages` -> `Source`

这是个下拉选项，默认应该是 `Deploy from a branch` ，修改为 `GitHub Actions` .

在你的本地仓库(目录)，创建文件

```
.github/workflows/hugo.yaml
```

插入下面的内容，注意修改仓库分支和`hugo`版本。内容皆来自[官方文档](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

```
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.115.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

然后提交代码，推送到GitHub

打开你的GitHub该项目仓库主页，上面找到 `Actions` , 在左侧应该可以看到 `Deploy Hugo site to Pages` ，这个工作流在执行或已经执行完成。

当工作流执行完成，你的主页也就更新完成了，不在需要你手动上传 `public` 文件夹了

并且可以在本地添加 `.gitignore` 文件，写入 `public/`， 意思是提交仓库代码时，忽略 `public/` 目录。