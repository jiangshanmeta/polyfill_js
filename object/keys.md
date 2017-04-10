## 基本描述

#### 语法

```javascript
Object.keys(obj)
```

#### 参数

* obj 要返回自身可枚举属性的对象

#### 返回值

一个表示obj自身可枚举属性的字符串数组

## polyfill

要获取一个对象**可枚举**的属性，可以使用```for in```，要判断是否是**自身**的属性，可以使用```hasOwnProperty```。这样核心功能的原理就介绍完了。


下面代码依然出自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)

```javascript
if (!Object.keys) {
  Object.keys = (function () {
    var hasOwnProperty = Object.prototype.hasOwnProperty,
        hasDontEnumBug = !({toString: null}).propertyIsEnumerable('toString'),
        dontEnums = [
          'toString',
          'toLocaleString',
          'valueOf',
          'hasOwnProperty',
          'isPrototypeOf',
          'propertyIsEnumerable',
          'constructor'
        ],
        dontEnumsLength = dontEnums.length;

    return function (obj) {
      if (typeof obj !== 'object' && typeof obj !== 'function' || obj === null) throw new TypeError('Object.keys called on non-object');

      var result = [];

      for (var prop in obj) {
        if (hasOwnProperty.call(obj, prop)) result.push(prop);
      }

      if (hasDontEnumBug) {
        for (var i=0; i < dontEnumsLength; i++) {
          if (hasOwnProperty.call(obj, dontEnums[i])) result.push(dontEnums[i]);
        }
      }
      return result;
    }
  })();
};
```

你可能会好奇，为什么用```hasOwnProperty.call(obj, prop)```，用```obj.hasOwnProperty(prop)```不行吗？其实对于绝大部分情况下没问题，但是如果是```Object.create(null)```创建出来的对象，它是无法访问到Object.prototype上的方法的，换句话说在这种情况下使用```obj.hasOwnProperty```会报错。

上面代码的绝大部分是处理某些浏览器的bug的，在underscore.js有类似代码，作者加了一行这样的注释：

> Keys in IE < 9 that won't be iterated by `for key in ...` and thus missed.

都懂的。

这里的立即执行的匿名函数表达式、闭包、模拟静态私有变量就不说了，都是日常了。

一个相关的方法是```Object.getOwnPropertyNames()```，他返回的是一个对象自身属性的属性名(包括不可枚举属性)，这个不可枚举就比较尴尬了，我没有办法通过```for in```来获取属性名，所以这个方法不能被polyfill。


以上