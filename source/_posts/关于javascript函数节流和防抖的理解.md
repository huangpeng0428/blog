---
title: 函数防抖和函数节流的理解
---

# 背景介绍
最近本人在开发移动端应用中，遇到关于函数节流和函数防抖功能使用，一个是搜索功能，用户输入完成之后进行接口请求这么一个简单的功能，这里需要用到防抖操作。还有一个功能是我需要监听滚动条距离底部的位置，此时我需要进行一些节流处理，优化前端性能。这是最近在项目中遇到的一些问题，所以记录在博客中以便以后查看，接下来就来简单谈下节流和防抖的的概念和简单的demo实现。


## 概念


**防抖**：将需要执行的方法放在setTimeout中来辅助实现。延迟执行需要跑的代码。如果方法多次触发，则把上次记录的延迟执行代码用clearTimeout清掉，重新开始。如果计时完毕，没有方法进来访问触发，则执行代码

**节流**：声明一个变量当标志位，记录当前代码是否在执行。如果空闲，则可以正常触发方法执行。如果代码正在执行，则取消这次方法执行，直接return
既然我们清楚了概念，接下来我们可以用代码来做简单的实现
## 页面基础页
```javascript 
<style type="text/css" >
		div{
		   width: 150px;
		   height: 150px;
		   text-align: center;
		   line-height: 150px;
		   color: #fff;
		   background: #000;
		   display:inline-block;
		   position:relative;
		   margin:5px
		}
		.div1::after,.div2::after,.div3::after{
		  position:absolute;
		  top:10px;
		  left:10px;
		  line-height:1
		}
		.div1::after{
		  content:'节流';
		}
		.div2::after{
		  content:'正常';
		}
		.div3::after{
		content:'防抖';
		}
	</style>
	<body>
		<div class="div1"></div>
		<div class="div2"></div>
		div class="div3"></div>
	</body>
```
## 防抖
 只要鼠标有操作就清除定时器,代码将不再执行,在500ms后鼠标没有任何移动操作则继续执行定时器中的代码（本人之前做web端自动保存有用到过）
 ```javascript 
var timer = false;
document.querySelector(".div3").onmousemove = function(){
	clearTimeout(timer); // 清除未执行的代码，重置回初始化状态
	timer = setTimeout(function(){
		console.log("函数防抖");
	}, 500);
};
```
## 节流
声明一个变量当标志位，记录当前代码是否在执行。如果空闲，则可以正常触发方法执行。如果代码正在执行，则取消这次方法执行
 ```javascript 
var canRun = true;
var a = 0
document.querySelector(".div1").onmousemove = function(){
		if(!canRun){
			// 判断是否已空闲，如果在执行中，则直接return
			return;
		}
		canRun = false;
		setTimeout(function(){
			console.log('节流')
			a += 1
			document.querySelector('.div1').innerHTML = a 
			canRun = true;
		}, 300);
	};
```
## 附
该博客仅为记录开发中一些遇到的问题及解决方案，后续还会继续更新维护。
