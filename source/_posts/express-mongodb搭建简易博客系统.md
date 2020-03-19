---
title: express+mongodb搭建简易博客系统
date: 2018-06-16 12:31:09
tags:
- 学习笔记
- NodeJS
categories: 
- 前端
---

## 准备事项

安装Nodejs，express，git，mongodb以及各种依赖库的过程略去不提。使用eslint规范代码。

`npm install -g eslint`

也可以 *--save-dev* 选择本地依赖安装 。懒人必备，直接init一个eslint规则。

`eslint --init`

这会提供几个可选选项，根据你的选择创建规范规则文件。放张图。

![20180616124006](\uploads\20180616124006.png)

express通过mongoose库配置mongodb连接。app.js文件添加

```javascript
var mongoose =require("mongoose");
mongoose.connect("mongodb://127.0.0.1:27017/express_blog");
```

package.json修改script项实现自动重启服务，此处需要node-dev包。

```javascript
"scripts": {
    "start": "node-dev ./bin/www"
},
```

新建git仓库。

`git init`

新建 .gitignore 文件忽略node_modules文件夹。之后add文件，add . 匹配所有文件。

` git add .`

commit提交初始的环境配置。

` git commit -m "init" `

github上create a new repository后关联到远程仓库

` git remote add origin git remote add origin https://github.com/flycatrix/express-blog.git `

将本地仓库的内容推到远程库上

` git push -u origin master `

本地和远程分支合并

` git pull origin master:master`

准备OK。

## 数据库操作

### mongoose（猫鼬）中间件

博客的功能很大程度上依赖数据库操作，使用mongoose库（猫鼬）简化mongodb操作。mongoose使用对象模型的方法对mongodb数据库进行操作，使用schema方法规范模型的变量个数，变量类型等。吐槽官方文档，一个技术文档写的那么萌。。

model文件夹存放要使用到的数据模型。简单的user模型

```javascript
var mongoose =require("mongoose");
var Schema= mongoose.Schema;
var  obj = {
	username:String,
	password:String
}
var model = mongoose.model("user" ,new Schema(obj));
module.exports = model;
```

建立模型需要引用mongoose模块，Schema构造函数将写好的数据模型obj包装成Schema对象，最终导出的model对象将继承Schema构造函数的原型中所定义的 访问和操作mongodb 的方法。

schema有十种数据类型，string是其中之一，MongoDB期望用户能按固定的规律存放数据，但实际上用户可以在MongoDB中的任一集合内存放任一类型的数据。mongoose采取事先说明好数据结构和类型的方式，规范用户写入MongoDB中的数据。同时，Schema也能在定义模型时定义MongoDB的索引，支持指定索引（index）或者唯一索引（unique）。对mongoose模型的使用将分散在各路由页面说明。

关于Schema的更多用法和解释可移步mongoose的文档，写文档的一定是个爱猫的程序员。

## session - cookie用户验证

### 关于session

用户的的登录状态和权限等级的确认均需要用到cookie机制，然而用户在浏览器端可以轻易更改、伪造cookie。就如xss攻击可以在用户不知情的情况下获取记录在用户浏览器中的无httponly属性的cookie信息，攻击者一旦成功伪造cookie登入，便可操作受害人的资料数据等。然而xss有绕过httponly的方法，且httponly对中间人攻击无效。一年以前曾经测试过中间人攻击，直接登入受害人QQ邮箱。。不过cookie的secure属性可以只在https通信中传递cookie，可以有效防范中间人，嗅探什么的。。emm废话不说了。

此外，cookie在每次请求的时候都会被附加在请求头中，大量的cookie必然会影响传输效率。那么，session应运而生。

session是存储在服务器中的数据，通过session_id运作。当收到请求时，取出保存在cookie中的session，在服务端查看其是否存在登录属性，借此判断用户的登录状态。

### express-session中间件

express-session中间件件实现会话。先npm install express-session --save-dev，然后app.js中添加

```javascript
var session = require("express-session");
app.use(session({
    name: "sessionid", // 在响应中cookie的name字段，默认是connect.sid
    secret:"xcola", //用secret签名保证是本服务器生成的session
    cookie: {maxAge: 1000*3600 }, //1小时
    resave: true,// 将session强制保存在store中
    saveUninitialized: true // 第一次访问即设置cookie
}));  
```

当用户首次访问页面时，就为用户分配一个cookie，其中保存对应的sessionid。

**登录逻辑：**若用户登录成功，那么为这个session新添一个属性，当下次请求来临时，取得用户session中新增的属性字段，若为空则是未登录用户，若不为空则是已登录用户。

