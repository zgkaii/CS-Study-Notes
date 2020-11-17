
# 第一章 React入门
## 1.1 React基础认识

用于构建用户界面的 JavaScript 库（只关注于 View），由 Facebook 开源。

**特点**

1. Declarative（声明式编码）
2. Component-Based（组件化编码）
3. Learn Once，Write Anywhere（支持客户端与服务器渲染，React-Native）
4. 高效
5. 单向数据流

**高效的原因**

1. 虚拟（virtual）DOM，不总是操作 DOM
2. DOM Diff 算法，最小化页面重绘

## 1.2 React基本使用

**相关 js 库**

1. **react.min.js** ——React 的核心库。
2. **react-dom.min.js** ——提供与 DOM 相关的功能。
3. **babel.min.js** ——Babel 可以将 ES6 代码转为 ES5 代码，这样我们就能在目前不支持 ES6 浏览器上执行 React 代码。Babel 内嵌了对 **JSX** 的支持。通过将 Babel 和 babel-sublime 包（package）一同使用可以让源码的语法渲染上升到一个全新的水平。

**使用 React 开发者工具调试**

* React DeveloperTools 浏览器插件。

基本案例：

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

### 1.3.1 虚拟DOM

* 创建虚拟DOM（特别的 js 对象）的两种方式：

1. React 提供的 API 来创建（纯 JS，一般不用）

```js
  <script type="text/javascript">
    // 1. 创建虚拟DOM
    const msg = 'I Like You!'
    const myId = 'Kai'
    /*方式一: 纯JS创建虚拟DOM元素对象*/
    const vDOM1 = React.createElement('h2', {id: myId}, msg.toUpperCase())
    // 2. 渲染到真实页面
	ReactDOM.render(vDOM2, document.getElementById('test1'))
  </script>
```

2. JSX 语法（需要 babel 转换为 js）

```js
  <script type="text/babel">
    // 1. 创建虚拟DOM
    /*方式二: JSX创建虚拟DOM元素对象*/
    const vDOM2 = <h3 id={myId.toUpperCase()}>{msg.toLowerCase()}</h3>
    // 2. 渲染到真实页面
    ReactDOM.render(vDOM2, document.getElementById('test2'))
  </script>
```

* 虚拟 DOM 对象最终都会被 React 转换为真实的 DOM。

* 使用虚拟DOM能**提高开发效率**。编码时基本只需要操作 React 的虚拟 DOM 的相关数据，React将其转换为真实DOM从而更新界面（因为虚拟 DOM 很“轻”，而真实 DOM 很“重”；真实 DOM 改变会重绘，而虚拟 DOM 变化不会更新界面，只有在渲染后才更新）

