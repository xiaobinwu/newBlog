---
title: React项目优化-服务治理平台
date: 2019-04-11 14:21:51
tags: react
categories:
- js
---
服务治理，搭建针对微服务架构的治理、监控、管理和运维平台，提供微服务治理、性能监控、服务故障分析、可视化的拓扑关系，监控服务运行状态和服务间的调用关系。

### webpack相关
基于create-react-app搭建构建工具，npm run eject暴露出webpack的配置，自己修改。个人对webpack配置进行划分，主要划分为entry.js、modules.js、env.js、polyfills.js等一些单元配置，其实最终就是组合成一份配置供webpack识别。

### webpack相关优化
1、提取公用chunk，将全局需要使用的js库，如react、redux等
2、使用antd，排除locale的引入，避免引入locale，减少体积
``` javascript
new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/),
```

3、保证引用umd版的moment，不会导致既引入es module，也引入umd版的moment，所以webpack的resolve中alias配置：
``` javascript
'moment$': path.resolve('node_modules/moment/moment')
```
4、使用extract-text-webpack-plugin提取css代码,更好利用浏览器缓存
5、对于体积小的图片，我们使用url-loader转换成base64，减少http请求
6、使用happyPack插件去开辟多个子进程去跑多个类型loader的转换，提高打包速度
7、使用UglifyJsPlugin插件对js进行混淆压缩
8、使用内置webpack.optimize.CommonschunkPlugin插件抽取异步chunk的公共代码，形成新的chunk
9、使用ModuleConcatenationPlugin插件做到作用域提升，所有模块都会被提取到同个作用域下面，可以减少代码体积，运行效率提升
10、HashedModulesPlugin，md5处理模块引用路径，减少文件体积
11、分离webpack runtime代码，提取出webpack相关代码，因为这部分代码每次编译会发生变化，导致提取的公共代码也会发生变化，起不到缓存作用
12、对Antd Icon图标库进行抽取，使用webpack-ant-icon-loader换成图标异步加载
13、添加魔法注释,让异步chunk可以跟踪

### babel相关
1、直接使用preset-es2015、preset-state-0、transform-runtime
2、使用@babel/babel-ployfill + preset-env（useBuildIns）
3、改写@babel/babel-ployfill 只引入需要的ployfill垫片
4、尝试性地使用ployfill.io服务

### 代码层面优化
1、自定义echart图标库的引用，整个echart的引入，文件体积相当大，所以使用官方提供的方法自己打包一份echart.custom.js，只把我们需要的图表进行整合
2、使用react-loadable对路由匹配的路由组件进行动态导入，这样将代码进行分割（code spliting），用时才动态引入
3、能够动态导入都会动态导入 import方法
4、使用react.lazy和Suspense对多标签页面组件进行预加载。
5、对数据不经常变动的请求进行数据请求缓存
6、使用tree shaking，"presets": [["env", { "modules": false }]]
7、请求头accept-encoding: g-zip

### react层面的优化
1、为根节点添加些东西，避免js、css未加载完页面空白，但是有一点需要注意的是，添加的内容保持只有一个父亲节点，平级节点过多，影响性能（react底层会去递归节点删除），多entry的时候，可以html-webpack-plugin自动插入loading
2、 使用 prerender-spa-plugin 渲染首屏，有时loading也是我们自己使用的ui框架中的组件，那么prerender-spa-plugin 是一个可以帮你在构建时就生成页面首屏 html 的一个 webpack 插件，原理大致如下：
一、指定 dist 目录和要渲染的路径
二、插件在 dist 目录中开启一个静态服务器，并且使用无头浏览器（puppeteer）访问对应的路径，执行 JS，抓取对应路径的 html。
三、把抓到的内容写入 html，这样即使没有做服务器端渲染，也能达到跟服务器端渲染几乎相同的作用（不考虑动态数据的话）
7、灵活使用shouldComponentUpdate，避免没必要的渲染

### 遇到的坑与难点
1、在使用react-router建立系统路由集合时，配置没错，却每次刷新时，都是404响应未找到页面，因为本身是单页面，使用的也是HTML5 History API，所以需要在devServer控制historyApiFallback，默认为为false，需要开启，才能404响应时能被替代成index.html
2、针对antd menu组件在处理多站点左侧菜单的变化，使用props.openKeys,会导致Menu组件的this.inlineOpenKeys中间存储变量失效，就会出现收缩时的bug，所以使用prop.defaultOpenKeys,可是defaultOpenKeys只会在初始化传入，导致不同系统切换左侧菜单栏打开定位不准，可以使用ref手动去修改Menu组件中的state.openKeys
3、需求存在多个url相同的情况，但是所属侧边栏菜单不一样，这个时候会在每次切换右侧菜单栏的时候纪录下当前右侧菜单命中的id（存入storage），在从新刷新页面执行匹配右侧菜单栏的时候，如果在传入的平级菜单集合对象中能匹配两条纪录以上的话，就可以用上存储在storage的id来定位当前菜单，顶级菜单同理。
4、使用antd，Menu，DropDown都是会点击触发多次，于是e.domEvent存在则，阻止合成事件的冒泡
5、错误边界ErrorBoundary应该在react-loadable自定义渲染配置中进行配置，这样错误边界就只会作用于内容区域，不会造成整个站点点不动的情况
6、对于很多带表单的modal组件，Form.create()能生成高阶组件，但是一个页面有多个表单，这个时候Form.create()在使用上会受限，那么就应该抽离出来再封装一层高阶组件，这样在每个高阶组件中就可以使用form prop了
7、提供两套主题变量（js），由于antd使用的事less，所以在支持主题切换的时候，动态引入less.js，使用其modifyVar对主题进行切换