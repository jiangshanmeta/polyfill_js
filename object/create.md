## 基本描述

#### 语法

```javascript
Object.create(proto, [ propertiesObject ])
```

#### 参数

* proto 创建新对象的原型
* propertiesObject 可选,该参数对象是一组属性与值，该对象的属性名称将是新创建的对象的属性名称，值是属性描述符

#### 返回值

一个新的对象，其内部属性```[[prototype]]```指向proto

## polyfill

差不多10年前，有人提出了一种创建对象实现继承的新方式：


```
function create(o){
	function temp(){};
	temp.prototype = o;
	return new temp();
}
```

后来这个终于被标准化了，就有了我们的```Object.create```。

这个方法最主要的用途就是实现继承。稍微偏门但是实用的用途是创建一个真·空对象，```Object.create(null)```，熟悉原型链的你应该可以看出来，这样创建的新对象的```[[prototype]]```指向的是null，新对象没法访问到```Object.prototype```的方法。


```javascript
if (typeof Object.create != 'function') {
  Object.create = (function() {
    function Temp() {}

    var hasOwn = Object.prototype.hasOwnProperty;

    return function (O) {
      if (typeof O != 'object') {
        throw TypeError('Object prototype may only be an Object or null');
      }
      Temp.prototype = O;
      var obj = new Temp();
      Temp.prototype = null; 

      if (arguments.length > 1) {
        var Properties = Object(arguments[1]);
        for (var prop in Properties) {
          if (hasOwn.call(Properties, prop)) {
            obj[prop] = Properties[prop];
          }
        }
      }
      return obj;
    };
  })();
}
```