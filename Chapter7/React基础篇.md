# 第1节：React基础篇

<!-- toc -->

- [1、react生命周期](#1、react生命周期)
- [2、react15和16的生命周期函数有什么变化？](#2、react15和16的生命周期函数有什么变化？)
- [3、getDerivedStateFromProps怎样使用](#3、getDerivedStateFromProps怎样使用)
- [4、列表组件中写 key的作用是什么？](#4、列表组件中写 key的作用是什么？)
- [5、组件更新](#5、组件更新)
- [6、组件通信](#6、组件通信)

<!-- tocstop -->



## 1、react生命周期

#### 1.1、组件初始化(initialization)阶段

也就是以下代码中类的构造方法( constructor() )，声明的组件继承了react Component这个基类，才能有render()，生命周期等方法可以使用，这也说明为什么函数组件不能使用这些方法的原因。

**super(props)**用来调用基类的构造方法( constructor() )，实际上就是调用parents.call(this)，也将父组件的props注入给子组件，供子组件读取(组件中props只读不可变，state可变)。

constructor()常用来做一些组件的初始化工作，如定义this.state的初始内容。

#### 1.2、组件挂载(Mounting)阶段

**此阶段分为componentWillMount，render，componentDidMount三个时期**

- **componentWillMount**

在组件挂载到DOM前调用，且只会被调用一次，在这边调用this.setState不会引起组件重新渲染，也可以把写在这边的内容提前到constructor()中，所以项目中很少用。

- **render**

根据组件的props和state，return 一个React元素（描述组件，即UI），不负责组件实际渲染工作，之后由React自身根据此元素去渲染出页面DOM。

render是纯函数（Pure function：函数的返回结果只依赖于它的参数；函数执行过程里面没有副作用），不能在里面执行this.setState，会有改变组件状态的副作用。

- **componentDidMount**

组件挂载到DOM后调用，且只会被调用一次，一般用于进行异步请求、起定时器或者进行事件订阅。

#### 1.3、组件更新(update)阶段

**此阶段分为componentWillReceiveProps，shouldComponentUpdate，componentWillUpdate，render，componentDidUpdate。**

- **componentWillReceiveProps(nextProps)**

此方法只调用于props引起的组件更新过程中，参数nextProps是父组件传给当前组件的新props。但父组件render方法的调用不能保证重传给当前组件的props是有变化的，所以在此方法中根据nextProps和this.props来查明重传的props是否改变，以及如果改变了要执行啥，比如根据新的props调用this.setState触发当前组件的重新render

- **shouldComponentUpdate(nextProps, nextState)**

此方法通过比较nextProps，nextState及当前组件的this.props，this.state，返回true时当前组件将继续执行更新过程，返回false则当前组件更新停止，以此可用来减少组件的不必要渲染，优化组件性能。

ps：这边也可以看出，就算componentWillReceiveProps()中执行了this.setState，更新了state，但在render前（如shouldComponentUpdate，componentWillUpdate），this.state依然指向更新前的state，不然nextState及当前组件的this.state的对比就一直是true了。

- **componentWillUpdate(nextProps, nextState)**

此方法在调用render方法前执行，在这边可执行一些组件更新发生前的工作，一般较少用。

- **render**

- **componentDidUpdate(prevProps, prevState)**

此方法在组件更新后被调用，可以操作组件更新的DOM，prevProps和prevState这两个参数指的是组件更新前的props和state

#### 1.4、组件卸载(UnMounting)阶段

**此阶段只有一个生命周期方法：componentWillUnmount**

此方法在组件被卸载前调用，**可以在这里执行一些清理工作，比如清除组件中使用的定时器，清除componentDidMount中手动创建的DOM元素等，以避免引起内存泄漏。**



## 2、react15和16的生命周期函数有什么变化？

16中引入了React Fiber的概念。

React Fiber一个更新过程被分为两个阶段(Phase)：第一个阶段Reconciliation Phase和第二阶段Commit Phase。

在第一阶段Reconciliation Phase，React Fiber会找出需要更新哪些DOM，这个阶段是可以被打断的；但是到了第二阶段Commit Phase，那就一鼓作气把DOM更新完，绝不会被打断。

这两个阶段大部分工作都是React Fiber做，和我们相关的也就是生命周期函数。

以render函数为界，第一阶段可能会调用下面这些生命周期函数，说是“可能会调用”是因为不同生命周期调用的函数不同。

- `componentWillMount`
- `componentWillReceiveProps`
- `shouldComponentUpdate`
- `componentWillUpdate`

React Fiber把更新过程碎片化，每执行完一段更新过程，就把控制权交还给React负责任务协调的模块，看看有没有其他紧急任务要做，如果没有就继续去更新，如果有紧急任务，那就去做紧急任务。

原来（React v16.0前）的生命周期在React v16推出的[Fiber](https://zhuanlan.zhihu.com/p/26027085)之后就不合适了，因为如果要开启async rendering，在render函数之前的所有函数（也就是第一阶段的函数），都有可能被执行多次。

原来（React v16.0前）的生命周期有哪些是在render前执行的呢？

- `componentWillMount`
- `componentWillReceiveProps`
- `shouldComponentUpdate`
- `componentWillUpdate`

如果开发者开了async rendering，而且又在以上这些render前执行的生命周期方法做AJAX请求的话，那AJAX将被无谓地多次调用。这明显不是我们期望的结果。而且在componentWillMount里发起AJAX，不管多快得到结果也赶不上首次render，而且componentWillMount在服务器端渲染也会被调用到（当然，也许这是预期的结果），这样的IO操作放在componentDidMount里更合适。

禁止不能用比劝导开发者不要这样用的效果更好，所以除了**shouldComponentUpdate**，其他在render函数之前的所有函数（componentWillMount，componentWillReceiveProps，componentWillUpdate）都被打上了UNSAFE的标记，从而被新的生命周期函数所取代。

新增的生命周期函数如下：

`static getDerivedStateFromProps(nextProps, prevState) // 也就是强制开发者在render之前只做无副作用的操作，而且能做的操作局限在根据props和state决定新的state。`

`getSnapshotBeforeUpdate(prevProps, prevState)`

`componentDidCatch(error, info)`



**static getDerivedStateFromProps 和 componentWillReceiveProps 的显著区别**

- **触发机制:**

UNSAFE_componentWillReceiveProps(nextProps) 在组件接收到新的参数时被触发。

当父组件导致子组件更新的时候, 即使接收的 props 并没有变化，这个函数也会被调用。

getDerivedStateFromProps(props, state) 会在每次组件渲染前被调用。

getDerivedStateFromProps 会在每次组件被重新渲染前被调用，这意味着无论是1、父组件的更新，2、props 的变化，3、组件内部执行了 setState(), 它都会被调用.

- **工作方式**

UNSAFE_componentWillReceiveProps(nextProps)：参数是组件接收到的新的 props ，用于比对新的 props 和原有的 props，用户需要在函数体中调用 setState() 来更新组件的数据。

static getDerivedStateFromProps(nextProps, currentState)：参数是组件接收到的新的 props 和组件当前的数据. 用户需要在这个函数中返回一个对象, 它将作为 setState() 中的 Updater 更新组件。



## 3、getDerivedStateFromProps怎样使用

- **让表单控件变成完全受控组件, 不论是 onChange 处理函数还是 value 都由父组件控制, 这样用户无需再考虑这个组件 props 的变化和 state 的更新.**

```javascript
function EmailInput(props) {
  return <input onChange={props.onChange} value={props.email} />;
}
```

- **让表单控件变成完全不受控组件, 但是具有 key 属性.**

仍然用自身的数据来控制 value. 但是接收 props 中的某个字段作为 key 属性的值, 以此响应 props 的更新: 当 key 的值变化时 React 会重置 (reset)组件，从而重新生成初始化数据.

如下例中，实际上达到的效果就是props中的user.id改变，email的value就改变：

```javascript
class EmailInput extends Component {
  state = { email: this.props.defaultEmail };

  handleChange = event => {
    this.setState({ email: event.target.value });
  };

  render() {
    return <input onChange={this.handleChange} value={this.state.email} />;
  }
}

// 在父组件中接收 props 中的数据作为 key
<EmailInput
  defaultEmail={this.props.user.email}
  key={this.props.user.id}
/>
```





## 4、列表组件中写 key的作用是什么？

react利用key来识别组件，有了key属性后，就可以与组件建立一种对应关系，**react根据key来决定是销毁重新创建组件还是更新组件**。

- key相同，若组件属性有所变化，则react只更新组件对应的属性；没有变化则不更新。
- key值不同，则react先销毁该组件(有状态组件的componentWillUnmount会执行)，然后重新创建该组件（有状态组件的constructor和componentWillUnmount都会执行）



**key取值原则：**

- **key值在同级同类型的组件上要保持唯一**

- **尽量不要用数组的index去作为key**

主要从以下两个方面考虑

--错误处理组件更新：

当我们对数据进行逆序等操作时，index和组件的对应关系就不再和之前一致，这种情况下diff算法对新老集合进行差异检测，就无法发现key值有变化然后重新渲染

--性能：

当我们对数据有删除、添加等操作时。我们所遍历的index，就会有所变化，这种情况下diff算法对新老集合进行差异检测，发现key值有变化然后就会重新渲染，

我们只需要乖乖的把id（或者其他唯一标识）作为key，这样就只会对key值有变化的进行重绘，就可以解决这种性能问题了。

- **不要试图在render的时候用随机数或者其他操作给元素加上不稳定的key，这样造成的性能开销比不加key的情况下更糟糕**



## 5、组件更新

造成组件更新有以下几种情况：

#### 5.1、父组件重新render引起子组件重新render

父组件重新render引起子组件重新render的情况有两种：

- 直接使用，每当父组件重新render导致的重传props，子组件将直接跟着重新渲染，无论props是否有变化（或无论是否传了props）

这种方式，父组件改变props后，子组件重新渲染，由于直接使用的props，所以我们不需要做什么就可以正常显示最新的props

```javascript
class Child extends Component {
    render() {
        return <div>{this.props.someThings}</div>
    }
}
```

可通过shouldComponentUpdate方法优化：

```javascript
class Child extends Component { 
  shouldComponentUpdate(nextProps){ 
  // 应该使用这个方法，否则无论props是否有变化都将会导致组件跟着重新渲染 
    if(nextProps.someThings === this.props.someThings)
    { return false } 
  } 
  render() { 
    return <div>{this.props.someThings}</div> 
  } 
}
```



-  将props转换成自己的state

这种方式，由于我们使用的是state，所以每当父组件每次重新传递props时，我们需要重新处理下，将props转换成自己的state，这里就用到了 **componentWillReceiveProps**。

每次子组件接收到新的props，都会重新渲染一次，除非你做了处理来阻止（比如使用：shouldComponentUpdate），但是你可以在这次渲染前，根据新的props更新state，更新state也会触发一次重新渲染，但react不会这么傻，所以只会渲染一次，这对应用的性能是有利的。

```javascript
class Child extends Component {
    constructor(props) {
        super(props);
        this.state = {
            someThings: props.someThings
        };
    }
    componentWillReceiveProps(nextProps) {
        this.setState({someThings: nextProps.someThings});
    }
    render() {
        return <div>{this.state.someThings}</div>
    }
}
class Child extends Component {
  constructor(props) {
    super(props);
    this.state = { someThings: props.someThings };
  }
  componentWillReceiveProps(nextProps) {
    // 父组件重传props时就会调用这个方法
    this.setState({someThings: nextProps.someThings});
  }
  render() {
    return <div>{this.state.someThings}</div>
  }
}
```

是因为componentWillReceiveProps中判断props是否变化了，若变化了，this.setState将引起state变化，从而引起render，此时就没必要再做第二次因重传props引起的render了，不然重复做一样的渲染了。

React16中新的生命周期函数被引入了, 即静态方法 getDerivedStateFromProps，来取代componentWillReceiveProps



#### 5.2、组件本身调用setState

可通过shouldComponentUpdate方法优化

```javascript
class Child extends Component { 
  constructor(props) { 
    super(props); 
    this.state = { someThings:1 } 
  } 
  shouldComponentUpdate(nextStates){ 
    // 应该使用这个方法，否则无论state是否有变化都将会导致组件重新渲染 
    if(nextStates.someThings === this.state.someThings){ 
      return false 
    } 
  } 
  handleClick = () => { 
    // 虽然调用了setState ，但state并无变化 
    const preSomeThings = this.state.someThings 
    this.setState({ someThings: preSomeThings }) 
  } 
  render() { 
    return <div onClick = {this.handleClick}>{this.state.someThings}</div> 
  } 
}
```



**扩展**

**【state和props作用和区别】**

props是一个从外部传进组件的参数，主要作用就是从父组件向子组件传递数据，它具有可读性和不变性，只能通过外部组件主动传入新的props来重新渲染子组件，否则子组件的props以及展现形式不会改变。

state的主要作用是用于组件保存、控制以及修改自己的状态，它只能在constructor中初始化，它算是组件的私有属性，不可通过外部访问和修改，只能通过组件内部的this.setState来修改，修改state属性会导致组件的重新渲染。



## 6、组件通信

#### 6.1、父组件向子组件通信

在 React 中，父组件可以向子组件通过传 props 的方式，向子组件进行通讯。

#### 6.2、子组件向父组件通信

- **利用回调函数**

利用回调函数，可以实现子组件向父组件通信：父组件将一个函数作为 props 传递给子组件，子组件调用该回调函数，便可以向父组件通信。

- **利用自定义事件机制** 

#### 6.3、跨级组件通信

所谓跨级组件通信，就是父组件向子组件的子组件通信，向更深层的子组件通信。跨级组件通信可以采用下面两种方式：

- 中间组件层层传递 props
- 使用 context 对象



**扩展**

**【 context 】**

如果要Context发挥作用，需要用到两种组件，一个是Context生产者(Provider)，通常是一个父节点，另外是一个Context的消费者(Consumer)，通常是一个或者多个子节点。所以Context的使用基于**生产者消费者模式**。

对于父组件，也就是Context生产者，需要通过一个静态属性childContextTypes声明提供给子组件的Context对象的属性，并实现一个实例getChildContext方法，返回一个代表Context的纯对象 (plain object) 。

```javascript
import React, { Component } from 'react'
import PropTypes from 'prop-types'

class ParentComponent extends Component {
  // 声明Context对象属性
  static childContextTypes = {
    propA: PropTypes.string,
    methodA: PropTypes.func
  }
  // 返回Context对象，方法名是约定好的
  getChildContext () {
    return {
      propA: 'A',
      methodA: () => {
        console.log('hello, world')
      }
    }
  }

  render () {
    return <MiddleComponent />
  }
}

class MiddleComponent extends Component {
  render () {
    return <ChildComponent />
  }
}

class ChildComponent extends Component {
    // 子组件需要通过一个静态属性contextTypes声明后，才能访问父组件Context对象的属性，否则，即使属性名没写错，拿到的对象也是undefined
    static contextTypes = {
        propA: PropTypes.string,
        methodA: PropTypes.func 
    }

  render () {
    const {
      propA,
      methodA
    } = this.context
    methodA()
    return (<div>{propA}</div>)
  }
}
```

对于无状态子组件(Stateless Component)，可以通过如下方式访问父组件的Context

```javascript
const ChildComponent = (props, context) => {
  const {
    propA
  } = context
    
  return (<div>{propA}</div>)
}
```



事实上，很多优秀的React组件都通过Context来完成自己的功能，**比如react-redux**，就是通过**Context**提供一个全局态的**store**，拖拽组件react-dnd，通过Context在组件中分发DOM的Drag和Drop事件，路由组件react-router通过Context管理路由状态等等。在React组件开发中，如果用好Context，可以让你的组件变得强大，而且灵活。