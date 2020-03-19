---
title: 对ajax的理解
date: 2018-05-26 18:19:42
tags:
- 学习笔记
categories: 
- 前端
---

&emsp;&emsp;先来引用一段mdn对ajax的描述。

> (ajax)（异步JavaScript和XML）Asynchronous JavaScript + XML, 其本身不是一种新技术，而是一个在 2005年被Jesse James Garrett提出的新术语，用来描述一种使用现有技术集合的‘新’方法，包括: HTMLor XHTML, Cascading Style Sheets, JavaScript, The Document Object Model, XML, XSLT, 以及最重要的 XMLHttpRequest object。当使用结合了这些技术的AJAX模型以后， 网页程序能够快速地将渐步更新呈现在用户界面上，不需要重载（刷新）整个页面。这使得程序能够更快地回应用户的操作。

&emsp;&emsp;ajax相当于在浏览器和服务器之间加了一层代理，使得浏览器请求和服务器响应异步化，将一部分请求暂存，当需要从服务器读取新数据时再由ajax代为处理，最终做到让我们在不重载页面的情况下读写服务器上的数据 。

### ajax的使用

&emsp;&emsp;谈谈原生js对ajax使用，js提供了一个api来使用ajax——XMLHttpRequest。在使用ajax之前，需要构造一个XMLHttpRequest对象。

  ```javascript
  var xhr = new XMLHttpRequest;
  ```

&emsp;&emsp;既然是一个对象，那么就会从它的构造函数中继承到一些属性和方法，这些属性和方法控制和说明ajax请求的状态，以及异步请求的方式、等等。

#### 向服务器发送请求的方法

&emsp;&emsp;既然是异步加载，终究还是需要向服务器请求数据的，异步体现在请求的时机和内容由js动态控制。XMLHttpRequest对象提供了open()和send()方法。这两个方法在工作内容上是顺接的，open方法规定请求的内容、URL（统一资源定位符，表明请求的资源地址）、以及是否异步处理请求。而send方法完成将请求发送到服务器的操作。

##### open()

&emsp;&emsp;open函数初始化一个请求，准备好请求的内容后由send函数发送。一个请求需要很多参数才能被准确说明，这些说明应当通过参数的方式传给open方法。

###### method

&emsp;&emsp;说明请求使用的http方法，包括 "GET", "POST", "PUT", "DELETE" 等等，虽然我是在不同的书上看到过put和delete请求，然而我是没见过这两种请求。。

###### url

&emsp;&emsp;当前请求的资源地址。

###### async

&emsp;&emsp;是否异步。true那么异步，false那么同步。这里要注意的是，如果该请求是异步模式，send()会立刻返回。相反，如果请求是同步模式，则直到请求的响应完全接受以后，该方法才会返回。

###### user

&emsp;&emsp;用户名。这是个可选的参数。

###### password

&emsp;&emsp;密码。有了用户名当然就有密码。。也是个可选的参数。

&emsp;&emsp;user和password我还没使用过，但是回想之前在linux系统下登录内网用户认证的时候，用过curl命令发送post请求，后面是需要带上user和password的，这个两个参数应该同理。

##### send()

&emsp;&emsp;send函数相对就简单一些，参数只用于post请求，功能是将设定好的请求发送出去。send函数也可以传递post参数，但是在传参时往往会将参数序列化（如果参数是文档），或者在遇到空格等特殊字符时被截断，因此通常会先将send中发送的参数encode一下。

#### XMLHttpRequest的属性

&emsp;&emsp;在整个响应的过程中，js需要判断响应的状态才能做出处理。XMLHttpRequest的属性提供了一些说明请求状态以及处理的方法。

##### readystate

&emsp;&emsp;这是一个数值类型的属性，其值说明了请求的5中状态。

- 0 未开始         open()尚未调用

- 1 未发送         open()已经被调用，但send()未调用

- 2  获取响应头 send()已经被调用，响应状态（status）和响应头已获取

- 3 下载响应体  正在下载responseText

- 4 请求完成      整个响应已经结束

##### status 

&emsp;&emsp;这个属性说明了请求的响应状态码，即Http协议返回的消息响应码。摘取如下。这个属性是只读的。

- 1xx:信息响应类，表示接收到请求并且继续处理
- 2xx:处理成功响应类，表示动作被成功接收、理解和接受
- 3xx:[重定向](https://baike.baidu.com/item/%E9%87%8D%E5%AE%9A%E5%90%91)响应类，为了完成指定的动作，必须接受进一步处理
- 4xx:客户端错误，客户请求包含语法错误或者是不能正确执行
- 5xx:服务端错误，服务器不能正确执行一个正确的请求


##### onreadystatechange

&emsp;&emsp;这是一个函数对象，只看名字应该也能猜到它的作用——当readyState属性改变时会调用它。在js中也依赖于这个函数在每次响应状态改变时判断响应是否结束，当响应成功且结束时读取响应文本，进行下一步的处理和操作。

##### responseText 

&emsp;&emsp;当响应成功且结束时，这个属性即为服务器响应的文本。

#### 来一段代码看看

```javascript
function ajax_request() {
    var xhr = new XMLHttpRequest();
    xhr.open('get','./part.json',true);
    xhr.onreadystatechange = function () {
        if(xhr.readystate !== 4){
            return;
        }
        if(xhr.status >= 200 && xhr.status <300){
            var result = JSON.parse(xhr.responseText);
        }else{
            console.log('request error');
        }
    }
    xhr.send();
}
```

&emsp;&emsp;1，创建一个XMLHttpRequest对象。2，open()发送请求。3，检测状态改变时进入匿名函数。4，如果响应未完成则返回。5，响应成功则继续判断响应状态，响应成功则处理响应文本，否则抛出错误。6，将请求发送，send()。嗯。

&emsp;&emsp;open函数将请求设定好，onreadystatechange设置了回调函数等待调用，send函数将请求发送，当有响应消息时调用onreadystatechange设置的回调函数。因此send总是放在后面。

### ajax的优点

&emsp;&emsp;网页程序能够快速地将渐步更新呈现在用户界面上，不需要重载（刷新）整个页面。程序能够更快地回应用户的操作，缓冲和转移服务器压力。 。。。。。。

- 不需要插件支持
- 优化用户体验
- 提高web性能
- 减轻服务器和带宽负担

### ajax的缺点

&emsp;&emsp;ajax工作在浏览器和服务器之间，实际上也破坏了浏览器的后退和历史机制。在动态加载的情况下，浏览器无法区分静态页面和动态修改后的页面，于是用户无法通过back回到上一个页面或者取消上一次操作（插件解决）。另一个问题在于，动态页面更新使得用户难于将普通ajax页面的某个特定的状态保存到收藏夹中（url加锚点解决）。

&emsp;&emsp;此外，ajax对搜索引擎支持不好，爬虫难以爬取，搜索引擎也不容易搜索到动态加载的内容。今天搜索这个问题看到阮一峰大神搬运的Discourse创始人之一的Robin Ward的解决方法——[阮一峰的网络日志 ](http://www.ruanyifeng.com/blog/2013/07/how_to_make_search_engines_find_ajax_content.html)，这种方法通过window.history使每个锚点都变成可爬取的url，膜拜一下。