---
title: angular6在使用antd的layout之后没有css样式
date: 2018-08-22 01:37:10
tags:
- Angular
- Typescript
categories:
- Angular
---

今天我在使用Angular6安装完antd之后，使用了layout模块，打开之后发现没有样式，最后解决方案如下：

**首先我们看看antd的官网**：

[https://ng.ant.design/docs/getting-started/zh](https://ng.ant.design/docs/getting-started/zh)

当我们开始使用之前，我们需要引入css全局样式，但是引入的位置不正确就会导致我们的模块没有样式。

>![](/img/angular/1.png)

**解决方案是在全局样式文件style.css的开头引入antd的css**

`@importnode_modules/ng-zorro-antd/src/ng-zorro-antd.min.css`

**如下图**：
>![](/img/angular/2.png)