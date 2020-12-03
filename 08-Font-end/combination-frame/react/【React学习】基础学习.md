# 1 React入门

## 1.1 React基础认识

React 是一个声明式，高效且灵活的用于构建用户界面的 JavaScript 库，由 Facebook 开源。使用 React 可以将一些简短、独立的代码片段组合成复杂的 UI 界面，这些代码片段被称作“组件”。

传统的Web UI开发的主要问题——DOM API关注太多细节。

![](https://img-blog.csdnimg.cn/2020113020122088.png)

而React的特点：

* Declarative（声明式编码）
* Component-Based（组件化编码）
* Learn Once，Write Anywhere（支持客户端与服务器渲染，React-Native）
* 高效
* 单向响应的数据流

**高效的原因**

* 虚拟（virtual）DOM，不总是操作真实DOM。采用DOM Diff 算法，最小化页面重绘。


## 1.2 React基本使用

**相关 js 库**

* **react.min.js** ——React 的核心库。
* **react-dom.min.js** ——提供与 DOM 相关的功能。
* **babel.min.js** ——Babel 可以将 ES6 代码转为 ES5 代码，这样我们就能在目前不支持 ES6 浏览器上执行 React 代码。Babel 内嵌了对 **JSX** 的支持，凡是使用 JSX 的地方，都要在 script 标签中加上 type="text/babel" 。

**React 开发者工具**

* [React DeveloperTools插件](https://github.com/facebook/react-devtools)。

![](https://img-blog.csdnimg.cn/2020111816474539.png)

入门案例——HelloReact：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>01_HelloWorld</title>
</head>
<body>
  <div id="test"></div>

  <script type="text/javascript" src="../js/react.development.js"></script>
  <script type="text/javascript" src="../js/react-dom.development.js"></script>
  <script type="text/javascript" src="../js/babel.min.js"></script>
  <script type="text/babel"> //凡是使用 JSX 的地方，都要加上 type="text/babel" 。
    // 创建虚拟DOM元素
    const vDom = <h1>Hello React</h1> // 千万不要加引号
    // 渲染虚拟DOM到页面真实DOM容器中
    ReactDOM.render(vDom, document.getElementById('test'))
  </script>
</body>
</html>
```

运行结果如下：

![](https://img-blog.csdnimg.cn/20201117174103335.png)

## 1.3 React JSX

### 1.3.1 JSX

JSX全称`JavaScript XML`，它是React 定义的一种类似于 XML 的 JS 扩展语法（XML + JS）。它**主要用来创建 React 虚拟 DOM**（元素）对象。例如`var vDom = <h1>Hello JSX!</h1>`。

* 它即不是字符串，也不是 HTML/XML 标签
* 它最终产生的就是一个 JS 对象

基本语法规则：

* 遇到 `<` 开头的代码，以标签的语法解析：html 同名标签转换为 html 同名元素，其它标签需要特别解析。
* 遇到 `{`开头的代码，以 JS 语法解析：**标签中的 js 代码必须用 {} 包含**。

 也就是说，`js代码`中可以直接嵌套`<标签>`，但`<标签>`要嵌套`js代码` 需要放在 {} 中。

### 1.3.2 虚拟DOM

创建虚拟DOM（特别的 js 对象）的两种方式：

（1）React 提供的 API 来创建（纯 JS，一般不用）

```js
  <script type="text/javascript">
    // 1. 纯JS创建虚拟DOM元素对象
    const msg1 = 'I Like You!'
    const myId1 = 'kai'
    const vDOM1 = React.createElement('h2', {id: myId1}, msg1.toUpperCase())
    // 2. 渲染到真实页面
	ReactDOM.render(vDOM2, document.getElementById('test1'))
  </script>
```

（2）JSX 语法（需要 babel 转换为 js）

```js
  <script type="text/babel">
    // 1. JSX创建虚拟DOM元素对象
    const msg1 = 'I Like You!'
    const myId1 = 'kai'  
    const vDOM2 = <h3 id={myId2.toUpperCase()}>{msg2.toLowerCase()}</h3>
    // 2. 渲染到真实页面
    ReactDOM.render(vDOM2, document.getElementById('test2'))
  </script>
```

* 虚拟 DOM 对象最终都会被 React 转换为真实的 DOM。

* 使用虚拟DOM能**提高开发效率**。编码时基本只需要操作 React 的虚拟 DOM 的相关数据，React将其转换为真实DOM从而更新界面。（因为虚拟 DOM 很“轻”，而真实 DOM 很“重”；真实 DOM 改变会重绘，而虚拟 DOM 变化不会更新界面，只有在渲染后才更新）

详细知识可以参考[【React深入】深入分析虚拟DOM的渲染原理和特性](https://segmentfault.com/a/1190000018891454)一文。

### 1.3.3 渲染虚拟 DOM(元素)

语法：`ReactDOM.render(virtualDOM, containerDOM)`

作用：将虚拟 DOM 元素渲染到页面中的真实容器 DOM 中并显示

参数说明：

* `virtualDOM`：纯 js 或 jsx 创建的虚拟 DOM 对象

* `containerDOM`：用来包含虚拟 DOM 元素的真实 DOM 元素对象(一般是一个 div)

## 1.4 模块与组件、模块化与组件化的理解

**模块**

* 理解：向外提供特定功能的 js 程序，一般就是一个 js 文件

* 作用：复用 js，简化 js 的编写，提高 js 运行效率

**组件**

* 理解：用来实现特定（局部）功能效果的代码集合（包含 html/css/js/图片 等）

* 作用：复用编码，简化项目编码，提高运行效率

**模块化**

当应用的 js 都以模块来编写的，这个应用就是一个模块化的应用

**组件化**

当应用是以多组件的方式实现，这个应用就是一个组件化的应用

# 2 React 面向组件编程

## 2.1 自定义组件

**定义组件**的两种方式

```js
    /*方式1: 函数组件(简单组件)*/
    function MyComponent () {
      return <h2>JavaScript 函数组件(简单组件)</h2>
    }
    /*方式2: ES6类组件(复杂组件)*/
    class MyComponent2 extends React.Component {
      render () {
        console.log(this) // MyComponent2的实例对象
        return <h2>ES6类组件(复杂组件)</h2>
      }
    }
```

**渲染组件**标签

```js
    ReactDOM.render(<MyComponent />, document.getElementById('example1'))
    ReactDOM.render(<MyComponent2 />, document.getElementById('example2'))
```

注意：
* 组件名必须首字母大写
* 虚拟 DOM 元素只能有一个根元素
* 虚拟 DOM 元素必须有结束标签

render() 渲染组件标签的基本流程：
* React 内部会创建组件实例对象
* 得到包含的虚拟 DOM 并解析为真实 DOM
* 插入到指定的页面元素内部

## 2.2 组件三大属性

### 2.2.1 state

* state 是组件对象最重要的属性，值是对象（可以包含多个数据）

* 组件被称为"状态机"，通过更新组件的 state 来更新对应的页面显示（重新渲染组件）

关于state状态的三个操作：

```js
// 1.初始化状态
constructor (props) {
  super(props)
  this.state = {
    stateProp1 : value1,
    stateProp2 : value2
  }
  // 将新增的方法中this强制绑定为组件对象（新添加的方法：内部this默认不是组件对象，而是undefined）
  this.handleClick = this.handleClick.bind(this) // bind返回一个新的处理过的函数
}
// 2.读取某个状态值
this.state.statePropertyName
// 3.更新状态 --> 组件界面更新
this.setState({
  stateProp1 : value1,
  stateProp2 : value2
})
```

### 2.2.2 props

* 每个组件对象都会有 props（properties）属性，组件标签的所有属性都保存在 props 中

* 通过标签属性从组件外向组件内传递变化的数据（组件内部不要修改 props 数据）

```js
// 1.内部读取某个属性值
this.props.propertyName
// 2.对props中的属性值进行类型限制和必要性限制
Person.propTypes = { // 使用 prop-types 库
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired
}
// 3.扩展属性：将对象的所有属性通过 props 传递
<Person {...person} />
// 4.默认属性
Person.defaultProps = {
  name: 'LiSi'
}
// 5.组件类的构造函数
constructor(props) {
  super(props)
  console.log(props) // 查看所有属性
}
```

组件的 props 和 state 属性区别：

* state：组件自身内部可变化的数据

* props：从组件外部向组件内部传递数据，组件内部只读不修改

### 2.2.3 refs

组件内的标签都可以定义 ref 属性来标识自己

* `<input type="text" ref={input => this.msgInput = input} />`

* ref 中的回调函数在组件初始化渲染完或卸载时自动调用（将 input 这个元素赋给组件实例对象的 this.msgInput）

在组件中可以通过 this.msgInput.value 来得到对应的真实 DOM 元素的值

作用：通过 ref 获取组件内容特定标签对象，进行读取其相关数据

### 2.2.4 事件处理

通过 onXxx 属性指定组件的事件处理函数（如：onClick、onBlur，注意需要大写）

* React 使用的是自定义（合成）事件，而不是使用的原生 DOM 事件

* React 中的事件是通过事件委托方式处理的（委托给组件最外层的元素）

通过 event.target 得到发生事件的 DOM 元素对象

```js
<input onFocus={this.handleFocus}/>

handleFocus(event) {
  event.target  //返回事件发生的input元素对象
}
```

**注意**

* 组件内置的方法中的 this 为组件对象

* 在组件类中自定义的方法中 this 为 null

  * 强制绑定 this：通过函数对象的 bind()
  * 箭头函数（ES6模块化编码时才能使用）

## 2.3 功能界面的组件化编码流程

第一步：拆分组件——拆分界面，抽取组件（有几个组件）

第二步：实现静态组件——使用组件实现静态页面效果（写 render）只有静态界面，没有动态数据和交互

第三步：实现动态组件
* 动态显示初始化数据（数据定义在哪一个组件中）
* 交互功能（从绑定事件监听开始）

## 2.4 收集表单数据

包含表单的组件分类 

* **受控组件**：表单项输入数据能自动收集成状态（onChange）

更贴近 react 思想，尽量少操作 DOM（一般更推荐这种写法）

```js
密码：<input type="password" value={this.state.pwd} onChange={this.handleChange} />
```

* **非受控组件**：需要时才手动读取表单输入框中的数据（ref）

写起来轻松，但是操作了原生 DOM（this.nameInput.value）

```js
用户名：<input type="text" ref={input => this.nameInput = input} />
```

## 2.5 组件生命周期

### 2.5.1 生命周期流程图

组件对象从创建到死亡会经历不同的生命周期阶段。React 组件对象包含一系列的**勾子函数**（生命周期回调函数），在生命周期特定时刻回调。在定义组件时，我们可以重写特定的生命周期回调函数，做特定的工作。

React 在 v16.3 之前，生命周期流程如下：

![](https://img-blog.csdnimg.cn/2020111815355123.png)

### 2.5.2 生命周期详述

1. 组件的三个生命周期状态

- Mount：挂载过程，第一次将组件插入到真实 DOM
- Update：更新过程，组件被重新渲染
- Unmount：卸载过程，被移出真实 DOM

2. **生命周期流程**：

**创建阶段（第一次初始化渲染显示）**

> ReactDOM.render()

- constructor()：`super(props)` 指定 this，`this.state={}` 创建初始化状态（getDefaultProps、getInitialState）

- componentWillMount()：组件将挂载到页面上

  - 可以在这里调用 setState() 方法修改 state

- render()：创建虚拟 DOM 但是还没有挂载上去

- componentDidMount()：已经挂载到页面上（初始界面已经渲染完毕）

  - 可以在这里通过 this.getDOMNode() 来进行访问 DOM 结构
- 可以在这里发送 ajax 请求
  - 添加监听器/订阅

**运行阶段（二次渲染）**

> 父组件传递的 `props` 发生更新，就会调用 componentWillReceiveProps()

- **componentWillReceiveProps(nextProps)**：当子组件接受到 nextProps 时，不管这个 props 与原来的是否相同都会调用

> `props` 改变或者调用 this.setState() 方法更新 `state`，都会触发组件的更新，调用后面的钩子函数

- shouldComponentUpdata(nextProps, nextState)：接收一个新的 props 和 state，返回true/false，表示是否允许更新

  - 通常情况下为了优化，需要对新的 props 以及 state 和原来的数据作对比，如果发生变化才更新

> 调用 this.forceUpdate() 方法会直接进入 componentWillUpdate。跳过 shouldComponentUpdate()

- **componentWillUpdate()**：将要更新
- render()：重新渲染
- **componentDidUpdate()**：已经完成更新

除了首次 render 之后调用 `componentDidMount`，其它 render 结束之后都是调用 `componentDidUpdate`

**销毁阶段（移除组件）**

> 执行 ReactDOM.unmountComponentAtNode(containerDom) 用来使组件从真实 DOM 中卸载（开始销毁阶段）

- componentWillUnmount()：组件将要被移除时（移出前）回调

  - 一般在 `componentDidMount` 里面注册的事件需要在这里删除

### 2.5.3 重要的勾子函数

* render()：初始化渲染时或更新渲染时调用

* componentDidMount()：开启监听，可以初始化一些异步操作：启动定时器/发送 ajax 请求

* componentWillUnmount()：做一些收尾工作，如：清理定时器

* componentWillReceiveProps()：当组件接收到（父元素传递的）新的 props 属性前调用

### 2.5.5 **新的生命周期**

`componentWillMount`、`componentWillReceiveProps` 和 `componentWillUpdate` 这三个生命周期函数都被添加了 UNSAFE_ 不安全标记，并且在 17.0 版本将会被删除。

> 由于 React 未来的版本中推出了异步渲染，在 `dom` 被挂载之前的阶段都可以被打断重来，导致 `componentWillMount`、`componentWillUpdate`、`componentWillReceiveProps` 在一次更新中可能会被触发多次，因此那些只希望触发一次的应该放在 `componentDidUpdate` 中。这也就是为什么要把异步请求放在 `componentDidMount` 中，而不是放在 `componentWillMount` 中的原因，为了向后兼容。

**新的生命周期流程图：**

![](https://img-blog.csdnimg.cn/20201118153504347.png)

**getDerivedStateFromProps**

功能：将 props 映射到 state 上面

触发时间：会在调用 render 方法之前调用（**每次渲染前都会触发**），并且在初始挂载及后续更新时都会被调用。

它是一个**静态**函数，所以函数体内不能访问 this，输出完全由输入的参数 nextProps 和 prevState 来决定，如果 props 传入的内容不需要影响到 state，那么就需要返回一个 null，这个返回值是必须的。

```js
static getDerivedStateFromProps(props, state) {
  if (props.currentRow !== state.lastRow) { 
    // 如果新的props的当前行大于之前的state的最后一行，就向下滚动
    return { // 返回的对象将被映射到state（state原来的属性和值还在）
      isScrollingDown: props.currentRow > state.lastRow,
      lastRow: props.currentRow,
    };
  }
  // 返回 null 表示无需更新 state。
  return null;
}
```

> 与 `componentDidUpdate` 一起，这个新的生命周期涵盖过时的 `componentWillReceiveProps` 的所有用例。

**getSnapshotBeforeUpdate**

功能：使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）

触发时间：在最近一次渲染输出（提交到 DOM 节点）之前调用

返回值将作为第三个参数传递给 `componentDidUpdate()`。在重新渲染过程中手动保留滚动位置等情况下非常有用。

> 与 `componentDidUpdate` 一起，这个新的生命周期涵盖过时的 `componentWillUpdate` 的所有用例。

## 2.6 虚拟 DOM 与 DOM Diff 算法

DOM Diff 能比较新旧虚拟 DOM 树，计算哪里改变，然后就只需要重绘变化的局部界面。

![](https://img-blog.csdnimg.cn/20201118223235467.png)

**DOM-diff**的过程：

1. 用JS对象模拟DOM（虚拟DOM）
2. 把此虚拟DOM转成真实DOM并插入页面中（render）
3. 如果有事件发生修改了虚拟DOM，比较两棵虚拟DOM树的差异，得到差异对象（diff）
4. 把差异对象应用到真正的DOM树上（patch）

# 3 react 脚手架

## 3.1 create-react-app

1. xxx 脚手架：用来帮助程序员快速创建一个基于 xxx 库的模板项目
   *  包含了所有需要的配置
   * 指定好了所有的依赖
   * 可以直接安装/编译/运行一个简单效果

2. react 提供了一个用于创建 react 项目的脚手架库：[create-react-app](https://github.com/facebook/create-react-app)

3. 项目的整体技术架构为：react + webpack + es6 + eslint

4. 使用脚手架开发的项目的特点：模块化、组件化、工程化

## 3.2 创建项目并启动

```shell
npm i -g create-react-app // 全局安装create-react-app脚手架
create-react-app hello-react // 创建一个react项目，项目名称是hello-react
cd hello-react
npm start // 启动项目
```

启动后页面：

![](https://img-blog.csdnimg.cn/20201118225032902.png)

# 4 react ajax

## 4.1 理解

**前置说明**

1.React 本身只关注于界面，并不包含发送 ajax 请求的代码

2.前端应用需要通过 ajax 请求与后台进行交互（json数据）

3.react 应用中需要集成第三方 ajax 库（或自己封装）

**常用的 ajax 请求库**

1.jQuery：比较重，如果需要另外引入不建议使用

2.axios：轻量级，建议使用

 	a. 封装了 XmlHttpRequest 对象的 ajax

 	b. 是 promise 风格

​	 c. 既可以用在浏览器端又可以用在 node 服务器端

3.fetch：原生函数，但老版本浏览器不支持

 	a. 不再使用 XmlHttpRequest 对象提交 ajax 请求

 	b. 为了兼容低版本的浏览器，可以引入兼容库 fetch.js

## 4.2 axios

**GET 请求**

```
axios.get('/user?ID=12345')
  .then(response => {
  console.log(response)
})
  .catch(error => {
  console.log(error)
})

axios.get('/user', {
  prams: {
    ID: 12345
  }
})
  .then(response => {
  console.log(response)
})
  .catch(error => {
  console.log(error)
}))
```

**POST 请求**

```
axios.post('/user', {
  firstName: 'Fred',
  lastName: 'Flintstone'
})
  .then(response => {
  console.log(response)
})
  .catch(error => {
  console.log(error)
})))
```

## 4.3 Fetch

**GET 请求**

```
fetch(url)
  .then(response => {
    return response.json()
}).then(data => {
    console.log(data)
}).catch(error => {
    console.log(error)
})
```

**POST 请求**

```
fetch(url, {
  method: 'POST',
  body: JSON.stringify(data)
}).then(data => {
    console.log(data)
}).catch(error => {
    console.log(error)
})
```

# 5 几个重要技术总结

## 5.1 组件间通信

### 5.1.1 方式一：通过 props 传递

1.共同的数据放在父组件上，特有的数据放在自己组件内部（state）

2.通过 props 可以传递一般数据和函数数据，只能一层一层传递

3.一般数据 --> 父组件传递数据给子组件 --> 子组件读取数据

4.函数数据 --> 子组件传递数据给父组件 --> 子组件调用函数

### 5.1.2 方式二：使用消息订阅-发布机制（subscribe-publish）

1.工具库：PubSubJS

2.下载：npm install pubsub-js --save

3.使用：

```
import PubSub from 'pubsub-js' //引入

PubSub.subscribe('delete', function(data){ }); //订阅消息，绑定监听
PubSub.publish('delete', data) //发布消息，触发事件
```

### 5.1.3 方式三：redux

在八、redux中

## 5.2 事件监听理解

### 5.2.1 原生 DOM 事件

1.绑定事件监听

 a. 事件名(类型)：只有有限的几个，不能随便写

 b. 回调函数

2.触发事件

 a. 用户操作界面

 b. 事件名(类型)

 c. 数据()

### 5.2.2 自定义事件(消息机制)

1.绑定事件监听

 a. 事件名(类型)：任意

 b. 回调函数：通过形参接收数据，在函数体处理事件

2.触发事件(编码)

 a. 事件名(类型)：与绑定的事件监听的事件名一致

 b. 数据：会自动传递给回调函数

# 6 react-router4

## 6.1 理解

### 6.1.1 react-router

1.react 的一个插件库

2.专门用来实现一个 SPA 应用

3.基于 react 的项目基本都会用到此库

### 6.1.2 SPA

1.单页 Web 应用（single page web application，SPA）

2.整个应用只有一个完整的页面

3.点击页面中的链接**不会刷新页面，也不会向服务器发请求**（会更新不同的组件）

4.当点击路由链接时，只会做页面的局部更新

5.数据都需要通过 ajax 请求获取，并在前端异步展现

### 6.1.3 路由

1.**什么是路由?**

 a. 一个路由就是一个映射关系（key:value）

 b. key 为路由路径，value 可能是 function/component

2.**路由分类**

 a. 后台路由：node 服务器端路由，value 是 function，用来处理客户端提交的请求并返回一个响应数据

 b. 前台路由：浏览器端路由，value 是 component，当请求的是路由 path 时，浏览器端没有发送 http 请求，但界面会更新显示对应的组件

3.**后台路由**

 a. 注册路由：router.get(path, function(req, res))，即路由器

 b. 当 node 接收到一个请求时，根据请求路径找到匹配的路由，调用路由中的函数来处理请求，返回响应数据

4.**前端路由**

 a. 注册路由：`<Route path="/about" component={About}>`

 b. 当浏览器的 hash 变为 #about 时，当前路由组件就会变为 About 组件

#### 6.1.4 前端路由的实现

1.history 库

 a. 管理浏览器会话历史（history）的工具库

 b. 包装的是原生 BOM 中的 window.history 和 window.location.hash

2.history API

- History.createBrowserHistory()：得到封装 window.history 的管理对象
- History.createHashHistory()：得到封装 window.location.hash 的管理对象
- history.push()：添加一个新的历史记录
- history.replace()：用一个新的历史记录替换当前的记录
- history.goBack()：回退到上一个历史记录
- history.goForword()：前进到下一个历史记录
- history.listen(function(location){})：监视历史记录的变化

## 6.2 react-router 相关 API

**组件**

路由器标签：

- `<BrowserRouter>`
- `<HashRouter>`：带#号

路由标签：

- `<Switch>`：包裹住`<Route>`，用来切换多个路由
- `<Route>`
- `<Redirect>`：自动跳转重定向，用来直接选中某个路由

路由链接：

- `<Link>`
- `<NavLink>`：比`<Link>`多了一个 class，选中有 active 效果

**其他**

1.this.props 中的

- match 对象：.match.params，通过路由参数向路由组件传递数据
- history 对象：.history，push()/replace()/goBack()/goForward()

2.withRouter 函数：用 this.props.history.push('/detail') 去跳转页面，但是报 this.props.history 错误 undefined，请在此组件中使用 withRouter 将 history 传入到 props 上。

## 6.3 路由使用

1.下载适用于 web 的 react-router：:`npm install --save react-router-dom`

2.流程

1. 编写**路由组件**
2. 在父路由组件中指定：
   - **路由链接**：`<NavLink>`
   - **路由**：`<Route>`
3. 嵌套路由：`path='/home/news'`

## 6.4 向路由组件传递参数数据

不能用 props 传递数据，因为加载组件用的不是标签的形式，而是：

```
<Route path='/home/message' component={Message}/>
// message.jsx
// 路由链接
<Link to={`/home/message/messagedetail/${m.id}`}>{m.title}</Link>
// 路由
<Route path={`/home/message/messagedetail/:id`} component={MessageDetail}/>
// message-detail.jsx
const {id} = this.props.match.params
```

这样就通过路由参数传递了数据，可以在 this.props.match.params 中获取

# 7 react-ui

## 7.1 最流行的开源 React UI 组件库

### 7.1.1 material-ui(国外)

1.官网：http://www.material-ui.com/#/

2.github：https://github.com/callemall/material-ui

### 7.1.2 ant-design(国内蚂蚁金服)

1.PC官网：https://ant.design/index-cn

2.移动官网：https://mobile.ant.design/index-cn

3.Github：https://github.com/ant-design/ant-design/

4.Github：https://github.com/ant-design/ant-design-mobile/

## 7.2 ant-design-mobile 使用入门

**搭建 antd-mobile 的基本开发环境**

[基本使用](https://mobile.ant.design/docs/react/introduce-cn#1.-创建一个项目)

**实现按需打包(组件js/css)**

[按需加载](https://mobile.ant.design/docs/react/use-with-create-react-app-cn#按需加载)

# 8 redux

## 8.1 理解

### 8.1.1 redux 是什么？

1. redux 是一个独立专门用于做状态管理的 JS 库（不是 react 插件库）
2. 它可以用在 react，angular，vue 等项目中，但基本与 react 配合使用
3. 作用：集中式管理 react 应用中多个组件共享的状态

### 8.1.2 redux 工作流程

![image-20200826085826481](https://img-blog.csdnimg.cn/20200830142119865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTA4ODMy,size_16,color_FFFFFF,t_70#pic_center)

### 8.1.3 什么情况下需要使用 redux

1. 总体原则：能不用就不用，如果不用比较吃力才考虑使用
2. 某个组件的状态，需要共享
3. 某个状态需要在任何地方都可以拿到
4. 一个组件需要改变全局状态
5. 一个组件需要改变另一个组件的状态

## 8.2 redux 的核心 API

### 8.2.1 createStore

1. 作用：创建包含指定 reducer 的 store 对象
2. 编码 store.js：

```
import {createStore} from 'redux'
import counter from './reducers/counter'

const store = createStore(counter)
```

### 8.2.2 store 对象

1. 作用：redux 库最核心的管理对象
2. 它内部维护着：state、reducer
3. 核心方法：getState()，dispatch()，subscribe(listener)
4. 编码 jsx：

```
store.getState() // 得到store中存储的state数据
store.dispatch({type: 'INCREMENT', data: number}) // 分发action对象，通知reducer更新state数据
store.subscribe(render) // 订阅监听，store中的状态变化就会调用进行重绘
```

### 8.2.3 applyMiddleware()

1.作用：应用上基于 redux 的中间件（插件库）

2.编码 store.js：

```
import {createStore, applyMiddleware} from 'redux'
import thunk from 'redux-thunk'  // redux异步中间件

const store = createStore(
  counter,
  applyMiddleware(thunk) // 应用上异步中间件
)
```

### 8.2.4 combineReducers()

1.作用：合并多个 reducer 函数

2.编码 reduces.js：

```
export default combineReducers({
  user,
  chatUser,
  chat
})
```

## 8.3 redux 的三个核心概念

### 8.3.1 action

1. 标识要执行行为的对象（**只是描述了有事情要发生，并没有描述如何去更新 state**）
2. 包含2个方面的属性：
   - type：标识属性，值为字符串，唯一，必要属性
   - data：数据属性，值类型任意，可选属性
3. 例子：

```
const action = {
  type: 'INCREMENT',
  data: 2
}
```

1. **Action Creator（创建 action 的工厂函数）**

```
export const increment = (number) => ({type: 'INCREMENT', data: number})
```

### 8.3.2 reducer

1. **根据老的 state 和 action，产生新的 state** 的**纯函数**
2. 例子：

```
export default function counter(state=0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + action.data
    case 'DECREMENT':
      return state - action.data
    default:
      return state
  }
}
```

1. 注意
   - **返回一个新的状态 state 给 store**
   - **不要修改原来的状态**

### 8.3.3 store

1. **将 state，action 与 reducer 联系在一起的对象**
2. 如何得到此对象？

```
import {createStore} from 'redux'
import reducer from './reducers'

