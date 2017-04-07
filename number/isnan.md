## 基本描述

#### 语法

```javascript
Number.isNaN(value)
```

#### 参数

* value 要被检测是否为**NaN**的值

#### 返回值

布尔值，表示给定的值是否为NaN

## polyfill

看到这个静态方法我们可以很轻松想起全局方法```isNaN```，但是```isNaN```有个问题，它会先把参数强转成为数值类型，然后才判断是否为**NaN**，因而会造成误判。所以以往要检验是否为**NaN**，需要先判断数值类型，然后再调用全局函数```isNaN```。```Number.isNaN```相当于是把我们的这种判断方式进行了标准化。


```javascript
Number.isNaN = Number.isNaN || function(value) {
    return typeof value === "number" && isNaN(value);
}
```

上面的代码是[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/isNaN)提供的一个polyfill。有了上面的描述这段polyfill不难理解了吧。


但是我并不是很认同这段polyfill。因为判断**NaN**还有一个方法，这个方法利用了**NaN**不等于**NaN**，即一个值如果不等于它本身，那么一定是**NaN**。

```javascript
if(!Number.isNaN){
	Number.isNaN = function(value){
		return value !== value;
	}
}
```