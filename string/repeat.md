## 基本描述

#### 语法

```javascript
str.repeat(n);
```

#### 参数

* n 重复的次数，要求为正整数

#### 返回值

一个字符串，其值为str重复n次

## polyfill

看到这个方法，我首先联想到的是PHP的```str_repeat```，这两个方法其实做的是一样的，都是生成一个新的字符串，其值为原字符串重复n次。

要实现这个polyfill似乎并不是很难，首先规格化参数n，然后for循环，循环体执行n次。这个方案完全可行，但不是最佳的方案。

```javascript
if (!String.prototype.repeat) {
  String.prototype.repeat = function(count) {
    'use strict';
    if (this == null) {
      throw new TypeError('can\'t convert ' + this + ' to object');
    }
    var str = '' + this;
    count = +count;
    if (count != count) {
      count = 0;
    }
    if (count < 0) {
      throw new RangeError('repeat count must be non-negative');
    }
    if (count == Infinity) {
      throw new RangeError('repeat count must be less than infinity');
    }
    count = Math.floor(count);
    if (str.length == 0 || count == 0) {
      return '';
    }
    // 确保 count 是一个 31 位的整数。这样我们就可以使用如下优化的算法。
    // 当前（2014年8月），绝大多数浏览器都不能支持 1 << 28 长的字符串，所以：
    if (str.length * count >= 1 << 28) {
      throw new RangeError('repeat count must not overflow maximum string size');
    }
    var rpt = '';
    for (;;) {
      if ((count & 1) == 1) {
        rpt += str;
      }
      count >>>= 1;
      if (count == 0) {
        break;
      }
      str += str;
    }
    return rpt;
  }
}
```

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/repeat)上给出的方案还是用了一些技巧的。比如```count = +count;```就利用了加号将count转成了数值类型。比较复杂的是最后那个for循环，这个循环和我上面说的执行n次的循环做的是同一件事，但是循环次数却少很多。

这里需要一些基本的二进制的知识和位运算的知识。我决定采用举例说明，比如count为6，表示成二进制为110，因此对于6，我们可以看成是1*4 + 1*2 + 0*1。要生成最终的字符串，我需要0个单倍体基础str，一个双倍体基础str和一个四倍体基础str，生成双倍体、四倍体对应代码是```str += str;```，```(count & 1) == 1```就是用来判断需要几个N被体，请注意，count有效参与按位与运算的只有最后一位。```count >>>= 1;```是相关联的右移操作，保证count的最后一位是我们需要的系数位。这里比较绕，需要仔细琢磨一下。我提出的polyfill方案时间复杂度为O(n)，而MDN上的解决方案事件复杂度为O(logN)，因而MDN上的算法更加合适。