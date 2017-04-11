## 基本描述

#### 语法

```javascript
Array.of(element0[, element1[, ...[, elementN]]])
```

#### 参数

* elementN 任意个参数，将按顺序成为返回数组中的元素。

#### 返回值

返回一个新的Array实例

## polyfill

要处理任意个参数，显然需要```arguments```这个类数组，要将类数组转换为真数组，需要```Array.prototype.slice```这个方法。这是最简单的实现思路了。

```javascript
if (!Array.of) {
  Array.of = function() {
    return Array.prototype.slice.call(arguments);
  };
}
```

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/of)上给出了这个简短的方案，并给了一个链接，内容是一个超长的方案，那里正好还有```Array.from```的超长polyfill，请看[Array.from](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/array/from.html)的相关内容。