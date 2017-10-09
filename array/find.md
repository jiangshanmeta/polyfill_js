## 基本描述

#### 语法

```javascript
arr.find(callback[, thisArg])
```

#### 参数

* callback 在数组每一项上执行的回调函数，执行时会传入一下参数
	* element 当前元素
	* index 当前元素的索引
	* array 调用find方法的数组本身
* thisArg 可选，callback执行时的this指向


#### 返回值

arr中第一个满足callback的元素的值


## polyfill

这个和[Array.prototype.findIndex](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/array/findindex.html)是一家，后者返回键，前者返回值。一个polyfill方案是在```Array.prototype.findIndex```的基础上包一层，另一个方案是把```findIndex```的polyfill改一下，把返回键改成返回值就好了。


[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find)采用的第二个方案。

```javascript
if (!Array.prototype.find) {
  Array.prototype.find = function(predicate) {
    'use strict';
    if (this == null) {
      throw new TypeError('Array.prototype.find called on null or undefined');
    }
    if (typeof predicate !== 'function') {
      throw new TypeError('predicate must be a function');
    }
    var list = Object(this);
    var length = list.length >>> 0;
    var thisArg = arguments[1];
    var value;

    for (var i = 0; i < length; i++) {
      value = list[i];
      if (predicate.call(thisArg, value, i, list)) {
        return value;
      }
    }
    return undefined;
  };
}
```