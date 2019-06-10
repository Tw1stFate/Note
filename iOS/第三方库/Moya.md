### 闭包的艺术
#### 1.利用闭包实现异步控制

先看初始化`MoyaProvider`的方法

```swift
/// Initializes a provider.
    public init(endpointClosure: @escaping EndpointClosure = MoyaProvider.defaultEndpointMapping,
                requestClosure: @escaping RequestClosure = MoyaProvider.defaultRequestMapping,
                stubClosure: @escaping StubClosure = MoyaProvider.neverStub,
                callbackQueue: DispatchQueue? = nil,
                manager: Manager = MoyaProvider<Target>.defaultAlamofireManager(),
                plugins: [PluginType] = [],
                trackInflights: Bool = false) {
```

这里以`MoyaProvider.defaultRequestMapping`为例(其他的也是同样的做法), `moya`提供了默认的实现:
![](media/15132366513701/15132369078594.jpg)
. 该闭包的作用是将`endpoint`, 转换成`urlRequest`, 然后执行`closure`中的内容. 

下面从`MoyaProvider+Internal.swift`文件中看`provider`是怎么发网络请求的, 还有上面的`closure`到底是什么.


```swift
func requestNormal(_ target: Target, callbackQueue: DispatchQueue?, progress: Moya.ProgressBlock?, completion: @escaping Moya.Completion) -> Cancellable {
        let endpoint = self.endpoint(target)
        let stubBehavior = self.stubClosure(target)
        let cancellableToken = CancellableWrapper()
    
        ...
        let performNetworking = { (requestResult: Result<URLRequest, MoyaError>) in
            if cancellableToken.isCancelled {
                self.cancelCompletion(pluginsWithCompletion, target: target)
                return
            }

            var request: URLRequest!

            switch requestResult {
            case .success(let urlRequest):
                request = urlRequest
            case .failure(let error):
                pluginsWithCompletion(.failure(error))
                return
            }

            // Allow plugins to modify request
            let preparedRequest = self.plugins.reduce(request) { $1.prepare($0, target: target) }

            let networkCompletion: Moya.Completion = { result in
              if self.trackInflights {
                self.inflightRequests[endpoint]?.forEach { $0(result) }

                objc_sync_enter(self)
                self.inflightRequests.removeValue(forKey: endpoint)
                objc_sync_exit(self)
              } else {
                pluginsWithCompletion(result)
              }
            }

            cancellableToken.innerCancellable = self.performRequest(target, request: preparedRequest, callbackQueue: callbackQueue, progress: progress, completion: networkCompletion, endpoint: endpoint, stubBehavior: stubBehavior)
        }

        requestClosure(endpoint, performNetworking)

        return cancellableToken
    }
```

主要看`performNetworking`这个闭包, 其实上面的`closure`就是这个闭包.  `requestClosure`是在调用闭包`MoyaProvider.defaultRequestMapping`

按照我的逻辑, `moya`在这个函数中做的事应该有一下几点:

 * 通过`target`生成`endpoint`.
 * 通过`endpoint`生成`request`
 * 发送网络请求, 并将结果回调出去.

 那么为什么`moya`要在这里绕一圈?  应该是为了方便开发者根据自己的需要, 在初始化的方法中传入自定义的`MoyaProvider.defaultRequestMapping`, 自己通过`endpoint`生成自定义的`request`.  
 
 so 实际效果就是: 发请求的时候先通过`target`生成`endpoint`, 然后**想要发送网络请求, 需要等开发者自定义完`request`之后才能进行. 于是将网络请求这一部分内容先用闭包保存, 然后传给外界, 外界自定义完`request`后, 再调用传递过来的闭包完成发请求的操作**
 
##### 总结:
如果在`A`任务中, 有子任务`B`需要`等待外界某个操作执行完` , `并依赖该外界操作的结果`. 这个时候就可以把任务`B`封装成闭包, 然后传给外界, 在外界操作执行完成后执行闭包 就可以达到目的.

> * **`RAC`的`creatSignal`和`subscribe`的过程也是同样的做法.**
> * 其实在封装网络请求的时候用的也是这种做法的简化版, 一般都是将成功/失败的回调传到调用网络请求的方法中, 然后请求成功/失败后执行对应的回调.  

> 在写组件提供给别人用, 如果想给外界提供一些自定义行为的时候也可以用参照这种做法. 

#### 2.利用闭包减少代码分散度
**不要写重复的代码**
接上面的总结, 在封装网络请求的时候, 都是将成功/失败的回调传到调用网络请求的方法中, 然后请求成功/失败后执行对应的回调. 由于有多处需要调用回调结果的闭包, `moya`的做法是在其基础上再嵌套一层. 并封装一些由外界提供的自定义操作. 嵌套一层的目的应该就是防止写重复的代码.

注意`pluginsWithCompletion`:

```swift
/// Performs normal requests.
    func requestNormal(_ target: Target, callbackQueue: DispatchQueue?, progress: Moya.ProgressBlock?, completion: @escaping Moya.Completion) -> Cancellable {
        ...
        // Allow plugins to modify response
        let pluginsWithCompletion: Moya.Completion = { result in
            let processedResult = self.plugins.reduce(result) { $1.process($0, target: target) }
            completion(processedResult)
        }

        if trackInflights {
            objc_sync_enter(self)
            var inflightCompletionBlocks = self.inflightRequests[endpoint]
            inflightCompletionBlocks?.append(pluginsWithCompletion)
            self.inflightRequests[endpoint] = inflightCompletionBlocks
            objc_sync_exit(self)
            ...
            
        let performNetworking = { (requestResult: Result<URLRequest, MoyaError>) in
            if cancellableToken.isCancelled {
                self.cancelCompletion(pluginsWithCompletion, target: target)
                return
            }

            var request: URLRequest!

            switch requestResult {
            case .success(let urlRequest):
                request = urlRequest
            case .failure(let error):
                pluginsWithCompletion(.failure(error))
                return
            }

            // Allow plugins to modify request
            let preparedRequest = self.plugins.reduce(request) { $1.prepare($0, target: target) }

            let networkCompletion: Moya.Completion = { result in
              if self.trackInflights {
                self.inflightRequests[endpoint]?.forEach { $0(result) }

                objc_sync_enter(self)
                self.inflightRequests.removeValue(forKey: endpoint)
                objc_sync_exit(self)
              } else {
                pluginsWithCompletion(result)
              }
            }

            cancellableToken.innerCancellable = self.performRequest(target, request: preparedRequest, callbackQueue: callbackQueue, progress: progress, completion: networkCompletion, endpoint: endpoint, stubBehavior: stubBehavior)
        }

        requestClosure(endpoint, performNetworking)

        return cancellableToken
    }
```


