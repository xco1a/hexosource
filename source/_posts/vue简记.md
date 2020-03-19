---
title: vue简记
date: 2018-07-05 17:13:16
tags:
- 学习笔记
- Vue
categories: 
- 前端
---

## vue-cli目录结构

npm安装vue-cli，init一个vue项目。tree生成树列表保留部分项目文件：

```bash
│  .babelrc
│  .editorconfig
│  .gitignore
│  .postcssrc.js
│  index.html
│  package.json
│  README.md
│  
├─build
│      build.js
│      check-versions.js
│      utils.js
│      vue-loader.conf.js
│      webpack.base.conf.js
│      webpack.dev.conf.js
│      webpack.prod.conf.js
│      
├─config
│      dev.env.js
│      index.js
│      prod.env.js
│      
├─src
│  │  App.vue
│  │  main.js
│  │  
│  ├─assets
│  │  └─images
│  │          swipe1.png
│  │          swipe2.png
│  │          swipe3.png
│  │          
│  ├─components
│  │  │  event.vue
│  │  │  explore.vue
│  │  │  invest.vue
│  │  │  questions.vue
│  │  │  
│  │  └─common
│  │          banner.vue
│  │          navbar.vue
│  │          
│  ├─router
│  │      index.js
│  │      
│  └─store
│          index.js
│          
└─static
        .gitkeep
        

```

### bulid文件夹

存放编译配置文件，大部分我都看不懂，webpack通过各种编译模块将项目文件编译成浏览器可读的html、css、js等。此外，vue-cli不支持sass语法编译，因此扩展安装的sass模块也需要在webpack.base.conf.js中引入。

### config文件夹

存放项目的配置文件，其中index.js说明了一系列项目本地运行时的参数，例如端口，静态虚拟文件夹，域名，反向代理等等。

### src文件夹

这是项目中主要操作的文件夹，其中存放着资源文件（assets）、组件（components）、路由（router）、vuex状态（store）。详细分析这个文件夹。assets略去不写。

#### components组件

这个文件夹存放vue文件，亦即单文件组件。单文件组件支持语法高亮，css，独立作用域，通过构建工具（webpack等）构建亦可以在项目中使用sass语法。common文件夹存放公用组件，如页面的导航、banner、等等。

项目把需求页面拆分组件，通过路由将合适的组件放进页面中，这样就形成了一个单页面应用 -- 更新页面但页面不跳转。

#### router路由

通过在文件夹内放一个index.js文件的方法省去了import中写文件名的步骤。项目在初始化时使用了vue-router，在路由模块中，先引入vue-router模块，使用vue.use注册该模块，随后导出一个router对象，其routes属性是一个数组，其中的每一个对象元素对应着一个路径和该路径对应的组件。若不指定mode属性，则默认为hash路由。

```javascript
import Vue from 'vue'
import Router from 'vue-router'
Vue.use(Router)
export default new Router({
  mode:"history",
  routes: [
    {
      path: '/explore',
      name: 'explore',        
      component: explore
    },
    {
      path: '/bonus/:id',  // 动态路由
      name: 'bonus',
      component: bonus,
      children:[{			// 二级路由
        path:'products',
        component:productsI
      },
      {
        path:'itemInfo',
        component:itemInfo
      }]
    }
  ]
})

```

**动态路由**：通过 ‘ : ’标识一个动态路由，随后在组件中通过全局变量 $route.params对象取得动态路由的参数。

**二级路由** ：通过children属性定义一个数组，其中存放二级路由。二级路由通过在当前一级路由组件中的路由容器`<router-view></router-view>`显示在当前路由页面上，实现部分页面更新。

**编程式路由**：$route变量可以全局访问到，且其值为一个数组，因此可以通过向这个数组中push一个新的路由实现跳转。

#### store仓库（vuex）

vuex是需要额外安装的，它为vue提供了一种状态管理模式 -- 集中管理应用中共用的状态，解决了vue组件间由作用域封闭而造成的通信困难的问题。此外在开发中，vuex也提供诸如快照等更强大的调试功能。

vuex使用单一状态树，因此一个应用仅包含一个store实例，在一次数据请求并将其保存在store中后，其他组件即可调用使用，若有组件再次发起请求且向store提交，则会将两次请求得到的结果合并。

vuex有分为四层的结构。数据在四个层中单向流动。祭出流程图一张。。![2018-7-6](/uploads/2018-7-6.png)

vue components中通过dispatch触发action中的异步方法，action将异步方法的结果commit提交给mutation，mutation的变化会触发状态的改变，由此触发vue components视图的更新。

```javascript
// store/index.js
import Vue from "vue";
import Vuex from "vuex";
import axios from "axios";
Vue.use(Vuex);

const store = new Vuex.Store({
	state:{
		dcplan:[],
		collected:[],
		interesting:[],
		finished:[],
		rest:[]
	},
	actions:{
		getdcplan(store){
			axios.get('/api/longterm/projects').then(res=>{
				store.commit('getdcplan',res.data.longterm);
			})
		}
	},
	mutations:{
		getdcplan(state,payload){
			state.dcplan = payload;
			for(let i in payload){
				switch(payload[i].status){
					case 'collected':
						state.collected.push(payload[i]);
						break;
					case 'interesting':
						state.interesting.push(payload[i]);
						break;
					case 'finished':
						state.finished.push(payload[i]);
						break;
					default:
						state.rest.push(payload[i]);
				}
			}
		}
	}
})

export default store;
```

在store模块中，创建了全局的vuex.Store实例，在这个实例中放置vuex流程图中的三个节点 -- state、actions、mutations。

* state中记录保存在Store状态树中的变量，被state记录后，应用中的各组件才能访问。
* mutations是调试时的主要参考。要改变state中的状态，只能通过向mutation提交，这样保证了每一次数据的更改都能被mutation记录，方便调试追踪。同时为了满足调试时捕捉状态更改次序和时机的需要，mutation必须是同步的，而action就没有此限制。
* actions中声明请求数据，回调函数等异步方法，可以看到actions中的方法在请求回数据后，会commit到mutations中去。action不能直接更改状态。

#### App.vue

App.vue是主组件，它为其他组件提供路由容器，是其他组件显示的基础。

```javascript
<template>
  <router-view></router-view>
</template>

<script>
import explore from './components/explore'
import dcplan from './components/dcplan'
export default {
  name: 'App',
  components:{
  	explore,
    dcplan
  }
}
</script>

<style lang="scss" scoped>
	
</style>

```



#### main.js

main.js是项目模块的入口，Vue实例在这里创建，同时也在这里引入注册项目需要的组件（router、vuex、第三方UI库等等）。

```javascript
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store'

import Mint from 'mint-ui'
import 'mint-ui/lib/style.css'

Vue.use(Mint);

import { Swipe, SwipeItem } from 'mint-ui';

Vue.use(Swipe);
Vue.use(SwipeItem)

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  components: { App },
  template: '<App/>'
})
```

### static

放置静态文件的文件夹，iconfont字体什么的塞进去。git会忽略空文件夹，为了防止它被忽略，给它里面放了一个.gitkeep文件。。

### index.html

项目页面。里面放了一个div。

`<div id="app"></div>`

main.js会将实例化后的Vue对象挂载在这个div上，main.js引入的子组件App.vue又将其他子组件通过路由的方法引入App.vue中的路由容器中，最终把路由到的组件显示在页面上。

