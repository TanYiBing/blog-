---
title: angular实战中踩过的坑（不定时更新）
date: 2018-09-07 12:08:48
tags:
- Angular
- Typescript
categories:
- Angular
---

## 动态表单中踩过的坑
### 2018-9-7
**问题：**
今天在写动态表单时遇到个问题，我在控制器中生成的`formControl`死活就是拿不到我输入的值。

**分析：**
生成`formControl`的代码肯定没有问题：

	let fb = new FormBuilder();
    this.formModel = fb.group({
      title: ['', Validators.minLength(3)],
      price: [null, this.positiveNumberValidator],
      category: ['-1']
    });
对吧，是没有问题，那肯定就是出在绑定模板的时候，于是我们认真看看吧。

**问题根源：**
>![](/img/angular/9.png)

发现了吧，这个`Name`由于打快了，打成了小写！低级错误！以后拼写细心点！
***
## 版本更新问题
**问题：**
我想要做个商品筛选的功能，其中我需要使用`debounceTime`方法来对我的输入行为产生一个延时的处理，我的调用方法如下：

>![](/img/angular/10.png)

引入的是：
>import 'rxjs/RW';

结果引入报错。

**分析：**
既然引入报错肯定是更新过了，目录结构发生变化了，甚至连方法的使用也有可能改变了，还是去官方找找吧。

**问题根源：**
一查果然是更新的问题，而且不出意料，方法也需要修改下，经过修改后引入的方式如下：
>import { debounceTime } from 'rxjs/operators';

方法也需要改进成下面这种:

>![](/img/angular/11.png)


