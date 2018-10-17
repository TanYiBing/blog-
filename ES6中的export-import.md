---
title: ES6中的export & import
date: 2018-10-07 23:44:24
tags:  
- JavaScript
categories:
- JavaScript
---
上次说了一下[exports和module.exports的区别](http://tanyibing.com/2018/09/19/exports%E5%92%8Cmodule-exports%E7%9A%84%E5%8C%BA%E5%88%AB/)，但是只讲了那一种状态，所以今天特意再来看看ES6中module的内容。

在ES6中，专门实现了模块化的功能。想要更仔细的阅读可以看一看阮一峰的《ESMAScript6入门》中的[Module的语法](http://es6.ruanyifeng.com/#docs/module)。里面是这么说的：

>在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

由此可见ES6中的模块化功能还是很强大的。

### export

先看几种export的写法：

	// 第一种 输出两个变量
	export var firstName = 'Michael';
	export var lastName = 'Jackson';

	// 第二种 和第一种一样，相同情况应该优先使用第二种。
	var firstName = 'Michael';
	var lastName = 'Jackson';
	export {firstName, lastName, year};

	// 第三种 输出函数或类
	export function multiply(x, y) {
		return x * y;
	};

上面的三种情况在注释里解释了，就按这种模式写。但是要注意的是不要写成下面的样子;

	// 报错
	export 1;

	// 报错
	var m = 1;
	export m;

这样写是直接输出，而不是通过接口，所以会报错。应该写成下面这样：

	// 写法一
	export var m = 1;
	
	// 写法二
	var m = 1;
	export {m};
	
	// 写法三
	var n = 1;
	export {n as m};

同样的，`function`和`class`也要按这种接口的方式去写：

	// 报错
	function f() {}
	export f;
	
	// 正确
	export function f() {};
	
	// 正确
	function f() {}
	export {f};

### import

看几个import的写法：

	// 普通写法
	import {firstName, lastName, year} from './profile.js';

	// 重命名写法
	import { lastName as surname } from './profile.js';

	// 整体加载写法
	import * as name from './profile.js';

这就是import的几种写法，不要妖来妖去的，例如使用表达式和变量、修改接口啊什么的操作，即使可以操作，也是极不推荐的，防止出现错误后找不到问题的根源。

### export default

`export default`是一种默认输出的方式，他的形式如下：

	// 输出
	export default function () {
	  console.log('foo');
	}

	// 输入
	import customName from './export-default';
	customName();

可以发现，**默认输出之后，引入时被当成匿名函数，我们可以给他起任意的名字。`export default`可以用在`非匿名函数`前面，但是在外部还是会被当成匿名函数来加载。还可以发现的是，使用默认输出之后，在`import`时不再需要大括号了，这是因为`export default`只能使用一次，所以在引入时可以不加大括号。**

正是因为export default命令其实只是输出一个叫做default的变量，所以它后面不能跟变量声明语句。

	// 正确
	export var a = 1;
	
	// 正确
	var a = 1;
	export default a;
	
	// 错误
	export default var a = 1;

	// 正确
	export default 42;
	
	// 报错
	export 42;

### 复合写法

如果在一个模块之中，先输入后输出同一个模块，import语句可以与export语句写在一起。例如下面：

	export { foo, bar } from 'my_module';

	// 可以简单理解为
	import { foo, bar } from 'my_module';
	export { foo, bar };

上面代码中，export和import语句可以结合在一起，写成一行。但需要注意的是，写成一行以后，foo和bar实际上并没有被导入当前模块，只是相当于对外转发了这两个接口，导致当前模块不能直接使用foo和bar。