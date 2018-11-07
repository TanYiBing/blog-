---
title: bootstrap使用总结
date: 2018-11-06 21:05:57
tags:
- Bootstrap
categories:
- Bootstrap
---

好久咩有写博客了，因为最近在撸很多页面，使用的是`Bootstrap`，因为设计没有给我两套设计图，所以只能使用这个框架来兼容移动端。现在也可以总结下我使用过程中掌握了啥新知识：

1. 因为我用的是3这个大版本，所以设备还是分为四种：xs（768）、sm（768-992）、md（992-1200）、lg（1200）。不用说了，这是栅格系统的核心，通过媒体查询来将一行分为12列的。每个col都会带有左右两边15px的padding。
2. `container`和`container-fluid`的区别是container-fluid是撑满整个屏幕的宽度，但是他两都有左右两边各15px的padding，为的是防止文字显示触摸到屏幕边界，如果不想要这个padding，可以在`container`和`container-fluid`中加上`row`，因为row的左右margin都是-15px，正好抵消padding。关于这个padding和margin其实可以很巧妙的使用，如果我们想在column中嵌套column，先把要被嵌套的column放到row中，再把row放到作为容器的column中，而不需要在放置一个container。这也是因为`Bootstrap`在一开始便说了:
	> 你的内容应当放置于“列（column）”内，并且，只有“列（column）”可以作为行（row）”的直接子元素。
	> 
	> “行（row）”必须包含在 .container （固定宽度）或 .container-fluid （100% 宽度）中，以便为其赋予合适的排列（aligment）和内补（padding）。
3. 还有一点是container和container-fluid是不能嵌套的，这一点我发现公司之前的项目中很多地方都没有注意。
4. 我们还可以通过`col-xxx-offset-xx`、`col-xxx-pull-xx`、`col-xxx-pull-xx`来实现跳过xx列、向右推xx列、向左推xx列的效果。
5. 当我的有先样式在移动端实在是不好展现的时候，这时我们就需要两个不一样的结构，一个在大屏上展示，一个在小屏上展示。这就需要用到`visibile-xs-xxx`、`hide-xs-xxx`、`visibile-xs-block/inline/block-inline`这些来控制展示。
6. 还有一点是form表单一定要添加`label`然后可以通过设置`.sr-only`来隐藏，这是为了给盲人更好的体验，同时也是seo的一种技巧。

以上是一些零零散散的感悟。我在工作的时候发现，有些人不能拎清`自适应`和`响应式`的区别，嘴里老说着‘自适应、自适应’，却只拿出一套设计图。其实响应式才是一套设计适应所有端，而自适用是需要多套设计图的。

在开发过程中发现有的地方的结构在移动端就是不好看，就需要改变结构，一旦地方多了的话其实不如设计两套。

还有一点就是bootstrap的定制，定制比较方便，可以节省很多代码，其实还有更好的方法就是修改源码，这样的话我们的自由度更高，可以更加深入的定制。
