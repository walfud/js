---
title: 彻底搞懂 js prototype
tag:
  - js
  - 彻底搞懂
---

本文不是基础教程, 如果对原型的基础概念还不理解, 请先看看 ref 中的链接.

###### 基础概念
先来看看基础概念:
* 普通对象
  - `__proto__` 指向该对象的原型 (后面有解释), 通常是一个 'Object 原型对象' (后图中的 A)
* 构造函数对象
  - `__proto__` 指向本函数对象的原型 (后面有解释), 通常是一个 'Function 原型对象' (后图中的 B)
  - `prototype` 的作用是: 该函数所创建对象后, 为被创建对象的 `__proto__` 赋值. 所以任何通过 `new Xxx()` 创建的对象的 `__proto__` 都等于 Xxx.prototype, 即 `new Xxx().__proto__ === Xxx.prototype`
* prototype 对象
  - 本对象也是一个普通对象
  - `__proto__` 指向本对象的原型, 通常是一个 'Object 原型对象' (后图中的 A)
  - `constructor` 是个反向链接, 反过来指向构造函数对象 (见后图)

有了上述基础, 我们来看一下 js 几个引擎内建对象的关系:

![](/images/js_prototype/prototype.png)

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

B 也是引擎内建对象:
* 是所有 **'函数对象'** 的原型
* 本对象的 `__proto__` 指向了 A. 换句话说: B 扩展了 A, 并增加了一些函数会用到的方法, 如下
* 提供了最基础的函数方法, 包括:
  - [apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
  - [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
  - [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
  - [isGenerator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/isGenerator)
  - [toSource](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/toSource)
  - [toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/toString)

C 和 D 依然是引擎的内建对象, 不过这两个是内建的函数对象:
* `Object` (即 C 对象) 函数:
  - `__proto__` 指向 B, 即: `Object` 自身是一个函数对象
  - `prototype` 指向 A, 即: 通过 `new Object()` 创建的对象都是 '普通对象'
* `Function` (即 D 对象) 函数:
  - `__proto__` 指向 B, 即: `Function` 自身是一个函数对象
  - `prototype` 指向 B, 即: 通过 `new Function()` 创建的对象都是 '函数对象'

综上, 可以理解为: `Object` 和 `Function` 都是函数对象, 但是: `new Object()` 创建的都是普通对象(被创建的对象 `__proto__` 指向 A), 而被 `new Function()` 创建的对象都是函数对象(`__proto__` 指向 B).

### 是时候捋一捋脉络了. 看一下完整的 js 原型图:

![](/images/js_prototype/js.png)

从上往下看.
###### 引擎内建级对象
  最上面是 `null`, 下面紧接着是一切对象的原型 --- `Op`, 其中包含了 `hasOwnProperty / toString...` 等 **对象** 所需要的方法. `Op.__proto__ == null` 代表着 `Op` 没有原型.
  `Op` 下面分别是 `Fp`, `Sp` (另一个泛指代表其他内建类型). `Fp` 是一切 **函数对象** 的原型, 包含了 `call / bind` 等 **函数** 所需的方法. 不过 `Fp` 也是一个对象, 所以它的原型是 `Op`(`Op` 是一切对象的原型). `Sp` 同理.

###### API 级对象
  再往下看, 这一层级是我们常见的 js API. 以 `Object`(图中 _this is the built-in \`Object\` function_ 所代表的对象) 为例. `Object` 自身是一个对象, 因此其 `__proto__` 指向 `Op`. 但是 `Function` 又是个函数, 所以它将具有 `prototype` 属性. 然而它的功能是创建新的 **普通对象**, 所以其 `prototype` 指向了 `Op`, 这代表凡是 `const myObj = new Object()` 创建的对象, 那么 `myObj.__proto__ === Op`(因为 `new` 方法会将 `Object.prototype` 赋值给新创建的对象).

###### 用户级对象
  看图中最后一行的 custom object 和 custom function object. 这些都是用户自定义对象, 也就是我们日常代码中的各种 `const myObj = { ... }`.

  默认情况下, 引擎在我们创建对象时帮我们把新建 **普通对象** 的 `__proto__` 设置为 `Op`,  **函数对象** 的 `__proto__` 设置为 `Fp`. 因此我们自定义的对象才能够调用 `getOwnProperty / call` 等方法.


### MISC.
###### 判断自定义对象原型 (A instanceOf B)
沿着 A 的 `__proto__` 这条线来找，同时沿着 B 的 `prototype` 这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回 `true`。如果找到终点还未重合，则返回 `false`.

### refs:
* [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
* [深入理解javascript原型和闭包（完结）](http://www.cnblogs.com/wangfupeng1988/p/3979533.html)
