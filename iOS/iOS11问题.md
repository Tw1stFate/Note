# 总结下ios11自己遇到的问题
#### 第一次更新时运行, `fmdb`crash. 
    修改头文件导入方式.
####  邮箱/日程 等使用到侧滑的地方, 侧滑过度一点就会造成cell直接被删除. 
     `ios11`更新了对应的代理方法.
#### 相册写入权限变更.
#### Safe Area.
#### iOS11的UIToolbar 增添了有一个UIToolbarContentView的子控件，覆盖在最表层，以至于添加的button都在底部，点击没有反应。
    在实例化UIToolBar之后， 添加 [toolbar layoutIfNeeded]; 即可.
#### 在iOS 11中默认启用Self-Sizing.
    如果目前项目中没有使用estimateRowHeight属性，在iOS11的环境下就要注意了，
    因为开启Self-Sizing之后，tableView是使用estimateRowHeight属性的，这样就会造成contentSize和contentOffset值的变化，
    如果是有动画是观察这两个属性的变化进行的，就会造成动画的异常，因为在估算行高机制下，contentSize的值是一点点地变化更新的，所有cell显示完后才是最终的contentSize值。
    因为不会缓存正确的行高，tableView reloadData的时候，会重新计算contentSize，就有可能会引起contentOffset的变化。
    iOS11下不想使用Self-Sizing的话，可以通过以下方式关闭：

```
self.tableView.estimatedRowHeight = 0;
self.tableView.estimatedSectionHeaderHeight = 0;
self.tableView.estimatedSectionFooterHeight = 0;
```
#### 多出了`_UIInteractiveHighlightEffectWindow`私有类.

#### 机制变化

`[cell.contentView addSubview:self.textView];` `ios11.1以前`下面这么写没问题, 之后就不行. 由于用的是`weak`, 在将`self.textView`添加到`cell`上并不能像之前那样直接强引用`textView`.

```
@property (nonatomic, weak) UITextView *textView;

[cell.contentView addSubview:self.textView];
#pragma mark - getter
- (UITextView *)textView {
    if (!_textView) {
        _textView = [[UITextView alloc] init];
        ...
    }
    return _textView;
}
```
#### UIBarButtonItem默认使用AutoLayout导致的布局问题.
[navigation bar rightbaritem image-button bug iOS 11](https://stackoverflow.com/questions/44442573/navigation-bar-rightbaritem-image-button-bug-ios-11)


