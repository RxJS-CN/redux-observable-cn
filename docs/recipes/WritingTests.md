# Writing Tests

> 测试异步代码不是很容易的事情。我们仍然在学习测试 Epics 最好的方式。如果你有完美的方案，[请分享](https://github.com/redux-observable/redux-observable/issues/new)! 

如果你对常规 Redux 还没有进行过测试。你需要首先转向[它们的文档](http://redux.js.org/docs/recipes/WritingTests.html),因为几乎所有都是
适用的。

一种方式是 mock 完整的 Redux store 并且在每个测试用例中替换根 Epic。

```js
import nock from 'nock';
import expect from 'expect';
import configureMockStore from 'redux-mock-store';
import { createEpicMiddleware } from 'redux-observable';
import { fetchUserEpic, fetchUser, FETCH_USER } from '../../redux/modules/user';

const epicMiddleware = createEpicMiddleware(fetchUserEpic);
const mockStore = configureMockStore([epicMiddleware]);

describe('fetchUserEpic', () => {
  let store;

  beforeEach(() => {
    store = mockStore();
  });

  afterEach(() => {
    nock.cleanAll();
    epicMiddleware.replaceEpic(fetchUserEpic);
  });

  it('produces the user model', () => {
    const payload = { id: 123 };
    nock('http://example.com/')
      .get('/api/users/123')
      .reply(200, payload);

    store.dispatch({ type: FETCH_USER });

    expect(store.getActions()).toEqual([
      { type: FETCH_USER },
      { type: FETCH_USER_FULFILLED, payload }
    ]);
  });
});
```

***

如果你是一个特别的冒险者，我们已经试验性的使用[RxJS's `TestScheduler`](https://github.com/ReactiveX/rxjs/blob/master/doc/writing-marble-tests.md) 和 [marble diagram helpers](https://github.com/ReactiveX/rxjs/blob/master/doc/writing-marble-tests.md)
老实讲，我们现在还不太建议这种方式，但是我们喜欢看到反馈或者某些人可以感兴趣帮助铺路。
