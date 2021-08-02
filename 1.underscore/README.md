## underscore的源码阅读

### [官网地址](http://underscorejs.org/)

### [Github](https://github.com/jashkenas/underscore)

### 1、_定义及导出
```js
(function (global, factory) {
  typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
  typeof define === 'function' && define.amd ? define('underscore', factory) :
  (global = typeof globalThis !== 'undefined' ? globalThis : global || self, (function () {
    var current = global._;
    var exports = global._ = factory();
    exports.noConflict = function () { global._ = current; return exports; };
  }()));
}(this, (function () {
  //     Underscore.js 1.13.1
  //     https://underscorejs.org
  //     (c) 2009-2021 Jeremy Ashkenas, Julian Gonggrijp, and DocumentCloud and Investigative Reporters & Editors
  //     Underscore may be freely distributed under the MIT license.
 
  // Current version.
  var VERSION = '1.13.1';
 
  function _$1(obj) {
    // 如果obj已经是underscore对象，直接返回obj
    if (obj instanceof _$1) return obj;
    // 如果不是使用new来创建underscore对象：使用new 方式创新调用构造函数
    if (!(this instanceof _$1)) return new _$1(obj);
    // 添加_wrapped属性保存obj
    this._wrapped = obj;
  }
 
  // Add your own custom functions to the Underscore object.
  function mixin(obj) {
    // Array.forEach
    each(functions(obj), function(name) {
      var func = _$1[name] = obj[name];
      _$1.prototype[name] = function() {
        var args = [this._wrapped];
        // 将所有参数组成数据： [this._wrapped, ...arguments]
        push.apply(args, arguments);
        // apply执行函数：所有函数的第一个参数都是this._wrapped值
        return chainResult(this, func.apply(_$1, args));
      };
    });
    return _$1;
  }
 
  var allExports = {
    __proto__: null,
    VERSION: VERSION,
    chunk: chunk,
    mixin: mixin,
    // ...
    'default': _$1
  };
 
  // Default Export
 
  // Add all of the Underscore functions to the wrapper object.
  // 将所有需要向外暴露的封装方法包装到 underscore的prototype上
  var _ = mixin(allExports);
  // Legacy Node.js API.
  _._ = _;
 
  return _;
})));

```

### 2、方法混入及扩展
在**mixin**中，将所有待导出的方法都挂载到prototype属性上，并重新封装调用细节：
1. 将实际调用时的参数和实例化时的参数合并： **[this._wrapped, ...arguments]**；
2. 使用*func.apply(_$1, args)*调用原方法；
3. 使用检测chainResult检测实例对象和调用返回具体的值
4. 在实例化对象上也可以调用mixin来添加扩展

```js
// 1、定义所有需要导入的方法
var allExports = {
    __proto__: null,
    VERSION: VERSION,
    chunk: chunk,
    mixin: mixin,
    // ...
    'default': _$1
};

// Add all of the Underscore functions to the wrapper object.
// 2、将所有需要向外暴露的封装方法包装到 underscore的prototype上
var _ = mixin(allExports);
// Legacy Node.js API.
_._ = _;
```

### 3、 链式调用处理
在underscore中，开启链式调用的方法如下：
```js
// .value() 是手动终止链式调用的操作
_([1, 1, 2, 3, 3]).chain().map(item => item * 2).unique().value()
```
#### 3.1、chain包装
```js
// Start chaining a wrapped Underscore object.
// 开启underscore的链式调用：创建一个新的underscore对象，再添加一个_chain属性
function chain(obj) {
    var instance = _$1(obj);
    instance._chain = true;
    return instance;
}
```

#### 3.2、chain包装
```js
// Start chaining a wrapped Underscore object.
// 开启underscore的链式调用：创建一个新的underscore对象，再添加一个_chain属性
function chain(obj) {
    var instance = _$1(obj);
    instance._chain = true;
    return instance;
}
```

### 3.2、检测运行结果
运行参数说明：
1. *instance*是当underscore方法运行时的this指向(当前运行环境的underscore实例)
2. *obj*是调用underscore方法所返回的结果

首次运行并调用chain方法时，instance还是一个普通的underscore对象，而obj才是一个chained underscore对象。在chain后续的调用方法中，instance每次都是新的实例化运行结果的chained underscore对象。
```js
// Helper function to continue chaining intermediate results.

function chainResult(instance, obj) {
    // 如果instance是chain包装过的，则将结果再次包装为新的链式调用对象，否则直接返回调用结果
    return instance._chain ? _$1(obj).chain() : obj;
}
```

### 3.3、终止chain
实现方法是通过直接返回underscore对象的_wrapped属性值，从而终止链式链式调用。
```js
// Extracts the result from a wrapped and chained object.
// 解析出经过链式调用中包装过的value值
_$1.prototype.value = function() {
    return this._wrapped;
};
```