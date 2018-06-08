---
title: 存储SHSH blobs
date: 2018-06-08 07:33:36
header-img: "post-bg.jpg"
tags:
	- iOS
categories:
       - 技术
---

最近国外coolStar大神即将发布iOS 11.3.1的越狱，所以特别想再次体验下越狱。记忆中还是当年iOS 7的是时候把我的iPhone 5越狱了玩了一把。后来就陆续错过了很多越狱版本。虽然越来越多的声音说越狱已经没那么有必要了，该有的功能不用越狱苹果也为我们做了。但是如果作为一个开发者，特别是对逆向这个方向有兴趣的话，我觉得还是很有必要的。今天我要写的是在你的手机越狱之前如果保存SHSH blobs？

什么是SHSH blobs， 做什么用的？

你就可以这样理解，SHSH blobs文件是用来降级的。 例如苹果某个系统版本已经关闭验证了（我在写这篇文章的时候，苹果刚关了11.3.1的验证, 所以尝鲜要趁早），但是我以后如果升级到新的系统之后，还想有机会降级到之前的版本，就必须用到SHSH blobs。所以说SHSH blobs对于降级很重要。

如何存SHSH blobs?
存blobs切记一定要在苹果关闭当前版本验证之前，否则貌似就是保存不了了，我猜测存储SHSH blobs也要走苹果某个服务验证. 我以iOS 11.3.1举栗子，操作步骤:
## 第一步:获取 ECID
* 首先你得有台PC，安装上iTunes, 连上手机,进入summary,单击Serial Number那一栏，一直等到出现ECID之后，复制拿到ECID.
## 第二步:查看Model Identifier
* 跟上面步骤一样，还是在iTunes里，单击summary那一行,等到出现Model Identifier.这就是你的Model Identifier。
## 第三步:存储SHSH blobs
* 进入网站https://tsssaver.1conan.com
* 粘贴你的ECID到ECID那一栏,注意选择框是选择Hex(iTunes).
* 选择你的iPhone Model Identifer. 就是第二步刚刚看到的.
* 然后submit. 你就会跳转到一个下载页面, 你的SHSH blobs就存在那个网盘里了。


你可能会问到，这个文件需要下载下来自己保存吗？ 不用，这个服务器会为你存储，以后就只需要进入刚才那个page, 下面有个Lost your link ,输入你的ECID, 即可找回你的blobs，还是很方便的。

还有一点，貌似你将你的ECID上传到这个服务器后，以后每个系统版本，服务器都会自动为你保存各个版本的blobs,你都不用再做什么了。

好，SHSH blobs文件我们已经知道如何存储了。我们就可以放心的去享受越狱了，哪怕以后系统升级了，越狱没了。还可以有办法降级回来了！




