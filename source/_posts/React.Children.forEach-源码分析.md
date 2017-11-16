---
title: React.Children.forEach 源码分析
tag:
  - react
date: 2017/11/15
---

<!-- TOC -->

- [追根溯源](#追根溯源)
- [框架分析](#框架分析)
- [细节源码](#细节源码)
    - [getPooledTraverseContext](#getpooledtraversecontext)
    - [traverseAllChildren](#traverseallchildren)
        - [绑定上下文调用处理函数](#绑定上下文调用处理函数)
        - [递归遍历](#递归遍历)
            - [处理单个 child](#处理单个-child)
            - [处理 children (集合)](#处理-children-集合)
    - [releaseTraverseContext](#releasetraversecontext)
    - [回顾](#回顾)
- [总结](#总结)
- [Refs](#refs)

<!-- /TOC -->

关于为什么会有 `React.Children.forEach` 而不直接用 `this.props.forEach` 的问题, 我在 [React.Children.xxx 的作用](http://js.walfud.com/React.Children.xxx%20%E7%9A%84%E4%BD%9C%E7%94%A8/) 已经说得很明白了. 本文我们要更进一步, 分析一下 `React.Children` 的实现, 从而更好的理解 react 背后的故事. Let's go.

# 追根溯源
几经跳转, 我们能看到 `React.Children` 的所有方法, 如下:
```js
var React = {
  Children: {
    map: ReactChildren_1.map,
    forEach: ReactChildren_1.forEach,
    count: ReactChildren_1.count,
    toArray: ReactChildren_1.toArray,
    only: onlyChild_1
  },
  ...
}
```
这几个函数除了 `only` 以外, 都要对 children 进行遍历. 而它们的代码结构也非常类似, 我们只要看懂了一个, 其他的就都通了. 

我们来重点分析 `forEach` 方法, 因为它是一个比较纯粹的遍历函数.

上面的代码中, `ReactChildren_1` 仅仅是 `ReactChildren` 的引用, 如下:
```js
var ReactChildren = {
  forEach: forEachChildren,
  map: mapChildren,
  count: countChildren,
  toArray: toArray
};

var ReactChildren_1 = ReactChildren;
```

因此, 最终代码实际上调用的是 `forEachChildren` 方法:
```js
function forEachChildren(children, forEachFunc, forEachContext) {
  if (children == null) {
    return children;
  }
  var traverseContext = getPooledTraverseContext(null, null, forEachFunc, forEachContext);
  traverseAllChildren(children, forEachSingleChild, traverseContext);
  releaseTraverseContext(traverseContext);
}
```
别问我为什么有什么多层赋值, 我也不知道啊...

# 框架分析
大部分 `React.Children` 函数都是这个套路, 
1. `getPooledTraverseContext`: 作用是将你的 `forEachFunc` 函数 `forEachContext`(相当于 `this` 指针) 封装在一起, 便于传入后面的 `traverseAllChildren` 方法
2. `traverseAllChildren` 这里具体执行了遍历, 它内部会进行 children 的递归操作
3. `releaseTraverseContext` 是将第一步 `getPooledTraverseContext` 的结果 "释放" 掉. 为什么需要释放? 是因为第一步相当于从一个 pool 中取了一个对象来用, 这里用完了, 要放回到 pool 中. 看完后面 '细节代码' 你就明白了

这里可以总结一下框架思路:
1. 把 *用户 forEach 传入的处理函数* 和 *传入的上下文* 打包成一个 context 对象
2. 递归遍历 children, 执行上述 context 中的 *用户处理函数*
3. 释放 1 中的 context

# 细节源码
下面, 我们进入重点, 来看一看其中的实现原理

## getPooledTraverseContext
```js
var POOL_SIZE = 10;
var traverseContextPool = [];
function getPooledTraverseContext(mapResult, keyPrefix, mapFunction, mapContext) {
  if (traverseContextPool.length) {
    var traverseContext = traverseContextPool.pop();
    traverseContext.result = mapResult;
    traverseContext.keyPrefix = keyPrefix;
    traverseContext.func = mapFunction;
    traverseContext.context = mapContext;
    traverseContext.count = 0;
    return traverseContext;
  } else {
    return {
      result: mapResult,
      keyPrefix: keyPrefix,
      func: mapFunction,
      context: mapContext,
      count: 0
    };
  }
}
```
`getPooledTraverseContext` 实际上维护了一个 size 为 10 的缓冲池. 如果 pool 中有存货, 则 pop 出一个进行使用. 如果 pool 中空空如也, 则 return 一个新的对象.

当然, 这个函数的重点并不是缓冲池, 而是返回的对象本身. 要记住这两个字段:
* func: 这就是用户传入的 *forEach 处理函数*
* context: 这是个可选参数, 用户可以传入作为调用上述 `func` 时的上下文. 看到这里你就知道, 默认情况下, 你的 **处理函数执行的时候, 是没有 context, 也就是处理函数中, this === undefined**. 如果想在 *处理函数中绑定 this*, 只能通过这个参数指定. 这一点在后面分析 `forEachSingleChild` 会看到原理

## traverseAllChildren
这是我们最重要的函数, 我们来回顾一下他的参数:
```js
function forEachChildren(children, forEachFunc, forEachContext) {
  ...
  traverseAllChildren(children, forEachSingleChild, traverseContext);
  ...
}
```

### 绑定上下文调用处理函数
其中, `children` 是要遍历的子节点对象, `traverseContext` 是上一步封装了 *处理函数* 和 *处理函数执行上下文* 的一个对象. 而 `forEachSingleChild` 则是真正调用处理函数的方法:
```js
function forEachSingleChild(bookKeeping, child, name) {
  var func = bookKeeping.func,
      context = bookKeeping.context;

  func.call(context, child, bookKeeping.count++);
}
```

看, `func.call(context, child, ...)` 这就是绑定上下文调用处理函数的秘密. 

### 递归遍历
`traverseAllChildren` 只是个入口, 真实的递归遍历是定义在 `traverseAllChildrenImpl` 中:
```js
// 外层入口
function traverseAllChildren(children, callback, traverseContext) {
  if (children == null) {
    return 0;
  }

  return traverseAllChildrenImpl(children, '', callback, traverseContext);
}

// 真正递归遍历的实现
function traverseAllChildrenImpl(children, nameSoFar, callback, traverseContext) {
  ...
}
```

我们来重点分析一下 `traverseAllChildrenImpl`. 

#### 处理单个 child
先来看上半部分:
```js
function traverseAllChildrenImpl(children, nameSoFar, callback, traverseContext) {
  var type = typeof children;

  if (type === 'undefined' || type === 'boolean') {
    // All of the above are perceived as null.
    children = null;
  }

  if (children === null || type === 'string' || type === 'number' ||
  // The following is inlined from ReactElement. This means we can optimize
  // some checks. React Fiber also inlines this logic for similar purposes.
  type === 'object' && children.$$typeof === REACT_ELEMENT_TYPE) {
    callback(traverseContext, children,
    // If it's the only child, treat the name as if it was wrapped in an array
    // so that it's consistent if the number of children grows.
    nameSoFar === '' ? SEPARATOR + getComponentKey(children, 0) : nameSoFar);
    return 1;
  }

  ... 下部分处理递归, 后面分析 ... 
}
```
`typeof` 操作符一共能有几种返回值? 来看看:

| 类型        | 返回值           |   备注   |
| ------------- |-------------|-----|
| <font color=red>Undefined</font> | "undefined"| |
| Null | "object"| 实际上是被 `traverseAllChildren` 在入口被处理了 |
| <font color=red>Boolean</font> | "boolean"| |
| <font color=red>Number</font> | "number"| |
| <font color=red>String</font> | "string"| |
| ~~Symbol~~ | "symbol"| |
| ~~函数对象~~ | "function"| |
| ~~任何其他对象~~ | "object"| |

*注*: 红色部分是当前被处理的类型

这个分支处理了大部分类型. 这里的 children 实际上是单个对象, 并不是像它的名字一样是个复数. 接下来执行 `callback(traverseContext, children, nameSoFar === '' ? SEPARATOR + getComponentKey(children, 0) : nameSoFar)` 并返回 1.

看到这里, 我们就知道, children 不仅仅可以是 Component, 还可以是 String/Boolean/Undefined 等等.

我们回忆一下 `traverseAllChildren` 就会知道, 这个 callback 实际上是 `forEachSingleChild`:
```js
function forEachChildren(children, forEachFunc, forEachContext) {
  ...
  traverseAllChildren(children, forEachSingleChild, traverseContext);
  ...
}
```

而 `forEachSingleChild` 则调用 `func.call(context, child, ...)` 实现了绑定上下文调用处理函数. 所以 `callback(traverseContext, children, ...)` 可以简单理解为以 `children`(实际上是单个对象, 并不是集合, 名字容易误导读者) 为参数, 调用了用户的处理函数.

#### 处理 children (集合)
```js
function traverseAllChildrenImpl(children, nameSoFar, callback, traverseContext) {
  
  ... 上部分处理单个 child, 已经分析过了. 下面来分析递归调用处理 children ... 

  var child;
  var nextName;
  var subtreeCount = 0; // Count of children found in the current subtree.
  var nextNamePrefix = nameSoFar === '' ? SEPARATOR : nameSoFar + SUBSEPARATOR;

  if (Array.isArray(children)) {

    // !! 如果是 Array, 则深度递归 !!

    for (var i = 0; i < children.length; i++) {
      child = children[i];
      nextName = nextNamePrefix + getComponentKey(child, i);
      subtreeCount += traverseAllChildrenImpl(child, nextName, callback, traverseContext);
    }
  } else {

    // !! 如果不是 Array, 则看该对象是否可迭代 !!

    var iteratorFn = ITERATOR_SYMBOL && children[ITERATOR_SYMBOL] || children[FAUX_ITERATOR_SYMBOL];
    if (typeof iteratorFn === 'function') {

      // !! 如果是 Map, 则警告用户不支持 !!

      {
        // Warn about using Maps as children
        if (iteratorFn === children.entries) {
          warning$2(didWarnAboutMaps, 'Using Maps as children is unsupported and will likely yield ' + 'unexpected results. Convert it to a sequence/iterable of keyed ' + 'ReactElements instead.%s', getStackAddendum());
          didWarnAboutMaps = true;
        }
      }

      // !! 其他可迭代对象, 则使用迭代方法, 深度遍历 !!

      var iterator = iteratorFn.call(children);
      var step;
      var ii = 0;
      while (!(step = iterator.next()).done) {
        child = step.value;
        nextName = nextNamePrefix + getComponentKey(child, ii++);
        subtreeCount += traverseAllChildrenImpl(child, nextName, callback, traverseContext);
      }
    } else if (type === 'object') {

      // !! 如果该对象不可迭代, 则提示错误 !!

      var addendum = '';
      {
        addendum = ' If you meant to render a collection of children, use an array ' + 'instead.' + getStackAddendum();
      }
      var childrenString = '' + children;
      invariant_1(false, 'Objects are not valid as a React child (found: %s).%s', childrenString === '[object Object]' ? 'object with keys {' + Object.keys(children).join(', ') + '}' : childrenString, addendum);
    }
  }

  return subtreeCount;
}
```

看代码中的注释, 应该很清楚的明白 `React.Children.forEach` 是不支持 `Map` 但却支持 `Set` 的. 

## releaseTraverseContext
```js
function releaseTraverseContext(traverseContext) {
  traverseContext.result = null;
  traverseContext.keyPrefix = null;
  traverseContext.func = null;
  traverseContext.context = null;
  traverseContext.count = 0;
  if (traverseContextPool.length < POOL_SIZE) {
    traverseContextPool.push(traverseContext);
  }
}
```

这个方法简单的不能再简单, 核心目的就是 `if` 里面的那块代码, 如果池数量小于 `POOL_SIZE`(上文中得知这个数字是 10), 则把对象放回到池中, 以备后续使用.

## 回顾
至此, `React.Children.forEach` 就分析完了. 回过头来看看:
```js
function forEachChildren(children, forEachFunc, forEachContext) {
  
  // !! 将 *处理函数* 和 *上下文* 封装成一个对象(`traverseContext`) !!
  var traverseContext = getPooledTraverseContext(null, null, forEachFunc, forEachContext);

  // !! 深度遍历子元素, 并调用 *处理函数* !!
  traverseAllChildren(children, forEachSingleChild, traverseContext);

  // !! 释放封装了 *处理函数* 和 *上下文* 的对象 !!
  releaseTraverseContext(traverseContext);

}
```

这三步, 是不是很简单?

# 总结
我们回顾一下 `React.Children.forEach` 能够处理的类型:

| 类型        | 返回值           |   备注   |
| ------------- |-------------|-----|
| Undefined | "undefined"| |
| Null | "object"| 被 `traverseAllChildren` 在入口被处理 |
| Boolean | "boolean"| |
| Number | "number"| |
| String | "string"| |
| ~~Symbol~~ | "symbol"| |
| ~~函数对象~~ | "function"| |
| 可迭代对象 | "object"| |
| ~~其他对象~~ | "object"| |

总的来说, `React.Children.forEach` 是通过 `typeof` 操作符, 对 children 进行判断, 进行深度遍历后完成了任务.

# Refs
[React.Children.xxx 的作用](http://js.walfud.com/React.Children.xxx%20%E7%9A%84%E4%BD%9C%E7%94%A8/)