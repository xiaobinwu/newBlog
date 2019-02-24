---
title: Webpack学习－工作原理（下）
date: 2019-02-17 10:51:20
tag: webpack
categories:
- webpack
---
继上篇文章介绍了Webpack的基本概念，完整流程，以及打包过程中广播的一些事件的作用，这篇文章主要讲生成的chunk文件如何输出成具体的文件。分同步和异步两种情况来分析输出的文件`使用的webpack版本：3.8.0`。 

***模块文件show.js***

```javascript
    function show(content) {
        window.document.getElementById('app').innerText = 'Hello,' + content;
    }

    // 通过 CommonJS 规范导出 show 函数
    module.exports = show;
```

## 同步加载

```javascript
// main.js

import show from './show';

show('TWENTY-FOUR K');

```

### 生成的bundle文件
```javascript
// webpackBootstrap启动函数
// modules存放的是所有模块的数组，每个模块都是一个函数
(function(modules) {
	var installedModules = {}; // 缓存安装过的模块，提升性能
	//去传入的modules数组中加载对应moduleId（index）的模块，与node的require语句相似
	function __webpack_require__(moduleId) {
		// 检查是否已经加载过，如果有的话直接从缓存installedModules中取出
		if(installedModules[moduleId]) {
			return installedModules[moduleId].exports;
		}
        // 如果没有的话，就直接新建一个模块，并且存进缓存installedModules
		var module = installedModules[moduleId] = {
			i: moduleId, // 对应modules的索引index，也就是moduleId
			l: false, // 标志模块是否已经加载
			exports: {}
		};
        // 执行对应模块函数，并且传入需要的参数
		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
		// 将标志设置为true
		module.l = true;
		// 返回这个模块的导出值
		return module.exports;
	}
	// 储存modules数组
	__webpack_require__.m = modules;
    // 储存缓存installedModules
	__webpack_require__.c = installedModules;
	// 为Harmony导出定义getter函数
	__webpack_require__.d = function(exports, name, getter) {
		if(!__webpack_require__.o(exports, name)) {
			Object.defineProperty(exports, name, {
				configurable: false,
				enumerable: true,
				get: getter
			});
		}
	};
	// 用于与非协调模块兼容的getdefaultexport函数，将module.default或非module声明成getter函数的a属性上
	__webpack_require__.n = function(module) {
		var getter = module && module.__esModule ?
			function getDefault() { return module['default']; } :
			function getModuleExports() { return module; };
		__webpack_require__.d(getter, 'a', getter);
		return getter;
	};
	// 工具函数，hasOwnProperty
	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
	// webpack配置中的publicPath，用于加载被分割出去的异步代码
	__webpack_require__.p = "";
	// 使用__webpack_require__函数去加载index为0的模块，并且返回index为0的模块也就是主入口文件的main.js的对应文件，__webpack_require__.s的含义是启动模块对应的index
	return __webpack_require__(__webpack_require__.s = 0);
})
/************************************************************************/
([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
// 设置__esModule为true，影响__webpack_require__.n函数的返回值
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
// 同步加载index为1的依赖模块
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__show__ = __webpack_require__(1);
// 获取index为1的依赖模块的export
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__show___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_0__show__);
// 同步加载


__WEBPACK_IMPORTED_MODULE_0__show___default()('wushaobin');

/***/ }),
/* 1 */
/***/ (function(module, exports) {

function show(content) {
  window.document.getElementById('app').innerText = 'Hello,' + content;
}

// 通过 CommonJS 规范导出 show 函数
module.exports = show;


/***/ })
]);
```
## 异步加载

