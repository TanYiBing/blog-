---
title: 封装个Koa
date: 2018-09-13 16:39:37
tags: 
- Node 
- Koa 
categories:
- Node
---
想了解下`Koa`是怎么个骚操作，之前使用的是`Koa2`，里面是`async`和`await`，用起来爽歪歪。不过，`async`和`await`是ES7里面的东东，还是有必要看一下koa的第一代的，发现里面的中间件是`Generator`，其实我对这个也不是很理解，ES6里面新增加的，一下子有点难接受，今天就提上日程，看一看怎么搞，自己封装个Koa试试！

想要真正搞懂这个`Generator`还是比较烦，我自己感觉整个ES6的主要部分应该就是这个Generator了，相知带它怎么回事先要知道`Iterator`、然后再看`Generator`，但是想和异步扯上关系你还要看`Chunk`函数，还要看看`co`库，当然啦，`Promise`肯定要知道，所以感觉这也不是个简单的东西，门道多着呢。

不说废话了，上代码吧，封装个`MyKoa`：

	class MyKoa extends Object {
    	constructor(props) {
        	super(props);

        	// 存储所有的中间件
        	this.middlewares = []
    	}

    	// 注入中间件
    	use (generator) {
        	this.middlewares.push(generator)
    	}

    	// 执行中间件
    	listen () {
        	this._run()
    	}

    	_run () {
        	const ctx = this;
        	const middlewares = ctx.middlewares;
        	co(function* () {
            	let prev = null
            	let i = middlewares.length
            	//从最后一个中间件到第一个中间件的顺序开始遍历
            	while (i--) {
                	// ctx 作为函数执行时的 this 才能保证多个中间件中数据的共享
                	//prev 将前面一个中间件传递给当前中间件，才使得中间件里面的 next 指向下一个中间件
                	prev = middlewares[i].call(ctx, prev);
            	}
            	//执行第一个中间件
            	yield prev;
        	})
    	}
	}

然后可以试验下效果：

	var app = new MyKoa();
	app.use(function *(next){
    	this.body = '1';
    	yield next;
    	this.body += '5';
    	console.log(this.body);  // 12345
	});
	app.use(function *(next){
    	this.body += '2';
    	yield next;
    	this.body += '4';
	});
	app.use(function *(next){
    	this.body += '3';
	});
	app.listen();

**其实Generator和我们的回调处理异步的callback根本没有一点关系，它的本质是‘暂停’，并将执行权转移，我们只是给它穿上了一层又一层的外套，强行让它和异步‘发生关系’。**

**PS：立个flag，2018年过年之前找个机会，我一定彻彻底底研究次js中的异步！！！**
