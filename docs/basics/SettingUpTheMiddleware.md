# 设置中间件

现在我们了解了什么是[Epics](Epics.md), 我们需要将其提供给 redux-observable 中间件从而使它可以开始监听 actions。

## 根 Epic

和 redux 需要单个根 Reducer 相似，redux-observable 需要单个根 Epic。正如我们在[之前学过](Epics.md)的，可以使用 `combineEpics()` 来完成它。 

我们建议将所有的 Epics 导入到单个文件中，然后导出根 Epic 和 根 Reducer。

### redux/modules/root.js

```js
import { combineEpics } from 'redux-observable';
import { combineReducers } from 'redux';
import ping, { pingEpic } from './ping';
import users, { fetchUserEpic } from './users';

export const rootEpic = combineEpics(
  pingEpic,
  fetchUserEpic
);

export const rootReducer = combineReducers({
  ping,
  users
});
```

> 这个模式继承自 [Ducks Modular Redux pattern](https://github.com/erikras/ducks-modular-redux).

## 配置 Store

现在创建 redux-observable 中间件的实例，传递最新的根 Epic。 

```js
import { createEpicMiddleware } from 'redux-observable';
import { rootEpic } from './modules/root';

const epicMiddleware = createEpicMiddleware(rootEpic);
```

将上面的代码和你已经存在的 Store 配置像下面这样集成。

### redux/configureStore.js

```js
import { createStore, applyMiddleware } from 'redux';
import { createEpicMiddleware } from 'redux-observable';
import { rootEpic, rootReducer } from './modules/root';

const epicMiddleware = createEpicMiddleware(rootEpic);

export default function configureStore() {
  const store = createStore(
    rootReducer,
	applyMiddleware(epicMiddleware)
  );
  

  return store;
}
```

## Redux DevTools

为了开启 Redux DevTools 扩展，使用 `window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__` 或者 [import `redux-devtools-extension` npm package](https://github.com/zalmoxisus/redux-devtools-extension#13-use-redux-devtools-extension-package-from-npm)。

```js
import { compose } from 'redux'; // and your other imports from before
const epicMiddleware = createEpicMiddleware(pingEpic);

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

const store = createStore(pingReducer,
  composeEnhancers(
    applyMiddleware(epicMiddleware)
  )
);
```
