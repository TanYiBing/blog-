---
title: angular中组件之间如何通信
date: 2018-08-31 15:27:24
tags:
- Angular
- Typescript
categories:
- Angular
---
在angular中，我们会设计很多的component组件，也就生成了组件树，组件之间有可能是父子组件，或者兄弟组件，或者彼此都没有关系。那么这么多组件之间该如何通讯呢？下面我们就简答的了解了解。

## 父子组件之间的通讯
父子之间的通讯比较简单，我们需要用到`@Input输入属性`和`@Output输出属性`。当我们需要**从父组件向子组件传递信息时**，我们首先需要为子组件定义一个属性：

	export class childComponent {
		@Input() hero: Hero;
	}

被`@Input`修饰的属性就即将接受从父组件传过来的信息。具体怎么传值呢？我们在父组件中通过子组件的属性进行传值：

	<app-child [hero]="控制器中的属性值"></app-child>

我们在父组件的模板中调用子组件时，只要在子组件中对`hero`属性进行绑定就行了。是不是很简单？而**从子组件向父组件传值**也很简单，其实就是一个反向的过程，我们需要借助EventEmitter的订阅和广播的功能就可以了，首先我们在子组件中定义要传输出去的属性值：

	export class childComponent {
		@Output() voted = new EventEmitter<boolean>();
		didVote = false;

		vote(agreed: boolean) {
			this.voted.emit(agreed);
			this.didVote = true;
		}
	}

我们将要输出的是通过EventEmitter进行实例化，然后在需要的时候触发`.emit(agreed)`事件就能将数据传出去。那接下来我们需要在父组件中进行接收：

	 <app-child (voted)="onVoted($event)"></app-child>
	
我们在父组件模板中的子组件上加上一个事件绑定，并且在父组件中定义`onVoted()`方法：

	export class ParentComponent {
		agreed = 0;
		disagreed = 0;

		onVoted(event: boolean) {//event就是传递过来的数据
			agreed ? this.agreed++ : this.disagreed++;
	}

在父组件的方法中的参数就是我们从子组件中拿到的数据。这样就成功的传递出了信息。但是需要注意的是：**默认情况下父组件模板中捕获事件的名字和子组件中输出属性的名字是一样的，都是voted！！！**但是我们也可以通过输出属性来重命名：

	@Output('XXX')//这样就可以改掉输出属性的名字，同样的捕获事件的名字也要同步改过来。
***

## 中间人模式
父子组件之间传递数据容易实现，但是当我们的组件不是父子关系的时候我们应该怎么传递数据？这就要用到我们的中间人模式了。

中间人模式最简单的一种关系就是**两个组件有共同的父组件**，也就是两个组件是兄弟关系。但是这个比喻不贴切，他们只是拥有共同的父组件，其实他们两之间谁都不认识谁。那么他们之间怎么通信呢？就把父组件当成中间人就行了：

	export class childOneComponent {
		@Output() voted = new EventEmitter<boolean>();
		didVote = false;

		vote(agreed: boolean) {
			this.voted.emit(agreed);
			this.didVote = true;
		}
	}

首先还是一个组件中输出一个属性，并在适当的时触发广播的行为，将我们的数据发送出去。接着需要在父组件中监听`voted`这个事件：

	<app-child-one (voted)="veteHandler($event)"></app-child-one>

我们的另一个子组件需要有个输入属性：

	export class childTwoComponent {
		@Input() voteCount;
	}

既然是我们的中间人，它就应该先接收它收到的数据，然后将数据再传给需要的人。这就很简单啦：

	
	export class ParentComponent {
		voteCount;

		veteHandler(event: boolean) {//event就是传递过来的数据
			this.voteCount = event;
	}

我想你已经猜到下一步应该怎么办了，我们将父组件接受的数据传给另一个子组件就行了，怎么传？通过属性绑定来传递啊：

	<app-child-one (voted)="veteHandler($event)"></app-child-one>
	<app-child-two [voteCount]="voteCount"></app-child-two>

这样我这个中间人的职责就完成啦，成功将一个组件传递给另一个组件。而且他们两互相之间根本不知情。

***
**那么没有共同的父组件的组件之间难道就不能通讯了吗？答案肯定是不是的，我将在下一篇博客中回答！**