---
title: 浅谈JS事件循环机制（event loop）的宏任务，微任务
---
首先看一段代码
```javascript 
async function f1(){
		await f2()
		console.log('f1')
	}
// 		
	async function f2(){
		console.log('f2')
	}
	
	console.log('正常1')
	f1()
	setTimeout(()=>{
		console.log('定时器')
	})
	console.log('正常2')
```
正确的打印顺序应该是：正常1，f2 ，正常2，f1，定时器
## 为什么会出现这样打印顺序呢
首先javascript是一门单线程语言，在最新的HTML5中提出了Web-Worker，但javascript是单线程这一核心仍未改变。既然js是单线程，那就像只有一个窗口的银行，客户需要排队一个一个办理业务，同理js任务也要一个一个顺序执行。如果一个任务耗时过长，那么后一个任务也必须等着。所以就出现了同步任务和异步任务。
## 概念
除了广义的同步任务和异步任务，对任务可以进行更精细的区分
macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
micro-task(微任务)：Promise，process.nextTick

**宏任务**：浏览器为了能够使得JS内部task与DOM任务能够有序的执行，会在一个task执行结束后，在下一个 task 执行开始前，对页面进行重新渲染 （task->渲染->task->...）
鼠标点击会触发一个事件回调，需要执行一个宏任务，然后解析HTMl

**微任务**：微任务通常来说就是需要在当前 同步任务 执行结束后立即执行的任务，比如对一系列动作做出反馈，或者是需要异步的执行任务而又不需要分配一个新的任务，这样便可以减小一点性能的开销。
既然我们清楚了概念，我们再看一遍代码
```javascript 
async function f1(){
		await f2()
		console.log('f1')
	}
// 		
	async function f2(){
		console.log('f2')
	}
	
	console.log('正常1')
	f1()
	setTimeout(()=>{
		console.log('定时器')
	})
	console.log('正常2')
```
## 执行顺序
首先我们进行正常的同步流程，打印出‘正常1’，接下来执行f1()函数，await后面的函数f2()会立即执行，所以会打印'f2',继续执行同步代码打印‘正常2’，至此同步任务全部结束，开始执行异步任务微任务，await f2()等待f2()方法执行完之后打印出f1，最后执行宏任务setTimeout打印‘定时器’

这就是为什么‘正常1’,正常2'会打印在‘f1’之前，因为所有微任务执行的时候，当前执行栈的代码必须已经执行完毕。‘f2’,'f1'会打印在‘定时器’之前是因为所有微任务总会在下一个宏任务之前全部执行完毕
## 附
该博客仅为记录开发中一些遇到的问题及解决方案，后续还会继续更新维护。
