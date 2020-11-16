# Generators
回顾下第二节提到的 Callback 的弊端：
- 

- 

在第三节中出现的 Promise 很好的解决了👆这些问题，那么接下去我们就要探索异步编程的流程控制 —— 同步模式的下的异步编程 —— Generator。

## 打破 “一气呵成”(Breaking Run-to-Completion)
普通的函数一旦开始执行，就只能执行到底，没有机制能够打断它的执行。

generator 是 ES6 引进的新类型的函数
