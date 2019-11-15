---
title: "Nginx Preview"
date: 2019-07-31T14:34:44+08:00
draft: false
tags: ["nginx"]
---

## 简单介绍
2004年发布第一个版本，2011年成立公司提供商业支持，2019年3月被F5 6.7亿美元收购

拥有反向代理，负载均衡，http缓存等功能

## 特点
为fastcgi 提供负载均衡

采用epoll非阻塞io模型提供高并发能力

## Apache

作为当时全球使用率最高的http 服务器，nginx的目标就是超越apache性能并已高性能为切入点抢占http 服务器市场

apache 采用select 模型，nginx 采用epoll模型，由于技术选型原因，nginx拥有先天优势带来高性能

## 模块

### http

1. ngx_http_core_module
1. ngx_http_access_module 提供基础黑名单白名单功能
1. ngx_http_addition_module 在请求之前或者之后添加文本
1. ngx_http_api_module 提供 REST API 用来查看nginx各种状态
1. ngx_http_auth_basic_module HTTP基本身份验证判断是否有权限
1. ngx_http_auth_jwt_module jwt验证
1. ngx_http_auth_request_module 根据请求授权是否可以访问
1. ngx_http_autoindex_module 自动生成文件夹列表
1. ngx_http_browser_module 判断浏览器是否是现代浏览器与ie
1. ngx_http_charset_module 设置响应字符集
1. ngx_http_dav_module WebDAV 模块
1. ngx_http_empty_gif_module 空白图片输出
1. ngx_http_f4f_module Adobe HTTP动态流
1. ngx_http_fastcgi_module fastcgi支持
1. ngx_http_flv_module flv视频支持
1. ngx_http_geo_module 使用ip地址进行变量映射
1. ngx_http_geoip_module 使用ip地址进行区域位置转换
1. ngx_http_grpc_module grpc代理
1. ngx_http_gunzip_module 在header中有gzip压缩选型但是客户端不支持gzip的时候生效
1. ngx_http_gzip_module 使用gzip传输数据
1. ngx_http_gzip_static_module 压缩后的文件存储为.gz 下次请求时直接输出文件
1. ngx_http_headers_module 追加响应header
1. ngx_http_hls_module 对视频文件提供实时流访问
1. ngx_http_image_filter_module 对图片进行旋转，剪切
1. ngx_http_index_module 入口执行文件
1. ngx_http_js_module js模块
1. ngx_http_keyval_module 定义key val变量
1. ngx_http_limit_conn_module 限制某个ip的连接数
1. ngx_http_limit_req_module 限制某个ip的请求数，使用漏桶规则
1. ngx_http_log_module 日志
1. ngx_http_map_module 定义map变量
1. ngx_http_memcached_module 代理memcached
1. ngx_http_mirror_module 镜像网站
1. ngx_http_mp4_module mp4流式传输
1. ngx_http_perl_module
1. ngx_http_proxy_module 代理http
1. ngx_http_random_index_module 随机index
1. ngx_http_realip_module  ip替换
1. ngx_http_referer_module 阻止无效Referer
1. ngx_http_rewrite_module uri 重写
1. ngx_http_scgi_module scgi代理
1. ngx_http_secure_link_module 检查请求链路的真实性
1. ngx_http_session_log_module
1. ngx_http_slice_module 将一个请求拆分为多个请求
1. ngx_http_spdy_module
1. ngx_http_split_clients_module 定义ab测试变量，不同的客户端拆分到不同的后端
1. ngx_http_ssi_module ssi 支持，将文件潜入到网页
1. ngx_http_ssl_module ssl支持
1. ngx_http_status_module nginx 状态
1. ngx_http_stub_status_module nginx基本状态
1. ngx_http_sub_module 替换字符串来修改响应结果
1. ngx_http_upstream_module upstream模块
1. ngx_http_upstream_conf_module 通过http请求动态配置upstrem
1. ngx_http_upstream_hc_module upstream中的服务器进行健康检查
1. ngx_http_userid_module 基于cookie设置客户端唯一标标识
1. ngx_http_uwsgi_module uwsgi 代理
1. ngx_http_v2_module http2.0
1. ngx_http_xslt_module 使用XSLT 转换 xml

### mail

1. ngx_mail_core_module
1. ngx_mail_auth_http_module
1. ngx_mail_proxy_module
1. ngx_mail_ssl_module
1. ngx_mail_imap_module
1. ngx_mail_pop3_module
1. ngx_mail_smtp_module

### stream

1. ngx_stream_core_module
1. ngx_stream_access_module
1. ngx_stream_geo_module
1. ngx_stream_geoip_module
1. ngx_stream_js_module
1. ngx_stream_keyval_module
1. ngx_stream_limit_conn_module
1. ngx_stream_log_module
1. ngx_stream_map_module
1. ngx_stream_proxy_module
1. ngx_stream_realip_module
1. ngx_stream_return_module
1. ngx_stream_split_clients_module
1. ngx_stream_ssl_module
1. ngx_stream_ssl_preread_module
1. ngx_stream_upstream_module
1. ngx_stream_upstream_hc_module
1. ngx_stream_zone_sync_module

### other

1. ngx_google_perftools_module
