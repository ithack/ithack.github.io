---
title: Chrome inspect调试webview的强制开启方法
date: 2017-10-18 15:41:01
tags: [APP调试]
---

对于inspect很多前端开发并不陌生，我们经常用inspect调试我们移动端页面嵌套进原生webview的页面；但这个调试方法需要安卓4.4以上版本支持，但同时由于安卓本身是开源的国内很多手机厂商都在安卓的基础上做了封装再开发，导致有些手机我们即使在inspect里识别了当前手机，但用浏览器或webview打开页面时却不能看到我们的html，之前在开发时遇到，在google搜索了一堆的解决方案，都是要安卓开启debug模式，但app内部测试版本就是开启状态，那么如何解决我们才能调试我们自己的html页面呢？之前搜到过资料没有进行记录，今天调试时又遇到了这个问题是个小米手机，安卓版本4.4.4；接下来看看怎么解决这个问题；

## 基于webkit核心的webview端调试

从 **Android 4.4** 开始，webkit是支持远程调试的，不过需要将app的debug模式打开，可以使用如下代码：

```
WebView.setWebContentsDebuggingEnabled(true);12
```

由于大部分 App 的 debug 模式是关闭的，即便是内部 App，比如 QQ/微信，要去找一个开启了debug 模式的版本还是比较麻烦的。因此需要使用借助第三方工具来强制开启任何 App 的 **Android webview debug模式**，使之可以使用 chrome inspect。**而这个工具就是 Xposed **。

我们已经提供了一份要安装的文件，请首先到 <https://github.com/feix760/WebViewDebugHook> 下载文件。

### 1、root设备

因为涉及到 root 权限，因此需要将手机进行 root。有很多工具可以来 root，比如KingRoot、一键root、360一键root等。如果你安装了QQ电脑管家，可以在“电脑管家-工具箱-其他”列表里面看到KingRoot。 <!--more-->

### 2、安装xposed框架



在下载文件的hook.zip中，找到 **de.robv.android.xposed.installer_v33_36570c.apk**，安装之。也可以去 [官网](http://repo.xposed.info/module/de.robv.android.xposed.installer) 下载。 

### 3、安装xposed webview debugging模块

在下载文件的hook.zip中，找到 **WebViewDebugHook.apk**，安装之。 

### 4、激活Xposed

安装后上述两个apk之后，可以看到手机上面出现了一个叫 **Xposed Installer** 的图标，点击进去之后会看到提示说Xposed未激活，点击红色字体部分，会切换到另外一条页面，点击“安装/更新”按钮即可。

![1](\images\webview调试\1.jpg)

但有部分手机会出现类似如下的错误，导致无法点击“安装/更新”，目前已知的是部分版本的MIUI是会出现这个问题的。

![1](\images\webview调试\2.jpg)

安装完成之后，重启再打开，再点击刚才点击过的地方，切入页面之后，勾选，再重启，重启之后即激活了Xposed。

![1](\images\webview调试\3.jpg)

![1](\images\webview调试\4.jpg)

### 5、关于QQ等

QQ等默认会使用X5内核，把下载文件中的 **debug.conf** 放在sd卡根目录下就可以强制它使用 Android 自带 webview 。 

### 6、测试

手机usb连接电脑，使用 chrome 打开 chrome://inspect ，然后打开任意 App 的 webview ，接下来就是见证奇迹的时候了。