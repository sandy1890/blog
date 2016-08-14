---
title: React开发技术栈
date: 2016-05-19 16:37:56
tags: react
---
## 概述
React本身仅只是一个视图层的解决方案，真实项目中还需要配合其它库来进行开发。这里介绍的是一个可进行完整项目开发的技术栈。将介绍以下这些库及它们之间如何配合来完成一个完整的项目开发。

* [react](https://facebook.github.io/react/) 处理视图
* [react-router](https://github.com/reactjs/react-router) 前端路由
* [redux](http://redux.js.org/) 应用状态管理，即数据获取及处理
* [react-redux](https://github.com/reactjs/react-redux) react官方redux绑定
* [redux-thunk](https://github.com/gaearon/redux-thunk)　扩展redux,方便处理异步请求

## react
react的核心是组件，了解组件的核心是搞清楚组件生命周期。组件之后，需要了解的是react组件之间信息流的传递，主要是通过`props`(包含回调函数，组件渲染所需要的数据等)

组件定义，组件通过 React.createClass()创建，包含以下一些定义
* render()
* getInitialState()
* getDefaultProps()
* propTypes
* mixins
* statics

react组件生命周期函数：
* componentWillMount()
* componentDidMount()
* componentWillReceiveProps()
* shouldComponentUpdate()
* componentWillUpdate()
* componentDidUpdate()
* componentWillUnmount()


包含所有定义和生命周期函数的组件定义看起来像下面这样

```javascript

var Home = React.createClass({
    mixins:[],
    statics: {},
    propTypes:{},
    getInitialState: function(){},
    getDefaultProps: function(){},
    render: function(){},
    componentWillMount: function(){},
    componentDidMount: function(){},
    componentWillReceiveProps: function(){},
    shouldComponentUpdate: function(){},
    componentWillUpdate: function(){},
    componentDidUpdate: function(){},
    componentWillUnmount: function(){}
});

```

`ES6`的写法略有不同，看起来像这样

```javascript
class Home extends Reac.Component{
    constructor(props){
        super(props);
        this.state = {}; //取代getDefaultState()
    }
    render(){
    }
}
```

## react-router
没有太多学习成本，主要功能就是提供路由和组件的绑定。

```javascript
ReactDOM.render((
    <Router history={history}>
            <Route path="/" component={App}>
                <IndexRoute component={Home}/>
                <Route path="about" component={About}/>
            </Route>
            <Route path="/other" component={Other}>
    </Router>
), document.getElementById('react-content'))
```

解释，上面的代码定义了一个路由规则
* 访问　`/`　路径的时候，渲染Home组件
* 访问　`/about` 路径的时候，渲染About组件
* 访问 `/other`　路径的时候，渲染Other组件

需要注意到`Home`和`About`被包含在App组件中，这样定义实现的效果是，`Home`和`About`组件的渲染会包裹在App组件中，实际上相当于我们平时开发流程中使用的`layout`视图层，这里的`App`组件就相当于`Home`和`About`的`layout`。来看一下App组件的render()方法

```javascript
class App extends React.Component {
  render() {
    return (
      <div>
          <Header/>
          {this.props.children}
          <Footer/>
      </div>
    )
  }
}
```
上面的代码，在App组件中，我们通过 `this.props.children` 来获取当前路由对应的子组件（本例中的Home或About）,可以看出，在App组件中，我们还增加了`Header`和`Footer`两个组件，给`About`和`Home`加上了页头页尾

另外需要注意的是`history`这个`props`,它有三个选项
* browserHistory 基于浏览器URL(使链接更有意义，符合通常的认识，一般需要服务端支持url重写)
* hashHistory 基于url的hash值(url中`#`之后的部分)
* createMemoryHistory

其它常用的组件:
* ``<Link>``
* ``<Redirect>``

以上是简单介绍，更多信息可访问[官方文档](https://github.com/reactjs/react-router/blob/master/docs/API.md)，查看说明


## redux 和 react-redux
redux是JavaScript状态容器,用于管理应用的状态（个人理解实际就是应用的数据和引起数据变更的操作，在react应用开发里，就是指定渲染react组件需要的数据和各种回调函数）

redux基础概念

### [API](http://cn.redux.js.org/docs/api/index.html)
* createStore
* Store
* combineReducers
* applyMiddleware
* bindActionCreators
* compose

### state
前端各种组件在特定的state下渲染会有所不同，一个展示组件就是通过state的变化来渲染界面的。
```
state = {
    todoReducer: {
        todos: [
            {text:'还书',completed: false},
            {text:'买菜',completed: true}
        ]
    }
}
```
### store
Store就是用来维持应用所有的state树的一个对象。应用的所有state以单一的对象存在一个单一的store中.可以理解为应用的数据库,存放渲染应用需要的所有数据。通过dispatch一个action来修改store中的state

Store对象的方法
* getState() 返回应用当前的 state 树。
* dispatch(action) 分发 action。这是触发 state 变化的惟一途径
* subscribe(listener) 添加一个变化监听器。每当 dispatch action 的时候就会执行，state 树中的一部分可能已经变化。你可以在回调函数里调用 getState() 来拿到当前 state。
* replaceReducer(nextReducer) 替换 store 当前用来计算 state 的 reducer。

### action
实际就是一个对像，必须包含一个`type`字段,`type`字段相当于定义了action的名字。通常长下面这个样子

```javascript
const ADD_TODO = "ADD_TODO"
{
    type: ADD_TODO
    text: ''
}
```

通常我们通过一个函数来封装action
```javascript
function addTodo(todo){
    return {
        type: ADD_TODO,
        text: todo
    }
}
```


### reducers
实际就是回调函数的处理逻辑,响应的是`action`定义的操作，当`action`被调用的时候，触发指定的reducers函数，函数接受旧的state和action作为参数,返回一个新的state,新的state引起页面组件发生变化，从而使页面展示的信息发生变化

需要特别理解的是，reducers函数必须返回一个新的state对象

```javascript
function todoReducer(state,action){
        switch(action.type){
            case ADD_TODO:
                return Object.assign({},state,{text:action.text,completed:false}); //合并新的
                break;
            default:
                return state;
        }
}

```

### dispatch
有了`action`和`reducers`定义，那怎么把它们联系起来呢，这时候就要用到`dispatch`了。通常就是下面这个样子

```javascript
dispatch({
    type: ADD_TODO,
    text: '喝水'
})
```

`dispatch` 的参数必须是一个`action` 对象（稍后会在react-thunk中再提到这个），调用`dispatch`后,指定的`reducers`将接收到这个调用，然后处理调用，返回新的状态。我们执行上面的`dispatch`后,state会变成下面这个样子

```javascript
state = {
    todoReducer: {
        todos: [
            {text:'还书',completed: false},
            {text:'买菜',completed: true},
            {text:'喝水',completed: false} //这一行是新添加的

    }
}
```

### 整体过一遍
* 定义好`action`和`reducers`
* 通过将reducers传递给｀createStore（）｀来创建一个`store`对象
* 在应用中通过`dispatch`方法触发`action`,改变`state`

### react-redux
通过提供容器组件把sotre和dispatch绑定到真正展示用的组件中去

API
* ``<Provider``
* `connect()`


```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import {createStore} from 'redux'
import {Provider} from 'react-redux'

import todoReducer from './reducers/todoReducer'

let store = createStore(todoReducer)
ReactDOM.render((
  <Provider store={store}>
        <App/>
  </Provider>
), document.getElementById('react-content'));

```

上面的代码中，我们通过传入｀todoReducer｀给`redux库`的`createStore`API函数，创建了`App`组件的唯一｀store｀对象。
然后再通过｀react-redux｀的`Provider`组件使得`Provider`之下的组件都能通过 `connect()` 方法获得 Redux store。

在App组件中

```javascript
import Reactfrom 'react'
import Header from '../pub/Header'
export default class App extends React.Component {
  render() {
    return (
      <div>
          <Header/>
          {this.props.children}
          <Footer/>
      </div>
    )
  }
}
```

因为App组件包裹在Provider之下，所以App下的Header组件也能通过connect获取store对象
```javascript
import React, {PropTypes} from 'react'
import {connect} from 'react-redux'

import {initMenu} from '../actions/Action'
import MainMenu from './MainMenu'

class Header extends React.Component {
  componentDidMount() {
    let {dispatch} = this.props.store
    dispatch(addTodo())
  }
  render() {
    return (
      <header id="header"></header>
    )
  }
}

function mapStateToProps(store) {
  return {store: store}
}
export default connect(mapStateToProps)(Header)

```

## react-thunk

上面谈`dispatch`的时候，我们说到，｀dispatch｀必须接受一个｀action｀对象作为参数，像如下这样

```javascript
function addTodo(todo){
    return {
        type: ADD_TODO,
        text: todo
    }
}
```

react-thunk是redux中间件，让`dispatch`能使用除了`action`以外的其它内容作为参数.比如下面的例子，我们定义一个fetchTodo函数，从某个远程接口获取一条todo数据，然后把这条数据通过dispatch添加到新的todo列表中

```javascript
function addTodo(todo){
    return {
        type: ADD_TODO,
        text: todo
    }
}

function fetchTodo(){
    return dispatch => {
        $.get("http://api.com/gettodo",function(todo){
            dispatch(addTodo(todo));
        })
    }
}
```
