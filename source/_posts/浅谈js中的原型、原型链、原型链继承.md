---
title: 浅谈js中的原型、原型链、原型链继承
---
## 原型对象、 构造函数、实例对象
**原型对象**：这个要从构造函数开始说起，每个构造函数都会带有一个 prototype 属性。该属性是个指针，指向了一个对象，我们称之为 原型对象。什么是指针？指针就好比学生的学号，原型对象则是那个学生。我们通过学号找到唯一的那个学生。假设突然，指针设置 null, 学号重置空了，不要慌，对象还存在，学生也没消失。只是不好找了,用代码来解释可以这样
```javascript 
Person.prototype				//指向原型对象
```
**构造函数**：简单来说我们声明一个方法function Person()，Person()就是我们所说的构造函数，原型对象上默认有一个属性 constructor，该属性也是一个指针，指向其相关联的构造函数，用代码来解释可以这样写
```javascript 
Person.prototype.constructor = Person				//原型对象的constructor属性指向构造函数
```
**实例对象**：new一个构造函数会产生一个实例，实例对象都有一个内部属性__proto__，指向了原型对象。所以实例能够访问原型对象上的所有属性和方法,代码可以这样写
```javascript 
var person = new Person()
person.__proto__ === Person.prototype				//实例对象内部属性__proto__指向原型对象
```
**所以三者的关系是，每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。通俗点说就是，实例通过内部指针可以访问到原型对象，原型对象通过constructor指针，又可以找到构造函数。**
## 例子
```javascript 
function Person (name) {
    this.name = name;
    this.sex = '男'; 
}
Person.prototype.action = function () {
　　console.log('打球');
}
var person= new Person('jiwawa');
person.action();  //打球
```
首先我们定义了一个构造函数 Person(),  Person.prototype 指向的原型对象，原型对象自带的属性construtor又指回了 Person，那么  Person.prototype.constructor == Person. 前面我们说了实例对象person由于其内部指针__proto__指向了该原型对象Person.prototype，所以可以访问到 action方法
## 原型链
 我们知道，实例有一个内部指针_proto__，指向它的原型对象，并且可以访问原型对象上的所有属性和方法。person实例指向了Person的原型对象，可以访问Person原型对象上的所有属性和方法；如果Person原型对象变成了某一个类的实例 xxx，这个实例又会指向一个新的原型对象 XXX，那么 person此时就能访问 xxx的实例属性和 XXX原型对象上的所有属性和方法了。同理，新的原型对象XXX碰巧又是另外一个对象的实例zzz，这个实例zzz又会指向新的原型对象 ZZZ，那么person此时就能访问 zzz的实例属性和 ZZZ原型对象上的所有属性和方法了,当我们一直顺着往上找的话，最终会指向一个null,这就是原型链的终点。
 ```javascript 
//定义一个 Person构造函数，作为 Child的父类
function Person() {
    this.type= '人类';
}

Person.prototype.superFunction= function () {
    console.log(this.type);
}

function Child(name) {
    this.name= name;
    this.sex= '男生';  
}
// Person实例赋值给Child的prototype指针
Child.prototype = new Person();
var child= new Child('小明');
child.superFunction();  //人类（父类的方法）
child.name;  //小明（子类属性）
child.sex;  //男生（子类属性）
```
## 原型链继承
上面的代码首先定义了一个 Person构造函数，通过new Person()得到实例，会包含一个实例属性 type 和一个原型属性 superFunction。另外又定义了一个Child构造函数。然后情况发生变化Child.prototype = new Person()，将Child的原型对象覆盖成了 person 实例。当 child 去访问superFunction属性时，js会先在child的实例属性中查找，发现找不到，然后，js就会去child的原型对象上去找，child的原型对象已经被我们改成了一个person实例，那就是去person实例上去找。先找person 的实例属性，发现还是没有 superFunction, 最后去 person 的原型对象上去找，最后才找到，代码解释如下
```javascript 
var child= new Child('小明');
child.__proto == Child.prototype
//因为我们将Child的原型对象覆盖成了 person 实例。Child.prototype = new Person();
//所以child.__proto__指向new Person()
var person = new Person('小明');
//所以child.__proto 可以找到person实例对象的所有方法
child.superFunction();  //人类（父类的方法）
```
## 附
该博客仅为记录开发中一些遇到的问题及解决方案，后续还会继续更新维护。
