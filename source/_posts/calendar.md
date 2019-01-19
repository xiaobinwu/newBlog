---
title: vue2.0项目－calendar.js(日历组件封装)
date: 2017/03/25 15:50:23
tags: javascript
categories:
- js
---
最近一直闲来无事，便寻思着做一下自己的个人项目，也想说能使用现在比较流行的一些mvvm框架来做，于是就选用了这样的一个技术栈vue2.0+vue-router+vuex+webpack来做，做得也是多页面应用，使用vue-router，也是想说把多个功能模块化，单个模块spa，实现更高的效果。当然现在还在做的过程中，如果感兴趣可以过来star一下，哈哈，https://github.com/xiaobinwu/Wuji，git clone下来看看。

今天要说的是在做这个项目的过程中，自己想加一个日历功能，但是找遍很多插件，没有多少是合我心意，于是就想说自己写一个，但是想象太美好，技术能力不够，写不出，此处省略一万字。最后找到百度日历还挺符合我要的日历功能，但是我想更加自定化更好一下，于是就拿这个来做了一下修改。结果长这样：
![图片描述](https://sfault-image.b0.upaiyun.com/410/481/4104814977-58d67d2eb8189_articlex)



```
 <div id="cal">
        <div id="top">
            <div class="select">
                <select id="year-select"></select>&nbsp;年
                <select id="month-select"></select>&nbsp;月
            </div>
            <div class="step">
                <span id="prev"><</span>
                <span id="next">></span>
            </div>
        </div>
        <ul id="wk">
            <li>一</li>
            <li>二</li>
            <li>三</li>
            <li>四</li>
            <li>五</li>
            <li><b>六</b></li>
            <li><b>日</b></li>
        </ul>
        <div id="cm"></div>
        <div id="bm">
            <div class="heavenly-branch">
                <span id="heavenly"></span>年 &nbsp;
                ［<span id="branch"></span>］
            </div>
            <a href="javascript:;;" id="now-date">回到今天</a>
        </div>
    </div>
```
js：

```
    calendar.init({
        cellClickCallback: function(cell, datedata){
            console.log(datedata);
            alert('你点击的是'+ datedata.solarYear + '年' + datedata.solarMonth + '月' + datedata.solarDate + '日');
        }
    });
```
于是对源代码做了一些注释，为了以后修改，可以去看详细的代码： https://github.com/xiaobinwu/MyResourceLibrary/tree/master/calendar.js