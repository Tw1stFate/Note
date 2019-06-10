#### 2018/02/26
> 场景; 触发Action调整tableView的内边距, 但是发现没效果. 
> 解决方案: 利用animateWithDuration的completion来达到等tableView刷新完后再修改内边距
            [UIView animateWithDuration:0 animations:^{
                [self.todoTableView reloadData];
                [self.todoTableView layoutIfNeeded];
            } completion:^(BOOL finished) {
                // 调整内边距(需要等tableView刷新完)
                [self resetTableViewContentInset];
            }];



### GUI编程中比较复杂的几个问题
* 非原生`UI`效果的实现. (各种动画效果的实现).
* 大量状态的维护. (通过各种不同的状态驱动不同的`UI`).
* 异步数据的处理. (多线程的处理).

### 2017/12/26
> 现象: 集成`FLEX`后发现`FLEX`提供的调试界面所有界面(都是`UITableView`)都是内容整体想上偏移64.
> 但是之前有新建一个测试的demo, 集成`FLEX`显示确是正常的.
猜测是滥用runtime导致的坑, 经检查后发现果然是给`UITableView`建了个分类, 交换了`initWithFrame:style:`这个方法, 自己在替换的方法中写了`table.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;`. 导致所有的`tableview`在有导航栏的情况下都不会自动向下调整`64`.


### (2018/02/24)popView在特定情况下会一直停留在屏幕上的bug
> 现象: 收件箱查看某条邮件 -> 返回 -> 点击右上角新建邮件 -> 返回 -> 点击titleView -> 弹出的popView会一直停留在屏幕上.

**分析原因:** 

    1. 查看源码发现是在`[UIApplication sharedApplication].keyWindow`上添加遮罩和`popView`.
    2. 调试图层发现`popView`被添加到了`_UIInteractiveHighlightEffectWindow`上面.
    
**查找资料:**

    ios11后多出了一个私有的`_UIInteractiveHighlightEffectWindow`.
    
**调试分析:**

    1. 正常情况下,[UIApplication sharedApplication].keyWindow应该获取的是加载app的UIWindow, 但是收到邮箱'ZSSRichTextEditor'这个类的影响(操作方式:先点一封邮件进入详情, 然后返回, 再点右上角新建邮件, 再返回就会发现keyWindow发生了改变), .keyW.indow获取到的是_UIInteractiveHighlightEffectWindow.
    2. 由于 `ZSSRichTextEditor`比较复杂不能很快找到具体为什么`keyW.indow`会变成`_UIInteractiveHighlightEffectWindow`, 所以目前直接将`popView`加到显示的`UIWindow`上而不是`keyWindow`.


### 关于Currying函数调用的疑惑
> 在阅读RxCocoa中`tableView.rx.items`方法时, 由于一开始对该方法不是很理解, 所以写了下面的测试代码, 引发下面的问题. 

```swift
func test(a:String) -> (_ b: String) -> (_ c: () -> String) -> String {
    return { b in
        return { c in
            c()
            return "a" + "b"
        }
    }
}
//1
let funcB = test(a: "a")
let testC = funcB("b")
let res = testC {
    print("closure")
    return "closure"
}
//2
let testFunc = test(a: "a")("b")
let res2 = testFunc {
    print("closure")
    return "closure"
}
//3
let res1 = test(a: "a")("b")() {
    print("closure")
    return "closure"
}
```

>  关于闭包: 如果闭包作为函数的唯一参数, 写尾随闭包时可以直接忽略掉()

**问题: 为什么第三种写法将最末尾的()去掉就会报错??? (对比2中的写法)**


### 关于`dispatch_semaphore`
> 场景: 在子线程做某个操作时, 需要获取应用程序的状态(是否在前台). 
想通过`dispatch_semaphore`同步处理, 结果发现`mainQueue`中的block永远不会执行(会导致`wait`那里也过不去).  why???

```swift
 __block UIApplicationState applicationState;
GCDSemaphore *sem = [[GCDSemaphore alloc] init];
[GCDQueue executeInMainQueue:^{
  applicationState = [UIApplication sharedApplication].applicationState;
  [sem signal];
}];
[sem wait];
```

###关于`blocks`和`delegate`的选择
*(一些普通的用法与区别这里就不说了, 这里主要记录一些之前没在意的东西):*

> * 有些时候使用了 `Delegate` 之后，代码阅读变成了一件很困难的事情，你需要在不同的 `Class` 文件中一次次的跳转来理解整个代码思路，要知道糟糕的 XCode 还经常会跳错方法（`Command + 点击`跳转，跳到了同名的其他方法），导致代码可读性的下降. 

> * `block`可以访问上下文，使代码更紧凑. 但是很容易造成循环引用. 比如按钮的点击事件，假如采用 block 实现，这种 block 就需要长期存储，并且会调用多次。调用之后，block 也不可以删除，可能还有下一次按钮的点击。这或许就是按钮时间的处理上`Apple`采用`target-action`的原因吧.

> * **关于两者的选择:** 单一 `callback` 用 `block`、多重 `callback` 用 `delegate`.  `Cocoa` 中的 API 设计也是这样的，临时性的，只会调用一次的，采用 `block`。而多次调用的，并不会使用 `block`。(主要看NSUrlSession的回调)

### 没有正确初始化model的小bug:
 *场景: 点击一行cell跳转到选人页面, 带回来处理人数据后刷新界面并且人员的cell需要初始化一些数据(model中的一些属性), 当时就将cell中数据初始化的方法写在cellForRow中创建cell(人员cell)的地方, 但是如果用户没有滚动至显示这个cell的话, 就不会去执行cellForRow中的代码, 就不会有数据. 尴尬...*
 
 > 没有遵守`view`中的代码规范, `cellForRow`中将`model`给`view`去显示, 初始化`model`的数据不应该写在这里, 可以在数据回调的地方进行初始化.

### 通知的一个重要场景:
*通常通知/delegate/block都可以用来callback.但是有些场景是其他方式替代不了的.*

> 场景: tableview有很多个同类cell(子控件有scrollview), 其中一个cell中scrollview滚动时其他cell跟着对齐. 这个时候只有通知能解决(自己在写ExcelCell控件的时候用到).