```javascript
// main.js

// 异步加载 show.js
import('./show').then((show) => {
  // 执行 show 函数
  show('TWENTY-FOUR K');
});


```
经webpack打包会生成两个文件0.bundle.js和bundle.js，怎么做的原因，是可以吧show.js以异步加载形式引入，这也是分离代码，达到减少文件体积的优化方法，两个文件分析如下。
### 0.bundle.js
```javascript
// 加载本文件（0.bundle.js）包含的模块， webpackJsonp用于从异步加载的文件中安装模块，挂载至全局（bundle.js）供其他文件使用
webpackJsonp(
// 在其他文件中存放的模块id
[0],[
// 本文件所包含的模块

/***/ (function(module, exports) {

function show(content) {
  window.document.getElementById('app').innerText = 'Hello,' + content;
}

// 通过 CommonJS 规范导出 show 函数
module.exports = show;


/***/ })
]);
```
### bundle.js
```javascript
 (function(modules) { // webpackBootstrap启动函数
 	// 安装用于块加载的JSONP回调
 	var parentJsonpFunction = window["webpackJsonp"];
	// chunkIds 异步加载文件（0.bundle.js）中存放的需要安装的模块对应的chunkId
	// moreModules 异步加载文件（0.bundle.js）中存放需要安装的模块列表
	// executeModules 异步加载文件（0.bundle.js）中存放需要安装的模块安装后需要执行的模块对应的index
 	window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
 		// 将 "moreModules" 添加到modules对象中,
		// 将所有chunkIds对应的模块都标记成已经加载成功
 		var moduleId, chunkId, i = 0, resolves = [], result;
 		for(;i < chunkIds.length; i++) {
 			chunkId = chunkIds[i];
 			if(installedChunks[chunkId]) {
 				resolves.push(installedChunks[chunkId][0]);
 			}
 			installedChunks[chunkId] = 0;
 		}
 		for(moduleId in moreModules) {
 			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
 				modules[moduleId] = moreModules[moduleId];
 			}
 		}
 		if(parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules, executeModules);
 		// 执行
		while(resolves.length) {
 			resolves.shift()();
 		}

 	};

 	// 缓存已经安装的模块
 	var installedModules = {};

 	// 储存每个chunk的加载状态
	// 键为chunk的id，值为0代表加载成功
 	var installedChunks = {
 		1: 0
 	};

 	// 去传入的modules数组中加载对应moduleId（index）的模块，与node的require语句相似，同上，此处省略
 	function __webpack_require__(moduleId) {
		...
 	}

 	// 用于加载被分割出去的需要异步加载的chunk对应的文件，
	// chunkId需要异步加载的chunk对应的id，返回的是一个promise
 	__webpack_require__.e = function requireEnsure(chunkId) {
		// 从installedChunks中获取chunkId对应的chunk文件的加载状态
 		var installedChunkData = installedChunks[chunkId];
		// 如果加载状态为0，则表示该chunk已经加载成功，直接返回promise resolve
 		if(installedChunkData === 0) {
 			return new Promise(function(resolve) { resolve(); });
 		}

		// installedChunkData不为空且不为0时表示chunk正在网络加载中
 		if(installedChunkData) {
 			return installedChunkData[2];
 		}

 		// installedChunkData为空，表示该chunk还没有加载过，去加载该chunk对应的文件
 		var promise = new Promise(function(resolve, reject) {
 			installedChunkData = installedChunks[chunkId] = [resolve, reject];
 		});
 		installedChunkData[2] = promise;

 		// 通过dom操作，向html head中插入一个script标签去异步加载chunk对应的javascript文件
 		var head = document.getElementsByTagName('head')[0];
 		var script = document.createElement('script');
 		script.type = "text/javascript";
 		script.charset = 'utf-8';
 		script.async = true;
 		script.timeout = 120000;
		// HTMLElement 接口的 nonce 属性返回只使用一次的加密数字，被内容安全政策用来决定这次请求是否被允许处理。
 		if (__webpack_require__.nc) {
 			script.setAttribute("nonce", __webpack_require__.nc);
 		}
		// 文件的路径由配置的publicPath、chunkid拼接而成
 		script.src = __webpack_require__.p + "" + chunkId + ".bundle.js";
		// 设置异步加载的最长超时时间
 		var timeout = setTimeout(onScriptComplete, 120000);
 		script.onerror = script.onload = onScriptComplete;
		// 在script加载和执行完成时回调
 		function onScriptComplete() {
 			// 防止内存泄漏
 			script.onerror = script.onload = null;
 			clearTimeout(timeout);
			
 			var chunk = installedChunks[chunkId];
			// 判断chunkid对应chunk是否安装成功
 			if(chunk !== 0) {
 				if(chunk) {
 					chunk[1](new Error('Loading chunk ' + chunkId + ' failed.'));
 				}
 				installedChunks[chunkId] = undefined;
 			}
 		};
 		head.appendChild(script);

 		return promise;
 	};

 	// 这里会给__webpack_require__设置多个属性和方法，同上，此处省略
 	

 	// 异步加载的出错函数
 	__webpack_require__.oe = function(err) { console.error(err); throw err; };

 	// Load entry module and return exports
 	return __webpack_require__(__webpack_require__.s = 0);
 })
/************************************************************************/
 ([
// 存放没有经过异步加载的，随着执行入口文件加载的模块，也就是同步的模块
/* 0 */
/***/ (function(module, exports, __webpack_require__) {
// 通过__webpack_require__.e异步加载show.js对应的chunk
// 异步加载 show.js
__webpack_require__.e/* import() */(0).then(__webpack_require__.bind(null, 1)).then((show) => {
  // 执行 show 函数
  show('Webpack');
});


/***/ })
 ]);
```
## 总结
这里我们通过对比同步加载和异步加载的简单应用，去分析两种情况webpack打包出来文件的差异性，我们发现，对于异步加载的bundle.js与同步的bundle.js基本相似，都是模拟node.js的require语句去导入模块，有所不同的是多了一个__webpack_require__.e函数，用来加载分割出的需要异步加载的chunk对应的文件，以及一个wepackJsonp函数，用于从异步加载的文件中去安装模块。