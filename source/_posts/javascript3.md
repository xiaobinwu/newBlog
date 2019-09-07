---
title: 你不知道的javascript(上卷)读后感（三）
date: 2019-08-17 22:29:27
tag: javascript
categories:
- javascript
---

### 类
传统语言，构造函数是属于类的，而Javascript，类是属于构造函数的，如（FOO.Prototype）  
父类和子类的关系只存在于两者构造函数对应的prototype对象中。

### 显示混入
```javascript
function mixin(sourceObj, targetObj) {
    for(var key in sourceObj) {
        if (!(key in targetObj)) {
            targetObj[key] = sourceObj[key];
        }
    }
    return targetObj;
}
var Vehicle = {
    engines: 1,
    ignition: function() {
        console.log('xxx');
    },
    drive: function() {
        this.ignition();
        console.log('xxx');
    }
};
var Car = mixin(Vehicle, {
    wheels: 4,
    drive: function() {
        Vehicle.drive.call(this);
        console.log('xxx'); // 显示伪多态
    }
});
```
### 寄生继承
```javascript
function Vehicle() {
    this.egines = 1;
}
Vehicle.prototype.ignition = function() {
    console.log('xxx');
}
Vehicle.prototype.drive = function() {
    this.ignition();
    console.log('xxx');
}
function Car() {
    var car = new Vehicle();
    car.wheel = 4;
    var carDrive = car.drive;
    car.drive = function() {
        carDrive.call(this);
        console.log('xxx');
    }
    return car;
}
var myCar = new Car();
myCar.drive();
```
### 注意
`new Foo()`，创建新对象，关联到其他对象的新对象，并不会去复制对象，只是关联。  
`Foo.prototype.construtor === Foo` => `true`  
`new` 会劫持所有普通函数并构造新对象的形式来调用  
`a = new Foo()` / `a.construtor === Foo` => `true`  
不代表`a`里面有`construtor`属性，实际同样是委托给`Foo.prototype`。而`Foo.prototype.construtor`指向了`Foo`，重写`Foo.prototype`,那默认便不会指向`construtor`，而是`Object`，因为委托给了原型链的顶端`Object.prototype`  
> * 判断是否为某个实例的原型，`Foo.prototype.isPrototypeOf(a)`
> * 判断是否为某个实例的原型，`Foo.prototype === Object.getPrototypeOf(a)`
> * `a.__proto__ === Foo.prototype`
与`construtor`、`isPrototypeOf`一样，不存在实例`a`里面，存在内置的`Object.prototype `

### __proto__内部实现
```javascript 
Object.defineProperty(Object.prototype, '__proto__', {
    get: function() {
        return Object.getPrototypeOf(this);
    },
    set: function(O) {
        Object.setPrototypeOf(this, o);
    }
})
```


### 原型链
一个对象没有找到需要的属性和方法引用，引擎就会在该对象[[Prototype]]关联的对象查找，以此类推，形成原型链。

### Object.create()
[[Prototype]]存在于对象中的一个内部链接，本质是对象之间的关系。
`Object.create()`创建一个新对象并把它关联到指定的对象
`Object.create(null)`无原型链  
#### polyfill
```javascript
if (!Object.create) {
    Object.create = function(o) {
        function F() {}
        F.prototype = o;
        return new F();
    }
}
```
`Object.create`还存在着第二个参数，指定要添加到新对象中的属性名以及属性描述符。

### 委托理论
```javascript
Task = {
    setId: function(id) {
        this.id = id;
    },
    outputId: function() {
        console.log(this.id);
    }
}
XYZ = Object.create(Task);
XYZ.set = function(id, label) {
    this.setId(id);
    this.label = label;
}
```
### 委托设计模式，也称为对象关联
#### 常见（“原型”）面向对象风格
```javascript
function Foo(who) {
    this.me = who;
}
Foo.prototype.identify = function() {
    return `I am ${this.me}`;
}
function Bar(who, age) {
    this.age = age;
    Foo.call(this, who);
}
Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.speak = function() { return `I ${this.age}`; }
```
#### 委托（对象关系）
```javascript
Foo = {
    init: function(who) {
        this.me = who;
    },
    indentify: function() {
        return `I am ${this.me}`
    }
};
Bar = Object.create(Foo);
Bar.speak = function(age) {
    return `I ${this.age}`; 
}
var b1 = Obejct.create(Bar);
b1.init('b1');
```
### 反词法
```javascript
var Foo = {
    // 匿名函数表达式
    bar() {
        ...
    }
    // 等价于
    // bar: function() { ... }
    // 具名函数表达式
    baz: function baz() { ... }
}
```
匿名函数没有`name`标识符，会导致
> * 1、调用栈难追踪
> * 2、自我引用难
> * 3、代码难理解

tip： ES6 class中的`super`不是动态绑定（晚绑定），创建时就能静态绑定