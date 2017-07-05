# Adding New Epics Asynchronously/Lazily

如果你正在做代码分隔或者在应用运行过程中你想动态的给中间件添加 Epic，你可以利用标准 RxJS 很简单的做到。 

```js
import { BehaviorSubject } from 'rxjs/BehaviorSubject';
import { combineEpics } from 'redux-observable';

const epic$ = new BehaviorSubject(combineEpics(epic1, epic2));
const rootEpic = (action$, store) =>
  epic$.mergeMap(epic =>
    epic(action$, store)
  );

// sometime later...add another Epic, keeping the state of the old ones...
epic$.next(asyncEpic1);
// and again later add another...
epic$.next(asyncEpic2);
```

牢记，任何存在的 Epics 将会仍然保持运行，这不是真正的 Epics 替换，像热加载。  

往根 Epic 中添加新的 Epics 十分安全，但是把正在运行的 Epics 替换成新的版本可能会潜在的奇怪的 bugs，因为 Epics 可能会维护状态或者依赖外部瞬时状态或者副作用。想要了解更多，查看 [`replaceEpic(nextEpic)`](HotModuleReplacement.md) 方法。  
