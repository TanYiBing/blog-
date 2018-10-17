---
title: exports和module.exports的区别
date: 2018-09-19 09:35:45
tags: 
- Node 
- JavaScript
categories:
- Node
---
# exports和moduls.exports的区别

每一个node.js执行文件，都会自动创建一个`module对象`，同时，module对象会创建一个叫`exports`的属性，初始化的值是{}：

	module.exports = {};

这时我们使用exports看看：

	math.js
	
	exports.add=function(arg1,arg2){
		return arg1+arg2;
	};
	exports.minus=function(arg1,arg2){
		return arg1-arg2;
	};
	console.log(module);


	test.js

	var math=require('./math.js');
	console.log('math:'+math);
	console.log(math.add(3,4));
	console.log(math.minus(5,3));
	
控制台中信息:
module:

	{ id: '/home/chander-zhang/NodeJS/test/math.js',
		exports: { add: [Function], minus: [Function] },
		parent: 
			{ ... },
		filename: '/home/chander-zhang/NodeJS/test/math.js',
		loaded: false,
		children: [],
		paths: [ 
			'/home/chander-zhang/NodeJS/test/node_modules',
			'/home/chander-zhang/NodeJS/node_modules',
     		'/home/chander-zhang/node_modules',
     		'/home/node_modules',
     		'/node_modules'
		] }
	
math: 

	[object Object]

这时的math是个对象，当我们使用module.exports时会怎么样：

	math.js
	
	exports.add=function(arg1,arg2){
		return arg1+arg2;
	};
	minus=function(arg1,arg2){
		return arg1-arg2;
	};
	module.exports=minus;
	console.log(module);


	test.js

	var math=require('./math.js');
	console.log('math:'+math);
	//console.log(math.add(3,4));
	console.log(math.minus(5,3));//相当于执行了 minus(5,3)

控制台中打印出：、
module:
	
	{ id: '/home/chander-zhang/NodeJS/test/math.js',
		exports: [Function],
		parent: 
			{ ... },
		filename: '/home/chander-zhang/NodeJS/test/math.js',
		loaded: false,
		children: [],
		paths: [ 
			'/home/chander-zhang/NodeJS/test/node_modules',
     		'/home/chander-zhang/NodeJS/node_modules',
     		'/home/chander-zhang/node_modules',
     		'/home/node_modules',
     		'/node_modules'
		] }

math: 

	function (arg1,arg2){
    	return arg1-arg2;
	}

如果使用math.add()或math.minus()将会出错,因为此时math是一个函数. 即便module中出现了exports.add,由于为module.exports赋了值,也不能使用math.add().
**因为exports是指向module.exports的引用，而require（）返回的是module.exports而不是exports，当我们自己定义module.exports时就会覆盖自动生成的moudle.exports。**
	