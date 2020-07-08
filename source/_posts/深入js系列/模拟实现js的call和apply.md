---
title: 模拟实现js的call和apply
date: 2020-04-07 10:46:01
tags: [JS, 深入js系列]
categories: JS
---

####  call方法

> call() 方法在使用一个指定的this和若干个指定的参数值的前提下调用某个函数或方法。

<!--more-->

#### call的使用
```js
/* 
*   call语法  
*   function.call(thisArg, arg1, arg2, ...)
*   thisArg 可选参数，在function函数运行时的this值。注意 this可能不是该方法看到的实际值：如果这个函数处于非严格模式，则指定
*   为null或者undefined时会自动替换为指向全局对象，原始值会被包装
*   (数字，字符串，布尔值)的this会执行该原始值的自动包装对象
*   
*   arg1,arg2... 指定的参数列表
*/

function greet() {
    let reply = [this.animal, 'typically sleep between', this.sleepDuration].join(' ')
    console.log(reply)
}

let obj = {
    animal: 'cats',
    sleepDuration: '12 and 16 huors'
}

greet.call(obj);          // cat typically sleep between 12 and 16 hours
```

#### call方法的实现
```js
Function.prototype.callFn = function(context){
    // 只有function类型才可以调用call方法
    if(typeof this !== 'function'){
        throw new TypeError('must be function')
    }

    // 有context时，指定为context，没有指向全局对象，浏览器里为
    // windows，node下需要修改为global
    context = context ? Object(context) : window
    // context = context ? Object(context) : global

    // 此时的this为调用callFn方法的函数的this 
    context.fn = this;

    // 取出函数执行需要的参数
    let args = [...arguments].slice(1)
    
    let r = context.fn(args)

    delete context.fn

    return r
}

// test
let value = 2;

let obj = {
    value: 1
}

function bar(name, age) {
    console.log(this.value);
    return {
        value: this.value,
        name: name,
        age: age
    }
}

bar.callFn(null); // 2

console.log(bar.callFn(obj, 'kevin', 18));
// 1
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }
```

#### apply方法

> call()方法的作用和 apply() 方法类似，区别就是call()方法接受的是参数列表，而apply()方法接受的是一个参数数组。

```js
/* func.apply(thisArg, [argsArray])
参数
thisArg
必选的。在 func 函数运行时使用的 this 值。请注意，this可能不是该方法看到的实际值：
如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对
象，原始值会被包装。
argsArray
可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 func 函数。如果
该参数的值为 null 或  undefined，则表示不需要传入任何参数。从ECMAScript 5 开始可
以使用类数组对象。
*/

window.val = 'window'
var obj = {
 val: 'obj'
}
function getVal(p1, p2) {
  console.log(p1, p2)
  console.log(this.val)
}
getVal('param1', 'param2')       // param1,param2,window
getVal.apply(obj, ['param1', 'param2'])   // param1,param2,obj
```

#### apply实现
```js
Function.prototype.applyFn = function(context) {
    if (typeof this !== 'function') {
      throw new TypeError('Error')
    }
    context = context || window
    // context = context || global    // node环境下
    context.fn = this
    let result

    if (arguments[1]) {
      result = context.fn(...arguments[1])
    } else {
      result = context.fn()
    }
    delete context.fn
    return result
}
  
// test node环境测试
global.val = 'window'
var obj = {
 val: 'obj'
}
function getVal(p1, p2) {
  console.log(p1, p2)
  console.log(this.val)
}
getVal('param1', 'param2') 
getVal.apply(obj, ['param1', 'param2'])
```

[参考文章: https://juejin.im/post/5c4592faf265da617265c60b#heading-2](https://juejin.im/post/5c4592faf265da617265c60b#heading-2)
[参考文章: https://segmentfault.com/a/1190000017206223](https://segmentfault.com/a/1190000017206223)
[参考文章: https://github.com/mqyqingfeng/Blog/issues/11](https://github.com/mqyqingfeng/Blog/issues/11)
