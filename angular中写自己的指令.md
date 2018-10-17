---
title: angular中写自己的指令
date: 2018-09-27 23:23:39
tags:
- Angular
- Typescript
categories:
- Angular
---
今天我们想实现一个组件拖拽的效果，以方便以后可以通过拖拽来切换我们组件的位置。所以，我们需要设计自己的指令`Directive`。

在angular中有三种指令：

1.	组件 — 拥有模板的指令
2.	结构型指令 — 通过添加和移除 DOM 元素改变 DOM 布局的指令
3.	属性型指令 — 改变元素、组件或其它指令的外观和行为的指令

组件是这三种指令中最常用的，而**结构型的指令会改变 DOM 结构，建议是不使用，但真要使用之前得好好思考下，否则可能会带来一些不好的结果**。我们今天就使用属性型的指令来实现我们想要的东西吧。

直接上代码吧：

	import { Directive, Input, HostListener, ElementRef, Renderer2 } from '@angular/core';
	import { DragDropService } from '../drag-drop.service';

	@Directive({
		selector: '[app-draggable]'
	})
	export class DragDirective {
		private _isDraggable = false;
		@Input() draggedClass: string;
		@Input('app-draggable')
		set isDraggable(val: boolean) {
	    	this._isDraggable = val;
	    	this.rd.setAttribute(this.el.nativeElement, 'draggable', `${val}`);
		}
		get isDraggable() {
	    	return this._isDraggable;
	  	}
	  	constructor(
	    	private el: ElementRef,
	    	private rd: Renderer2 ) { }
	
		@HostListener('dragstart', ['$event'])
		onDragStart(ev: Event) {
	    	if (this.el.nativeElement === ev.target) {
				this.rd.addClass(this.el.nativeElement, this.draggedClass);
	    	}
	  	}
	
	  	@HostListener('dragend', ['$event'])
	  	onDragEnd(ev: Event) {
	    	if (this.el.nativeElement === ev.target) {
	      		this.rd.removeClass(this.el.nativeElement, this.draggedClass);
	    	}
	  	}
	
	}

这是我们的 `dragDective`,当然代码没有写全，我们只是列一个思路，首先我们可以使用angular-cli直接生成一个指令模版：

	ng g d xxx //xxx代表指令的名称

生成之后直接就有了`selector`，里面就是我们指令的名称，然后我们通过类似于.net的语法来获取下这个指令是`true`还是`false`：

	@Input('app-draggable')
		set isDraggable(val: boolean) {
	    	this._isDraggable = val;
	    	this.rd.setAttribute(this.el.nativeElement, 'draggable', `${val}`);
		}
		get isDraggable() {
	    	return this._isDraggable;
	  	}

这样我们就可以在我们的组件模板中加上这个指令了：

	<app-task-list
    	class="list-container"
    	app-droppable

	    [app-draggable]="true"//这就是指令的位置
	   
	    (dropped)="handleMove($event,list)"
	    *ngFor="let list of lists"
	>

如此一来可以说是完成了添加，但是我们的指令啥都没有做，所以也就不会有效果，我们可以发现的是，我们的指令代码中出现很多`@HostListener`，没错，这就是我们监听鼠标事件的方法，通过响应事件来进行一系列样式的添加：

	@HostListener('dragstart', ['$event'])
		onDragStart(ev: Event) {
	    	if (this.el.nativeElement === ev.target) {
				this.rd.addClass(this.el.nativeElement, this.draggedClass);
	    	}
	  	}
	
	  	@HostListener('dragend', ['$event'])
	  	onDragEnd(ev: Event) {
	    	if (this.el.nativeElement === ev.target) {
	      		this.rd.removeClass(this.el.nativeElement, this.draggedClass);
	    	}
	  	}

那这个其中的`draggedClass`就是我们自己定义的样式，我们的鼠标进行拖拽的时候就能显示我们的样式。


以上只是项目中的一小块代码，感兴趣的可以去我的Github上看看哦：[这是地址](https://github.com/TanYiBing/taskmgr)