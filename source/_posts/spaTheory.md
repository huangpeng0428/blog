---
title: 浅谈单页应用SPA实现原理
---
# 背景介绍
项目中本人使用Vue作为主要技术栈，众所周知Vue主要用于开发单页面（SPA）应用的框架，具有良好的用户体验，用户不需要重新刷新页面，获取数据也是通过Axios异步获取，使得页面显示更为流畅。


## 分析

既然这样，那我们就探究下单页SPA的实现原理吧：查阅相关资料，目前主要有两种实现方式1、监听hash的改变 2、H5新增的的history API

## Hash
 在url中可以带上一个#,这个就相当于是hash,比如下面的singlepage
```javascript 
http://www.paul.com/index.html#singlepage
```
当url的片段标识符发生变化时，将触发window对象中的onhashchange事件， 然后做一些操作即可，在以下几种情况下会触发window的onhashchange事件

 1. 直接更改浏览器地址，在最后面增加或改变#hash；
 2. 通过改变location.href或location.hash的值；
 3. 通过触发点击带锚点的链接；
 4. 浏览器前进后退可能导致hash的变化，前提是两个网页地址中的hash值不同。


我们可以拿到window.onhashchange中的方法回调,此时回调中有两个重要的属性如下，分别代表跳转前后的url地址

```javascript 
<a href="#singlepage">click me</a>
<script type="text/javascript">
  window.onhashchange = function(e){
    console.log(e.oldURL);   //http://www.paul.com/index.html
    console.log(e.newURL);   //http://www.paul.com/index.html#singlepage
  }
</script>  
```
现在我们只需要做一些简单的对比即可，那么我们如何拿到当前的hash值呢。其实呢也简单，我们都知道页面的location对象，其中就有一个我们需要的属性hash：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311184846625.png)
关键的地方我们已经找到了，既然这样的话现在就能用代码实现简单的单页应用了
```javascript
//html
<a href="#singlepage">单页SPA</a>
<a href="#singlepage1">单页1</a>
<a href="#singlepage2">单页2</a>
<p id="singlepageId"></p>
```
```javascript
//js
//监听onhashchange事件
window.onhashchange = function(e) {
  let page = location.hash;
  if (page === "#singlepage") {
    document.getElementById("singlepageId").innerHTML = "这是单页SPA";
    return;
  } else if (page === "#singlepage1") {
    document.getElementById("singlepageId").innerHTML = "这是单页1";
    return;
  }else if (page === "#singlepage2") {
    document.getElementById("singlepageId").innerHTML = "这是单页2";
    return;
  }
};
```
hash实现起来和简单，无非就是监听url变化，将对应的内容嵌套进相应的内容组件。所以这也不是主流的单页面方法，其实目前使用最多的方法是H5的history方法，我们来浅谈下history的实现。
## History
**属性**
length：该属性代表着浏览器历史列表中的URL数量。初始值为1，如果当前窗口先后访问了两个网址，那该属性的值变为2。
```javascript
history.length          // 1  当前窗口1个网址
history.length          // 2  当前窗口2个网址
```
**方法**
go():传入number作为参数，页面会跳进History的url列表的相应位置
```javascript
window.history.go(-1)           // 后退
window.history.go(1)            // 前进
window.history.go(0)            // 刷新
window.history.go()             // 刷新
```
forward():可以加载历史列表中的下一个URL，类似于go(1)，实现了点击浏览器前进按钮的效果
```javascript
window.history.forward()        // 前进
window.history.go(1)            // 前进
```
back():可以加载历史列表中的上一个URL，类似于go(-1)，实现了点击浏览器后退按钮的效果
```javascript
window.history.back()           // 后退
window.history.go(-1)           // 后退
```
注意：当URL队列中没有上一个URL时，back方法会失效；同理，当URL队列中没有下一个URL时，forward方法会失效。

HTML5提出了pushState(state,title,url)方法和replaceState(state,title,url) 方法，这两种方法可以向浏览器的历史栈中添加数据。
这两个新增方法都接接收3个参数

 1. state：一个与指定网址相关的状态对象，popstate事件触发时，该对象会传入回调函数。如果不需要这个对象（即不需要传参），可以设为null
 2. title：新页面的标题，但是大部分浏览器目前都忽略这个值，所以这里也可以设为null
 3. url：新的网址，必须与当前页处在同一域，浏览器的地址栏将显示这个网址


**pushState**
pushState方法不会触发页面刷新，只是导致history对象发生变化，地址栏会有反应,只有当触发前进后退等事件（back()和forward()等）时浏览器才会刷新这里的。 url 是受到同源策略限制的，防止恶意脚本模仿其他网站 url 用来欺骗用户，所以当违背同源策略时将会报错。
**replaceState**
replaceState(stateObj, title, url) 和pushState的区别就在于它不是写入而是替换修改浏览历史中当前纪录，其余和 pushState一模一样。
**popstate()**
history模式在Vue中，vue-router每次需要跳转页面时，pushState()方法都会在历史记录中添加一条新的url，当用户点击前进、后退按钮，或者调用back、forward、go方法时会触发popstate()，popstate事件的state属性会包含一份来自history实体的state对象的拷贝，此时可以做一些相应的操作。

总结来说就是pushState()改变浏览器url地址并携带state对象作为参数，url的变化会触发popstate(),并拿到state对象，然后做一些逻辑处理，代码如下：

```javascript
<a class="page demoA">demoA.html</a>
<a class="page demoB">demoB.html</a>
 // 注册路由
  document.querySelectorAll('.page').forEach((e,i) => {
   e.addEventListener('click', item => {
    item .preventDefault();
    let link = e.textContent;
    if (!!(window.history && history.pushState)) {
     // 支持History API
     window.history.pushState({name: 'api'}, link, link);
    } else {
     // 不支持,可使用一些Polyfill库来实现
    }
   }, false)
  });
 
  // 监听路由
  window.addEventListener('popstate', e => {
   console.log({
    location: location.href,
    state: e.state
   })
  }, false)
```
## 附
该博客仅为记录开发中一些遇到的问题及解决方案，后续还会继续更新维护。
