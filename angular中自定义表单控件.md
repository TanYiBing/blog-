---
title: angular中自定义表单控件
date: 2018-09-28 23:51:23
tags:
- Angular
- Typescript
- Material
categories:
- Angular
---
angular中有两种表单，一种是模版驱动的表单，还有一种是响应式的表单。如果我们的表单比较简单的话我们可以使用模版驱动型表单，但是一旦我么的表单复杂起来，那么响应式表单是我们的首选。

但是一旦我们的表单过于复杂之后，我们就需要将很多的逻辑写在一起，对于我们来说是件很糟糕的事情，所以，我们需要`把复杂问题简单化`，因此，我们可以自定义表单控件。

我需要实现一个如下的头像选择效果：

>![](/img/material/8.png)

如果光是添加头像我们可以直接写，没必要写成控件，但是，我在之后的项目中可能也想给我其他表单添加图片也实现这样的功能，那我就需要封装一下，方便以后复用。

我在此就写一些删减后的代码，理清思路。首先我们给出我们自定义表单控件的模板：

	<div>
		<mat-icon *ngIf="useSvgIcon else imgSelect"></mat-icon>
		<ng-template #imgSelect>
			<img [src]="selected" alt="image selected">
		</ng-template>
	</div>
	<div class="scroll-container">
		<mat-grid-list>
	    <mat-grid-tile *ngFor="let item of items">
	      <div class="image-container">
	        <mat-icon [svgIcon]="item" *ngIf="useSvgIcon else imgItem"></mat-icon>
	        <ng-template #imgItem>
	          <img [src]="item" alt="image item">
	        </ng-template>
	    </mat-grid-tile>
	</div>

第一个div是我们展示选中的，下面的是展示所有头像，使用的是material中的`gridlist`。

我们想要自定义表单控件，最重要的就是实现`ControlValueAccessor`接口，例子如下：
	
	import { Component, forwardRef } from '@angular/core';
	import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

	@Component({
		selector: 'app-image-list-select',
		templateUrl: './image-list-select.component.html',
		styleUrls: ['./image-list-select.component.scss'],
		providers: [
			{
			provide: NG_VALUE_ACCESSOR,
			useExisting: forwardRef(() => ImageListSelectComponent),
			multi: true // 允许令牌多对一
			}
		]
	})
	export class ImageListSelectComponent implements ControlValueAccessor {
		constructor() { }

		// 这里是做一个空函数体，真正使用的方法在 registerOnChange 中
		// 由框架注册，然后我们使用它把变化发回表单
		// 注意，和 EventEmitter 尽管很像，但发送回的对象不同
		private propagateChange = (_: any) => {};

		// 写入控件值
		writeValue(obj: any): void {}

		// 当表单控件值改变时，函数 fn 会被调用
		// 这也是我们把变化 emit 回表单的机制
		registerOnChange(fn: any): void {
    		this.propagateChange = fn;
		}

		// 这里没有使用，用于注册 touched 状态
		registerOnTouched(fn: any): void {}

		setDisabledState?(isDisabled: boolean): void {}
	}

要实现`ControlValueAccessor`接口，我们就需要实现上面四中方法，在`writeValue`，我们就能获取到控件中值的值，然后我们通过`private propagateChange = (_: any) => {};`这个类似于`EventEmitter`的函数将值暴露出去。差不多就行了。

想要了解这个项目的可以去我的github上看我的项目：[这是地址](https://github.com/TanYiBing/taskmgr)
