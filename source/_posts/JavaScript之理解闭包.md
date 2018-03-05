---
title: JavaScript之理解闭包
date: 2016-10-10
---
## 一个简单的闭包

例1
```
function sayHello(name) {
  var text = 'Hello ' + name;
  var say = function() { console.log(text); }  
  //say()是个内部函数，
  //a closures use variable declared in the parent function 
  say();
}
sayHello('Joe');
```

<!--more-->
## 理解闭包
闭包是指在 JavaScript 中，内部函数总是可以访问其所在的外部函数中声明的参数和变量，即使在其外部函数被返回（寿命终结）了之后。

闭包是一个集合（combination）；它由两部分组成：函数，以及**创建**该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。可以赋值给某个变量，可以作为参数传递给函数，也可以作为一个函数返回值返回。

在我们的例子中say就是个闭包，因为它含有创建函数的环境，而环境中有着被定义的变量 text，所以它能使用该变量。

## 闭包的用途
 可以访问函数内部的量，闭包允许将函数与其所操作的某些数据（环境）关连起来。这显然类似于面向对象编程。在面对象编程中，对象允许我们将某些数据（对象的属性）与一个或者多个方法相关联。

因而，一般说来，可以使用只有一个方法的对象的地方，都可以使用闭包

例2
```
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);  //闭包
var add10 = makeAdder(10);  //闭包

console.log(add5(2));  // 7
console.log(add10(2)); // 12
```

例3
```
var invrement,decrement,value;

var makeCounter = function() {
  var privateCounter = 0;
  increment = function (){
    privateCounter += 1;
  };
  decrement = function (){
    privateCounter -= 1;
  };
  setCounter = function (x) {
    privateCounter = x;
  };
  value = function () {
    return privateCounter;
  };
};

makeCounter();
increment();
console.log(value());  //1

setCounter(5);
console.log(value());  //5

var oldCounter = value;

makeCounter();
decrement();
console.log(value());  //-1

console.log(oldCounter());  //5
```
从例3我们可以看到，setupSomeGlobals每次执行，都会重新创建一个闭包。而且两个闭包互不影响。即它们具有独立性。

我们将例3稍稍改变一下，就可以看到如何用闭包**模仿私有方法**

例4
```
var makeCounter = function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    setCouter: function(x) {
      privateCounter = x;
    },
    value: function() {
      return privateCounter;
    }
  }  
};

var Counter1 = makeCounter();
var Counter2 = makeCounter();
console.log(Counter1.value()); /* logs 0 */
Counter1.increment();
Counter1.increment();
console.log(Counter1.value()); /* logs 2 */
Counter1.decrement();
console.log(Counter1.value()); /* logs 1 */
console.log(Counter2.value()); /* logs 0 */
```
在例4中，changeBy就是一个私有方法，无法从外部直接调用。




另一特点就是 保存变量。这是通俗的说法。

从根本上说，闭包是函数开始执行的时候被分配的一个[栈帧](http://baike.baidu.com/link?url=x9za8fl-K8Gsdc0IFBbC5fTininX3H8qVBuSPsChIJd8bmzTRXvd8scDL1uCYKLS26m6GMbXgHFC5K8yXz7nZ3eImibufpfwiBWzlBDAyT_)，在函数执行结束返回后仍不会被释放(就好像一个栈帧被分配在堆里而不是栈里！)


例5 
```
function say667() {
  var num = 42;
    num+=1;
  
  var say3 = function() {
    console.log(num+=1);
  };
  say3();
  var say4 = function() {
    console.log(num+=1);
  };
  return say4;
}
say667(); //44
say667(); //44，并未自增，是因为这又重新创建了一个闭包。
var sayt = say667(); //44
sayt(); //45
sayt(); //46， 自增了，是因为say667并未重新创建闭包。
         //sayt引用了里面的闭包，而且造成了num的值被保存了。在函数执行完后并未被销毁。
```
> 注意：正是因为闭包有这些特点，它给我们带来了便利。但我们仍需要
小心使用。它有两个缺点：
- 1、由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除；
- 2、闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。

## 闭包的循环陷阱及补救方法

例6
```
var arr = [];
for(var i=0;i<3;i++) {
 arr.push(function(){console.log(i);});
 } 
arr[0](); //?
arr[1](); //?
arr[2](); //?
```
例6输出的结果：三个 "3" ，详情解析可以看这个[知乎问答](https://www.zhihu.com/question/33468703)，里面有人解释的很清楚。简要来说就是，由于执行顺序的关系，在三个闭包执行前，for循环早已循环完毕（for循环没有块级作用域），i 已经自增到3。到闭包执行时，取到的i自然是3。

解决方案：
```
//方案一
for(var i=0,arr=[];i<3;i++) {
		arr.push(
  //既然是执行顺序的问题，那我们给闭包加上一个自执行函数
  //内部函数引用自执行函数传入的 i ，这里的 i
 是正常的
			(function(i){
				return function(){
					console.log(i);
				}
			})(i)
		);
	}
arr[0](); //0
arr[1](); //1
arr[2](); //2

//方案2
for(let i=0,arr=[];i<3;i++)
//把 i 的定义换成 let 就行，因为 let 定义的变量具有块级作用域，同样解决了这个问题
```

## 总结：
- 闭包是指在 JavaScript 中，内部函数总是可以访问其所在的外部函数中声明的参数和变量，即使在其外部函数被返回（寿命终结）了之后。
- 每当你在另一个函数里使用了关键字function，一个闭包就被创建了
- JavaScript中的闭包，就像一个副本，将某函数在退出时候的所有局部变量复制保存其中。
- 使用闭包前，需要性能考量

-------------------------------------

参考文章：
- [闭包 ---MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures#Performance_considerations)
- [Javascript中this与闭包学习笔记](https://segmentfault.com/a/1190000006150835)
- [JavaScript 闭包入门（译文）](https://gold.xitu.io/post/58832fe72f301e00697b672d)
- [JavaScript的闭包](http://imweb.io/topic/566567e4d91952db73b41f5d)
- [js关于for循环中的闭包问题？](https://www.zhihu.com/question/33468703)