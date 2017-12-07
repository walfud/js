---
title: js 对象的扩展, 密封和冻结
tag:
  - js
date: 2017/11/30
---

js 运行过程中, 每个 '对象' 都有三种约束, 即 *禁止扩展*, *密封* 和 *冻结*

# 禁止扩展 (Object.preventExtensions)
禁止扩展的含义是 **某个对象不能添加新的属性**.例如:
```js
const foo = {}

// 可以随意添加属性
foo.a = 1  

// 对象被 *禁止扩展* 后, 则不能添加新属性
Object.preventExtensions(foo)
foo.b = 1   // 无法添加新属性. strict mode 下会抛异常

// 但是依然可以修改已有属性的值或者删除已有属性
foo.a = 2
delete foo.a

// 有一种假象: 我们可以修改对象的 __proto__ 属性, 从而间接 *扩展* 该对象
Object.getPrototypeOf(foo).b = 3
console.log(foo.b)    // 假象. 貌似 b 被扩展, 而实际上是原型链被扩展
foo.b = 4             // 无效. 无法给 b 添加新属性
```

## 相关链接
* [Object.preventExtensions](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions)
* [Object.isExtensible](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isExtensible)

# 密封 (Object.seal)
密封的含义是 **某个对象禁止扩展, 同时该对象所有的属性描述符 configurable 变为 false**(关于对象属性描述符, 参见 [js 对象属性描述符详解](http://js.walfud.com/js-%E5%AF%B9%E8%B1%A1%E5%B1%9E%E6%80%A7%E6%8F%8F%E8%BF%B0%E7%AC%A6%E8%AF%A6%E8%A7%A3/)). 例如:
```js
const bar = {
  x: 1,
  y: 2,
}

// 一旦 *密封*, 则无法修改属性描述符
Object.seal(bar)
Object.defineProperty(bar, 'x', {   // TypeError: Cannot redefine property: x
  enumerable: false,
})

// 当然, 也无法删除已有属性
delete bar.x      // 失败, 返回 false

// 一个 *密封* 的对象, 必然是 *禁止扩展* 的
console.log(Object.isExtensible(bar))  // false
```

有一个例外, 属性的 writable 可以由 true 变为 false, 但是反向是不可以的:
```js
const bar = {
  x: 1,
}

// 即使 *密封* 的对象, 其属性描述符 writable 也可以 true -> false 
Object.seal(bar)
Object.defineProperty(bar, 'x', {   // 成功. x 属性将变得不可写
  writable: false,
})

// writable 由 false -> true 是不可以的
Object.defineProperty(bar, 'x', {   // TypeError: Cannot redefine property: x
  writable: true,
})
```

## 相关链接
* [Object.seal](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/seal)
* [Object.isSealed](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isSealed)


# 冻结 (Object.freeze)
冻结的含义是 **某个对象 *密封*, 同时该对象所有属性描述符 writable 变为 false**. 例如:
```js
const baz = {
  a: 1,
  b: 2,
}

// 一旦 *冻结*, 无法修改属性描述符, 也无法修改属性的值
Object.freeze(baz)

Object.defineProperty(bar, 'a', {   // TypeError: Cannot redefine property: a
  enumerable: false,
})
delete baz.a   // 失败, 返回 false
baz.a = 100    // 无效

// 一个 *冻结* 的对象, 必然是 *密封* 的
console.log(Object.isSeal(baz))  // true
```

## 相关链接
* [Object.freeze](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
* [Object.isFrozen](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isFrozen)

# 总结
禁止扩展 ⊆ 密封 ⊆ 冻结

|         | 增加新属性 | 属性描述符 configurable | 属性描述符 writable | 属性描述符 enumerable | 属性的值 (value) |
|-|:-:|:-:|:-:|:-:|:-:|
| 禁止扩展 (Object.preventExtensions) | 不可以 | - | - | - | - |
| 密封 (Object.seal) | 不可以 | false | - | - | - |
| 冻结 (Object.freeze) | 不可以 | false | false | - | - |