---
title: Webpack学习－Loader
date: 2019-03-09 22:39:27
tag: webpack
categories:
- webpack
---
## 什么是Loader?
继上两篇文章webpack工作原理介绍（[上篇](http://wushaobin.top/2019/02/12/webpackPrinciple/)、[下篇](http://wushaobin.top/2019/02/17/webpackPrinciple1/)）,我们了解到`Loader：模块转换器，也就是将模块的内容按照需求装换成新内容`，而且每个Loader的职责都是单一，只会完成一种转换，所以我们一般对源文件的处理，也是由多个Loader以链式顺序执行的方式来进行多次装换，然后得到我们要的结果。

那么这样Loader只需要关心输入和输出，Loader其实是一个Node.js模块，该模块导出的是一个函数（意味着，所有node.js的api我们都可以使用），如下：
```javascript
    module.exports = function (source) {
        // 对source做一系列的转换
        return source;
    }
```
下面我们介绍一下webpack提供了哪些供Loader调用的api，对Loader有个比较深刻的理解，然后来分析`babel-loader`的源码，看看我们常用的loader是怎么编写出来的。
## 获得Loader的options
```javascript
    const loaderUtils = require('loader-utils');
    module.exports = function(source) {
        // 获取用户为当前Loader传入的options
        console.log(loaderUtils.getOptions(this));
        return source;
    }
```
## 返回其他结果
如上，我们返回的是转换后的内容，但是有些情况下，我们不仅仅需要返回转换后的内容，还需要返回一些其他的内容，如sourceMap或是AST语法树，那么这时候我们可以使用webpack提供的API`this.callback`，当使用`this.callback`了，那么我们就必须需要在Loader函数返回`undefined`,以此来让webpack知道返回的结果在`this.callback`中，API详细参数如下：
```javascript
    this.callback(
        // 无法装换原内容时的Error
        err: Error || null,
        // 装换后的的内容，如上述的source
        content: string | Buffer,
        // 用于通过装换后的内容得出原内容的Source Map，方便调试
        // 我们了解到，SourceMap我们只是会在开发环境去使用，于是就会变成可控制的，
        // webpack也提供了this.sourceMap去告诉是否需要使用sourceMap，
        // 当然也可以使用loader的option来做判断，如css-loader
        sourceMap?: SourceMap,
        // 如果本次转换同时生成ast语法树，也可以将这个ast返回，方便后续loader需要复用该ast，这样可以提高性能
        abstractSyntaxTree? AST
    );
```
## 同步与异步
看看异步Loader在`this.async`API下如何实现，
```javascript
    module.exports = async function (source) {
        const callback = this.async();
        const { err, content, sourceMap, AST } = await Func();
        callback(err, content, sourceMap, AST); // 如上诉`this.callback`参数一样
    }
```
## 处理二进制数据
像`file-loader`这样的Loader，处理的是二进制数据，那么就需要告诉webpack给loader传入二进制格式的数据，代码可以如下：
```javascript
    module.exports = function(source) {
        if (source instanceof Buffer) {
            // 一系列操作
            return source; //当然我本身也可以返回二进制数据提供给下一个loader
        }
    }
    moudle.exports.raw = true; //不设置，就会拿到字符串
```
通过`moudle.exports.raw = true;`告知webpack，自己本身需要二进制数据。

## 缓存加速
优化的最佳点，可以使用`this.cacheable(Boolen)`，缓存loader转换后的内容，当处理文件或依赖文件没有发生变化时，使用缓存的转换内容，以此提速！

## 其他API
说到学习，当然越系统越好了，api多介绍 ，除了上面常用的api之外，还存在以下常用的api。
> * `this.context`: 当前处理转换的文件所在的目录
> * `this.resource`: 当前处理转换的文件完整请求路径，包括querystring
> * `this.resourcePath`: 当前处理转换的文件的路径
> * `this.resourceQuery`: 当前处理文件的querystring
> * `this.target`: webpack配置的target
> * `this.loadMoudle`: 处理文件时，需要依赖其他文件的处理结果时，可以使用this.loadMoudle(request: string, callback: function(err, source, sourceMap, module))去获取到依赖文件的处理结果。
> * `this.resolve`: 获取指定文件的完整路径，this.resolve(context: string, request: string, callback: function(err, result: string))
> * `this.addDependency`: 为当前处理文件添加依赖文件，以便依赖文件发生变化时重新调用Loader转换该文件,this.addDependency(file: string)
> * `this.addContextDependency`: 为当前处理文件添加依赖文件目录，以便依赖文件目录里文件发生变化时重新调用Loader转换该文件，this.addContextDependency(dir: string)
> * `this.clearDependencies`: 清除当前正在处理的文件的所有依赖
> * `this.emitFile`: 输出一个文件，使用的方法为this.emitFile(name: string, content: Buffer | string, sourceMap: {...})

## babel-loader源码简析
源码第一行如下：
```javascript
    let babel;
    try {
        babel = require("@babel/core");
    } catch (err) {
        if (err.code === "MODULE_NOT_FOUND") {
            err.message +=
            "\n babel-loader@8 requires Babel 7.x (the package '@babel/core'). " +
            "If you'd like to use Babel 6.x ('babel-core'), you should install 'babel-loader@7'.";
        }
        throw err;
    }
```
babel-loader依赖了`@babel/core`,这就是安装babel-loader需要同时安装`@babel/core`（通常会再安装`babel-preset-env`、`babel-plugin-transform-runtime`、`babel-runtime`）的原因。我们接下去看，`src/index.js`整个文件是不是按照我们前面所讲编写Loader的方法来组织代码的。
``` javascript
//引入package.json
const pkg = require("../package.json");
/* 
根据babel-loader是否配置cacheDirectory属性来告诉
babel-loader是否缓存loader的执行结果，如果true，
便会使用cache方法去实现，`cache.js`文件有着read、write、filename（文件命名方法）
以及如何处理缓存的handleCache方法（有则读，无则写再读），有兴趣可以去看看。
*/
const cache = require("./cache");
/*
    transfrom.js用来转换内容，内部调用了babel.transform方法进行转换，这里简单介绍一下babel的原理：
    babylon将es6/es7代码解析成ast，babel-traverse对ast进行转译，得到新的ast，新的ast通过
    babel-generator转换成es5，核心方法在@babel/core/lib/transformation/index.js中的`runSync`
    方法，有兴趣可以去了解一下。
*/
const transform = require("./transform");
const injectCaller = require("./injectCaller");
const path = require("path");
// 获取Loader参数options
const loaderUtils = require("loader-utils");

module.exports = makeLoader();
module.exports.custom = makeLoader;

function makeLoader(callback) {
  const overrides = callback ? callback(babel) : undefined;

  return function(source, inputSourceMap) {
    // 上面介绍过的api可以得知，这是个异步Loader,做的是异步装换的工作
    const callback = this.async();

    loader
      .call(this, source, inputSourceMap, overrides)
      .then(args => callback(null, ...args), err => callback(err));
  };
}

async function loader(source, inputSourceMap, overrides) {
    ....
}

```
可以看到确实和我们Loader编写方式是一样的，通过`module.exports = makeLoader();`导出一个函数，`makeLoader()`是一个高阶函数，又返回了一个函数，通过`const callback = this.async();`可以知道，这是一个异步的loader，不难看出最重要的实现都在这一步函数loader里面了，那么到底在loader函数里面究竟做了些什么呢？我们来看看，在阅读源码前，最好先看看`babel-loader`的[README](https://github.com/babel/babel-loader)，先做个基本了解.

上面代码可以看出`loader(source, inputSourceMap, overrides)`函数入参有三个，分别是`source=>待转换的code`，`inputSourceMap=>上一个loader处理后的sourceMap,有的话`，`overrides=>自定义加载器`，整块源码可以分成几部分，
* `let loaderOptions = loaderUtils.getOptions(this) || {};`，获取options，并且获取当前处理转换的文件的路径`this.resourcePath`。
* 判断是否自定义加载器转换，这里会进行一系列对options.customize进行判断，options.customize一个相对路径，loader函数参数overrides为空时起效，执行`let override = require(loaderOptions.customize);`，有了override之后，后续逻辑（如转换、获取option）override都会进行介入处理。
* 将函数传入参数和LoaderOptions归并，得到programmaticOptions。
* 调用`babel.loadPartialConfig`可以拿到babel配置并赋值给config变量,其实就是为了允许系统轻松操作和验证用户的配置，此功能解决了插件和预设
* 生成cacheIdentifier
* 判断options.cacheDirectory是否需要缓存Loader转换内容，如为true，调用cache.js的module.export Cache方法（上面已做介绍）
* ` config.babelrc`不为空，则有.babelrc文件，依赖.babelrc文件变化，使用`this.addDependency(config.babelrc);`
* metadataSubscribers 订阅元数据，主要作用是订阅一些编译过程中的一些元数据，订阅以后这些元数据将会被添加 到webpack的上下文中。通常我们是用不上的，估计在某些babel-plugin中可能会使用到。
* 最后将处理后的结果返回

## 小结
每一个Loader其实返回值就是一个Function，而且就是把带转换内容传入，得到转换后的内容，做的事情就是这样，这篇文章先对Loader的基本概念进行介绍，并且了解webpack为Loader的编写提供一些常用的API,最后通过简析babel-loader的源码，我觉得应该差不多知道如何去写一个简单的Loader了。