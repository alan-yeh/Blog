---
layout: post
date: 2016-03-07
title: PSPromise在项目中实践
tags: Objective-C
---
　　经过上一篇博客，对于[PSPromise](/blog/PSPromise)的介绍，对Promise的相关语法大概了解了。这篇博客主要介绍PSPromise在项目中如何实践，如何优雅地写代码。代码已上传至[GitHub](https://github.com/Poi-Son/Examples/tree/master/PromiseNetwork)中。

　　本项目使用了AFNetworking、SVProgressHUD、PSPromise三个框架。项目主要内容是，获取豆瓣网上一本书的标题与简介(字段太多了，只取了两个简单的字段)。通过Promise，将整个流程串起来。
###将AFNetworking封装
---
　　AFNetworking是通过success block和failure block的回调来简化网络访问。通过Promise，我们将它再次简化。项目中仅封装了get方法。

```objective-c
PSPromise *__get(NSString *path, id param){
    return PSPromiseWithBlock(^id{
        //访问网络前，先检测网络状态
        if ([AFNetworkReachabilityManager sharedManager].isReachable) {
            return @YES;
        }else{
            return NSErrorMake(@"当前网络不可用", nil);
        }
    }).thenPromise(^(id null, PSResolve resolve) {
        //访问网络
        AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
        manager.responseSerializer = [AFHTTPResponseSerializer serializer];
        [manager GET:path
          parameters:param
            progress:nil
             success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
                 resolve(responseObject);
             }
             failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
                 resolve(NSErrorMake(@"网络访问错误", error));
             }];
    });
}
```
　　以上代码片段，将AFNetworking封装成业务无关的方法，并在网络访问前检测网络状态，同时给出各类错误提示。
> AFNetworking触发failure的条件不一定就是网络超时之类的错误，当服务器返回404、500之类的也同样会被failure处理。所以在封装AFNetworking时，需要按实际业务来进行错误提示。

###封装Json序列化
---
　　实际上，AFNetworking已经实现了AFJSONResponseSerializer用于实现自动将服务器返回的字符转换成Json对象。但是为了做演示，同时为了让AFNeworking的逻辑更单一，将序列化单独抽离开来。将来如果服务器采用XML作为通信协议的话，我们只需要将Json序列化变成XML序列化便可以兼容新接口了。

```objective-c
PSPromise *__json(NSData *responseObject){
    return PSPromiseAsyncWithBlock(^id{
        if (![responseObject isKindOfClass:[NSData class]]) {
            return NSErrorMake(@"无法序列化对象", nil);
        }
        
        NSError *error;
        NSDictionary *jsonData = [NSJSONSerialization JSONObjectWithData:responseObject options:kNilOptions error:&error];
        if (error) {
            return NSErrorMake(@"Json 序列化错误", error);
        }else{
            return jsonData;
        }
    });
}
```
　　同样的，json序列化的代码也比较单一，但是对参数进行了一些校验，对序列化错误也有一些提示优化。
###BookDetail实体
---
　　BookDetail实体仅仅只是一个Model，用于保存传递一些信息。所以这部分比较简单，不作解释。

```objective-c
@interface BookDetail : NSObject
- (instancetype)initWithJsonObject:(NSDictionary *)json;

@property (nonatomic, copy) NSString *title;
@property (nonatomic, copy) NSString *summary;
@end

@implementation BookDetail
- (instancetype)initWithJsonObject:(NSDictionary *)json{
    if (self = [super init]) {
        self.title = json[@"title"];
        self.summary = json[@"summary"];
    }
    return self;
}
@end
```

###封装Services层
---
　　部份项目结构没有Services层，有的可能会直接封装成Model层。这里仅给出一个演示。Promise是一个非常灵活的框架，具体项目可以具体分析。

```objective-c
@interface DoubanServices : NSObject
- (PSPromise<BookDetail *> *)getBookDetailByID:(NSString *)bookID;
@end

@implementation DoubanServices
- (PSPromise<BookDetail *> *)getBookDetailByID:(NSString *)bookID{
    return PSPromiseWithBlock(^{
        //访问网络
        return __get([@"https://api.douban.com/v2/book/" stringByAppendingString:bookID], nil);
    }).then(^(id responseObject){
        //解析成Json
        return __json(responseObject);
    }).thenAsync(^(NSDictionary *json){
        //解析成实体
        return [[BookDetail alloc] initWithJsonObject:json];
    });
}
@end
```
　　Services层的逻辑也非常纯粹，毫不拖拉。

* 第一步，访问网络，直接返回__get生成的Promise就完成了网络访问。
* 第二步，将网络访问返回回来的数据解析成Json，同样的，直接返回__json生成的Promise就完成了。
* 第三步，将Json解析成实体。

###ViewController
---
　　ViewController中，btnFetchDetail点击后触发事件，然后获取后将结果绑定到界面的UITextField和UITextView中。

```objective-c
- (IBAction)btnFetchDetailCliced:(UIButton *)sender {
    PSPromiseWithBlock(^{
        //任务开始前显示HUD提示
        [SVProgressHUD showWithStatus:@"加载中"];
    }).then(^{
        //获取详情
        return [[DoubanServices new] getBookDetailByID:@"1220562"];
    }).then(^(BookDetail *detail){
        //将实体中的信息绑定到界面元素中
        self.tfTitle.text = detail.title;
        self.tvSummary.text = detail.summary;
    }).catch(^(NSError *error){
        //统一处理所有错误
        [[[UIAlertView alloc] initWithTitle:@"错误"
                                   message:error.localizedDescription
                                  delegate:nil
                         cancelButtonTitle:@"确认"
                          otherButtonTitles:nil] show];
    }).always(^{
        //在处理完毕之后，统一将HUD隐藏掉
        [SVProgressHUD dismiss];
    });
}
```
　　现在点击一下界面的按扭，可以看到数据成功加载了。

![](http://7xqack.com1.z0.glb.clouddn.com/16-3-7/48084267.jpg)

　　关闭网络，再来测试一下。

![](http://7xqack.com1.z0.glb.clouddn.com/16-3-7/92967144.jpg)

###总结
---
　　通过Promise的抽象，AFNetworking被隔离开来了，Json序列化也被隔离开来了，而且它们都具有高度可测试性和可复用性的一些代码。通过分离，我们不再需要关注那些错误了，更集中精力去处理业务的正确流程。