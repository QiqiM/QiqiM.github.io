---
title: 模拟实现js的bind方法
date: 2020-02-26 21:58:01
tags: [JS, 深入js系列]
---
```js
var obj = {};
console.log(obj);
console.log(typeof Function.prototype.bind); // function
console.log(typeof Function.prototype.bind());  // function
console.log(Function.prototype.bind.name);  // bind
console.log(Function.prototype.bind().name);  // bound
```
![20200229141250imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200229141250-image.png)

1. ####  bind是什么

+ a. `bind` 是`Function`原型链中`Function.prototype`的一个函数，每个函数都可以调用它。

+ b. `bind`本身是一个函数名为`bind`的函数，返回值也是函数，函数名是`bound`。(console出来为

  `bound  `加上一个空格)。

```js
let obj = {
    name: "yato"
}

function original(a,b){
    console.log(this.name)
    console.log([a,b])
    return false
}

let bound = original.bind(obj, 1)
let boundInvoke = bound(2)                    // 'yato', Array(2)[1,2]

console.log(boundInvoke)                       // false
console.log(original.bind.name)                // bind
console.log(original.bind.length)              // 1
console.log(original.bind().length)            // 2 返回original函数形参个数
console.log(bound.name)                        // 'bound original'
console.log((function(){}).bind().name)        // 'bound '
console.log((function(){}).bind().length)      // 0
```

![20200301155455imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200301155455-image.png)

####  2. 进一步理解bind

+ a. 调用`bind`的函数中的`this`指向`bind()`函数的第一个参数。
+ b. 函数`bind()`时传递的参数被bind接受处理，`bind()`完毕之后，程序调用返回的函数（**bound**）时，传递的参数也接收处理了，也就是在`bind()`内部合并处理了。

+ c. 并且`bind()`后的函数的name为`bound+空格+调用bind的函数名`。如果调用函数为匿名函数，则名字为`bound+空格`
+ d. `bind`后的返回值函数，执行后返回值时原函数`（original）`的返回值（上例中的false）

+ e. `bind`函数的形参（即函数的`length`）是`1`。`bind`后返回的`bound函数形参不定`，根据绑定的函数原函数（`original`）形参个数决定。

####  3.根据上面的两个例子，模拟实现一个简单版的`bindFn`

```js
Function.prototype.bindFn = function bindFake(thisArg){
    if(typeof this !== 'function'){
        throw new TypeError(this + 'must be a function')
    }

    // 存储函数本身
    let self  = this
    
    // 去除thisArg的其他参数，转成数组
    let args = [].slice.call(arguments, 1)
    let bound = function(){
        // bind 返回的函数，也就是bound，在程序中被调用时传递的参数转成数组
        let boundArg = [].slice.call(arguments);

        // apply修改this指向，把两个函数的参数合并传给self函数，返回执行结果
        return self.apply(thisArg, args.concat(boundArg))
    }

    return bound
}

// Test
let obj = {
    name: 'yato'
}

function original(a, b){
    console.log(this.name)
    console.log([a,b])
}

let bound = original.bindFn(obj, 1)
bound(2);  // 'yato', [1,2]
```

#### 4.但是函数是可以使用`new`来实例化的。

```js
 let obj = {name : 'yato'}

 function original(a, b){
     console.log('this : ', this)
     console.log('typeof this : ', typeof this)
     this.name = b
     console.log('name: ', this.name)
     console.log('this: ', this)
     console.log([a,b])
 }

 let bound = original.bind(obj, 1)
 let newBoundInvoke = new bound(2)
 console.log('newBoundInvoke: ', newBoundInvoke)
```

**分析例子可以得出结论**

+ a. 从例子中可以看出`this`指向了`new bound()`生成的对象

+ b.  new bound() 的返回值是以original原函数构造器生成的新对象。original原函数的this指向的就是这个新对象。

+ c.简要剖析下new做了什么
  + 1. 创建一个全新的空对象
    2. 对这个对象指向原型链接（`instance.__proto__ = Class.prototype` ），其实`Class.prototype`就是`constructor`
    3. 生成的新对象会绑定到函数调用的this
    4. 通过new创建的每个对象最终被`[[prototype]]`链接这个函数的`prototype`上（参考2）
    5. 如果函数没有返回对象类型`Object`(包含`Function`, `Array`, `Date`, `RegExg`, `Error`),那么`new表达式`中的函数调用会自动返回这个新的对象

##### 4.1所有相当于在new调用时，bind的返回值函数bound内部要实现new的操作

```js
// 第二版 实现new调用
Function.prototype.bindFn = function bindFake(thisArg){
    if(typeof this !== 'function'){
        throw new TypeError(this + ' must be a function')
    }

    // 存储调用bind的函数本身的引用
    let self = this

    // 去除thisArg参数，其他转成数组
    let args = [].slice.call(arguments, 1)
    let bound = function(){
        let boundArgs = [].slice.call(arguments)
        let finalArgs = args.concat(boundArgs)

        // new 调用时，其实this instanceof bound 判断不是很准确。es6
        // new.target就是解决这一问题的
        if(this instanceof bound){
            // 这里是实现上文描述的 new 的第 1, 2, 4 步
            // 1.创建一个全新的对象
            // 2.并且执行[[Prototype]]链接
            // 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。
            // self可能是ES6的箭头函数，没有prototype，所以就没必要再指向做prototype操作。
            if(self.prototype){
                function Empty(){}
                Empty.prototype = self.prototype
                bound.prototype = new Empty()
            }

            // 这里实现的时上文描述的第三步
            // 3.生成的新对象会绑定到函数调用的this
            let result = self.apply(this, finalArgs);

            // 这里是实现上文描述的 new 的第 5 步
            // 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`,               //   `Error`)，
            // 那么`new`表达式中的函数调用会自动返回这个新的对象。
            let isObject = typeof result === 'object' && result !== null
            let isFunction = typeof result === 'function'

            if(isObject || isFunction)
                return result

            return this
        }else{
            // apply修改this指向，把两个函数的参数合并传给self函数，并执行self函数，返回执行结果
            return self.apply(thisArg, finalArgs)
        }
    }
    return bound
}

// Test
let obj = {name : 'yato'}

function original(a, b){
    console.log('this : ', this)
    console.log('typeof this : ', typeof this)
    this.name = b
    console.log('name: ', this.name)
    console.log('this: ', this)
    console.log([a,b])
}

let bound = original.bindFn(obj, 1)
let newBoundInvoke = new bound(2)
console.log('newBoundInvoke: ', newBoundInvoke)
```

#### 5. 总结

+ 1. `bind`是`Function`原型链中`Function.prototype`的一个属性，它是一个函数，修改`this指向`，合并参数传递给原函数，`返回值是一个新的函数`。
+ 2. `bind`返回的函数可以通过`new`调用，这是提供的`this参数被忽略`，指向了new生成的全新对象。`bind()`内部模拟实现了`new`操作符

