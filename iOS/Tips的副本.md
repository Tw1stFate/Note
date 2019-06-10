# tips


### 查看真机UDID


```objc
    instruments -s | awk '{print $NF}' | sed -n 3p | awk '{print substr($0,2,length($0)-2)}' | xargs rvictl -s
```
### 关于weakSelf 和 strongSelf
* `weakSelf`: 使用不用细说, 一般是用于防止循环引用.
* `strongSelf`: 主要作用是考虑在执行block内容的过程中self释放的情况, 使用`strongSelf`会暂时强持有self, 等执行完block中的内容之后再让其释放.

场景:

```objc
    __weak typeof(self) weakSelf = self;
    [self doABlockOperation:^{
        __strong typeof(weakSelf) strongSelf = weakSelf;
        if (strongSelf) {
            [strongSelf doSomething];
            ...
        }
    }];
    
    - (void)doSomething {
        [self xxx];
    }
```

以前使用`weakSelf`的时候总是纠结上面的代码如果没用`strongSelf`的话(将`strongSelf`那行注释)直接在`doSomething`方法中调用其他方法要不要用`weakSelf`.   现在想来, 如果`block`中还没执行self就释放了, 那`doSomething`就不会被调用, 如果在执行`block`过程中`self`释放了, 为了保证`doSomething`执行完, 或者防止`doSomething`过程中出现莫名其妙的情况, 最好还是加上` __strong typeof(weakSelf) strongSelf = weakSelf;`.


参考文章: [I finally figured out weakSelf and strongSelf](https://dhoerl.wordpress.com/2013/04/23/i-finally-figured-out-weakself-and-strongself/)


### 不要滥用单例
* [避免滥用单例](https://objccn.io/issue-13-2/)
![](media/15142576158662/15142776381655.jpg)

*通常不同的模块之间是不应该存在耦合的, 如果使用单例: `A模块`访问单例中存储的状态, 在`B模块`修改了该状态, 无形之中使得`A`,`B`之间产生了耦合.*
![](media/15142576158662/15142783250935.jpg)


在objcio里面还讲了一种情况就是一个有用户登录然后还有好友列表的应用，登录之后朋友列表SPFriendListViewController里可以看到朋友的头像，然后作者实现了一个单例SPThumbnailCache来下载和缓存朋友们的头像。由于单例“create once, live forever”的特性，直到登出这个需求的出现之前，在这里都非常适用。登出功能一旦出现，意味着要清理掉所有缓存和更换一个SPThumbnailCache（其实我很好奇为什么作者不干脆写一个清理缓存的接口得了）。而由于dispatch_once的特点，单例没法反复创建，按照文中的意思，可以不使用dispatch_once来创建单例，接着再实现一个teardown方法来将单例置nil，完了再新建单例来满足需求。 
### 依赖倒置原则
* `高层模块不应该依赖低层模块，两者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。`

 [小话设计模式原则之：依赖倒置原则DIP](https://zhuanlan.zhihu.com/p/24175489)

`工厂方法模式`遵循了`依赖倒置原则`.  在上面的链接的例子中, 工厂模式是这么写的:
![](media/15142576158662/15142999208697.jpg)

![](media/15142576158662/15142999365804.jpg)

**思考**: 是否应该将![](media/15142576158662/15143000693375.jpg)
这一步像`移动OA`中用的网络层一样封装在`factory`中? 
理由: 考虑到`依赖注入`, [如何用最简单的方式解释依赖注入？依赖注入是如何实现解耦的?
](https://www.zhihu.com/question/32108444) . 如果初始化device的地方特别多的话, 使用依赖注入的方式能规避可能发生的改动.(但是这个初始化方法好像也不会依赖外部变量, 所以是不是有必要呢?)


## * `swift`支持函数重载, `oc`不支持, `oc`的替代方式是`withXXX`

## 关于数组的`count`

```objc
@property (readonly) NSUInteger count;
```
数组的count是无符号的, 所以千万不要写这种代码: `0 > arr.count - 1`, 返回的结果永远是`NO`.

## 关于pch文件
* pch文件中引入其他头文件时, **需要注意顺序**. (例如当A.h中内容依赖B.h中定义的宏时).
* 需要添加`#ifdef __OBJC__`, `#endif`, 以区分非oc文件. 否则可能引发错误.

Default.h

```objc
#import <Foundation/Foundation.h>
@interface Default : NSObject
- (void)lynn_aaa;
@end
```

codeObfuscation.h

```objc
#ifndef Demo_codeObfuscation_h
#define Demo_codeObfuscation_h
//confuse string at Sat Jun 16 00:24:40 CST 2018
#define lynn_aaa FyxWAJEmWMCvZcAx
#endif
```

PrefixHeader.pch

```objc
#ifndef PrefixHeader_pch
#define PrefixHeader_pch
#import "Default.h"
//添加混淆作用的头文件（这个文件名是脚本confuse.sh中定义的）
#import "codeObfuscation.h"  
#endif /* PrefixHeader_pch */
```

**如果按照上面的pch文件写法, 应该是先将`Default.h`中内容加入pch, 然后才是`PrefixHeader.pch`中内容, 所以`PrefixHeader.pch`中的宏对`Default.h`中的方法名就不会有作用了. 会导致方法名并没有被宏替换掉.**


## 关于__strong
* 使用`block`的时候需要注意一件事, 就是这段`block`中的代码很有可能是在其他地方的某个时机才会执行. 
* 以网络请求为例, successBlock中的回调执行的时候可能控制器已经释放了(假设网络请求是写在vc中的), 一般涉及到网络请求都会用`__strong`修饰self, 目的是延迟vc的释放时机至`block`内容执行完. 

这里要提的一点是**要根据具体情况使用, 用了有时反而可能出现问题. 凡是还是要搞懂why**.  例如如下场景: 
> 背景: app内的跳转栈是由自己维护的.

    点击按钮从A界面跳转到B界面, 同时执行一次网络请求, 请求回调中会dismiss B界面. 如果用户在执行网络请求的过程中返回至A界面. 此时B控制器已经dismiss一次了, 如果用__strong修饰的话会多执行了一次dismiss. 

## 大文件处理

## 缓存设计

## 网络层设计
[iOS应用架构谈 网络层设计方案](https://casatwy.com/iosying-yong-jia-gou-tan-wang-luo-ceng-she-ji-fang-an.html)

## WKWebView
[WKWebViewTips](https://github.com/ShingoFukuyama/WKWebViewTips)


