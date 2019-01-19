---
title: Css Secret 案例全套
date: 2019-01-19 18:11:56
tags: css
categories:
- css
---
# Css Secret 案例全套

------
##[github地址](https://github.com/xiaobinwu/css-secret-demo)
##[案例地址](http://htmlpreview.github.io/?https://github.com/xiaobinwu/css-secret-demo/blob/master/index.html)
去年买了一本CSS揭秘的css专题书，该书揭示了 47 个鲜为人知的 CSS 技巧，主要内容包括背景与边框、形状、 视觉效果、字体排印、用户体验、结构与布局、过渡与动画等。去年买入时，就决定将里面所有Demo案例全部撸一遍，作为自己2018年学习清单中的首项。这个过程中也可以学习到一些比较实用的css技巧，对于工作中css布局上面也有挺大的帮助。

下面是几种比较有趣的css场景的实现方式：
> 饼图(基于transform实现方式)
``` html
<div class="picture1">20</div
```
``` css
	/*基于transform的解决方案*/
	.picture1 {
		position: relative;
		width: 100px;
		line-height: 100px;
		text-align: center;
		color: transparent;
		background: yellowgreen;
		background-image: linear-gradient(to right, transparent 50%, #655 0);
		border-radius: 50%;
		/*animation-delay: -20s;*/
	}
	@keyframes spin {
		to { transform: rotate(.5turn); }
	}
	@keyframes bg {
		50% { background: #655; }
	}
	.picture1::before {
		content: '';
		position: absolute;
		top: 0;
		left: 50%;
		width: 50%;
		height: 100%;
		border-radius: 0 100% 100% 0 / 50%;
		background-color: inherit;
		transform-origin: left;
		animation: spin 50s linear infinite,
				   bg 100s step-end infinite;
		animation-play-state: paused;
		animation-delay: inherit;
	}
```
``` javacript
	// 基于transform的解决方案
	let picture1 = document.querySelector('.picture1');
	let rate1 = parseFloat(picture1.textContent);
	picture1.style.animationDelay = `-${rate1}s`;
```
> 饼图(基于svg实现方式)
``` html
	<svg viewBox="0 0 32 32">
		<circle id="circle2" r="16" cx="16" cy="16"></circle>
	</svg>
```
``` css
	/*基于svg的解决方案*/
	svg {
		width: 100px;
		height: 100px;
		transform: rotate(-90deg);
		background: yellowgreen;
		border-radius: 50%;
	}
	circle{
		fill: yellowgreen;
		stroke: #655;
		stroke-width: 32;
	}
	#circle2 {
		stroke-dasharray: 38 100;
	}
```
> 插入换行
``` html
	<dl>
		<dt>Name:</dt>
		<dd>wushaobin</dd>
		<dt>Email:</dt>
		<dd>739288994@qq.com</dd>
		<dd>12345@qq.com</dd>
		<dd>54321@qq.com</dd>
		<dt>Location:</dt>
		<dd>shenzhen</dd>
	</dl>
```
```css
	dt,dd {
		display: inline;
	}
	dd{
		margin: 0;
		font-weight: bold;
	}
	dd+dt::before {
		content: '\A';
		white-space: pre;
	}
	dd+dd::before {
		content: ', ';
		font-weight: normal;
	}
```
