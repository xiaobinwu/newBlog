---
title: 你不知道的javascript(上卷)读后感（二）
date: 2019-08-06 15:48:27
tag: javascript
categories:
- javascript
---

### this词法
熟悉ES6语法的开发者，箭头函数在涉及this绑定时的行为和普通函数的行为完全不一致。跟普通this绑定规则不一样，它使用了当前的词法作用域覆盖了`this`本来的值。

### 误解
`this`理解成指向函数本身，函数对象的属性(Fun.X)不是this.X <b style="color: red">【X】 </b>
`this`指向函数的作用域 <b style="color: red">【X】 </b>
`this`是运行时进行绑定的，而不是编写时绑定的，它的执行上下文取决于函数调用时的各种条件，`this`的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式（调用位置），具有动态性，简单地说，函数执行过程中的调用位置决定了this的绑定对象。

### 何为执行上下文？
函数被调用时，会创建一个活动记录，这个也称作执行上下文，这个记录包含了函数在哪里被调用（调用栈）、函数调用方式、传入的参数等信息，而`this`就是执行上下文的一个属性，函数调用时会被用到。

### this绑定规则
> * 默认绑定，this指向全局对象（严格模式，不允许使用）
> * 隐式绑定，会把函数调用的this绑定到所在的上下文对象，对于隐式绑定，常常伴随着隐式丢失的问题，`函数别名`、`传入回调函数`、`函数传入内置函数`均会造成此问题
> * 显式绑定，就是常见`call`、`apply`方法，仍无法解决绑定丢失问题，但有个变种方法，叫做`硬绑定`
└── 硬绑定，函数bar中强制绑定（显示绑定）绑定foo在obj中执行
    ├── 包裹函数
    ├── 辅助函数（bind基本实现原理，下面有详细实现）
> * new绑定，过程如下（发生在构造函数调用时）：
1、创建（或者说构造）一个新对象
2、这个新对象会被执行[[Porototype]]连接
3、这个新对象会被绑定到函数调用的this
4、如果函数没有返回其他的对象，那么new表达式的函数调用会自动返回新对象

规则优先级：默认绑定 < 隐式绑定 < 显式绑定 < new绑定

☆☆☆☆☆☆ 辅助函数与bind函数的实现 ☆☆☆☆☆☆ 

```javascript
    function bind(fn, obj) {
        return function() {
            fn.apply(obj, arguments);
        }
    }
```
摘自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind):
```javascript
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
          // 关键代码，如果是new调用，就是使用新创建的this替换硬绑定的this，说明了new绑定优先级大于硬绑定
          return fToBind.apply(this instanceof fBound
                 ? this
                 : oThis,
                 // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    // 维护原型关系
    if (this.prototype) {
      fNOP.prototype = this.prototype; 
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```
### 更安全的this
`bind`辅助函数修改改变this的同时，可以使用`Object.create(null)`，如`.call/.apply(Object.create(null), ...argument)`，这样可以有效防止修改全局对象。

### 被忽略的this
null/undefined作为this的绑定对象传入`call`、`apply`，使用的是默认绑定规则。

### 软绑定
上述对硬绑定的介绍，会强制把this强制绑定到指定的对象上，防止了函数调用应用默认绑定规则，但是造成的问题就是，不够灵活，使用不了隐式绑定和显式绑定来修改this，于此同时，`软绑定`便出现了。
```javascript
if (!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
        var fn = this;
        // 捕获所有curried参数
        var curried = [].slice.call(arguments, 1);
        var bound = function() {
            return fn.apply(
                (!this || this === (window || global)) ? obj : this,
                curried.concat.apply(curried, arguments)
            );
        }
        bound.prototype = Object.create(fn.prototype);
        return bound;
    }
}
```
软绑定DEMO:
```javascript
function foo() {
    console.log("name:" + this.name);
}
var obj = { name: "obj" };
var obj2 = { name: "obj2" };
var obj3 = { name: "obj3" };
var fooBj = foo.softBind(obj);
fooBj(); // name: obj
obj2.foo = foo.softBind(obj2);
obj2.foo(); // name: obj2
fooBj.call(obj3); // name: obj3
setTimeout(obj2.foo, 10); // name: obj
```
### 基本类型、内置对象
基本类型：string、number、boolean、null、undefined、object
内置对象（也是内置函数）： String、Number、Boolean、Object、Function、Array、Date、RegExp、Error