**注销逻辑：**使用destroy方法摧毁当前会话，在摧毁后会重新生成一个新的没有登录状态的session会话。

## 登录与注册功能

在express添加新路由需要三步。

* app.js 中require 路由模块（存放在routes文件夹下）
* app.js 中use注册路由
* routes文件夹中添加新路由模块文件并设置路由规则

### register - 注册页

#### register.ejs模版部分

views文件夹新建register.ejs模版。页面部件偷懒扒取自bootstrap。

```ejs
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
    <link rel="stylesheet" href="/bootstrap/css/bootstrap.css">
    <script type="text/javascript" src="/bootstrap/js/jquery-3.3.1.min.js"></script>
    <script src="/bootstrap/js/bootstrap.js"></script>
  </head>
  <body>
    <%- include('./header.ejs',{title: "register",elsetitle:"login"})%>
    <div class="container">
    	<form method="post" action="/register/validate">
    	  <div class="form-group">
    	    <label for="exampleInputEmail1">Email address</label>
    	    <input type="text" class="form-control" placeholder="Username" name="username">
    	  </div>
    	  <div class="form-group">
    	    <label for="exampleInputPassword1">Password</label>
    	    <input type="password" class="form-control" id="exampleInputPassword1" placeholder="Password" name="password">
    	  </div>
    	  <div class="checkbox">
    	    <label>
    	      <input type="checkbox"> Check me out
    	    </label>
    	  </div>
    	  <button type="submit" class="btn btn-default">Submit</button>
    	  <%if(show){%>
    	  	<div class="alert alert-danger" role="alert">用户名已被注册</div>
    	  <%}%>
    	</form>
    </div>
  </body>
</html>

```

页面头部的导航栏写成独立的header.ejs文件，在使用到header的ejs中include一下，还可以在include的同时向header.ejs传入变量。想起来之前用php写公共部分再include，通过js改变页面内容的做法，简直不要太傻。

#### register.js路由模块部分

路由模块中设置路由规则，也承担根据用户输入连接和操作数据库的任务。

```javascript
var express = require('express');
var router = express.Router();
var userModel = require("../model/userModel");

router.get("/",function(req,res){
	res.render("register",{title:"Register",elsetitle:"Login",show:0});
}) 

router.post("/validate",function(req,res){

	userModel.find({
		username:req.body.username
	},function(error,docs){

		if(docs.length==0){
			userModel.create({
				username:req.body.username,
				password:req.body.password
			},function(error,info){

				if(!error){
					res.redirect("/");
				}
			})
		}else{
			res.render("register",{title:"Register",elsetitle:"Login",show:1});
		}
	})
})
 
module.exports = router;
```

路由文件中引入我们事先写好的userModel（mongoose模块），以便按userModel中规定的模型检索数据库。在register.js路由模块中，get方法负责渲染页面，post方法在二级路由中实现数据库查询以及写入的操作。通过**find方法**检索用户提交的用户名，在**回调函数**中处理查询结果，**error**用于抛出一个错误信息，**docs**为查询后得出的文档数组，若docs长度为0，表示这是一个未被注册的用户名，因此调用**create方法**写入一个新用户信息。若docs的长度不为0，则显示 “用户名已被注册” 的消息。

### login - 登录页

login的ejs模版和register.ejs大同小异，不再贴出赘述。

#### login.js路由模块部分

login部分的路由，需要判断用户名和密码是否能在数据库中成功匹配。判断成功后，还应该为用户设置一条新的session属性，以便根据新添加的属性判断用户的登录状态，在数据库中取得用户信息。

login.js路由模块

```javascript
var express = require('express');
var router = express.Router();
var userModel = require("../model/userModel");
router.get("/",function(req,res){
	res.render("login",{title:"Login",elsetitle:"Register",show:0});
}) 
router.post("/validate",function(req,res){
	userModel.find({
		email:req.body.email,
		password:req.body.password
	},function(error, docs){
		if(docs.length==0){
			res.render("login",{title:"Login",elsetitle:"Register",show:1});
		}else{
			req.session.userInfo = docs[0];
			res.redirect("/") 
		}
 	})
})
module.exports = router;
```

get方法渲染登录页面，post请求中接收用户输入，若用户登录成功，那么为这个session新添一个属性userInfo并跳转至首页，若登录失败则在页面上显示错误消息。

## 博客的增删改查

### 准备博客模型

在用户登录之后，可以撰写、发表博客。博客内容和信息存储在MongoDB中，在model文件夹中建立mongoose模型

