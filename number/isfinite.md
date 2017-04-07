## 基本描述

#### 语法

```javascript
Number.isFinite(value)
```

#### 参数

* value 需要被检测有穷性的值

#### 返回值

布尔值，表示给定的值是否为有穷

## polyfill

先说一下什么是有穷数，首先，要是一个数，然后，它必须有穷(不是NaN、不是Infinity)。

然后，我们可能会联想到```isFinite```这个方法，但是```isFinite```会把传入的参数强制转换成数值类型，因而会造成误判。所以之前判断是否是有穷数需要我们手动先判断是否是数值，然后再调用```isFinite```，```Number.isFinite```是把我们的这个行为进行了标准化。

```javascript
Number.isFinite = Number.isFinite || function(value) {
    return typeof value === "number" && isFinite(value);
}
```

上面的代码是[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/isFinite)上给出的polyfill