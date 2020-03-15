---
title: 模拟实现js的bind方法
date: 2020-03-15 17:01:01
tags: [JS, 深入js系列]
categories: JS
---

####  new是什么

> 一句话介绍**new** ：**new**运算符创建一个用户自定义的对象类型的实例，或者具有构造函数的内置对象类型之一。看下下面的代码来了解new操作符都做了什么事情

<!--more-->

```js
// Class (constructor)
function Person(name,age){
    this.name = name
    this.age = age
    this.habit = 'watch tv'
}

// 每个函数都有prototype对象属性
// 在类的原型上挂载属性和方法，挂载载原型上，每个实例都可以调用，并且不会每个实例都挂载相同的属性和方法
Person.prototype.strLength = 60
Person.prototype.sayName = function(){
    console.log('I am '+ this.name)
}

// 实例化对象
const person = new Person("yato", 50)
person.sayName();
console.log(person)

```

![20200315154922imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200315154922-image.png)

####   进一步理解new

> 从上面这个例子中，我们可以看到，实例person可以
>
> + 访问到Person构造函数里的属性
> + 访问到Person.prototype中的属性

####  接下来模拟实现一个类似new的`newFake`，使用方式如下

```js
function Person(arguments){
    // ...
}

// 使用new
let person = new Person(arguments)

// 使用newFake
let person = newFake(Person,arguments)
```

#### 初步实现

```js
function newFake(){
    let obj = Object.create({})

    let Constructor = [].shift.call(arguments)

    if(typeof Constructor !== 'function'){
        throw 'newOperator function the first param must be a function';
    }
        
    // 将新建对象的[[prototype]]属性指向到构造函数的prototype属性
    obj.__proto__ = Constructor.prototype

    // 修改this指向到obj
    Constructor.apply(obj, arguments)

    return obj
}

function Person(name,age){
    this.name = name
    this.age = age
    this.habit = 'watch tv'
}

// 每个函数都有prototype对象属性
// 在类的原型上挂载属性和方法，挂载载原型上，每个实例都可以调用，并且不会每个实例都挂载相同的属性和方法
Person.prototype.strLength = 60
Person.prototype.sayName = function(){
    console.log('I am '+ this.name)
}

// 实例化对象
const person = newFake(Person,"yato", 50)
person.sayName();
console.log(person)
```

####  返回值处理

> 如果构造函数有返回值的情况

```js
function Person(name,age){
    this.strLength = 60
    this.age = age
    
    return {
 		name: name,
        habit: 'game'
    }
}

let person = new Person('yato', 18)

console.log(person.name) // yato
console.log(person.habit) // game
console.log(person.strength) // undefined
console.log(person.age) // undefined
```

> 在这个例子中，构造函数返回了一个对象，在实例person中只能访问返回对象中的属性，而且还要注意一点，这里我们是返回一个对象，假设我们只返回一个基本类型值呢，看下面的例子

```js
function Person(name,age){
    this.strLength = 60
    this.age = age
    
    return 'good job'
}

let person = new Person('yato', 18)

console.log(person.name) // undefined
console.log(person.habit) // undefined
console.log(person.strength) // 60
console.log(person.age) // 18
```

> 结果和正常new实例，无返回值的时候表现是一样的！可以得出结论：new操作最后一步，需要判断一返回值是否是一个对象，如果是一个对象就返回这个对象，如果不是对象就返回我们new内部的实例对象

#### 第二版的new实现

```js
function newFake(){
    let obj = Object.create({})

    let Constructor = [].shift.call(arguments)

    if(typeof Constructor !== 'function'){
        throw 'newOperator function the first param must be a function';
    }
    
    // ES6 new.target 是指向构造函数
    newFake.target = ctor;
        
    // 将新建对象的[[prototype]]属性指向到构造函数的prototype属性
    obj.__proto__ = Constructor.prototype

    // 修改this指向到obj
	let ret = Constructor.apply(obj, arguments)
    
    // 判断返回值是否为对象 Object(包含Functoin, Array, Date, RegExg, Error)都会直接返回这些值。
    // 这些类型中合并起来只有Object和Function两种类型 typeof null 也是'object'所以要不等于null，排除	 // null
    let isObject = typeof ret === 'object' && ret !== null;
    let isFunction = typeof ret === 'function';

    if(isObject || isFunction){
        return ret;
    }
    
    return obj
}
```

####  总结一下new做了什么

+ 创建了一个全新的对象
+ 这个对象会被执行实例**[[prototype]]属性**到**Class**的**`prototype`**对象属性的链接（原型链）
+ 生成的新对象会成为构造函数的调用的**this**(修改this指向)
+ 通过new创建的每个**实例对象**最终将被**[[prototype]]**链接到构造函数的**prototype**对象上
+ 如果函数没有返回对象类型（包含`Function`,` Array`,`Date`,`RegExg`,`Error`）,那么new表达式中的函数会自动返回这个新的对象