```javascript
// article model
var mongoose =require("mongoose");
var Schema= mongoose.Schema;
var  obj = {
	author:String,
	title:String,
	content:String,
	createTime:Date,
    category: String
}
var model = mongoose.model("article" ,new Schema(obj));
module.exports = model;
```

author，title，content，createTime为博客的基本信息，category为博客添加分类信息。

### 首页显示博客列表

首页提供对未登录或非博主用户展示博客列表功能，对博主提供删改接口。此外，添加销毁路由的操作。

#### index.js路由模块部分

index路由判断用户登录状态及用户权限，读取保存在数据库中的博客列表传输给index.ejs模版，最终显示在用户界面上。

```javascript
var express = require('express');
var router = express.Router();
var articleModel =require("../model/articlemodel");

/* GET home page. */
router.get('/', function(req, res, next) {
	if(req.session.userInfo){
		articleModel.find({},function(error,docs){
			res.render('index', { title: 'Express' ,welcome:req.session.userInfo.username,
				list:docs,show:1
			});
		})
	}else{
		articleModel.find({},function(error,docs){
			res.render('index', { title: 'Express' ,welcome:null,
				list:docs,show:0
			});
		})
	}
});
//destroy session
router.get("/logout",function(req,res){
	req.session.destroy(function(error){
		if(!error){
			res.redirect('/login');
		}
	})
})
module.exports = router;
```

get方法中首先读取用户登录状态，若存在则显示欢迎标签，同时在页面上增加对当前用户博客内容的删改功能，若是未登录用户，则仅能查看当前博客列表。

#### index.ejs模版部分

```ejs
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel="stylesheet" href="/bootstrap/css/bootstrap.css">
    <script type="text/javascript" src="/bootstrap/js/jquery-3.3.1.min.js"></script>
    <script src="/bootstrap/js/bootstrap.js"></script>
  </head>
  <body>
    <%- include('./header.ejs',{title: "index",elsetitle:"article"})%>
    
    <div class="container">
		<%if(show){%>
    	<p>Welcome back! <%= welcome %></p>
    	 <%}%>
	    <table class="table table-striped table-bordered ">
        	<caption>Optional table caption.</caption>
        	<thead>
        		<tr>
        		  <th>作者</th>
        		  <th>标题</th>
        		  <th>创建时间</th>
                  <th>分类</th>
        		  <%if(show){%>
        		  <th>操作</th>
        		  <%}%>
        		</tr>
        	</thead>
        	<tbody>
        		<%for(var i=0;i<list.length;i++) { %>
        			<tr id="<%= list[i]._id %>">
        				<th scope="row"><%= list[i].author%></th>
        				<td><%= list[i].title%></td>
        				<td><%= list[i].createTime%></td>
                        <td><%= list[i].category%></td>
        				<%if(show){%>
        				<td>
        					<a class="btn btn-default" role="button" href="/update/<%= list[i]._id %>" id="update">更新</a>
        					<a class="btn btn-danger" role="button" href="/remove/<%= list[i]._id %>" id="delete">删除</a>
        				</td>
						 <%}%>
        			</tr>
        		<%}%>
        	</tbody>
	    </table>
        
    </div>
  </body>
</html>
```

在此指出，index.ejs模版中使用ejs语法中的for循环创建表格，同时将当次循环中的博客的id写入对应的tr的id属性以及删除按钮的href属性中去，这样当用户点击删除时才能告知服务器具体需要删除哪篇博客。后面查寻博客时则通过事件代理取tr的id值确定路由。

article模版展示编写博客的页面，新建博客和更新博客共用此模版。

#### article.ejs模版部分

```ejs
<!DOCTYPE html>
<html>
  <head>
    <title></title>
    <link rel="stylesheet" href="/bootstrap/css/bootstrap.css">
    <script type="text/javascript" src="/bootstrap/js/jquery-3.3.1.min.js"></script>
    <script type="text/javascript" src="/bootstrap/js/bootstrap.js"></script>

  </head>
  <body>
  	<%- include("header.ejs",{title:"write",elsetitle:"logout",show:0})%>
   	<div class="container">
   		<form action="/article" method="post">
   		 
   		  <div class="form-group">
   		    <label for="exampleInputEmail1">标题</label>
   		    <input type="text" class="form-control" id="exampleInputEmail1" name="title" placeholder="title" value="<%= art_title%>">
   		  </div>
        <div class="form-group">
          <label for="exampleInputEmail1">分类</label>
          <input type="text" class="form-control" id="exampleInputEmail1" name="category" placeholder="category" value="<%= art_category%>">
        </div>
   		  <div class="form-group">
   		    <label for="exampleInputPassword1">内容 </label>
   		   <textarea rows="4" class="form-control" name="content"><%= art_content%></textarea>
   		  </div>
   		  <button type="submit" class="btn btn-default">Submit</button>

   		  
   		</form>
   	</div>
  </body>
</html>

```

