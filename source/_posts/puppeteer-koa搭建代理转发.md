---
title: puppeteer+koa搭建代理转发
date: 2019-12-01 17:20:48
tags: 
- 学习笔记
- NodeJS
categories: 
- 前端
---
<!-- more -->
## 背景

继上次我把umi嵌入vue以来，线上开发转变成了线下开发，一瞬间多了ts，eslint，antd，云构建，cdn发布，一堆vscode插件，以及aone的代码质量、评审工具可以用。说实话还是挺爽的。然而好事总是多磨，由于鲁班平台自带一层接口安全认证和转发机制，当我们在线下开发的时候，没法直接连接口调试，mock等等方案总是没有直连接口调试开发爽。大致情况如下。。

![image.png](http://blog.xcola.top/uploads/puppeteer&koa1.png)



如果跨过鲁班平台的转发层，直接连后端提供的ip:port，其实也是能通的，但是由于鲁班还改变了参数的格式，导致了这么干要在本地写一套调用接口代码，等搬上线再调试一遍。写两遍代码还要反复发布调试真的超级麻烦。。

所以我还是需要经过鲁班转发我的请求，然而在线下开发的时候，localhost是没有cookie的，所以当我在webpack的proxy里配完转发，本地项目发出的请求都是没有cookie的。鲁班就会返回给我一个302，要求我先登陆。



所以我需要让本地的环境获得平台的登陆态，这样才能通过鲁班访问接口。



我试过脑残的把我们日常环境的cookie抠出来，在本地开发页面粘贴进去。然后在nginx里配一个可以携带cookie和重写host的转发。这么做确实可以在刚配完的一段时间里用几次，然鹅调几次以后cookie就过期了，还是302，所以基本是不能用。后来经过两位大佬的指点，他们向我介绍了selenium（一个安全专家介绍的python工具）和puppeteer（一个前端专家介绍的node工具）。考虑到我惨不忍睹的python水平，我还是决定用node尝试一下。



## 工具介绍

### koa

Koa 是一个新的 web 框架，由 Express 幕后的原班人马打造， 致力于成为 web 应用和 API 开发领域中的一个更小、更富有表现力、更健壮的基石。 通过利用 async 函数，Koa 帮你丢弃回调函数，并有力地增强错误处理。 Koa 并没有捆绑任何中间件， 而是提供了一套优雅的方法，帮助您快速而愉快地编写服务端应用程序。

当然，这段话是我抄的。<https://koa.bootcss.com/>



### puppeteer

Puppeteer是一个Node库，它提供了高级API来通过[DevTools协议](https://chromedevtools.github.io/devtools-protocol/)控制Chrome或Chromium 。

当然，这句话也是我抄的。<https://github.com/puppeteer/puppeteer>。简单的说就是Puppeteer提供了一个可供node环境使用的一个无界面、代码控制的谷歌浏览器。可以通过调用api来实现一个用户可能有的所有操作。说实话我用完以后觉得这个拿来当爬虫用真的杠杠的。。



## 方案

基本是围绕着怎么生成cookie，怎么请求接口进行。

我决定让puppeteer去登陆平台，请求接口，拿到响应以后再用koa响应本地的前端请求。大概就是下面这条线。

![image.png](http://blog.xcola.top/uploads/puppeteer&koa3.png)



## 代码实现

代码实现思路大致分两部分，第一部分是让koa去识别和响应前端请求，第二部分是puppeteer收到koa拆解出协议和参数后，收发请求。

### koa部分

用koa生成一个http server，开始监听某个端口的请求。这里使用了koa-log4帮助记录日志。



```javascript
const Koa = require("koa");
const log4js = require("koa-log4");
const logger = log4js.getLogger("app");
const { PORT } = require("./config");
require("../log");
const bodyParser = require("koa-bodyparser");

const { instance: lubanProxy } = require("./lubanProxy");

async function server() {
  // init app & proxy
  const app = new Koa();
  app.use(bodyParser());
  app.use(log4js.koaLogger(log4js.getLogger("http"), { level: "auto" }));
  await lubanProxy.loginLuban();

  // add logger to a request
  app.use(async (ctx, next) => {
    const start = new Date();
    await next();
    const ms = new Date() - start;
    logger.info(`${start} ${ctx.method} ${ctx.url} - ${ms}ms`);
  });

  // send request to lb-test
  app.use(async ctx => {
    if (ctx.method === "GET") {
      let res = await lubanProxy.get(ctx.url);
      ctx.body = res;
    } else if (ctx.method === "POST") {
      let res = await lubanProxy.post(ctx.url, ctx.request.body);
      ctx.body = res;
    }
  });

  app.on("error", (err, ctx) => {
    logger.error("server error", err, ctx);
  });

  app.listen(PORT);
}
server();
```



### puppeteer部分

这块的文件名是lubanProxy.js。生成一个浏览器，登陆鲁班平台，向koa提供get和post方法。



#### 登陆部分

模拟真实用户的登陆操作即可。



```javascript
  loginLuban = async () => {
    this.browser = await puppeteer.launch();
    const page = await this.browser.newPage();

    await page
      .goto(
        "https://login.abc.com/login.html“
      )
      .catch(async e => {
        await page.waitFor(2000);
        logger.error(`load login page timeout，retrying...`);
      });
    await page
      .evaluate(() => {
        document.querySelector("input[name='account']").focus();
      })
      .catch(async _ => {
        logger.error(`Cannot get login form`);
      });
    await page.keyboard.type(ACCOUNT);

    await page
      .evaluate(() => {
        document.querySelector("input[name='password']").focus();
      })
      .catch(async _ => {
        logger.error(`Cannot get login form`);
      });
    await page.keyboard.type(PASSWORD);

    await Promise.all([page.waitForNavigation(), page.click(".submit")])
      .then(async () => {
        await page.goto("http://luban.abc.net/manage.htm");
        console.log("proxy ready...");
      })
      .catch(async _ => {
        logger.error(
          `Navigation timeout of 30000 ms exceeded,restarting browser...`
        );
        await this.restartBrowser();
      });
  };
```



#### get请求

puppeteer只能用来模拟用户操作，无法自如的发送ajax，所以get请求的思路是用page.goto来模拟。

```javascript
  get = async url => {
    const page = await this.browser.newPage();
    let res = "";
    await page
      .goto("http://luban.abc.net" + url)
      .then(async () => {
        res = await page.content();
        res = res.slice(res.indexOf("{"), res.lastIndexOf("}") + 1);
      })
      .catch(_ => {
        logger.error(`get request got net::ERR_CONNECTION_RESET error`);
        res = {
          code: -1,
          message: "Got a net error, try again?"
        };
      });
    await page.close();
    return res;
  };
```



#### post请求

post请求比get复杂一些，因为get可以通过页面跳转，而post无法模拟。目前的方式有点hack意味：先通过发起一次page.goto模拟一个get请求，然后通过拦截器拦下来，修改这次请求的参数和协议，再把请求发出去。



```javascript
  post = async (url, param) => {
    const page = await this.browser.newPage();
    await page.setRequestInterception(true);
    let res = "";

    page.on("request", async request => {
      let overrides = {
        method: "POST",
        postData: param
      };
      await request.continue(overrides).catch(e => {
        logger.error("Request is already handled!");
      });
    });
    await page
      .goto("http://luban.abc.net" + url)
      .then(async () => {
        res = await page.content();
        res = res.slice(res.indexOf("{"), res.lastIndexOf("}") + 1);
      })
      .catch(e => {
        logger.error(`post request got net::ERR_CONNECTION_RESET error`);
      });
    await page.close();
    return res;
  };
```



#### 全部代码

附上全部代码。。

```javascript
const { ACCOUNT, PASSWORD } = require("./config");
const puppeteer = require("puppeteer");
const log4js = require("koa-log4");
const logger = log4js.getLogger("app");
require("../log");

class lubanProxy {
  loginLuban = async () => {
    this.browser = await puppeteer.launch();
    const page = await this.browser.newPage();

    await page
      .goto(
        "https://login.abc.com/login.html“
      )
      .catch(async e => {
        await page.waitFor(2000);
        logger.error(`load login page timeout，retrying...`);
      });
    await page
      .evaluate(() => {
        document.querySelector("input[name='account']").focus();
      })
      .catch(async _ => {
        logger.error(`Cannot get login form`);
      });
    await page.keyboard.type(ACCOUNT);

    await page
      .evaluate(() => {
        document.querySelector("input[name='password']").focus();
      })
      .catch(async _ => {
        logger.error(`Cannot get login form`);
      });
    await page.keyboard.type(PASSWORD);

    await Promise.all([page.waitForNavigation(), page.click(".submit")])
      .then(async () => {
        await page.goto("http://luban.abc.net/manage.htm");
        console.log("proxy ready...");
      })
      .catch(async _ => {
        logger.error(
          `Navigation timeout of 30000 ms exceeded,restarting browser...`
        );
        await this.restartBrowser();
      });
  };

  get = async url => {
    const page = await this.browser.newPage();
    let res = "";
    await page
      .goto("http://luban.abc.net" + url)
      .then(async () => {
        res = await page.content();
        res = res.slice(res.indexOf("{"), res.lastIndexOf("}") + 1);
      })
      .catch(_ => {
        logger.error(`get request got net::ERR_CONNECTION_RESET error`);
        res = {
          code: -1,
          message: "Get net error, try again?"
        };
      });

    if (res.indexOf("404 not found") > -1) {
      return {
        code: 404,
        message: "404 not found"
      };
    }
    await page.close();
    return res;
  };
  post = async (url, param) => {
    const page = await this.browser.newPage();
    await page.setRequestInterception(true);
    let res = "";

    page.on("request", async request => {
      let overrides = {
        method: "POST",
        postData: param
      };
      await request.continue(overrides).catch(e => {
        logger.error("Request is already handled!");
      });
    });
    await page
      .goto("http://luban.abc.net" + url)
      .then(async () => {
        res = await page.content();
        res = res.slice(res.indexOf("{"), res.lastIndexOf("}") + 1);
      })
      .catch(e => {
        logger.error(`post request got net::ERR_CONNECTION_RESET error`);
      });
    await page.close();
    return res;
  };
  restartBrowser = async () => {
    const { browser } = this;
    await browser.close();
    await this.loginLuban();
  };
}
const instance = new lubanProxy();
module.exports = { instance };
```



项目全部文件在此。。

![image.png](http://blog.xcola.top/uploads/puppeteer&koa4.png)