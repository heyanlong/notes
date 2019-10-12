---
title: "彻底删除Mac上的TeamViewer"
date: 2019-10-12T10:29:40+08:00
draft: false
---

今天，微信群又炸锅了～，原因是TeamViewer这个大量使用的软件被某黑客组织拿下了后台系统，黑客可以攻击任何一台安装了TeamViewer的电脑，好恐怖的感觉。

到公司第一件事想卸载电脑上的TeamViewer，但是回想起来当时安装的时候是用的pkg安装方式，这种方式安装比app包安装更复杂，所以不能按照app那种直接拖入垃圾桶来解决，所以想起来写一篇教程，分享完全卸载的过程

## app 和 pkg 的区别

### .app

简而言之app相当于windows的绿色软件，软件运行的所有东西都在一个文件夹，想卸载删除这个文件夹即可，所以你的app直接拖入垃圾桶就卸载完成了

### pkg

pkg 属于复杂的安装模式，需要点击下一步等操作，直到安装完成，令人讨厌的是安装过程需要root权限，并且没有完善的卸载措施，安装完成后文件分散在系统各个目录

## 分析 pkg 包的内容

我们在官方下载的TeamViewer是dmg格式的，一般命名为TeamViewer.dmg，这是一种光盘文件双击就可以挂载到Finder，并显示内部内容，我们打开发现里面是Install TeamViewer.pkg，这时候不要双击打开，而是复制到一个新建的目录，比如我复制到了`~/workspace/tv`这个目录

### 查看文件类型

```bash
❯ file Install\ TeamViewer.pkg
Install TeamViewer.pkg: xar archive compressed TOC: 6565, SHA-1 checksum
```
可以看出pkg其实就是xar压缩格式

### 使用xar解压

```bash
❯ xar -xf Install\ TeamViewer.pkg
❯ ll
total 87016
drwx------  5 yanlong  staff   160B  1  1  1970 AuthPluginPackage.pkg
-rw-r--r--  1 yanlong  staff   4.8K 10 12 10:22 Distribution
drwx------  5 yanlong  staff   160B  1  1  1970 FullRestarterPackage.pkg
drwx------  5 yanlong  staff   160B  1  1  1970 FullSilentInstallerPackage.pkg
-rw-r--r--@ 1 yanlong  staff    42M  8  7 01:15 Install TeamViewer.pkg
drwx------  6 yanlong  staff   192B  1  1  1970 LauncherFull.pkg
drwx------  5 yanlong  staff   160B  1  1  1970 PriviledgedHelperPackage.pkg
drwx------  5 yanlong  staff   160B  1  1  1970 RemotePrintingPackage.pkg
drwx------  4 yanlong  staff   128B  1  1  1970 Resources
drwx------  6 yanlong  staff   192B  1  1  1970 TeamViewerApp.pkg
```
解压后我们得到`AuthPluginPackage.pkg` `FullRestarterPackage.pkg` `FullSilentInstallerPackage.pkg` `LauncherFull.pkg` `PriviledgedHelperPackage.pkg` `RemotePrintingPackage.pkg` `TeamViewerApp.pkg`
这些文件夹，其中每个文件夹里面的内容是固定的，分别为`Bom` `PackageInfo` `Payload`

```bash
~/workspace/tv/AuthPluginPackage.pkg
❯ ll
total 104
-rw-r--r--  1 yanlong  staff    35K  8  7 01:14 Bom
-rw-r--r--  1 yanlong  staff   749B 10 12 10:22 PackageInfo
-rw-r--r--  1 yanlong  staff    10K  8  7 01:14 Payload
```

### 查看文件类型

```bash
❯ file Bom PackageInfo Payload
Bom:         Mac OS X bill of materials (BOM) file
PackageInfo: XML 1.0 document text, ASCII text
Payload:     gzip compressed data, from Unix, original size modulo 2^32 38912
```

其实安装过程就是把Payload解压到各个目录，已`AuthPluginPackage.pkg`我们用gzip解压它的`Payload`看看里面都有什么目录
```bash
~/workspace/tv/AuthPluginPackage.pkg
❯ tar zxvf Payload
x .
x ./TeamViewerAuthPlugin.bundle
x ./TeamViewerAuthPlugin.bundle/Contents
x ./TeamViewerAuthPlugin.bundle/Contents/_CodeSignature
x ./TeamViewerAuthPlugin.bundle/Contents/_CodeSignature/CodeResources
x ./TeamViewerAuthPlugin.bundle/Contents/MacOS
x ./TeamViewerAuthPlugin.bundle/Contents/MacOS/TeamViewerAuthPlugin
x ./TeamViewerAuthPlugin.bundle/Contents/Info.plist
```

### 查找安装路径

可以看出解压出了TeamViewerAuthPlugin.bundle文件夹，那这个文件夹安装到哪了呢？可以通过find命令查找一下
```bash
❯ sudo find / -name "TeamViewerAuthPlugin.bundle" -type d
/Library/Security/SecurityAgentPlugins/TeamViewerAuthPlugin.bundle
```

### 删除

可以看到，这个文件夹被安装到了`/Library/Security/SecurityAgentPlugins/`目录下面，需要做的操作是到这个目录删除即可
```bash
/Library/Security/SecurityAgentPlugins
❯ sudo rm -fr TeamViewerAuthPlugin.bundle
```
其他的pkg文件夹以此类推，执行过程都一样，全部删除完成后把TeamViewer.app拖进垃圾桶即可

## 另外一个简单版本
以上完全是炫技操作，一点也不科学，实用价值不高，现在说一下推荐做法

1. 打开TeamViewer
2. 打开首选项Preferences，快捷键`cmmmand+,`
3. 点击`高级`选项卡
4. 点击``Uninstall`按钮
5. 卸载完成～～～

