---
title: 你不知道的javascript(上卷)读后感（一）
date: 2019-08-04 12:44:27
tag: javascript
categories:
- javascript
---

### 三剑客

编译，顾名思义，就是源代码执行前会经历的过程，分三个步骤，
> * 分词／词法分析，将我们写的代码字符串分解成多个词法单元
> * 解析／语法分析，将词法单元集合生成抽象语法树（AST）
> * 代码生成，抽象语法树（AST）转换成可执行代码的过程

`Tip1`：js在语法分析和代码生成阶段有对运行性能进行优化，对冗余元素进行优化
`Tip2`：js的编译过程不是发生在构建之前，而是代码执行之前

理解作用域，首先知道三剑客，分别是
> * 引擎：负责整个代码编译及执行的过程
> * 编译器： 负责词法分析、语法分析、代码生成
> * 作用域：负责维护与收集所有声明的标识符，保证当前执行代码对这些标识符的访问权限

举例子，加深印象，对于`var a = 2`，三剑客如何协同工作，
编译器进行分词、语法分析，然后要代码生成时，遇到 `var a`，问一下当前作用域集合“你有没有这个名称的变量呀？”，作用域如果说有，那么忽略声明，继续编译，如果说没有，那么就要要求作用域收集一下，并且给它个名字`a`,然后编译器就生成了代码，引擎准备来执行了，先问当前作用域集合，“你这里有a这个变量吗？”，有引擎就拿来用，没有就继续找该变量，要么找到，就给它附个值`2`，没有那就给你报个错！

### 理解编译器的相关术语
> * LSH查询，通俗解释就是找到所声明变量，并且对其赋值的行为
> * RSH查询，通俗解释就是查找声明的变量

### 作用域嵌套
当一个块或是函数嵌套在另外一个块或函数时，就会产生作用域嵌套，于是在当前作用域找不到某个变量时，引擎会往外层嵌套作用域继续查找，直达到最外层作用域（全局作用域）为止，也就是所谓的作用域链啦！

### 词法作用域

### 相信有很多人都是搞不懂词法作用域是什么？
所谓的词法作用域，就是定义在词法阶段（词法分析）的作用域，由你写代码时将变量和块作用域写在哪里来决定的。

### 遮蔽效应
作用域查找会在找到第一个匹配的标识符时停止，不会继续往上层作用域查找，这就会产生`遮蔽效应`。

### 欺骗词法作用域
> * eval函数，修改词法作用域
> * with关键字，创建词法作用域

导致性能下降的原因，前面我们提过，在解析/语法分析、生成代码阶段，我们会对代码进行优化，剔除冗余元素，但是当使用eval/with时，所有优化变得没有意义，因为存在不可预见性，不知道修改和创建的词法作用域是什么？所以会导致性能下降。

### 函数声明、函数表达式、匿名函数表达式
函数表达式：`function`为第一个词，那么就是一个函数声明，否则就是一个函数表达式
匿名函数表达式：没有函数名，匿名函数在栈追踪这种不会显示有意义的函数名，使得调试困难
IIFE: 立即执行函数表达式（`(function() { ... }())`、`(function(){ ... })()`）

### 块级作用域
`let`为其声明的变量隐式地去劫持了所在的块级作用域，不会在块级作用域中进行提升【变量提升】
`Demo:` with关键字为块级作用域、{...}为块级作用域，用完即销毁
`const`常量，不可修改！
任何声明在某个作用域（函数作用域和块级作用域）的变量，都是属于这个作用域。
每个作用域都会进行提升操作。
函数声明会被提升，函数表达式不会提升，变量声明提升的过程中，函数会优先！

### 闭包
闭包，有权访问另外一个函数的变量标识符的函数，比较常见的一个闭包问题，就是for循环。
```javascript
    for(var i = 1; i <= 5; i++) {
        setTimeout(function() {
            console.log(i);
        }, i*1000)
    }
```
会发现每一次输出的都是`6`,为啥勒？所有的回调函数回在循环结束后才会执行（事件循环）。所以每次都是输出一个6来。

解决办法：

1、IIFE，每个迭代储存`i`的值
```javascript
    for(var i = 1; i <= 5; i++) {
        (function(i) {
            var j = i;
            setTimeout(function() {
                console.log(j);
            }, j*1000);
        })(i)
    }
```
2、IIFE，将`i`入参修改成`j`
```javascript
    for(var i = 1; i <= 5; i++) {
        (function(j) {
            setTimeout(function() {
                console.log(j);
            }, j*1000);
        })(i)
    }
```

3、创建闭包的块作用域
```javascript
    for(var i = 1; i <= 5; i++) {
        let j = i;
        setTimeout(function() {
            console.log(j);
        }, j*1000);
    }
```

4、最优
```javascript
    for(let i = 1; i <= 5; i++) {
        setTimeout(function() {
            console.log(i);
        }, i*1000)
    }
```

另外一个闭包常用的场景，就是模块暴露了，在这里提供一个现代模块机制的实现方式，大家可以细细品尝，
```javascript
var MyMoudles = (function Manger() {
    var modules = {};
    function define(name, deps, impl) {
        for (var i = 0; i < deps.length; i++) {
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply(impl, deps);
    }
    function get(name) {
        return modules[name];
    }
})()
```
`调用Demo`:
```javascript
MyMoudles.define('bar', [], function() {
    function hello(who) {
        return "Let me introduce: " + who;
    }
    return {
        hello: hello
    }
})
MyMoudles.define('foo', ['bar'], function(bar) {
    var hungry = 'hippo';
    function awesome() {
        console.log(bar.hello(hungry).toUpperCase());
    }
    return {
        awesome: awesome
    }
})
var bar = MyMoudles.get('bar');
var foo = MyMoudles.get('foo');

console.log(bar.hello('hippo')); // Let me introduce: hippo
foo.awesome();
```

### 未来模块机制
`ES6`提供了全新的模块机制，基于函数的模块（如上述现代模块机制）并不是一个能被静态识别的模式（编译器无法识别），它们的API语义只有等到代码运行时才会考虑进来，而`ES6`模块就是一个能被静态识别的模式，就是说API在编译阶段就会检查API成员是否存在。