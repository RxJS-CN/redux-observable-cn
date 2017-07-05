# FAQ

> 因为 redux-observable 的使用强依赖于 RxJS，你绝大多数的问题是和 RxJS 相关而不是 redux-observable 自身。我们强烈
鼓励你在强大的 RxJS 社区获取帮助。
> 
> 如果你的 redux-observable 问题没有被在这里列出来，请首先 [使用 Stack Overflow](http://stackoverflow.com/questions/tagged/redux-observable) 如果你在一段合理的时间后没有收到响应，创建一个包含 Stack Overflow 标签的 issue。 

## Table of Contents

- **General**
  - [和 RxJS 4/most.js/bacon.js 能一起工作吗?](#general-rxjs-v4)
  - [thunkservables 为什么被废弃?](#general-thunkservables-deprecated)

## General

<a id="general-rxjs-v4"></a>
### 和 RxJS 4/most.js/bacon.js/等等 能一起工作吗? 

##### RxJS v4

不在计划之中，但是我们提供了适配器[redux-observable-adapter-rxjs-v4](https://github.com/redux-observable/redux-observable-adapter-rxjs-v4)
在你升级到 RxJS v5 你可以先安装它作为过渡。请尽快升级到 v5 因为我们不打算长期支持这个。  

##### most.js
##### bacon.js
##### etc.

你可以创建适配器提供支持。参考[redux-observable-adapter-rxjs-v4](https://github.com/redux-observable/redux-observable-adapter-rxjs-v4)。

<a id="miscellaneous-thunkservables-deprecated"></a>
<a id="why-were-thunkservables-removed"></a>
### thunkservables 为什么被移除?

在最初的 redux-observable [Epics](basics/Epics.md) 实现被称为 thunkservables 因为你可以分发它，就像[redux-thunk](https://github.com/gaearon/redux-thunk)。虽然这样更简洁，我们发现在实践中当处理像 自动补全／防抖／取消 这种用户相互型时不具有扩展性并且数据流不一致。
因为我们不想用两种相似的选项困惑开发者，所以我们决定完全移除 thunkservables。查看[Epics](basics/Epics.md)文档学习如何开始使用它们。
作为奖励，你可以很轻易的和 redux-observable 一起使用 [redux-thunk](https://github.com/gaearon/redux-thunk) (或者其他 thunk-like 中间件)
所以如果你喜欢，你可以在绝大多数简单异步任务使用 edux-thunk 然后在复杂异步任务使用 redux-observable; 或者解除集成。
