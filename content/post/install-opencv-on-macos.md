---
title: "在MacOS上安装OpenCV"
date: 2019-10-27T10:54:44+08:00
draft: false
thumbnail: "/images/install-opencv-on-macos/install-opencv-3-on-macos.jpg"
tags: ["MacOS", "OpenCV", "Python"]
keywords: ["MacOS", "OSX", "python", "opencv", "安装opencv", "install opencv on macos", "xcode-select", "xcode command line tools"]
description: "在MacOS安装Opencv，install opencv on macos"

categories:
  - "计算机视觉"
  - "AI"
---


最近在参加公司举办的黑客马拉松比赛，由于使用到了OpenCV，简单记录一下如何在MacOS上安装OpenCV。

## 步骤1：安装Xcode Command Line Tools

由于安装OpenCV需要用到clang等UNIX工具包，所以先安装Xcode Command Line Tools

1. 启动`终端`app
1. 输入如下字符串
```shell
xcode-select --install
```
1. 将出现一个弹出窗口，询问：“xcode-select需要Xcode Command Line Tools，是否立即安装它？”，点击Install进行安装
1. 等待软件包下载完毕，大约140MB，下载时间看网速
1. 安装程序消失后，输入`git`命令，测试是否安装成功

## 步骤2：安装Python3

```shell
brew install python

# 验证python版本
python --version
```
这里使用`brew`工具安装`Python`，简单快速。

## 步骤3：安装Python虚拟环境

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

## 步骤4：安装OpenCV
1. 使用brew安装
```shell
brew install opencv
```
2. 将包含cv2.so的site-packages目录包含到python的site-packages目录内
```shell
echo /usr/local/opt/opencv/lib/python3.7/site-packages >> /usr/local/lib/python3.7/site-packages/opencv3.pth
```

## 步骤5：在虚拟环境中创建OpenCV的软链接

不同版本的库路径可能不一样，所以需要先进行查找
```shell
# 查找so名称
find /usr/local/opt/opencv/lib/ -name cv2*.so

# 在刚刚的venv目录内创建软连接
cd venv/lib/python3.7/site-packages/
ln -s /usr/local/opt/opencv/lib/python3.7/site-packages/cv2/python-3.7/cv2.cpython-37m-darwin.so cv2.so
```

## 步骤6：测试OpenCV
```shell
workon venv

# 打开ipython
ipython

import cv2

print cv2.__version__
```

至此，OpenCV安装完成

软件版本为

1. OpenCV 4
1. Python 3.7
