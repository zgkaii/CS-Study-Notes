# 1. 最流行的开源React UI组件库
## 1). material-ui(国外)
	官网: http://www.material-ui.com/#/
	github: https://github.com/callemall/material-ui
## 2). ant-design(国内蚂蚁金服)
	官网: https://ant.design/
	github: https://github.com/ant-design/ant-design/

# 2. ant-design使用入门
## 1). 使用create-react-app搭建react开发环境
	npm install create-react-app -g
	create-react-app antd-demo
	cd antd-demo
	npm start
## 2). 搭建antd的基本开发环境
	1. 下载
    	npm install antd@2.7.4 --save
	2. src/App.js
	    import React, { Component } from 'react';
	    import { Button } from 'antd';
	    import './App.css';
	    
	    class App extends Component {
	      render() {
	        return (
	          <div className="app">
	            <Button type="primary">Button</Button>
	          </div>
	        );
	      }
	    }
    	export default App;
	3. src/App.css
	    @import '~antd/dist/antd.css';
	    
	    .app {
	      text-align: center;
	    }

## 3). 实现按需加载(组件js/组件css)
	1. 使用eject命令将所有内建的配置暴露出来
    	npm run eject
	2. 下载babel-plugin-import(用于按需加载组件代码和样式的 babel 插件)
    	npm install babel-plugin-import --save-dev
	3. 修改配置: config/webpack.config.dev.js
	    // Process JS with Babel.
	    {
	      test: /\.(js|jsx)$/,
	      include: paths.appSrc,
	      loader: 'babel',
	      options: {
          +   plugins: [
          +     ['import', { libraryName: 'antd', style: 'css' }],
          +   ],
              // This is a feature of `babel-loader` for webpack (not Babel itself).
              // It enables caching results in ./node_modules/.cache/babel-loader/
              // directory for faster rebuilds.
              cacheDirectory: true
            }
		 },
	4. 去除引入全量样式的语句: src/App.css
	    @import '~antd/dist/antd.css' 
