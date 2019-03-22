---
title: Webpack学习－Plugin
date: 2019-03-15 23:33:27
tag: webpack
categories:
- webpack
---
## 什么是Plugin?
在[Webpack学习－工作原理（上）](http://wushaobin.top/2019/02/12/webpackPrinciple/)一文中我们就已经介绍了`Plugin`的基本概念，同时知道了webpack其实很像一条生产线，要经过一系列处理流程后才能将源文件转换成我们理想的输出结果。而webpack构建过程中，会在特定的时机广播对应的事件，插件可以监听这些事件的发生，`Plugin`在webpack构建流程中就是这样的一个角色。同时我们也介绍了很多整个构建流程会广播的事件，那么这篇文章我们一起详细地学习一下如何编写`Plugin`。

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
> * Webpack的事件机制应用了观察者模式，Compiler和Compilation同时继承Taptable，所以可以直接在Compiler和Compilation对象广播和监听事件，广播事件`[Compiler | Compilation].apply('event-name', params)`，监听事件`[Compiler | Compilation].plugin('event-name', function(params){...})`，event-name不能和现有的事件重名

## 开发插件需注意的点
> * 只要能拿到Compiler或是Compilation对象，就能广播新的事件，供其他插件使用
> * Compiler或是Compilation对象为同一个引用，一旦修改就会影响后面的插件
> * 如果事件是异步的，会带两个参数，第二个参数为回调函数，在插件处理完任务时需要调用回调函数通知webpack，才会进入下一个处理流程。如：
```javascript
  compiler.plugin('emit',function(compilation, callback) {
    // 支持处理逻辑

    // 处理完毕后执行 callback 以通知 Webpack 
    // 如果不执行 callback，运行流程将会一直卡在这不往下执行 
    callback();
  });
```
## 常用api
### 读取输出资源、代码块、模块及其依赖
``` javascript
class Plugin {
  apply(compiler) {
    compiler.plugin('emit', function (compilation, callback) {
      // compilation.chunks 存放所有代码块，是一个数组
      compilation.chunks.forEach(function (chunk) {
        // chunk 代表一个代码块
        // 代码块由多个模块组成，通过 chunk.forEachModule 能读取组成代码块的每个模块
        chunk.forEachModule(function (module) {
          // module 代表一个模块
          // module.fileDependencies 存放当前模块的所有依赖的文件路径，是一个数组
          module.fileDependencies.forEach(function (filepath) {
          });
        });

        // Webpack 会根据 Chunk 去生成输出的文件资源，每个 Chunk 都对应一个及其以上的输出文件
        // 例如在 Chunk 中包含了 CSS 模块并且使用了 ExtractTextPlugin 时，
        // 该 Chunk 就会生成 .js 和 .css 两个文件
        chunk.files.forEach(function (filename) {
          // compilation.assets 存放当前所有即将输出的资源
          // 调用一个输出资源的 source() 方法能获取到输出资源的内容
          let source = compilation.assets[filename].source();
        });
      });

      // 这是一个异步事件，要记得调用 callback 通知 Webpack 本次事件监听处理结束。
      // 如果忘记了调用 callback，Webpack 将一直卡在这里而不会往后执行。
      callback();
    })
  }
}
```
### 监听文件变化
``` javascript
// 当依赖的文件发生变化时会触发 watch-run 事件
compiler.plugin('watch-run', (watching, callback) => {
    // 获取发生变化的文件列表
    const changedFiles = watching.compiler.watchFileSystem.watcher.mtimes;
    // changedFiles 格式为键值对，键为发生变化的文件路径。
    if (changedFiles[filePath] !== undefined) {
      // filePath 对应的文件发生了变化
    }
    callback();
});
```
###  为了监听 HTML 文件的变化，我们需要把 HTML 文件加入到依赖列表中，可以怎么做：
``` javascript
compiler.plugin('after-compile', (compilation, callback) => {
  // 把 HTML 文件添加到文件依赖列表，好让 Webpack 去监听 HTML 模块文件，在 HTML 模版文件发生变化时重新启动一次编译
    compilation.fileDependencies.push(filePath);
    callback();
});
```
### 修改输出资源
``` javascript
// 设置 compilation.assets 的代码如下：
compiler.plugin('emit', (compilation, callback) => {
  // 设置名称为 fileName 的输出资源
  compilation.assets[fileName] = {
    // 返回文件内容
    source: () => {
      // fileContent 既可以是代表文本文件的字符串，也可以是代表二进制文件的 Buffer
      return fileContent;
      },
    // 返回文件大小
      size: () => {
      return Buffer.byteLength(fileContent, 'utf8');
    }
  };
  callback();
});
// 读取 compilation.assets 的代码如下：
compiler.plugin('emit', (compilation, callback) => {
  // 读取名称为 fileName 的输出资源
  const asset = compilation.assets[fileName];
  // 获取输出资源的内容
  asset.source();
  // 获取输出资源的文件大小
  asset.size();
  callback();
});
```
### 判断webpack使用了哪些插件
```javascript
// 判断当前配置使用使用了 ExtractTextPlugin，
// compiler 参数即为 Webpack 在 apply(compiler) 中传入的参数
function hasExtractTextPlugin(compiler) {
  // 当前配置所有使用的插件列表
  const plugins = compiler.options.plugins;
  // 去 plugins 中寻找有没有 ExtractTextPlugin 的实例
  return plugins.find(plugin=>plugin.__proto__.constructor === ExtractTextPlugin) != null;
}
```
### 对于上面常用api的讲解，我们可以知道compiler和compilation在plugin中占据着举足轻重的作用，那么具体它们长什么样子的，我们编写个例子打印出来看看,下面以`extract-text-webpack-plugin`插件进行断点调试的截图，可以来看看这两个分别打印出来的东西，
![compiler](http://bin-blog.oss-cn-shenzhen.aliyuncs.com/webpack/1.png)
![compilation](http://bin-blog.oss-cn-shenzhen.aliyuncs.com/webpack/2.png)

## 总结
一般情况下，我们是不需要去写`Plugin`，但是有时候我们有些业务需求是没有插件可以满足的，那么我们便得需要自己去写`Plugin`，那了解`Plugin`的一些相关知识点就是有必要的，我们不一定要每个钩子或是API都相当熟，但是我们需要思路，了解如何编写`Plugin`，也是有必要的，`Plugin`中最重要的compiler和compilation，一个`Plugin`插件也就是围绕着这个去扩展，对应详细内容可以去webpack官网了解，[compiler链接](https://webpack.docschina.org/api/compiler-hooks/)，[compilation链接](https://webpack.docschina.org/api/compilation-hooks/)。