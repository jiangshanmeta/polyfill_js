## 基本描述

#### 语法

```javascript
Object.is(value1, value2);
```

#### 参数

* value1 需要比较的第一个值
* value2 需要比较的第二个值

#### 返回值

返回布尔值，表示value1和value2是否相等

## polyfill

比较两个变量是否相等，这个直接用```===```不行吗？绝大部分情况是可行的，但是有例外。最容易想出来的例外是**NaN**，毕竟NaN不等于NaN。这个情况写个特殊的ifelse没啥难度。

还有一个特殊情况是+0和-0，```===```认为+0和-0是相等的。我们如何区分+0和-0呢？我们可以利用```+Infinity```和```-Infinity```不一致这个特性。

```javascript
if (!Object.is) {
  Object.is = function(x, y) {
    if (x === y) { 
      return x !== 0 || 1 / x === 1 / y;
    } else {
      return x !== x && y !== y;
    }
  };
}
```

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is)提供的polyfill就是对上面描述的实现。

默默觉得这个函数应用价值不大。