---
title: JavaScript的异步编程
date: 2017-03-08
---
> 写JavaScript代码少不了处理回调函数，如果只是很少的回调嵌套，倒也还好。但是遇到需要用了过多回调嵌套的时候，就会很麻烦（主要是语法层面的难以阅读和不支持异常的捕捉），为了解决这一麻烦，主要有以下几种方法。

<!--more-->
## Promise
Promise 是 ES6 引入的解决办法，表示一个**现在**或**将来**可用，亦或**永远不**可用的值。把原来嵌套的回调改为了级联的方式
如果说原始的回调函数是这样

![image.png](http://upload-images.jianshu.io/upload_images/2575359-7141d9b562ca062f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么Promise的示意图就是这样

![image.png](http://upload-images.jianshu.io/upload_images/2575359-2f8d767ce860d5db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
var a = new Promise(function(resolve, reject) {
  setTimeout(function() {
      resolve('a')
  }, 2000)
})


function b(val) {
  console.log(val)

  return new Promise(function(resolve, reject) {
    setTimeout(function() {
        resolve('b')
    }, 2000)
  })
}


a.then(b)
 .catch(r1 => console.log('rejected' + r1))
 .then(function(val) {
    console.log(val)
  })
 .catch(r2 => console.log('rejected' + r2))
// 等2秒输出 "a"，再等2秒输出 "b"
```
我们只需要 return 一个 Promise 即可实现这种多个异步操作的顺序执行。

粗略来看，这是一个比较优雅的异步解决方案了，并且在 Promise 中我们也可以实现分级的 catch。

## Generator
Generator 同样也是 ES6 出现的新语法简单来说，Generator 可以理解为一个可以遍历的状态机，调用 next 就可以切换到下一个状态。

在 JavaScript 中，Generator 的 function 与 函数名之间有一个 *， 函数内部使用 yield 关键词，定义不同的状态。
```
function* anotherGenerator(i) {
  yield i + 1;
  yield i + 2;
  yield i + 3;
}

function* generator(i){
  yield i;
  yield* anotherGenerator(i);
  yield i + 10;
}

var gen = generator(10);

console.log(gen.next().value); // 10
console.log(gen.next().value); // 11
console.log(gen.next().value); // 12
console.log(gen.next().value); // 13
console.log(gen.next().value); // 20
```
调用一个**生成器函数**并不马上执行它的主体，而是返回一个这个生成器函数的**迭代器（iterator）对象**。当这个迭代器的next()方法被调用时，**生成器函数**的主体会被执行直至第一个[yield
](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield)表达式，该表达式定义了迭代器返回的值，或者，被 [yield*
](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield*)委派至另一个生成器函数。next()方法返回一个对象，该对象有一个value属性，表示产出的值，和一个done属性，表示生成器是否已经产出了它最后的值。

## await/async
前面的两种似乎都不是最完美的写法，总有这样那样的问题。而 ES7 带来了 await/async，它是基于 Promise 的，它解决了 Promise 的一个痛点 --- 每一个 then 方法内部是一个独立的作用域，要是想共享数据，就要将部分数据暴露在最外层，在 then 内部赋值一次。
```
const f1 = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(111);
    }, 2000);
  });
};

const f2 = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(222);
    }, 3000);
  });
};

const testAsync = async () => {
  try {
    const t1 = await f1();
    console.log(t1);
    const t2 = await f2();
    console.log(t2);
  } catch (err) {
    console.log('err ' + err);
  }
};

testAsync();
```
是不是就像是在写同步代码一样，可读性好。和try catch的配合使用，捕捉异常也很自然
