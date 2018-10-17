---
title: angular中的两种表单
date: 2018-09-03 14:44:55
tags:
- Angular
- Typescript
categories:
- Angular
---
表单forms是前端中很重要的一块内容，Angular中同样如此。在Angular中，有两种表单的形式，一种是`模板驱动表单`，还有一种是`响应式表单`。接下来就简单的介绍一下：

## 模版驱动表单

区别：只能在模板中操作，不能在代码中操作。
不说了，直接上个表单：

	<form #myForm="ngForm" (ngSubmit)="onSubmit(myForm.value)">
		<div>用户名：<input ngModel name="username" type="text"></div>
		<div>手机号：<input ngModel name="mobile" type="number"></div>
		<div ngModelGroup="passwordsGroup">
			<div>密码：<input ngModel name="password" type="password"></div>
			<div>确认密码：<input ngModel name="pconfirm" type="password"></div>
		</div>
		<button type="submit">注册</button>
	</form>



## 响应式表单

区别：只能在代码中操作，不能在模板中操作。
响应式表单一共分为两步：

### 创建数据模型
由三个类组成：`FormControl`、`FormGroup`、`FormArray`，怎么创建呢？直接贴出代码：

	export class ReactiveFormComponent implements OnInit {
		username: FormControl = new FormControl('username');
		
		formModel: FormGroup = new FormGroup({
			dateRange: new FormGroup({
				from: new FormControl(),
				to: new FormControl()
			}),
			emails: FormArray = new FormArray([
				new FormControl("a@a.com"),
				new FormControl("b@b.com")
			]);
		});	
	}

### 使用响应式表单指令绑定表单模版

>![](/img/angular/6.png)

	html
	<form [formGroup]="formModel" (submit)="onSubmit()">
		<div formGroupName="dateRange">
			起始日期：<input type="date" formControlName="from">
			截止日期：<input type="date" formControlName="to">
		</div>
		<div>
			<ul formArryName="emails">
				<li *ngFor="let e of this.formModel.get('emails').controls; let i = index;">
					<input type="text" [formControlName]="i">
				</li>
			</ul>
			<button type="button" (click)="addEmail()">增加email</button>
		</div>
		<div>
			<button type="submit">保存</button>
		</div>
	</form>
	
	js
	addEmail(){
		let emails = this.formModel.get("emails") as FormArray;
		emails.push(new FormControl());
	}

这样看上去是不是代码很多，有个简化的方法就是使用`FormBuilder`，使用首先要引入，然后再`constructor`中注册，然后给个对比吧：

	profileForm = new FormGroup({
		firstName: new FormControl(''),
		lastName: new FormControl(''),
		address: new FormGroup({
    		street: new FormControl(''),
    		city: new FormControl(''),
    		state: new FormControl(''),
    		zip: new FormControl('')
		})
	});

上面的等同于：
	
	profileForm = this.fb.group({
		firstName: [''],
		lastName: [''],
		address: this.fb.group({
			street: [''],
			city: [''],
    		state: [''],
    		zip: ['']
		}),
	});

`FormGroup`用`this.fb.group`替代，`FormContrl`用`['']`替代,怎么进行校验，我的天，又是好多字，今天就先这样吧！！！反正是我自己看的，我记得就行了！！！哈哈哈！！！