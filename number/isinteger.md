## 基本描述

#### 语法

Number.isInteger(value)

#### 参数

* value为要检验是否为整数的值

#### 返回值

返回布尔值，表示value是否为整数

## polyfill

判断整数这个话题，我们先拿出《计算机组成原理》，然后直接放弃就好了。

这种古老的话题居然要到es6才被规范下来我也只能无语了。

首先我们看下[*php.js*是如何实现](http://locutus.io/php/var/is_int/)的：

```javascript
return mixedVar === +mixedVar && isFinite(mixedVar) && !(mixedVar % 1);
```

核心代码就一行但是知识量还是挺丰富的。第一个判断```mixedVar === +mixedVar```中的加号会强制将类型转换成数值，通过这个判断保证了数据类型为数值并且不是NaN才向下进行判断。第二个判断```isFinite(mixedVar)```在第一步的基础上对是否为无穷```Infinity```进行了判断，这样剩下的就是有穷数了。最后一个判断通过模1判断是整数还是浮点数。

```javascript
Number.isInteger = Number.isInteger || function(value) {
    return typeof value === "number" && 
           isFinite(value) && 
           Math.floor(value) === value;
};
```

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/isInteger)给出的方案类似，前两步限制数据类型限制有穷数，最后一步通过比较取整结果和原数是否相同判断是整数还是浮点数。

其实我在想，为什么就没有```Number.isFloat```这个方法呢？