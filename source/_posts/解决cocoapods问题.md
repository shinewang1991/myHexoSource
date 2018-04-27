---
title: 解决cocoapods问题
date: 2018-04-25 10:26:43
tags:
    - iOS
---

网上搜了很多方案，都没有解决我的问题。最后看到这个方案，是可行的

移除cocopods后重新安装
1、打开终端，运行sudo gem install cocoapods-deintegrate安装快速解除项目cocopods依赖的库
2、安装成功后，cd到你项目的更目录运行pod deintegrate解除项目cocopods依赖
3、运行pod install,重新安装cocopods

