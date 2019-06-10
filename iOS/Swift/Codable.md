# Codable

```swift
    public typealias Codable = Decodable & Encodable
```
先看Decodable 对 JSON 数据对象的解析。Swift 为我们做了绝大部分的工作，Swift 中的基本数据类型例如 String、Int、Float 等都已经实现了 Codable 协议，因此如果你的数据类型只包含这些基本数据类型的属性，只需要在类型声明中加上 Codable 协议就可以了，不需要写任何实际实现的代码. 

## 正常使用
通过声明为Optional来避免服务端返回nil的情况.

```swift
    struct Student: Codable {
        let name: String?
        let amount: Double?
        let adress: String?
    }
    
    let okData = """
    {
       "name": "here",
     "amount": 100.0,
     "adress": "woodpecker avenue 1"
    }
    """.data(using: .utf8)!
    
    let decoder = JSONDecoder()
    let okStudent = try decoder.decode(Student.self, from:okData)
```

## 处理服务端返回的字段名不一致的情况

* Codable 通过在编译代码时根据类型的属性，自动生成了一个 CodingKeys 的枚举类型定义，这是一个以 String 类型作为原始值的枚举类型，对应每一个属性的名称。然后再给每一个声明实现 Codable 协议的类型自动生成 init(from:) 和 encode(to:) 两个函数的具体实现，最终完成了整个协议的实现.  
* 如果服务端返回的字段跟自己定义的字段不一致. 则可以自己手动实现`CodingKeys`的类型定义并设置对应的字段实现自定义字段的解析, 这样编译器不会自动生成默认的`CodingKeys`.

```swift
    struct Student: Codable {
        let studentName: String?
        let studentAge: Int?
    
        // 自定义字段解析  
         enum CodingKeys: String, CodingKey {
            case studentName = "student_name"
            case studentAge = "student_age"
        }
    }
```

## 处理多态的情况

[How can I implement polymorphic decoding of JSON data in Swift 4?](https://stackoverflow.com/questions/46595246/how-can-i-implement-polymorphic-decoding-of-json-data-in-swift-4)

**注意:**  重写`public init(from decoder: Decoder) throws`并对各字段进行单独解析时, 如果key为空, 可能会抛出异常, 所以都要用`container.decodeIfPresent`来处理.

## Date类型转换

```swift
    /// The strategy to use for decoding `Date` values.
    public enum DateDecodingStrategy {

        /// Defer to `Date` for decoding. This is the default strategy.
        case deferredToDate

        /// Decode the `Date` as a UNIX timestamp from a JSON number.
        case secondsSince1970

        /// Decode the `Date` as UNIX millisecond timestamp from a JSON number.
        case millisecondsSince1970

        /// Decode the `Date` as an ISO-8601-formatted string (in RFC 3339 format).
        @available(OSX 10.12, iOS 10.0, watchOS 3.0, tvOS 10.0, *)
        case iso8601

        /// Decode the `Date` as a string parsed by the given formatter.
        case formatted(DateFormatter)

        /// Decode the `Date` as a custom value decoded by the given closure.
        case custom(@escaping (Decoder) throws -> Date)
    }
```

系统提供了`DateDecodingStrategy`来处理日期格式, `iso8601`是`iOS 10.0`之后提供的. 如果需要自定义日期处理格式, 可以对`Formatter`进行拓展.
[How to convert a date string with optional fractional seconds using Codable in Swift4](https://stackoverflow.com/questions/46458487/how-to-convert-a-date-string-with-optional-fractional-seconds-using-codable-in-s)


