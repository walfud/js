---
title: js 对象的扩展, 密封和冻结
tag:
  - js
date: 2017/11/30
---

js 运行过程中, 每个 '对象' 都有三种约束, 即 *禁止扩展*, *密封* 和 *冻结*

# 禁止扩展 (Object.preventExtensions)
常规创建的对象都是可扩展的, 即可以在对象上随意添加属性. 例如:
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

# 密封 (Object.seal)

# 冻结 (Object.freeze)