---
title: "Nginx 静态编译"
date: 2019-11-15T10:51:55+08:00
draft: flase
tags: ["nginx"]
keywords: ["nginx", "编译", "静态编译", "nginx静态编译", "多阶段打包"]
---

在使用 [docker多阶段部署](https://yanlong.me/post/docker-multi-stage-build/) 后，镜像大小得到了有效瘦身，部署速度大幅度提升。

最近需要在项目中放一个nginx来代理http请求到golang，一般来说nginx编译都是动态编译模式，会依赖很多系统so。如果只是单纯的把nginx的binary复制到docker 第二阶段，会导致
在新的image中找不到依赖的so文件，从而nginx不能正常工作。

第一想到的就是可以静态编译nginx，让依赖的所有so都集成到binary里面，这样就可以复制整个nginx目录到新image。

## 静态编译

由于线上环境是ubuntu 16.04，为了兼容性，我们找一台ubuntu 16.04机器进行打包。

### 创建临时打包环境
```shell
$ docker run -it ubuntu:16.04 bash
```

### 编译nginx
```shell
# 下载openssl依赖
$ wget https://www.openssl.org/source/openssl-1.0.2g.tar.gz

# 下载nginx
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz

# 解压openssl & 解压nginx
$ tar zxvf openssl-1.0.2g.tar.gz
$ tar zxvf nginx-1.16.1.tar.gz

# configure nginx
# 这一步很关键，需要添加cc 和 ld 选项
# openssl 也必须用源码，要不然不会成功
$ cd nginx-1.16.1
$ ./configure --with-cc-opt='-static -static-libgcc' --with-ld-opt=-static --prefix=/usr/local/nginx --user=root --group=root --with-http_ssl_module --with-openssl=../openssl-1.0.2g

# 编译 安装
$ make && make install

# 如果需要更快速度的编译，可以加上-j选项，启动多核心同时编译，比如40核处理器
# make -j40 && make install

# 打包整个nginx包，上传到私有仓库，这里我用static.yanlong.me模拟私有仓库
$ cd /usr/local
$ tar zcvf nginx-static-1.16.1.tar.gz nginx

$ scp nginx-static-1.16.1.tar.gz root@static.yanlong.me:/var/static/libraries
```

## 多阶段打包

上面步骤已经准备好了nginx的静态编译包，现在我们只需在多阶段部署时添加进去即可。

```dockerfile
# 第一阶段
FROM ubuntu:16.04 as build-env

RUN apt-get update -y && \
    apt-get install --no-install-recommends -y make curl tar unzip ca-certificates && apt-get install gcc -y

# 下载已经编译好的二进制包，这样可以加快CI速度
RUN curl -o nginx-static-1.16.1.tar.gz https://static.yanlong.me/libraries/nginx-static-1.16.1.tar.gz && tar zxf nginx-static-1.16.1.tar.gz && mv nginx /usr/local/

# 第二阶段
FROM ubuntu:16.04
# 把第一阶段下载的nginx包复制到当前阶段
COPY --from=build-env /usr/local/nginx/ /usr/local/nginx/
```

## 测试
```shell
docker run -it ubuntu:nginx bash
# 在容器内shell运行
/usr/local/nginx/sbin/nginx -t

# 输出
# nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
# nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

经过这样改动，CI速度大幅度提升，docker image占用空间也得到了优化。
