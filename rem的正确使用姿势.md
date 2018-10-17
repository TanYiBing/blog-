---
title: rem的正确使用姿势
date: 2018-08-27 22:14:03
tags:
- JavaScript 
- HTML
- CSS
categories:
- JavaScript
---
最近用的比较多的就是rem了，所以写一篇rem的文章来记录下使用rem来制作移动端的页面的关键之处。

首先简单的阐述下px、em、rem三者之间的关系：

**px**：像素是相对于显示器屏幕分辨率而言的相对长度单位。pc端使用px倒也无所谓，可是在移动端，因为手机分辨率种类颇多，不可能一个个去适配，这时px就显得非常无力，所以就要考虑em和rem。

**em**：继承父级的，假设html的font-size默认为16px，body字体大小定义为50%，那么在body里字体大小就是1em=8px了。可当你又定义了一个div，然后把字体设置成了50%，请问，现在div下的1em等于多少？因为继承了父级的值，现在这个div里的1em=4px，这么嵌套下去的话，抱歉，我数学不好！所以rem就出现了。

**rem**：是em的升级版，rem只会相对html的值，不会受到父级的影响，这样的好处在于：从em里的例子来讲，1rem始终会等于8px。使用的时候不需要重新计算rem此时的大小。rem因为是css3增加的，所以ie8或以下请无视（始终想不明白，为什么国人至今对微软都放弃的ie这么恋恋不舍）。

*使用方法*：
首先在css中先全局声明font-size=625%,这里为什么用625%呢？因为100%=16px,1px=6.25%,所以100px=625%，1rem=100px。这样在后面使用的时候就方便很多。需要注意的是，rem是相对于根元素html的font-size，也就是说只需要设置根元素html的百分比就行了：

    html{font-size: 625%;}

还可以动态的来设置rem，例如使用媒体查询：

    @media screen and (min-width: 320px) {
      html {font-size: 14px;}
	}	
    @media screen and (min-width: 360px) {
      html {font-size: 16px;}
	}
	@media screen and (min-width: 400px) {
      html {font-size: 18px;}
	}
	@media screen and (min-width: 440px) {
      html {font-size: 20px;}
	}
	@media screen and (min-width: 480px) {
      html {font-size: 22px;}
	}
	@media screen and (min-width: 640px) {
      html {font-size: 28px;}
	}

或者利用js计算当前设备来设置：

    (function (doc, win) {
    	var docEl = doc.documentElement,
        resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
        recalc = function () {
        	var clientWidth = docEl.clientWidth;
        	if (!clientWidth) return;
       		if(clientWidth>=640){
        		docEl.style.fontSize = '100px';
        	}else{
				//这里的640是根据设计图实际大小来的
        		docEl.style.fontSize = 100 * (clientWidth / 640) + 'px';
        	}
       	};
        if (!doc.addEventListener) return;
            win.addEventListener(resizeEvt, recalc, false);
            doc.addEventListener('DOMContentLoaded', recalc, false);
    })(document, window);

