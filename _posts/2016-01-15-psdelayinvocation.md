---
layout: post
date: 2016-01-15
title: PSDelayInvocation 间隔延时调用
tags: Objective-C
---

　　刚刚遇到一个问题，大概是这样的：UISlider滑动时，将UISlider当前值发送给蓝牙。很简单的需求，但是有一个问题，就是不能过于频繁地向蓝牙设备发送消息，否则蓝牙设备由于处理不过来而断开连接。最后测试结果是，最小间隔不能低于0.4秒，否则可能断开连接。

　　刚开始，我觉得很简单。不能频繁调用嘛，用[PSLockFunc](https://github.com/Poi-Son/PSExtensions/blob/master/PSExtensions/PSExtensions/Foudation/Extensions/NSObject_Kit.h)将它锁起来就可以了，0.4秒之后再解锁。完成任务！

　　发给客户之后，不久客户向我反馈，说蓝牙现在很稳定不会断开了，但是有一个问题就是快速滑动UISlider之后，要再点击一次控件才能更新蓝牙设备的状态。我想了一会，发现`PSLockFunc`是将频繁调用的操作丢弃了，所以快速滑动之后，最后一次操作被丢弃了，需要再点击一次，发送多一次命令给蓝牙设备才能更新。

　　知道问题出现在哪之后，就应该去解决这个问题。第一反应是使用个变量去记录最后一次操作的值，然后在停止调用之后再去向蓝牙设备发送这个值。但是这个操作是很频繁操作的，这个`最后一次操作`也在频繁变更，也无法确认在<mark>适当的时机</mark>将最后一次操作发送给蓝牙，所以这个可行性应该不高。

　　后面在搜资料的时候，发现iOS提供了一个静态方法`[+ cancelPreviousPerformRequestsWithTarget:]`，用于取消`[- performSelector:withObject:afterDelay:]`系列的操作。配合这个方法，我将这类需求封装成PSDelayInvocation，用于解决类似的问题。

###解决思路
---
　　将所有操作封装成NSInvocation，放到一个栈中，然后每隔一段间隔去执行栈顶的操作，同时延迟X秒之后去取栈顶的操作执行（解决需要再点一次才会发送最新的状态给蓝牙设备）。Talk is cheap, show you the code.

　　最终代码。

```objective-c
//ViewController.m
@implementation ViewController{
    PSDelayInvocation *_delayInvocation;
}

- (PSDelayInvocation *)delay{
    return _delayInvocation ?: ({
        _delayInvocation = [PSDelayInvocation new];//初始化
        _delayInvocation.delay = 0.4;//间隔0.4秒发送最新状态
        _delayInvocation;
    });
}

- (IBAction)sliderValueChanged:(UISlider *)sender{
    //将需要处理的业务交给PSDelayInvocation去调用
    [[self.delay prepareWithTarget:self] sendValueToBluetooth:(int)sender.value];
}

- (void)sendValueToBluetooth:(int)value{
    //发送给蓝牙设备的代码
}
@end
```
###实现细节
---
　　将所有操作封装成NSInvocation。`[PSDelayInvocation -prepareWithTarget:]`返回一个NSProxy对象，用于将操作封装成对象。

```objective-c
//PSDelayInvocation.m
- (id)prepareWithTarget:(id)target{
    self.proxy.target = target;
    return self.proxy;
}
```
　　保存操作到栈中，同时去调用`[- exeInvocation]`方法。exeInvocation会判断执行间隔有没有满足间隔，如果满足则取出栈中最上一个操作，调用并清空栈；如果没有满足间隔，则延迟调用自身`[- exeInvocation]`。

```objective-c
//PSDelayInvocation.m
- (void)exeInvocation{
    static NSTimeInterval lastExeTime = 0;
    NSTimeInterval now = [[NSDate date] timeIntervalSince1970];
    if (now - lastExeTime < self.delay) {
        [self.class cancelPreviousPerformRequestsWithTarget:self];
        [self performSelector:@selector(exeInvocation) withObject:nil afterDelay:self.delay];
        return;
    }
    lastExeTime = now;
    
    NSInvocation *invocation = [self.stack lastObject];
    [self.stack removeAllObjects];
    
    if (invocation != nil) {
        [invocation invoke];
    }
}
```
　　完成这一步之后，用户的需求完美满足了。

　　以上代码已集成到[PSExtension](https://github.com/Poi-Son/PSExtensions)中，PSDelayInvocation的[源代码地址](https://github.com/Poi-Son/PSExtensions/blob/master/PSExtensions/PSExtensions/Foudation/Extensions/NSObject_Kit.h)。
