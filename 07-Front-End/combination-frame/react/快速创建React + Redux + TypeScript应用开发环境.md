# 1 create-react-app

react 提供了一个用于创建 react 项目的脚手架库：[create-react-app](https://github.com/facebook/create-react-app)。

[TypeScript](https://www.typescriptlang.org/) 是 JavaScript 的类型超集，可编译为纯 JavaScript ，要使用 TypeScript启动新的 Create React App 项目，可以利用：`--typescript`创建项目。

```shell
$ npx create-react-app my-react-app --typescript
或者 yarn create my-react-app --typescript
```

然后，将 TypeScript添加到 Create React App 项目：

```shell
$ npm install --save typescript @types/node @types/react @types/react-dom @types/jest
或者 yarn add typescript @types/node @types/react @types/react-dom @types/jest
```

接下来，将任何文件重命名为 TypeScript 文件（例如 `src/index.js` 重命名为 `src/index.tsx` ）并 **重新启动开发服务器**！

类型错误将显示在与构建错误相同的控制台中。

# 2 redux

下载`react-redux`：

```shell
$ npm install --save redux react-redux
```

安装react-redux类型定义文件

```shell
$ npm install -D @types/react-redux
```

## 2.1 typescript-fsa

编写Redux样板代码太迟钝，无法最快地完成。因此，让我们使用typescript-fsa。

```shell
$ npm install --save typescript-fsa typescript-fsa-reducers
```

## 2.2 Action実装

首先定义Action

src/actions/sampleAction.tsx

```react
import actionCreatorFactory from 'typescript-fsa';

const actionCreator = actionCreatorFactory();

export const sampleActions = {
  updateName: actionCreator<string>('ACTIONS_UPDATE_NAME'),
  updateEmail: actionCreator<string>('ACTIONS_UPDATE_EMAIL')
};
```

## 2.3 Reducer実装

接下来，定义一个Reducer以从该状态和当前状态生成一个新状态。

src/states/sampleState.tsx

```react
import { reducerWithInitialState } from 'typescript-fsa-reducers';
import { sampleActions } from '../actions/sampleAction';

export interface sampleState {
  name: string;
  email: string;
}

const initialState: sampleState = {
  name: '',
  email: ''
};

export const sampleReducer = reducerWithInitialState(initialState)
  .case(sampleActions.updateName, (state, name) => {
    return Object.assign({}, state, { name });
  })
  .case(sampleActions.updateEmail, (state, email) => {
    return Object.assign({}, state, { email });
  });
```

## 2.4 Store実装

次にすべてのStateを管理するstoreを定義します。

Redux 是一个可预测的 JavaScript 应用状态管理容器，这个**状态容器**就是这里的 Store。

将类似“ store.ts”的文件直接放在“ src”下，并使用“ createStore”定义商店。

store.ts

```react
import { createStore, combineReducers } from 'redux';
import { sampleReducer, sampleState } from './states/sampleState';

export type AppState = {
  sample: sampleState
};

const store = createStore(
  combineReducers<AppState>({
    sample: sampleReducer
  })
);

export default store;
```

# 3 Component implementation

这次，我们将以Presentational / Container组件[2]（https://qiita.com/IzumiSy/items/b7d8a96eacd2cd8ad510#fn2）的模式继续实施。

Presentatonal和Container将视图分为两层，以确保从责任角度考虑松散耦合。 具体而言，存在以下差异。

|                         |      Container       |     Presentational     |
| :---------------------: | :------------------: | :--------------------: |
|        **role**         | Implemented behavior | Implemented appearance |
|       **Status**        |         yes          |           no           |
| **Dependence on Redux** |         yes          |           no           |

也就是说，Container用Redux包装了依赖关系（状态，动作执行等），而Component只需要知道Container包装的行为。





## 3.1 Container実装

用src / containers剪切目录并像这样实现

src/containers/sampleContainer.tsx

```
import { Action } from 'typescript-fsa';
import { Dispatch } from 'redux';
import { connect } from 'react-redux';
import { AppState } from '../store';
import { sampleActions } from '../actions/sampleAction';
import { sampleComponent } from '../components/sampleComponent';

export interface sampleActions {
  updateName: (v: string) => Action<string>;
  updateEmail: (v: string) => Action<string>;
}

function mapDispatchToProps(dispatch: Dispatch<Action<string>>) {
  return {
    updateName: (v: string) => dispatch(sampleActions.updateName(v)),
    updateEmail: (v: string) => dispatch(sampleActions.updateEmail(v))
  };
}

function mapStateToProps(appState: AppState) {
  return Object.assign({}, appState.sample);
}

export default connect(mapStateToProps, mapDispatchToProps)(sampleComponent);

```

停止在mapDispatchToProps中使用`bindActionCreators`。 使用它与直接在组件中调用`dispatch`相同，并且没有必要放置Container层。 如果要这样做，则应使用Mobx等。

## 3.2 Component実装



接下来，定义一个用于显示数据的组件和一个用于注入特定回调函数的容器。
在这里，如果输入值，则只输出它。

src/components/sampleComponent.tsx

停止向Presentational组件提供状态。 逻辑应在容器端定义。

将代码剪切到诸如src / components之类的目录中，并将其放在此处。 通常，这是一个没有州**的地方，因此使用“ React.SFC”是适当的。

sampleComponent.tsx

```
import * as React from 'react';
import { sampleState } from '../states/sampleState';
import { sampleActions } from '../containers/sampleContainer';

interface OwnProps {}

type sampleProps = OwnProps & sampleState & sampleActions;

export const sampleComponent: React.SFC<sampleProps> = (props: sampleProps) => {
  return (
    <div>
      <div className="field">
        <input
          type="text"
          placeholder="name"
          value={props.name}
          onChange={(e) => props.updateName(e.target.value)}
        />
      </div>
      <div className="field">
        <input
          type="email"
          placeholder="email"
          value={props.email}
          onChange={(e) => props.updateEmail(e.target.value)}
        />
      </div>
    </div>
  );
};
```

## 3.3 `App.tsx`への追加

在src下有一个名为App.tsx的文件，因此我们在此处显示sampleComponent。

App.tsx

```
import * as React from 'react';
import sampleContainer from '../src/containers/sampleContainer';

class App extends React.Component {
  render() {
    return (
      <div className="App">
        <sampleContainer />
      </div>
    );
  }
}

export default App;
```

## 3.4 `index.tsx`の更新

src下的index.tsx是实际安装React根组件的地方。
最后，让我们重写它以在此处使用store。

index.tsx

```
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import Store from './store';
import App from './App';
import registerServiceWorker from './registerServiceWorker';
import './index.css';

ReactDOM.render(
  <Provider store={Store}>
    <App />
  </Provider>,
  document.getElementById('root') as HTMLElement
);

registerServiceWorker();
```

# 4 起動

之后，使用以下命令启动它。

```
$ npm start
```



# 5 我要异步处理

通过以上操作，您已经创建了应用程序框架，但是在更正确的应用程序开发中，您将使用API。

由于使用了typescript-fsa，因此Action独自负责状态更新的起点，并且访问API的过程在容器中，该容器定义了组件的行为[3]（https://qiita.com/） 将其放在IzumiSy /项目/ b7d8a96eacd2cd8ad510＃fn3）。

# 6 异步动作的定义

如果您使用typescript-fsa中提供的`actionCreator.async`方法，则Action将在内部定义，其类型后缀为STARTED，DONE和FAILED。

```
import actionCreatorFactory, { ActionCreator, Success, Failure } from 'typescript-fsa';

const actionCreator = actionCreatorFactory();

const submit =
  actionCreator.async<{}, {}, {}>('ACTIONS_SUBMIT')

export interface sampleAsyncActions {
  startLogin: ActionCreator<{}>;
  failedLogin: ActionCreator<Failure<{}, {}>>;
  doneLogin: ActionCreator<Success<{}, {}>>;
}

export const sampleAsyncActions = {
  startLogin: submit.started,
  failedLogin: submit.failed,
  doneLogin: submit.done
}
```

## 6.1 Reducerの定義

Reducer仅创建一个处理Action中定义的三个状态的案例。

sampleAsyncState.ts

```
import { reducerWithInitialState } from 'typescript-fsa-reducers';
import { sampleAsyncActions } from '../actions/sampleAsyncActions';

// ...

export const sampleAsyncReducers = reducerWithInitialState(initialState)
  .case(sampleAsyncActions.startLogin, (state) => {
    // ...
  })
  .case(sampleAsyncActions.failedLogin, (state) => {
    // ...
  })
  .case(sampleAsyncActions.doneLogin, (state) => {
    // ...
  })
```

## 6.2 ContainerでのAPIアクセス処理の実装

通过上面已经实现的Container中的mapDispatchToProps，实现如下一系列的API访问过程。

sampleContainer.ts

```
// ...

import { sampleAsyncActions } from '../actions/sampleAsyncActions';

// ...


function mapDispatchToProps(dispatch: Dispatch<void>) {
  return {
    submit() {
      dispatch(sampleAsyncActions.startLogin({}));

      callYouOwnAPI()
        .then(() => {
          dispatch(sampleAsyncActions.doneLogin({
            params: {}, result: {}
          }));
        })
        .catch(() => {
          dispatch(sampleAsyncActions.failedLogin({
            params: {}, error: {}
          }));
        });
    },

    // ...
  };
}
```

您所要做的就是从组件内部调用它。

# 7 tslintの設定

将以下内容添加到tslint.json中

```
"rules": {
  "interface-name": [
    false
  ],
  "object-literal-sort-keys": false,
  "ordered-imports": false,
  "no-empty-interface": false,
  "interface-over-type-literal": false
}
```

------

1. https://github.com/acdlite/flux-standard-action [↩](https://qiita.com/IzumiSy/items/b7d8a96eacd2cd8ad510#fnref1)
2. https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0 [↩](https://qiita.com/IzumiSy/items/b7d8a96eacd2cd8ad510#fnref2)