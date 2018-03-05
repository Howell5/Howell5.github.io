---
title: JavaScript数据结构与算法（一）
date: 2017-03-22
---
> 作为软件开发者，我们要能够用编程语言和数据结构来解决问题。对于任何方向的编程人员来说数据结构和算法都是极其重要的，称之为程序的灵魂也不为过。然许多前端人员对由于错误的理解，认为前端不需学习算法，只需要熟练的实现网页布局和特效，便可万事大吉。首先持这种想法的前端人员显然是认为自己还是10年前的前端。面对如今日新月异的技术发展和前端职责。如果抱着这种思想工作的话，离被行业淘汰也就不远了。本系列将简要介绍JavaScript实现基本的数据结构和常见的算法。也是自己对这方面的知识做个简单的整理。

<!--more-->
> 本书主要参考[《学习JavaScript数据结构与算法》](https://book.douban.com/subject/26639401/)一书写成。文章如有错误，欢迎指出。

## JavaScript中的数组
> 几乎所有编程语言都原生支持数组类型，它是一段线性分配的内存，通过整数计算偏移并访问其中的元素。但JavaScript中的数组和我们常见如Java中的数组有些不同，众所周知，“JavaScript中，一切皆为对象”。JavaScript中的数组是一种拥有类数组（array-like）特性的对象，它比一个真正的数组慢，但它使用起来十分方便。

### 数组的创建与初始化

```
var arr = [];   // var arr = new Array();
```
其中很重要的点是与大多数其它语言不同，JavaScript数组的length是没有上界的。

```
var arr = [1, 2, 3];
arr[10] = 10;  // 如果添加元素，它会动态增长
console.log(arr.length);  //  11，也就是除了前3个和最后定义的10。中间的是8个undefined。
```
如果给数组的length设小，则将导致下标大于等于新length的属性被删除

```
var arr = [1, 2, 3, 4, 11];
arr.length = 2;
console.log(arr);  // [1, 2]
```
一个比较常见的坑是数组的排序

```
var arr1 = [1, 3, 11, 2]，
	 arr2 = ['Ana', 'ana', 'john', 'John'];
arr1.sort(); // 1, 11, 2, 3
arr2.sort(); // 'Ana', 'John', 'ana', 'john',
```
这是因为数组的排序方法sort是按字符串对应的ASCII值来比较的，所以上面变成这样的结果。
好在我们可以简单的修补这个问题

```
function compareNumber (a, b) {
	return a - b;
}
function compareString (a, b) {
  return a.toLowerCase() - b.toLowerCase();
}
arr1.sort(compareNumber); // 1, 11, 2, 3
arr2.sort(compareString); // 'Ana', 'ana', 'john', 'John'
```

数组的的常见增删方法就不一一介绍了，大家可以看[MDN Array的详细介绍](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)


### 二维数组及多维数组
JavaScript只支持一维数组，并不支持矩阵。但我们可以使用数组套数组实现多维数组。

```
var Matrix = [
  [1, 2, 2],
  [2, 5, 7],
  [2, 6, 8]
];
Matrix[2][1];  //6
```
JavaScript应该对矩阵提供更好的支持，我们可以自己定义一个。

```
Array.matrix = function (m, n, initial) {
	var a, i, j, mat = [];
	for (i = 0; i < m; i += 1) {
    a = [];
    for (j = 0; j < n; j += 1) {
      a[j] = initial;
    }
    mat[i] = a;
  }
  return mat;
};

var myMatrix = Array.matrix(4, 4, 0);
console.log(myMatrix[3][3]); // 0
```
## 栈
栈是一种遵从后进先出原则(LIFO,全称为Last In First Out)的有序集合。栈顶永远是最新的元素。

```
/**
 * 栈的构造函数
 */
function Stack() {

  /**
   * 用数组来模拟栈
   * @type {Array}
   */
  var items = [];

  /**
   * 将元素送入栈，放置于数组的最后一位
   * @param  {Any} element 接受的元素，不限制类型
   */
  this.push = function(element) {
    items.push(element);
  };

  /**
   * 弹出栈顶元素
   * @return {Any} 返回被弹出的值
   */
  this.pop = function() {
    return items.pop();
  };

  /**
   * 查看栈顶元素
   * @return {Any} 返回栈顶元素
   */
  this.peek = function() {
    return items[items.length - 1];
  }

  /**
   * 确定栈是否为空
   * @return {Boolean} 若栈为空则返回true,不为空则返回false
   */
  this.isAmpty = function() {
    return items.length === 0
  };

  /**
   * 清空栈中所有内容
   */
  this.clear = function() {
    items = [];
  };

  /**
   * 返回栈的长度
   * @return {Number} 栈的长度
   */
  this.size = function() {
    return items.length;
  };

  /**
   * 以字符串显示栈中所有内容
   */
  this.print = function() {
    console.log(items.toString());
  };
}
```
栈的实际应用比较多，书中有个十进制转二进制的函数。(不懂二进制怎么算的话可以百度)下面是函数的源代码。原理就是输入要转换的数字，不断的除以二并取整。并且最后运用while循环，将栈中所有数字拼接成字符串输出，不过仅是10转2就写一个函数，那又碰上了10转8，10转16呢？所以我们写一个复用函数处理这些。

```
/**
 * decNumber：要转化的10进制数，base: 需要转化为的进制（2， 8， 16）
 */
function baseConverter(decNumber， base) {

  var remStack = new Stack(),
    rem,
    binaryString = '',
    digits = '0123456789ABCDEF'; 

  while (decNumber > 0) {
    rem = Math.floor(decNumber % base);
    remStack.push(rem);
    decNumber = Math.floor(decNumber / base);
  }

  while (!remStack.isAmpty()) {
    binaryString += digits[remStack.pop()];
  }

  return binaryString;
};

console.log(baseConverter(100345, 2)); //输出11000011111111001
console.log(baseConverter(100345, 8)); //输出303771
console.log(baseConverter(100345, 8)); //输出187F9
```

## 队列
队列是一种遵从后进先出原则(FIFO,全称为First In First Out)的有序的项。队列在尾部添加新元素。最新添加的元素必须排在队列的末尾。

```
/**
 * 队列构造函数
 */
function Queue() {

  /**
   * 用数组来模拟队列
   * @type {Array}
   */
  var items = [];

  /**
   * 将元素推入队列
   * @param  {Any} ele 要推入队列的元素
   */
  this.enqueue = function(ele) {
    items.push(ele);
  };

  /**
   * 将队列中第一个元素弹出
   * @return {Any} 返回被弹出的元素
   */
  this.dequeue = function() {
    return items.shift()
  };

  /**
   * 查看队列的第一个元素
   * @return {Any} 返回队列中第一个元素
   */
  this.front = function() {
    return items[0];
  };

  /**
   * 确定队列是否为空
   * @return {Boolean} 若队列为空则返回true,不为空则返回false
   */
  this.isAmpty = function() {
    return items.length === 0
  };

  /**
   * 返回队列的长度
   * @return {Number} 队列的长度
   */
  this.size = function() {
    return items.length;
  };

  /**
   * 清空队列中所有内容
   */
  this.clear = function() {
    items = [];
  };

  /**
   * 以字符串显示队列中所有内容
   */
  this.print = function() {
    console.log(items.toString());
  };
}
```

*还有第二章的更新~*