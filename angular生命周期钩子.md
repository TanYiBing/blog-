---
title: angular生命周期钩子
date: 2018-09-02 22:40:24
tags:
- Angular
- Typescript
categories:
- Angular
---
Angular中每个组件都有它的生命周期，从创建、渲染，到变更检测，再到最后的销毁过程过程，Angular一共提供了**八个**生命周期钩子，他们按照顺序分别是：`constructor`、`ngOnChanges`、`ngOnInit`、`ngDoCheck`、`ngAfterContentInit`、`ngAfterContentChecked`、`ngAfterViewInit`、`ngAfterViewChecked`、`ngOnDestroy`，其中有的钩子在生命周期内只调用一次，有的则可以反复调用（变更检测机制中需要调用方法）。如下图：

>![](/img/angular/4.png)

其中红色的表示只调一次，绿色的则可以被多次调用。我们将介绍其中重要的几个方法。

#### ngOnChanges:
这个钩子在父组件初始化或修改子组件的**输入属性**时会被调用。想要理解这个方法为什么会被调用或不被调用，我们需要了解`可变对象`和`不可变对象`，在js中，字符串是不可变对象，但一个对象是一个可变对象，至于为什么，请自行百度。最后的结论是：
**当输入属性是变对象例如对象时，对象的属性发生变化并不会触发`ngOnChanges`方法；当输入属性是不可变对象例如字符串时，字符串改变会触发`ngOnChanges`方法**。
*还需要注意一点就是*：
**`ngOnChanges`方法首次调用一定是发生在`ngOninit`方法之前的。**

#### ngDoCheck:	
变更检测可以说是Angular中最复杂的模块：
>![](/img/angular/5.png)

angular有两种变更检测机制，一种是Default策略，一种是OnPush策略。Default策略会在组件发生变化时去检查组件树中**所有**组件，而使用OnPush的组件只会在其`输入属性`发生变化时去检查它。当我们对性能要求比较高的时候，例如有大量的表格数据在时时发生变化时，这两种策略的正确选择就显得尤为重要。
在`ngOnChanges`中我们知道
>当输入属性是可变对象例如对象时，对象的属性发生变化并不会触发`ngOnChanges`方法。

那么怎么检测对象的属性呢，这时就需要用到`ngDoCheck`方法了。`check`的调用十分频繁，发生一点点动静都会调用`check`方法，就比如我们的鼠标光标切换input但是没有改变值框这类的动作，组件树上所有的有`check`关键字钩子都会调用，所以，我们的所有`check`方法一定要小心，**一定要高效，一定要轻量级**。

#### ngAfterViewInit & ngAfterViewChecked:	
顾名思义，这两个方法都是跟组件的视图密切相关的，前者是在视图都组建完毕之后才调用，后者是在视图发生变化时调用，值得注意的是：
1.	**这两个方法都是在视图组建完毕之后被调用的；**
2.	**如果父组件存在子组件，那么父组件的这两个方法会在所有子组件的视图都组建完毕后调用；**
3.	**不要早在这两个方法里面去改变视图中绑定的东西，这样做会报错。如果真的需要修改，需要写在一个timeout方法里面。**

#### ngAfterContentInit & ngAfterContentChecked:
要使用这两个方法，就得再说到一个`ng-content投影`，什么是`内容投影`呢？就是从组件外部导入HTML内容，并把它插在组件中指定位置上的一种途径，举个例子，我们在子组件模板中这样定义一个投影点：

	<ng-content></ng-content>

这样我们在父组件中可以定义：

	<app-child>
	//这里面可以写想投影到子组件的内容
	</app-child>

这样我们就将父组件中的内容投影到了子组件中。不仅如此，我们还能同时投影多个内容，在父组件中这样定义：

	<app-child>
	//这里面可以写想投影到子组件的内容
	<div class='header'>{{title}}</div>
	<div class='footer'>{{content}}</div>
	</app-child>

然后再子组件中可以这样接收：

	<ng-content select='.header'></ng-content>
	<ng-content select='.footer'></ng-content>

这样我们不仅将两个HTML片段投影到子组件中，还将父组件控制器中的`title`、`content`通过插值表达式投影到了子组件中。
`ngAfterContentInit`和`ngAfterViewInit`不同的是：**`ngAfterViewInit`不能够改变视图中绑定的值，而在`ngAfterContentInit`中是可以的，不会报错**。因为在`ngAfterContentInit`的时候，视图还没有组装完毕，只是投影进来的内容被组装完毕了。

#### ngOnDestroy
组件什么时候被销毁呢，就是路由到别处的时候，当前组件会被销毁，创建新的路由的组件。在这一般是用来释放那些不会被垃圾收集器自动回收的各类资源的地方。取消那些对可观察对象和 DOM 事件的订阅。停止定时器。注销该指令曾注册到全局服务或应用级服务中的各种回调函数。如果不这么做，就会有导致内存泄露的风险。