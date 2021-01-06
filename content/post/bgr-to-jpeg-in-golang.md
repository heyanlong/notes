---
title: "BGR to JPEG in golang"
date: 2021-01-06
draft: false
tags: ["图像处理"]
keywords: ["bgr", "bgr 转换 jpeg", "rgb", "golang"]

categories:
  - "golang"

thumbnail: "/images/bgr-to-jpeg-in-golang/rgb.png"
---

JPEG 图片一般使用`RGB` 或者 `CMYK` 颜色数据进行填充，而OpenCV的image Mat一般为`BGR`数据。
如果使用OpenCV可以很好的存储为JPEG文件，但是在golang中并没有提供类似`cv.imwrite`的函数。
所以需要一点简单的处理将Mat中存储的BGR数据存储为JPEG图片。

## RGB 与 BGR 数据差异
比如显示红色

`RGB` 表述为`#FF0000` (`#RRGGBB`)

`BGR` 表述为`#0000FF` (`#BBGGRR`)

其实本质区别为读取顺序一个为从左到右，一个为从右到左。了解到差异之后即可使用`golang`提供的内置函数进行转换

## 转换程序
```golang
func BGRToJpeg(o io.Writer, bgr []byte, w int, h int, opt *jpeg.Options) error {
	if len(bgr) != w*h*3 {
		return fmt.Errorf("bgr input error")
	}

	rgba := image.NewRGBA(image.Rect(0, 0, w, h))

	x, y := 0, 0
	for i := 0; i < len(bgr); i++ {
		if i > 0 && i%3 == 0 {
			b, g, r := bgr[i-3], bgr[i-2], bgr[i-1]
			rgba.Set(x, y, color.RGBA{R: r, G: g, B: b, A: 0})
			if x > 0 && x%(w-1) == 0 {
				x = 0
				y++
			} else {
				x++
			}
		}
	}

	return jpeg.Encode(o, rgba, opt)
}
```
