## 基本描述

#### 语法

```javascript
str.includes(searchString[, position])
```

#### 参数

* searchString 在str中需要搜索的字符串
* position 可选，搜索的起始位置，默认为0


#### 返回值

布尔值，表示是否查询到子串

## polyfill

看到这个方法，很容易联想到```Array.prototype.includes```，后者是语义化的```Array.prototype.indexOf```，```String.prototype.includes```是语义化的```String.prototype.indexOf```。所以这个方法polyfill的核心思路就是在```indexOf```进行一层包装。


```javascript
if (!String.prototype.includes) {
  String.prototype.includes = function(search, start) {
    'use strict';
    if (typeof start !== 'number') {
      start = 0;
    }
    if (start + search.length > this.length) {
      return false;
    } 
    return this.indexOf(search, start) !== -1;
  };
}
```