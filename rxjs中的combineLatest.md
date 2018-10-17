---
title: rxjs中的combineLatest
date: 2018-10-17 11:43:35
tags:
- Angular
- Rxjs
categories:
- Rxjs
---
在Rxjs第六版之前，我们使用`combineLatest`这个`operator`时的方式如下：

	 const age$ = Observable
		.combineLatest(ageNum$, ageUnit$, (_num, _unit) => this.toDate({age: _num, unit: _unit}))
		.map(d => ({date: d, from: 'age'}))
		.filter(_ => this.form.get('age').valid);

但是，在Rxjs第六版中`combineLatest`这个`operator`被遗弃了，而是被改成一个function，然后配合`pipe()`使用，改造后的方法如下：

	const age$ = combineLatest(ageNum$, ageUnit$).pipe(
		map(([_n, _u]) => this.toDate({ age: _n, unit: _u })),
		map(d => {
			return { date: d, from: 'age' };
		}),
		filter(_ => this.form.get('age').valid)
	);

这之间发生了明显的变化，首先调用的方式就不同，版本六之前是调用`Observable.combineLatest`，而在版本六之后直接就可以使用`combineLatest()`；还有一个不同就是参数不同了，原先的参数最后可以接受一个project，实现对元素的操作，但是第六版之后，我们需要pipe出来之后，使用map来对元素进行操作，**而且元素是在一个数组里面的**，这归根结底是因为返回的类型不一样了，operator返回的就是一个operator，而第六版之后返回的是一个`Observable`对象，我们可以继续对其进行操作。