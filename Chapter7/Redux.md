# 第3节：Redux

<!-- toc -->

- [1、redux机制](#1、redux机制)
- [2、react-redux](#2、react-redux)

<!-- tocstop -->



## 1、redux机制

#### 1.1、什么是redux

Redux是一个流行的JavaScript框架，为应用程序提供一个可预测的**状态容器**。Redux基于简化版本的Flux框架，Flux是Facebook开发的一个框架。

在Redux中，所有的数据（比如state）被保存在一个被称为store的容器中（一个应用程序中只能有一个store）。**store本质上是一个状态树，保存了所有对象的状态。**任何UI组件都可以直接从store访问特定对象的状态。想要更改状态，需要分发一个action（分发在这里意味着将可执行信息发送到store）。当store接收到action后，会把这个action代理给相关的reducer。reducer是一个纯函数，它通过action.type执行对应action并且返回一个新的状态。

使用Redux的主要优势之一是它可以帮你处理应用的共享状态。如果两个组件需要访问同一状态，该怎么办？这种两个组件同时需要访问同一状态的现象称为“共享状态”。你可以将该状态提升到附近的父组件，但是如果该父组件在组件树中向上好几个组件的位置，那么将状态当做属性向下一个一个地传递，很容易就会使组件的状态变得混乱且难以管理。



#### 1.2、redux的三大原则

- **单一数据源**

Redux的基本原则之一是存在单一数据源：Store。也就是说，所有状态全部存储在一个对象树store中。

只有单个状态树，对于应用的很多方面都有好处。假设在构建应用时尝试实现撤消/重做功能，如果所有状态都存储在一个树（单一数据源）中，则实现起来比数据分散在多个组件中简单多了。状态集中到一个位置后，调试和检测过程也会简单很多。

- **State 只读**

  **唯一改变 state的方法就是触发action**，action是一个用于描述发生事件(动作)的对象。

  这样设计的好处是：

  - 增强了可预测性和可靠性
  - 避免产生副作用
  - 阻止外部文件修改state
  - 所有对state的改动都被集中于一个地方，并且被严格地依次触发
  - 更改state的唯一方式是派发相应的action，以描述所需的更改

- **使用纯函数来执行修改**

为了描述action如何改变state tree ，你需要编写reducers，他是一个纯函数，在这里进行一系列处理并返回一个新的state。



#### 1.3、纯函数

- **纯函数**

  纯函数是 Redux 应用中，更新状态的必要手段。纯函数的定义是:

  - 对于同一参数，返回同一结果

  该函数结果值不依赖任何隐藏信息或程序执行处理可能改变的状态或在程序的两个不同的执行

  - 输出完全取决于传入的参数
  - 不会产生副作用

- **纯函数的条件之一是不产生副作用**

  副作用是指函数对其外部世界产生影响，包括：

  - 发出 HTTP 调用
  - 改变外部状态
  - 检索今天的日期
  - Math.random()
  - 向控制台输出消息
  - 向数据库中添加数据

- **纯函数优点：**

  - 纯函数本质上就是模块化的，这使它更容易被测试。由于当参数相同时，纯函数总是产⽣相同的结果，你不必担⼼应用其他部分的数据受到影响。在调试期间，这将给予明确定义的额外控制点。

  - 纯函数使代码更好维护。纯函数不会产生副作用。这意味着你在重构应用时，纯函数不会对其外部内容产生任何不利影响。

    

## 2、react-redux

#### 2.1、react-redux 解决的问题

- 负责应用的状态管理，保证单向数据流
- 监听状态，在数据发生改变时，执行预期的操作



#### 2.2、react-redux原理分析

**react-redux 的使用分为两步：**

1. **第一步：使用 Provider 在顶层创建一个 Root 节点，将创建的 store 作为 Provider 的 props 传入**
2. **第二步：在需要使用 store 的页面，使用 connect 将组件与 store 建立连接关系，需要用到的值通过 mapStateToProps 传入对应的组件中**

 

-  **Provider**

首先在最外层容器中，把所有内容包裹在Provider组件中，将之前创建的store作为prop传给Provider。 

Provider接收redux的createStore()的结果，并且放到context里，让子组件可以通过context属性直接获取到这个createStore的结果。

```javascript
const store = createStore(combineReducers, {})

const App = () => {
  return (
    <Provider store={store}>
      <Comp/>
    </Provider>
  )
}
```

createStore返回的结果内容如下：

```javascript
return {
        //真正的返回，执行createStore其实返回的就是这些东东
        dispatch,       //触发action去执行reducer，更新state
        subscribe,     //订阅state改变，state改变时会执行subscribe的参数（自己定义的一个函数）
        getState,      //获取state树
        replaceReducer,       //替换reducer
 }
```

- **Connect**

**connect接收到mapStateToProps，会在内部subscribe全局state的改变，来判断props是否更改，如果需要更新，才触发更新。**

一个基础的connect方法如下：

`connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {})` 

`mapStateToProps：传入所有state，返回指定的state数据`

`mapDispatchToProps：传入dispatch，返回action方法`

通过connect这个高阶组件包裹的对象组件，可以访问到定制化的数据或者方法，并且无需自己去subscribe全局state的变化，当state变化时，所有被connect包裹的对象组件都会被一次触发更新。

一个connect高阶组件的简单实现：

```javascript
export const connect = (mapStateToProps, mapDispatchToProps) => (WrappedComponent) => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    constructor () {
      super()
      this.state = {
        allProps: {}
      }
    }

    componentWillMount () {
      const { store } = this.context
      this._updateProps()
      store.subscribe(() => this._updateProps())
      // 通过subscribe订阅state的变化，state改变时会执行subscribe传入的入参函数
    }

    _updateProps () {
      const { store } = this.context
      let stateProps = mapStateToProps
        ? mapStateToProps(store.getState(), this.props)
        : {} // 防止 mapStateToProps 没有传入
      let dispatchProps = mapDispatchToProps
        ? mapDispatchToProps(store.dispatch, this.props)
        : {} // 防止 mapDispatchToProps 没有传入
      this.setState({
        allProps: {
          ...stateProps,
          ...dispatchProps,
          ...this.props
        }
      })
    }

    render () {
      return <WrappedComponent {...this.state.allProps} />
    }
  }
  return Connect
}
```

可以看出，在connect这个高阶组件中，我们通过context拿到store对象，然后给一个_updateProps的更新函数，这个更新函数中将connect入参mapStateToProps和mapDispatchToProps去做setState更新，然后做为props注入目标组件。

由于这个_updateProps不光会在componentWillMount执行一次，同时也会做为subscribe的入参，subscribe能够订阅state的改变，因此后续有state发生变化时，订阅过的组件（也就是被connect包裹的组件）都会被触发更新。

- **middleware**

redux工作的整个过程都是同步的，只要action被dispatch到reducer，对应state发生变化，UI就立即更新。

如果dispatch一个action之后，到达reducer之前，进行一些额外的操作，就需要用到middleware。你可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

换言之，中间件都是对store.dispatch()的增强。

```javascript
import { applyMiddleware, createStore } from 'redux';
import thunk from 'redux-thunk';
const store = createStore(
  reducer,
  applyMiddleware(thunk)
)
```

中间件都要放到applyMiddleware里，如果要添加中间件，可以依次添加，但是要遵循文档定义的顺序。

createStore其实可以接受三个参数，第二个参数preloadedState一般作为整个应用的初始化数据，如果传入了这个参数，applyMiddleware就会被当做第三个参数处理。

上述代码中applyMiddleware的入参thunk是一个包裹着一个表达式以延迟其求值的函数，他可以用来延迟dispatch一个action或者只在某些特定场景下才dispatch。

在我们没有加上thunk这个中间件之前，store的dispatch方法只能传入一个action对象，thunk的作用就是能够让我们可以将一个function方法传入diptach，这在做异步的时候非常有用。

```javascript
const login = (userName) => (dispatch) => {
  request.post('/api/login', { data: userName }, () => {
    dispatch({ type: 'loginSuccess', payload: userName })
  })
}
```



【扩展】

applyMiddleware源代码

```javascript
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {...store, dispatch}
  }
}
```

从源代码可以看出，实现中间件的核心代码是compose，他接受多个函数做为参数，并返回一个新的函数，新的函数会从右至左依次执行原函数，并且上一个结果的返回值会做为下一个函数的参数。

compose具体实现可以参考：[第一章 Javascript/第2节： 原理性实现](https://piggyzss.github.io/piggybook/Chapter1/%E5%8E%9F%E7%90%86%E6%80%A7%E5%AE%9E%E7%8E%B0.html#6%E3%80%81%E5%AE%9E%E7%8E%B0compose%E5%92%8Cpipe)

 