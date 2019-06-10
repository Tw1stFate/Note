# RAC

[http://www.jianshu.com/p/e10e5ca413b7
](袁峥RAC进阶篇)

##### **映射:**

* `FlatternMap` /  `Map`
    * 1.FlatternMap中的Block返回信号。
    * 2.Map中的Block返回对象。

**组合:**

* `Concat`:按一定顺序拼接信号，当多个信号发出的时候，有顺序的接收信号 .   (注意: 前面的信号sendCompleted之后, 后面的信号才会被订阅)
* `then`:用于连接两个信号，当第一个信号完成，才会连接then返回的信号。只接收`then`返回的信号 `Concat`是都能接收
* `merge`:把多个信号合并为一个信号，任何一个信号有新值的时候就会调用.  区别concat
* `zipWith`:把两个信号压缩成一个信号，只有当两个信号同时发出信号内容时，并且把两个信号的内容合并成一个元组，才会触发压缩流的next事件.
* `combineLatest`:将多个信号合并起来，并且拿到各个信号的最新的值,必须每个合并的signal至少都有过一次sendNext，才会触发合并的信号。
* `reduce`聚合:用于信号发出的内容是元组，把信号发出元组的值聚合成一个值

**过滤:**

*  `filter` : 过滤信号，使用它可以获取满足条件的信号.
* `ignore` : 忽略完某些值的信号.
* `distinctUntilChanged` : 当上一次的值和当前的值有明显的变化就会发出信号，否则会被忽略掉
* `take` : 从开始一共取N次的信号
* `takeLast` : 取最后N次的信号,前提条件，订阅者必须调用完成，因为只有完成，就知道总共有多少信号.
* `takeUntil` : (RACSignal *):获取信号直到某个信号执行完成
* `skip`:(NSUInteger) : 跳过几个信号,不接受。
* `switchToLatest` : 用于signalOfSignals（信号的信号），有时候信号也会发出信号，会在signalOfSignals中，获取signalOfSignals发送的最新信号。

**秩序:**

* `doNext`: 执行Next之前，会先执行这个Block
* `doCompleted`: 执行sendCompleted之前，会先执行这个Block

**线程:**

* `deliverOn`: 内容传递切换到制定线程中，副作用在原来线程中,把在创建信号时block中的代码称之为副作用。
* `subscribeOn`: 内容传递和副作用都会切换到制定线程中

**时间:**

* `timeout`：超时，可以让一个信号在一定的时间后，自动报错
* `interval` 定时：每隔一段时间发出信号
* `delay` 延迟发送next。

**重复:**

* `retry`重试 ：只要失败，就会重新执行创建信号中的block,直到成功.
* `replay`重放：当一个信号被多次订阅,反复播放内容
* `throttle`节流:当某个信号发送比较频繁时，可以使用节流，在某一段时间不发送信号内容，过了一段时间获取信号的最新内容发出。



# throttle 


今天在看`RAC`源码, 看到`throttle`, 想知道为什么`throttle`能在每次接收到信号后重置`interval`时间的, 本来是猜测接收到新的信号直接重置`interval`. 看到下面发现并不是超过`interval`后再去调用`block`中的内容, 而是**每次接收到信号都会先将之前的`nextDisposable.disposable`释放, 然后调用`after:schedule:`延迟执行**, 在延迟的时间内如果收到新的信号了, 就重复前面的过程, 造成的结果就是将前一次`nextDisposable.disposable`释放, 那么在`schedule`的`block`中的就会判断出`disposable.dispose`为`YES`, 直接`return`.  如果延迟时间过了还没收到下一次的值就就不会去调用`flushNext`这个`block`去释放`nextDisposable.disposable`.
![](media/15126273661876/15126464548167.jpg)

![](media/15126273661876/15126464212975.jpg)

![](media/15126273661876/15126273821943.jpg)

### 总结: 
* 自己的猜测的想法: 开启一个定时器(用于倒计时`interval`), 每次接收到新的信号重置定时器的时间. 倒计时完后才转发信号. 
* `RAC`做法: 每次接收到信号都延迟`interval`后转发信号, **利用延迟执行的时间达到转换倒计时的效果**. 就好像设置一个变量, 每次发生事件就去改变这个变量, 每次事件启动后延迟2s去查看改变量是否与自己刚启动的时候一致, `true` -> 执行; `false` -> 不执行.

* 如果用`throttle`这种处理方式做按钮的防重复点击,  就是每次点击按钮都将点击事件延迟1s(假设)执行, 每次点击如果距离前一次点击不超过1s就将之前的点击废弃. 问题在于**第一次点击不会触发**, 如果用`startWith`来想办法给个初始值应该可以.

附:

```
- (void)throttle
{
    NSArray *arr = [@"0 1 1 1 2 3 3 4" componentsSeparatedByString:@" "];

    __block NSInteger delayTime = 0;
    RACSignal *s = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [arr.rac_sequence.signal subscribeNext:^(id x) {
           
            delayTime += arc4random()%2 + 1;
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayTime * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
               
                NSLog(@"=================%@",x);
                [subscriber sendNext:x];
            });
        } completed:^{
//            [subscriber sendCompleted];
        }];
        return [RACDisposable disposableWithBlock:^{}];
    }];

    // 1.throttle用于保证接收到的 所有相邻的信号之前的时长大于interval这个时间 才会被接收, 并且接收的是最新的信号.
    // 必须注意的是这个interval*会在每次接收到信号后重置*, 所以throttle应该是用在防止在单位时间内重复接收信号.
    // 如果interval时间内不接收任何信号内容, 过了这个时间, 获取最后发送的信号内容发出.
    // 感觉不适合用于防止按钮重复点击. 因为从1可以第一次信号并不会被订阅, 就是说点了一次,interval时间后再点一次, 只有后面那次才起作用.
    [[s throttle:2] subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
}
```



