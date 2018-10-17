---
title: javascript异步（三）
date: 2018-09-18 09:18:08
tags:
- JavaScript 
categories:
- JavaScript
---
# JavaScript异步（三）————Generator
上次已经介绍过一点`Generator`对象的知识了，这次就继续说说Generator的应用吧。

## Generator的应用

### 使用next和yield传递参数
我们已经知道，`yield`具有返回数据的功能，如下代码。yield后面的数据被返回，存放到返回结果中的value属性中。这算是一个方向的参数传递。

	function* G() {
    	yield 100
	}
	const g = G()
	console.log( g.next() ) // {value: 100, done: false}

还有另一个方向的参数传递，就是next向yield传递，如下：

	function* G() {
    	const a = yield 100
    	console.log('a', a)  // a aaa
    	const b = yield 200
    	console.log('b', b)  // b bbb
    	const c = yield 300
    	console.log('c', c)  // c ccc
	}
	const g = G()
	g.next()    // value: 100, done: false
	g.next('aaa') // value: 200, done: false
	g.next('bbb') // value: 300, done: false
	g.next('ccc') // value: undefined, done: true

我们看看上面代码的执行过程：

*	执行第一个g.next()时，未传递任何参数，返回的{value: 100, done: false}，这个应该没有疑问
*	执行第二个g.next('aaa')时，传递的参数是'aaa'，这个'aaa'就会被赋值到G内部的a标量中，然后执行console.log('a', a)打印出来，最后返回{value: 200, done: false}
*	执行第三个、第四个时，道理都是完全一样的，大家自己捋一捋。

有一个要点需要注意，就g.next('aaa')是将'aaa'传递给上一个已经执行完了的yield语句前面的变量，而不是即将执行的yield前面的变量。

### for...of的应用
for...of是Iterator对象的一个经典操作，我们使用一个斐波那契数列来看一看：

	function* fibonacci() {
    	let [prev, curr] = [0, 1]
    	for (;;) {
        	[prev, curr] = [curr, prev + curr]
        	// 将中间值通过 yield 返回，并且保留函数执行的状态，因此可以非常简单的实现 fibonacci
        	yield curr
    	}
	}
	for (let n of fibonacci()) {
    	if (n > 1000) {
        	break
    	}
    	console.log(n)
	}

这样我们就能找到1000里面的斐波那契数列了。

### yield*嵌套Generator
如果我们有两个Generator，我们想在第一个中包含第二个：

	function* G1() {
    	yield 'a'
    	yield* G2()  // 使用 yield* 执行 G2()
    	yield 'b'
	}
	function* G2() {
    	yield 'x'
    	yield 'y'
	}
	for (let item of G1()) {
    	console.log(item)
	}

之前学过的yield后面会接一个普通的 JS 对象，而yield* 后面会接一个Generator，而且会把它其中的yield按照规则来一步一步执行。如果有多个Generator串联使用的话（例如Koa源码中），用yield*来操作非常方便。

### Generator中的this
对于以下这种写法，大家可能会和构造函数创建对象的写法产生混淆，这里一定要注意 —— Generator 不是函数，更不是构造函数

	function* G() {}
	const g = G()

而以下这种写法，更加不会成功。只有构造函数才会这么用，构造函数返回的是this，而Generator返回的是一个Iterator对象。完全是两码事，千万不要搞混了。

	function* G() {
    	this.a = 10
	}
	const g = G()
	console.log(g.a) // 报错

## Thunk函数
为什么要说说Thunk函数呢，因为它和Generator处理异步操作还是有关系的，我们先看看。

### 普通异步函数

	fs.readFile('data1.json', 'utf-8', (err, data) => {
    	// 获取文件内容
	})

这个普通的node读取文件的函数传递了三个函数，接下来我们进行一点改造。

### 封装成Thunk函数

	const thunk = function (fileName, codeType) {
    	// 返回一个只接受 callback 参数的函数
    	return function (callback) {
        	fs.readFile(fileName, codeType, callback)
    	}
	}
	const readFileThunk = thunk('data1.json', 'utf-8')
	readFileThunk((err, data) => {
    	// 获取文件内容
	})

从上面的Thunk函数可以看出，执行const readFileThunk = thunk('data1.json', 'utf-8')返回的其实是一个函数，readFileThunk这个函数，只接受一个参数，而且这个参数是一个callback函数。

### 使用thunkify库
上面的代码封装使我们自己做的，但我们不需要每遇到一个情况就自己做，我们可以直接使用第三方的thunkify就可以了。

	onst thunk = thunkify(fs.readFile)
	const readFileThunk = thunk('data1.json', 'utf-8')
	readFileThunk((err, data) => {
    	// 获取文件内容
	})

## Generator异步操作
上次我们只是大概的讲了讲，接下来我们详细看看Generator是如何进行异步操作的

### Generator中使用Thunk
直接看代码吧

	const readFileThunk = thunkify(fs.readFile)
	const gen = function* () {
    	const r1 = yield readFileThunk('data1.json')
    	console.log(r1)
    	const r2 = yield readFileThunk('data2.json')
    	console.log(r2)
	}

