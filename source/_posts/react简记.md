---
title: react简记
date: 2018-07-20 08:34:52
tags:
- 学习笔记
- React
categories: 
- 前端
---



## 目录结构和流程

create-react-app生成项目。目录树。

```bash
│  .gitignore
│  config-overrides.js
│  package-lock.json
│  package.json
│  README.md
│  
├─public
│      favicon.ico
│      index.html
│      manifest.json
│      
├─screenshot
│      bookPage.jpg
│      eventPage.jpg
│      eventPage01.jpg
│      homePage.jpg
│      repoPage.jpg
│      
└─src
    │  App.js
    │  index.js
    │  registerServiceWorker.js
    │  
    ├─assets
    │  └─images
    │          allbooks0.svg
    │          allbooks1.svg
    │          buy0.svg
    │          buy1.svg
    │          comment.png
    │          data.png
    │          like.png
    │          link.svg
    │          location.png
    │          logo.svg
    │          me.png
    │          me0.svg
    │          me1.svg
    │          outline.png
    │          star.png
    │          three.png
    │          yy.png
    │          
    ├─Components
    │  ├─Books
    │  │  │  index.css
    │  │  │  index.js
    │  │  │  index.scss
    │  │  │  
    │  │  ├─Allbooks
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  ├─Bought
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  └─Me
    │  │          index.css
    │  │          index.js
    │  │          index.scss
    │  │          
    │  ├─Common
    │  │  └─Navbar
    │  │          index.css
    │  │          index.js
    │  │          index.scss
    │  │          
    │  ├─Event
    │  │  │  index.css
    │  │  │  index.js
    │  │  │  index.scss
    │  │  │  
    │  │  ├─all
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  ├─beijing
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  ├─guangzhou
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  ├─hangzhou
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  ├─shanghai
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  └─shenzhen
    │  │          index.css
    │  │          index.js
    │  │          index.scss
    │  │          
    │  ├─Home
    │  │      index.css
    │  │      index.js
    │  │      index.scss
    │  │      
    │  ├─Pins
    │  │  │  index.css
    │  │  │  index.js
    │  │  │  index.scss
    │  │  │  
    │  │  ├─Recommended
    │  │  │      index.css
    │  │  │      index.js
    │  │  │      index.scss
    │  │  │      
    │  │  └─Subscribed
    │  │          index.css
    │  │          index.js
    │  │          index.scss
    │  │          
    │  └─Repos
    │          index.css
    │          index.js
    │          index.scss
    │          
    ├─Redux
    │  │  index.js
    │  │  
    │  └─Reducer
    │          bookListReducer.js
    │          changeTitleReducer.js
    │          getEventBannerReducer.js
    │          getEventDataReducer.js
    │          getRecommendListReducer.js
    │          getReposListReducer.js
    │          recommendListReducer.js
    │          
    └─Router
            index.js
            

```

public中存放的index.html是项目页面，其中的`<div id="root"></div>`标签包裹项目组件。src目录下index.js为入口文件，通过render函数将路由组件挂载到index.html中的`<div id="root"></div>`上，在路由组件中，通过匹配路由规则，将不同的组件显示在App根组件中`this.props.children`的位置。

## 组件

react提出了组件可以是函数的思想，构造组件有两种方式——类或函数。根据组件功能的不同，亦可划分为容器组件和展示组件。容器组件负责数据获取和状态更新等功能，而展示组件描述页面的结构和样式。

### 展示组件

通常使用函数的方法定义，除非它需要状态或者有生命周期函数性能优化等需求。

```jsx
function Welcome (props) {
    return <h1>welcome! {props.name}</h1>;
}
```

### 容器组件

通常使用类的方式定义，容器组件一般不写什么样式和页面结构，作用是向其他组件提供数据和功能，因为充当数据源的角色，所以容器组件一般都有状态。redux的操作也在容器组件中进行，并把方法和数据通过Provider提供给展示组件使用。

