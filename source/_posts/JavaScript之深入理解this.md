---
title: JavaScript之深入理解this
date: 2016-12-01
---
> 此文参考 [MDN_this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) 写成

## 定义
this是函数运行时自动生成的内部对象，即调用函数的那个对象。（不一定很准确的定义，但还算通俗易懂）
 在大多数情况下，this的值由函数调用方式决定，它不能在执行期间赋值来设置，它在每次执行下可能都有不同的值。
 
<!--more-->
## 全局执行环境（outside function）
在全局执行环境中，this一直指向全局对象（global object），不管是在严格模式还是在非严格模式中。

**代码1**
```
console.log(this.document === document);   //true

// 在浏览器中，window对象也就是全局对象（global object）
console.log(this === window);   //true

this.a  = 37;
console.log(window.a);   //37
```
## 函数执行环境（inside function）
在函数执行环境中，this的值取决于函数的调用方式。

**而函数的调用方式主要有4种：**

- 函数直接调用
- 对象方法调用
- 构造函数调用
- call / apply / bind
- 箭头函数（ES6）

### 函数直接调用

下面的代码在非严格模式执行时，this的值会指向全局对象；而在严格模式中，this的值将会默认为undefined。

**代码2**
```javascript
/* 非严格模式 */
function f1 () {
  return this;
}
console.log(f1() === window);   //true

// in node;
console.log(f1() === global);   //true

/* 严格模式 */
function f2 () {
  'use strict'
  return this;
}
console.log(f1() === undefined);   //true
```
----------------------------------
### call / apply / bind 改变this的指向
#### call / apply
call和apply的用法很像，只是后面参数的传入形式不同。

**代码3**
```
function add(c, d) {
  return this.a + this.b + c + d;
}

var o = {a: 1, b: 3};

// call的第一个参数 是对象，也就是this的指向对象。后面的参数就是函数arguments对象的成员
add.call(o, 5, 7); // 1 + 3 + 5 + 7 = 16

// call的第一个参数 是对象，也就是this的指向对象。后面的参数是数组，数组里的成员也就是函数arguments对象成员
add.apply(o, [10, 20]); // 1 + 3 + 10 + 20 = 34
```
**使用call和apply时需要注意的是**，当传入的第一个参数的值不是对象时，JavaScript会尝试使用ToObject 操作将其转化为对象。

**代码4**
```
function bar() {
  console.log(Object.prototype.toString.call(this));
}

bar.call(7); // [object Number]
```
#### bind 方法
ECMAScript 5 引入了 [Function.prototype.bind](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/bind)。调用f.bind(someObject)会创建一个与f具有相同函数体和作用域的函数，但是在这个新函数中，this将永久地被绑定到了bind的第一个参数，无论这个函数是如何被调用的。

**代码5**
```
function f() {
  return this.a;
}

var g = f.bind({a: 'azerty'}); //生成一个绑定函数g
console.log(g()); // azerty

var o = {a: 10, f: f, g: g};
console.log(o.f(), o.g());   //10, azerty

//需要注意的是，绑定函数不可以再bind
var h = g.bind({a: 'foo'});
console.log(h());  //azerty，不会变成foo
```
-------------------------------------
### 对象方法调用
当以对象里的方法的方式调用函数时，它们的 this 是调用该函数的对象.

下面的例子中，当 o.f() 被调用时，函数内的this将绑定到o对象。
```
var prop = 36;
var o = {
  prop: 37,
  bar: function() {
    return this.prop;
  }
};
console.log(o.bar());  //37
```
### 构造函数调用

**代码6**
```
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.introduce = function () {
    console.log('My name is ' + this.name + ', I\'m ' + this.age);
  };
}
var Joseph = new Person('Joseph', 19);
Joseph.introduce();  // "My name is Joseph, I'm 19"
```
由上述代码可以清晰的看到this与被新创建的对象绑定了。

*注意*：当构造器返回的默认值是一个this引用的对象时，可以手动设置返回其他的对象，如果返回值不是一个对象，返回this。（这句话看起来比较难理解，我们看下一个例子）。

**代码7**
```
function Fn2() {
  this.a = 9;  // dead code
  return {a: 10};
}
var o = new Fn2();
console.log(o.a);  // 10
```
这个例子说明了当构造函数返回的是一个对象的话，此时this的值会变成此时返回的对象。‘this.a = 9’成了僵尸代码。

### 箭头函数
在箭头函数（ [Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)）中，this的值是封闭执行环境决定的。在全局环境中，那么被赋值为全局对象。
```
var globalObject = this;
var foo = (() => this);
console.log(foo() === globalObject); // true
```
更重要的是它与其他情况不同的是，不管函数如何调用，上面this的值一直都是全局对象。call / bind 也不能改变它的值。

**代码8**
```
// 作为对象方法被调用
var obj = {foo: foo};
console.log(obj.foo() === globalObject); // true

// 尝试用 call 改变this的值
console.log(foo.call(obj) === globalObject); // true //this的值并未变成obj

// 尝试用 bind 改变this的值
foo = foo.bind(obj);
console.log(foo() === globalObject); // true
```

--------------------------------
## 案例
本文知识点都看完了。让我们看几个案例，检查自己的掌握情况。

例1
```
var prop = 36;
var o = {
  prop: 37,
  bar1: function() {
    function foo1() {
      return this.prop;
    }
    return foo1;
  },
  bar2: function() {
    var foo2  = (() => this.prop); //ES6箭头函数
    return foo2;
  }
};
console.log('result1:'+o.bar1()()); // result1 ?
console.log('result2:'+o.bar2()()); // result2 ?
var fn2 = o.bar2;
console.log('result3:'+fn2()()); // result3 ?
```
先揭晓答案：例1 result1 = 36，result2 = 37，result3 = 36。我的理解是，在result1中，o.bar1()执行导致foo函数return到了全局环境中，然后执行就变成了在全局中执行，所以得到的是全局中36的值。result2呢？因为this在箭头函数中。它的值不会改变。所以this仍指向o。那为什么result3又重新变了呢？因为此时‘var fn2  = o.bar2’相当于重新定义了一个函数，而this的值当然也就变为了全局对象。
```
// 相当于这样
var fn2 = function() {
    function foo1() {
      return this.prop;
    }
    return foo1;
  }
fn2()();
```

例2
```
function sum(a,b) {
 return a+b;
};
var o = {
  num: 1,
  fn: function() {
        function handle() {
          return this.num = sum(this.num, this.num);
        }
    handle();
  }
};
console.log('result:'+o.fn());  // result ?
```
同样先揭晓答案：result = undefined，用控制台可以看到此时this指向window，而不是o。这是个比较容易掉进去的坑（一般认为是当初的语言设计错误，被人诟病不少）。看似函数是由对象方法调用的，其实细心的话，我们可以看到。handle函数的执行，前面的没有对象的。这种情况下，this指向全局对象。解决办法也很简单。
```
// 1、取消 handle函数的定义，直接在对象的方法中使用this
fn2: function() {
    this.value = sum(this.value, this.value);  //2
},
///2、使用变量保存外部函数的this。
fn3: function() {
    var that = this;   // that == o
    function handle() {
        that.value = add(that.value, that.value);
    }
    handle();
}
```
---------------------------

参考文章：
- [Javascript中this与闭包学习笔记](https://segmentfault.com/a/1190000006150835)
- [理解JS中的this](https://segmentfault.com/a/1190000004905145)