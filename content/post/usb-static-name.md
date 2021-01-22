---
title: "Ubuntu 下固定USB摄像头设备名称"
date: 2021-01-22
draft: false
tags: ["linux"]
keywords: ["linux", "udevadm", "USB", "UBUNTU"]

categories:
  - "linux"

thumbnail: "/images/linux-usb-static-name/image.jpg"
---

在一个人工智能分析系统中，需要用到大量的USB摄像头做视频输入源。如果使用linux默认的`/dev/video*`命名方式，会根据USB设备插入顺序进行命名，名称为`/dev/video0` `/dev/video1`。

这样存在一个问题，如果系统启动后每个USB设备初始化时间不确定，就会打乱顺序，最终导致分析系统错误识别。因此，需要固定USB设备的名称，不使用默认名称。

## 固定方法

使用lsusb查看设备
```shell
# lsusb
Bus 001 Device 017: ID 32e4:9422 USB Video

# 返回数据

# 32e4为idVendor，USB设备ID

# 9422为idProduct，USB设备的产品号
```

得到基础数据后就可以进行固定设备名称了。这里需要编辑 `/etc/udev/rules.d/video.rules` ，如果文件不存在需要创建一个。`video.rules`存储了重命名的规则。

```shell
KERNEL=="video*", ATTRS{idVendor}=="32e4", ATTRS{idProduct}=="9422", MODE:="0777", SYMLINK+="usb_video_0"
```

如果多个设备的idVendor和idProduct都一样，可以使用KERNELS来区分
```shell
KERNELS=="1-7:1-0" SYMLINK+="usb_video_0"
```

然后执行命令让规则生效
```shell
sudo udevadm trigger
```
