## 基本描述

#### 语法

```javascript
arr.findIndex(callback[, thisArg])
```

#### 参数

* callback 针对数组中的每个元素执行的回调函数，执行时会传入以下参数
	* element 当前元素
	* index 当前元素的索引
	* array 调用callback的数组
* thisArg 可选，callback执行时的this指向。

#### 返回值


#### 返回值

arr中第一个满足callback的值的索引


## polyfill

这个方法相当于是加强版的```indexOf```，比较时不是通过```===```，而是通过用户自定义的一个回调函数。要实现这个功能思路也不复杂，手动遍历就好了。


```javascript
if (!Array.prototype.findIndex) {
  Array.prototype.findIndex = function(predicate) {
    if (this === null) {
      throw new TypeError('Array.prototype.findIndex called on null or undefined');
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
        return i;
      }
    }
    return -1;
  };
}
```

在underscore.js中也有相似的功能```_.findIndex```，实现的基本思路也是一致的。