日常开发，大家可能会有疑问，比如`var a = 'i am string'`中`a`变量，它可以使用`a.length、a.charAt(0)`等属性和方法。它本来是一个字符串，为什么会有类似对象的特性呢，这里会涉及到一个`基本包装对象`，可以怎么理解这样的一个概念性知识，一瞬间的引用类型，用完即毁，真正的引用类型会一直存在内存中，而基本包装对象只会存在一瞬间。

衍生方法：`typeof`、`instanceof`、`Object.prototype.toString.call`

### 对象
对象的内容是由一些存储在特定命名位置的值组成的，叫做属性（指针），有两种访问方式，`myObject['a']`、`myObject.a`，对象属性同时也是无序的。

ES6可计算属性名：
```javascript
const prefix = 'a';
const myObject = {
    [prefix + 'bar']: "xxxx"
}
```
记住，函数永远不属于一个对象，只是一个引用，只是函数的this，会因为隐性绑定，在上下文做一层绑定而已。

### 复制对象、属性描述符
> * 深复制，`JSON.parse(JSON.stringify(obj))`
> * 浅复制，`Object.assign(...)`，会遍历一个或多个源对象的所有可枚举的自由键到目标对象，源对象的一些特性（如writable）不会被复制

属性描述符，即`writable`、`configuable`、`enumerable`、`value`
获得属性描述符，`Object.getOwnPropertyDescriptor(myObject, 'a')`
设置属性描述符，`Object.defineProperty(myObject, "a", {...})`
`configuable`配置为false，不能使用delete关键字
`enumerable`控制属性是否可枚举

访问描述符，（Getter/Setter）,可以改写默认的`value`，
```javascript
var myObject = {
    get a() {
        return 2;
    }
}
myObject.a // 2
Object.defineProperty(myObject, "b", {
    get: function() {
        return this.a * 2 // 指向当前对象
    }
});
myObject.b // 4
```

### 遍历对象属性的几种方法
> * `for...in`，遍历对象自身及原型上可枚举的属性
> * `Object.keys()`，遍历对象自身可枚举的属性
> * `Object.getOwnPropertyNames`，遍历对象自身的属性
> * `Object.getOwnPropertySymbol`，遍历对象自身Symbol类型属性
> * `Reflect.ownkeys`，遍历对象自身的属性（包含不可枚举属性，Symbol类型属性）

### 对象不可变性
如果你希望对象属性或是对象是不可改变，可以通过配置对象的属性描述符来实现，但是这种不可变是一种浅不可变，如果对象中属性的引用是对象、数组、函数，那么它们是可变的，实现方式有如下：
> * 对象常量，`writable:false` | `configuable:false`
> * 禁止扩展，`Object.preventExtensions(obj)`
> * 密封，`Object.seal(obj)`调用禁止扩展，且不能重新配置，及删除属性，及`configuable:false`
> * 冻结，`Object.freeze(obj)`，在密封的基础上将`writable:false` 


### 存在性
> * in操作符，检查属性是否存在对象里或是[[Prototype]]原型链上
> * `Object.hasOwnProperty`，只会检查对象
问题：myObject.hasOwnProperty()可能会报错，myObject可能是`Object.create(null)`生成不带[[Prototype]]原型链，而`Object.hasOwnProperty`是由[[Prototype]]委托，所以可以这样，`Object.prototype.hasOwnProperty.call(myObject, "a")`

有坑：
```javascript
4 in [2,4,6] // false
4 in [2,2,6,8,0] // true
```
判断是否可枚举 `myObject.propertyIsEnumerable('a')`

### @@iterator迭代器对象
`for...of`被访问的对象请求一个迭代器对象，然后通过.next()方法遍历所有返回来的值
数组内置有`@@iterator`，所以可以直接使用`for...of`。
使用内置`@@iterator`遍历数组：
```javascript
var myArray = [1,2,3];
var it = myArray[Symbol.iterator](); // 返回迭代器函数
it.next(); // { value: 1, done: false }
it.next(); // { value: 2, done: false }
it.next(); // { value: 3, done: false }
it.next(); // { done: true }
```
普通对象不含有`@@iterator`，无法使用`for...of`，可以进行改造，
```javascript
var myObject = { a: 2, b: 3 };
Object.defineProperty(myObject, Symbol.iterator, {
    enumerable: false,
    writable: false,
    configuabale: false,
    value: function() {
        var o = this;
        var idx = 0;
        var ks = Object.keys(o);
        return {
            next: function() {
                return {
                    value: o[ks[idx++]],
                    done: (idx > ks.length)
                };
            }
        }
    }
});
vat it = myObject[Symbol.iterator]();
it.next(); // { value: 2, done: false }
it.next(); // { value: 3, done: false }
it.next(); // { done: true }

```