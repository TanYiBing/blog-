---
title: Promise对象
date: 2018-09-11 16:07:08
tags:
- JavaScript 
- ECMAScript6
categories:
- ECMAScript6
---
# Promise对象

## Promise的介绍
`Promise`在`ECMAScript6`中被正式写进了语言标准，这是异步编程的一种解决方案，较之传统的函数回调和事件，`Promise`来得更加强大。
`Promise`被理解成一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。
`Promise`有三个状态，分别是：

* `pending`：进行中
* `fulfilled`：已成功
* `rejected`：已失败

这三个状态是根据异步操作的结果来决定的，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。这三个状态一旦确定就不会在改变了。

## Promise的使用
### 创建
怎么创建一个Promise实例呢：

	const promise = new Promise(function(resolve, reject) {
		// ... some code

		if (/* 异步操作成功 */){
			resolve(value);
		} else {
			reject(error);
		}
	});


其中两个方法，`resolve`用来处理异步操作成功的结果，将其结果传递出去，在后面的`then`方法中去接受，`reject`用来处理操作失败的结果并传递出去，也可用`then`方法接收。

### then方法的使用
`Promise`实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数：

	promise.then(function(value) {
		// success
	}, function(error) {
		// failure
	});

其中第二个函数是可选的。这两个函数就是用来接收上面传出来的结果的。而且需要注意的是**then方法返回是一个新的Promise，也就是说我们可以进行链式编程。**

### Promise执行顺序
`Promise`在新建之后就会立即执行，举个例子：
	
	let promise = new Promise(function(resolve, reject) {
		console.log('a');
		resolve();
	});

	promise.then(function() {
		console.log('b');
	});

	console.log('c');

	// a
	// c
	// b

上面的代码先输出a，因为Promise新建之后就立即执行，`then`方法在当前脚本所有同步任务执行完才执行，最后才会输出b。

## Promise和其他技术
昨天我看了RxJS，发现这两货好像有点像，一个是把所有数据都变成流进行操作。一个是一直返回`Promise`，通过`then`方法一直链式操作，那就比较一下吧。

|操作|可观察对象|承诺|
|:----------:|---------|----------|
|创建|new Observable((observer) => {observer.next(123);});|new Promise((resolve, reject) => {resolve(123);});|
|转换|obs.map((value) => value * 2 );|promise.then((value) => value * 2);|
|订阅|sub = obs.subscribe((value) => {console.log(value)});|promise.then((value) => {console.log(value);});|
|取消订阅|sub.unsubscribe();|承诺被解析时隐式完成。|

## Promise的应用
### 加载图片
我们可以将图片的加载写成一个Promise，一旦加载完成，Promise的状态就发生变化。

	const preloadImage = function (path) {
		return new Promise(function (resolve, reject) {
    		const image = new Image();
    		image.onload  = resolve;
    		image.onerror = reject;
    		image.src = path;
		});
	};