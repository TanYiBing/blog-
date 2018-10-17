---
title: 使用Rx.js和json-server模拟随机获取内容
date: 2018-09-29 17:03:56
tags:
- Angular
- Rxjs
categories:
- Rxjs
---
今天使用angular中的`rx.js`来将我们的数据转换成`Observable`，毕竟angular中到处都是`Observable`可观察对象这种方式，因为使用起来很方便。

我们没有真的后台怎么模拟接口呢，我直接使用了`json-server`，这个使用起来很方便，只需要写好一个模拟接口的`json`文件，然后执行命令就行了。首先我们安装：

	cnpm install --save-dev json-server

然后我们写一个json文件，紧接着执行一条命令：

	json-server ./mock/data.json

这样我就能在`localhost:3000`下面通过http请求到我的json文件啦。

紧接着我们把获取json文件功能封装成angular中的一个服务：

	import { Injectable, Inject } from '@angular/core';
	import { Quote } from '../domain/quote.model';
	import { Observable } from 'rxjs';
	import { HttpClient } from '@angular/common/http';
	import { map } from 'rxjs/operators';
	
	@Injectable()
	export class QuoteService {
    	constructor(
        	private Http: HttpClient,
        	@Inject('BASE_CONFIG') private config
    	) {}

	    getQuote(): Observable<Quote> {
	        const uri = `${this.config.uri}/quotes/${Math.floor(Math.random() * 10)}`;
	        return this.Http.get(uri).pipe(map(res => res as Quote));
	    }
	}

这样我们就注册了一个我们自己的服务，而且我们同过随机请求uri可以实现随机获取内容中的一天。而且，我们最后在返回的时候，返回的是`Observable`类型的内容。在需要使用这些内容的组件内，我们只要注册这个服务就可以了：

	constructor(private quoteService$: QuoteService) {
    	this.quoteService$
      		.getQuote()
      		.subscribe(q => this.quote = q);
	}

我们在组件中注册这个服务之后，通过订阅的方式获取到了数据，这样是不是很简单。

**我们使用`quoteService$`这样的写法是因为，这是一个流！**