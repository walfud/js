---
title: js 精华
---

### 类型
* 值类型: 用 `typeof` 判断
  - undefined
  - number
  - string
  - boolean
* 引用类型: 用 `instanceOf` 判断
  - 普通对象
  - 数组对象
  - 函数对象
  - null 对象

### 函数
javascript 引擎会自动给函数添加一个 prototype 属性. 但是其他对象就没有.
prototype 的属性值是一个对象, 该对象只有一个叫做 constructor 的属性，指向这个函数本身
```js
obj = {};
console.log(obj.hasOwnProperty('prototype'));   // false

function fun() {}
console.log(fun.hasOwnProperty('prototype'));   // true
console.log(typeof fun.prototype);              // object
```
todo: http://www.cnblogs.com/wangfupeng1988/p/3978131.html  的图
