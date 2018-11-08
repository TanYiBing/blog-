---
title: api接口跨域问题
date: 2018-11-08 14:55:20
tags:
- Ajax
- Node
categories:
- Ajax
---
今天在自己的项目中想要提供一些api，暴露出来想要提供数据，下面是我的api接口，很简单：

	const router = require('koa-router')();

	router.get('/', async (ctx) => {
		ctx.body = 'api';
	});
	
	router.get('/catelist', async (ctx) => {
		let catelist = await DB.find('articlecate', {});
		ctx.body = {
			result: catelist
		}
	});
	
	module.exports = router.routes();

我们一般调用会怎么写呢？如下：

	$(function(){
		$('#button').click(function(){
			$.getJSON('http://localhost:8000/api/catelist', function(data){
				onsole.log(data);
			})		
		})
	})

我们通过点击一个按钮来接收数据。但是一旦我们`跨域`之后这样子就没办法了。啥叫跨域呢，就是当我们的域名、端口、协议中只要有一个不一致，就请求不到数据啦，那一般的解决方法怎么整呢？我们可以用`jsonp`。

`jsonp`是个什么原理呢？就是利用`script src`可以跨域的特性来实现。拢共分两步：

1. 本地写一个回调函数
2. 在远程执行这个回调函数，把远程的数据传到本地

下面给个例子：

	function xxxx(data){
		console.log(data);
	}

咋们先写个函数，然后用个script来远程调用下：

	<script src="http://localhost:8000/api/catelist?callback=xxxx"></script>

没错，咋们在后面加上`callback`就行啦。再看一个jquery版本的：

	$(function(){
		var url='http://localhost:8000/api/catelist';
		$.ajax({
			url:url,
			dataType:'jsonp',    /*定义jsonp请求*/
			data:'',      /*get传值*/
			jsonp:'callback',      /*回调函数的参数*/
			success:function(data) {
				console.log(data);
			},
			timeout:3000     /*超时时间*/
		});
	})

**`但是这样做有一个前提，那就是后台要允许jsonp，但这样的话会极度不安全，怎么解决呢，我们一般是加上数字签名来保证安全。`**