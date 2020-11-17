# Generators
回顾下第二节提到的使用 callback 进行异步编程的弊端：
- 不符合我们大脑按步骤进行的工作模式

- 因为 IoC 的问题，callback 并不安全可信

在第三节中出现的 Promise 很好的解决了👆安全可信的问题，但对于符合大脑的工作模式，依然无能为力。而接下去我们的探索按照同步模式实现的异步编程 —— Generator 异步编程的流程控制，就是解决这个问题的一剂良药。

## 打破 “一跑就到底”(Breaking Run-to-Completion)
众所周知，在 JS 中，普通的函数一旦开始执行，就只能执行到底，没有机制能够打断它的执行。

而 ES6 引入的 generator 是一个新类型的函数，它能够打破这个 “无法停机” 问题。

一个简单的思想实验帮助我们更好的理解：

```js
var x = 1;

function foo () {
  x++;
  bar();
  console.log('x: ', x);
}

function bar () {
  x++;
}

foo(); // x: 3
```

👆显而易见 `x` 的结果是 `3`；但若当 `bar()` 没有在 `foo()` 中被调用，而又想让 `x` 的结果不变，有什么办法能实现呢？

在 可抢占式的多线程语言(preemptive multithreaded languages) 中，当某个线程执行到 `x++` 时，另一个线程就能见缝插针的调用 `bar()`，从而将结果维持不变。但 JS 不仅不能实现 “抢占”，而且它还是单线程的，那这样的实现也就无从谈起了。

不过，有了 generator 之后，实现这样的 协同并发(cooperative concurrency) 机制成为了可能，因为你能暂停 `foo()` 的执行：

```js
var x = 1;

function *foo () {
  x++;
  yield;
  console.log('x: ', x);
}

function bar () {
  x++;
}

var it = foo();
it.next();
x; // 2
bar();
x; // 3
it.next(); // x: 3
```

**Note**：关于 `*` 的位置，牵扯到了风格的选择，比如有的文档或代码中是将 `*` 靠近 `function`：`function* foo () {……}`。而作者选择将 `*` 靠近函数名是为了更便于表达，就比如 `*foo()` 和 `foo()` 的区别一望便知。

上面整个 generator 函数 `*foo()` 执行的过程如下：
  1. `it = foo()` 并不是执行 generator，而是相当于构造了一个 *iterator* 对象，用来控制整个 generator 的执行；

  2. 第一次调用 `it.next()` 才是正式开始执行 generator，并且执行了第一行 `x++` 代码；

  3. 随后遇到了第一个 `yield` 关键字，函数 `*foo()` 的执行就暂停了；

  4. 而后我们输入 `x` 得到了其当前的值为 `2`；

  5. 接下去执行了 `bar()`，在其内部执行了 `x++` 的代码；

  6. 再一次检查 `x` 的值为 `3`； 

  7. 最终，调用 `it.next()` 将暂停的 generator 函数恢复执行，并控制台打印出了 `"x: 3"` 的信息……

总的来讲，在 generator 函数中，通过关键字 `yield` 就能暂停 generator 函数的执行，而后可以通过调用 `next()` 恢复函数的执行。并且，函数执不执行完显然不是强制要求的 —— 看上去没啥了不起？

### 输入和输出(Input and Output)
generator 函数本质上来看依然是函数，因此一些普通函数的基本特性依然对它有效，就比如 “输入” —— 参数 和 “输出” —— 返回值：

```js
function *foo(x, y) {
  return x * y;
}

var it = foo();

var res = it.next(6, 7);

res.value; // 42
```

👆 我们将 `6` 和 `7` 作为 `*foo(…)` 的参数传入，而后计算出 `6 * 7` 的值后返回 —— 除了在激活 generator 函数的形式上，这些操作好像和普通的函数调用没有什么太大的区别。但细节却不能忽视：`*foo(6, 7)` 并不是函数调用，而是构建出了一个 iterator 对象；`it.next()` 才开始执行 generator 的函数体内容，而其返回值是一个包含了 `value` 属性的对象；而 `value` 的值则是 `yield` 或 `return` 输出的值。

但为什么我们需要一个可迭代的对象来控制 generator 函数呢？让它自执行不好吗？

#### 迭代的信息(Iteration Messaging)
除了用参数和返回值实现 “输入” 和 “输出” 之外，通过关键字 `yield` 和 迭代对象的 `next(…)` 方法，能够实现颗粒度更细，更有用的 “输入” 和 “输出”：

```js
function *foo(x) {
  const baseInd = Math.round(Math.random() * 10);
  var y = x * (yield baseInd);
  return y;
}

var it = foo(6);

var res = it.next();

res.value;

res = it.next(res.value * 5);

res.value;
```

`foo(6)` 将 `6` 作为参数传入 generator 函数，调用 `it.next()` 后，获得 `(yield baseInd)` 中的 `baseInd`，而后将其乘以 `5`，并作为参数再次传递到 generator 中，最终生成的值是这一系列参数传递的组合，即 `6 * baseInd * 5`。

需要强调的是，在上面执行 generator 的过程中，有一个有趣的现象 —— `next(…)` 始终比 `yield` 多一个。有些人误以为这是个错配，但导致这个 “错配” 的原因是因为第一个 `next(…)` 的作用是启动 generator 函数，而后在遇到第一个 `yield` 关键字后暂停整个函数的后续执行 —— 这即意味着说，你想要获得 `yield` 之后的结果，至少还需要执行一次 `next(…)` 才行。

##### 两个问题的故事(Tale of Two Question)
换个角度来看这个所谓的 “错配” 的问题可能会得到不一样的解读。

类似于服务端和客户端通过API进行通信一样，我们将 generator 的内部和外部的 iterator 同样视为是服务端和客户端：

```js
function *server(x) {
  const baseInd = Math.round(Math.random() * 10);
  var y = x * (yield baseInd);
  return y;
}

var client = server(6); // 和服务端建立了连接，并发送了 6 作为参数

var res = client.next(); // 客户端：服务端，请把第一个返回值告诉我，谢谢

res.value; // 一个由服务端生成的值

res = client.next(res.value * 5); // 将这个值乘以 5 发送给服务端

res.value; // 得到服务端最终生成的值
```

👆 一个类似于 “半双工通信” 的机制由此而来，并且由于第一次调用 `next()` 的时候，本质上来看就是启动 generator 函数，若是将它想象成函数一开始就有一个隐藏的 `yield`，并且从这里出发，那么不用传递任何参数(argument-free)也是在情理之中的。

最后的 `return y` 标示着整个 generator 函数执行完毕。若是没有显示的写 `return` 的代码，那么和普通函数一样，JS 引擎会隐式的帮你加上 `return undefined;` 的语句。
 
这种通过 `yield` 和 `next(…)` 实现的通信机制非常强大，但离异步流程控制好像还有点距离？！别急，接着看下去。

### 多个迭代器(Multiple Iterators)