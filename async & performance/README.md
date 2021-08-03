# 读书笔记 之《You Don't Know JS》- 异步和性能

## 异步和性能 —— 强烈推荐[阅读原著](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/sync-async/README.md)

### 简介
同步和异步的话题在 JS 中屡见不鲜，特别是当 ES6+ 规范新增了 `Promise`、`Generator`、`Async` 等新特性后，JS 中的异步编程更是如虎添翼，不必在局限于以前的回调函数。`Promise` 让状态可控，避免出现控制反转的安全问题，`Async` 函数则让回调地狱不复存在，更有意思的是 `Generator` 能让过去只能一口气执行到底的函数，按照你的意图只执行一部分，并保留调用栈，等待下次从暂停的地方继续恢复执行……

这一卷从第五节开始的后面两节中，探讨了作为一个资深程序员永远绕不开的话题 —— 性能。想要在 JS 的众多运行环境中测试到底是 `Number('5')` 还是 `parseInt('5')` 的性能更好，其实是一个伪命题。而作者提出了 **关键路径(critical path)** 的概念，在各种嘈杂声中抓住了性能优化的核心本质 —— 微观层面的性能优化并不能解决本质问题，只有根据具体的业务，有针对性的分析，才是正解。

### 导航
- [异步：当前 & 未来](/async%20%26%20performance/Now%20%26%20Later.md) (Asynchrony: Now & Later)

- [回调函数](/async%20%26%20performance/Callback.md) (Callback)

- [Promises](/async%20%26%20performance/Promises.md) (Promises)

- [Generators](/async%20%26%20performance/Generators.md) (Generators)

- [程序的性能](/async%20%26%20performance/Program%20Performance.md) (Program Performance)