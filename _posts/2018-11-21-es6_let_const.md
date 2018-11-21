---
layout: post
title: ‘从闭包到es6变量声明'
subtitle: 'var let const 闭包'
date: 2018-11-21
tags: JavaScript es6
---



之前学习闭包的时候碰到一道题

```javascript
var arr = [];
for (var i = 0; i < 10; i++){
    arr[i] = function(){
        return i
    }
}
console.log(arr[3]())  //10
```

`var`声明的变量没有块级作用域。所以实际上这里的`i`实际上是定义在全局作用域下的

` console.log(window.i) //10`

函数在循环之后调用。所以从作用域链中查找`i`，结果为10

如果要输出3，可以使用立即执行函数修改

```javascript
var arr = []
for (var i = 0; i < 10; i++){
    arr[i] = (function(j){
    	return function(){
    		return j
    	}
    })(i)
}
console.log(arr[3]()) //3
```

立即执行函数传入`i`，形参`j`在当前作用域保存了`i`的值。形成一个闭包。

> mdn对闭包的解释
>
> 闭包是由函数以及创建该函数的词法环境组合而成。**这个环境包含了这个闭包创建时所能访问的所有局部变量**。

这道题有个更优雅的解法,就是我们今天的主角`let`

```javascript
var arr = []
for (let i = 0; i < 10; i++){
    arr[i] = function(){
        return i
    }
}
console.log(arr[3]())  //3
```

阮一峰老师的是这么解释的:

> 上面代码中，变量`i`是`let`声明的，当前的`i`只在本轮循环有效，所以每一次循环的`i`其实都是一个新的变量，所以最后输出的是`6`。你可能会问，如果每一轮循环的变量`i`都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量`i`时，就在上一轮循环的基础上进行计算

babel转换为ES5是这样的:

```javascript
"use strict";

var arr = [];

function _loop(i) {
    arr[i] = function () {
        return i;
    };
};

for (var i = 0; i < 10; i++) {
    _loop(i);
}

console.log(arr[3]());
```

我的理解是创建一个函数，生成一个闭包存储`i`。所以每次遍历就能得到对应的`i`

关于闭包，我觉得还有一个很重要的题目

```javascript
function makeCounter() {
  var count = 0

  return function() {
    return count++
  };
}

var counter = makeCounter() 
var counter2 = makeCounter();
// 执行函数并赋值给counter，counter2
// counter， counter2对应的是不同的函数
// 他们都初始化了一个 count = 0
// 之前按引用类型指针理解，所以错了

console.log( counter() ) // 0
console.log( counter() ) // 1

console.log( counter2() ) // 0
console.log( counter2() ) // 1
```



## let 命令总结

1. 块级作用域

   ```javascript
   {
       let a = 10;
       var b = 1;
   }
   a // a is not defined
   b // 1
   ```

2. 在全局作用域下let声明的变量并不会绑定到window上

   ```javascript
   let a = 1;
   var b = 2;
   window.a //undefined
   window.b //2
   ```

3. for循环除了上文提及的特性，还有一个特点就是父子作用域

   ```javascript
   for (let i = 0; i < 3; i++){
       let i = 'abc';
       console.log(i)
   }
   
   ```

4. 不存在变量提升。所以一定要先声明再使用

5. 暂时性死区
   ES6 明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

   ```javascript
   var tmp = 123;
   if (true){
       tmp = 'abc'; //ReferenceError
       let tmp;
   }
   ```

6. 不可重复声明

## const 命令总结

1. const声明一个只读的常量。改变值会报错，只声明不赋值也会报错。其他特性与let相同

2. **本质**：`const`实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于引用类型来说，保存的只是指针。所以可以改变对象，但不能改变指针

   ```javascript
   const foo = {}
   foo.prop = 123  //ok
   foo = {} //报错
   ```



参考资料



[阮一峰-es6入门](http://es6.ruanyifeng.com/#docs/let)

[怎么理解for循环中用let声明的迭代变量每次是新的变量？](https://segmentfault.com/q/1010000007541743)

