### 增 - 编写博客

在routes文件夹中新建article.js路由模块，在views文件夹下新建article.ejs模版文件，生成编写博客页面。

#### article.js路由模块部分

```javascript
var express = require('express');
var router = express.Router();
var articleModel =require("../model/articlemodel");
/* GET home page. */
router.get('/', function(req, res, next) {
	if(req.session.userInfo){
		res.render('article');
	}else{
		res.redirect("/login");
	}
  
});

router.post("/",function(req,res){
	articleModel.create({
		author: req.session.userInfo.username,
		title: req.body.title,
		content: req.body.content,
		createTime: new Date(),
		category: req.body.category
	},function(error,info){
		if(!error){
			res.redirect("/");
		}
	})
})
module.exports = router;

```

get方法中，先通过session的userInfo属性判断用户是否登入，若为已登录用户则加载article页面，若为未登录用户则跳转登录页。

post方法中，接收用户输入，将用户编写的博客内容保存到MongoDB中。

### 删 - 删除博客

使用动态路由的方式完成删除博客的操作。首页对已登录博主用户提供删除博客接口，当点击删除时，将被点击的博客的id当作路由地址通过get方式传给remove.js路由模块，在remove.js中通过deleteOne()方法安全删除博客。

#### remove.js路由模块部分

```javascript
var express = require('express');
var router = express.Router();
var articleModel =require("../model/articlemodel");
var userModel = require("../model/userModel");
/* GET home page. */
router.get("/:id",function(req,res){
	// console.log(req.session.userInfo);
	articleModel.deleteOne({
		_id: req.params.id
	},function(error,docs){
		if(!error){
			articleModel.find({},function(error,docs){
				res.render('index', { title: 'Express' ,welcome:req.session.userInfo.username,
							list:docs,show:1
				});
			});
		}
	})
})
module.exports = router;

```

在express中，获取不同方式提交的数据的方法分以下几种。

* req.params获取动态路由字段。
* req.query获取get查询字段。
* req.body获取post字段。

在删除路由模块后，需要再执行一次find()操作，否则首页的博客列表将为空。

### 改 - 更新博客

更新博客也使用动态路由的方法。和删除操作很类似。需要注意的是更新博客需要获取原有的博客填充进article.ejs模版中。

#### update.js路由模块部分

```javascript
var express = require('express');
var router = express.Router();
var articleModel =require("../model/articlemodel");
var userModel = require("../model/userModel");
/* GET home page. */

router.get("/:id",function(req,res){
	articleModel.find({
		_id: req.params.id
	},function(error,docs){
		
		if(!error){
			res.render('article',{art_title:docs[0].title,art_content:docs[0].content,art_category:docs[0].category});
		}
	})
})
module.exports = router;
```

更改之后仍使用编写博客时的方式提交博客，博客的createtime属性也将更新为最后一次提交更新的时间。

### 查 - 查看博客

在index.ejs中对展示博客列表的table标签添加事件代理。监听每个tr的点击事件跳转到check路由上。

index.ejs添加

```html
<script type="text/javascript">
        $('tbody').on('click',"tr",function(){
            location.href = '/check/'+$(this).attr('id');
        })
</script>
```

在for循环输出博客列表时，事先给每个tr的id属性都设置成了博客的id。

#### check.js路由模块部分

```javascript
var express = require('express');
var router = express.Router();
var articleModel =require("../model/articlemodel");
var userModel = require("../model/userModel");
/* GET home page. */
router.get("/:id",function(req,res){
	articleModel.find({
		_id: req.params.id
	},function(error,docs){
		if(!error){
			res.render('check',{title:"blog",show:0,art_title:docs[0].title,art_content:docs[0].content,art_category:docs[0].category});
		}
	})
})
module.exports = router;
```

这应该是最简单的一个模块吧。find一下然后传给check.ejs模版就好了。

## 总结

至此，已经实现了一个简单的可以登录注册增删改查博文的小博客，但还缺少分类、评论等功能。第一次记叙小项目的实现过程，行文不免凌乱缺乏条理，章节安排参考了@liuxing的github项目[node-blog](https://github.com/liuxing/node-blog)，项目代码已上传我的github [express-blog](https://github.com/flycatrix/express-blog)。此外，由于对代码的理解较浅，表述不当之处，还望不吝赐教。