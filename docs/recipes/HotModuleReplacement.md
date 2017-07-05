# Hot Module Replacement

替换正在运行的 Epics 为新的版本可能会引起潜在的奇怪 bugs,因为 Epics 可能会维护一些内部的状态或者依赖一些外部的瞬时状态或者副作用。

想想如何去抖动跟踪，或者取消你将 store 放入阻塞的状态的在 AJAX 请求之前更阴险。这并不是 redux-observable 特有的;每个类似的中间件都会有这个问题因为它和处理副作用是绑定的。

然而实践中，我们并不确定这是否会影响到普通的开发着，因为热替换只发生在本地开发中，我们确实提供了 `replaceEpic(nextEpic)` 方法可用于此目的。

```js
import rootEpic from './where-ever-they-are';
const epicMiddleware = createEpicMiddleware(rootEpic);

if (module.hot) {
  module.hot.accept('./where-ever-they-are', () => {
    const rootEpic = require('./where-ever-the-are').default;
    epicMiddleware.replaceEpic(rootEpic);
  });
}
```

当你调用 `replaceEpic`时， `@@redux-observable/EPIC_END` action 会在替换真实发生之前分发。这给你一个 **同步** 的机会做一些你需要的清理工作。你可以
选择在 Epic 内部监听这个 action 或者 在 reducer 中监听，总之最有意义的地方。   

#### Inside your Reducer

```js
import { EPIC_END } from 'redux-observable';

const userIsPending = (state = false, action) => {
  switch (action.type) {
    // Listen for the Epic ending to put things
    // back into a safe state
	case EPIC_END:
	case FETCH_USER_FULFILLED:
	case FETCH_USER_CANCELLED:
	  return false;
	  
	case FETCH_USER:
      return true;

    default:
      return state;
  }
};
```

#### Inside your Epic

```js
import { EPIC_END } from 'redux-observable';
import { race } from 'rxjs/observable/race';

// Race between the AJAX call and an EPIC_END.
// If the EPIC_END, emit a cancel action to
// put the store in the correct state
const fetchUserEpic = action$ =>
  action$.ofType(FETCH_USER)
    .mergeMap(action =>
      race(
        ajax(`/api/users/${action.payload}`),
        action$.ofType(EPIC_END)
          .take(1)
          .mapTo({ type: FETCH_USER_CANCELLED })
      )
    );
```

如果你使用 `replaceEpic` 并且出现了上文提到的任何种类的 bugs (或者它对你很有效)，[请告诉我们](https://github.com/redux-observable/redux-observable/issues/new) 以便于我们可以评估这个特性的未来！  
