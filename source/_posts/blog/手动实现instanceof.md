---
title: 实现instanceof
categories:
- 前端
tags:
- JS
- 手撕
cover: https://tva1.sinaimg.cn/large/00897VxHly8gpso9oxqxpj30st0gndjt.jpg
---
## 原理

```js
a instanceof Object
```

判断`Object`的prototype是否在`a`的原型链上。

## 实现

```js
    function myInstanceof(target, origin) {
      const proto = target.__proto__;
      if (proto) {
        if (origin.prototype === proto) {
          return true;
        } else {
          return myInstanceof(proto, origin)
        }
      } else {
        return false;
      }
    }
```

