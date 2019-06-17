[toc]

# 数据结构及算法

# 程序设计
## OC中为什么要有`Meta Class`
* OC的面向对象是基于类来设计的, 每个对象都是类的实例. 对象有个`isa`指针指向类对象, 类包含描述该对象的相关数据和行为信息. 对象的属性列表和方法列表存储在类中, 用来对该对象收到的消息进行相应. 当给对象发消息时, `objc_msgSend()`检查该对象的类的方法列表中是否有该方法, 并作出相应. 
* 每个类自身也是一个对象, 也有一个`isa`指针和其他相关数据和可供相应的方法. 当你调用"class method", 你实际上是发送一条消息给类对象. 
* 由于类也是一个对象, 所以类对象也是其他类的实例. 因此需要`meta class`来描述类对象, 以及相应类对象收到的消息. 所以类对象的方法列表存储在元类中.

**参考:**
[Classes and metaclasses](http://www.sealiesoftware.com/blog/archive/2009/04/14/objc_explain_Classes_and_metaclasses.html)
[What is a meta-class in Objective-C?](http://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html)
[简书What is a meta-class in Objective-C?](https://www.jianshu.com/p/ea7c42e16da8)

# 架构模式
<!--TODO: 架构图-->

# 设计模式

# 数据安全

# iOS 内存管理 

## Block的内存管理

# Runtime

## weak实现原理
## AssociatedObject原理
## Runtime应用
<details>

* 获取类的属性. (YYModel字典转模型, 数据库ORM).
* 交换方法. (Aspect hook方法, JSPatch, 数据埋点).
* 设置关联对象. (建个分类给UILabel拓展个是否可复制属性).
</details>

# Runloop

# UIKit

# Foundation

# 网络

# 多线程


# WebView

# 动画

# 图像/音视频处理

# 第三方库源码阅读

# iOS逆向



# 性能优化


# 单测 

> 单元测试的价值体现在修改代码的时候. 自动化测试唯一的理由是它让我们能在将来修改我们的代码. 通过`依赖注入`的方式来编写单元测试. 通过提供的`XCTest`框架完成.  
> 快捷键: `cmd + 5 `切换到测试选项卡后会看到很多小箭头，点击可以单独或整体测试. `cmd + U` 运行整个单元测试
 
## 测试方法
 
 <details>

```
- (void)setUp {
    [super setUp];
     //每次测试前调用，可以在测试之前创建在test case方法中需要用到的一些对象等.
    // Put setup code here. This method is called before the invocation of each test method in the class.
}

- (void)tearDown {
-   //每次测试结束时调用tearDown方法.
    // Put teardown code here. This method is called after the invocation of each test method in the class.
    [super tearDown];
}

- (void)testExample {
-   // 该类中以test开头的方法且void返回类型的方法都会变成单元测试用例.
    // This is an example of a functional test case.
    // Use XCTAssert and related functions to verify your tests produce the correct results.
}

- (void)testPerformanceExample {
    // This is an example of a performance test case.
    [self measureBlock:^{
        // 性能测试主要使用 measureBlock 方法 ，用于测试一组方法的执行时间，通过设置baseline（基准）和stddev（标准偏差）来判断方法是否能通过性能测试。
        // Put the code you want to measure the time of here.
    }];
}

//性能测试方法，通过测试block中方法执行的时间，比对设定的标准值和偏差觉得是否可以通过测试.
measureBlock
```
 </details>

## 断言
> 大部分的测试方法使用断言决定的测试结果。所有断言都有一个类似的形式：比较，表达式为真假，强行失败等。

<details>

```
//通用断言
XCTAssert(expression, format...)
//常用断言：
XCTAssertTrue(expression, format...)
XCTAssertFalse(expression, format...)
XCTAssertEqual(expression1, expression2, format...)
XCTAssertNotEqual(expression1, expression2, format...)
XCTAssertEqualWithAccuracy(expression1, expression2, accuracy, format...)
XCTAssertNotEqualWithAccuracy(expression1, expression2, accuracy, format...)
XCTAssertNil(expression, format...)
XCTAssertNotNil(expression, format...)

XCTFail(format...) //直接Fail的断言
```
</details>

## 期望
> 期望实际上是异步测试，当测试异步方法时，因为结果并不是立刻获得，所以我们可以设置一个期望，期望是有时间限定的的，fulfill表示满足期望.

<details>


```
- (void)testAsynExample {
    XCTestExpectation *exp = [self expectationWithDescription:@"这里可以是操作出错的原因描述。。。"];
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    [queue addOperationWithBlock:^{
        //模拟这个异步操作需要2秒后才能获取结果，比如一个异步网络请求
        sleep(2);
        //模拟获取的异步操作后，获取结果，判断异步方法的结果是否正确
        XCTAssertEqual(@"a", @"a");
        //如果断言没问题，就调用fulfill宣布测试满足
        [exp fulfill];
    }];
    
    //设置延迟多少秒后，如果没有满足测试条件就报错
    [self waitForExpectationsWithTimeout:3 handler:^(NSError * _Nullable error) {
        if (error) {
            NSLog(@"Timeout Error: %@", error);
        }
    }];
}
```
</details>

## 命令行测试
> 测试不仅可以在xcode中执行，也可以在命令行中执行，这个便于代码持续集成和构建，在git提交中也编译检查代码.(例如jenkins等ci工具).

# UI测试

# CVS & CI

# 功能
## 网络层
## 缓存框架
## 组件化 & 路由
> [iOS应用架构谈 组件化方案](https://casatwy.com/iOS-Modulization.html)
> [iOS 组件化 —— 路由设计思路分析](https://www.jianshu.com/p/76da56b3bd55)

## 富文本
## 换肤
* [关于iOS App换肤的几种方式](http://www.cocoachina.com/ios/20171012/20762.html)
    - demo: [yanjunz/ThemeManager](https://github.com/yanjunz/ThemeManager) 
    - 缺点: 无法适配第三方 UI 控件
* [成熟的夜间模式解决方案](https://draveness.me/night)
    - [Draveness/DKNightVersion](https://github.com/Draveness/DKNightVersion)
* [jiecao-fm/SwiftTheme](https://github.com/jiecao-fm/SwiftTheme)

1. 简单阅读了下前两个的源码(两个都是oc所以看了下...), 两者大致的实现思路是一致的, 只是后者更完善精致一些. 
2. 简单概括下第一种的思路:
        利用`runtime`给`UIView`添加关联属性用于存储制定好的不同皮肤策略, 在具体设置各个控件的策略时, 会在`setter`方法中注册观察者, 监听换肤切换动作. 如果发生切换动作, 调用对应的方法修改写在各自控件中的修改外观的方法.

## 埋点

# 零碎知识点

## oc 和 swift对比
* 动态/静态 
    - `oc`是一门动态语言, 很多东西都是在运行时决定. 好处:**灵活**. 缺点: 不安全. 在 OC 中动态派发让我们承担了在运行时才发现错误的风险.
    - `swift`静态语言, 严格约束类型. 好处: 安全. 缺点: 写的很蛋疼...
* 语言特性:
    - `swift`中有很多特性: 泛型(oc后面也支持, 但是没swift强大), 协议.
* `String`,`Array`,`Dictionary`等类在`swift`中为值类型. 更加高效.
* `swift`中的`struct`和`enum`比oc中变态得多.

* 对swift中extension的理解:
    - 首先extension在swift中类似oc的类目，可以扩展方法，计算属性，不能添加存储属性；
    - 可以通过extension让类实现某个协议，一般这个用的也比较多，比如实现Comparable这个协议；
    - 还有一个很重要的，就是可以通过extension对协议进行扩展，添加默认实现，属于黑魔法吧，非常好用。

## 谓词(正则)
## NSUrlProtocol




## 面向对象的缺陷
**事物往往是一系列特质的组合，而不单单是以一脉相承并逐渐扩展的方式构建的.**
[面向对象编程的弊端是什么？](https://www.zhihu.com/question/20275578)

[面向协议编程与 Cocoa 的邂逅 (上)](https://onevcat.com/2016/11/pop-cocoa-1/)
[面向协议编程与 Cocoa 的邂逅 (下)](https://onevcat.com/2016/11/pop-cocoa-2/)
面向协议可以解决以下问题:
✅ 动态派发安全性
✅ 横切关注点
❓菱形缺陷 (菱形缺陷没有被完全解决，Swift 还不能很好地处理多个协议的冲突，这是 Swift 现在的不足)


    










