---
layout: 【翻译】调试
title: 【翻译】调试Rxjs（二）：日志记录
date: 2018-11-25 23:32:52
tags: 
- Rxjs
categories:
- Rxjs
---
# [翻译] 调试 Rxjs（二）：日志记录

> 原文：[Debugging RxJS, Part 2: Logging](https://blog.angularindepth.com/debugging-rxjs-part-2-logging-56904459f144)
>
> 译者：[Ice Panpan](https://github.com/TanYiBing)；校验者：暂无

![banner](/img/rxjs/17/banner_01.jpeg?raw=true)

日志记录并不是一件让人兴奋的事。

然而，这是获得足够的信息来推理问题最直接的方法，而不需要去猜测。它通常是调试 `RxJS` 代码的首选方法。这是这个系列文章的第二篇，专注于使用日志记录来解决实际问题。在第一篇[调试 Rxjs（一）：工具](./16.[翻译]-调试-Rxjs（一）：工具.md)中,主要介绍的是 [`rxjs-spy`](https://github.com/cartant/rxjs-spy)。在本文中，我将展示如何使用 `rxjs-spy` 以最小的影响来获取详细并有针对性的信息。

让我们看一个使用 `rxjs` 和 `rxjs-spy` UMD捆绑的简单案例：

```js
RxSpy.spy();
RxSpy.log(/user-.+/);
RxSpy.log('users');

const names = ['benlesh', 'kwonoj', 'staltz'];
const users = Rx.Observable.forkJoin(...names.map(name =>
  Rx.Observable
    .ajax
    .getJSON(`https://api.github.com/users/${name}`)
    .tag(`user-${name}`)
))
.tag('users');

users.subscribe();
```

该示例使用 `forkJoin` 组合了一个用来发射出GitHub用户数组的 `Observable`。

`rxjs-spy` 使用 `tag` 操作符来标记 `Observable`，并且仅仅通过字符串来给 `Observable` 注释。这个示例在组合 `Observable` 之前，首先启用监听功能，并配置了哪些 `Observable` 要被记录——匹配 `/user-.+/` 的正则表达式或者带有 `users` 标签的那些 `Observable`。

这个示例的控制台输出如下：

![Console_01](/img/rxjs/17/console_01.png?raw=true)

除了 `Observable` 的 `next` 和 `complete` 的通知之外，记录的输出还包括订阅和取消订阅的通知。它显示了发生的一切：

1. 对组合的 `Observable` 的订阅影响的每一个用户API请求的 `Observable` 的并行订阅；
2. 请求以任意顺序完成；
3. `Observable` 全部完成；
4. 并且在全部完成后取消对组合 `Observable` 的订阅。

每个记录的通知还包括有关接受通知的订阅者的信息——包括订阅者具有的订阅量以及 `subscribe` 调用的堆栈痕迹：

![Console_02](/img/rxjs/17/console_02.png?raw=true)

堆栈痕迹指的是 `subscribe` 调用的根——即影响订阅者对 `Observable` 订阅的显式调用。因此，用户请求的 `Observable` 的堆栈痕迹也参考了 `medium.js` 中的 `subscribe` 调用：

![Console_03](/img/rxjs/17/console_03.png?raw=true)

当我调试时，我发现知道调用 `subscribe` 的实际的根位置比知道组合 `Observable` 中某个 `subscribe` 的位置更有用。

现在让我们看一个现实中实际的问题。

在编写 `redux-observable epics` 或 `ngrx effects` 时，我看到几个开发人员类似的代码：

```js
import { Observable } from 'rxjs/Observable';
import { ajax } from 'rxjs/observable/dom/ajax';

const getRepos = action$ =>
  action$.ofType('REPOS_REQUEST')
    .map(action => action.payload.user)
    .switchMap(user => ajax.getJSON(`https://api.notgithub.com/users/${user}/repos`))
    .map(repos => { type: 'REPOS_RESPONSE', payload: { repos } })
    .catch(error => Observable.of({ type: 'REPOS_ERROR' }))
    .tag('getRepos');
```

乍一看，这看上去还不错。大部分时间它都可以正常工作，它是那种可以偷偷混过单元测试的bug。

问题是，它有时会停止工作，特别是在一个错误的动作发生之后。

记录显示正在发生的事：

![Console_04](/img/rxjs/17/console_04.png?raw=true)

在错误的动作被发射出去之后，看到 `redux-observalbe`基础结构从epic中解除订阅的 `Observable` 完成了。该[文档](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-catch)的 `catch` 解释了为什么出现这种情况。

>无论 `selector` 返回了什么样的 `Observable`，都将连在 `Observable` 链后面。

在这个epic中，`catch` 完成后返回的 `Observable` 也看到了epic的完成。

解决的方案是将 `map` 和 `catch` 移到 `switchMap` 中，像下面这样：

```js
import { Observable } from 'rxjs/Observable';
import { ajax } from 'rxjs/observable/dom/ajax';

const getRepos = action$ =>
  action$.ofType('REPOS_REQUEST')
    .map(action => action.payload.user)
    .switchMap(user => ajax
      .getJSON(`https://api.notgithub.com/users/${user}/repos`)
      .map(repos => { type: 'REPOS_RESPONSE', payload: { repos } })
      .catch(error => Observable.of({ type: 'REPOS_ERROR' }))
    )
    .tag('getRepos');
```

这个epic将不再完成，并继续发送错误动作：

![Console_05](/img/rxjs/17/console_05.png?raw=true)

在这两个示例中，需要对正在调试的代码进行的唯一修改是添加了一些标记注释。

注释的影响很小，一旦添加，我倾向于将它们留在代码中。标签运算符可以独立于诊断 `rxjs-spy` 使用——使用`rxjs-spy/add/operator/tag` 或直接导入 `rxjs-spy/operator/tag`，因此保留标记的开销很小。

可以使用正则表达式配置记录器，这可以产生许多可能的标记方法。例如，使用复合标记,例如 `github/users` 和 `github/repos` 将允许您为所有被标记后存储在存储库里的 `github Observable` 启用日志记录。

记录并不令人兴奋，但可以从记录的输出中收集的信息通常可以节省大量时间。采用灵活的标记方法可以进一步减少处理与日志记录相关的代码所花费的时间。