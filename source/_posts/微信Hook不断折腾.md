---
title: 微信Hook不断折腾
date: 2018-05-26 10:05:43
tags:
---



折腾中遇到最多的问题就是装不上机器. 
1. !AMDeviceSecureInstallApplication问题
    这个错误一般都还是mobileprovision文件的问题。一定要是可用的，用个人的免费开发账号随便写个工程run一下。就可以从app里提取出一个这样的文件.
    
2. !AMDeviceSecureInstallApplication 这个是你在重签名时bundle id改的不全造成的，默认你用了你的免费开发证书重签后，会把app的bundle id改成你的包名，但是可能watchkit和extension的包名没改过来。就会导致这个问题。改成一致就好了
2. This app contains a WatchKit app with an invalid bundle identifier. The bundle identifier of a WatchKit app must have a prefix consisting of the companion app's bundle identifier, followed by a '.'.
    这个错误的原因是用了企业证书重签名，改过bundle ID 了，而bundle id 改的还不全，watchkit 和 extension的bundle id和app的bundle id对不上。 解决办法就是要么把主app重新改bundle id为微信官方的bundle id(com.tencent.xin)，这样还不会被微信查到. 要么就是把watchkit 和 extension的bundle id也改成你要的bundle id 。总之保持bundle id一致就行了.

3. 目前遇到的问题，是由于我用了企业证书重签，导致了收不到push推送了。网上有说企业证书重签也是可以实现接收push的，但你的企业证书得生成一个支持APNS的证书才可以。这个后续再更新.
4. 在猜测函数功能的时候，我们可以hook住这个函数，然后加个NSLog打印出参数。在Mac自带的console里删选，看log分析具体这个函数是做什么的。记得在NSLog后记得CHSuper调用原来的函数


