Object.isExtensible 方法是用来判断是否可以再对象上添加新属性的，默认情况下可以添加新属性，而 Object.preventExtensions 、Object.seal 以及 Object.freeze 都可以标记一个对象为不可扩展的。

严格来说并没有什么直接的办法可以在es5环境下模拟 Object.isExtensible，那怎么办呢？core-js的思路是这样的：当没有原生的Object.isExtensible方法时，Object.preventExtensions 、Object.seal 以及 Object.freeze这些方法都不存在，即我们无法让一个对象不可扩展，而当有原生的方法时，我们就直接用原生的好了。


```javascript
require('../internals/object-statics-accept-primitives')('isExtensible', function (nativeIsExtensible) {
  return function isExtensible(it) {
    return isObject(it) ? nativeIsExtensible ? nativeIsExtensible(it) : true : false;
  };
});
```