const store = createStore(reducer)
```

1. 此对象的功能？
   - getState()：得到 state
   - dispatch(action)：分发 action，触发 reducer 调用，产生新的 state
   - subscribe(listener)：注册监听，当产生了新的 state 时，自动调用

**问题**

1.redux 与 react 组件的代码耦合度太高

2.编码不够简洁（经常重复写 this.props.store）

## 8.4 react-redux

### 8.4.1 理解

1. 一个 react 插件库
2. 专门用来简化 react 应用中使用的 redux

### 8.4.2 React-Redux 将所有组件分成两大类

1.UI 组件

- 只负责 UI 的呈现，不带有任何业务逻辑
- 通过 props 接收数据（一般数据和函数）
- **不使用任何 Redux 的 API**
- **一般保存在 components 文件夹下**

2.容器组件

- 负责管理数据和业务逻辑，不负责 UI 的呈现
- **使用 Redux 的 API**
- **一般保存在 containers 文件夹下**

### 8.4.3 相关 API

1.**Provider：让所有组件都可以得到 state 数据**

```
import {Provider} from 'react-redux'

<Provider store={store}>
  <App />
</Provider>
```

2.**connect()：用于包装 UI 组件生成容器组件**

将 UI 组件与 Redux 关联起来生成一个容器组件，为了向 UI 组件中传递 props 属性。

Provider 内部的组件想要获取到 redux 中的数据和方法，就必须要用 connect 进行一层包裹封装。内部轻松获得 state 的数据，并为 actionCreator 的函数调用 dispatch。

```
import {connect} from 'react-redux'

