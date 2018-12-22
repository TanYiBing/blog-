---
title: 【翻译】调试Rxjs（一）：工具
date: 2018-11-19 20:34:46
tags: 
- Rxjs
categories:
- Rxjs
---
# [翻译] 调试Rxjs（一）：工具

> 原文：[Debugging RxJS, Part 1: Tooling](https://blog.angularindepth.com/debugging-rxjs-4f0340286dd3)

我是一个 `Rxjs` 的信仰者，我在我所有的项目中都使用 `Rxjs`。有了 `Rxjs`，我发现很多曾经觉得乏味的事都变得痛快。但是有一件事不是这样：调试。

`Rxjs` 中异步的本质在组合之后让调试变得更具挑战性：没有太多的状态（state）供你检查，并且调用堆栈（call stack）也帮不了太大的忙。我过去使用的方法是在整个代码多处添加 `do` 操作符并且记录，以此来检查那些组合的 `Observable` 产生的值。这并不是我想要的方法，因为：

1. 当我在调试时修改代码，我不得不进行更多的日志记录工作；
2. 当调试结束之后，我必须删除日志记录或者注释掉它；
3. 当在一个正常的组合Observable中存在‘拍扁’的操作时，如何进行日志记录需要格外的小心。
4. 就算是专门有的 `log` 操作符，结果也不会很理想。

最近，我留了一些时间来为 `Rxjs` 构建一个调试工具，我觉得这个工具必须具备以下的功能：

1. 应该尽可能的不显眼；
2. 不需要通过修改代码来进行调试；
3. 在调试结束后，不需要删除或注释掉调试的代码；
4. 应该可以轻松的启用和禁用日志记录；
5. 它应该提供与浏览器控制台的一些集成————用于打开/关闭调试用能和检查状态等。

如果想要更完美，还要一些东西：

1. 它应该支持暂停 `Observable`；
2. 它应该支持修改 `Observable` 或者它们发出的值；
3. 它应该支持控制台意外的日志记录机制；
4. 它应该是可扩展的;
5. 它应该在某种程度上可以捕获可视化订阅依赖关系所需的数据。

考虑到这些功能，我建立了 `rxjs-spy`。

## 核心概念
`rxjs-spy` 引入了 `tag` 操作符，将字符串标记与 `Observable` 相关联。这个操作符不会以任何方式更改 `Observable` 的行为或值。

`tag` 操作符可以被单独导入： `import "rxjs-spy/add/operator/tag"`。并且其他的 `rxjs-spy` 方法可以在生产环境下省略，所以唯一的开销就是字符串注释。

大多数工具的方法接受匹配器，以确定它们将应用于哪些标记的 `Observable` 。匹配器可以是传递标签本身的简单字符串，正则表达式或谓词。

通过调用 `spy` 来配置工具时，它会修改 `Observable.prototype.subscribe` 来监听所有的 `subscriptions`, `notifications ` 和 `unsubscriptions`。这也就是意味着，只有已经被订阅的 `Observable` 才会被监听。

`rxjs-spy` 公开了一个旨在从代码中调用的模块API和一个用于在浏览器控制台中交互使用的控制台API。大多数时候，我早早地在应用程序启动代码里条用模块API的方法，并使用控制台API进行剩余的调试。

## 控制台API功能
在调试时，我通常使用浏览器的控制台来检查和操作标记的 `Observables`。控制台API功能最容易通过示例解释：

```js
import { Observable } from "rxjs/Observable";
import { spy } from "rxjs-spy";

import "rxjs/add/observable/interval";
import "rxjs/add/operator/map";
import "rxjs/add/operator/mapTo";
import "rxjs-spy/add/operator/tag";

spy();

const interval = new Observable
  .interval(2000)
  .tag("interval");

const people = interval
  .map((value) => {
    const names = ["alice", "bob"];
    return names[value % names.length];
  })
  .tag("people")
  .subscribe();
```

`rxjs-spy` 中的控制台API通过 `rxSpy` 在全局暴露。

调用 `rxSpy.show()` 将显示所有已经被标记的 `Observable` 的列表，指示其状态（`incomplete`，`complete ` 或者 `errored`），订阅者（`subscribers `）的数量和最近发出的值（如果已经发出一个值）。控制台输出将如下所示：

![Console_1](/img/rxjs/1.png)

要显示特定标记的 `Observable` 的信息，可以将标记名称或者正则表达式传递给 `show`：

![Console_2](/img/rxjs/2.png)

可以通过调用 `rxSpy.log` 来显示被标记的 `Observable` 的日志信息：

![Console_3](/img/rxjs/3.png)

`log` 不带参数调用将会显示所有标记的 `Observable` 的日志记录。

模块API中的大多数方法都返回一个撤销功能，可以调用该功能来撤销方法调用。在控制台中，管理起来很繁琐，所以还有另一种选择。

调用 `rxSpy.undo()` 将显示已经调用的方法的列表：

![Console_4](/img/rxjs/4.png)

调用 `rxSpy.undo` 并传递与方法调用关联的数字将看到该调用的撤销函数被执行。例如，调用 `rxSpy.undo(3)` 将看到被标记为 `interval` 的 `Observable` 的记录被撤销之后的结果：

![Console_5](/img/rxjs/5.png)

有时，在调试时修改 `Observable` 或其值时，这个方法就很有用。控制台API包含一种 `let` 方法，其功能与 `RxJS` 中的 `let` 操作符大致相同。它的实现方式是通过调用 `let` 方法对标记的 `Observable` 的当前和未来的订阅者产生影响。例如。以下调用将看到 `people` `Observable` 发射 `mallory` 而不是 `alice` 或 `bob`:

![Console_6](/img/rxjs/6.png)

与 `log` 方法一样，`let` 可以撤消对该方法的调用：

![Console_7](/img/rxjs/7.png)

能够在调试时暂停一个 `Observable` 对我来说几乎是不可或缺的。调用 `rxSpy.pause` 将暂停一个标记的 `Observable`，并返回一个可用于控制和检查 `Observable` 的通知（`notifications`）的 `deck`：

![Console_8](/img/rxjs/8.png)

在该 `deck` 上调用 `log` 将显示 `Observable` 是否暂停，并显示被暂停的通知（`notifications`）。（通知是 `Notification` 使用 `materialize` 操作符获得的rxjs实例）

![Console_9](/img/rxjs/9.png)

在 `deck` 上调用 `step` 将发出一个被暂停住的通知（`notifications`）：

![Console_10](/img/rxjs/10.png)

调用 `resume` 将发出所有被暂停的通知（`notifications`），并将恢复 `Observable`：

![Console_11](/img/rxjs/11.png)

调用 `pause` 将看到 `Observable` 重新进入暂停状态：

![Console_12](/img/rxjs/12.png)

很容易忘记将返回的 `deck` 分配给变量，因此控制台API包含一个 `deck` 方法，和 `undo` 方法行为相似。调用它将显示 `pause` 调用的列表：

![Console_13](/img/rxjs/13.png)

调用它并传递与调用相关联的数字将返回相对应的 `deck` ：

![Console_14](/img/rxjs/14.png)

像 `log` 和 `let` 的调用一样，`pause` 的调用也可以撤销。撤销 `pause` 的调用将看到标记的 `Observable` 恢复正常：

![Console_15](/img/rxjs/15.png)

希望以上的例子可以对 `rxjs-spy` 的控制台API进行一个概述。`Debugging RxJS` 的后续部分将重点介绍 `rxjs-spy` 的具体功能以及如何使用它们来解决实际的调试问题。

对我来说，`rxjs-spy` 肯定让调试Rxjs不再那么繁琐。

## 更多信息
`rxjs-spy` 的代码可以在 [GitHub](https://github.com/cartant/rxjs-spy)上找到，并且有一个[在线的控制台API示例](https://cartant.github.io/rxjs-spy/)。

该包可以通过[NPM](https://www.npmjs.com/package/rxjs-spy)进行安装。

有关本系列的一下篇文章，正在翻译中，敬请期待。。。