# Epics

>##### 不熟悉 Observables/RxJS v5?
> redux-observable 需要理解 RxJS v5 的 Observables。 如果你是用 RxJS v5 进行响应式编程的新手, 转向 [http://reactivex.io/rxjs/](http://reactivex.io/rxjs/) 先熟悉下。
> 
> redux-observable (因为 RxJS) 的真正光芒在于处理最为复杂的异步逻辑。如果你对 RxJS 感到不太舒适，你可以考虑使用[redux-thunk](https://github.com/gaearon/redux-thunk) 处理简单异步逻辑，然后使用 redux-observable 处理复杂情况。这样你既可以保持生产力又可以学习 RxJS。redux-thunk 学习和使用起来更简单，这也意味着它远没有那么强大。所以，如果你已经和我们一样喜爱 Rx，你可能会用它来做每一件事情。

**Epic** 是 redux-observable 的核心原语。

它是一个函数，接收 actions 流作为参数并且返回 actions 流。 **Actions 入, actions 出.**

它的签名如下:

```js
function (action$: Observable<Action>, store: Store): Observable<Action>;
```

虽然通常你会响应接收到的 action 而产出 actions，但这不是必须的！一旦进入你的 Epic，使用任何你想使用的 Observable 模式，只要最后返回 action 流即可。  

你发出的 actions 会通过 `store.dispatch()` 立刻被分发，所以 redux-observable 实际上会做 `epic(action$, store).subscribe(store.dispatch)`

Epics 运行在正常的 Redux 分发通道上，在 reducers 接受到**之后**，所以你不会 “吞掉” 一个 action。
在 Epics 实际接收 Actions 前，Actions 将始终贯穿你的 reducers 。

如果你让传入到 action 传出，会造成无限循环:

```js
// 不要这么做
const actionEpic = (action$) => action$; // 创建无限循环
```

> 这种处理副作用的方式和"*过程管理*"模式相似，有些地方也称为["*saga*"](https://msdn.microsoft.com/en-us/library/jj591569.aspx)，但是 [saga 的
原始定义]并不适用。(http://kellabyte.com/2012/05/30/clarifying-the-saga-pattern/)。如果你熟悉 [redux-saga](https://redux-saga.github.io/redux-saga/), redux-observable 和它很像。但是因为它使用了 RxJS， 所以它是更加声明式的，同时还可以扩展你现有的
RxJS 能力。

## 一个基本例子

> **重要:** redux-observable 并没有给 `Observable.prototype` 添加任何操作符，所以你需要在入口文件添加你使用的或者所有的操作符。
> 
> 因为有很多种方式可以添加它们，我们的例子不会包含任何导入。如果你想添加所有的操作符，将 `import 'rxjs';` 添加到你的入口 `index.js`。[Learn more](http://redux-observable.js.org/docs/Troubleshooting.html#rxjs-operators-are-missing-eg-typeerror-actionoftypeswitchmap-is-not-a-function).


让我们从一个简单的 Epic 例子开始:

```js
const pingEpic = action$ =>
  action$.filter(action => action.type === 'PING')
    .mapTo({ type: 'PONG' });
    
// later...
dispatch({ type: 'PING' });
```

> 注意，为什么 `action$` 是以美元符结尾呢? 这是 RxJS 的基本公约用来引用流。
 
`pingEpic` 会监听类型为 `PING` 的 actions，然后投射为新的 action，`PONG`。这个例子功能上相当于做了这件事情: 

```js
dispatch({ type: 'PING' });
dispatch({ type: 'PONG' });
```

> 牢记: Epics 运行在正常分发渠道旁, 在 reducers 完全接受到它们**之后**。当你将一个 action 投射成另一个 action， **你不会** 阻止原始的 action 到达 reducers; 该 action 已经通过了它!

真正的力量来自于你需要做一些异步事情。假设我们需要在接受到 `PING` 1秒后分发 `PONG`:

```js
const pingEpic = action$ =>
  action$.filter(action => action.type === 'PING')
    .delay(1000) // 异步等待 1000ms 然后继续
    .mapTo({ type: 'PONG' });

// 稍后...
dispatch({ type: 'PING' });
```

你的 reducers 会接收原始的 `PING` action，然后 1 秒后接收 `PONG`。

```js
const pingReducer = (state = { isPinging: false }, action) => {
  switch (action.type) {
    case 'PING':
      return { isPinging: true };

    case 'PONG':
      return { isPinging: false };

    default:
      return state;
  }
};
```
因为过滤特定的 action 类型是很常见的需求，`action$` 流拥有 `ofType()` 操作符来减少这种复杂度。

```js
const pingEpic = action$ =>
  action$.ofType('PING')
    .delay(1000) // 异步等待 1000ms 然后继续
    .mapTo({ type: 'PONG' });
```

***

### Try It Live!

<a class="jsbin-embed" href="https://jsbin.com/vayoho/embed?js,output&height=500px">View this demo on JSBin</a><script src="https://static.jsbin.com/js/embed.min.js?3.37.0"></script>

## 来着真实世界的例子

现在我们对 Epic 是什么有了大概的了解，让我们继续，看下这个更加真实的例子:

```js
import { ajax } from 'rxjs/observable/dom/ajax';

// action creators
const fetchUser = username => ({ type: FETCH_USER, payload: username });
const fetchUserFulfilled = payload => ({ type: FETCH_USER_FULFILLED, payload });

// epic
const fetchUserEpic = action$ =>
  action$.ofType(FETCH_USER)
    .mergeMap(action =>
      ajax.getJSON(`https://api.github.com/users/${action.payload}`)
        .map(response => fetchUserFulfilled(response))
    );
    
// 稍后...
dispatch(fetchUser('torvalds'));
```

> 我们使用 `fetchUser` action 创建函数(工厂)代替直接创建 action POJO。这是一个完全可选的 Redux 惯例。 

我们用个标准的 Redux action 创建者 `fetchUser`，同样也有一个对应的 Epic 去编排实际的 AJAX 调用。当 AJAX 调用返回时，我们将响应投射为 `FETCH_USER_FULFILLED` action。  

>记住，Epics 是一个 **actions in** 和 **actions out** 的流。如果你发现 RxJS 操作符和行为如此陌生，在继续之前你应该[深入看看 RxJS](http://reactivex.io/rxjs/)  

在 `FETCH_USER_FULFILLED` action 的响应中，你可以修改你的 Store's state。 

```js
const users = (state = {}, action) => {
  switch (action.type) {
    case FETCH_USER_FULFILLED:
      return {
        ...state,
        // `login` is the username
        [action.payload.login]: action.payload
      };

    default:
      return state;
  }
};
```

***

### Try It Live!

<a class="jsbin-embed" href="https://jsbin.com/jopuza/embed?js,output&height=500px">View this demo on JSBin</a><script src="https://static.jsbin.com/js/embed.min.js?3.40.2"></script>

## 访问 the Store's State

你的 Epics 接收的第二个参数，一个轻量版的 Redux store。 

```js
type LightStore = { getState: Function, dispatch: Function };

function (action$: ActionsObservable<Action>, store: LightStore ): ActionsObservable<Action>;
```

> 这不是完整的 store 对象的引用，只包含了 `store.getState()` 和 `store.dispatch()`; 它[目前不支持](https://github.com/reactjs/redux/issues/1833)`Observable.from(store)`。  

拥有它，你就可以调用 `store.getState()` 同步获取当前 state:

```js
const INCREMENT = 'INCREMENT';
const INCREMENT_IF_ODD = 'INCREMENT_IF_ODD';

const increment = () => ({ type: INCREMENT });
const incrementIfOdd = () => ({ type: INCREMENT_IF_ODD });

const incrementIfOddEpic = (action$, store) =>
  action$.ofType(INCREMENT_IF_ODD)
    .filter(() => store.getState().counter % 2 === 1)
    .map(() => increment());

// later...
dispatch(incrementIfOdd());
```
> 记住: 当 Epic 接收到 action, 它已经运行通过你的 reducers 并且 state 被修改了。

在 Epic 内部使用 `store.dispatch()` 是一个快速黑客的方便逃生舱，但是节制的使用。这被认为是反模式的并且会被在未来的版本中移除。

***

### Try It Live!

<a class="jsbin-embed" href="https://jsbin.com/somuvur/embed?js,output&height=500px">View this demo on JSBinn</a><script src="https://static.jsbin.com/js/embed.min.js?3.37.0"></script>

## 结合 Epics

最后，redux-observable 提供了一个工具方法 `combineEpics()`](../api/combineEpics.md)，该方法运行你轻易的将多个 Epics 结合为一个: 

```js
import { combineEpics } from 'redux-observable';

const rootEpic = combineEpics(
  pingEpic,
  fetchUserEpic
);
```

等价于:

```js
import { merge } from 'rxjs/observable/merge';

const rootEpic = (action$, store) => merge(
  pingEpic(action$, store),
  fetchUserEpic(action$, store)
);
```

## 下一步

接下来，我们会探索怎样 [激活 Epics](SettingUpTheMiddleware.md) 才能开始监听 actions。
