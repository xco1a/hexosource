---
title: jQuery-core.js（整体架构部分）阅读笔记
date: 2018-05-31 21:32:29
tags:
- 学习笔记
- jQuery
categories: 
- 前端
---
<!-- more -->

&emsp;&emsp;突然有了看看jQuery源码的念头。希望在拜读源码的过程中，能学习到一些关于构建库和模块的思路。

&emsp;&emsp;源码下载自github，目前为最新版3.3.1，先从总体架构看起 —— core,js。摘取部分并整理一下。

```javascript
var jQuery = window.jQuery = window.$ = function( selector, context ) {		
	return new jQuery.fn.init( selector, context );			
};
var quickExpr = /^[^<]*(<(.|\s)+>)[^>]*$|^#(\w+)$/,	
	isSimple = /^.[^:#\[\.]*$/,
	undefined;	
jQuery.fn = jQuery.prototype = {			
	init: function( selector, context ) {},
    
    	size: function() {},
    // 返回jq对象中个元素个数
    	get: function( num ) {},
    // 以类似数组索引的方法取得一个jq对象中的元素
    	pushStack: function( elems ) {},
    // 将一个元素压入jq栈
    	each: function( callback, args ) {},
    // 迭代器，可遍历数组和对象
    	index: function( elem ) {},
    // 搜寻指定元素，返回其位置
    	attr: function( name, value, type ) {},
    // 获取匹配元素集合中第一个元素的属性值，或为每个匹配元素设置一个或多个属性
    	css: function( key, value ) {},
    // 获取匹配元素中第一个元素的样式或者给指定的元素设定CSS样式
    	wrapAll: function( html ) {},
    // 在对象外部插入html标签
		append: function() {},
    // 将由参数指定的内容插入匹配元素集合中每个元素的末尾
    	find: function( selector ) {},
	// 获取当前匹配元素中符合参数选择器的对象
	clone: function( events ) {},
	// 深拷贝一组匹配元素的值
	add: function( selector ) {},
	// 以给定的参数创建一个新的jq对象
	is: function( selector ) {},
	// 根据给定的条件检查当前匹配元素，存在匹配时返回true
	val: function( value ) {},
	// 返回当前元素的value值
	html: function( value ) {},
	// 返回或设定当前元素的html标签内容
	replaceWith: function( value ) {},
	// 用提供的内容替换当前匹配元素
	eq: function( i ) {},
	// 以索引的方式取得一组jq对象中的某一个对象
	slice: function() {},
	// 和数组的slice方法类似，可以缩小当前jq对象数组的范围
	map: function( callback ) {},
	// 和数组的map方法类似
	data: function( key, value ){},
	// 向dom元素附加一个数据（键值对）
	domManip: function( args, table, reverse, callback ) {}
};
	// dom操作的核心函数
jQuery.fn.init.prototype = jQuery.fn;
	// 把jQuery的原型传递给init的原型
```



&emsp;&emsp;首先在全局注册jQuery变量（$），随后此变量指向jq的构造函数。有个很有意思的小地方 ——  定义了一个变量叫 undefined，由于没给它赋值，因此它的值也是undefined。这样做的意义可以分为两个方面，首先它减少了程序在作用域链中搜寻undefined值的时间，其次也在防范用户有意无意的改变undefined的值（undefined是可以赋值的，但谷歌浏览器会忽视用户给undefined的赋值）。

&emsp;&emsp;值得学习的是，在构造函数中 “return new ” 的方法使得jq获得了无new构造jq对象的特性，但是，如果直接return 自己的构造函数无疑会造成无限递归自己的问题。如下所示。

```javascript
var jQuery = function(selector){
    this.selector = selector;
    return new jQuery(selector);
}
```



&emsp;&emsp;因此这个构造函数把构造对象的任务交给了jQuery.fn.init函数。jQuery.fn.init实际上是jQuery.prototype.init，在这个过程中，分离出两个不同的构造器 —— jQuery.prototype和jQuery.prototype.init.prototype。到了这里虽然解决了无new操作且避免递归的问题，也自然而然引出另一个问题 —— 构建jq对象的任务是在jQuery.prototype.init.prototype中完成的，所以构建出的对象其原型会指向jQuery.prototype.init.prototype而不是jQuery.prototype，其this也无法引用到jQuery的原型方法。

&emsp;&emsp;比如，在jQuery.prototype中，还有很多种方法，在上面的代码块中仅列出一部分函数的声明，注释中给出了每个方法的作用。这些方法均为实例方法，实例化出的jq对象即可调用。

&emsp;&emsp;另外，jQuery本身还存在很多静态方法，这些方法由jQuery直接调用，例如扩展方法（extend）。可以发现，在这其中存在静态和实例方法中有共用的方法，即既是静态方法，也是实例方法，举个栗子。

```javascript
each: function( callback, args ) {
		return jQuery.each( this, callback, args );
	},
```



&emsp;&emsp;each方法可以被实例化的对象调用也可以被jQuery本身调用。且这个静态方法的实现还依赖于实例方法。但jq实例是通过init构造器构造的，init构造器和jQuery构造器又是分离的，那么怎么解决这个问题呢，在这段程序的最后。

`jQuery.fn.init.prototype = jQuery.fn;`

&emsp;&emsp;把jQuery的原型传递给init的原型，这样jQuery.prototype.init.prototype会指向jQuery.prototype，事实上也就是jQuery的原型对象覆盖了init构造器的原型对象。通过这种方法，将两个分离的构造器整合在了一起。当用户使用$()时，首先通过init构造器创建了一个jq对象，其次通过jQuery.fn.init.prototype = jQuery.fn的方法将实例方法挂载，静态方法又依赖于实例方法才得以实现，至此，通过init构造器完成了无new化jq对象的构建、通过init和jQuery的原型传递实现了实例方法挂载、在实例方法中call或者apply方法又实现了jQuery静态方法的挂载。

&emsp;&emsp;这一部分原理和思路是我在参考了很多前辈和大神的分析讲解后才得以勉强理解。在构建对象的核心模块上尽显其思路之巧妙写法之风骚。此外，core.js中在实例方法和静态方法的处理中展现出的编程技巧也十分值得学习。阅读源码过程艰难，基本就处于理解偏差乃至根本看不懂的尴尬境地，但希望拜读之后，这些思路和技巧亦能成为我的血肉。