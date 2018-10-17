---
title: angular动画
date: 2018-09-07 14:48:12
tags:
- Angular
- Typescript
categories:
- Angular
---
# Angular动画
在angular2的时候，angular动画还是个核心的组件库，但是到了angular4的时候，为了减小核心库的体积，所以移除出核心组件。但并不是不重要，它依然是angular中很重要的组成部分，而且也是官方提供的支持。

*angular动画架构其实很简单，就是在组件里面定义数个触发器，每个触发器会有一系列的状态和过渡效果，其实这就是动画。*

## State & Transiton（这两个是核心）
*	动画其实就是从一个状态过渡到另一个状态
*	状态本身包含形状、颜色、大小等等
*	State就是定义状态而Transition是定义如何过渡

## Animate函数
**其实在Transition函数中，还会调用另一个函数，那就是Animate**

*	Animate规定了具体怎么样过渡，比如时间、过渡的速度等
*	Animate有多个重载形式

## 使用流程
### 引入关键模块并声明

	import { BrowserModule } from '@angular/platform-browser';
	import { BrowserAnimationsModule } from '@angular/platform-browser/animations';

	@NgModule({
		imports: [ 
			BrowserModule,
			BrowserAnimationsModule 
		],
	})
	export class AppModule { }

建议在`imports`中最后引入`BrowserAnimationsModule`模块，放在前面可能会踩坑。

### 定义各种状态
拿个官网的例子来看看：

	@Component({
		...    //这里是一些正常的定义
		animations: [
			trigger('heroState', [
			state('inactive', style({
				backgroundColor: '#eee',
				transform: 'scale(1)'
			})),
			state('active',   style({
				backgroundColor: '#cfd8dc',
				transform: 'scale(1.1)'
			})),
			transition('inactive => active', animate('100ms ease-in')),
			transition('active => inactive', animate('100ms ease-out'))
			])
		]
	})

这里定义了两个状态，一个是`active`还有个是`incative`，这两个状态分别有不同的style样式，transition定义了从不同状态过渡到另一个状态的过程。

### 附加到模板上

	template: `
		<ul>
			<li *ngFor="let hero of heroes"
				[@heroState]="hero.state"
				(click)="hero.toggleState()">
				{{hero.name}}
			</li>
		</ul>
		`
只要使用`[@triggerName]`语法加到模板上就行了。这就是一个简单的动画效果。

**更多的深入知识，例如缓动函数、关键帧需要的话可以去官网看看，在这贴两个好玩的网址出来：**

[https://easings.net/zh-cn](https://easings.net/zh-cn)
[http://cubic-bezier.com/#.17,.67,.83,-0.63](http://cubic-bezier.com/#.17,.67,.83,-0.63)