## 基本描述

#### 语法

```javascript
arr.fill(value, start, end)
```

#### 参数

* value 用来填充数组的值
* start 可选，填充的开始位置(含)
* end 可选，填充的结束位置


#### 返回值

填充后的数组


## polyfill

我其实很想知道这个方法是处于什么考虑而设计出来的，我完全想象不出来这个方法的实际用途。关于这个方法的polyfill，在预处理好始末位置之后，只要循环赋值就可以了。


```javascript
if (!Array.prototype.fill) {
  Array.prototype.fill = function(value) {
    if (this == null) {
      throw new TypeError('this is null or not defined');
    }

    var O = Object(this);
    var len = O.length >>> 0;
    var start = arguments[1];
    var relativeStart = start >> 0;
    var k = relativeStart < 0 ?
      Math.max(len + relativeStart, 0) :
      Math.min(relativeStart, len);

    var end = arguments[2];
    var relativeEnd = end === undefined ?
      len : 
      end >> 0;

    var final = relativeEnd < 0 ?
      Math.max(len + relativeEnd, 0) :
      Math.min(relativeEnd, len);

    while(k < final) {
        O[k] = value;
        k++;
    }

    return O;
  };
}
```

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)给出的实现有大量的代码是在对始末位置进行预处理，为了性能上的优化还使用了位运算符处理。