---
title: 关于JavaScript模块化开发的梳理
date: 2018-06-10 14:52:35
tags:
- 学习笔记
categories: 
- 前端
---
<!-- more -->
## commonJS

&emsp;&emsp;在我对js的理解还停留在只用来做做页面认证和特效的时候，它早已牛逼到支持操作系统层面的操作了。根据学习整理，大致梳理出这么一个故事 —— 虽然js天生的弱类型和半吊子面向对象的特性在逐步向强类型语言靠拢，但js没有模块系统、标准库较少、没有包管理工具的痛点阻碍着js发展出开发大型应用的能力。于是[commonJS](http://www.commonjs.org/)跳出来尝试解决这个问题。[commonJS](http://www.commonjs.org/)的首页上就写着

> javascript: not just for browsers any more !

可见[commonJS](http://www.commonjs.org/)是希望Js运行在浏览器以外发挥作用的。

&emsp;&emsp;[commonJS](http://www.commonjs.org/)针对JS模块化开发制定了一系列规范，主要包括模块引用、模块定义、模块标识。规范仅仅是规范，亦可以说是指导，而node.js实现了它。nodejs把Chrome V8从浏览器中剥离出来，又编写了一部分底层代码，添加了对I/O操作的支持，js获得了在操作系统层面的运行能力，于是浏览器框不住js了。

&emsp;&emsp;在commonJS中，一个文件就是一个模块，拥有单独的作用域， 普通方式定义的变量、函数、对象都属于该模块内，因为所有代码都运行在局部作用域内，因此避免了污染全局作用域的问题。模块可以多次加载但只会运行一次，此后模块被缓存起来可以被直接调用，模块的加载顺序由书写顺序决定，以同步的方式加载。

&emsp;&emsp;但commonJS旨在使js获得在服务端开发的能力，因此模块规范的订制都采用同步的方式，但在浏览器中，同步意味着用户必须等待程序的执行、加载等步骤，由此造成严重的用户体验和使用性能问题。那么浏览器中js程序的模块化规范怎么办呢？

## AMD和CMD

&emsp;&emsp;实际上，为了达到浏览器端模块化开发的目的，目前有两种常用的规范——AMD（Asynchronous Module Definition）和CMD（Common Module Definition ）。引用玉伯大佬在知乎上的解释。

> AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。
>
>  CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。 

&emsp;&emsp;AMD和CMD均实现了异步模块加载，但有其各自的特点。

### AMD

&emsp;&emsp;RequireJS应用的异步加载模块规范，需要注意的是，依赖必须在模块开头写明。AMD主要提供define和require两个方法来进行模块化编程 ，define用于定义模块，require用于引入模块。

#### AMD的define

&emsp;&emsp;define用于定义模块，通过return对象来暴露内部定义的属性和方法，实际上应用的就是采用 匿名函数模块化开发 的思路。关于基本js模块化开发的介绍可移步另一篇博客 —— [关于javascript模块化开发](http://blog.xcola.top/2018/05/23/%E5%85%B3%E4%BA%8Ejavascript%E6%A8%A1%E5%9D%97%E5%8C%96%E5%BC%80%E5%8F%91/)

```javascript
define(function(){
    function func1 (args){
        //....
        return result;
    } 
    function func2 (args1,args2){
        //...
        return result;
    }
    return {
        func1: func1,
        func2: func2
    }
})
```

&emsp;&emsp;当模块独立运行不需要依赖别的模块时，上面的代码是可行的。当模块之间存在依赖时，需要在define中以数组的方式指明依赖的模块。

```javascript
define(['./modules1','./modules2',..],function(m1,m2,...){
    function func1 (args){
        //可调用modules1、modules2中的方法
        return m1.func1(args);
    } 
    function func2 (args1,args2){
        //...
        return result;
    }
    return {
        func1: func1,
        func2: func2
    }
})
```

&emsp;&emsp;可以看出，在添加了依赖的模块后，define的回调函数中指定了几个形参，这几个形参即为各模块中返回的对象，因此可以通过调用形参的方法获得别的模块中定义的功能。

#### AMD的require

&emsp;&emsp;require用于引入模块，语法也与define相近，处理亦置于回调函数中。

```javascript
require(['./modules1','./modules2',..],function(m1,m2,..){
    //....
})
```

#### AMD的调用方式

&emsp;&emsp;RequireJS的调用采取标签的形式，标签中指定入口。

```html
<script data-main='./scripts/main' src='./scripts/require.js'></script>
```

&emsp;&emsp;data-main指定了模块的入口，这里main.js即为入口，依赖模块将通过入口开始异步加载进入执行。

### CMD

&emsp;&emsp;这是淘宝的攻城狮玉伯开发的seaJS执行的标准，名字据说是有“ 海纳百川，有容乃大 ”的寓意。CMD和AMD都有define和require方法，但CMD实际上是不需要在require和define中指定依赖的。

#### CMD的define

define的三个参数分别指明

* require -- 依赖的模块（以id为标识符）（推荐就近书写）

* exports -- 向外界提供的接口

* modules -- 一个对象，存储当前模块的属性和方法【id（标识符）,uri（绝对路径）,exports（导出接口）】

  

```javascript
//CMD
define(function(require,exports,modules){
    //依赖模块的使用
    var m1 = require('./modules1');
    m1.func1();
    //导出方法
    exports.func = function(){
        //...
    }
    // 或者。。
    modules.exports = {
        func: func
    }
})
```

#### CMD的调用方式

首先是毫无疑问的第一步。。引入seaJS。

```html
<script src="../sea-modules/seajs/seajs/2.2.0/sea.js"></script>
```

在引入seaJS后，进行基础的配置。

```javascript
// seajs 的简单配置
seajs.config({
    // 配置路径，Sea.js 在解析顶级标识时，会相对 base 路径来解析。
  base: "../sea-modules/",
    // 配置别名，有了alias后可以require('jquery')，否则只能require ID
  alias: {		
    "jquery": "jquery/jquery/1.10.1/jquery.js" 
  }
})

// 加载入口模块
seajs.use("../static/hello/src/main")
```

当然seaJS的配置还有很多。。有个博客写的很详细，[传送门](http://yslove.net/seajs/)。

### CMD和AMD的区别

&emsp;&emsp;至于AMD和CMD的差别，引用玉伯的描述：AMD先定义所有依赖，操作在加载完成后的回调函数中执行，而CMD依赖就近，用到什么依赖到时候再说。对于依赖的模块，AMD提前执行，CMD延后执行。（但执行AMD标准的RequireJS也已添加延迟执行的选项。）另外，AMD 的 API 默认是一个当多个用，CMD 的 API 严格区分，推崇职责单一。 

&emsp;&emsp;CMD的哲学是 as lazy as possible，emm深得我心。。