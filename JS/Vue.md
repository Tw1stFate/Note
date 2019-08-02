# 文档阅读笔记
> [社区](https://cn.vuejs.org/)
> [文档教程](https://cn.vuejs.org/v2/guide/)

#### 模板
* 如果你在 DOM 中使用模板 (直接在一个 HTML 文件里撰写模板)，需要留意浏览器会把特性名全部强制转为小写.

```
<!-- 在 DOM 中使用模板时这段代码会被转换为 `v-bind:[someattr]` -->
<a v-bind:[someAttr]="value"> ... </a>
```

#### 指令
* `v-`为指令前缀, `v-bind` 和 `v-on` 这两个指令太常用了, 简写为`:` 和 `@`.

```
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

#### 计算属性
* 可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是**计算属性是基于它们的响应式依赖进行缓存的。只在相关响应式依赖发生改变时它们才会重新求值**。

* 计算属性默认只有`getter`, 也可以提供`setter`.

* 能用`计算属性`的时候就不要滥用`侦听属性watch`. `watch`可用于检测属性改变时发送网络请求(作为参数)的场景.

#### 单文件组件

怎么看待关注点分离？

* 一个重要的事情值得注意，**关注点分离不等于文件类型分离**.
* 相比于把代码库分离成三个大的层次并将其相互交织起来，把它们划分为松散耦合的组件再将其组合起来更合理一些。
* 在一个组件里，其模板、逻辑和样式是内部耦合的，并且把他们搭配在一起实际上使得组件更加内聚且更可维护.

# 其他
## `render: h => h(App) `是什么意思?
[Explanation for `render: h => h(App)`](https://github.com/vuejs-templates/webpack-simple/issues/29#issuecomment-312902539)

render: h => h(App) 是下面内容的缩写：

```
render: function (createElement) {
    return createElement(App);
}
```

进一步缩写为(ES6 语法)：

```
render (createElement) {
    return createElement(App);
}
```

再进一步缩写为：

```
render (h){
    return h(App);
}
```

按照 ES6 箭头函数的写法，就得到了：

```
render: h => h(App);
```

## 组件之间传值

	父子组件之间传值
		父组件向子组件传值: 通过props.
		子组件向父组件传值: 通过事件触发夹带参数的方式传值.

	非父子组件之间传值
		bus/总线/发布订阅模式/观察者模式
	

## 