```javascript
class Datalist extends React.Component {
    constructor(){
        super();
        this.state = {
            list:[]
        }
    }
    render(){
        return <div></div>
    }
    componentDidMount(){
        this.props.getList()
    }
}
```

通过类定义的组件有更完善的功能，包括状态、render、生命周期函数、用户自定义方法、以及redux扩展的方法等。

用展示组件和容器组件相分离的方法编写组件，有利于理解UI和应用之间的关系。通过给展示组件传递不同属性的方法，提高了展示组件的可复用性。

## 路由

react的路由是提供导航和跳转的组件。得益于jsx语法，书写路由也像在写dom结构。

```javascript
const router = (
	<Provider store = {Store}>
		<Router>
			<App>
				<Switch>
					<Route path="/home" component={Home} />
					<Route path="/repos" component={Repos} />
					<Route path="/books" render={()=>
						<Books>
							<Switch>
								<Route path="/books/allbooks" component={Allbooks} />
								<Route path="/books/bought" component={Bought} />
								<Route path="/books/me" component={Me} />
								<Redirect from="/books" to="/books/allbooks"/>
							</Switch>
						</Books>
					} />
					<Route path="/event" render={()=>
						<Event>
							<Switch>
								<Route path="/event/all" component={all} />
								<Route path="/event/beijing" component={beijing} />
								<Route path="/event/guangzhou" component={guangzhou} />
								<Route path="/event/hangzhou" component={hangzhou} />
								<Route path="/event/shenzhen" component={shenzhen} />
								<Route path="/event/shanghai" component={shanghai} />
								<Redirect from="/event" to="/event/all"/>
							</Switch>
						</Event>
					} />
					<Route path="/pins" render={()=>
						<Pins>
							<Switch>
								<Route path="/pins/recommended" component={Recommended} />
								<Route path="/pins/subscribed" component={Subscribed} />
								<Redirect from="/pins" to="/pins/recommended"/>
							</Switch>
						</Pins>
					} />
					<Redirect from="*" to="/home"/>
					
				</Switch>
			</App>
		</Router>
	</Provider>
)
export default router
```

Provider组件包裹在路由最外层，为所有组件提供上下文环境，redux发出的状态和回调函数通过这个上下文环境传递和通信。

App组件作为根组件初始化，内部的组件通过`this.props.children` 路由容器显示在App组件中。

Switch开关保证同一location下只有第一个被匹配到的路由会被渲染。

Route组件的path属性匹配路径，component或render属性都可以指明此路由匹配的组件，但render可以被用来编写二级路由。exact属性接受一个布尔值，精确匹配会防止包容性路由匹配到路径相似的路由规则。

redirect重定向。

## 生命周期&State更新问题

react也提供生命周期函数，大致可以分为三个阶段——初始化阶段、运行阶段、销毁阶段。

### 挂载阶段

getDefaultProps()

getInitialState()

componentWillMount()

render()

*在此之前DOM节点都未被渲染，也无法获取*

componentDidMount()

*DOM节点已被渲染，异步操作可以在这里执行*

### 更新阶段

componentWillReciveProps()

*在组件接收到一个新的 prop 时被调用*

shouldComponentUpdate()

*返回一个布尔值。在组件接收到新的props或者state时被调用。合理使用有性能优化的效果*

componentWillUpdate()

*在组件接收到新的props或者state但还没有render时被调用。*

render()

componentDidUpdate()

### 销毁阶段

componentWillUnmount()

*组件被移除时调用*

### setState方法

直接修改state不会触发render更新组件，且直接修改的state会被之后setState方法提交的state状态所覆盖，造成难以预估的错误。因此正确的姿势是使用setState方法。同时也由于setState方法会触发render函数，因此在render函数中setState会陷入死循环直至溢出。

