# Cancellation

取消一些异步行为是 Epics 的最基本需求。虽然有很多种方式可以做到这一点，这取决于你的需求，最常见的方式是让应用程序分发取消 action 然后在 Epic 内部监听。 

可以用 RxJS 的操作符 [`.takeUntil()`](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-takeUntil)来完成 :

```js
import { ajax } from 'rxjs/observable/dom/ajax';

const fetchUserEpic = action$ =>
  action$.ofType(FETCH_USER)
    .mergeMap(action =>
      ajax.getJSON(`/api/users/${action.payload}`)
        .map(response => fetchUserFulfilled(response))
        .takeUntil(action$.ofType(FETCH_USER_CANCELLED))
    );
```

这里我们将 `.takeUntil()` 放到[`.mergeMap()`](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-mergeMap)内部, 并且在 AJAX 调用之后; 这很重要因为我们想要取消的是 AJAX 请求，而不是停止 Epic 监听任何未来的 actions。 像这样隔离 observable 链将是很重要的概念你会经常使用。如果你不是很明白，你应该花点时间尽快的熟悉 RxJS 和 操作符链通常是如何工作的。 Ben Lesh [有一个很棒的演讲，解释了 Observables 是如何工作的](https://www.youtube.com/watch?v=3LKMwkuK0ZE) 同时覆盖到了隔离链!

> 这个例子使用了 `mergeMap` (别名 `flatMap`)，它允许多个并行的 `FETCH_USER` 请求。如果你想取消任何请求然后选择最近的一个，你可以使用 [`switchMap`](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-switchMap)操作符。  

***

### Try It Live!

<a class="jsbin-embed" href="https://jsbin.com/fivaca/embed?js,output&height=500px">View this demo on JSBin</a><script src="https://static.jsbin.com/js/embed.min.js?3.37.0"></script>


## 取消然后做一些其他的事情 (发出另一个 Action)

有时候你想不仅仅取消像 AJAX 调用这种副作用，同时还要做一些其他事情，例如发出完全不同的 action。

你可以通过使用 [`race`](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-race) 操作符来达到这个目的。
它允许你使得流之间进行比赛;谁先发出值谁获胜！失败的流会被取消订阅，取消它执行的任何操作。

例如，当某人分发 `FETCH_USER` 时我们会执行一个 AJAX 调用，但是如果某人分发 `FETCH_USER_CANCELLED` 我们会取消 AJAX 请求并且发出一个完全不同的 action 做为代替，在本例中，增加一个计数: 


```js
import { ajax } from 'rxjs/observable/dom/ajax';

const fetchUserEpic = action$ =>
  action$.ofType(FETCH_USER)
    .mergeMap(action =>
      ajax.getJSON(`/api/users/${action.payload}`)
        .map(response => fetchUserFulfilled(response))
        .race(
          action$.ofType(FETCH_USER_CANCELLED)
            .map(() => incrementCounter())
            .take(1)
        )
    );
```

我们同样也要使用 `.take(1)`，因为当我们和 AJAX 调用竞赛时只想监听取消 action 一次。  

> 这引发了一个值得深思的问题: 是否可以惯用原始取消 action 被 reducers 吸收，而不是用一个单独的 action 追踪取消事件？换句话说，是不是依赖于单个同时触发
取消本身和后来会发生什么或者定义两个独立的 actions 更能反映你的本意？  
