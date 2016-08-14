---
title: Redux概念
categories:
    - node
    - 前端
---

# Redux

## state
当前应用的数据状态，决定了UI应该如何展示，state变化通常将导致UI变化

## action
描述“发生了什么的”对象，用于数据传递。必须包含一个type字段用于表示将执行的动作.
通常被 store.dispatch()调用

例如以下action可用于描述id为42的文章被点赞这个行为
```
｛type: 'LIKE_ARTICLE', articleId: 42 };
```

## reducer
计算数据，将变化的数据合并或去除，返回变化后的state。实际上就是响应store.dispatch()调用。
处理action返回新的state

## store
存放应用中所有的state，将action和reducer联系在一起。通过dispatch一个action触发reducer，从而改变应用的state

Store 有以下职责：
* 维持应用的 state；
* 提供 getState() 方法获取 state；
* 提供 dispatch(action) 方法更新 state；
* 通过 subscribe(listener) 注册监听器。

## Redux函数和react-redux组件

### Redux
* `createStore(reducer, [initialState])` 创建一个 Redux store 来以存放应用中所有的 state　　
    - 参数
        - reducer (Function):

        接收两个参数，分别是当前的 state 树和要处理的 action，返回新的 state 树。

        - [initialState] (any):

        初始时的 state。在同构应用中，你可以决定是否把服务端传来的 state 水合（hydrate）后传给它，或者从之前保存的用户会话中恢复一个传给它。如果你使用 combineReducers 创建 reducer，它必须是一个普通对象，与传入的 keys 保持同样的结构。否则，你可以自由传入任何 reducer 可理解的内容。

* `combineReducers(reducers)`

    随着应用变得复杂，需要对 reducer 函数 进行拆分，拆分后的每一块独立负责管理 state 的一部分。

    combineReducers 辅助函数的作用是，把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 createStore。

    合并后的 reducer 可以调用各个子 reducer，并把它们的结果合并成一个 state 对象。state 对象的结构由传入的多个 reducer 的 key 决定。

### react-redux

* ``<Provider store={}>``

    对组件注入`Redux Store`

* connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])

 连接 `React` 组件与 `Redux store`

 参数
 * mapStateToProps(state, [ownProps]): stateProps

     如果定义该参数，组件将会监听 Redux store 的变化。任何时候，只要 Redux store 发生改变，mapStateToProps 函数就会被调用。该回调函数必须返回一个纯对象，这个对象会与组件的 props 合并。如果你省略了这个参数，你的组件将不会监听 Redux store。如果指定了该回调函数中的第二个参数 ownProps，则该参数的值为传递到组件的 props，而且只要组件接收到新的 props，mapStateToProps 也会被调用。

 * mapDispatchToProps(dispatch, [ownProps]): dispatchProps

    如果传递的是一个对象，那么每个定义在该对象的函数都将被当作 Redux action creator，而且这个对象会与 Redux store 绑定在一起，其中所定义的方法名将作为属性名，合并到组件的 props 中。如果传递的是一个函数，该函数将接收一个 dispatch 函数，然后由你来决定如何返回一个对象，这个对象通过 dispatch 函数与 action creator 以某种方式绑定在一起（提示：你也许会用到 Redux 的辅助函数 bindActionCreators()）。如果你省略这个 mapDispatchToProps 参数，默认情况下，dispatch 会注入到你的组件 props 中。如果指定了该回调函数中第二个参数 ownProps，该参数的值为传递到组件的 props，而且只要组件接收到新 props，mapDispatchToProps 也会被调用。

 * mergeProps(stateProps, dispatchProps, ownProps): props

    如果指定了这个参数，mapStateToProps() 与 mapDispatchToProps() 的执行结果和组件自身的 props 将传入到这个回调函数中。该回调函数返回的对象将作为 props 传递到被包装的组件中。你也许可以用这个回调函数，根据组件的 props 来筛选部分的 state 数据，或者把 props 中的某个特定变量与 action creator 绑定在一起。如果你省略这个参数，默认情况下返回 Object.assign({}, ownProps, stateProps, dispatchProps) 的结果。

 * options 如果指定这个参数，可以定制 connector 的行为。

     * pure = true

     如果为 true，connector 将执行 shouldComponentUpdate 并且浅对比 mergeProps 的结果，避免不必要的更新，前提是当前组件是一个“纯”组件，它不依赖于任何的输入或 state 而只依赖于 props 和 Redux store 的 state。默认值为 true。

     * withRef = false

     如果为 true，connector 会保存一个对被包装组件实例的引用，该引用通过 getWrappedInstance() 方法获得。默认值为 false



### 步骤


* 使用Redux的createStore()创建一个包含了action和reducer的store
* 将应用根组件嵌套到react-redux的 ``<Provider store>`` 组件中，从页实现绑定
* 在应用根组件中使用react-redux的connect()实现连接，此时应用根组件的props可以接收到connect函数mapStateToProps参数中指定的store数据和方法


### 参考网站

* [http://cn.redux.js.org](http://cn.redux.js.org)
