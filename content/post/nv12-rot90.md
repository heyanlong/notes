---
title: "使用numpy高效旋转NV12数据，旋转90度"
date: 2021-04-21
draft: false
tags: ["ML", "AI"]
keywords: ["ML", "AI", "OpenCV", "numpy", "NV12", "Nvidia"]

categories:
  - "AI"

thumbnail: "/images/nv12-rot90/image.png"
---

最近需要YUV数据图像旋转，横屏转竖屏。网上找了一些资料都是for循环去做的，效率极低，写了一个numpy版本的提升效率，分享一下。

## 转换机制

![image](/images/nv12-rot90/image.png)

```python
def rot90(frame, width=1920, height=1080):
    """
    nv12 rot90

    YYYYYY           YYYY           1  2  3  4  5  6             19 13 7  1
    YYYYYY           YYYY           7  8  9  10 11 12            20 14 8  2
    YYYYYY --------> YYYY --------> 13 14 15 16 17 18  --------> 21 15 9  3
    YYYYYY           YYYY           19 20 21 22 23 24            22 16 10 4
    UVUVUV           YYYY           25 26 27 28 29 30            23 17 11 5
    UVUVUV           YYYY           31 32 33 34 35 36            24 18 12 6
                     UVUV                                        31 32 25 26
                     UVUV                                        33 34 27 28
                     UVUV                                        35 36 29 30
                     UVUV

    :param frame: numpy image data
    :param width: image width
    :param height: image height
    :return:
    """
    out = np.array(frame, copy=True).reshape(width * 3 // 2, height)
    frame = frame.reshape(height * 3 // 2, width)
    y = np.rot90(frame[:height, :], k=1, axes=(1, 0))
    out[:width, :height] = y

    for i in range(int(width / 2)):
        uv = frame[height:, i * 2:i * 2 + 2][::-1, :].reshape(1, height)
        out[width + i:width + i + 1] = uv
    return out.ravel()
```

原始YUV 转 RGB

![image](/images/nv12-rot90/0.jpg)

旋转后YUV转RGB

![image](/images/nv12-rot90/1.jpg)

原始YUV灰度图

![image](/images/nv12-rot90/2.jpg)

旋转后YUV灰度图

![image](/images/nv12-rot90/3.jpg)
