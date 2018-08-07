# Note&P

## 关于Currying函数调用的疑惑
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

-----------

## 关于`tableView.rx.items`
```swift

let data = Observable.just([
        Music(name: "无条件", singer: "陈奕迅"),
        Music(name: "你曾是少年", singer: "S.H.E"),
        Music(name: "从前的我", singer: "陈洁仪"),
        Music(name: "在木星", singer: "朴树"),
        ])
    /*
         - bind方法: bind(to: (Observable<[Music]>) -> R) 接收一个参数是Observable返回值R的闭包
         - tableView.rx.items函数是一个currying的函数, 调用tableView.rx.items(cellIdentifier: "cellID")会返回一个接收Observable类型, 返回一个 '接收(row,model,cell)作为参数,返回Disposable'的闭包.
         - 就是说上面的bind函数返回值R就是 '接收(row,model,cell)作为参数,返回Disposable'的闭包. 属于单参数的尾随闭包.
        */
        data
            .bind(to: tableView.rx.items(cellIdentifier: "cellID")) { (_, music, cell) in
                cell.textLabel?.text = music.name
                cell.detailTextLabel?.text = music.singer
            }.disposed(by: disposeBag)
```

## 关于`share`

和`RAC`中一样. 冷信号会有副作用. `rxSwift`源码还没细看, 以`RAC`来说, 如果按照下面这样连续订阅两次, `create`中的闭包将会被调用两次, 同时生产两个`subscriber`,  目前没看源码, 猜测`share`应该是会共享`subscriber`(先记下, 待后续验证)
 
```swift
    o = Observable.create({ (observer) -> Disposable in
        observer.onNext("e")
        observer.onCompleted()
        return Disposables.create()
    })
    
    o.subscribe(onNext: { (e) in
        print(e)
    }).disposed(by: DisposeBag())
```


-------------------------

# RxSwift基础 (大都跟RAC差不多)
## Observable
创建Observable

*  just (单独元素)

    ```swift
    o = Observable.just("e")
    o.subscribe(onNext: { (e) in
        print(e)
    }).disposed(by: DisposeBag())
    ```
    
* of   (与just的区别: 注意参数是可变参数, 可以是多个元素)

    ```swift
    o = Observable.of("e", "元素")
    ```
    
*  from (与of的区别: 参数是array或者遵守sequence协议的类型, 用于将一个序列转换成可观察的序列)

    ```swift
    o = Observable.from(["e","e"])
    ```
    
*  empty

    ```swift
    o = Observable.empty()
    ```
        
* never

    ```swift
    o = Observable.never()
     ```
        
*  error

    ```swift
    enum NetworkError: Error {
        case DomainError
    }
    o = Observable.error(NetworkError.DomainError)
    ```
        
*  range

    ```swift
    var o1 : Observable<Int> = Observable.range(start: 0, count: 10)
//    let o2 : Observable<String> = Observable.range(start: "a", count: "z") // 为什么String不行?
    ```
        
*  repeatElement

    ```swift
    o = Observable.repeatElement("e")
     ```
        
*  generate (初始值, 条件, 规则)

    ```swift
    o1 = Observable<Int>.generate(initialState: 0, condition: { $0 < 10 }, iterate: { $0 + 2 })
    // (0 , 2 ,4 ,6 ,8 ,10)
    ```
        
*  create (跟RAC一样)

    ```swift
    o = Observable.create({ (observer) -> Disposable in
        observer.onNext("e")
        observer.onCompleted()
        return Disposables.create()
    })
    ```
        
*  deferred (被订阅才生成可观察序列)

    ```swift
    o = Observable.deferred({ () -> Observable<Any> in
        return Observable.just("e")
    })
    ```
        
*  interval (每隔一段设定的时间，会发出一个索引数的元素。而且它会一直发送下去)
    - ~~通过interval + sample可以实现函数防抖(每隔一段时间执行一次函数) | (区别于throttle, throttle前一次调用后指定时间
    内有新信号就不调用, 接收到新信号后会重置时间)~~  **修正: rx中的throttle跟RAC中的不同, 前者是指定时间内取最后一次信号, 并不会重置时间**
    
    
    ```swift
    o1 = Observable<Int>.interval(1.0, scheduler: MainScheduler.instance)
    o1.subscribe { (event) in
        print(event)
    }.dispose()
    ```
        
