---
title: "编译并使用GPU加速的ffmpeg"
date: 2020-03-26T14:26:03+08:00
draft: false
tags: ["ffmpeg", "nvidia"]
keywords: ["ffmpeg", "nvidia", "GPU", "hugo"]
---

最近在开发一款基于视频流的视频分析系统。在使用浏览器播放处理完成的视频时，浏览器需要fmp4格式的数据流，需要通过ffmpeg进行转码。在开发阶段使用的时CPU版本的ffmpeg，线上部署后效果非常不理想。索性编译了一个GPU版本的，做个记录。

## 系统架构

```shell
+----------------------------------------------------------------+
|                                                                |
|                          /--queue-->ffmpeg-->\                 |
| rtsp h.264--open cv --->                      -->merge--> web  |
|                          \--queue-->models-->/                 |
|                                                                |
+----------------------------------------------------------------+
```

## 编译

从源代码编译硬件加速的ffmpeg需要几个步骤

1. 下载 `ffmpeg` 源代码 https://git.ffmpeg.org/ffmpeg.git
1. 下载Nvidia驱动程序
1. 安装CUDA工具包
1. 下载 `nv-codec-headers` 源代码 https://github.com/FFmpeg/nv-codec-headers.git
1. 安装 `nasm` `yasm` `pkgconf`

### 编译步骤

由于我本机已安装指定版本的Nvidia驱动和CUDA工具包，则跳过这两个步骤

```shell
apt install -y nasm yasm pkgconf
git clone https://git.ffmpeg.org/ffmpeg.git
git clone https://github.com/FFmpeg/nv-codec-headers.git

# 切换到你想要的tag版本，我这里nv-codec-headers使用的n9.0.18.3，ffmpeg使用的n4.2.2

# 设置 nv-codec-headers
cd nv-codec-headers
git checkout n9.0.18.3
make install
cd ..

# 编译ffmpeg
cd ffmpeg
git checkout n4.2.2
./configure --enable-cuda --enable-cuvid --enable-nvenc --enable-nonfree --enable-libnpp --extra-cflags=-I/usr/local/cuda/include  --extra-ldflags=-L/usr/local/cuda/lib64
make -j$(nproc)
make install
```

经过以上步骤，GPU版本的ffmpeg就编译好了，测试一下是否正常。

原有CPU版本的转码命令
```shell
ffmpeg -i input.mp4 -c:a copy -c:v h264 -b:v 5M output.mp4
```

修改为GPU版本的
```shell
ffmpeg -vsync 0 -hwaccel cuvid -c:v h264_cuvid -i input.mp4 -c:a copy -c:v h264_nvenc -b:v 5M output.mp4
```

经测试转换成功。

当然，在视频分析系统中，不是这样直接使用命令行转码的，而是通过输入输出管道，输入参数和输出参数也需要根据实际编码格式做相应调整。





