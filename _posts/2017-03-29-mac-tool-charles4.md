---
layout:     post
title:      "mac下的抓包工具：Charles4"
subtitle:   "mac-tool-charles4"
date:       2017-03-29 24:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
- 工具
- Charles
- 抓包
---


> Charles是一款运行在你自己计算机上的web代理工具，可以有效地获取http通信信息，主要用于网页的开发和调试等。

[TOC]

### 安装

以下为来自[史蒂芬周的博客的Charles 4.0 Mac破解版](http://www.sdifenzhou.com/charles4.html)

下载地址：[Charles 4.0 Mac 破解版下载](https://pan.baidu.com/s/1slSXWvz) 密码：6jp3

Charles的破解方法：

1. 打开dmg镜像，将"Charles.app"拖入应用程序中；
2. 打开应用程序——右键"Charles.app"显示包内容——Contents-Java;
3. 将dmg镜像包内的"charles.jar"替换覆盖到第二部的Java文件夹中；
4. 打开"Charles.app",等待30秒，菜单栏中找到"Help"-"Register...",输入任意信息完成注册；
5. Have done！

### 代理本机http请求

设置：顶部菜单栏——Proxy——Mac OS X Proxy

![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/mac-os-x-proxy.png)

这样就打开了对本机http请求的代理，然后就会看到各类对本机的请求出现在主界面中。

![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/record.png)

同时，在mac的`系统偏好设置——网络——高级...——代理`中，我们可以看到`Web代理(HTTP)`和`安全Web代理(HTTPS)`已经勾选，如图：

![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/setting.png)

如果在Charles主界面中没有看到请求Sequence,请仔细检查以下地方：

- 是否配置其他代理，如`系统偏好设置——网络——高级...——代理——自动代理配置`勾选，则应取消勾选。
- 是否打开Charles界面上方同心圆录制按钮，按钮内圆呈红色说明已打开录制。

### 设置安卓端代理http请求

首先保证，mac与安卓手机在同一内网环境中（家用wiki使用同一wifi，公司使用同一内网）。

配置步骤如下：

1. 进入`设置——WLAN——TP-LINK（所用网络）`，长按对应网络，选择`修改网络`。
2. 在`高级选项`部分，`代理`选择`手动`，`代理服务器主机名`填写本机内网IP,如192.168.0.107,代理服务器端口填写设置的端口，默认为8888，保存。
	![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/android-wifi-set.png)

3. 打开浏览器进行页面请求，第一次请求http请求时，电脑端会弹窗如下图，点击`Allow`,则手机端连上了电脑端的代理。
	![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/allow-android-join.png)

### 代理本机https请求

在`代理本机http请求`小节配置的基础上，在本机添加https的证书，并进行https请求监控设置。

配置步骤如下：

1. 进入`顶部菜单栏——Help——SSL Proxying`，点击`Install Charles Root Certificate`，界面跳转到钥匙串访问中。
2. 这时可以看到一个名称前缀为`Charles Proxy Custom Root Certificate`，有红叉标记的证书。
3. 双击该证书，`信任——使用次证书时`，始终信任。保存，证书图标变为蓝色。
4. 进入`顶部菜单栏——Proxy——SSL Proxying Setting...`,Location设置为Host:`*`,Port:`*`，通配所有域名和端口（一般https为443端口）。
5. 此时，就可以监控到https请求了。

![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/ssl-proxy-setting.png)

### 设置安卓端代理https请求

在`设置安卓端代理http请求`及`代理本机https请求`小节配置的基础上，在安卓端添加https请求证书。

配置步骤如下：


1. 进入`顶部菜单栏——Help——SSL Proxying`,点击`Install Charles Root Certificate..bile Device or Remote Browser`，出现下图所示弹窗。

	![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/android-https-dialog.png)

2. 按照提示，在浏览器中输入`chls.pro/ssl`，下载证书至手机本地，并为证书命名，如`charles-shenge`，此时，有些手机已经可以直接使用https代理了，如一加3.
3. 如果手机没有自动安装证书，那么我们需要手动安装证书，以华为P9为例，进入`设置——高级设置——安全和隐私：安全——凭据存储：从SD卡安装——找到之前下载的目录`，进行安装，安装成功后可以在`凭据存储：受信任的凭据——用户`中查看到我们安装的证书。

	![](https://shenpengyan.github.io/img/in-post/mac-tool-charles4/android-CA-install.png)

4. 此时，在手机端请求https请求，如<https://www.baidu.com> 则在Charles软件中可以看到相关的请求及详细信息。


### 总结

以上只是一个Charles的安装及简单使用的介绍，熟练使用http抓包工具对于web server程序员来说简直是福音，因为你可以通过抓包来熟悉每一个接口的使用场景（毕竟文档你们懂的），也可以在与FE、端开发者进行联调时快速定位问题，是你的锅背起来，不是你的锅坚决不认，有理有据。