*  timer

    ```swift
    // 1. 3s后发出仅且1个元素(1次)
    o1 = Observable.timer(3.0, scheduler: MainScheduler.instance)
    // 2. 在经过3s后，每隔1s产生一个元素
    o1 = Observable.timer(3.0, period: 1.0, scheduler: MainScheduler.instance)
    ```

## Traits (特征序列)
除了 `Observable`，RxSwift 还为我们提供了一些特征序列, 相比`Observable`的通用性, `Traits`序列范围更窄, 用来准确的描述序列.
### Single
特点: 

* 发出一个元素，或一个 error 事件
* 不会共享状态变化

> 常用场景: 比较常见的例子就是执行 `HTTP` 请求，然后返回一个应答或错误。不过我们也可以用 `Single` 来描述任何只有一个元素的序列. 

`SingleEvent`:  为 Single 订阅提供的一个枚举.
`asSingle()`: 用于将`Observable`转换成`Single`.

## Driver
> `Driver` 可以说是最复杂的 trait，它的目标是提供一种简便的方式在 UI 层编写响应式代码. `用序列来驱动应用程序`. 

如果我们的序列满足如下特征，就可以使用`Driver`：

* 不会产生 error 事件
* 一定在主线程监听（`MainScheduler`）
* 共享状态变化（`shareReplayLatestWhileConnected`）

### Completable
特点: 

* 不会发出任何元素.
* 只会发出一个 completed 事件或者一个 error 事件.
* 不会共享状态.

> 场景: 和`Observable<Void>` 有点类似。适用于那些只关心任务是否完成，而不需要在意任务返回值的情况. 例如将数据存入数据库中只关心是否存成功.

`CompletableEvent`: 同样是枚举值.

### Maybe
特点: 

* 介于 Single 和 Completable 之间，它要么只能发出一个元素，要么产生一个 completed 事件，要么产生一个 error 事件.
* 不会共享状态变化

> 场景: 使用于可能需要发出一个元素，又可能不需要发出的情况.

`MaybeEvent`
`asMaybe()`

## Observer
创建Observer有两种方式:
### AnyObserver : 
可以用来描叙任意一种观察者。

```swift
    let observer = AnyObserver<String> { event in
        // handle event
    }
    
    let o = Observable.of("1", "2", "3")
    o.bind(to: observer).dispose()
```

### Binder: 
*   相较于 AnyObserver 的大而全，Binder 更专注于特定的场景。Binder 主要有以下两个特征：

    * 不会处理错误事件

    * 确保绑定都是在给定 Scheduler 上执行（默认 MainScheduler, UI操作都是主线程中进行的, 所以适合用于绑定事件到更新UI控件上）
        
* 一旦产生错误事件，在调试环境下将执行 fatalError，在发布环境下将打印错误信息。

    ```swift
    let label = UILabel()
    let observer1 : Binder<String> = Binder(label) { (label, text) in
        label.text = text
    }
    // Observable序列（每隔1秒钟发出一个索引数）
    let observable = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
    observable
        .map { "当前索引数：\($0 )"}
        .bind(to: observer1)
        .dispose()
    
    // 由于rx已有UILabel的拓展, 上面的绑定可以这么写.
    observable
        .map{ "index: \($0)" }
        .bind(to: label.rx.text)
        .dispose()
    ```

## Subject
RxSwift中有四种subject: 

*  `PublishSubject` :  普通的subject, 可以接收订阅后发出的值
* `BehaviorSubject`:  创建需要提供一个默认值, 每次订阅会立即收到上一次发出的event(首次就是默认值)
* `ReplaySubject`:  创建时候需要设置一个 bufferSize，表示它对于它发送过的 event 的缓存个数, 每次被订阅都会将缓存的信号发送出去. 对BehaviorSubject的拓展.
* `Variable`:  对BehaviorSubject的封装, 不同的是发出信号的行为是修改value的值, 注意在脱离其作用域后会自动发送.completed事件.

