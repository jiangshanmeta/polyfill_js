## 基本描述

#### 语法

```javascript
Object.values(obj)
```

#### 参数

* obj 想要返回自身可枚举属性值的对象

#### 返回值

一个数组，数组的值由obj的自身可枚举属性值组成。

## polyfill

这个方法返回一个对象自身可枚举属性值，看到这个描述会不会想起```Object.keys```的描述，前者返回的是值，后者返回的是键。那为啥后者早就成为规范了前者还在草案阶段。言归正传，想要对```Object.values```进行polyfill，最简单的方案就是把```Object.keys```的polyfill拿过来改改就行了。但是那段代码有点长，我们可以直接利用```Object.keys```的结果。


```javascript
if(!Object.values){
	Object.values = function(obj){
		obj = Object(obj);
		var keys = Object.keys(obj);
		var length = keys.length;
		var values = [];
		for(var i=0;i< length;i++){
			values[i] = obj[keys[i]];
		}
		return values;
	}
}
```

在underscore.js中，```_.values```也是使用了相同的思路。


[MDN上提供的一个github链接](https://github.com/tc39/proposal-object-values-entries/blob/master/polyfill.js)


```javascript
const reduce = Function.bind.call(Function.call, Array.prototype.reduce);
const isEnumerable = Function.bind.call(Function.call, Object.prototype.propertyIsEnumerable);
const concat = Function.bind.call(Function.call, Array.prototype.concat);
const keys = Reflect.ownKeys;

if (!Object.values) {
	Object.values = function values(O) {
		return reduce(keys(O), (v, k) => concat(v, typeof k === 'string' && isEnumerable(O, k) ? [O[k]] : []), []);
	};
}
```

虽然这个polyfill我不是很推荐但是其中包含的知识量还是挺丰富的。

首先这是ES6的语法，我准备转述一下

```javascript
if(!Object.values){
	Object.values = (function(){
		var reduce = Function.bind.call(Function.call, Array.prototype.reduce);
		//var reduce = Function.call.bind(Array.prototype.reduce);
		var isEnumerable = Function.bind.call(Function.call, Object.prototype.propertyIsEnumerable);
		var concat = Function.bind.call(Function.call, Array.prototype.concat);
		var keys = Reflect.ownKeys;
		return function(obj){
			return reduce(keys(obj),function(v,k){
				return concat(v,typeof k === 'string' && isEnumerable(obj,k)?[obj[k]]:[]    );
			},[]);
		}
	})();
}
```

第一个问题，为什么会有```Function.bind```，```bind```方法不是在```Function.prototype```上吗？这个问题还是蛮复杂的。首先，一个正常的函数，它有一个```prototype```属性指向一个对象。其次，一个对象(一般情况蛤)，它会有一个内部属性```[[prototype]]```，指向这个对象的构造函数的```prototype```。最后，```Function```即是一个函数也是一个对象，它的```prototype```和```[[prototype]]```指向的是同一个对象。```Function.bind```是把```Function```看成是一个对象，通过原型链访问到了```bind```方法。

第二个问题，```bind```方法在这里起到了什么作用？答案是制造偏函数。

第三个问题，两个```call```起到了什么作用？```Function.bind.call```这个call我不太理解，因为```Function.call```本来就可以访问到```bind```方法，所以可以直接写成```var reduce = Function.call.bind(Array.prototype.reduce);```。```Function.call```的call被转移到了偏函数中，以```reduce```为例，调用的时候相当于```Array.prototype.reduce.call```，就是为了不在返回的匿名函数中显示调用call所使用的语法糖。感觉炫技的成分更多一点，这个使用方法可以学一学。

第四个问题是```Reflect.ownKeys```，我们暂时可以把它当成```Object.getOwnPropertyNames```。

最后的问题就是reduce实现了，其实这里用each之类的都可以，实现上并没有什么太复杂的，自己看吧。