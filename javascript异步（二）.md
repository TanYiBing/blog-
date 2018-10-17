---
layout: w
title: javascript异步（二）
date: 2018-09-17 14:40:16
tags:
- JavaScript 
categories:
- JavaScript
---
# JavaScript异步（二）————Promise

## ES6中的Promise

### 写一个传统的异步操作
我们先写一段异步的代码，然后用promise来封装一下：

	var wait = function () {
    	var task = function () {
        	console.log('执行完成')
    	}
    	setTimeout(task, 2000)
	}
	wait()
### 用Promise进行封装

	const wait =  function () {
    	// 定义一个 promise 对象
    	const promise = new Promise((resolve, reject) => {
        	// 将之前的异步操作，包括到这个 new Promise 函数之内
        	const task = function () {
            	console.log('执行完成')
            	resolve()  // callback 中去执行 resolve 或者 reject
        	}
        	setTimeout(task, 2000)
    	})
    	// 返回 promise 对象
    	return promise
	}

我们首先将之前传统的异步方法用`new Promise((resolve, reject) => {...})`包装起来，然后return就行了。异步操作的内部，在callback中执行`resolve()`（表明成功了，失败的话执行reject）。

我们接下来怎么处理返回的数据呢：
	
	const w = wait()
	w.then(() => {
    	console.log('ok 1')
	}, () => {
    	console.log('err 1')
	}).then(() => {
    	console.log('ok 2')
	}, () => {
    	console.log('err 2')
	})

我们调用`then`方法，这个方法接收两个参数，第一个在成功时（触发resolve）执行，第二个在失败时(触发reject)时执行。而且，then还可以进行链式操作。

以上就是ES6中Promise的基本使用方法，接下来看一些Promise的常见用法吧。

## Promise在ES6中的常见用法

为了方便使用，我们首先封装个Promise，为后面使用：

	const fs = require('fs')
	const path = require('path')  // 后面获取文件路径时候会用到
	const readFilePromise = function (fileName) {
    	return new Promise((resolve, reject) => {
        	fs.readFile(fileName, (err, data) => {
            	if (err) {
                	reject(err)  // 注意，这里执行 reject 是传递了参数，后面会有地方接收到这个参数
            	} else {
                	resolve(data.toString())  // 注意，这里执行 resolve 时传递了参数，后面会有地方接收到这个参数
            	}
        	})
    	})
	}

### 参数传递

我们要使用上面封装的readFilePromise读取一个 json 文件../data/data2.json，这个文件内容非常简单：{"a":100, "b":200}

先将文件内容打印出来，代码如下。大家需要注意，readFilePromise函数中，执行`resolve(data.toString())`传递的参数内容，会被下面代码中的data参数所接收到。

	const fullFileName = path.resolve(__dirname, '../data/data2.json')
	const result = readFilePromise(fullFileName)
	result.then(data => {
    	console.log(data)
	})

参数传递就是在异步里面通过resolve来传递至，然后就会被第一个`then`处理时接收到，而且通过`then`的链式操作可以继续将处理后的值传递下去。

### 异常捕获
我们知道then会接收两个参数（函数），第一个参数会在执行resolve之后触发（还能传递参数），第二个参数会在执行reject之后触发（其实也可以传递参数，和resolve传递参数一样），但是上面的例子中，我们没有用到then的第二个参数。这是为何呢 ———— 因为不建议这么用。

对于Promise中的异常处理，我们建议用catch方法，而不是then的第二个参数。请看下面的代码，以及注释。

	const fullFileName = path.resolve(__dirname, '../data/data2.json')
	const result = readFilePromise(fullFileName)
	result.then(data => {
    	console.log(data)
    	return JSON.parse(data).a
	}).then(a => {
    	console.log(a)
	).catch(err => {
    	console.log(err.stack)  // 这里的 catch 就能捕获 readFilePromise 中触发的 reject ，而且能接收 reject 传递的参数
	})

在若干个then串联之后，我们一般会在最后跟一个.catch来捕获异常，而且执行reject时传递的参数也会在catch中获取到。这样做的好处是：

*	让程序看起来更加简洁，是一个串联的关系，没有分支（如果用then的两个参数，就会出现分支，影响阅读）
*	更像是try - catch的样子，更易理解

### 串联多个异步操作

如果现在有一个需求：先读取data2.json的内容，当成功之后，再去读取data1.json。这样的需求，如果用传统的callback去实现，会变得很麻烦。而且，现在只是两个文件，如果是十几个文件这样做，写出来的代码就没法看了（臭名昭著的callback-hell）。但是用刚刚学到的Promise就可以轻松胜任这项工作：

	const fullFileName2 = path.resolve(__dirname, '../data/data2.json')
	const result2 = readFilePromise(fullFileName2)
	const fullFileName1 = path.resolve(__dirname, '../data/data1.json')
	const result1 = readFilePromise(fullFileName1)

	result2.then(data => {
    	console.log('data2.json', data)
    	return result1  // 此处只需返回读取 data1.json 的 Promise 即可
	}).then(data => {
    	console.log('data1.json', data) // data 即可接收到 data1.json 的内容
	})

上文“参数传递”提到过，如果then有链式操作，前面步骤返回的值，会被后面的步骤获取到。但是，如果前面步骤返回值是一个Promise的话，情况就不一样了。如果前面返回的是Promise对象，后面的then将会被当做这个返回的Promise的第一个then来对待。

### Promise.all & Promise.race

现在我需要一起读取data1.json和data2.json这两个文件，等待它们全部都被读取完，再做下一步的操作。此时需要用到Promise.all：

	// Promise.all 接收一个包含多个 promise 对象的数组
	Promise.all([result1, result2]).then(datas => {
    	// 接收到的 datas 是一个数组，依次包含了多个 promise 返回的内容
    	console.log(datas[0])
    	console.log(datas[1])
	})

读取两个文件data1.json和data2.json，现在我需要一起读取这两个文件，但是只要有一个已经读取了，就可以进行下一步的操作。此时需要用到Promise.race：

	// Promise.race 接收一个包含多个 promise 对象的数组
	Promise.race([result1, result2]).then(data => {
    	// data 即最先执行完成的 promise 的返回值
    	console.log(data)
	})

