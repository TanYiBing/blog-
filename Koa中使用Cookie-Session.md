---
title: Koa中使用Cookie & Session
date: 2018-09-06 10:43:21
tags: 
- Node 
- Koa 
categories:
- Node
---
# koa中cookie和session使用
---
### cookie介绍
cookie 是存储于访问者的计算机中的变量（客户端）。可以让我们用同一个浏览器访问同一个域名的时候共享数据。那为什么不使用http呢？很简单，因为HTTP 是无状态协议。简单地说，当你浏览了一个页面，然后转到同一个网站的另一个页面，服务器无法认识到这是同一个浏览器在访问同一个网站。每一次的访问，都是没有任何关系的。

### cookie用处
随便举几个例子：

1.	存储用户信息，例如登陆信息。
2.	浏览历史记录。
3.	猜你喜欢的功能。
4.	10天免登陆。
5.	多个页面之间的数据传递。
6.	实现购物车功能。

### Koa Cookie的使用

###### Koa中设置Cookie的值

	ctx.cookies.set(name, value, [options])

通过options设置cookie name的value:

|options名称|options值|
|:----------:|---------|
|maxAge|一个数字表示从 Date.now() 得到的毫秒数。|
|expires|cookie 过期的 Date|
|path|cookie 路径, 默认是'/'。|
|secure|安全 cookie 默认 false，设置成 true 表示只有 https 可以访问。|
|httpOnly|是否只是服务器可访问 cookie, 默认是 true|
|overwrite|一个布尔值，表示是否覆盖以前设置的同名的 cookie (默认是 false). 如果是 true, 在同一个请求中设置相同名称的所有 Cookie（不管路径或域）是否在设置此 Cookie 时从Set-Cookie 标头中过滤掉。|

###### Koa中设置中文Cookie

	console.log(new Buffer('hello, world!').toString('base64'));// 转换成 base64 字符串：aGVsbG8sIHdvcmxkIQ==

	console.log(new Buffer('aGVsbG8sIHdvcmxkIQ==', 'base64').toString());// 还原 base64 字符串：hello, world!

---
### Session介绍
session 是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而Session保存在服务器上。

### Session 的工作流程
当浏览器访问服务器并发送第一次请求时，服务器端会创建一个session 对象，生成一个类似于 key,value的键值对，然后将key(cookie)返回到浏览器(客户)端，浏览器下次再访问时，携带key(cookie)，找到对应的Session(value)。 客户的信息都保存在Session中。

### koa-session 的使用

	npm install koa-session --save

###### 引入 express-session

	const session = require('koa-session');

###### 设置官方文档提供的中间件
	
	app.keys = ['some secret hurr'];
	const CONFIG = {
		key: 'koa:sess', //cookie key (default is koa:sess)
		maxAge: 86400000, // cookie 的过期时间 maxAge in ms (default is 1 days)
		overwrite: true, //是否可以 overwrite (默认 default true)
		httpOnly: true, //cookie 是否只有服务器端可以访问 httpOnly or not (default true)
		signed: true, //签名默认 true
		rolling: false, //在每次请求时强行设置 cookie，这将重置 cookie 过期时间（默认：false）
		renew: false, //(boolean) renew session when session is nearly expired,
	};

	app.use(session(CONFIG, app));

###### Cookie 和 Session 区别

1.	cookie 数据存放在客户的浏览器上，session 数据放在服务器上。
2.	cookie 不是很安全，别人可以分析存放在本地的 COOKIE 并进行 COOKIE 欺骗考虑到安全应当使用 session。
3.	session 会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能考虑到减轻服务器性能方面，应当使用 COOKIE。
4.	单个 cookie 保存的数据不能超过 4K，很多浏览器都限制一个站点最多保存 20 个 cookie。