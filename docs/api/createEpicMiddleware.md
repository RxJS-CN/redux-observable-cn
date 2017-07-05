# `createEpicMiddleware(rootEpic, [options])`

`createEpicMiddleware()` 用来创建 redux-observable 中间件的实例。你提供单个根 Epic。

#### Arguments

1. *`rootEpic: Epic`*: 根 [Epic](../basics/Epics.md)
2. *`[options: Object]`*: 可选的配置. 选项:
    * *`dependencies`*: 如果提供, 它会以第三个参数注入给所有的 epics.
    * *`adapter`*: 适配器对象，可以转换 输入／输出流给你的 epics。 通常用来适配 RxJS v5 之外的流库，例如 [adapter-rxjs-v4](https://github.com/redux-observable/redux-observable-adapter-rxjs-v4) 或者 [adapter-most](https://github.com/redux-observable/redux-observable-adapter-most) 选项:
       * *`input: ActionsObservable => any`*: 转化 actions 输入流, `ActionsObservable` 传递给你的根 Epic (转化发生在传递给根 epic 之前)。
       * *`output: any => Observable`*: 转化根 Epic 的返回流 (转化发生在根 epic 返回之后)。

#### Returns

(*`MiddlewareAPI`*): redux-observable 中间件的实例。

#### Example

### redux/configureStore.js

```js
import { createStore, applyMiddleware, compose } from 'redux';
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
