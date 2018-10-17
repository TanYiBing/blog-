---
title: angular数据绑定
date: 2018-08-30 22:56:05
tags:
- Angular
- Typescript
categories:
- Angular
---

Angular数据绑定是Angular中最常见的，绑定的方式有单向数据绑定和双向数据绑定。单向的数据绑定通常有以下几种方式：

**1.使用插值表达式将一个表达式的值显示在模板上：**

    <h1>{{titke}}</h1>
***

**2.使用方括号将HTML标签的一个属性绑定到一个表达式上：**

	<img [src] = "imgUrl">

其实我们的属性绑定和我们的插值表达式是一样的，我们举个例子;

	<img [src] = "imgUrl">
	<img src = {{imgUrl}}>

这两种写法其实都一样，插值的表达式都会被Angular转化成属性绑定。**在这还有一个需要理解的东西就是HTML属性和DOM属性的区别**，举个例子：

	<input value="hello" (click)="doOnInput($event)">

我们的控制器定义一下我们的输入事件：

	doOnInput(event: any) {
		console.log(event.target.value);//打印DOM属性的值
		console.log(event.target.getAttribute('value');//打印HTML属性的值
	}

我们可以发现DOM属性的值是随着你改变输入的值而一直在改变的，而HTML属性的值一直是hello，因为HTML是初始化DOM属性的值的，不能改变，而DOM属性的值可以改变。

**HTML属性和DOM属性的关系如下：**
1. 少量HTML属性和DOM属性之间有1:1的映射，比如id。
2. 有些HTML属性没有对应的DOM属性，如colspan。
3. 有些DOM属性没有对应的HTML属性，如textContent。
4. 就算名字相同，HTML属性和DOM属性也不是一样东西。
5. HTML属性的值指定了初始值；DOM属性的值表示当前值.DOM属性的值可以改变；HTML属性的值不能改变。
6. 模板绑定是通过DOM属性和事件来工作的，而不是HTML属性。

**以上是DOM属性的绑定，下面介绍下HTML属性的绑定**
简单的举个例子：

	<td [attr.colspan]={{1+1}}>

因为colspan是个DOM属性没有的HTML属性，所以我们在前面加上`attr`这个前缀来表示它是HTML属性。

更多的我们可以通过`[ngClass]`或者`[class.classname]`、`[class]`这种来控制部分或者全部的类名，以此来达到控制样式的效果。`ngclass`的表达式需要接收一个对象，对象中通过`{cssClassName: Boolen}`这种键值对来表示该类是否要加上，`true`表示加上，`false`代表移除，这就是**CSS类绑定**。

类似的，我们还可以通过**css样式绑定**，不过不再是`class`开头了，而是`style`开头，举个栗子：

	<div [style.color]=" isDev? 'red': 'blue'">

其中的`isDev`就是控制器来控制的。和上面css类绑定一样，这只是单一样式的绑定，所以我们还有个`[ngStyle]`来批量控制，它也是接受一个对象，但和css类不同的是，这个对象里面定义的是样式的键值对，例如{color: red,background: black}
***
**3.上面这两个绑定方式都是控制器向模板传输数据，下面这种是模板向控制器传输数据，也就是事件绑定：**

	<button (click)="onClickEvent($event)">点击执行</button>
	//小括号表示这是一个事件绑定
	//小括号中的是事件的名称
	//等号后面双引号中的是事件发生时执行的表达式
	//如果处理事件的方法需要了解事件的属性，我们就可以给方法加上$event参数

我们的事件绑定中也可以不是一个方法调用，而是一个属性赋值：
	
	<button (click)="saved = true">

这样我们就能够直接给控制器中的saved属性赋值为true。
***
**接下来我们看一下最重要的双向数据绑定**

>盒子里面装香蕉

这句话是官网文档上面说的，就是指`[(ngModel)]`是方括号包着小括号，其实从上面就能看出，控制器传值给模板是方括号，模板传值给控制器是小括号，两个融合到一起就是双向数据绑定了，举个栗子：

	<input [value]="currentHero.name"
           (input)="currentHero.name=$event.target.value" >

下面这个而其实就是一个双向数据绑定，但是显得太过笨重，于是有了下面这种写法：

	<input [(ngModel)]="currentHero.name">

这样就能够同步输入和显示了，最常用的地方就是我们的表单元素了。