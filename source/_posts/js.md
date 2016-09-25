---
title: js 精华
tag:
  - js
---

### 类型
* 值类型: 用 `typeof` 判断
  - undefined
  - null  (`typeof` 返回是 object. 这是一个被大家接受的 bug)
  - number
  - string
  - boolean
* 引用(对象) 类型: 用 `instanceOf` 判断
  * 普通对象
  * 函数对象
  * 数组对象


### 原型
* `__proto__`
  - 每个对象都有个 `__proto__` 属性
  - 指向另一个对象, 该对象则是本对象的原型
  - 由创建该对象的函数的 `prototype` 赋值而来
* `prototype`
  - 每个函数对象还有一个 `prototype` 属性
  - 指向另一个对象
  - 用于创建新对象的时候, 为新对象的 `__proto__` 赋值
* `constructor`
  - 对于 `prototype` 所指向的对象中存在. 如: Object.prototype 中, 一定会有 constructor 属性
  - 指向一个函数对象, 该函数对象一般用于构造新对象

有了上述基础, 我们来看一下 js 几个引擎内建对象的关系:

![](/images/js/prototype.png)

_(<font color="#fa1716">红色是普通对象</font>, <font color="#264ccf">蓝色是函数对象</font>)_


首先, 引擎有个内建的 A 对象, 该对象:
* 是所有 **对象**(包括所有对象类型) 的原型
* 本对象 `__proto__ == null`. 也就是说: 本对象没有原型
* 提供了最基础的对象方法, 包括:
  - [hasOwnProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)
  - [isPrototypeOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isPrototypeOf)
  - [propertyIsEnumerable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable)
  - [toLocaleString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toLocaleString)
  - [toSource](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toSource)
  - [toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)
  - [unwatch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/unwatch)
  - [valueOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf)
  - [watch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/watch)

B 依然是引擎内建对象:
* 是所有 **'函数对象'** 的原型
* 本对象的 `__proto__` 指向了 A. 换句话说: B 扩展了 A, 并增加了一些函数会用到的方法, 如下
* 提供了最基础的函数方法, 包括:
  - [apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
  - [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
  - [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
  - [isGenerator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/isGenerator)
  - [toSource](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/toSource)
  - [toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/toString)

C

http://www.cnblogs.com/wangfupeng1988/p/3979533.html: 沿着A的__proto__这条线来找，同时沿着B的prototype这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回true。如果找到终点还未重合，则返回false
