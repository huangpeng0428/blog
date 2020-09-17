---
title: vue.set（this.$set）的正确用法
---
# 项目前沿

在做移动端项目的时候，往往有这样一个需求。头部有多个type切换，对应不同的内容，在以往面向web开发的时候我们往往采用的是点击哪个type传对应的值请求哪个的内容，这样的话每次点击就会产生一次请求，对于用户体验来说是不怎么友好的，在移动webapp的时代，我们完全可以做成原生应用的切换效果。

## 需求分析

既然这样，那么我们就需要创建多个变量来做一些滑动的效果，之前我是想直接这样写的，如下：
```javascript 
//头部分类请求
        async getcategoryList(){
            this.categoryList = await this.$http.post('/pdd/opt_list',{},this.showLoading)
            this.categoryList.forEach((e,i)=>{              //创建需要滑动的多个变量
                 this.dataObj[i] = []						
             })         
```
此时虽然已经创建了对应的多个变量，但是不会触发视图更新,原因在于**javasecipt 的限制，Vue.js 不能检测到对象属性的添加或删除。因为 Vue.js 在初始化实例时将属性转为 getter/setter，所以属性必须在 data 对象上才能让 Vue.js 转换它，才能让它是响应的** 正确的写法应该是：
```javascript 
//头部分类请求
        async getcategoryList(){
            this.categoryList = await this.$http.post('/pdd/opt_list',{},this.showLoading)
            this.categoryList.forEach((e,i)=>{              //创建需要滑动的多个变量
                 this.$set(this.dataObj, `${i}`, [])						
             })         
```

## 解决方案
这样的话，我们现在就有了对应的头部type的多个content，当我们每次点击的时候只需要改变dataObj的下标就可以了，当我们对应的dataObj[i]有数据我们就不进行接口请求代码如下：
```javascript
async ontypeClick(i){			//记录点击的当前type下标
	  if(this.dataObj[i].length == 0){		//如果没有数据进行接口请求
	      await this.getPindata(this.pages[i],this.id,this.tabListIndex,true,true)		//传入对应的下标的page,
	  }
```
getPindata方法代码如下
```javascript
async getPindata(page,id,i,kry,showLoading){
            let postUrl 
            if(this._isSearch){
                postUrl = '/pdd/good_search'
            }else{
                postUrl = '/pdd/query_good_by_opt'
            } 
                let sendData = {
                    opt_id:id,
                    sort_mode:this.sort_mode,
                    sort_type:this.sort_type,
                    pagesize:this.pagesize,
                    pageno:page,
                    keyword:this.keyword
                let data = await this.$http.post(postUrl,sendData,showLoading)
                if(kry){						//下拉刷新
                    if(this._isSearch){         //搜索的格式
                        this.dataObj[i] = data.goods_list
                    }else{
                        this.dataObj[i] = data
                    }
                }else{		//上拉加载
                    if(this._isSearch){
                        this.$set(this.dataObj, i, this.dataObj[i].concat(data.goods_list))
                    }else{
                         this.$set(this.dataObj, i, this.dataObj[i].concat(data))             //进行赋值操作
                    }
                    console.log(this.dataObj)
                }
        },
```
type切换完成的页面
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190127112933436.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hwMTIzNDU2MDAz,size_16,color_FFFFFF,t_70)
## 代码分析
上面的代码主要是介绍数据的一些操作，我们可以将每次请求的数据放在一个对象之中，在循环我们操作完成的对象，这样相对来说会减少一些不必要的请求，其实还可以优化的更好，我们可以在刚进入此页面的时候进行**Promise.all** 进行所有数据的拉取，点击type的时候只需要做左滑右滑的操作就ok拉，下面是一些动画的代码，这边就不解释了，我用的是weex官方的animation，所有没什么可说的。
```javascript
//动画代码如下
animation.transition(
     this.$refs['sowTransform'],
     {
     styles: {
     transform: `translateY(${this.transformStyle}px)`
     },
     duration: 300, //ms
     timingFunction: "ease",
     needLayout: false,
     delay: 0 //ms
     },
 ); 
```
## 附
该博客仅为记录开发中一些遇到的问题及解决方案，后续还会继续更新维护。
