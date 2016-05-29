---
layout: post
date: 2016-01-09
title: PSJSWebview Javascript增强的WebView
tags: Objective-C
---

　　PSJSWebView是PSAspect的一个简单应用。

　　PSJSWebView为Javascript提供非常方便的调用OC代码的接口。现在已支持Json的基础数据类型。

　　OC调用Javascript直接使用`[UIWebView -stringByEvaluatingJavaScriptFromString]`方法即可。

### 2 代码示例
---
```objective-c
//Student.h
@interface Student : NSObject
- (void)doHomework;
- (void)makeFriendWith:(NSString *)friend;
- (void)sayTo:(NSDictionary *)dic;
//不支持此方法, Teacher不是Json基本类型, 不支持一个参数以上的方法
- (void)sayTo:(Teacher *)aTeacher words:(NSString *)words;
@end
```

```objective-c
//ViewController.m
    //向Javascript注册一个名为stu的JS对象
    self.student = [Student new];
    [self.webview addJavascriptInterface:self.student withJSObjectName:@"stu"];
```

```html
//Javascript
   <script>
       //这段代码执行stu对象下面的doHomework方法
       stu.execute("doHomework");
       //这段代码执行stu对象下的makeFriendWith:方法
       stu.execute("makeFriendWith", "Jack");
       //这段代码扫行stu对象下的sayTo:方法
       stu.execute("sayTo", {
            Jack: "hello",
            Mary: "hi",
            Lily: "I love you!"
       });
   </script>
```
### 3 Console.log
---
　　PSJSWebView覆盖了Console对象，并提供了log方法，实现在Javascript在Xcode中输出Log信息，方便开发者进行调试。

　　以下是代码示例：

```html
   <script>
      function onClick(){
         Console.log("this is a log.");
      }
   </script>
```
> 需要设置webview.ps_printLogs = YES;