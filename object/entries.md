## 基本描述

#### 语法

```javascript
Object.entries(obj)
```

#### 参数

* obj 想返回由可枚举属性名和对应属性值组成的的键值对的对象

#### 返回值

一个给定对象**自己**的**枚举**属性[key，value]对的数组

## polyfill

之前提过```Object.keys```和```Object.values```，加上这个```Object.entries```，可以组成一个全家桶了，一个返回键，一个返回值，一个返回键值组成的数组。说到这里它的polyfill思路很明确了。

```javascript
if(!Object.entries){
	Object.entries = function(obj){
		obj = Object(obj);
		var keys = Object.keys(obj);
		var length = keys.length;
		var rst = [];
		for(var i=0;i< length;i++){
			rst[i] = [keys[i],obj[keys[i]]];
		}
		return rst;
	}
}
```

这个实现思路和underscore.js中的```_.pairs```是一个思路。


[MDN上提供的一个github链接](https://github.com/tc39/proposal-object-values-entries/blob/master/polyfill.js)，这个方案的思路和```Object.values```的思路是一致的，需要说明的点之前已经说过了。

```javascript
const reduce = Function.bind.call(Function.call, Array.prototype.reduce);
const isEnumerable = Function.bind.call(Function.call, Object.prototype.propertyIsEnumerable);
const concat = Function.bind.call(Function.call, Array.prototype.concat);
const keys = Reflect.ownKeys;
if (!Object.entries) {
	Object.entries = function entries(O) {
		return reduce(keys(O), (e, k) => concat(e, typeof k === 'string' && isEnumerable(O, k) ? [[k, O[k]]] : []), []);
	};
}
```