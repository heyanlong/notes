---
title: "在MacOS上安装Dlib"
date: 2019-10-30T17:29:44+08:00
draft: false
thumbnail: "/images/install-dlib-on-macos/install-dlib-on-macos.jpg"
tags: ["MacOS", "OpenCV", "Python"]
keywords: ["MacOS", "OSX", "python", "dlib", "安装dlib", "install dlib on macos"]
categories:
  - "计算机视觉"
  - "AI"
---


最近在参加公司举办的黑客马拉松比赛，由于使用到了Dlib，简单记录一下如何在MacOS上安装Dlib。

## 步骤1：安装操作系统依赖库
```shell
brew cask install xquartz
brew install gtk+3 boost
brew install boost-python
```

## 步骤2：安装python

已安装的可以省略此步骤
```shell
brew install python
python --version
```

## 步骤3：安装python虚拟环境

我们需要一个项目专用的虚拟环境，用来和全局环境分离，避免安装的依赖污染全局环境

```shell
pip install virtualenv virtualenvwrapper

# 创建虚拟环境
mkvirtualenv venv
workon venv

# 在虚拟环境中安装依赖的软件包
pip install numpy scipy matplotlib scikit-image scikit-learn ipython pandas

# 退出虚拟环境
deactivate
```

## 步骤4：安装Dlib
```shell
brew install dlib
```

## 步骤5：安装python dlib
```shell
workon venv
# 使用pip安装Dlib
pip install dlib
# 安装完成。您现在可以使用deactive从virtualenv退出。
deactivate
```



