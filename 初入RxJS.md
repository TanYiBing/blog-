---
title: 初入RxJS
date: 2018-09-10 11:21:19
tags:
- Angular
- Typescript
categories:
- Angular
---
# RxJS

## 简介
刚开始接触angular的时候就发现，RxJS在angular中是使用的最多的框架。它的全称是：`Reactive Extension`。这套框架其实是源自于微软，微软在2011年就已经开发出这套框架了，但真正火起来是在NetFlix，支持很多语言的一套响应式编程的框架，感兴趣可以去它的官网看看：[reactivex.io](reactivex.io)，支持了这么多语言！！！

>![](/img/angular/12.png)

它的优势在于：`在思考的维度上加上时间考量。`，这也是一个问题，导致更难理解！

## Observable
### Observable的性质
Observable有三种状态，分别是：

* `next`：必要。用来处理每个送达值。在开始执行后可能执行零次或多次。
* `error`：可选。用来处理错误通知。错误会中断这个可观察对象实例的执行过程。
* `complete`：可选。用来处理执行完毕（complete）通知。当执行完毕后，这些值就会继续传给下一个处理器。 

还有特殊的Observable：`永不结束的`、`Never`、`Empty（结束但不发射）`、`Throw`

本来是想写介绍点操作符的，但是我发现没啥意思，反正文档都能查到，那就记录点自己对Observable的理解吧，具体操作符到时候查api。
其实Observable就是将数据以流的形式来进行处理，可以理解成node中的流，我们可以对流进行一系列的操作，最后我们再订阅这个流，对流里面的数据进行使用。

举个例子吧:

	const a: Array<number> = [1, 2, 3, 4];
	const a$ = Rx.Observable.from(a).pipe(map( val => val * val));
	const observer = {
		next: (val) => console.log(val);
		error： (err) => console.log(err);
		complete: () => console.log(`everything is completed`);
	}
	a$.subscribe(observer);

上面的例子会打印出：
>1 4 9 16

因为我们对流中的数据进行了map操作，使它们自己乘以自己。这就是对流的一个简单处理，连`pipe`方法都和node中的流一样。

### Observable的冷和热
Observable有两种，一种是冷的，一种是热的。怎么去理解呢？**下面的内容在angular的官网上并没有说明**

* 冷的就是表示，每次你订阅之后我们都会从头执行一遍，就像看重播一样，点击之后都是重头开始播放；
* 热代表每次订阅之后大家之间的进度是一样的，就像看直播一样，打开之后都是最新时刻的通知。

未完待续。