---
title: Webpack学习－Plugin
date: 2019-03-15 23:33:27
tag: webpack
categories:
- webpack
---
## 什么是Plugin?
在[Webpack学习－工作原理（上）](http://wushaobin.top/2019/02/12/webpackPrinciple/)一文中我们就已经介绍了`Plugin`的基本概念，同时知道了webpack其实很像一条生产线，要经过一系列处理流程后才能将源文件转换成我们理想的输出结果。而webpack构建过程中，`Plugin`会在特定的时机广播对应的事件，插件可以监听这些事件的发生，`Plugin`在webpack构建流程中就是这样的一个角色。同时我们也介绍了很多整个构建流程会广播的事件，那么这篇文章我们一起详细地学习一下如何编写`Plugin`。

其实`Plugin`本质上就是一个class，一个最基础的`Plugin`代码如下：
``` javascript
    class BasePlugin {
        // 构造函数，接收options配置
        constructor(options) {
            ...
        }
        apply(compiler) {
            // 在此处去监听webpack广播的所有事件
            compiler.plugin('compilation', function(compilation) {
                ...
            });
        }
    }

    moudle.exports = BasePlugin;
```
我们可以再看看，webpack会怎么配置`Plugin`,
``` javascript
    module.exports = {
        plugins: [
            new BasePlugin(options)
        ]
    }
```
我们回忆一下，[Webpack学习－工作原理（上）](http://wushaobin.top/2019/02/12/webpackPrinciple/)文章中们介绍过webpack的构建详细流程，初始化的时候会去`new Plugin()`，那么便是会去实例化webpack配置plugins所有的插件，那么第一步插件实例化就有了，而插件中的apply方法会在开始编译时依次被调用，并且传入Compiler对象(后面会深入介绍)，然后调用Compiler.run()开始编译。

## 了解Compiler 和 Compilation
> * Compiler对象包含了Webpack环境所有的配置信息，包含options，loaders，plugins这些信息，这个对象在webpack启动时被实例化，全局唯一，可以简单理解成就是webpack实例
> * Compilation代表着一次新的编译，包含当前的模块资源、编译生成的资源，变化的文件，之前我们了解到compilation事件中compilation对象也会提供很多事件给插件做扩展，同时很多事件的的回调中都会将compilation传入，以便使用
> * Webpack的事件机制应用了观察者模式，Compiler和Compilation同时继承Taptable，所以可以直接在Compiler和Compilation对象广播和监听事件，广播事件`[Compiler | Compilation].apply('event-name', params)`，监听事件`[Compiler | Compilation].plugin('event-name', function(params){...})`