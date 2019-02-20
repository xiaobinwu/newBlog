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
webpackJsonp([0],[
/* 0 */,
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
### bundle.js
```javascript
 (function(modules) { // webpackBootstrap
 	// install a JSONP callback for chunk loading
 	var parentJsonpFunction = window["webpackJsonp"];
 	window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
 		// add "moreModules" to the modules object,
 		// then flag all "chunkIds" as loaded and fire callback
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
 		while(resolves.length) {
 			resolves.shift()();
 		}

 	};

 	// The module cache
 	var installedModules = {};

 	// objects to store loaded and loading chunks
 	var installedChunks = {
 		1: 0
 	};

 	// //去传入的modules数组中加载对应moduleId（index）的模块，与node的require语句相似，同上，此处省略
 	function __webpack_require__(moduleId) {
		...
 	}

 	// This file contains only the entry chunk.
 	// The chunk loading function for additional chunks
 	__webpack_require__.e = function requireEnsure(chunkId) {
 		var installedChunkData = installedChunks[chunkId];
 		if(installedChunkData === 0) {
 			return new Promise(function(resolve) { resolve(); });
 		}

 		// a Promise means "currently loading".
 		if(installedChunkData) {
 			return installedChunkData[2];
 		}

 		// setup Promise in chunk cache
 		var promise = new Promise(function(resolve, reject) {
 			installedChunkData = installedChunks[chunkId] = [resolve, reject];
 		});
 		installedChunkData[2] = promise;

 		// start chunk loading
 		var head = document.getElementsByTagName('head')[0];
 		var script = document.createElement('script');
 		script.type = "text/javascript";
 		script.charset = 'utf-8';
 		script.async = true;
 		script.timeout = 120000;

 		if (__webpack_require__.nc) {
 			script.setAttribute("nonce", __webpack_require__.nc);
 		}
 		script.src = __webpack_require__.p + "" + chunkId + ".bundle.js";
 		var timeout = setTimeout(onScriptComplete, 120000);
 		script.onerror = script.onload = onScriptComplete;
 		function onScriptComplete() {
 			// avoid mem leaks in IE.
 			script.onerror = script.onload = null;
 			clearTimeout(timeout);
 			var chunk = installedChunks[chunkId];
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
 	

 	// on error function for async loading
 	__webpack_require__.oe = function(err) { console.error(err); throw err; };

 	// Load entry module and return exports
 	return __webpack_require__(__webpack_require__.s = 0);
 })
/************************************************************************/
 ([
/* 0 */
/***/ (function(module, exports, __webpack_require__) {

// 异步加载 show.js
__webpack_require__.e/* import() */(0).then(__webpack_require__.bind(null, 1)).then((show) => {
  // 执行 show 函数
  show('Webpack');
});


/***/ })
 ]);
```