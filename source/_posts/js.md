---
title: js
tag:
  - js
---

TODO:

### 类型
js 类型分为如下几种:
* 值类型
  - undefined
  - null
  - number
  - string
  - boolean
* 引用(对象) 类型
  - 对象
    * 普通对象
      - 最普通的对象: 具有 `__proto__`, 指向其原型链
      - 被 `prototype` 指向的对象: 这类对象除了有 `__proto__` 以外, 还有 `constructor` 属性, 指向构造函数对象
    * 函数对象
      - 除了具有 `__proto__` 以外, 还具有 `prototype` 属性, 指向通过本函数生成对象的原型
    * 数组对象