// 用connect包装Counter组件再返回出去
export default connect(
  mapStateToprops, // 是个回调函数，将状态映射成属性，返回对象
  mapDispatchToProps // 是个对象，包含actions中的方法（将在内部被转换成调用dispatch的函数）
)(Counter)
```

3.mapStateToprops()：将保存在 redux 中的数据（即 state 对象）转换为 UI 组件的标签属性props

```
const mapStateToprops = function (state) {
  return {
    value: state
  }
}
```

4.mapDispatchToProps()：将分发 action 的函数转换为 UI 组件的标签属性 props

可以直接指定为 actions 对象或包含多个 action 方法的对象

```
// 包含多个 action 方法的对象，mapDispatchToProps即{increment, decrement}
import {increment, decrement} from '../redux/actions'
// 或 指定为 actions 对象
import * as mapDispatchToProps from '../redux/actions'
```

**问题**

1.redux 默认是**不能进行异步处理的**

2.应用中又需要在 redux 中执行异步任务（ajax，定时器）

## 8.5 redux 异步编程

下载 redux 插件（异步中间件）：npm install --save redux-thunk

store.js 中应用

```
import {createStore, applyMiddleware} from 'redux'
import thunk from 'redux-thunk'

const store = createStore(
  counter,
  applyMiddleware(thunk) // 应用异步中间件
)
```

actions.js 中添加异步 action

```
export const incrementAsync = (number) => {
  return dispatch => { // 异步action会返回一个函数
    // 异步的代码必须被封装到action中
    setTimeout(() => {
      // 1s后才去分发一个同步的action（dispatch一个同步action）
      dispatch(increment(number))
    }, 1000)
  }
}
```

然后在 UI组件的 propTypes 和容器组件的 connect() 中添加对应的异步 action 即可。

### 8.6 redux 调试工具

安装 chrome 浏览器插件：redux-devtools

> 遇到的问题：下载 2.15.1 老版本后报错：*TypeError:* *Cannot* *read* *property* *'state'* *of* undefined，下载最新版本 2.17.0 后解决

要想能使用调试工具还需要在项目中下载工具依赖包：npm install --save-dev redux-devtools-extension

编码 store.js：

```
import { composeWithDevTools } from 'redux-devtools-extension'

