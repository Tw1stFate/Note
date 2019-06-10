##DZNEmptyDataSet

### 场景: 很方便的做空白页/错误页展示
### 源码阅读:
* 主要类: `DZNTableDataSetView`(没内容时显示的空白页) / `UITableView+DataSet`
* 以tableview为例:
	* 分类中新增方法
	
	```objc
	    - (void)setEmptyDataSetSource:(id<DZNEmptyDataSetSource>)source
{
    // Registers for device orientation changes
    if (source && !self.emptyDataSetSource) {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(deviceDidChangeOrientation:) name:UIDeviceOrientationDidChangeNotification object:nil];
    }
    else if (!source && self.emptyDataSetSource) {
        [[NSNotificationCenter defaultCenter] removeObserver:self name:UIDeviceOrientationDidChangeNotification object:nil];
    }
    
    objc_setAssociatedObject(self, kEmptyDataSetSource, source, OBJC_ASSOCIATION_ASSIGN);
    
    if (![self dzn_canDisplay]) {
        return;
    }
    
    // We add method sizzling for injecting -dzn_reloadData implementation to the native -reloadData implementation
    [self swizzle:@selector(reloadData)];
    
    // If UITableView, we also inject -dzn_reloadData to -endUpdates
    if ([self isKindOfClass:[UITableView class]]) {
        [self swizzle:@selector(endUpdates)];
    }
}
	```

	```objc
- (void)setEmptyDataSetDelegate:(id<DZNEmptyDataSetDelegate>)delegate
{
    objc_setAssociatedObject(self, kEmptyDataSetDelegate, delegate, OBJC_ASSOCIATION_ASSIGN);
    
    if (!delegate) {
        [self dzn_invalidate];
    }
}

	```
	
	这样就可以调用下面的方法
	
	```objc
	    _tableView.emptyDataSetSource = self;
        _tableView.emptyDataSetDelegate = self;

	```
	然后触发setter方法, 通过KVO监控列表页的内容变化,  通过runtime给UIscrollview类新增属性. 并且在reloadData方法中注入额外的实现`dzn_reloadEmptyDataSet`. 其他的自己看源码.

