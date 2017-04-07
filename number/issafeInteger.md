## 基本描述

#### 语法

```javascript
Number.isSafeInteger(value)
```

#### 参数

* value 要被检测是否为安全整数的变量

#### 返回值

返回布尔值表示value是否为安全整数

## polyfill

整数的概念我们都清楚，但是安全整数是啥？请拿出《计算机组成原理》然后不用入门直接放弃。最简单的理解就是限制在一定范围内的整数。

在```Number```对象上有个静态属性```MAX_SAFE_INTEGER```就是这个范围的边界。绝对值比这个值还大的数就不是安全数。


```javascript
Number.isSafeInteger = Number.isSafeInteger || function (value) {
   return Number.isInteger(value) && Math.abs(value) <= Number.MAX_SAFE_INTEGER;
};
```

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/isSafeInteger)提供的polyfill的思路是先检查是否为整数，然后看它是否在安全范围内。