---
title: 自己实现一个new方法
---
## 前沿
好久没有写点东西了，最近看了下js基础教程，觉得有些东西还是有必要记录下来
## 关于new
new关键字通常和构造函数一起使用，用于创建对象。
```javascript 
function Animal(name, action) {
        this.name= name;
        this.action= action;
        this.run= function () {
            alert(this.action);
        };
    }
    let animal1= new Animal("tigger","run");
    let animal2= new Animal("cat","jump");
    console.log(animal1.__proto__ === animal2.__proto__);//true
    console.log(animal1.__proto__ === Animal.prototype);//true
    console.log(Animal.prototype.constructor === Animal);//true
    //因此person1.__proto__ = person2.__proto__ = Person
    console.log(animal1.name);//yan
```
## 自己实现New
```javascript 
function Person(name,age,job){
	this.name = name;
	this.age = age;
	this.job = job;
	this.sayName = function() {
		alert(this.name)
	}
}
let New = function NEW_OBJECT(P){
var obj={};
let arg = Array.prototype.slice.call(arguments,1)
obj.__proto__=P.prototype;
obj.__proto__.constructor=P;
P.apply(obj,arg)
return obj;
 
}
let _obj = New(Person,"Polo")
```

let arg = Array.prototype.slice.call(arguments,1)
截取参数，转成数组，改变this指向

## 总结
NEW_OBJECT方法详解：
obj.__proto__=P.prototype; 主要将NEW_OBJECT上的属性赋值给obj对象中
## 附
该博客仅为记录开发中一些遇到的问题及解决方案，后续还会继续更新维护。