详细知识可以参考[【React深入】深入分析虚拟DOM的渲染原理和特性](https://segmentfault.com/a/1190000018891454)一文。

> 在 Web 开发中，我们总需要将变化的数据实时反应到 UI 上，这时就需要对 DOM 进行操作。而复杂或频繁的DOM操作通常是性能瓶颈产生的原因，React 为此引入了虚拟 DOM（Virtual DOM）的机制：
>
> 在浏览器端用 JS 实现了一套 DOM API。基于 React 进行开发时所有的 DOM 构造都是通过虚拟 DOM 进行，每当数据变化时，React 都会重新构建整个 DOM 树，然后 React 将当前整个 DOM 树和上一次的 DOM 树进行对比，得到 DOM 结构的区别，然后仅仅将需要变化的部分进行实际的浏览器 DOM 更新。而且 React 能够批处理虚拟 DOM 的刷新，在一个事件循环（Event Loop）内的两次数据变化会被合并，例如你连续的先将节点内容从 A 变成 B，然后又从 B 变成 A，React 会认为 UI 不发生任何变化，而如果通过手动控制，这种逻辑通常是极其复杂的。尽管每一次都需要构造完整的虚拟 DOM 树，但是因为虚拟 DOM 是内存数据，性能是极高的（很“轻”），而对实际 DOM 进行操作的仅仅是 Diff 部分，因而能达到提高性能的目的。
>
> 这样，在保证性能的同时，开发者将不再需要关注某个数据的变化如何更新到一个或多个具体的 DOM 元素，而只需要关心在任意一个数据状态下，整个界面是如何 Render 的。

### 1.3.2 JSX

JSX全称: JavaScript XML，它是React 定义的一种类似于 XML 的 JS 扩展语法 XML + JS。

作用：用来创建 React 虚拟 DOM（元素）对象

* `var vDom = <h1>Hello JSX!</h1>`
* 注意1：它不是字符串，也不是 HTML/XML 标签
* 注意2：它最终产生的就是一个 JS 对象

标签名任意：HTML 标签或其它标签（可以自定义）

标签属性任意：HTML 标签属性或其它（可以自定义）

基本语法规则

* 遇到 '<' 开头的代码，以标签的语法解析：html 同名标签转换为 html 同名元素，其它标签需要特别解析
* 遇到 '{' 开头的代码，以 JS 语法解析：**标签中的 js 代码必须用 {} 包含**

 也就是说：js 中可以直接嵌套<标签>，但标签要嵌套 js 需要放在 {} 中

babel.js 的作用

* 浏览器不能直接解析 JSX 代码，需要 babel 转译为纯 JS 的代码才能运行
* 只要用了 JSX，都要在 script 标签中加上 `type="text/babel"` 来声明需要 babel 来处理

### 1.3.3 渲染虚拟 DOM(元素)

1.语法：`ReactDOM.render(virtualDOM, containerDOM)`

2.作用：将虚拟 DOM 元素渲染到页面中的真实容器 DOM 中显示

3.参数说明

* 参数一：纯 js 或 jsx 创建的虚拟 dom 对象
* 参数二：用来包含虚拟 DOM 元素的真实 dom 元素对象(一般是一个 div)

## 1.4 模块与组件、模块化与组件化的理解

**模块**

1.理解：向外提供特定功能的 js 程序，一般就是一个 js 文件

2.为什么：js 代码很多很复杂

3.作用：复用 js，简化 js 的编写，提高 js 运行效率

**组件**

1.理解：用来实现特定（局部）功能效果的代码集合（包含 html/css/js/图片 等）

2.为什么：一个界面的功能很复杂

3.作用：复用编码，简化项目编码，提高运行效率

**模块化**

当应用的 js 都以模块来编写的，这个应用就是一个模块化的应用

**组件化**

当应用是以多组件的方式实现，这个应用就是一个组件化的应用

## 二、React 面向组件编程

### 2.1 自定义组件

1.定义组件的两种方式

```
// 方式1：工厂函数组件（是简单组件，没有state状态）
function MyComponent() {
  return <h2>工厂函数组件（简单组件）</h2>
}
// 方式2：ES6类组件（是复杂组件，可以有state）
class MyComponent2 extends React.Component {
  render() {
    console.log(this) // MyComponent2的实例对象
    return <h2>ES6类组件（复杂组件）</h2>
  }
}
```

2.渲染组件标签

```
ReactDOM.render(<MyComponent />, document.getElementById('example1'))
```

3.注意：

- 组件名必须首字母大写
- 虚拟 DOM 元素只能有一个根元素
- 虚拟 DOM 元素必须有结束标签

4.render() 渲染组件标签的基本流程：

1. React 内部会创建组件实例对象
2. 得到包含的虚拟 DOM 并解析为真实 DOM
3. 插入到指定的页面元素内部

### 2.2 组件三大属性

#### 2.2.1 state

**理解**

1.state 是组件对象最重要的属性，值是对象（可以包含多个数据）

2.组件被称为"状态机"，通过更新组件的 state 来更新对应的页面显示（重新渲染组件）

**编码**

```
1.初始化状态
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

#### 2.2.2 props

**理解**

1.每个组件对象都会有 props（properties）属性

2.组件标签的所有属性都保存在 props 中

**作用**

1.通过标签属性从组件外向组件内传递变化的数据

2.注意：组件内部不要修改 props 数据

**编码**

```
// 1.内部读取某个属性值
this.props.propertyName
// 2.对 props 中的属性值进行类型限制和必要性限制
Person.propTypes = { // 使用 prop-types 库
  name: PropTypes.string.isRequired,
  age: PropTypes.number
}
// 3.扩展属性：将对象的所有属性通过 props 传递
<Person {...person} />
// 4.默认属性
Person.defaultProps = {
  name: 'Mary'
}
// 5.组件类的构造函数
constructor(props) {
  super(props)
  console.log(props) // 查看所有属性
}
```

**问题**

请区别一下组件的 props 和 state 属性

1.state：组件自身内部可变化的数据

2.props：从组件外部向组件内部传递数据，组件内部只读不修改

#### 2.2.3 refs

1.组件内的标签都可以定义 ref 属性来标识自己

 a. `<input type="text" ref={input => this.msgInput = input} />`

 b. ref 中的回调函数在组件初始化渲染完或卸载时自动调用（将 input 这个元素赋给组件实例对象的 this.msgInput）

2.在组件中可以通过 this.msgInput.value 来得到对应的真实 DOM 元素的值

3.作用：通过 ref 获取组件内容特定标签对象，进行读取其相关数据

#### 2.2.4 事件处理

1.通过 onXxx 属性指定组件的事件处理函数（如：onClick、onBlur，注意需要大写）

 a. React 使用的是自定义（合成）事件，而不是使用的原生 DOM 事件

 b. React 中的事件是通过事件委托方式处理的（委托给组件最外层的元素）

2.通过 event.target 得到发生事件的 DOM 元素对象

```
<input onFocus={this.handleFocus}/>

handleFocus(event) {
  event.target  //返回事件发生的input元素对象
}
```

**注意**

1.组件内置的方法中的 this 为组件对象

2.在组件类中自定义的方法中 this 为 null

 a. 强制绑定 this：通过函数对象的 bind()

 b. 箭头函数（ES6模块化编码时才能使用）

### 2.3 组件的组合

#### 2.3.1 功能界面的组件化编码流程（无比重要）

1.拆分组件：拆分界面，抽取组件（有几个组件）

2.实现静态组件：使用组件实现静态页面效果（写 render）只有静态界面，没有动态数据和交互

3.实现动态组件

 a. 动态显示初始化数据（数据定义在哪一个组件中）

 b. 交互功能（从绑定事件监听开始）

### 2.4 收集表单数据

**理解**

1.问题：在 react 应用中，如何收集表单输入数据

2.包含表单的组件分类

a. **受控组件**：表单项输入数据能自动收集成状态（onChange）

更贴近 react 思想，尽量少操作 DOM（一般更推荐这种写法）

```
密码：<input type="password" value={this.state.pwd} onChange={this.handleChange} />
```

b. **非受控组件**：需要时才手动读取表单输入框中的数据（ref）

写起来轻松，但是操作了原生 DOM（this.nameInput.value）

```
用户名：<input type="text" ref={input => this.nameInput = input} />
```

### 2.5 组件生命周期

#### 2.5.1 理解

1.组件对象从创建到死亡它会经历特定的生命周期阶段

2.React 组件对象包含一系列的**钩子函数**（生命周期回调函数），在生命周期特定时刻回调

3.我们在定义组件时，可以重写特定的生命周期回调函数，做特定的工作

#### 2.5.2 生命周期流程图

![image-20200823105645557](https://img-blog.csdnimg.cn/20200830142058207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTA4ODMy,size_16,color_FFFFFF,t_70#pic_center)

#### 2.5.3 生命周期详述

1.组件的三个生命周期状态：

- Mount：挂载过程，第一次将组件插入到真实 DOM
- Update：更新过程，组件被重新渲染
- Unmount：卸载过程，被移出真实 DOM

2.**生命周期流程**：

1）创建阶段（第一次初始化渲染显示）

> ReactDOM.render()

- constructor()：`super(props)` 指定 this，`this.state={}` 创建初始化状态（getDefaultProps、getInitialState）

- componentWillMount()

  ：组件将要挂载到页面上

  - 可以在这里调用 setState() 方法修改 state

- render()：创建虚拟 DOM 但是还没有挂载上去

- componentDidMount()

  ：已经挂载到页面上（初始界面已经渲染完毕）

  - 可以在这里通过 this.getDOMNode() 来进行访问 DOM 结构
  - 可以在这里发送 ajax 请求
  - 添加监听器/订阅

2）运行阶段（二次渲染）

> 父组件传递的 `props` 发生更新，就会调用 componentWillReceiveProps()

- **componentWillReceiveProps(nextProps)**：当子组件接受到 nextProps 时，不管这个 props 与原来的是否相同都会调用

> `props` 改变或者调用 this.setState() 方法更新 `state`，都会触发组件的更新，调用后面的钩子函数

- shouldComponentUpdata(nextProps, nextState)

  ：接收一个新的 props 和 state，返回true/false，表示是否允许更新

  - 通常情况下为了优化，需要对新的 props 以及 state 和原来的数据作对比，如果发生变化才更新

> 调用 this.forceUpdate() 方法会直接进入 componentWillUpdate。跳过 shouldComponentUpdate()

- **componentWillUpdate()**：将要更新
- render()：重新渲染
- **componentDidUpdate()**：已经完成更新

除了首次 render 之后调用 `componentDidMount`，其它 render 结束之后都是调用 `componentDidUpdate`

3）销毁阶段（移除组件）

> 执行 ReactDOM.unmountComponentAtNode(containerDom) 用来使组件从真实 DOM 中卸载（开始销毁阶段）

- componentWillUnmount()

   

  ：组件将要被移除时（移出前）回调

  - 一般在 `componentDidMount` 里面注册的事件需要在这里删除

#### 2.5.4 重要的勾子

1. render()：初始化渲染时或更新渲染时调用
2. componentDidMount()：开启监听，可以初始化一些异步操作：启动定时器/发送 ajax 请求
3. componentWillUnmount()：做一些收尾工作，如：清理定时器
4. componentWillReceiveProps()：当组件接收到（父元素传递的）新的 props 属性前调用

#### 2.5.5 **新的生命周期**

**注意**

`componentWillMount`、`componentWillReceiveProps` 和 `componentWillUpdate` 这三个生命周期函数都被添加了 UNSAFE_ 不安全标记，并且在 17.0 版本将会被删除。

> 由于 React 未来的版本中推出了异步渲染，在 `dom` 被挂载之前的阶段都可以被打断重来，导致 `componentWillMount`、`componentWillUpdate`、`componentWillReceiveProps` 在一次更新中可能会被触发多次，因此那些只希望触发一次的应该放在 `componentDidUpdate` 中。这也就是为什么要把异步请求放在 `componentDidMount` 中，而不是放在 `componentWillMount` 中的原因，为了向后兼容。

**目前新的生命周期流程图：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830142058322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTA4ODMy,size_16,color_FFFFFF,t_70#pic_center)

**getDerivedStateFromProps**

功能：将 props 映射到 state 上面

触发时间：会在调用 render 方法之前调用（**每次渲染前都会触发**），并且在初始挂载及后续更新时都会被调用。

它是一个**静态**函数，所以函数体内不能访问 this，输出完全由输入的参数 nextProps 和 prevState 来决定，如果 props 传入的内容不需要影响到 state，那么就需要返回一个 null，这个返回值是必须的。

```
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

### 2.6 虚拟 DOM 与 DOM Diff 算法

#### 2.6.1 基本原理图

![image-20200823152235286](https://img-blog.csdnimg.cn/20200830142058100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTA4ODMy,size_16,color_FFFFFF,t_70#pic_center)

DOM Diff 能比较新旧虚拟 DOM 树，计算哪里改变，然后就只需要重绘变化的局部界面。

# 参考资料

[尚硅谷React实战教程全套完整版(轻松入门到精通)](https://www.bilibili.com/video/BV184411x7F9)

[React中文文档](https://react.docschina.org/)

http://www.woc12138.com/article/55

[BootCDN](https://www.bootcdn.cn/)

[React 入门实例教程](https://www.ruanyifeng.com/blog/2015/03/react.html)

https://www.html.cn/qa/react/15081.html