关于setState异步执行的问题，受教于知乎@[Lucas HC](https://www.zhihu.com/people/lucas-hc) 的[回答](https://www.zhihu.com/question/66749082/answer/246217812)。setState在react合成事件中触发是异步更新的，也就是说，setState后state并不会立即更新。setState是一个复杂的过程，不仅要更新state，也同时会根据所谓diff算法决定是否更新，如何更新，多个setState调用时还有可能需要被合并，因此被设计为延时异步更新state。

但如果通过js原生方法addEventListener添加一个事件，在原生事件中通过setState方法更新state，会发现state被同步更新了。也就是说，由 React 控制的事件处理过程 setState 不会同步更新 this.state，React 控制之外的情况， setState 会同步更新 this.state。

当然，大部分情况下在react项目中都是大量使用react合成后的组件与事件，因此setState基本上是异步执行的。

## react-redux

react也没有自己的状态管理机制，单向数据流的通信方式使得组件间通信复杂麻烦。redux提供的状态管理可以追踪状态变化的发生时间和前因后果，其带来的便利性不仅体现在开发上，也体现在调试上。

### 三个原则

redux中的几个概念——state，action，reducer。state是redux要管理的状态；action是一个简单对象，描述了一个行为；reducer接受一个action，按照action的描述返回一个新的状态。在这个过程中，redux遵循三个原则：

* 单一数据源——整个应用的state同一存在于一个唯一的store中。

* state只读——state不能直接修改，必须通过提交一个action的方式变更。

* reducer是纯函数——reducer接受action和state，返回新的state。

*纯函数的概念：*

> - 给定相同的输入(传入值)，一定会返回相同输出值结果(返回值)
> - 不会产生副作用
> - 不依赖任何外部的状态

### 使用

#### 在路由中

react-redux通过props读取状态和调用函数，因此需要在路由的最外层包裹一个Provider容器提供store。若使用常见的父子通信传递props，势必要将props挂载到传递路径中的每一个组件上，因此采用provider组件提供context上下文环境，有状态管理需求的组件通过上下文挂载props属性。

#### 在组件中

react-redux提供connect方法，将UI部分和逻辑部分连接在一起。connect是一个高阶函数，接受两个参数并返回另一个以当前组件为参数的函数。connect的两个参数分别提供从state到组件props的映射（mapStatetoProps），和从组件到dispatch方法的映射（mapDispatchtoProps）。

```javascript
export default connect(
	(state)=>{
		return {
			List:state.getListReducer
		}
	},
	{
		getList(){
			return (dispatch)=>{
				axios.get('url.com').then(res=>{
					dispatch({
						type:"getList",
						payload:res.data.d
					});
				})
			}
		}
		
	}
)(Datalist);
```

##### mapStateToProps

组件通过这个参数描述自己需要从某个reducer中接受一个状态，这个状态将通过props的方式挂载在组件上。

##### mapDispatchtoProps

组件通过这个参数定义几个dispatch函数，用以向reducer返回一个状态，这些dispatch函数将以回调函数的形式挂载在组件上。

#### redux模块中

通过combine方法合并不同的reducer函数。

```javascript
import {createStore} from "redux"
import {combineReducers,applyMiddleware,compose} from "redux"
import thunk from "redux-thunk"
import promiseMiddleware from "redux-promise"
import getEventDataReducer from "./Reducer/getEventDataReducer"
import getRecommendListReducer from "./Reducer/getRecommendListReducer"
const reducer = combineReducers({
    getEventDataReducer,
    getRecommendListReducer
})

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
const store = createStore(reducer, composeEnhancers(
    applyMiddleware(thunk,promiseMiddleware)
));

export default store
```

createStore方法创建一个store容器，接受reducer和composeEnhancers为两个参数，composeEnhancers为解决异步action添加中间件，同时为redux浏览器调试插件提供支持。

#### reducer函数

reducer接受dispatch方法传来的action，根据action的type属性判断处理方法，返回一个新的状态。

```javascript
// getEventDataReducer.js
const getEventData = (prevstate=[],data={})=>{
    let {type,payload} = data;
    switch(type){
        case "getEventData":
            return [...prevstate,...payload];
        default:
            return prevstate;
    }
}
export default getEventData
```



~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~一条分割线~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

暂时写到这里，日后抽空填坑。。
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~