## 基本描述

#### 语法

```javascript
object.watch(prop, handler)
```

#### 参数

* prop 想要观察变化的属性
* handler prop变化时执行的回调，其返回值作为最终的属性值

## polyfill

这个方法目前是一个非标准方法，当这并不妨碍我们了解这个方法的polyfill。当一个属性值变化的时候，触发回调handler，handler传入三个参数，分别是属性名、属性的旧值、属性的新值，回调的返回值作为属性的值。乍看起来确实没什么思路，我们换个角度去看，一个属性值变化，意味着进行了赋值操作，在这时我们要拦截下来赋值操作，才能执行回调。拦截下来赋值操作，一个方案就是使用```Object.defineProperty```。这个API相关知识值得介绍一下。

对于一个属性，它有以下描述符：```configurable``` 可配置、```enumerable``` 可枚举、```writable``` 可写、```value``` 属性值、```get```与属性相关的getter方法、```set``` 与属性相关的setter方法。这些属性描述符不是同时存在的，我们日常使用的一组描述符是```configurable```、```enumerable```、```writable```、```value```。在这本电子书中，会见到一些polyfill就是使用这一组描述符进行定义的，主要目的就是让它不可枚举不可写，看起来就像是个原生的方法，下面的polyfill也是应用了这种方法。另一组描述符是```get```、```set```、```enumerable```、```configurable```，重点是前两个。```set```方法可以拦截赋值操作，这就是我们polyfill的关键了。

```javascript
if (!Object.prototype.watch) {
	Object.defineProperty(Object.prototype, "watch", {
		  enumerable: false,
		  configurable: true,
		  writable: false,
		  value: function (prop, handler) {
			var
			  oldval = this[prop],
			  newval = oldval,
			  getter = function () {
				return newval;
			  },
			  setter = function (val) {
				oldval = newval;
				return newval = handler.call(this, prop, oldval, val);
			  };

			if (delete this[prop]) { // can't watch constants
				Object.defineProperty(this, prop, {
					  get: getter,
					  set: setter,
					  enumerable: true,
					  configurable: true,
				});
			}
		}
	});
}
```

[github](https://gist.github.com/eligrey/384583)上的polyfill方案算是```Object.defineProperty```的综合运用了，两种使用方式都用到了。watch方法的polyfill基本思路是找一个变量```newval```存储值，当被赋值的时候，执行```setter```，在```setter```内部调用我们的回调。类似的用法在Vue里面有很多，以后有机会介绍吧。

关于这几个描述符再多说几句，有几个相关的方法```Object.getOwnPropertyDescriptor```、```Object.prototype.propertyIsEnumerable```，兼容性都还算可以。