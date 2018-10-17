---
title: node中踩的坑（不定时更新）
date: 2018-09-14 17:02:35
tags: 
- Node 
- Koa 
categories:
- Node
---
### 2018-9-14
**问题**
在Koa中想要使用art-template这个模板引擎，但是使用之后在渲染模板的时候一直报下面的错：
>![](/img/node/1.png)

**分析**
报的错说我的路径必须为一个字符串，我重新检查后，发现并不存在语法错误呀，果断去度娘看看，试了几个方法解决不了我的问题，但是反馈最多的一个方法就是node版本和其他的不兼容。我靠！这要重搞工作量有点大，果断换个模板引擎吧。

**解决方法**
最后回去官网看一看，发现有`art-template`和`koa-art-template`之分，但是配置好像没啥区别啊。把引入的模块改成`koa-art-template`，再试试，成了！什么鬼！不管了，问题暂时是解决了。

**另外附上项目的地址**：[https://github.com/TanYiBing/cms_demo](https://github.com/TanYiBing/cms_demo)