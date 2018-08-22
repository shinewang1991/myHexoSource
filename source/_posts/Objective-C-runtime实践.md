---
title: Objective-C runtime实践
header-img: post_header.jpg
tags:
  - null
categories:
  - null
date: 2018-08-22 11:41:27
---

load函数与initialize函数作用和调用时机?

load函数: NSObject对象load函数是在app启动时main函数执行之前调用的. 先说继承关系，父类和子类都有实现load函数，则先执行父类load,再执行子类load. 

如果工程里同时存在NSObject多个category, 它们的调用时机是由编译器编译顺序决定的，理论上来说是后添加到工程里的category会先被执行. 可在build Phases里的Compile Source里调整编译顺序，Compile Source是个栈结构，后放进来的也就是靠近栈顶的先出栈被编译。这点无论是继承或者category都是如此。

如果同时存在继承关系和分类关系(都实现了load函数)，则先执行父类load, 然后子类load, 然后按照分类的规则去执行.

initialize函数: initialze函数只会在对象第一初始化时会被执行，一个对象的initialize函数只会执行一次，无论有几个category. 而且存在多个category时。会执行最后编译的文件的initialize函数，这点和load函数相反. 例如工程里有个Person对象，Person有A和B两个category. A先添加进工程，B后添加工程，则编译器会先编译B再编译A. initialize函数只有后编译的的A会执行. 

如果父类和子类同时重写了initialze函数，初始化父类时，只会执行一次父类的initialze函数.

初始化子类对象时，则会先执行父类的initialze，然后执行子类的initialze。如果父类存在category，则优先执行 后编译进来的category的initialze, 再执行子类的initialze。
 

