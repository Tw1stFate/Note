# JSPatch

### 宏
替换成原始数据
`[[UIScreen mainScreen] bounds].size.width` 替换成 `UIScreen.mainScreen().bounds().width` (**注意没有size**)

### CGRectMake

{x:0, y:0 width:100, height:100}

### UIEdgeInsetsMake

先声明, 然后就能`{top:0, left:0, bottom:0, right:0}`


```objc
require('JPEngine').defineStruct({
                            "name": "UIEdgeInsets",
                            "types": "FFFF",
                            "keys": ["top", "left", "bottom", "right"]
                            });
```

### 枚举值

替换成数值

```objc
btn.setTitle_forState('Push JPTableViewController', 0)
btn.addTarget_action_forControlEvents(self, "handleBtn:", 1 << 6)
```

### array, dictionary

获取长度: `array.toJS().length`

如果有加`autoConvertOCType(1); ` , 可以直接使用`array[index]`. 

否则:
**创建数组不能是用字面量语法.**
通过索引取值用`array.objectAtIndex(index)`

`dictionary`同理, `dict.objectForKey(key)`

### block
* js向oc传递block, 需要使用 block(paramTypes, function) 函数封装. 在 Objective-C 中传入到 JSPatch 中的 Block 会转换为 function，如果需要再将该 Block 传回到 OC，依旧需要用 block(paramTypes, function) 封装

```objc
// Obj-C  
@implementation JPObject  
+ (void)request:(void(^)(NSString *content, BOOL success))callback
{ 
  callback(@"I'm content", YES); 
  
  [self testBlock:callback]; //直接将blcok传给其他函数.
} 
@end 
// JS 调用
require('JPObject').request(block("NSString *, BOOL", function(ctn, succ) { 
//block implementation
}))

//重写, 传递给其他函数写法(在移动oa写补丁文件的时候结果参数并没有回调过去,一直是nil, 暂时不知原因)
self.testBlock(block("NSString*, BOOL", callback));
```

* 不允许在 JS 的 block 中传入含有 undefined 的数组或对象。比如传入 ["JSPatch", undefined] 的数组或 {obj: undefined} 的对象，这种语法在 JavaScript 中没有错误，但是在 Objective-C 中是不允许的.
* 如果 block 的参数里含有 block，paramTypes 需要写成 NSBlock *.

```objc
var blk = block("BOOL, NSBlock *", function(b, blk){
  if (b) {
    blk(true)
  }
})
```

* 暂时不支持向OC端传递含有浮点型参数的block.
* 在block中使用到self时，需要在使用block之前先将self赋值给另外一个变量，然后在block中使用这个变量.

```objc
//JSPatch脚本
var slf = self
self.requestWithCompletion(block("BOOL, NSError*", function(success, error){
    slf.label().setText("complete")
    })
)
```