## Operation
### 变换操作:

  * `buffer`: 缓冲时间 和 缓冲个数 任意一个条件满足都会将满足条件的信号组装成一个新的 `元素集合` 发送出去.
  * `window`: 与buffer类似, 不同的是window`将元素集合包装成新的信号`(可观察).

  * `map`
  * `flatMap`: 与map主要区别是降维
  * `flatMapLatest`: flatMapLatest 与 flatMap 的唯一区别是：flatMapLatest 只会接收最新的 value 事件
  * `flatMapFirst`: 与flatMapLatest相反, 只会接收最初的值.
  * `concatMap`: concatMap 与 flatMap 的唯一区别是：当前一个 Observable 元素发送完毕后，后一个Observable 才可以开始发出元素。或者说等待前一个 Observable 产生完成事件后，才对后一个 Observable 进行订阅.

  * `scan`: 先给一个初始化的数，然后不断的拿前一个结果和最新的值进行处理操作.(类似swift提供的reduce)
  
  * `groupBy`: 将信号进行分组, 并将每组各自作为新信号.

### 过滤操作: 

* `filter`
* `distinctUntilChanged`
* `single`
* `elementAt`: 只处理在指定位置的事件.
* `ignoreElements`
* `take`
* `takeLast`
* `skip`
* `sample`
* `debounce`: 防抖, 但是处理方式是上次信号开始计时, 指定时间内如果接收到新信号就忽略掉新信号**并重置时间**, 超过指定时间接收到的信号才会被处理.
* `throttle`: 区别于`debounce`, 指定时间内接收到新信号**不会重置时间**.

### 条件和布尔操作符（Conditional and Boolean Operators）:

* `amb`: 当传入多个 Observables 到 `amb` 操作符时，它将取第一个发出元素或产生事件的 Observable，然后只发出它的元素。并忽略掉其他的 Observables.
* `takeWhile`
* `takeUntil`
* `skipWhile`
* `skipUntil`

### 结合操作（Combining Observables）:

* `startWith`
* `merge`
* `zip`
* `combineLatest`
* `withLatestFrom`: 该方法将两个 Observable 序列合并为一个。每当 self 队列发射一个元素时，便从第二个序列中取出最新的一个值.
* `switchLatest`

### 算数、以及聚合操作（Mathematical and Aggregate Operators）:
* `toArray`
* `reduce`: 接受一个初始值，和一个操作符号,  将给定的初始值，与序列里的每个值进行累计运算。得到一个最终结果，并将其作为单个值发送出去. 
* `concat`: 串联合并. 只有当前面一个 Observable 序列发出了 completed 事件，才会开始发送下一个 Observable 序列事件.

### 连接操作（Connectable Observable Operators）
> 将一个正常的序列转换成一个可连接的序列

* `publish`:  该序列不会立刻发送事件，只有在调用 connect 之后才会开始. 
* `replay` :

    * 同上面的 publish 方法相同之处在于：该序列不会立刻发送事件，只有在调用 connect 之后才会开始。
    * replay 与 publish 不同在于：新的订阅者还能接收到订阅之前的事件消息（数量由设置的 bufferSize 决定)

* `multicast`: 可以传入一个 Subject，每当序列发送事件时都会触发这个 Subject 的发送.
* `refCount`: 可以将可被连接的 Observable 转换为普通 Observable. 该操作符可以自动连接和断开可连接的 Observable。当第一个观察者对可连接的 Observable 订阅时，那么底层的 Observable 将被自动连接。当最后一个观察者离开时，那么底层的 Observable 将被自动断开连接。
* `share(relay:)`: 将使得观察者共享源 Observable，并且缓存最新的 n 个元素，将这些元素直接发送给新的观察者. (replay 和 refCount 的组合).

### 其他一些实用的操作符（Observable Utility Operators）
* `delay`: 延迟发出信号.
* `delaySubscription`: 延迟订阅信号.
* `materialize`: 将序列产生的事件，转换成元素. 通常一个有限的 Observable 将产生零个或者多个 onNext 事件，最后产生一个 onCompleted 或者 onError 事件。而 `materialize` 操作符会将 Observable 产生的这些事件全部转换成元素，然后发送出来.
* `dematerialize`: 跟上面`materialize`相反, 用于将 `materialize` 转换后的元素还原.
* `timeout`: 规定时间内没有发任何出元素，就产生一个超时的 error 事件.
* `using`: 创建 Observable 时，同时会创建一个可被清除的资源，一旦 Observable 终止了，那么这个资源就会被清除掉了。





