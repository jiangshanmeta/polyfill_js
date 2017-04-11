## 基本描述

#### 语法

```javascript
arr.includes(searchElement, fromIndex)
```

#### 参数

* searchElement 需要查找的元素值
* fromIndex  可选，查找的起始索引，如果为负值则从arr.length+fromIndex开始查找，默认为0

#### 返回值

布尔值，表示从fromIndex开始arr是否包含searchElement

## polyfill

看到这个方法我的第一反应是这是个语义化的```indexOf```，用```indexOf```判断的时候每次都要将返回值和-1比较，用起来真心不爽，偶尔就羡慕PHP家的```in_array```。jQuery虽说有个叫```inArray```的静态方法，但依然走的是```indexOf```的老路，依然需要我们把返回值和-1相比。

要实现这个方法的polyfill，最简单的思路就是包装一下```indexOf```方法

```javascript
if(!Array.prototype.includes){
	Array.prototype.includes = function(searchElement,fromIndex){
		return this.indexOf(searchElement,fromIndex) > -1;
	}	
}
```

对于一般的业务逻辑来说这个polyfill已经足够用了，然而文档说法是```indexOf```判断时用了```===```，对于```NaN```会造成误判，```includes```修复了这个问题。我个人认为这个就过于学院派了，正常使用的话其实出现```NaN```最可能的就是出bug了。不过解决这个也不难，[在这本电子书中也包含判断NaN的方法](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/number/isnan.html)，我们所要做的就是手动遍历，在判断的时候加个ifelse。


```javascript
if (!Array.prototype.includes) {
  Object.defineProperty(Array.prototype, 'includes', {
    value: function(searchElement, fromIndex) {
      if (this == null) {
        throw new TypeError('"this" is null or not defined');
      }
      var o = Object(this);
      var len = o.length >>> 0;
      if (len === 0) {
        return false;
      }
      var n = fromIndex | 0;
      var k = Math.max(n >= 0 ? n : len - Math.abs(n), 0);
      while (k < len) {
        if (o[k] === searchElement) {
          return true;
        }
        k++;
      }
      return false;
    }
  });
}
```

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)提供的polyfill并没有考虑```NaN```的问题，而是直接选择了```===```进行判断。代码的绝大部分都是在处理异常和规格化参数，比较难以理解的是```var len = o.length >>> 0;```，```>>>```这个符号说实话真的很少见人使用，毕竟是高大上的移位运算。这里感兴趣的请自行翻阅《计算机组成原理》相关教材，学习无符号数、有符号数、原码、反码、补码的基本概念，了解符号数值化、移位操作，顺带可以考虑看一下定点数和浮点数在计算机中的表示。当然最现实的方案是知道这里所做的就是规格化参数。