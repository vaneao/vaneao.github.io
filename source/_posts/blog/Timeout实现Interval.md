---
title: 用setTimeout、clearTimeout实现setInterval、clearInterval
categories:
- 前端
tags:
- JS
- 手撕
cover: https://tva1.sinaimg.cn/large/00897VxHly8gpsktrr06qj30jg0jgani.jpg
---
# 简单实现setInterval
```js
var mySetInterval = function(f, time) {
	//定义一个递归函数，持续调用定时器
	var excute = function (f, time) {
		setTimeout(function() {
			f();
			excute(f, time);
		}, time);
	}
	excute(f, time);
}
```
# 改进实现clearInterval
```js
var timeWorker = {};
var mySetInterval = function(f, time) {
	//定义一个key，来标识此定时器
	let key = Symbol();
	//定义一个递归函数，持续调用定时器
	var excute = function(f, time) {
		timeWorker[key] = setTimeout(function() {
			f();
			excute(f, time);
		}, time);
	}
	excute(f, time);
	return key;
}
var myClearInterval = function(key) {
	if (timeWorker.hasOwnProperty(key)) {
		clearTimeout(timeWorker[key]);
		delete timeWorker[key];
	}
}
```