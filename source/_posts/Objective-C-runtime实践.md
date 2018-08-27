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


## 消息转发
我们首先来做个实验，给一个class发送一个不存在的消息.

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        instrumentObjcMessageSends(YES);
        
        Person *p = [[Person alloc] init];
        [p performSelector:@selector(eat)];   //eat方法是不存在的
    }
    return 0;
}
```
 
说明一下这里的instrumentObjcMessageSends，这是runtime里的系统方法。用来记录所有消息调用的日志，日志会存在/private/tmp目录. 

```
instrumentObjcMessageSends(YES);    //开始记录，一般写在你要记录的函数之前
instrumentObjcMessageSends(NO);     //停止记录，一般写在你要记录的函数之后

```
run一下程序，程序会crash在 `-[Person eat]: unrecognized selector sent to instance `,我们再打开日志观察

```
+ Person NSObject initialize
+ Person NSObject alloc
- Person NSObject init
- Person NSObject performSelector:
+ Person Person resolveInstanceMethod:
+ Person Person resolveInstanceMethod:
+ NSObject NSObject resolveInstanceMethod:
- Person NSObject forwardingTargetForSelector:
- Person NSObject forwardingTargetForSelector:
- Person NSObject methodSignatureForSelector:
- Person NSObject methodSignatureForSelector:
- Person NSObject class
- Person NSObject doesNotRecognizeSelector:
- Person NSObject doesNotRecognizeSelector:
- Person NSObject class
```

我们可以看到Person这个class在调用performSelector方法之后，会依次调用
-> [Person resolveInstanceMethod]类方法 
-> [NSObject resolveInstanceMethod]父类方法 
-> [Person - forwardingTargetForSelector]实例方法 
-> [Persion - methodSignatureForSelector]实例方法
-> [Persion - doesNotRecognizeSelector]实例方法

我们可以看到，给一个class发送某个消息，系统会经历3个步骤去尝试动态转发处理这个消息，看是否有合适补救措施。这三种转发方式都不行，再报unrecognized selector sent to instance错误.

我们来分别研究下这三个步骤:
* resolveInstanceMethod. 通过查dash文档的解释是动态给一个实例方法提供一个IMP.  也就是我们可以动态给这个对象增加一个本来不存在的方法. 对应的还有个类似的resolveClassMethod，就是给一个类方法增加IMP.

我们试着用这个方式动态给这个Person添加一个eat方法.

```
#import "Person.h"
#import <objc/runtime.h>

void dynamicMethodIMP(id self, SEL _cmd){
    //implementaion...
    printf("eat IMP is here \n");
}

@implementation Person
+ (BOOL)resolveInstanceMethod:(SEL)sel{
    if(sel == @selector(eat)){
        class_addMethod([self class], sel, (IMP)dynamicMethodIMP, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```
然后再编译，就不会报错crash了。

* forwardingTargetForSelector:(SEL)aSelector. 这个方法让你将这个原对象无法处理的消息转给另一个对象尝试去处理。所以我们再创建一个Pet对象，并实现eat函数.

```
@implementation Pet
- (void)eat{
    NSLog(@"%s",__func__);
}
@end

```

Person类中实现forwardingTargetForSelector方法，返回一个pet对象。这样就将eat消息交给pet对象去处理了.

```
@interface Person : NSObject
@property (nonatomic, strong) Pet *pet;
...
@end
@implementation Person
- (instancetype)init{
    if(self = [super init]){
        Pet *pet = [[Pet alloc] init];
        self.pet = pet;
    }
    return self;
}

...
- (id)forwardingTargetForSelector:(SEL)aSelector{
    if([self.pet respondsToSelector:aSelector]){
        return self.pet;
    }
    return nil;
}

@end
```

再run一下程序，不会crash了，并且打印出了[Pet eat]

