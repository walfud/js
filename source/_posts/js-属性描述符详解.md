---
title: js 属性描述符详解
tag:
  - js
date: 2017/11/27
---

# 什么是 property, 什么是 descriptor
对象的没一个属性(property), 实际上都有一组描述符(descriptor) 对其进行约束. 比如我们日常写的 `foo.a = 1`, 实际上对应的底层实现是:
```js
foo.a = 1 
// 等价于
Object.defineProperty(foo, 'a', {
  value: 1,
  writable: true,
  configurable: true,
  enumerable: true,
})
```

那么到底这些描述符都是干什么的, 又有多少个描述符呢? 我们往下看

# 一个 property 到底有多少个 descriptor
简言之, 一个 property 一共有 6 个 descriptor, 他们是:
* value / writable
* get / set
* configurable
* enumerable

这些 descriptor 被分为两类, data descriptor 和 access descriptor. 

## data descriptor
value/writable 称之为 data descriptor. 它可以组合 configurable / enumerable, 比如:
```js
Object.defineProperty(foo, 'a', {
  value: 1,
  writable: true,
  configurable: true,
  enumerable: true,
})
```

## access descriptor
get/set 称之为 access descriptor. 它也可以组合 configurable / enumerable, 比如:
```js
Object.defineProperty(foo, 'a', {
  get: function() { ... },
  set: function() { ... },
  configurable: true,
  enumerable: true,
})
```

但是 data descriptor 和 access descriptor 是互斥的, 也就是说, 一个 property 要么有 value/writable, 要么有 get/set, 但是不能同时拥有两者.

# descriptor 的作用
## writable
*writable* 定义了对象的属性是否能够被重新赋值. 赋值操作符中默认是 true, *Object.defineProperty()* 函数中默认值是 false

如果 writable 为 false, 那么给该属性赋值时, 在严格模式下会抛出异常, 非严格模式下赋值操作无效.
```js
'use strict'
const foo = {}
Object.defineProperty(foo, 'a', {
  value: 1,
  writable: false,
})

foo.x = 2  // TypeError: Cannot assign to read only property 'x' of object '#<Object>'
```

## enumerable
*enumerable* 定义了对象的属性是否可以在 *for...in* 循环和 *Object.keys()* 中被枚举. 赋值操作符中默认是 true, *Object.defineProperty()* 函数中默认值是 false
```js
const foo = {}
Object.defineProperty(foo, 'a', {
  value: 1,
  enumerable: true,
})
Object.defineProperty(foo, 'b', {
  value: 2,
  enumerable: false,
})

for (let i in foo) {
  console.log(i)
}

// 输出了 a

Object.keys(foo).forEach((i) => console.log(i))

// 输出了 a
```

## configurable
*configurable* 定义了对象的属性是否可以被删除或者修改. 赋值操作符中默认是 true, *Object.defineProperty()* 函数中默认值是 false

但是有一条例外, 无论 configurable 配置如何, 将 writable 由 true => false 的转变总是能够成功.
```js
const foo = {}
Object.defineProperty(foo, 'a', {
  value: 1,
  configurable: false,
  writable: true,
  enumerable: true,
})

Object.defineProperty(foo, 'a', {
  enumerable: false,
})

// TypeError: Cannot redefine property: a
```

# 总结
## data descriptor 和 access descriptor
描述符被分为两组:
* data descriptor: value/writable 与 configurable/enumerable 的组合
* access descriptor: get/set 与 configurable/enumerable 的组合

## descriptor 作用和默认值
| descriptor | 作用 | 赋值操作时的默认值 | Object.defineProperty 时的默认值 |
| - | - | - | - |
| value | 设置 property 的值 | true | false |
| writable | property 的值可以被修改. 3, property 本身可以被 delete | true | false |
| get | 读该 property 时返回的值 | undefined | undefined |
| set | 写 property 时的具体操作 | undefined | undefined |
| configurable | 该 property 的 descriptor 是否能够被再次修改 | true | false |
| enumerable | 该 property 是否能够被 for .. in 和 Object.keys 枚举到 | true | false |

```js
const foo = {};

foo.a = 1;
// 等同于 :
Object.defineProperty(o, "a", {
  value : 1,
  writable : true,
  configurable : true,
  enumerable : true
});


// 另一方面，
Object.defineProperty(foo, "a", { value : 1 });
// 等同于 :
Object.defineProperty(foo, "a", {
  value : 1,
  writable : false,
  configurable : false,
  enumerable : false
});
```

# refs
[Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)