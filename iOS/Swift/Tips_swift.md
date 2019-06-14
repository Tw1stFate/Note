# @autoclosure
实现`||`操作:

* `||`操作符: 左边值为真时返回真, 无需计算表达式右侧的值(当右侧需要较复杂的计算操作的时候)
* 利用`@autoclosure`可以将表达式**自动封装**成一个闭包

```swift
func ||(left: Bool, right: @autoclosure () -> Bool) -> Bool {
    if left {
        return true``
    } else {
        return right()
    }
}
```

# [Capturing Self with Swift 4.2](https://benscheirman.com/2018/09/capturing-self-with-swift-4-2/)

