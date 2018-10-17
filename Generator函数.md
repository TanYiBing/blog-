---
title: Generator函数
date: 2018-09-12 16:09:54
tags:
- JavaScript 
- ECMAScript6
categories:
- ECMAScript6
---
# Generator函数
昨天的`Promise`对象就是一种异步编程解决方案，今天再看一种方案，这就是`Generator`函数。
网上有很多关于`Generator`的介绍，例如：
>Generator 函数有多种理解角度。语法上，首先可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。
执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

## Generator的创建
接下来我们看一下怎么定义一个Generator函数：

	function* helloWorldGenerator() {
		yield 'hello';
		yield 'world';
		return 'ending';
	}
	var hw = helloWorldGenerator();

我们发现在`function`关键字和函数名称之间有个星号，这表示这是一个Generator函数。但这样还不行，进入函数我们发现使用了`yield`表达式，定义了不同的内部状态(`yield`在英语中就是“产生”的意思)。这样我们一共定义了三个状态：hello、word和一个return语句。

## Generator的调用
Generator函数返回的是一个遍历器对象，也就是说我们需要调用`next`方法来遍历函数中的状态：
	
	hw.next()
	// { value: 'hello', done: false }

	hw.next()
	/ { value: 'world', done: false }

	hw.next()
	// { value: 'ending', done: true }

	hw.next()
	/ { value: undefined, done: true }

上面一共运行了四次`next`方法。

第一次调用，Generator 函数开始执行，直到遇到第一个yield表达式为止。next方法返回一个对象，它的value属性就是当前yield表达式的值hello，done属性的值false，表示遍历还没有结束。

第二次调用，Generator 函数从上次yield表达式停下的地方，一直执行到下一个yield表达式。next方法返回的对象的value属性就是当前yield表达式的值world，done属性的值false，表示遍历还没有结束。

第三次调用，Generator 函数从上次yield表达式停下的地方，一直执行到return语句（如果没有return语句，就执行到函数结束）。next方法返回的对象的value属性，就是紧跟在return语句后面的表达式的值（如果没有return语句，则value属性的值为undefined），done属性的值true，表示遍历已经结束。

第四次调用，此时 Generator 函数已经运行完毕，next方法返回对象的value属性为undefined，done属性为true。以后再调用next方法，返回的都是这个值。

## Generator函数的作用
这个东西看上去好像没啥用，那么它到底能干点啥呢？

### 把异步回调变成‘同步’代码
例如我们写个ajax代码：

	ajax('http://url-1', data1, function (err, result) {
    	if (err) {
        	return handle(err);
    	}
    	ajax('http://url-2', data2, function (err, result) {
        	if (err) {
            	return handle(err);
        	}
        	ajax('http://url-3', data3, function (err, result) {
            	if (err) {
                	return handle(err);
            	}
            	return success(result);
        	});
    	});
	});

这个代码简直不想看，那么我们可以用Generator来改造下：

	try {
    	r1 = yield ajax('http://url-1', data1);
    	r2 = yield ajax('http://url-2', data2);
    	r3 = yield ajax('http://url-3', data3);
    	success(r3);
	}
	catch (err) {
    	handle(err);
	}

这样看上去好多了，但不是真正的同步代码，至少看上去很像。