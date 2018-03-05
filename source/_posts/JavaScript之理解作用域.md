---
title: JavaScript之理解作用域
date: 2016-07-09
---

>在javascript的学习中，执行环境、作用域是2个非常非常重要和基本的概念，理解了这2个概念对于javsacript中很多脚本的运行结果就能明白其中的道理了，比如搞清作用域和执行环境对于闭包的理解至关重要。

<!--more-->
## 一、执行环境（exection context,也有称之为执行上下文）
>执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为。每个执行环境都有一个与之关联的**变量对象** (variable object)，环境中定义的所有变量和函数都保存在这个对象中。虽然我们编写的代码无法访问这个对象，但解析器会处理数据时会在后台使用它。                                  ---《JavaScript高程》

当这个执行环境是函数的时候，这个变量对象就是函数的活动对象。活动对象最开始时只包含一个变量，即 arguments 对象（这个对象在全局环境中是不存在的）。
## 二、作用域链（scope chain）
了解了变量对象之后，理解作用域链就容易了。
>在代码在一个环境执行时，会创建变量对象的一个 **作用域链**。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。

**整个作用域链是由不同执行位置上的变量对象（Variable Object）按照规则所构建一个链表。**

作用域链的前端，始终都是当前执行的代码所在环境的变量对象。下一个变量对象来自包含（外部）环境，而下一个变量对象则来自下一个包含对象。这样，一直延续到全局执行环境；全局执行环境的变量对象始终都是作用域链的最后一个对象。

引用高程中的例子：
```
 var color="blue";
 function changecolor(){
    var anothercolor="red";
    function swapcolors(){
	var tempcolor=anothercolor;
	anothercolor=color;
	color=tempcolor;
       // Todo something		
     }
    swapcolors();
}
changecolor();
 //这里不能访问tempcolor和anocolor;但是可以访问color;
alert("Color is now  "+color);
```
上面例子涉及了3个执行环境：全局变量、changeColor()的局部变量环境和swapColors()的局部变量环境。swapcolor（）的局部环境开始先在自己的Variable Object中搜索变量和函数名，找不到，则向上搜索changecolor作用域链。。。。。以此类推。但是，changecolor()函数是无法访问swapcolor中的变量。

![作用域链](http://upload-images.jianshu.io/upload_images/2575359-31c07041f11d1e92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
***
通过上面的分析，我们可以得知内部环境可以通过作用域链访问所有的外部环境，但外部环境不能访问内部环境中的任何变量和函数。这些环境之间是线性、有次序的。每个环境都可以向上搜索作用域链，以便查询变量和函数名；

tip:通过下面两种情况，即当执行流进入下列任何语句时，作用域链就会得到加长：
- try-catch 语句的 catch 块;
- with 语句。

## 三、没有块级作用域（在 let 出现以前）

看两个例子

```
情况1：
for (var i = 0;i<10;i++) {
  // do something
}
console.info(i);  // 10
情况2：
for (let i = 0;i<10;i++) {
  // do something
}
console.info(i);  // error
```

从上面的例子中可以看出，JavaScript没有像其他类C的语言（由花括号封闭的代码块都有自己的作用域）一样。在使用for语句时尤其记住这点。

## 四、声明变量

使用var声明变量时，这个变量将被自动添加到距离最近的可用环境中。对于函数而言，自然声明的变量就会被添加到函数的局部环境中，变量在整个函数环境内都是可用的。

   但是，如果变量没有是用var进行声明，将会被添加到全局环境，也就是说成位全局变量了。（只有当执行流执行到该语句时，才会被添加到全局变量的变量对象中）

看一个例子：
```
情况1：
var x = 1;
function rain() {
  console.info(x);  // 1
}
rain(); 
情况2：
var x = 1;
function rain() {
  console.info(x);  //undefined
  var x = 10;
  console.info(x); // 10 
}
rain(); 
```

情况1中得到的1 和 情况2中得到的10，通过上述作用域链的关系，我们较为容易理解。那情况2中的 undefined 是怎么回事儿呢？答案是预解析，我们看下面一段代码，和上面的代码等价

```
var x = 1;
function rain() {
  var x;
  console.info(x);  //undefined
  x = 10;
  console.info(x); // 10 
}
rain(); 
```
因为JavaScript引擎会预解析代码中变量和函数的定义。导致调用x的时候，局部执行环境已经有了x，就不向上继续寻找x，也就访问不到全局变量的x。而此时局部变量x还未赋值，这时候输出也就是 undefined 了。