---
title: "配置 ubuntu open files 永久生效"
date: 2020-04-01
draft: false
tags: ["ubuntu", "linux"]
keywords: ["ubuntu", "limit", "file open", "fs.max-open"]

categories:
  - "ubuntu"
  - "linux"

thumbnail: "/images/ubuntu-limits/images.jpg"
---

最近在配置vps的文件最大打开数，根据网上的教程大部分是修改如下几个文件。

1. `/etc/sysctl.conf`
1. `/etc/security/limits.conf`

修改完发现每次登陆ssh后运行`ulimit -a`查看结果还是1024默认值

```shell
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 1752
max locked memory       (kbytes, -l) 16384
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 65535
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

经过搜寻发现一个正确的配置方法，需要修改以下文件
1. `/etc/sysctl.conf`
1. `/etc/security/limits.conf`
1. `/etc/systemd/system.conf`
1. `/etc/systemd/user.conf`
1. `/etc/pam.d/common-session`

1、修改 `/etc/sysctl.conf`

需要在 `/etc/sysctl.conf` 添加 `fs.file-max = 2000000`，由于这里为全局文件打开数量，需要设置的大一些

```shell
$ cat /etc/sysctl.conf

fs.file-max = 2000000
```

2、修改 `/etc/security/limits.conf`

添加一下内容
```ini
* soft     nproc          65535
* hard     nproc          65535
* soft     nofile         65535
* hard     nofile         65535
root soft     nproc          65535
root hard     nproc          65535
root soft     nofile         65535
root hard     nofile         65535
```

3、修改 `/etc/systemd/system.conf` 和 `/etc/systemd/user.conf`

找到 DefaultLimitNOFILE 这一行，删除前面的注释，修改为
```ini
DefaultLimitNOFILE=65535
```


4、修改 `/etc/pam.d/common-session`
添加以下内容
```ini
session required        pam_limits.so
```

配置完成之后，重启系统即可
