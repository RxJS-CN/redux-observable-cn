# Error Handling

在 Epics 中处理像 AJAX 调用这种副作用中的错误是一个很常见的需求。尽管有多种方式可以实现，这取决于你的需求，最常见的方式是在 Epic 内部捕获错误并且发出
带有错误信息的 action 便于展现或者日志记录。

这可以通过 RxJS 的操作符 `.catch()` 来完成:

```js
import { ajax } from 'rxjs/observable/dom/ajax';

const fetchUserEpic = action$ =>
  action$.ofType(FETCH_USER)
    .mergeMap(action =>
	  ajax.getJSON(`/api/users/${action.payload}`)
        .map(response => fetchUserFulfilled(response))
        .catch(error => Observable.of({
        	type: FETCH_USER_REJECTED,
        	payload: error.xhr.response,
        	error: true
        }))
    );
```

这里我们将 `.catch()` 放到了 `.mergeMap()` 内部，AJAX 调用的后面; 这很重要因为如果让错误到达 `action$.ofType()`，epic 会终止并且不会监听任何 actions。

***

### Try It Live!

<a class="jsbin-embed" href="https://jsbin.com/yuleju/embed?js,output&height=500px">View this demo on JSBin</a><script src="https://static.jsbin.com/js/embed.min.js?3.37.0"></script>

