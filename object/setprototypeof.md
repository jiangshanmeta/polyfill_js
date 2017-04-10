## 基本描述

#### 语法

```javascript
Object.setPrototypeOf(obj,prototype);
```

#### 参数

* obj 要设定原型的对象
* prototype 新的原型对象

## polyfill

与[Object.getPrototypeOf](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/object/getprototypeof.html)类似，这个方法也没有很好的办法去polyfill，只能根据非标准的```__proto__```去进行操作。

```javascript
Object.setPrototypeOf = Object.setPrototypeOf || function (obj, proto) {
  obj.__proto__ = proto;
  return obj; 
}
```

根据文档的说法，这个操作性能并不好，除非有特殊的需求不要试图进行这个操作。

在Vue源码里我们会看到一个设定```__proto__```的例子。以后有机会再细说吧。