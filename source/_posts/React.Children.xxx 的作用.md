---
title: React.Children.xxx 的作用
tag:
  - react
date: 2017/11/2
---

我们在编写 React 的时候, 经常会遇到 `React.Children.map(...)` 这种写法. 然而 js 中明明有对等的函数, 如下:
* `React.Children.map(this.props.children, ...)` <=> `this.props.children.map(...)`
* `React.Children.forEach(this.props.children, ...)` <=> `this.props.children.forEach(...)`
* `React.Children.count(this.props.children)` <=> `this.props.children.length`
那么为什么还会有 `React.Children.xxx` 呢?

要想知道这个问题, 首先要回答 `this.props.children` 到底是什么.

### `this.props.children` 到底是什么
我们来列举几种 case
![](/images/React.Children.xxx 的作用/React.Children.xxx-typeof.png)
可见, **children 可以是任何类型**

### `this.props.children.count / map` 有什么问题?
![](/images/React.Children.xxx 的作用/React.Children.xxx-result.png)
这就很明显了, 当 children 为 *Component 数组* 的时候, `React.Children.xxx` 和 `this.props.children.xxx` 都可以正常工作. 

同样, `Set` 也属于可迭代对象, 所以 `React.Children` 系列也能够正确的计算出长度, 而 `this.props` 系列显然无能为力. 但是 React 对于 `Map` 作为 children 是不支持的, 会得到一个 warning *'Using Maps as children is unsupported and...'*

当 children 是 **单一对象** 时, `React.Children.xxx` 可以正常计算 count, 但是 `this.props.children` 由于不是数组, 所以 length 操作会失败. 

但是无论 `React.Children` 系列还是 `this.props` 系列, 对于 function 而言都是有问题的. 这里并不是说不能够使用 function 作为 children, 而是说如果你要将 function 作为 children 传入, 希望你知道自己在做什么.

### 结论
1. 除非使用 PropTypes 限定, 否则不要对 children 有任何假设, **children 可以是任何类型**
2. React.Children 提供了一些方便的函数帮助我们遍历和统计 children, 不要自己重造轮子了

# refs
[A deep dive into children in React](https://mxstbr.blog/2017/02/react-children-deepdive/)
