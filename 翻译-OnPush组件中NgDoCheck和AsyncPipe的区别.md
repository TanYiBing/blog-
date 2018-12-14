---
title: '[翻译]OnPush组件中NgDoCheck和AsyncPipe的区别'
date: 2018-12-14 21:59:55
tags: 
- Angular
categories:
- Angular
---

# [翻译] OnPush 组件中 NgDoCheck 和 AsyncPipe 的区别

> 原文：[The difference between NgDoCheck and AsyncPipe in OnPush components](https://blog.angularindepth.com/the-difference-between-ngdocheck-and-asyncpipe-in-onpush-components-4918ec4b29d4)
> 作者：**[Max Koretskyi](http://twitter.com/maxim_koretskyi)**
> 原技术博文由 `Max Koretskyi` 撰写发布，他目前于 [ag-Grid](https://angular-grid.ag-grid.com/?utm_source=medium&utm_medium=blog&utm_campaign=angularcustom) 担任开发者职位

> 译者：**[Ice Panpan](https://github.com/TanYiBing)**

![async or ngdocheck](/img/rxjs/1.jpeg)

这篇文章是对[Shai这条推特](https://twitter.com/shai_reznik/status/1054868497363283968)的回应。他询问使用 `NgDoCheck` 生命周期钩子来手动比较值而不是使用 `asyncPipe` 是否有意义。这是一个非常好的问题，需要对引擎的工作原理有很多了解：变化检测(change detection)，管道(pipe)和生命周期钩子(lifecycle hooks)。那就是我探索的入口😎。

在本文中，我将向您展示如何手动处理变更检测。这些技术使您可以更好地掌控 Angular 的输入绑定(input bindings)的自动执行和异步值检查(async values checks)。掌握了这些知识之后，我还将与您分享我对这些解决方案的性能影响的看法。让我们开始吧！


## OnPush 组件
在 Angular 中，我们有一种非常常见的优化技术，需要将 `ChangeDetectionStrategy.OnPush` 添加到组件中。假设我们有如下两个简单的组件：

```ts
@Component({
    selector: 'a-comp',
    template: `
        <span>I am A component</span>
        <b-comp></b-comp>
    `
})
export class AComponent {}

@Component({
    selector: 'b-comp',
    template: `<span>I am B component</span>`
})
export class BComponent {}
```

这样设置之后， Angular 每次都会对 `A` 和 `B` 两个组件运行变更检测。如果我们现在为 `B` 组件添加上 `OnPush` 策略：

```ts
@Component({
    selector: 'b-comp',
    template: `<span>I am B component</span>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class BComponent {}
```

**只有在输入绑定的值发生变化时** Angular 才会对 `B` 运行变更检测。由于它现在没有任何绑定，因此该组件只会在初始化的时候检查一次。

## 手动触发变更检测
有没有办法强制对 `B` 组件进行变更检测？是的，我们可以注入 `changeDetectorRef` 并使用它的方法 `markForCheck` 来指示 Angular 需要检查该组件。并且由于 [*NgDoCheck 钩子仍然会被 B 组件触发*](https://blog.angularindepth.com/if-you-think-ngdocheck-means-your-component-is-being-checked-read-this-article-36ce63a3f3e5)，所以我们应该在 *NgDoCheck* 中调用 *markForCheck*：

```ts
@Component({
    selector: 'b-comp',
    template: `<span>I am B component</span>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class BComponent {
    constructor(private cd: ChangeDetectorRef) {}

    ngDoCheck() {
        this.cd.markForCheck();
    }
}
```

现在，当 Angular 检查父组件 `A` 时，将始终检查 `B` 组件。现在让我们看看我们可以在哪里使用它。

## 输入绑定
我之前说过，Angular 只在 `OnPush` 组件的绑定发生变化时运行的变化检测。所以让我们看一下输入绑定的例子。假设我们有一个通过输入绑定从父组件传递下来的对象：

```ts
@Component({
    selector: 'b-comp',
    template: `
        <span>I am B component</span>
        <span>User name: {{user.name}}</span>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class BComponent {
    @Input() user;
}
```
在父组件 `A` 中，我们定义了一个对象，并实现了在单击按钮时来更新对象名称的  *changeName*  方法：

```ts
@Component({
    selector: 'a-comp',
    template: `
        <span>I am A component</span>
        <button (click)="changeName()">Trigger change detection</button>
        <b-comp [user]="user"></b-comp>
    `
})
export class AComponent {
    user = {name: 'A'};

    changeName() {
        this.user.name = 'B';
    }
}
```

如果您现在[运行此示例](https://stackblitz.com/edit/angular-kq26qe)，则在第一次变更检测后，您将看到用户名称被打印出来：

```ts
User name: A
```

但是当我们点击按钮并回调中更改名称时：

```ts
changeName() {
    this.user.name = 'B';
}
```

**该名称并没有在屏幕上更新**，这是因为 Angular 对输入参数执行浅比较，并且对 user 对象的引用没有改变。那我们怎么解决这个问题呢？

好吧，我们可以在检测到差异时手动检查名称并触发变更检测：

```ts
@Component({
    selector: 'b-comp',
    template: `
        <span>I am B component</span>
        <span>User name: {{user.name}}</span>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class BComponent {
    @Input() user;
    previousName = '';

    constructor(private cd: ChangeDetectorRef) {}

    ngDoCheck() {
        if (this.previousName !== this.user.name) {
            this.previousName = this.user.name;
            this.cd.markForCheck();
        }
    }
}
```

如果您现在[运行此代码](https://stackblitz.com/edit/angular-8dkwct)，你将在屏幕上看到更新的名称。

## 异步更新
现在，让我们的例子更复杂一点。我们将介绍一种基于 RxJs 的服务，它可以异步发出更新。这类似于 NgRx 的体系结构。我将使用一个 `BehaviorSubject` 作为值的来源，因为我们需要在这个流的最开始设置初始值：

```ts
@Component({
    selector: 'a-comp',
    template: `
        <span>I am A component</span>
        <button (click)="changeName()">Trigger change detection</button>
        <b-comp [user]="user"></b-comp>
    `
})
export class AComponent {
    stream = new BehaviorSubject({name: 'A'});
    user = this.stream.asObservable();

    changeName() {
        this.stream.next({name: 'B'});
    }
}
```

所以我们需要在子组件中订阅这个流并从中获取到 `user` 对象。我们需要订阅流并检查值是否更新。这样做的常用方法是使用 [AsyncPipe](https://angular.io/api/common/AsyncPipe)。

## AsyncPipe
所以这里是子组件 `B` 的实现：

```ts
@Component({
    selector: 'b-comp',
    template: `
        <span>I am B component</span>
        <span>User name: {{(user | async).name}}</span>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class BComponent {
    @Input() user;
}
```

[这是演示](https://stackblitz.com/edit/angular-q8n3qj)。但是，还有另一种不使用管道的方法吗？

## 手动检查并且变更检测
是的，我们可以手动检查值并在需要时触发变更检测。正如开头的例子一样，我们可以使用 `NgDoCheck` 生命周期钩子：

```ts
@Component({
    selector: 'b-comp',
    template: `
        <span>I am B component</span>
        <span>User name: {{user.name}}</span>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class BComponent {
    @Input('user') user$;
    user;
    previousName = '';

    constructor(private cd: ChangeDetectorRef) {}

    ngOnInit() {
        this.user$.subscribe((user) => {
            this.user = user;
        })
    }

    ngDoCheck() {
        if (this.previousName !== this.user.name) {
            this.previousName = this.user.name;
            this.cd.markForCheck();
        }
    }
}
```

你可以[在这查看](https://stackblitz.com/edit/angular-4xuug1)。

我们希望把值的比较与更新逻辑从 `NgDoCheck` 中移至订阅的回调函数，因为我们是从那里获取到新值的：

```ts
export class BComponent {
    @Input('user') user$;
    user = {name: null};

    constructor(private cd: ChangeDetectorRef) {}

    ngOnInit() {
        this.user$.subscribe((user) => {
            if (this.user.name !== user.name) {
                this.cd.markForCheck();
                this.user = user;
            }
        })
    }
}
```

[例子在这](https://stackblitz.com/edit/angular-lvtfve)。

有趣的是，这其实正是 [AsyncPipe 背后的工作原理](https://github.com/maximusk/angular/blob/725bae1921cfcdcf5a5b0c35252c632198d1a7a4/packages/common/src/pipes/async_pipe.ts#L139)：

```ts
@Pipe({name: 'async', pure: false})
export class AsyncPipe implements OnDestroy, PipeTransform {
  constructor(private _ref: ChangeDetectorRef) {}

  transform(obj: ...): any {
    ...
    this._subscribe(obj);

    ...
    if (this._latestValue === this._latestReturnedValue) {
      return this._latestReturnedValue;
    }

    this._latestReturnedValue = this._latestValue;
    return WrappedValue.wrap(this._latestValue);
  }

  private _subscribe(obj): void {
    ...
    this._strategy.createSubscription(
        obj, (value: Object) => this._updateLatestValue(obj, value));
  }

  private _updateLatestValue(async: any, value: Object): void {
    if (async === this._obj) {
      this._latestValue = value;
      this._ref.markForCheck();
    }
  }
}
```

## 那么那种解决方案更快？
现在我们知道如何使用手动进行变更检测而不是使用 AsyncPipe，让我们回答下最一开始的问题。那种方法更快？

嗯...这取决于你如何比较它们，但在其他条件相同的情况下，手动方法会更快。尽管我不认为两者会有明显区别。以下是为什么手动方法可以更快的几个例子。

就内存而言，您不需要创建 Pipe 类的实例。就编译时间而言，编译器不必花时间解析管道特定语法并生成管道特定输出。就运行时间而言，节省了异步管道为组件进行变更检测所调用的函数的时间。这个例子演示了当代码中包含 pipe 时 [updateRenderer](https://blog.angularindepth.com/the-mechanics-of-dom-updates-in-angular-3b2970d5c03d) 所生成的代码：

```ts
function (_ck, _v) {
    var _co = _v.component;
    var currVal_0 = jit_unwrapValue_7(_v, 3, 0, asyncpipe.transform(_co.user)).name;
    _ck(_v, 3, 0, currVal_0);
}
```

如您所见，异步管道的代码调用管道实例上的 `transform` 方法以获取新值。管道将返回从订阅中收到的最新值。

将其与为手动方法生成的普通代码进行比较：

```ts
function(_ck,_v) {
    var _co = _v.component;
    var currVal_0 = _co.user.name;
    _ck(_v,3,0,currVal_0);
}
```

这就是 Angular 在检查 `B` 组件时调用的方法。

## 一些更有趣的事情
与执行浅比较的输入绑定不同，**异步管道的实现根本不执行比较**（感谢 [Olena Horal](https://medium.com/@sharlatenok) 注意到这一点）。它将每个新发射的值认为是更新，即使它与先前发射的值一样。下面的代码是父组件 `A` 的实现，它每次都发射出相同的对象。尽管如此，Angular 仍然会对 `B` 组件进行变更检测：


```ts
export class AComponent {
    o = {name: 'A'};
    user = new BehaviorSubject(this.o);

    changeName() {
        this.user.next(this.o);
    }
}
```

这意味着每次发出新值时，使用异步管道的组件都会被标记以进行检查。并且 Angular 将在下次运行变更检测时检查该组件，即使该值未更改。

这是应用于什么情况呢？嗯...在我们的例子中，我们只关注 `user` 对象的 `name` 属性，因为我们需要在模板中使用它。我们并不关心整个对象以及对象的引用可能会改变的事实。如果 name 没有发生改变，我们不需要重新渲染组件。但你无法用异步管道来避免这种情况。

`NgDoCheck` 并不是没有问题:)由于仅在检查父组件时触发钩子，如果其中一个父组件使用 `OnPush` 策略并且在变更检测期间未检查，则不会触发该钩子。因此，当您通过服务收到新值时，不能依赖它来触发变更检测。在这种情况下，我在订阅回调中调用 `markForCheck` 方法是正确的解决方案。

## 总结
基本上，手动比较可以让您更好地控制检查。您可以定义何时需要检查组件。这与许多其他工具相同 - 手动控制为您提供了更大的灵活性，但您必须知道自己在做什么。为了获得这些知识，我鼓励您投入时间和精力学习和阅读[更多文章](https://blog.angularindepth.com/level-up-your-reverse-engineering-skills-8f910ae10630)。

你不用担心 `NgDoCheck` 生命周期钩子被调用的频率，或者它会比管道的 `transform` 方法更频繁地被调用。首先，我上面已经展示了解决方案，当使用异步流时，你应该在订阅的回调中而非在该钩子函数中手动执行变更检测。其次，只有在父组件被检测后才会调用该钩子函数。如果父组件没有被检查，则不会调用该钩子。对于管道而言，由于流中的浅比较和更改引用的原因，管道的 `transform` 方法被调用的次数只会和手动方法相同甚至更多。

## 想要了解更过关于 Angular 中 change detection 的相关知识？
从这5篇文章入手会让你成为 Angular 变更检测机制的专家。如果你想要牢固掌握 Angular 中变更检测机制，那么这一系列的文章是必读的。每一篇文章都会基于前一篇文章中所解释的相关信息，既包含高层次的概述又囊括了具体的实现细节，并且都附有相关源代码。

