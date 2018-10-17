---
title: angular中select的change事件
date: 2018-10-11 22:53:39
tags:
- Angular
- Typescript
- Material
categories:
- Angular
---
今天在使用`material`中的select组件的时候发现，我的select上面的`change`事件通过下面的方法并不能接收数据，模板如下：

	<mat-form-field>
		<mat-select placeholder="请选择省份" [(ngModel)]="_address.province" (change)="onProvinceChange()">
			<mat-option *ngFor="let p of provinces$ | async" [value]="p">
				{{ p }}
			</mat-option>
		</mat-select>
	</mat-form-field>

我想通过select的变化来控制其他数据的显示，但是change方法好像监听不到改变一样，最后通过度娘，找到了方法，将change事件改成`ngModelChange`就可以了：

	<mat-form-field>
		<mat-select placeholder="请选择省份" [(ngModel)]="_address.province" (ngModelChange)="onProvinceChange()">
			<mat-option *ngFor="let p of provinces$ | async" [value]="p">
				{{ p }}
			</mat-option>
		</mat-select>
	</mat-form-field>

这样就可以获得事件了。