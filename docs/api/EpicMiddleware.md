# EpicMiddleware

edux-observable 中间件的实例.

想要创建它, 传递你的根 [Epic](../basics/Epics.md) 给 [`createEpicMiddleware`](createEpicMiddleware.md).

### EpicMiddleware Methods

- [`replaceEpic(nextEpic)`](#replaceEpic)

<hr>

### <a id='replaceEpic'></a>[`replaceEpic(nextEpic)`](#replaceEpic)

替换中间件正在使用的 epic。 

这是个高阶 API。 如果你想要应用实现代码分割并且想要动态加载 epics 或者使用热更新时，你会需要它们。

#### Arguments

1. `epic` (*Epic*) 中间件要使用的下一个 epic。
