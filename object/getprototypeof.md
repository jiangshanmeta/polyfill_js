## 基本描述

#### 语法

```javascript
Object.getPrototypeOf(obj);
```

#### 参数

* obj 要返回原型的对象

#### 返回值

给定对象的原型

## polyfill

我在讲[Object.values的polyfill](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/object/values.html)提到过，一个对象会有一个```[[prototype]]```指向构造函数的原型对象，```Object.getPrototypeOf```就是用来返回这个原型对象的。

其实这个方法并没有什么特别好的polyfill，在出现这个API以前，大家会通过```__proto__```这个非标准属性获得原型对象。不过考虑到IE9就已经支持这个方法了，大家都懂。

```javascript
if(!Object.getPrototypeOf){
	Object.getPrototypeOf = function(obj){
		return obj.__proto__;
	}
}
```