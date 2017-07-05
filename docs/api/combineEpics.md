# `combineEpics(...epics)`

`combineEpics()`, 意如其名, 允许你将多个 epics 合并成单个。

#### Arguments

1. *`...epics: Epic[]`*: 要合并的 [epics](../basics/Epics.md)。

#### Returns

(*`Epic`*): 合并每个 Epic，并提供给它们 `ActionsObservable` 和 redux store 作为参数的 Epic。

#### Example

#### `epics/index.js`

```js
import { combineEpics } from 'redux-observable';
import pingEpic from './ping';
import fetchUserEpic from './fetchUser';

export default combineEpics(
  pingEpic,
  fetchUserEpic
);
```
