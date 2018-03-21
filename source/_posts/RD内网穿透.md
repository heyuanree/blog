---
title: RD+内网穿透
tags:
  - web
categories: []
date: 2017-11-08 22:27:02
---

## 前言

这个技术在内网渗透中应该是极为常见的，但这次处于比赛的缘故，我不得不学习了起来。另外，之前一白师傅用的vpn内网穿透的方法还需要研究一下，初步猜测是一个有公网ip的vpn+4g网卡。

## 内网穿透

研究了一下，目前开源的方法主要是[ngork](https://github.com/inconshreveable/ngrok)，[frp](https://github.com/fatedier/frp)，[lanproxy](https://github.com/ffay/lanproxy)，最终从配置的便捷性上选择lanproxy。

lanproxy的安装非常简单，下载server和client的release即可，服务器端需要python3和相应的包，client需要java1.8。

## 家庭版win10开启RD

家庭版在win10中阉割了RD，取而代之的是一个叫做**远程协助**的功能。emmmm，对此，在我查到解决方法之前就像吃了苍蝇一样。。。

后来在github上看到了一个解决方案，[rdpwarp](https://github.com/stascorp/rdpwrap)，粗看了一下，应该是把pro版本的RD中的相关ini和exe文件重新安装在了机器上，然后开启了这个服务。但是在使用过称中发现了还有一个叫`rfxvmt.dll`的文件缺失，需要下载并替换[link](https://github.com/stascorp/rdpwrap/issues/194#issuecomment-325627235)，目录是`system32`和`SysWOW64`这两个文件夹。