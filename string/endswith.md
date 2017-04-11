## 基本描述

#### 语法

```javascript
str.endsWith(searchString [, position]);
```

#### 参数

* searchString 要搜索的子字符串
* position 可选，在str中搜索searchString的结束位置，默认值为str.length也就是真正的字符串结尾处

#### 返回值

布尔值，表示str是否以searchString为结尾

## polyfill

这个方法和```String.prototype.startsWith```的功能很像，所以不要重复造轮子，稍微改改以最小的代价把endsWith问题转换成startsWith就好了。

```javascript
if (!String.prototype.endsWith) {
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
    var endsWith = function(search) {
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
      var pos = stringLength;
      if (arguments.length > 1) {
        var position = arguments[1];
        if (position !== undefined) {
          pos = position ? Number(position) : 0;
          if (pos != pos) {
            pos = 0;
          }
        }
      }
      var end = Math.min(Math.max(pos, 0), stringLength);
      var start = end - searchLength;
      if (start < 0) {
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
      defineProperty(String.prototype, 'endsWith', {
        'value': endsWith,
        'configurable': true,
        'writable': true
      });
    } else {
      String.prototype.endsWith = endsWith;
    }
  }());
}
```