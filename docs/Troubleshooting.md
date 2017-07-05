# Troubleshooting [![Join the chat at https://gitter.im/redux-observable/redux-observable](https://badges.gitter.im/redux-observable/redux-observable.svg)](https://gitter.im/redux-observable/redux-observable?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

这是一个分享公告问题并且找到解决方案的地方。

>如果你的问题没有被在这里列出来或者你需求其他帮助，请首先使用[Stack Overflow](http://stackoverflow.com/questions/tagged/redux-observable)添加 
`redux-observable` 标签。如果在一个合理的时间段后没有得到响应，创建包含 Stack Overflow 问题链接的 issue   
>
> 你同样可以从公共[Gitter channel]中得到帮助(https://gitter.im/redux-observable/redux-observable)!


* * *

### RxJS operators are missing! e.g. TypeError: action$.ofType(...).switchMap is not a function

RxJS 支持[单独添加操作符](https://github.com/ReactiveX/rxjs#installation-and-usage)的能力从而使你的包会很小。
redux-observable 通过使 `ActionsObservable` 继承 `Observable`，但是没有在原型上添加任何操作符来实现。

#### Add all operators

如果你想添加所以的操作符, 你可以在你的入口文件 `index.js` 中添加完整的库:

```js
import 'rxjs';
```
这会将所有的 RxJS 核心操作符添加到 `Observable` 原型上。

#### Add only the operators you use

tl;dr


```js
import 'rxjs/add/operator/switchMap';
action$.ofType(...).switchMap(...);

// OR

import { switchMap } from 'rxjs/operator/switchMap';
action$.ofType(...)::switchMap(...);
```

有很多种方法可以做到，所以我们在文档中并没有特别的推荐某个。产看 [RxJS 文档]
(https://github.com/ReactiveX/rxjs#installation-and-usage).

如果巧妙的使用了 `'rxjs/add/operator/name'`，你可能会发现将所有依赖的操作符引用放到创建的单个文件中，你就不会引入同一个操作符多次。 

* * *

### TypeError: object is not observable

这通常意味着你给 RxJS 操作符传递了非 observable-like 的对象。这意味不是 Observable，Promise，Array，不支持 `Symbol.observable` 等等。    

下面是一些例子

#### (action$, store) => Observable.from(store)

提供给 Epics 的 store 和 redux 给中间件的 store 相同。并不是完全版本的 store, 所以它不支持 `Symbol.observable` 从而不允许 `Observable.from(store)`。你可以 [看这里学到更多](https://github.com/redux-observable/redux-observable/issues/56).

* * *

### TypeError: Cannot read property 'subscribe' of undefined

这大多数意味着你正在使用期望提供 observable 的操作符但是你什么也没有提供。经常，你提供了变量但是被某些未知设置为 `undefined`，所以 debugger 进一步确认。   
#### Happens from `combineEpics()`

通常，这意味着你的某个 Epics 没有返回 observable。很多情况下都是缺失了 `return`。

```js
const myEpic = action$ => { // MISSING EXPLICIT RETURN!
  action$.ofType(...).etc(...)
};
```

### this is set to Window

如果你将 epics 放到类中。(例如，为了获取 Angular 2 依赖注入的好处)，你可能在使用类方法的时候遇到错误: 

```typescript
class TooFancy {
  constructor(private somethingInjected:SomethingInjected)
  checkAutoLogin (action$: Observable<IPayloadAction>) {
    console.log(this); // Is Window! when called from redux-observable
  }
}
```
修改为:

```typescript
class TooFancy {
  constructor(private somethingInjected:SomethingInjected)
  checkAutoLogin =  (action$: Observable<IPayloadAction>) => {
    console.log(this); // YOu can access somethingInjected
  }
}
```

察看 https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions (箭头函数)

* * *

## Something else doesn’t work

如果你认为你的 issue 是 redux-observable 的一个 bug，[创建 issue](https://github.com/redux-observable/redux-observable/issues);  

如果你弄明白了，[编辑该文档](https://github.com/redux-observable/redux-observable/edit/master/docs/Troubleshooting.md) 这样对下个遇到同样问题
的人很礼貌。
