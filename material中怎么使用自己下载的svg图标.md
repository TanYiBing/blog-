---
title: material中怎么使用自己下载的svg图标
date: 2018-09-21 14:37:43
tags:
- Angular
- Typescript
- Material
categories:
- Material
---
`material`是谷歌开发的很好看的一个框架，我就在我最近的项目中使用了这个框架，喜欢的童鞋可以去他们的官网看看：[material.io](https://material.io/)

开发过程中我发现他们的图标库咩有阿里的那么丰富，于是想用自己下载的，那我该怎么操作呢？，下面介绍一下流程。

为了方便，我没有在需要的模块中反复的添加，而是创建了一个`svg.util.ts`,并选择在`core module`中一次性加载，省掉很多事。首先我需要在`svg.util.ts`中引入必须的模块：

>![](/img/material/1.png)

这两个模块是核心模块，一个负责注册新加入的svg图标，一个是为了防止XSS漏洞的将URL设为信任对象。

接下来就将我们的图标都注册进去：

>![](/img/material/2.png)	

这样子差不多快可以了，**但我们还需要在`core module`中引入HttpClient模块（这一步必须要，否则会报错）**，因为需要依赖它，最后在`core module`的构造函数中调用就行了。

然后我们就能光明正大的使用啦：

	<mat-icon svgIcon='注册的名称'></mat-icon>