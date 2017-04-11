## 基本描述

#### 语法

```javascript
str.startsWith(searchString [, position]);
```

#### 参数

* searchString 要搜索的子字符串
* position 可选，搜索的起始位置，默认为0

#### 返回值

布尔值，表示str是否以searchString为起始


## polyfill

要判断str是否以searchString为开始，最偷懒的做法是调用indexOf然后判断结果是否为0，可行是可行的但其中做了大量的无用功，所以性能很差。那就只能一位一位比较了，这对应下面[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/startsWith)给出的polyfill中的那个while循环。然而绝大部分代码是用来处理查询的开始位置的。写业务逻辑写多了就会对这种核心代码没几行大部分都在预处理习惯了。


```javascript
if (!String.prototype.startsWith) {
  (function() {
    'use strict'; 
    var defineProperty = (function() {
      try {
        var object = {};
        var $defineProperty = Object.defineProperty;
        var result = $defineProperty(object, object, object) && $defineProperty;
      } catch(error) {}
      return result;
    }());
    var toString = {}.toString;
    var startsWith = function(search) {
      if (this == null) {
        throw TypeError();
      }
      var string = String(this);
      if (search && toString.call(search) == '[object RegExp]') {
        throw TypeError();
      }
      var stringLength = string.length;
      var searchString = String(search);
      var searchLength = searchString.length;
      var position = arguments.length > 1 ? arguments[1] : undefined;
      var pos = position ? Number(position) : 0;
      if (pos != pos) { 
        pos = 0;
      }
      var start = Math.min(Math.max(pos, 0), stringLength);
      if (searchLength + start > stringLength) {
        return false;
      }
      var index = -1;
      while (++index < searchLength) {
        if (string.charCodeAt(start + index) != searchString.charCodeAt(index)) {
          return false;
        }
      }
      return true;
    };
    if (defineProperty) {
      defineProperty(String.prototype, 'startsWith', {
        'value': startsWith,
        'configurable': true,
        'writable': true
      });
    } else {
      String.prototype.startsWith = startsWith;
    }
  }());
}
```