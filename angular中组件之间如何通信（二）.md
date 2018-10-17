---
title: angular中组件之间如何通信(二)
date: 2018-09-01 19:19:43
tags:
- Angular
- Typescript
categories:
- Angular
---

接着上一篇的内容，上一篇最后说到有公共父组件的组件之间是如何通信的，最后还有一个问题没有解决。但是在这之前我需要补充一下，父组件和子组件之间的通信还有一种方式，那就是**通过本地变量或者使用`@ViewChiad`来进行传递**，举个例子：

	export class childComponent {
		private sayHello() {
			console.log('Hello');
		}
	}

我在子组件中定义这样一个方法，我怎么在父组件中调用呢，一个是通过本地变量的方法：

	<app-child #child1></app-child>
	<button (click)="child1.syaHello()">sayHello</button>

我在父组件的模板中使用子组件并加上`#child1`这个属性，然后就可以在**父组件的模版中直接调用子组件的方法了**。但是我们发现父组件的控制器并不能调用到这个方法，于是，我们就可以使用`@ViewChiad`：

	export class ParentComponent {
		@ViewChild('child1')
		private child: ChildComponent;//声明一个子组件

		sayBay() {
			this.child.sayHello();
		}
	}

这样我们就实现了**在父组件的控制器中调用子组件的方法**。

***
**接下来我们就讨论一下最后一个问题：当组件之间没有公共的父组件时，它们应该如何通信呢？答案就是使用服务进行通信**
。。。loading