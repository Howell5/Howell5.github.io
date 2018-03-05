---
title: JavaScript之深入之事件轮询
date: 2017-03-02
---
> 在详细聊event-loop前，我们先厘清几个基本概念：JavaScript是单线程的，即同一个时间只能做一件事，但浏览器中是会存在异步任务的，总不能按着正常顺序等到异步任务（万一异步任务是请求一个很大的文件，耗时很长）结束了，才去执行下一步事。这显然是反人类的。而我们平常打开的网页也没有这样反人类的体验，这说明JavaScript的运行机制解决了这点。

<!--more-->
## 执行栈与任务队列
当程序往下执行时，遇到了异步任务，主线程这时不管异步任务，挂起处于等待中的任务，先运行排在后面的任务。等到异步任务返回了结果，再回过头，把挂起的任务继续执行下去。
于是，所有任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

具体来说，异步执行的运行机制如下：
- 所有同步任务都在主线程上执行，形成一个[执行栈](http://www.ruanyifeng.com/blog/2013/11/stack.html)（execution context stack）。
- 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。
- 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
- 主线程不断重复上面的第三步。

![image.png](http://upload-images.jianshu.io/upload_images/2575359-77bbd6427c25c6fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Marcotask和Mircotask
上面所说的只是Event loop的基本知识，现在我们引入两个新概念 Marcotask 和 Mircotask ，在Event queue中不同的异步任务中，他们是被存入不同的queue中的，而且它们也并非简单的先进先出的执行的。

![image.png](http://upload-images.jianshu.io/upload_images/2575359-d8fe0a4efb36f750.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如上图所示，在单次的迭代中，event loop首先检查macrotask队列，如果有一个macrotask等待执行，那么执行该任务。当该任务执行完毕后（或者macrotask队列为空），event loop继续执行microtask队列。如果microtask队列有等待执行的任务，那么event loop就一直取出任务执行知道microtask为空。这里我们注意到处理microtask和macrotask的不同之处：**在单次循环中，一次最多处理一个macrotask（其他的仍然驻留在队列中），然而却可以处理完所有的microtask。**

当microtask队列为空时，event loop检查是否需要执行UI重渲染，如果需要则重渲染UI。这样就结束了当次循环，继续从头开始检查macrotask队列。

- macrotasks: setTimeout setInterval setImmediate I/O UI渲染
- microtasks: Promise process.nextTick Object.observe MutationObserver

拿个例子帮助我们理解这一点：
```
(function () {
    setTimeout(function() {console.log(4)}, 0);
    new Promise(function executor(resolve) {
        console.log(1);
        for( var i=0 ; i<10000 ; i++ ) {
            i == 9999 && resolve();
        }
        console.log(2);
    }).then(function() {
        console.log(5);
    });
    console.log(3);
})()
```
分析过程：
1. 当前task运行，执行代码。首先setTimeout的callback被添加到tasks queue中；
2. 实例化promise，输出 1; promise resolved；输出 2;
3. promise.then的callback被添加到microtasks queue中；
4. 输出 3;
5. 已到当前task的end，执行microtasks，输出 5;
6. 执行下一个task，输出4。

## Web Worker
HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM，并未改变JavaScript单线程事实。

专用web worker例子:

```
// html
    <form>
      <div>
        <label for="number1">Multiply number 1: </label>    
        <input type="text" id="number1" value="0">
      </div>
      <div>
        <label for="number2">Multiply number 2: </label>   
        <input type="text" id="number2" value="0">
      </div>
    </form>

    <p class="result">Result: 0</p>
<script>
var first = document.querySelector('#number1');
var second = document.querySelector('#number2');

var result = document.querySelector('.result');

if (window.Worker) { // Check if Browser supports the Worker api.
	var myWorker = new Worker("worker.js");

	first.onchange = function() {
	  myWorker.postMessage([first.value,second.value]); // Sending message as an array to the worker
	  console.log('Message posted to worker');
	};

	second.onchange = function() {
	  myWorker.postMessage([first.value,second.value]);
	  console.log('Message posted to worker');
	};

	myWorker.onmessage = function(e) {
		result.textContent = e.data;
		console.log('Message received from worker');
	};
}
</script>

//work.js

onmessage = function(e) {
  console.log('Message received from main script');
  var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
  console.log('Posting message back to main script');
  postMessage(workerResult);
}
```
从上面代码可以看到，两者通过postMessage传递数据。

## 参考文章
[JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

[从Promise来看JavaScript中的Event Loop、Tasks和Microtasks](https://github.com/creeperyang/blog/issues/21)

[HTML系列：macrotask和microtask](https://zhuanlan.zhihu.com/p/24460769)