### 1.OC中不能对`结构体属性`单独赋值.Swift可以.  但是值类型(结构体和枚举)默认方法是不可以修改属性的,如果需要修改属性,需要在方法前加上`mutating`关键字, 让该方法变为一个改变方法
### 2. `Swift` `protocol extensions`. 原文:[Protocol-Oriented Views in Swift](https://www.natashatherobot.com/protocol-oriented-views-in-swift/)
 
 - UIView及其子类才能通过Shakeable protocol获得shake()方法

```swift
import UIKit
 
protocol Shakeable { }
 
// we can constrain the shake method to only UIViews!
extension Shakeable where Self: UIView {
    
    // default shake implementation
    func shake() {
        let animation = CABasicAnimation(keyPath: "position")
        animation.duration = 0.05
        animation.repeatCount = 5
        animation.autoreverses = true
        animation.fromValue = NSValue(CGPoint: CGPointMake(self.center.x - 4.0, self.center.y))
        animation.toValue = NSValue(CGPoint: CGPointMake(self.center.x + 4.0, self.center.y))
        layer.addAnimation(animation, forKey: "position")
    }
}
```
### 3. [Swift3.0 闭包整理](http://www.tuicool.com/articles/JNFVfyn) `逃逸闭包`和`自动闭包`
```objc
//单行表达式闭包可以隐式返回，如下，省略return
let calAdd3:(Int,Int)->(Int) = {(a,b) in a + b}
print(calAdd3(50,200))
```
> 逃逸闭包

> * 当一个闭包作为参数传到一个函数中，需要这个闭包在函数返回之后才被执行，我们就称该闭包从函数种逃逸。一般如果闭包在函数体内涉及到异步操作，但函数却是很快就会执行完毕并返回的，闭包必须要逃逸掉，以便异步操作的回调。

> * 逃逸闭包一般用于异步函数的回调，比如网络请求成功的回调和失败的回调。语法：在函数的闭包行参前加关键字“@escaping”。

```objc
//例1
func doSomething(some: @escaping () -> Void){
    //延时操作，注意这里的单位是秒
    DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 1) {
        //1秒后操作
        some()
    }
    print("函数体")
}
doSomething {
    print("逃逸闭包")
}

//例2
var comletionHandle: ()->String = {"约吗?"}

func doSomething2(some: @escaping ()->String){
    comletionHandle = some
}
doSomething2 {
    return "叔叔，我们不约"
}
print(comletionHandle())

//将一个闭包标记为@escaping意味着你必须在闭包中显式的引用self。
//其实@escaping和self都是在提醒你，这是一个逃逸闭包，
//别误操作导致了循环引用！而非逃逸包可以隐式引用self。

//例子如下
var completionHandlers: [() -> Void] = []
//逃逸
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
//非逃逸
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
```
> 自动闭包

> * 顾名思义，自动闭包是一种自动创建的闭包，封装一堆表达式在自动闭包中，然后将自动闭包作为参数传给函数。而自动闭包是不接受任何参数的，但可以返回自动闭包中表达式产生的值。

> * 自动闭包让你能够延迟求值，直到调用这个闭包，闭包代码块才会被执行。说白了，就是语法简洁了，有点懒加载的意思。

```objc
var array = ["I","have","a","apple"]
print(array.count)
//打印出"4"

let removeBlock = {array.remove(at: 3)}//测试了下，这里代码超过一行，返回值失效。
print(array.count)
//打印出"4"

print("执行代码块移除\(removeBlock())")
//打印出"执行代码块移除apple" 这里自动闭包返回了apple值

print(array.count)
//打印出"3"
```
 
### 4. [Swift协议和扩展](http://www.tuicool.com/articles/773a6f)

 
###5. [Swift学习: 从Objective-C到Swift](http://www.tuicool.com/articles/Vz6Vzyb)

###6. Value and Reference Types
 **Use a value type when:**

> * Comparing instance data with == makes sense
* You want copies to have independent state
* The data will be used in code across multiple threads

**Use a reference type (e.g. use a class) when:**
> * Comparing instance identity with === makes sense
* You want to create shared, mutable state


###7. [iOS夯实：内存管理](https://github.com/100mango/zen/blob/master/iOS夯实：内存管理/iOS夯实：内存管理.md)

>  **不要在初始化方法和dealloc方法中使用Accessor Methods(OC中 init 和 dealloc 不能使用属性 self.property = XXX 来进行设置, 应使用_property = xxx的方式)**

 
###8. 使用 stride() 函数来进行更多可配置的迭代
```objc
let colors = ["blue", "green", "red", "white", "black"]  
for index in 0..<colors.count {  
  if (index % 2 == 0) {
    print(colors[index])
  }
}
// => "blue" "red" "black"
```

这种情况很符合使用 Swift 的stride(from: value, to: value, by: value)函数。定义开始，结束（不包括上限）和步长值，函数返回相应的数字序列。

让我们在我们的场景中应用stride：

```objc
let colors = ["blue", "green", "red", "white", "black"]  
for index in stride(from: 0, to: colors.count, by: 2) {  
  print(colors[index])
}
// => "blue" "red" "black"

```

stride(from: 0, to: colors.count, by: 2) 返回以0开始到5的数字（上限不包含5），步长为2。对于 for-loop，这是一个好的替代。

如果上限必须包含进来，这里有另外一种函数格式：

stride(from: value, through: value, by: value) 。第二个参数的标签是 through ， 这个标签是用以指明是包含上限的。

-----
###9. [39个优秀的Swift UI开源库](http://www.tuicool.com/articles/EZVV3m7)


