---
title: Objective-C runtime实践
header-img: post_header.jpg
tags:
  - null
categories:
  - null
date: 2018-08-22 11:41:27
---

## load函数与initialize函数作用和调用时机?

load函数: NSObject对象load函数是在app启动时main函数执行之前调用的. 先说继承关系，父类和子类都有实现load函数，则先执行父类load,再执行子类load. 

如果工程里同时存在NSObject多个category, 它们的调用时机是由编译器编译顺序决定的，理论上来说是后添加到工程里的category会先被执行. 可在build Phases里的Compile Source里调整编译顺序，Compile Source是个栈结构，后放进来的也就是靠近栈顶的先出栈被编译。这点无论是继承或者category都是如此。

如果同时存在继承关系和分类关系(都实现了load函数)，则先执行父类load, 然后子类load, 然后按照分类的规则去执行.

initialize函数: initialze函数只会在对象第一初始化时会被执行，一个对象的initialize函数只会执行一次，无论有几个category. 而且存在多个category时。会执行最后编译的文件的initialize函数，这点和load函数相反. 例如工程里有个Person对象，Person有A和B两个category. A先添加进工程，B后添加工程，则编译器会先编译B再编译A. initialize函数只有后编译的的A会执行. 

如果父类和子类同时重写了initialze函数，初始化父类时，只会执行一次父类的initialze函数.

初始化子类对象时，则会先执行父类的initialze，然后执行子类的initialze。如果父类存在category，则优先执行 后编译进来的category的initialze, 再执行子类的initialze。


## Method Swizzing
method swizzing即为方法交换，由于OC的运行时特性，程序在运行时调用某个方法，会由这个Method的selector(方法名)去找对应的implemention(方法实现). 我们要做的就是替换掉这个implemention，达到更改方法实现的目的. 

OC中一个Class的category是可以给这个Class扩展方法的，由上文中我们又知道了Class的load函数在main函数之前就会被执行。所以我们要做的就是在load函数中去做method swizzing.

以一个Person类为例，Person有个eat成员方法:
`
@implementation Person
- (void)eat{
    NSLog(@"%s",__func__);
}
`

新增一个Person的category, category中有个testEat方法:
`
@implementation Person (test)
- (void)testEat{
    NSLog(@"%s",__func__);
}
`

现在要做的就是在person实例对象在调用eat函数时，我需要打印出的是testEat函数，也就是执行testEat方法。这就是method swizzing(方法替换). 直接上代码:
     
    #import "Person+test.h"
    #import <objc/runtime.h>

    @implementation Person (test)
    + (void)load{
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            Class class = [self class];
            SEL originSelector = @selector(eat);
            SEL newSelector = @selector(testEat);
            Method originMethod = class_getInstanceMethod(class, originSelector);
            Method newMethod = class_getInstanceMethod(class, newSelector);
        
            BOOL didAddMethod = class_addMethod(class, originSelector, method_getImplementation(newMethod), method_getTypeEncoding(newMethod));
            //给当前class添加eat为名字testEat为实现的方法。
            //YES: 以前不存在eat的实现,新添加的方法成功
            //NO: 以前就存在eat的实现，所以新添加方法失败
            if(didAddMethod){  //添加eat为名字testEat实现成功，然后需要交换testEat为名字的方法的实现为eat。 双双互换。
                class_replaceMethod(class, newSelector, method_getImplementation(originMethod), method_getTypeEncoding(originMethod));
            }
            else{  //没添加成功，说明eat为名字的eat实现存在。 只需要交换eat和testEat两个的实现即可.
                method_exchangeImplementations(originMethod, newMethod);
            }
     });
    
    }

    + (void)initialize{
        NSLog(@"%s",__func__);      //initialized函数没有执行
    }

    - (void)testEat{
        [self testEat];     //testEat函数这里已经变成eat方法了。 不要理解成递归了
        NSLog(@"%s",__func__);
    }
    @end
    
我们在main函数中初始化一个person. 执行person的eat方法:
`
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSLog(@"Hello, World!");
        Person *p = [[Person alloc] init];
        [p eat];    //这里的eat方法已经被替换成testEat方法了。这就是method swizzing.
    }
    return 0;
}
`
这里会先打印出test函数，再打印出testEat函数. 实现了方法替换. 实际用例一般用在ViewController统计打点上， 对ViewWillAppear做方法替换，先执行原来的viewWillAppear逻辑，再执行我们要做的打点逻辑.



 

