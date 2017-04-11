## 基本描述

#### 语法

```javascript
obj.unwatch(prop);
```

#### 参数

* prop 想要停止监视的属性名

## polyfill

这个方法和```Object.prototype.watch```是一家子，它的作用是取消对prop的监视，和watch正好想反。下面polyfill的思路也比并不复杂，首先把```getter setter```那一套直接使用```delete```干掉，然后正常赋值。


```javascript
if (!Object.prototype.unwatch) {
	Object.defineProperty(Object.prototype, "unwatch", {
		  enumerable: false
		, configurable: true
		, writable: false
		, value: function (prop) {
			var val = this[prop];
			delete this[prop]; // remove accessors
			this[prop] = val;
		}
	});
}
```