### 挨个读取两个文件的内容
接着上面的代码继续写：

	const g = gen()
	// 试着打印 g.next() 这里一定要明白 value 是一个 thunk函数 ，否则下面的代码你都看不懂
	// console.log( g.next() )  // g.next() 返回 {{ value: thunk函数, done: false }} 

	// 下一行中，g.next().value 是一个 thunk 函数，它需要一个 callback 函数作为参数传递进去
	g.next().value((err, data1) => {
    	// 这里的 data1 获取的就是第一个文件的内容。下一行中，g.next(data1) 可以将数据传递给上面的 r1 变量，此前已经讲过这种参数传递的形式
    	// 下一行中，g.next(data1).value 又是一个 thunk 函数，它又需要一个 callback 函数作为参数传递进去
    	g.next(data1).value((err, data2) => {
        	// 这里的 data2 获取的是第二个文件的内容，通过 g.next(data2) 将数据传递个上面的 r2 变量
        	g.next(data2)
    	})
	})

仔细看望上面的注释，也许会有中恍然大悟的感觉，原来是这样子把异步写成同步的感觉。

### 自驱动流程
接下来我们做一个自驱动的流程，定义好Generator的代码之后，就让他自动执行：

	// 自动流程管理的函数
	function run(generator) {
    	const g = generator()
    	function next(err, data) {
        	const result = g.next(data)  // 返回 { value: thunk函数, done: ... }
        	if (result.done) {
            	// result.done 表示是否结束，如果结束了那就 return 作罢
            	return
        	}
        	result.value(next)  // result.value 是一个 thunk 函数，需要一个 callback 函数作为参数，而 next 就是一个 callback 形式的函数
    	}
    	next() // 手动执行以启动第一次 next
	}

	// 定义 Generator
	const readFileThunk = thunkify(fs.readFile)
	const gen = function* () {
    	const r1 = yield readFileThunk('data1.json')
    	console.log(r1.toString())
    	const r2 = yield readFileThunk('data2.json')
    	console.log(r2.toString())
	}

	// 启动执行
	run(gen)

我们简单分析下：

*	最后一行run(gen)之后，进入run函数内部执行
*	先const g = generator()创建Generator实例，然后定义一个next方法，并且立即执行next()
*	注意这个next函数的参数是err, data两个，和我们fs.readFile用到的callback函数形式完全一样
*	第一次执行next时，会执行const result = g.next(data)，而g.next(data)返回的是{ value: thunk函数, done: ... }，value是一个thunk函数，done表示是否结束
*	如果done: true，那就直接return了，否则继续进行
*	result.value是一个thunk函数，需要接受一个callback函数作为参数传递进去，因此正好把next给传递进去，让next一直被执行下去

### co库
这个流程我们也可以使用第三方的库co，用Generator的工程师肯定都要用co，两者天生一对。

	// 定义 Generator
	const readFileThunk = thunkify(fs.readFile)
		const gen = function* () {
    	const r1 = yield readFileThunk('data1.json')
    	console.log(r1.toString())
    	const r2 = yield readFileThunk('data2.json')
    	console.log(r2.toString())
	}
	const c = co(gen)

而且const c = co(gen)返回的是一个Promise对象，可以接着这么写

	c.then(data => {
    	console.log('结束')
	})

## Koa中使用Generator
Koa第一版中大量使用了Generator，接下来我们去看看怎么使用的。

### Koa中如何使用Generator
koa 是一个 web 框架，处理 http 请求，但是这里我们不去管它如何处理 http 请求，而是直接关注它使用Genertor的部分————中间件。
例如，我们现在要用 3 个Generator输出12345，我们如下代码这么写。

	let info = ''
	function* g1() {
    	info += '1'  // 拼接 1
    	yield* g2()  // 拼接 234
    	info += '5'  // 拼接 5
	}
	function* g2() {
    	info += '2'  // 拼接 2
    	yield* g3()  // 拼接 3
    	info += '4'  // 拼接 4
	}
	function* g3() {
    	info += '3'  // 拼接 3
	}

	var g = g1()
	g.next()
	console.log(info)  // 12345

但是如果用 koa 的 中间件 的思路来做，就需要如下这么写。

	app.use(function *(next){
    	this.body = '1';
    	yield next;
    	this.body += '5';
    	console.log(this.body);
	});
	app.use(function *(next){
    	this.body += '2';
    	yield next;
    	this.body += '4';
	});
	app.use(function *(next){
    	this.body += '3';
	});

我们需要注意几点：

*	app.use()中传入的每一个Generator就是一个 中间件，中间件按照传入的顺序排列，顺序不能乱
*	每个中间件内部，next表示下一个中间件。yield next就是先将程序暂停，先去执行下一个中间件，等next被执行完之后，再回过头来执行当前代码的下一行。因此，koa 的中间件执行顺序是一种洋葱圈模型，不过这里看不懂也没问题。
*	每个中间件内部，this可以共享变量。即第一个中间件改变了this的属性，在第二个中间件中可以看到效果。

### Koa的这种机制如何实现的
我们封住个简单的Koa：

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
        	const ctx = this
        	const middlewares = ctx.middlewares
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

如何使用呢：

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
