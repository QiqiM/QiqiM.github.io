---
title: 深入理解js原型链
date: 2020-04-26 21:58:01
tags: [JS, 深入js系列]
categories: JS
---

#### 概念
> 每个函数对象（构造函数constructor）都有一个原型对象属性（prototype），实例对象没有，指向`prototype {}`对象，原型对象都包含一个指向构造函数的指针。实例对象包含一个指向原型对象的内部指针`[[prototype]]`，在浏览器和Node中实现为__proto__。一图胜千言，直接上图

<!--more-->

![原型链1](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200214105542-image.png)

![原型链2](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200428104203-image.png)

![原型链3--经典宝藏](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200309115309-image.png)

#### 在浏览器中看最后一张图的输出

```js
console.log(mycat)
/*
	-Cp1: ""
	constructor:f()
	__proto__: native code
*/

mycat.__proto__
// {Cp1: "", constructor: f(),__proto__: Object}

Cat.prototype
// {Cp1: "", constructor: f(),__proto__: Object}

console.log(mycat.__proto__ === Cat.prototype)  // true

Cat.prototype.constructor
// f () {} ==> 指向函数对象本身（构造函数）
```
![__proto__和prototype关系](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200428112245-image.png)

```js
mycat.__proto__                  // 函数对象

mycat.__proto__.__proto__        // Objec 函数对象

mycat.__proto__.__proto__.__proto__   // null
```


> ![__proto__和prototype关系](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200428113148-image.png)

```js
Object.__proto__                 // Function.prototype 

Object.__proto__.__proto__       // 等价与 Function.prototype.__proto__ 

Object.__proto === Function.prototype 

Function.prototype.__proto__  === Object.prototype
```
> ![Object原型查看](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200428114152-image.png)

![Object原型查看](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200428114655-image.png)