后来看到一个关于手淘的文章，有了下面这种最佳的方法，通过js计算当前设备的DPR，动态设置在html标签上，并动态的设置html的font-size，利用css选择器根据DPR来设置不同DPR下的字体的大小：

    function(win, lib) {
    	var timer,
        doc     = win.document,
        docElem = doc.documentElement,

        vpMeta   = doc.querySelector('meta[name="viewport"]'),
        flexMeta = doc.querySelector('meta[name="flexible"]'),
 
        dpr   = 0,
        scale = 0,
 
        flexible = lib.flexible || (lib.flexible = {});
 
    	// 设置了 viewport meta
    	if (vpMeta) {
 
        	console.warn("将根据已有的meta标签来设置缩放比例");
        	var initial = vpMeta.getAttribute("content").match(/initial\-scale=([\d\.]+)/);
 
        	if (initial) {
            	scale = parseFloat(initial[1]); // 已设置的 initialScale
            	dpr = parseInt(1 / scale);      // 设备像素比 devicePixelRatio
        	}
		// 设置了 flexible Meta
    	}else if (flexMeta) {
        	var flexMetaContent = flexMeta.getAttribute("content");
        	if (flexMetaContent) {
 
            	var initial = flexMetaContent.match(/initial\-dpr=([\d\.]+)/),
                	maximum = flexMetaContent.match(/maximum\-dpr=([\d\.]+)/);
 
            	if (initial) {
                	dpr = parseFloat(initial[1]);
                	scale = parseFloat((1 / dpr).toFixed(2));
            	}
 
            	if (maximum) {
                	dpr = parseFloat(maximum[1]);
                	scale = parseFloat((1 / dpr).toFixed(2));
            	}
        	}
    	}
 
    	// viewport 或 flexible
    	// meta 均未设置
    	if (!dpr && !scale) {
        	// QST
        	// 这里的 第一句有什么用 ?
        	// 和 Android 有毛关系 ?
        	var u = (win.navigator.appVersion.match(/android/gi), win.navigator.appVersion.match(/iphone/gi)),
            _dpr = win.devicePixelRatio;
 
        // 所以这里似乎是将所有 Android 设备都设置为 1 了
        	dpr = u ? ( (_dpr >= 3 && (!dpr || dpr >= 3))
                        ? 3
                        : (_dpr >= 2 && (!dpr || dpr >= 2))
                            ? 2
                            : 1
                  )
                : 1;
 
        	scale = 1 / dpr;
    	}
 
    	docElem.setAttribute("data-dpr", dpr);
 
    	// 插入 viewport meta
    	if (!vpMeta) {
        	vpMeta = doc.createElement("meta");
         
        	vpMeta.setAttribute("name", "viewport");
        	vpMeta.setAttribute("content",
            "initial-scale=" + scale + ", maximum-scale=" + scale + ", minimum-scale=" + scale + ", user-scalable=no");
 
        if (docElem.firstElementChild) {
            docElem.firstElementChild.appendChild(vpMeta)
        } else {
            var div = doc.createElement("div");
            div.appendChild(vpMeta);
            doc.write(div.innerHTML);
        }
    	}
 
    	function setFontSize() {
        	var winWidth = docElem.getBoundingClientRect().width;
 
        	if (winWidth / dpr > 540) {
            	(winWidth = 540 * dpr);
        	}
 
        	// 根节点 fontSize 根据宽度决定
        	var baseSize = winWidth / 10;
 
        	docElem.style.fontSize = baseSize + "px";
        	flexible.rem = win.rem = baseSize;
    	}
 
    	// 调整窗口时重置
    	win.addEventListener("resize", function() {
        	clearTimeout(timer);
        	timer = setTimeout(setFontSize, 300);
    	}, false);
 
     
    	// 这一段是我自己加的
    	// orientationchange 时也需要重算下吧
    	win.addEventListener("orientationchange", function() {
        	clearTimeout(timer);
        	timer = setTimeout(setFontSize, 300);
    	}, false);
 
 
    	// pageshow
    	// keyword: 倒退 缓存相关
    	win.addEventListener("pageshow", function(e) {
        	if (e.persisted) {
            	clearTimeout(timer);
            	timer = setTimeout(setFontSize, 300);
        	}
    	}, false);
 
    	// 设置基准字体
    	if ("complete" === doc.readyState) {
        	doc.body.style.fontSize = 12 * dpr + "px";
    	} else {
        	doc.addEventListener("DOMContentLoaded", function() {
            	doc.body.style.fontSize = 12 * dpr + "px";
        	}, false);
    	}
  
    	setFontSize();
 
    	flexible.dpr = win.dpr = dpr;
 
    	flexible.refreshRem = setFontSize;
 
    	flexible.rem2px = function(d) {
        	var c = parseFloat(d) * this.rem;
        	if ("string" == typeof d && d.match(/rem$/)) {
            	c += "px";
        	}
        	return c;
    	};
 
    	flexible.px2rem = function(d) {
        	var c = parseFloat(d) / this.rem;
         
        	if ("string" == typeof d && d.match(/px$/)) {
            	c += "rem";
        	}
        	return c;
    	}
	}(window, window.lib || (window.lib = {}));