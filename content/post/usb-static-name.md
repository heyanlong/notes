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

执行代码得到KERNELS编码

```shell
udevadm info --attribute-walk --name=/dev/video0

# 返回数据如下
  looking at device '/devices/pci0000:00/0000:00:14.0/usb1/1-3/1-3:1.0/video4linux/video0':
    KERNEL=="video0"
    SUBSYSTEM=="video4linux"
    DRIVER==""
    ATTR{dev_debug}=="0"
    ATTR{index}=="0"
    ATTR{name}=="H264 USB Camera"

  looking at parent device '/devices/pci0000:00/0000:00:14.0/usb1/1-3/1-3:1.0':
    KERNELS=="1-3:1.0"
    SUBSYSTEMS=="usb"
    DRIVERS=="uvcvideo"
    ATTRS{authorized}=="1"
    ATTRS{bAlternateSetting}==" 0"
    ATTRS{bInterfaceClass}=="0e"
    ATTRS{bInterfaceNumber}=="00"
    ATTRS{bInterfaceProtocol}=="00"
    ATTRS{bInterfaceSubClass}=="01"
    ATTRS{bNumEndpoints}=="01"
    ATTRS{iad_bFirstInterface}=="00"
    ATTRS{iad_bFunctionClass}=="0e"
    ATTRS{iad_bFunctionProtocol}=="00"
    ATTRS{iad_bFunctionSubClass}=="03"
    ATTRS{iad_bInterfaceCount}=="03"
    ATTRS{interface}=="USB Camera"
    ATTRS{supports_autosuspend}=="1"

  looking at parent device '/devices/pci0000:00/0000:00:14.0/usb1/1-3':
    KERNELS=="1-3"
    SUBSYSTEMS=="usb"
    DRIVERS=="usb"
    ATTRS{authorized}=="1"
    ATTRS{avoid_reset_quirk}=="0"
    ATTRS{bConfigurationValue}=="1"

```

可以看到第三级里面的`KERNELS=="1-3"`每个设备是不一样的，因此可以用此属性对设备进行区分

```shell
KERNEL=="video*", KERNELS=="1-3" ATTRS{idVendor}=="32e4", ATTRS{idProduct}=="9422", MODE:="0777", SYMLINK+="usb_video_0"
```

然后执行命令让规则生效
```shell
sudo udevadm trigger
```
