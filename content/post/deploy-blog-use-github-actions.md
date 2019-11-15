---
title: "使用Github Actions自动编译部署基于的hugo博客"
date: 2019-11-15T16:51:03+08:00
draft: false
tags: ["github"]
keywords: ["GitHub", "Actions", "GitHub actions", "hugo", "ACTIONS_DEPLOY_KEY"]
---

在GitHub推出actions功能后，利用GitHub托管博客源码变得更加方便快捷，这里分享一下本站自动化部署的方式。

## 配置`ACTIONS_DEPLOY_KEY`
由于actions部署阶段需要进行代码推送，所以需要先生成一个`ACTIONS_DEPLOY_KEY`。

1. 生成密钥
```shell
# 执行后会生成gh-pages.pub 和 gh-pages
$ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```
1. 上传gh-pages.pub
点击博客仓库的Settings->Deploy keys->add deploy key，Title填写`ACTIONS_DEPLOY_KEY`，Key填写`gh-pages.pub`文件的内容。

1. 上传gh-pages
点击博客仓库的Settings->Secrets->Add a new secret，Name填写`ACTIONS_DEPLOY_KEY`，Value填写`gh-pages`文件的内容。


## 配置 Github actions

```yml
name: github pages

on:
  push:
    branches:
    - master

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
    - uses: actions/checkout@master
      # with:
      #   submodules: true

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2.2.2
      with:
        hugo-version: '0.58.3'
        # extended: true

    - name: Build
      run: hugo --minify

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v2.5.0
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: pages
        PUBLISH_DIR: ./public
```

以上配置会监听master分支的push事件，当有push时，进行hugo打包，并把打包结果推送到pages分支。

现在自动化编译已经完成了，服务器如何自动化部署呢？

其实很简单，只需使用linux crontab 功能自动更新就好，先把仓库clone到 `/var/www`下面，然后切换到pages分支，之后添加crontab自动更新。
```shell
*/5 * * * * git -C /var/www/notes pull
```

经过这么一套配置，只需写md然后推送到远程就行了，哎，太懒了
