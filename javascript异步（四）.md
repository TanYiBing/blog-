---
title: javascript异步（四）
date: 2018-09-18 10:56:41
tags:
- JavaScript 
categories:
- JavaScript
---
# JavaScript异步（四）————async & await
前面已经介绍了`Generator`，我的唯一感觉就是————学习成本真特喵的高，我们今天还要看一个异步的终极解决方案————`async-await`，这就是ES7自己参照Generator封装的，其实更像是Generator的语法糖。

## ES7中的async-await

### Generator和async-await对比
我们先来一段Generator的异步处理代码：

	co(function* () {
    	const r1 = yield readFilePromise('some1.json')
    	console.log(r1)  // 打印第 1 个文件内容
    	const r2 = yield readFilePromise('some2.json')
    	console.log(r2)  // 打印第 2 个文件内容
	})

接着看看使用async-await，对比一下;

	const readFilePromise = Q.denodeify(fs.readFile)

	// 定义 async 函数
	const readFileAsync = async function () {
    	const f1 = await readFilePromise('data1.json')
    	const f2 = await readFilePromise('data2.json')
    	console.log('data1.json', f1.toString())
    	console.log('data2.json', f2.toString())

    	return 'done' // 先忽略，后面会讲到
	}
	// 执行
	const result = readFileAsync()

我们可以比较看出，async function代替了function* ，await代替了yield，而且使用async-await不在用co这个第三方库了，直接执行即可。

### 使用async-await的差异
第一，await后面不能再跟thunk函数，而必须跟一个Promise对象（因此，Promise才是异步的终极解决方案和未来）。跟其他类型的数据也OK，但是会直接同步执行，而不是异步。

第二，执行const result = readFileAsync()返回的是个Promise对象，而且上面代码中的return 'done'会直接被下面的then函数接收到

	result.then(data => {
    	console.log(data)  // done
	})

第三，从代码的易读性来将，async-await更加易读简介，也更加符合代码的语意。而且还不用引用第三方库，也无需学习Generator那一堆东西，使用成本非常低。

因此，如果 ES7 正式发布了之后，强烈推荐使用async-await。但是现在尚未正式发布，从稳定性考虑，还是Generator更好一些。

## 异步操作代码的演变历程
最后再来感受下从`callback`到`async-await`的历程吧。

### callback方式

	fs.readFile('some1.json', (err, data) => {
    	fs.readFile('some2.json', (err, data) => {
        	fs.readFile('some3.json', (err, data) => {
            	fs.readFile('some4.json', (err, data) => {

            	})
        	})
    	})
	})

### Promise方式

	readFilePromise('some1.json').then(data => {
    	return readFilePromise('some2.json')
	}).then(data => {
    	return readFilePromise('some3.json')
	}).then(data => {
    	return readFilePromise('some4.json')
	})

### Generator方式

	co(function* () {
    	const r1 = yield readFilePromise('some1.json')
    	const r2 = yield readFilePromise('some2.json')
    	const r3 = yield readFilePromise('some3.json')
    	const r4 = yield readFilePromise('some4.json')
	})

### async-await方式

	const readFileAsync = async function () {
    	const f1 = await readFilePromise('data1.json')
    	const f2 = await readFilePromise('data2.json')
    	const f3 = await readFilePromise('data3.json')
    	const f4 = await readFilePromise('data4.json')
	}

到这基本就完了，如果有看到好文章以后再补充，js异步探索的任务算是结束了。