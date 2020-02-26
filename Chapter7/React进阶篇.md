# 第2节：React进阶篇

<!-- toc -->

- [1、虚拟dom](#1、虚拟dom)
- [2、diff算法](#2、diff算法)
- [3、setState](#3、setState)
- [4、react事件和原生事件](#4、react事件和原生事件)
- [5、高阶组件](#5、高阶组件)

<!-- tocstop -->



## 1、react生命周期

#### 1.1、概念

虚拟DOM就是一个JavaScript 对象，包含了 tag、props、children 三个属性。

**虚拟DOM能够提高渲染的性能，数据与UI分离，并且有利于服务器渲染。**

```html
<div id="app">
  <p class="text">hello world!!!</p>
</div>
```

上面的 HTML 转换为虚拟 DOM 如下：

```json
{
  tag: 'div',
  props: {
    id: 'app'
  },
  chidren: [
    {
      tag: 'p',
      props: {
        className: 'text'
      },
      chidren: [
        'hello world!!!'
      ]
    }
  ]
}
```

**virtual dom 基本步骤**

①用JS对象构建一颗虚拟DOM树，然后用虚拟树构建一颗真实的DOM树，然后插入到文档中。

②当状态变更时，重新构造一颗新的对象树，然后新树旧树进行比较，记录两树差异。

③把步骤2的差异部分应用到构建的真实DOM树上，视图就更新了。



#### 1.2、为什么需要虚拟DOM 

前端性能优化的一个很重要的点就是尽可能少地操作DOM，不仅仅是DOM相对较慢，更因为频繁变动DOM会造成浏览器的回流或者重绘，因此需要这一层抽象，在patch过程中尽可能地一次性将差异更新到DOM中，从而保证了DOM性能。



#### 1.3、虚拟DOM 性能

**直接操作 DOM 的性能并不会低于虚拟 DOM 和 Diff 算法，甚至还会优于。**

这么说的原因是因为 Diff 算法的比较过程，比较是为了找出不同从而有的放矢的更新页面。但是比较也是要消耗性能的。而直接操作 DOM 就是有的放矢，我们知道该更新什么不该更新什么，所以不需要有比较的过程。所以直接操作 DOM 效率可能更高。

那么为什么还需要虚拟DOM 呢？

虚拟 DOM 和 Diff 算法的出现是为了解决由命令式编程转变为声明式编程、数据驱动后所带来的性能问题。 DOM 发生变化的时候，通过 diff 算法比对 JavaScript 原生对象，计算出需要变更的 DOM，然后只对变化的 DOM 进行操作，而不是更新整个视图。



#### 1.4、渲染虚拟DOM

```javascript
function render(vdom) {
  // 如果是字符串或者数字，创建一个文本节点
  if (typeof vdom === 'string' || typeof vdom === 'number') {
    return document.createTextNode(vdom)
  }
  const { tag, props, children } = vdom
  // 创建真实DOM
  const element = document.createElement(tag)
  // 设置属性
  setProps(element, props)
  // 遍历子节点，并获取创建真实DOM，插入到当前节点
  children
    .map(render)
    .forEach(element.appendChild.bind(element))

  // 虚拟 DOM 中缓存真实 DOM 节点
  vdom.dom = element

  // 返回 DOM 节点
  return element
}

function setProps (element, props) {
  Object.entries(props).forEach(([key, value]) => {
    setProp(element, key, value)
  })
}

function setProp (element, key, vlaue) {
  element.setAttribute(
    // className使用class代替
    key === 'className' ? 'class' : key,
    vlaue
  )
}
```

将虚拟 DOM 渲染成真实 DOM 后，只需要插入到对应的根节点即可

```javascript
class Component {
  vdom = null // 组件的虚拟DOM表示
  $el  = null // 虚拟DOM生成的真实节点
  state = {
    text: 'Initialize the Component'
  }

  render() {
    const { text } = this.state
    return (
      <div>{ text }</div>
    )
  }
}

function createElement (root, component) {
  const vdom = component.render()
  component.vdom = vdom
  component.$el = render(vdom) // 将虚拟 DOM 转换为真实 DOM
  root.appendChild(component.$el)
}

const root = document.getElementById('root')
const component = new Component()
createElement(root, component)
```



## 2、diff算法

#### 2.1、概念

diff 算法，顾名思义，就是比对新老 VDOM 的变化，然后将变化的部分更新到视图上。对应到代码上，就是一个 diff 函数，返回一个 patches （补丁）。

#### 2.2、React diff 三大策略

- **策略一（tree diff）**

Web UI中DOM节点跨层级的移动操作特别少，可以忽略不计。（DOM结构发生改变-----直接卸载并重新creat）

React的做法是把dom tree分层级，对于两个dom tree只比较同一层次的节点，忽略Dom中节点跨层级移动操作，只对同一个父节点下的所有的子节点进行比较。如果对比发现该父节点不存在则直接删除该节点下所有子节点，不会做进一步比较，这样只需要对dom tree进行一次遍历就完成了两个tree的比较。

- **策略二（component diff）**

DOM结构一样-----不会卸载，但是会update

对于同一类型组件合理使用shouldComponentUpdate（），应该避免结构相同类型不同的组件

- **策略三（element diff）**

所有同一层级的子节点，他们都可以通过key来区分-----同时遵循1.2两点

React会先进行新集合遍历，for(name in nextChildren)，通过key值判断两个对比集合中是否存在相同的节点，即if(prevChild === nextChild)，如何为true则进行移动操作



## 3、setState

#### 3.1、setState做了什么

调用了setstate之后，react会将传入的参数对象与组件当前的状态做合并，然后触发调和过程，react会根据新的状态构建react元素树，并重新渲染整个UI界面。

react得到元素树之后，react会自动计算出新树与老树节点的差异，然后根据差异对界面进行最小化重渲染

#### 3.2、关于setState的异步问题

setState 不会立刻改变React组件中state的值，他通过触发一次组件的更新来引发重绘，多次 setState 函数调用产生的效果会合并，然后一次引发更新过程，为的就是把 Virtual DOM 和 DOM 树操作降到最小，用于提高性能。

setState 通过 batchingStrategy，也就是批量更新策略来完成一次更新，isBatchingUpdates则是用来标志批量更新的标志位，通过它判断是直接更新 this.state还是放到队列中，默认是false，也就表示setState会同步更新this.state；如果为true就放到队列中，异步更新。



- 组件生命周期中的异步更新

例如，在componentDidMount中调用setState并不会立即更新state，因为正处于一个更新流程中，isBatchingUpdates为true，所以只会放入dirtyComponents中等待稍后更新。

- react事件中的异步更新

由React引发的事件处理（比如通过onClick引发的事件处理），调用 setState 只能异步更新 this.state。

React在调用事件处理函数之前就会把isBatchingUpdates置为true，因此由React控制的事件处理过程setState不会同步更新this.state。



#### 3.3、扩展

下面这个例子，企图通过点击事件之后就使用修改之后的state的值，但是会发state中的并没有被立即修改，还是原先的值，我们都知道那是因为setState就相当于是一个异步操作，不能立即被修改。

```javascript
handleClick = () => {
    this.setState({
        count: this.state.count + 1
    })
    console.log('count', this.state.count)
}
```

**解决方法：**

- 利用setState第二个参数（传入一个函数），该函数会在setState函数调用完成并且组件开始重渲染的时候被调用，我们可以用该函数来监听渲染是否完成

```javascript
handleClick = () => {
    this.setState({
    count: this.state.count + 1
    }, () => {
        console.log('count', this.state.count)
    })
}
```

- async&await

```javascript
handleClick = async () => {
    await this.setState({
        count: this.state.count + 1
    })
    console.log('count', this.state.count)
}
```



#### 3.4、setState 什么时候会执行同步更新？

**在React中，如果是由React引发的事件处理（比如通过onClick引发的事件处理），调用 setState 不会同步更新 this.state，除此之外的setState调用会同步执行this.state。**

所谓“除此之外”，指的是绕过React通过 addEventListener 直接添加的事件处理函数，还有通过setTimeout或setInterval 产生的异步调用。

**简单一点说， 就是经过React 处理的事件是不会同步更新this.state的。通过addEventListener、setTimeout、setInterval的方式处理的则会同步更新。**



## 4、react事件和原生事件

#### 4.1、React合成事件机制

React并不是将click事件直接绑定在dom上面，而是采用事件冒泡的形式冒泡到document上面，然后React将事件内容封装交给中间层SyntheticEvent（负责所有事件合成）。

当事件触发的时候，对使用统一的分发函数dispatchEvent将指定函数执行，这样 React 在更新 DOM 的时候就不需要考虑如何去处理附着在 DOM 上的事件监听器，最终达到优化性能的目的。

React 中的event 不是原生的event，e.nativeEvent 才是原生 DOM 事件的那个 event。



#### 4.2、React合成事件有哪些

鼠标事件

- onClick
- onDoubleClick
- onMouseDown
- onMouseUp
- onMouseEnter
- onMouseLeave
- onMouseMove

拖拽事件

- Drop
- onDrag

键盘事件

- onKeyPress
- onKeyDown
- onKeyUp

 焦点事件

- onFocus
- onBlur

表单事件

- onChange
- onSubmit



#### 4.3、React合成事件的事件流和事件委托

**事件流**

React中，默认的事件传播方式为**冒泡**

如果希望以捕获的方式来触发事件的话，可以使用onClickCapture来绑定事件，也就是在事件类型后面加一个后缀Capture

**事件委托**

- 在合成事件系统中，事件没有在目标对象上绑定，所有的事件都是绑定在document元素上。
- react会在内部维护一个映射表记录事件与组件事件处理函数的对应关系
- 当某个事件触发时，React根据这个内部映射表将事件分派给指定的事件处理函数
- 当一个组件安装或者卸载时，相应的事件处理函数会自动被添加到事件监听器的内部映射表中或从表中删除



#### 4.4、一些处理

- 阻止默认行为

React 中另一个不同是你不能使用返回 false 的方式阻止默认行为，明确的使用 preventDefault。

- 阻止冒泡

用reactEvent.nativeEvent.stopPropagation()来阻止冒泡是不行的。

阻止 React 事件冒泡的行为只能用于 React 合成事件系统中，且没办法阻止原生事件的冒泡。反之，在原生事件中的阻止冒泡行为，却可以阻止 React 合成事件的传播。

```javascript
    componentDidMount() {
        document.getElementById('li').addEventListener('click', this.onDomLiClick, false)
        document.getElementById('ul').addEventListener('click', this.onDomUlClick, false)
    }
    onDomLiClick(e: any) {  // 事件委托
        // e.stopPropagation()
        console.log('Li Dom click')
    }

    onDomUlClick(e: any) {
        console.log('Ul Dom click')
    }

    onReactUlClick() {  // react合成事件
        console.log('React Ul click');
    }

    onReactLiClick(e: any) {  // react合成事件
         e.stopPropagation()
        console.log('React Li click');
    }


    <ul id='ul' onClick={this.onReactUlClick}>
      <li id='li' onClick={this.onReactLiClick}>zhaoshanshan</li>
    </ul>
```



## 5、高阶组件

#### 5.1、什么是高阶组件

**高阶组件**:高阶组件是一个函数，它接受一个组件作为参数，返回一个新的组件

**高阶组件**通过包裹（wrapped）被传入的React组件，经过一系列处理，最终返回一个相对增强（enhanced）的React组件，供其他组件调用。

高阶组件让我们的代码更具有复用性, 逻辑性与抽象性

**它可以对render方法做挟持,  也可以控制props和state**

**--高阶组件的两种形式**

- **属性代理**：高阶组件通过包裹的React组件来操作props
- **反向继承**：高阶组件继承于被包裹的React组件

（待完成）



https://axiu.me/coding/react-batchedupdates-and-transaction/