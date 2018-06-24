---
title: iOS 子线程中NSTimer中的启动与关闭
date: 2018-06-10 16:16:47
header-img: "post_header.jpg"
tags:
    - iOS
    - 技术
---

> 同事今天突然过来问到我一个问题，在GCD中的创建的子线程中如何将一个timer销毁掉？ 当时想了会儿还真想到不错的思路。因为在子线程中创建的timer肯定要在子线程中才能拿到该timer，然后才能关闭和销毁。 而如果仅仅是GCD创建出来的线程，如何在主线程中拿到该线程呢，当时我对GCD的感官只是并发编程中的一个方案，是没有办法像NSThread一样单独去操作某个线程的。当时我给他的解决方案是，不要把timer放子线程里了，直接放在主线程操作，就不会有这问题了。 但是这个问题我觉得放在子线程里肯定还是有办法能解决的，于是写了个demo进行了一番探索。。。

首先我们来看看这段代码

```
static dispatch_once_t onceToken;
__weak typeof(self) weakSelf = self;
dispatch_once(&onceToken, ^{
        dispatch_async(dispatch_queue_create("com.test.demo", DISPATCH_QUEUE_CONCURRENT), ^{
        weakSelf.timer = [NSTimer timerWithTimeInterval:3 target:weakSelf selector:@selector(updateTimer) userInfo:nil repeats:YES];
        [[NSRunLoop currentRunLoop] addTimer:weakSelf.timer forMode:NSRunLoopCommonModes];
        [[NSRunLoop currentRunLoop] run];
    });
});

- (void)updateTimer{
    NSLog(@"currentThread -------- %@", [NSThread currentThread]);
}
```

如果我现在放一个stop的button, 用于停止这个timer. 一般情况我们可能会直接在button的响应事件里直接这样做

```
NSLog(@"stopTimerThread -------- %@", [NSThread currentThread]);
if(self.timer.isValid){
    [self.timer invalidate];
    self.timer = nil;
}
```

经过实验这样是可以停止的。但是这个我确实没太理解，跨线程能操作timer吗？

为了让停止timer的方法也在创建timer的线程中去执行，会想到使用线程间的通信，我们改造一下这个创建timer的函数:

```
static dispatch_once_t onceToken;
__weak typeof(self) weakSelf = self;
dispatch_once(&onceToken, ^{
        dispatch_async(dispatch_queue_create("com.test.demo", DISPATCH_QUEUE_CONCURRENT), ^{
        weakSelf.timerThread = [NSThread currentThread];
        weakSelf.timerThread.name = @"createTimerThraed";
        weakSelf.timer = [NSTimer timerWithTimeInterval:3 target:weakSelf selector:@selector(updateTimer) userInfo:nil repeats:YES];
        [[NSRunLoop currentRunLoop] addTimer:weakSelf.timer forMode:NSRunLoopCommonModes];
        [[NSRunLoop currentRunLoop] run];
    });
});

- (IBAction)stopEvent:(id)sender {
    if(self.timerThread){
        [self performSelector:@selector(stopTimer) onThread:self.timerThread withObject:nil waitUntilDone:NO];
    }
}

- (void)stopTimer{
    NSLog(@"stopTimerThread -------- %@", [NSThread currentThread]);
    if(self.timer.isValid){
        [self.timer invalidate];
        self.timer = nil;
    }
}
```

这样根据打印出来的日志来看，创建timer和停止timer确实是在同一个线程了，尽管上面在主线程中也是能够stop这个timer，但严谨的来说还是应该遵循"哪个线程创建，哪个线程销毁”的原则比较好。

