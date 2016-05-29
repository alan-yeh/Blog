---
layout: post
date: 2016-03-05
title: PSPromise Promise的简易实现
tags: Objective-C
---
　　[PSPromise](https://github.com/Poi-Son/PSPromise)是Promise的简易实现。遵循CommonJS的Promise/A接口标准，同时实现了一些扩展接口，使PSPromise更实用。

　　近期看到一个很有意思的框架[PromiseKit](https://github.com/mxcl/PromiseKit)。看到PromiseKit的源码之后，对它的编写方式感到神奇，刚开始完全看不懂在写什么东西，而且代码是怎么运行的也完全没个头绪。仔细分析了大半天之后，感叹大神写代码的确与众不同。由于个人的一些“不良嗜好”，决定写一个简易版的Promise，同时实践一下这种新型语法。

###简介
---
####什么是Promise
　　Promise，承诺，在开发中的意思是，我承诺我去做一些事情，但不一定现在就去做，而是在将来满足一些条件之后才执行。Promise刚开始出现在前端开发领域中，主要用来解决JS开发中的异步问题。在使用Promise之前，异步的处理使用最多的就是用回调这种形式，比如：

```javascript
doSomethingAsync(function(result, error){
    if (error){
        ...//处理error
    } else {
        ...//处理result
    }
})
```
　　在Objective-C中，这类代码也是非常常见的。例如著名的AFNetworking中，访问网络就是使用block回调。

```objective-c
    [client getPath:@"xxx"
         parameters:params
            success:^(AFHTTPRequestOperation *operation, id responseObject) {
                //处理success
            }
            failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                //处理failure
            }]
```

　　这种书写方式，可以很容易解决对异步操作的处理。但是这样的写法，很容易引起回调金字塔的情况。Promise则对异步处理和处理方法都做了规范和抽象，还给了开发者在异步代码中使用return和throw的能力。这也是Promise存在的真正意义。
####使用Promise
　　来看一个常见的业务场景，获取联系人需要先访问一次服务器(登录或一些必要的操作)，然后再访问一次服务器才能真正获取到有效数据，然后再进行一系列的错误处理，代码冗余复杂。

```objective-c
- (void)getContactSuccess:(void(^)(NSArray *))success failure:(void(^)(NSError *))failure{
    [client getPath:@"xxx"
         parameters:params
            success:^(AFHTTPRequestOperation *operation, id responseObject) {
                //处理Json序列化
                NSError *error;
                NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:responseObject options:kNilOptions error:&error];
                if (error) {
                    failure(error);
                    return;
                }
                if ([dic[@"status"] intValue] == 200) {
                    //需要依赖上一次网络访问的结果，进行下一步访问
                    [client getPath:@"yyy"
                         parameters:params
                            success:^(AFHTTPRequestOperation *operation, id responseObject) {
                                //处理Json
                                //处理业务
                                //组装实体
                                //....
                                success(result);
                            }
                            failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                                //处理网络错误
                                failure(error);
                            }];
                }else{
                    //处理错误
                    failure(/*返回错误*/);
                }
            }
            failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                //处理网络错误
                failure(error);
            }];
}
```

　　使用Promise改造一下

```objective-c
//网络访问与业务分离
- (void)get:(NSString *)getPath withParam:(id)param andResolve:(PSResolve)resolve{
    [client getPath:@"xxx"
         parameters:param
            success:^(AFHTTPRequestOperation *operation, id responseObject) {
                resolve(responseObject);
            } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                resolve(error);
            }];
}
//Json序列化与业务分离
- (void)parseJson:(id)responseObject andResolve:(PSResolve)resolve{
    NSError *error;
    NSDictionary *json = [NSJSONSerialization JSONObjectWithData:responseObject options:kNilOptions error:&error];
    if (error) {
        resolve(error);
    }else{
        resolve(json);
    }
}

- (PSPromise *)getContacts{
    return PSPromiseWithResolve(^(PSResolve resolve) {
        //第一次访问网络
        [self get:@"xxx" withParam:params andResolve:resolve];
    }). thenPromise(^id(id responseObject, PSResolve resolve){
        //处理Json序列化
        [self parseJson:responseObject andResolve:resolve];
    }).thenPromise(^(NSDictionary *json, PSResolve resolve){
        //第二次访问网络
        [self get:@"yyy" withParam:params andResolve:resolve];
    }). thenPromise(^(id responseObject, PSResolve resolve){
        //再次处理Json序列化
        [self parseJson:responseObject andResolve:resolve];
    }).then(^(NSDictionary* result){
        //处理业务正确性
        if ([result[@"status"] intValue] == 200){
            return result;
        }else{
            return error;/*构建一个NSError对象返回*/
        }
    }).then(^(NSDictionary* result){
        //组装实体
    });
}
```

　　可以看到，通过Promise的改造，原本层层嵌套代码，变得有序、清晰起来。什么？你问哪里处理错误？放心，通过Promise，可以进行统一的错误处理。
###Promise状态
---
　　每个Promise都只会被成功或失败一次，并且这个状态不会被改变。

　　一个Promise必须处于以下几个状态之一：

* PSPromiseStatePending(Pending): 操作正在执行中(等待执行)，可以转换到Fulfilled或Rejected状态。
* PSPromiseStateFulfilled(Fulfilled): 操作执行成功，且状态不可改变
* PSPromiseStateRejected(Rejected): 操作执行失败，且状态不可改变

　　有些文章或Promise实现会出现第4种状态Settled，代表操作已结束，可以认为Settled = Fulfilled & Rejected。他本身并不是一种状态，因为非Pending就是Settled，只是为了说的方便而引入Settled这个说法。

　　Promise处于Pending状态时，其value一定为nil；处于Fulfilled状态时，其value为处理结果，可能为nil；处于Rejected状态时，其value一定为NSError对象，用于描述Promise被拒绝原因。

###CommonJS Promise/A
---
　　PSPromise支持标准的CommonJS Promise/A语法。由于语言的特殊，对其中部份Api进行小量改造。
####构造函数
　　由于PSPromise使用链式调用语法，抛弃OC的传统构造函数，使用C构造PSPromise对象更有利于书写方便。PSPromise有以下构造函数：

* PSPromise *PSPromiseWithResolve(void (^)(PSResolve resolve))

```objective-c
PSPromise *promise = PSPromiseWithResolve(^(PSResolve resolve){
    resolve(@"aaa");
});
```

* PSPromise *PSPromiseWithBlock(id block)

```objective-c
PSPromise *promise = PSPromiseWithBlock(^{
    return @"aaa";
});
```

####then
　　then语法用于处理正确(Fulfilled)的Promise结果。在CommonJS Promise/A的语法中，then应该支持2个回调函数，第一个回调函数处理Fulfilled的结果，第二个回调函数处理Rejected的结果。为了减少其书写上的复杂性，PSPromise的then语法仅支持1个block，用于处理正确(Fulfilled)结果。

```objective-c
promise.then(^(NSString *result){
	return [result stringByAppendingString:@"bbb"];
});
```

　　then语法必须接受一个不为空的block类型的参数。then语法中的block支持无参或1个参数。

　　如果block：

* 返回普通OC对象(包括nil)，当前Promise状态变更成Fulfilled，vlaue被赋值为返回值。
* 没有返回值，当前Promise状态变更成Fulfilled，value被赋值为nil。
* 返回NSError对象，当前Promise状态变更成Rejected，value被赋值为NSError对象。
* 返回Promise对象，当前Promise对象抛弃，当前Promise链之后的Promise被托管至返回的Promise对象。
* 抛出错误(仅捕捉NSError错误，不捕捉其它对象)，当前Promise状态变更成Rejected，value被赋值为NSError对象。

####catch
　　catch语法用于处理被拒绝(Rejected)的Promise对象的结果。

```objective-c
PSPromiseWithResolve:(^(PSResolve resolve){
    NSError *error = ...;
    resolve(error);
}).then(^{
	//由于Promise为Rejected，因此then不执行
}).catch(^(NSError *error){
	//处理Promise为Rejected
}).catch(^(NSError *error){
	//由于当前的catch之前，Rejected的Promise已经被处理了，所以这个catch不执行
});
```
　　catch语法必须接受一个不为空的block类型参数。catch语法中的block支持无参或1个NSError参数。catch专门用于处理被拒绝(Rejected)的Promise，then专门处理正确(Fulfilled)的Promise。

```objective-c
PSPromiseWithResolve:(^(PSResolve resolve){
    resolve(@"aaa");
}).then(^{
	NSError *error;//业务产生了一个错误
	@throw error;//直接抛出异常可以被下一个catch捕捉到
	//return error;//直接返回也可以被下一个catch捕捉到
}).catch(^(NSError *error){
	//可以处理catch之前错误
});
```
####all
　　all语法是静态类函数，接受一个Promise数组，并返回一个包装后的Promise对象，称之为A。A的状态改变有两个条件：

1. 当数组中所有的promise对象变成成功状态(Fulfilled)，这个包装后的A才会把自己变成成功状态。A会等待最慢的那个promise对象变成成功态后才把自己变成成功态，并将promise数组的结果封装成它们的结果数据。
2. 只要其中一个promise对象变成失败状态(Rejected)，这个包装后的A就变成失败状态，并且第一个rejected promise传递过来的NSError值会传递给A后面的catch。

　　因此，all语法可以理解为判断语句『且』，当所有条件都成功时它才成功，当有一个条件失败时，它就是失败。

```objective-c
PSPromise *p1 = PSPromiseWithBlock(^{
    return @"aa";
});
PSPromise *p2 = PSPromiseWithBlock(^{
    @throw error;
});
PSPromise *p3 = PSPromiseWithBlock(^{
    return @"cc";
});

PSPromise *pAll = PSPromise.all(@[p1, p2, p3]).then(^(NSArray *result){
    //then语法不会执行，因为p2抛出异常了
}).catch(^(NSError *error){
    //catch会执行，捕捉到p2所抛出来的错误
});
    
/************************************************************/

PSPromise *p1 = PSPromiseWithBlock(^{
    return @"aa";
});
PSPromise *p2 = PSPromiseWithBlock(^{
    return @"bb";
});
PSPromise *p3 = PSPromiseWithBlock(^{
    return @"cc";
});

PSPromise *pAll = PSPromise.all(@[p1, p2, p3]).then(^(NSArray *result){
    //then会执行，result = @[@"aa", @"bb", @"cc"];
}).catch(^(NSError *error){
    //catch不会执行
});
```

####race
　　race语法是静态类函数，接受一个Promise数组，并返回一个包装后的Promise对象，称之为R。R的状态改变有两个条件:

1. 当数组中所有的promise对象变成失败状态(Rejected)，这个包装后的R才会把自己变成失败状态。R会等待最慢的那个promise对象变成失败状态后才把自己变成失败态，并将promise数组所有NSError封装成一个NSError传递给后面的catch。
2. 只要其中一个promise对象变成成功状态(Fulfilled)，这个包装后的R就变成成功状态，并且将第一个fulfilled promise的结果值传给R后面的then。

　　因此，race语法可以理解为判断语句的『或』，当所有条件都失败时它才失败，当其中一个条件成功时，它成功。

```objective-c
PSPromise *p1 = PSPromiseWithBlock(^{
    return @"aa";
});
PSPromise *p2 = PSPromiseWithBlock(^{
    @throw error;
});
PSPromise *p3 = PSPromiseWithResolve:(^(PSResolve resolve){
    resolve(@"cc");
});

PSPromise *pRace = PSPromise.race(@[p1, p2, p3]).then(^(NSString *result){
    //then会执行，result = @"aa"
}).catch(^(NSError *error){
    //catch不会执行，因为p1是成功状态，代表race是成功的
});
    
/************************************************************/

PSPromise *p1 = PSPromiseWithBlock(^{
    @throw error;
});
PSPromise *p2 = PSPromiseWithResolve:(^(PSResolve resolve){
    resolve(error);
});
PSPromise *p3 = PSPromiseWithBlock(^{
    return error;
});

PSPromise *pRace = PSPromise.race(@[p1, p2, p3]).then(^(NSString *result){
    //then不会执行，因为race中所有的promise都失败了
}).catch(^(NSError *error){
    //catch会执行，并且NSArray<NSError *> *errors = error.userInfo[PSPromiseInternalErrorsKey]可以获取所有错误
});
```

###扩展语法
---
　　CommonJS Promise/A中的语法较少，考虑到OC实际运用时的便捷性，PSPromise增加了一些实用方法。
####always
　　always语法与then、catch语法类似，与then只处理正确逻辑、catch只处理错误逻辑不同的是，always总是会执行。
####thenAsync
　　thenAsync与then语法使用一致，但是该方法是在`dispatch_get_global_queue(0, 0)`线程中执行，而then方法总是在主线程中执行。
####thenOn
　　thenOn与then语法使用一致，但参数要求用户传递一个指定线程，则block在该线程下执行。
####thenPromise
　　thenPromise是then语法的一个变种，通过回调来返回结果
####catchAsync
　　catchAsync语法与catch语法使用方法一致，但是该方法是在`dispatch_get_global_queue(0, 0)`线程中执行，而catch方法总是在主线程中执行。
####catchOn
　　catchOn与catch语法使用方法一致，但参数要求用户传递一个指定线程，则block在该线程下执行。
###其它
---

1. 关于多线程：PSPromise的方法中，如果没有Async、On这类标识，则block会在主线程中执行，否则会在`dispatch_get_global_queue(0, 0)`或用户指定的线程中执行，添加这些方法主要方便用户在各个线程中切换。

2. 关于泛型：PSPromise其实不支持泛型。但是为什么又引用泛型，目的是在于当Promise作为函数结果返回时，标识一下当前Promise将会返回什么样的数据，当继续then的时候，可以知道then所获取的结果是一个怎么样的类型。