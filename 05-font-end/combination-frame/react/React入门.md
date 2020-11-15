# 第1章 React入门

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

1. **react.min.js** - React 的核心库。
2. **react-dom.min.js** - 提供与 DOM 相关的功能。
3. **babel.min.js** - Babel 可以将 ES6 代码转为 ES5 代码，这样我们就能在目前不支持 ES6 浏览器上执行 React 代码。Babel 内嵌了对 **JSX** 的支持。通过将 Babel 和 babel-sublime 包（package）一同使用可以让源码的语法渲染上升到一个全新的水平。

**使用 React 开发者工具调试**

React_DeveloperTools 浏览器插件。



# 参考

[尚硅谷React实战教程全套完整版(轻松入门到精通)](https://www.bilibili.com/video/BV1oW41157DY)

[React中文文档](https://react.docschina.org/)

http://www.woc12138.com/article/55

[BootCDN](https://www.bootcdn.cn/)