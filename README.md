<img title="logo" src="logo/logo-small.gif" width="16.5%">
<img title="redux-observable" src="logo/logo-text-small.png" width="72%">

[![Join the chat at https://gitter.im/redux-observable/redux-observable](https://badges.gitter.im/redux-observable/redux-observable.svg)](https://gitter.im/redux-observable/redux-observable?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![build status](https://img.shields.io/travis/reactjs/redux/master.svg)](https://travis-ci.org/redux-observable/redux-observable)
[![npm version](https://img.shields.io/npm/v/redux-observable.svg)](https://www.npmjs.com/package/redux-observable)
[![npm downloads](https://img.shields.io/npm/dm/redux-observable.svg)](https://www.npmjs.com/package/redux-observable)
[![code climate](https://codeclimate.com/github/redux-observable/redux-observable/badges/gpa.svg)](https://codeclimate.com/github/redux-observable/redux-observable)
[![Greenkeeper badge](https://badges.greenkeeper.io/redux-observable/redux-observable.svg)](https://greenkeeper.io/)

基于 [RxJS 5](http://github.com/ReactiveX/RxJS) 的 [Redux](http://github.com/reactjs/redux) 中间件
。 通过组合和取消异步动作去创建副作用。

[https://redux-observable-cn.js.org/](https://redux-observable-cn.js.org/)

## 安装

强依赖于 `rxjs@5.x.x` 和 `redux`，所以同样也要安装。

```bash
npm install --save redux-observable
```

**重要:** redux-observable 并没有给 `Observable.prototype` 添加任何 RxJS 操作符，所以你需要在入口导入你使用的或者所有操作符。 [更多](http://redux-observable.js.org/docs/Troubleshooting.html#rxjs-operators-are-missing-eg-typeerror-actionoftypeswitchmap-is-not-a-function).

##### 可选的适配器

Epics 默认使用 RxJS v5。 你可以通过适配器使用任何其他流库。(除了 RxJS v5 之外的)。

* [RxJS v4](https://github.com/redux-observable/redux-observable-adapter-rxjs-v4)
* [most.js](https://github.com/redux-observable/redux-observable-adapter-most)

你也可以实现你自己的适配器:

```js
const adapter = {
  input: input$ => /* 将 Observable 转化为你喜欢的流库 */,
  output: output$ => /* 将你喜欢的流库转化为 Observable */
};
```

参见现有适配器样例。牢记，虽然你仍然需要安装 RxJS v5， redux-observable 仅仅拉取了它内部需要的最小体积的 RxJS －不需要导入所有的 RxJS。

##### UMD

我们发布了 npm 包构件出的 UMD。 你可以通过 [unpkg](https://unpkg.com/) 来使用:

[https://unpkg.com/redux-observable@latest/dist/redux-observable.min.js](https://unpkg.com/redux-observable@latest/dist/redux-observable.min.js)

## 观看介绍

[![Watch a video on redux-observable](http://img.youtube.com/vi/AslncyG8whg/0.jpg)](https://www.youtube.com/watch?v=AslncyG8whg)

## JSBin 例子

看看实践中的 redux-observable，下面是一些简单的 JSBin 例子。

* [Using Raw HTML APIs](http://jsbin.com/birogu/edit?js,output)
* [Using React](http://jsbin.com/jexomi/edit?js,output)
* [Using Angular v1](http://jsbin.com/laviti/edit?js,output)
* Using Angular v2 (TODO)
* Using Ember (TODO)

## 文档

### [https://redux-observable.js.org](https://redux-observable.js.org)

## 讨论

[![Join the chat at https://gitter.im/redux-observable/redux-observable](https://badges.gitter.im/redux-observable/redux-observable.svg)](https://gitter.im/redux-observable/redux-observable?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

欢迎每个人 [Gitter channel](https://gitter.im/redux-observable/redux-observable?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)!

## 定制的 Emoji

##### 保存它:

<img src="https://cloud.githubusercontent.com/assets/762949/18562188/905876f6-7b37-11e6-8677-f9dd091490f6.gif" width="22" height="22" />

将 redux-observable 的旋转 logo 添加到你的 Slack channel 中！[Slack Instructions](https://get.slack.help/hc/en-us/articles/206870177-Create-custom-emoji)

***

*redux-observable 是一个社区驱动的项目和 Netflix 没有官方的附属关系。

:shipit:
