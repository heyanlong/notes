---
title: "Docker 多阶段构建"
date: 2019-10-10T10:56:32+08:00
draft: false
---

## 为什么要使用多阶段构建

我们在开发完成进行发版时常见场景是：服务器编译，编译出的结果导入到base image，或者一个完整的Dockerfile进行基础环境设置与编译，不论是那种场景都会导致发版速度慢，环境不兼容，image过大等问题

## 多阶段构建例子

利用docker multi-stage 特性，我们可以把编译阶段和运行环境写成两个FORM，看上去就是两个Dockerfile写到一起了，鹅妹子嘤

```
FROM golang:alpine as builder

WORKDIR /go/src
COPY httpserver.go .

RUN go build -o myhttpserver ./httpserver.go

From alpine:latest

WORKDIR /root/
COPY --from=builder /go/src/myhttpserver .
RUN chmod +x /root/myhttpserver

ENTRYPOINT ["/root/myhttpserver"]
```

如上代码所示，先使用golang官方image进行程序打包，打包结果copy到运行image

## 总结

多阶段构建可以让开发者在一个Dockerfile写入多个构建过程，可以有效快速的构建出更小的image，提升CI/CD系统体验～