const store = createStore(
  counter,
  composeWithDevTools(applyMiddleware(thunk))
)
```

## 8.7 纯函数和高阶函数

### 8.8.1 纯函数

1.一类特别的函数：只要是同样的输入，必定得到同样的输出

2.必须遵守以下一些约束

 a. 不得改写参数

 b. 不能调用系统 I/O 的 API

 c. 能调用 Date.now() 或者 Math.random() 等不纯的方法

3.reducer 函数必须是一个纯函数

### 8.8.2 高阶函数

1.理解：一类特别的函数

 a. 情况1：参数是函数

 b. 情况2：返回是函数

2.常见的高阶函数：

 a. 定时器设置函数

 b. 数组的 map()/filter()/reduce()/find()/bind()

 c. react-redux 中的 connect 函数

3.作用：能实现更加动态，更加可扩展的功能

# 参考资料

[React实战教程](https://www.bilibili.com/video/BV184411x7F9)

[React中文文档](https://zh-hans.reactjs.org/docs/getting-started.html)

[React 入门实例教程](https://www.ruanyifeng.com/blog/2015/03/react.html)

[图解ES6中的React生命周期](https://juejin.im/post/6844903510538977287)

[React 的生命周期变化](https://juejin.im/post/6844904005152276487)

[让虚拟DOM和DOM-diff不再成为你的绊脚石](https://juejin.im/post/6844903806132568072)