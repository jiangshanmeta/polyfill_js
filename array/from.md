## 基本描述

#### 语法

```javascript
Array.from(arrayLike[, mapFn[, thisArg]])
```

#### 参数

* arrayLike 想转换成数组的类数组
* mapFn 可选，如果指定则返回经过map操作的数组
* thisArg 可选，map操作时需要绑定的this

#### 返回值

一个新的Array实例

## polyfill

首先说下我的思路：要实现这个feature可以分成两个子feature，一个是将类数组转换成数组，另一个是进行map操作。将类数组转换成数组直接使用```Array.prototype.slice.call```就好了，map操作可以直接使用Array自带的map方法。


```javascript
if(!Array.from){
	Array.from = (function(){
		var slice = Array.prototype.slice;
		return function(arrayLike,mapFunc,thisArg){
			var arr = slice.call(arrayLike);
			if(typeof mapFunc === 'function'){
				arr = thisArg?arr.map(mapFunc,thisArg):arr.map(mapFunc);
			}
			return arr;
		}
	})();
}
```

然而[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)上给出了一个超级复杂的polyfill


```javascript
if (!Array.from) {
  Array.from = (function () {
    var toStr = Object.prototype.toString;
    var isCallable = function (fn) {
      return typeof fn === 'function' || toStr.call(fn) === '[object Function]';
    };
    var toInteger = function (value) {
      var number = Number(value);
      if (isNaN(number)) { return 0; }
      if (number === 0 || !isFinite(number)) { return number; }
      return (number > 0 ? 1 : -1) * Math.floor(Math.abs(number));
    };
    var maxSafeInteger = Math.pow(2, 53) - 1;
    var toLength = function (value) {
      var len = toInteger(value);
      return Math.min(Math.max(len, 0), maxSafeInteger);
    };

    return function from(arrayLike/*, mapFn, thisArg */) {
      var C = this;
      var items = Object(arrayLike);

      if (arrayLike == null) {
        throw new TypeError("Array.from requires an array-like object - not null or undefined");
      }

      var mapFn = arguments.length > 1 ? arguments[1] : void undefined;
      var T;
      if (typeof mapFn !== 'undefined') {
        if (!isCallable(mapFn)) {
          throw new TypeError('Array.from: when provided, the second argument must be a function');
        }

        if (arguments.length > 2) {
          T = arguments[2];
        }
      }

      var len = toLength(items.length);

      var A = isCallable(C) ? Object(new C(len)) : new Array(len);

      var k = 0;

      var kValue;
      while (k < len) {
        kValue = items[k];
        if (mapFn) {
          A[k] = typeof T === 'undefined' ? mapFn(kValue, k) : mapFn.call(T, kValue, k);
        } else {
          A[k] = kValue;
        }
        k += 1;
      }

      A.length = len;

      return A;
    };
  }());
}
```

作者是手动进行了遍历，在遍历过程中做了map操作。一个让我不太理解的是```var A = isCallable(C) ? Object(new C(len)) : new Array(len);```，作者想在这里表达些什么，有知道的请劳驾告知，必有重谢。