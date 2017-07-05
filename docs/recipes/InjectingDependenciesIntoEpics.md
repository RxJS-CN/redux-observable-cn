# Injecting Dependencies Into Epics

将依赖注入到你 Epics 中帮助测试。 

假设你需要和网络交互。你可能会直接从 `rxjs` 中使用 `ajax` 工具:  

```js
import { ajax } from 'rxjs/observable/dom/ajax';

const fetchUserEpic = (action$, store) =>
  action$.ofType('FETCH_USER')
    .mergeMap(({ payload }) =>
      ajax.getJSON(`/api/users/${payload}`)
        .map(response => ({
          type: 'FETCH_USER_FULFILLED',
          payload: response
        }))
    );
```

但是这种方式有一个问题: 包含 epic 的文件直接导入了它的依赖，所以 mocking 就会非常困难。 

一种方式可能是 mock `window.XMLHttpRequest`, 但是这工作量很大并且你不是在仅仅测试你的 Epic，你是在测试 RxJS 正确使用 XMLHttpRequest，事实上，这并不是你的测试目的。 

### 注入依赖

你可以使用 `createEpicMiddleware` 的 `dependencies` 配置项来注入依赖:

```js
import { createEpicMiddleware, combineEpics } from 'redux-observable';
import { ajax } from 'rxjs/observable/dom/ajax';
import rootEpic from './somewhere';

const epicMiddleware = createEpicMiddleware(rootEpic, {
  dependencies: { getJSON: ajax.getJSON }
});
```

任何你提供的都会在创建 store 之后被当作所有 Epics 的第三个参数传入。

现在你的 Epic 可以使用注入的 `getJSON`，而不用导入:

```js
// 注意第三个参数是我们注入的依赖!
const fetchUserEpic = (action$, store, { getJSON }) =>
  action$.ofType('FETCH_USER')
    .mergeMap(({ payload }) =>
      getJSON(`/api/users/${payload}`)
        .map(response => ({
          type: 'FETCH_USER_FULFILLED',
          payload: response
        }))
    );

```

为了测试，你可以直接调用 Epic，传递 mock 的 `getJSON`:

```js
import { ActionsObservable } from 'redux-observable';
import { fetchUserEpic } from './somewhere/fetchUserEpic';

const mockResponse = { name: 'Bilbo Baggins' };
const action$ = ActionsObservable.of({ type: 'FETCH_USERS_REQUESTED' });
const store = null; // 该 epic 不需要
const dependencies = {
  getJSON: url => Observable.of(mockResponse)
};

// 将这个例子适用到你的测试框架和特定用例 
fetchUserEpic(action$, store, dependencies)
  .toArray() // 缓冲所有发出的 actions 直到 epic 自然完成
  .subscribe(actions => {
    assertDeepEqual(actions, [{
      type: 'FETCH_USER_FULFILLED',
      payload: mockResponse
    }]);
  });
```
