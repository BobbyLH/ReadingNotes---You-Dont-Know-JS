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
每次调用 genenrator 函数，都会创建一个新的 iterator 实例，即意味着你能同时创建多个 iterator 的实例，而且它们还能相互作用：

```js
function *foo() {
  var x = yield 2;
  z++;
  var y = yield (x * z);
  console.log(x, y, z);
}

var z = 1;

var it1 = foo();
var it2 = foo();

var val1 = it1.next().value; // 2
var val2 = it2.next().value; // 2

val1 = it1.next(val2 * 10).value; // 40
val2 = it2.next(val1 * 5).value; // 600

it1.next(val2 / 2); // 2 300 3
it2.next(val1 / 4); // 200 10 3

```

👆上面的逻辑有点绕，而且也没人会在真实的开发中这么做，但并不妨碍我们将它作为一个案例来研究多个 iterator 的实例能够实现互相影响的现象。

简单分析下过程：
1. `it1` 和 `it2` 是两个由 `*foo` 创建的 iterator 实例，它们第一次调用的 `next()` 方法并获取返回值中的 `value` 属性时，其结果都是 `2`；

2. `val2 * 10` 就相当于 `2 * 10`，其结果 `20` 作为参数传递给 `it1`，并将其赋值给变量 `x`，而后变量 `z` 自增的结果是 `2`，因此 `x * z` 的结果相当于 `20 * 2` 即 `40`，并将其作为返回值复制给 `val1`；

3. `val1 * 5` 相当于 `40 * 5` 即 `200`，并作为参数传入 `it2` 中，此时 `it2` 会经历 `it1` 的步骤，只是 `x` 和 `z` 数值发生了变化，分别是 `200` 和 `3`，因此最终 `val2` 的值变成了 `600`(来自 `yield (x * z)` 的返回值)；

4. `val2 / 2` 的结果 `300` 作为参数再一次传递到 `it1` 中，而后赋值给了 `y`，因此 `it1` 最终打印的结果是 `2 300 3`；

5. 同理可得 `val1 / 4` 的结果为 `40 / 4`，而后传入 `it2`，打印的结果为 `200 10 3`


#### 交错进行(interleaving)
回顾下之前提到的普通函数的运行模式：

```js
var a = 1;
var b = 2;

function foo () {
  a++;
  b = b * a;
  a = b + 3;
}

function bar () {
  b--;
  a = 8 + b;
  b = a * 2;
}
```

👆上面的代码无论是 `foo` 或 `bar` 先执行，最终都只能得到两种不一样的结果。但是，若我们使用 generator 函数改造一下，那就能得到很多不一样的结果：

```js
var a = 1;
var b = 2;

function *foo() {
  a++;
  yield;
  b = b * a;
  a = (yield b) + 3;
}

function *bar() {
  b--;
  yield;
  a = (yield 8) * b;
  b = a * (yield 2);
}
```

👆 `*foo()` 和 `*bar()` 不同的调用顺序会产生不一样的结果，而用这个特性来模拟 “线程竞争(threaded race conditions)” 也是可行的 —— 比如这个函数有副作用，依赖全局变量啥的：

先实现一个工具函数用来简化 iterator 的执行：

```js
function step (gen) {
  var it = gen();
  var last;

  return function () {
    last = it.next(last).value;
  }
}
```

接下来，仅仅是为了实验的目的，我们再实现因多个 iterator 交互而相互影响的例子：

```js
a = 1;
b = 2;

var s1 = step(foo);
var s2 = step(bar);

s1();
s1();
s1();

s2();
s2();
s2();
s2();

console.log(a, b); // 11 22
```

若是你真的理解了 generator 函数的执行机制，那得出结果并不困难。若是调换一下 `s1()` 和 `s2()` 的调用顺序的话，结果会又不一样：

```js
// ……
s1();
s2();
s2();
s1();
s2();
s1();
s2();
// ……
```

👆 通过 generator 改造，其最终可能的结果的个数，和普通函数只有两个结果相比起来大大增加。不过，通常我们并不会这么写代码，但这个 “并发” 的能力其实有大用处，后面细说。

## Generator生成的值(Generator'ing Values)
之前的一些尝试只是开胃菜，要搞清楚 generator 的原理，依然需要一步步的分析实现的细节，譬如我们先从 iterator 入手。

### 制造者和迭代器(Producers and Iterators)
想象一下有这个场景：你需要生成多个互相关联的值，当前值的状态依赖之前的值。为了实现这个需求，一般你需要一个状态管理器，来帮忙记住每一个之前的值的状态。

比如，可以借助闭包来实现：

```js
var giveMeSomething = (function () {
  var nextVal;

  return function () {
    if (nextVal === undefined) {
      nextVal = 1;
    } else {
      nextVal = (3 * nextVal) + 6;
    }

    return nextVal;
  };
})();

giveMeSomething(); // 1
giveMeSomething(); // 9
giveMeSomething(); // 33
giveMeSomething(); // 105
```

首先，这种方式👆的实现会有内存泄漏的问题；其次，若是用这种方式来 “记住” 的非简单类型的值，那用起来就很繁琐了。

实际上，对于上面的这种情况，在 JS 的实现中，已经有很通用的解决方案了：和大多数的语言一致，在 JS 中，实现一套调用 `next()` 方法，来获得返回的值的机制，即可满足调用状态的保留：

```js
var something = (function () {
  var nextVal;

  return {
    [Symbol.iterator]: function () {
      return this;
    },
    next: function () {
      if (nextVal === undefined) {
        nextVal = 1;
      } else {
        nextVal = (3 * nextVal) + 6;
      }

      return { done: false, value: nextVal };
    }
  }
})();

something.next().value; // 1
something.next().value; // 9
something.next().value; // 33
something.next().value; // 105
```

调用 `next()` 而得到的返回值是一个包含了两个属性的对象：`done` 和 `value` 分别代表是迭代完成的状态和本次迭代的值。

ES6 新增的 `for…of` 循环是一个标准的可自动迭代的循环语法：

```js
for (const v of something) {
  console.info(v);
  if (v > 500) {
    break;
  }
}
// 1 9 33 105 321 969
```

因为 `something` 迭代器返回的 `done` 始终为 `false`，因此只能手动在条件满足后，`break` 终止循环，否则就会一直循环下去。

`for…of` 循环在每次迭代的过程中，都会调用被循环对象的 `next()` 方法，但不会传递任何参数进去。当 `done` 为 `true` 的时候，自动结束循环迭代 —— 这整个机制十分适合用来遍历一组数据。

当然，你也能手动使用普通的 `for` 循环，来模拟这个过程：

```js
for (let ret = something.next(); !ret.done; ret = something.next()) {
  console.info(ret.value);
  if (ret.value > 500) {
    break;
  }
}
```

👆虽然没有 `for…of` 循环来的优雅，但好处是能够往 `next(…)` 中传递需要的值。

许多 JS 原生的值，都内置了一个默认的迭代器，比如数组(列表)：

```js
var a = [1, 3, 5, 7, 9];

for (const val of a) {
  console.info(val);
}
// 1 3 5 7 9
```

**Note**：ES6 中并没有为普通的 `object` 内置迭代器，其中一个缘由是没办法保证迭代的顺序，除此之外，若是普通对象内置了迭代器，那么其他继承自普通对象的其他数据类型都会受到影响。但有一个折中的办法能让普通的对象也能用上 `for…of` —— 调用 `Object.keys(…)` API，获取基于普通对象属性名的数组，而后在遍历它，并挨个取出属性值。

### 可迭代对象(Iterable)
`iterable` 可迭代对象就是一个包含了迭代器的 js 对象，比如上面提到的 `something` —— 它实现了 `next()` 方法以及定义了 `Symbol.iterator` 接口。

在 ES6 中，中可迭代对象中获取迭代器只能通过调用 `Symbol.iterator` 所定义的函数，当这个函数被调用时，应该返回一个迭代器。并且，虽然不是强制性的，但每次调用都应该返回一个全新的迭代器。

比如，在上例中的数组 `a`，当把它放到 `for…of` 循环中时，会自动调用它内置的 `Symbol.iterator`。既然如此，我们当然也能手动的调用它：

```js
var a = [1, 3, 5, 7, 9];

var it = a[Symbol.iterator]();

it.next().value; // 1
it.next().value; // 3
it.next().value; // 5
```

不知道你有没有注意到之前在定义 `something` 的时候，它的 `Symbol.iterator` 是这样实现的：

```js
[Symbol.iterator]: function () { return this; }
```

而后若是将它用在 `for…of` 中的话：

```js
var a = [1, 3, 5, 7, 9];

for (const val of something) {
  console.info(val);
  if (val > 500) break;
}
// 1 3 5 7 9
```

👆虽然看上去有点怪，但这行得通，因为当 `for…of` 调用 `something` 的时候，它确定义了 `Symbol.iterator`，并且返回了它自己 `this`，而它自己定义了 `next()` 方法，符合规范，于是接下去就能进行循环遍历了。

### 生成器中的迭代器(Generator Iterator)
生成器(generator) 从其功能上看，可以将其视为是一个生成迭代器的函数，该函数上定义了 `next()` 方法。

因此，从技术角度来看，generator 并非是一个可迭代的对象，尽管它们好像差不多，但当你调用 generator 的时候，它的返回值是一个迭代器：

```js
function *foo() {};

var it = foo(); // it 是迭代对象
```

当然，我们同样能够用 generator 来实现一个 `something`，用来执行一个无限循环的迭代：

```js
function *something() {
  let nextVal;

  while (true) {
    if (nextVal === undefined) {
      nextVal = 1;
    } else {
      nextVal = (3 * nextVal) + 1;
    }

    yield nextVal;
  }
}
```

**Note**：虽然非常不推荐，在真实的代码中使用不带任何 `break` 或 `return` 语句的 `while(true) {…}` —— 因为这样会造成浏览器的UI线程卡死；但在 generator 中，并没有这个问题，因为 generator 中每一次的 `yield` 会暂停 `while(true) {…}`，等到 JS主线程 的代码都执行完毕，才会再次执行。

由于每次 `yield` 都会保留 `*something()` 的调用栈，因此也用不着用一套很八股的模板，配合闭包(closure)，来完成对当前调用栈的状态记录 —— 这可不是仅仅简化了代码而已 —— 不仅不用自己手动定义 iterator 接口，而且代码也更语义化，能够清楚的表达它的职责所在。比如，`while(true) {…}` 循环就是告诉我们 generator 可以无限执行下去。

把 `*something()` 和 `for…of` 循环组合起来看看和之前的区别：

```js
for (const v of something()) {
  console.info(v);
  if (v > 500) {
    break;
  }
}
// 1 9 33 105 321 969
```

倘若是你仔细观察👆上面的代码，通常会有几个疑问：

  1. 为啥不直接调用 `for (const v of something) {…}` 而是 `const v of something()` 呢？

  2. 调用 `something()` 生成的是个迭代器，但为之前说过，可用 `for…of` 遍历的是一个可迭代的对象是吧？

针对第一个问题，`something` 仅仅是一个生成器，`for…of` 循环可不能遍历它；而第二个问题，generator 返回的迭代器同样定义了 `Symbol.iterator` 的函数，并且它的返回值依旧是 `this`。

#### 停止生成器(Stopping the Generator)
在之前的代码中，当 `*something()` 遇到 `break` 语句后，会一直保留其调用栈么？答案当然是否定的：当`for…of` 循环发生了 "非正常完结(Abnormal completion)" 后 —— 通常是由 `break`、`return` 或 未捕获的异常(uncaught exception) 导致 —— 此时 generator 就会接受到终止的信号。

**Note**：从技术角度讲，当正常的遍历结束是，`for…of` 循环会给迭代器发送一个终止信号。对于 generator 生成的迭代器，这样的结束信号并没有什么用，因为 generator 的迭代器会早一步 `for…of` 自动结束迭代。但是，对于自定义的迭代器，这个正常结束遍历信号，有时候就显得很有价值了。

`for…of` 会自动触发结束信号并发送给迭代器，若是想要手动触发的话，可以调用迭代器上定义的 `return(…)` 方法。

在 generator 中定义的 `try…finally` 在有一个妙用 —— 当触发结束信号的时候，你可以在 `finally` 语句中做一些收尾的工作，比如清除缓存、断开数据库链接啥的：

```js
function *something() {
  try {
    let nextVal;

    while (true) {
      if (nextVal === undefined) {
        nextVal = 1;
      } else {
        nextVal = (3 * nextVal) + 1;
      }

      yield nextVal;
    }
  } finally {
    console.info('cleaning up!');
  }
}
```

在外部手动的调用 `return()` 来结束迭代：

```js
const it = something();
for (const v of it) {
	console.info(v);

	if (v > 500) {
		console.info(it.return('Hello World').value);
	}
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```

`it.return(…)` 会立即结束 generator，而后执行 `finally` 里面的代码。传入的值 `'Hello World'` 会直接返回到属性 `value` 中，此时返回的 `done` 为 `true`，因此 `for…of` 会结束遍历。

以上种种，仅仅是 generator 使用的冰山一角。接下去我们要探讨的才是 generator 最核心的使用方式 —— 配合异步使用。

## 生成器异步的迭代(Iterating Generators Asynchronously)
那么，generator 到底解决了异步编程什么问题呢？

先快速回顾下之前我们使用的异步编程方式 —— 回调函数：

```js
function foo(x, y, cb) {
  ajax(
    'http://some.url.1/?x=' + x + '&y=' + y,
		cb
  );
}

foo(11, 31, function (err, txt) {
  if (err) {
    console.error(err);
  } else {
    console.log(txt);
  }
});
```

若是将上面👆的代码改写成 generator 来实现👇的话：

```js
function *main() {
  try {
    var txt = yield bar(11, 31);
    console.log(txt);
  } catch (err) {
    console.error(err);
  }
}

var it = main();

function bar(x, y) {
  ajax(
    'http://some.url.1/?x=' + x + '&y=' + y,
		function (err, data) {
      if (err) {
        it.throw(err);
      } else {
        it.next(data);
      }
    }
  );
}

it.next();
```

第一眼看上去，好像 generator 的实现的异步代码又臭又长。但千万别被表像迷惑，generator 的实现实际上要比回调的实现好很多，就比如对比下面这两段这段代码：

一个 `yield` 就能同步获取到内容：
  ```js
    var txt = yield bar(11, 31);
    console.log(txt);
  ```

而若是直接调用 `foo(…)` 的话，`txt` 却根本没办法拿到网络请求返回的内容：
  ```js
    var txt = foo(
      11, 31,
      function (err, data) {
        //……
      }
    );
    console.log(txt);
  ```

仅仅通过 `yield` 关键字，👆我们就实现了一段看上去像同步代码的异步编程，并且在 `bar(…)` 中屏蔽了异步的细节 —— 这解决了之前我们提及的 *回调函数的执行和我们大脑的工作方式不匹配* 问题。

本质上来看，这一切的实现是基于将异步代码的细节抽象出来，而后我们只用按照 *有序的流* 一样的同步的代码来实现具体细节即可。

### 同步代码的错误处理
既然是以同步的方式来书写的异步代码，那么对于错误的处理，`try…catch` 依然是不二的选择：

```js
try {
  var txt = yield bar(11, 31);
  console.log(txt);
} catch (err) {
  console.error(err);
}
```

这一切能够实现，都源于 `yield` 的暂停机制 —— 不仅能够暂停后续代码的运行，等待异步的回调函数 `return` 的值，而后恢复代码的执行；同样的，对于异步函数里所产生的错误，同样可以以同步的方式获取到；并且，无论这个错误是产生在 generator 函数内，还是函数外：

比如在 generator 函数外，通过迭代对象调用 `throw(…)` 方法，手动触发的错误：

  ```js
    function *main() {
      var x = yield 'Hello Generator';
      console.log(x);
    }

    var it = main();
    it.next();
    try {
      it.throw(42);
    } catch (err) {
      console.error(err); // 42
    }
  ```

又比如，直接在 generator 函数内产生的错误：

  ```js
    function *main() {
      var x = yield 'Hello Generator';
      yield x.toLowerCase(); // x 非字符串，调用 toLowerCase 方法会导致错误
    }

    var it = main();

    it.next();

    try {
      it.next(42); // x 为数字 42
    } catch (err) {
      console.error(err); // TypeError
    }
  ```

  或者在 generator 函数内部就搞定：

  ```js
    function *main() {
      try {
        var x = yield 'Hello Generator';
        var y = yield x.toLowerCase();
      } catch (err) {
        console.error(err); // TypeError
      }
    }

    var it = main();
    it.next();
    it.next(42);
  ```

无论是手动触发的错误，还差 generator 函数自身的错误，通过 `try…catch` 的捕获，在 generator 内部，就有处理错误的机会；但是当其内部没有错误处理的时候，在 generator 函数的外部一定要搞定这些。

在 generator 中，能够通过 `try…catch` 来实现同步的错误处理，在可读性和可维护性上都是一个巨大的胜利。

## Generators + Promises
在之前的讨论中，我们看到 generator 相对于意大利面条式的回调地狱，在按照大脑工作的心智模型之上 —— 按顺序理解 —— 有了巨大的进步，但相对之前 Promise 带来的可信任和可组合的提升却束手无措。

显然，若是将 generator + Promise 组合 —— 用同步模式来书写异步代码 + 可信任和可组合 —— 是压死 回调函数 的最后一根稻草。

比如之前使用 Promise 来实行 Ajax 请求的代码：

```js
function foo (x, y) {
  return request("http://some.url.1/?x=" + x + "&y=" + y);
}

foo(11, 31)
  .then(
    function (txt) {
      console.log(txt);
    },
    function (err) {
      console.error(err);
    }
  );
```


👆若是用 generator 对其进行改造的话，比如用 `yield` 关键字将 `foo(…)` 中创建并返回 Promise 拿到，那 iterator 迭代器就能拿到这个 Promise。

但是 iterator 应该对 Promise 做点啥呢？

当然应该监听 Promise 的 resolve，而后继续恢复 generator 的执行，并且将 fulfill 或 reject 的信息返回出来。

简单地重复一遍，`yield` 一个 Promise 并且用 generator 的迭代器去控制 Promise，才能发挥这个组合的最大潜能。

试一下：

```js
function foo (x, y) {
  return request("http://some.url.1/?x=" + x + "&y=" + y);
}

function *main () {
  try {
    var txt = yield foo(11, 31);
    console.log(txt);
  } catch (err) {
    console.error(err);
  }
}
```

`*main()` 最牛逼的地方在于对于 `foo()` 内部的逻辑 **啥都不用改** —— 无论 `yield` 拿到是何值。

不过后续代码还需要手动执行：

```js
var it = main();

var p = it.next().value;

p.then(
  function (txt) { it.next(txt); },
  function (err) { it.throw(err); }
)
```

👆不需要使用 `error-first callback` 来特别的在 generator 中处理异常，因为 Promise 已经清晰地将 fulfillment 和 rejection 分开了。

不过到目前为止，上面的实现依旧有些问题需要解决：

  1. 如何高效地执行多个 Promise —— 我们肯定不想要手动的管理 Promise 的链式调用，最好能有一个工具帮我们自动的调用 generator 返回的迭代器；

  2. 在执行 `it.next(…)` 时，若发生了异常，如何处理？退出整个 generator 的执行？或是捕获到错误而后直接作为值返回？那要是在 generator 函数中显示的调用了 `it.throw(…)` 呢？

### 一个由Promise和Generator结合而成的Runner(Promise-Aware Generator Runner)
在越发深入的探索之后，显然一个能自动帮你搞定 Promise + Generator 组合的工具会显得十分有必要。

有很多第三方的库提供了这样的功能，但是为了学习起见，还是要手动的定义一个简单的 `run(…)` 来做展示：

```js
function run (gen) {
  // 其他参数处理
  var args = [].slice().call(arguments, 1);

  // 调用 generator 获得 iterator 迭代器
  var it = gen.apply(this, args);

  // 首次调用 iterator 并不需要参数
  return Promise.resolve()
    .then(function handleNext(value) {
      var next = it.next(value);

      return (function handleResult(next) {
        if (next.done) {
          // 迭代结束，直接返回值
          return next.value;
        } else {
          return Promise.resolve(next.value)
            .then(
              // 递归调用handleNext，并将当前值直接传递进去
              handleNext,
              // 若是返回一个 rejected 的 Promise
              // it.throw(err) 会进入到自定义的错误处理中，它的返回值
              function handleErr(err) {
                return Promise.resolve(it.throw(err)).then(handleResult);
              }
            )
        }
      })(next);
    });
}
```

用 `run(…)` 来运行之前的 `*main()` 的话，就能实现自动完成所有的迭代工作：

```js
run(main);
```

#### ES7: `async` and `await`
`run()` 能够自动的完成一些重复琐碎的工作，不过在 ES7 中落地的 `async` 和 `await` 是JS内置的语法糖，能够更为简练的搞定这一切：

```js
function foo (x, y) {
  return request("http://some.url.1/?x=" + x + "&y=" + y);
}

async function main() {
  try {
    var txt = await foo(11, 31);
    console.log(txt);
  } catch (err) {
    console.error(err);
  }
}

main();
```

Tip: JS 的 `async`/`await` 的语法，其实是借鉴了 C# 中的思想，而且本质上来看也是在解决相同的问题。

`main()` 虽然和普通的函数调用没啥两样，但在其内部的 `await` 会阻塞后面代码的执行，只有在它后面的 Promise 的状态确定后，才会恢复执行。

### Promise 在 Generator 中的并发执行(Promise Concurrency in Generator)
之前我们讨论的都是单个的异步任务，但是在现实中，多个异步任务在一个函数中执行是在是太常见了，而且若是不当心的话，很可能你会得写出一段性能很差的代码 —— 比如有个场景要求你先发送两个异步的请求获取到不同的数据，而后将它们组装在一起，再发送第三个请求拿到最终的数据：

```js
function *foo (x, y) {
  var r1 = yield request("http://some.url.1");
  var r2 = yield request("http://some.url.2");

  var r3 = yield request("http://some.url.3/?" + r1 + r2);

  console.log(r3);
}

run(foo);
```

👆这段用 Generator 实现的异步代码的问题很明显 —— `r1` 和 `r2` 是相互独立的，因此明明可以并发执行，但在最终却是串行 —— `r2` 要等到 `r1` 执行完毕后才会继续执行。但问题在于说，`yield` 只能暂停一个点，不能同时作用于多个点……

一个简单的解决办法是使用 *时间独立(time-independent)* 的思路：

```js
function *foo (x, y) {
  var p1 = request("http://some.url.1");
  var p2 = request("http://some.url.2");

  var r1 = yield p1;
  var r2 = yield p2;
  var r3 = yield request("http://some.url.3/?" + r1 + r2);

  console.log(r3);
}

run(foo);
```

👆本质上来看，因为发送任务 `request(…)` 的过程可以看做是并发的，因此在仅仅改变了 `yield` 的作用对象的时机，就能解决这个问题。

事实上，使用 `Promise.all([…])` 同样也能解决这个问题：

```js
function *foo (x, y) {
  var results = yield Promise.all([
    request("http://some.url.1"),
    request("http://some.url.2")
  ])

  var r1 = results[0];
  var r2 = results[1];
  var r3 = yield request("http://some.url.3/?" + r1 + r2);

  console.log(r3);
}

run(foo);
```

👆这其实是将原本多个异步任务都放到一个大的异步任务中，而后通过 `yield` 整个大的异步任务，就完成了并发的异步任务。

#### 细节的隐藏(Promise, Hidden)
屏蔽细节，特别是在 Generator 中的 Promise 实现相关的细节，从代码的可读性和可维护性都是一个更好的选择：

```js
function bar(url1, url2) {
  return Promise.all([
    request(url1),
    request(url2)
  ]);
}

function *foo (x, y) {
  var results = yield bar("http://some.url.1", "http://some.url.2");

  var r1 = results[0];
  var r2 = results[1];
  var r3 = yield request("http://some.url.3/?" + r1 + r2);

  console.log(r3);
}

run(foo);
```

**我们把异步的代码，即 Promise 的部分，视为实现的细节。**

无论 Promise 相关的逻辑有多么复杂，在 Generator 中你只需要关心异步执行的步骤 —— 因为写一段代码除了让其能够正常的工作并且具有不错的性能之外，还要考虑到代码的易读性和可维护性。

**Note**：对代码进行更高维度的抽象，不一定百分之百都是好事，因为这有事也会极大的增加代码的复杂程度，降低了代码的简洁性。不过还是那句话，凡是都没那么绝对，根据自身和团队的真实情况来做出选择，显然是更为合理的。

## Generator 代理(Generator Delegation)
利用 `run(…)` helper，我们能够实现在 generator 函数中运行另一个 generator 函数：

```js
function *foo () {
  var r2 = yield request('http://some.url.2');
  var r3 = yield request('http://some.url.3?v=' + r2);

  return r3;
}

function *bar() {
  var r1 = yield request('http://some.url.1');
  var r3 = yield run(foo);

  console.log(r3);
}

run(bar);
```

但更好的方式并不是用 helper 来帮忙完成 generator 的自动执行，而是用内置的 `yield *__` 代理机制：

```js
function *foo () {
  console.log('`*foo()` starting');
	yield 3;
	yield 4;
	console.log('`*foo()` finished');
}

function *bar() {
  yield 1;
	yield 2;
	yield *foo();
	yield 5;
}

var it = bar();

it.next().value; // 1
it.next().value; // 2
it.next().value; // `*foo()` starting
                            // 3
it.next().value; // 4
it.next().value; // `*foo()` finished
                            // 5
```

借用👆上面的例子，我们来看看关于 `yield *foo()` 代理实现的基本过程：

- 首先，前两次的 `it.next()value` 的执行，迭代器迭代的对象是 `bar()` 这个 generator 函数；

- 而后在第三次执行 `it.next().value` 时，在 `bar` 的内部对 `foo()` 的调用，本质上是创建了一个 iterator 迭代器，而后又由 `yield *` 自动的将 `foo()` 返回的这个迭代器的控制权，交到了外层由 `bar()` 创建的迭代器手中；

- 紧接着，我们执行了第五次 `it.next().value`，此时 `foo()` 返回的迭代器已经迭代完成，因此控制权再次交由 `bar()`，而后结束整个迭代。

因此，若是将之前一开始的三个网络请求用 generator 代理的方式来实现的话：

```js
function *foo () {
  var r2 = yield request('http://some.url.2');
  var r3 = yield request('http://some.url.3?v=' + r2);

  return r3;
}

function *bar() {
  var r1 = yield request('http://some.url.1');
  var r3 = yield *foo();

  console.log(r3);
}

run(bar);
```

从写法上来看，唯一的区别是用 `yield *foo()` 替换掉了 `yield run(foo)`。

**Note**：需要注意的是，`yield *` 并不是代理 generator 函数，而是 iterator 迭代器，这也就意味着 `yield *[1,2,3]` 是完全合法的。

### 为啥要用代理(Why Delegation)?
generator 的代理机制，最主要的功能是让基于 generator 代码的可读性、可维护性得到巨大提升。

想象一下一个常见的场景：将一个涵盖了异步交互的业务功能，拆分成两个部分为 `foo` 和 `bar`，并用 generator 进行了封装。有的时候 `foo` 单独调用，有的时候，需要在 `bar` 中执行 `foo` 的逻辑，因此这样的拆分很合理。

而若是没有 `yield *` 的语法，想要实现在 `bar` 中 执行 `foo`，那就少不了用到类似 `run(…)` 这样的 helper 函数。这无疑增加了程序的复杂程度，降低了代码的可维护性。

### 代理信息(Delegation Messages)
看下面的例子了解下 generator 函数，和它生成的迭代器，是如何在执行的过程中，实现信息的交互的：

```js
function *foo() {
  console.log('inside *foo:', yield 'B');

  console.log('inside *foo:', yield 'C');

  return 'D';
}

function *bar() {
  console.log('inside *bar:', yield 'A');

  console.log('inside *bar:', yield *foo());

  console.log('inside *bar:', yield 'E');

  return 'F';
}

var it = bar();

console.log('outside:', it.next().value);
// outside: A

console.log('outside:', it.next(1).value);
// inside *bar: 1
// outside: B


console.log('outside:', it.next(2).value);
// inside *foo: 2
// outside: C

console.log('outside:', it.next(3).value);
// inside *foo: 3
// inside *bar: D
// outside: E

console.log('outside:', it.next(4).value);
// inside *bar: 4
// outside: F
```

👆其他的过程都还好说，重点分析下当调用 `it.next(3)` 时，实际发生的整个过程：

  1. 数字 `3` 被当做值传递进了 `*bar()` 中，通过 generator 代理，进而传递给了 `*foo()` 的迭代器，并最终作为 `yield 'C'` 生成的值；

  2. 接下去，在 `*foo()` 中执行的是 `return 'D'` 这段语句，但返回值 `D` 并不会成为 `it.next(3)` 的值，而是作为 `*yield *foo()` 生成的值；

  3. 继续下去，当遇到 `yield 'E'` 的时候，generator 函数又会停止执行，并把 `E` 作为 `it.next(3)` 的 `value` 输出……

从外部 iterator(it) 的使用方式来看，在处理被代理的 generator 和其本身的时候，并没有什么区别。

实际上，`yield`-delegation 在处理非 generator 函数而是一个定义了 iterator 接口的可迭代对象时，也不会有任何区别：

```js
function *bar() {
  console.log('inside *bar:', yield 'A');

  console.log('inside *bar:', yield *['B', 'C', 'D']);

  console.log('inside *bar:', yield 'E');

  return 'F';
}

var it = bar();

console.log('outside:', it.next().value);
// outside: A

console.log('outside:', it.next(1).value);
// inside *bar: 1
// outside: B

console.log('outside:', it.next(2).value);
// outside: C

console.log('outside:', it.next(3).value);
// outside: D

console.log('outside:', it.next(4).value);
// inside *bar: undefined
// outside: E

console.log('outside:', it.next(5).value);
// inside *bar: 5
// outside: F
```

唯一的区别在于，`array` 默认的迭代器没有针对消息传递做任何的处理，因此导致的结果是任何通过 `next(…)` 传递的参数都被无视了，并且没有任何的 `return` 值导致 `yield *` 表达式只能获得 `undefined` 作为返回值(`inside *bar: undefined`)。

#### 代理的异常处理(Exception Delegated, Too!)
异常处理在 `yield`-delegation 中当然也不能缺席：

```js
function *foo() {
  try {
    yield 'B';
  }
  catch (err) {
    console.log('error catch inside *foo:', err);
  }

  yield 'C';

  throw 'D';
}

function *bar() {
  yield 'A';

  try {
    yield *foo();
  }
  catch (err) {
    console.log('error catch inside *bar:', err);
  }

  yield 'E';

  yield *baz();

  return 'G';
}

function *baz () {
  throw 'F';
}

var it = bar();

console.log('outside:', it.next().value);
// outside: A

console.log('outside:', it.next(1).value);
// outside: B

console.log('outside:', it.throw(2).value);
// error catch inside *foo: 2

console.log('outside:', it.next(3).value);
// error catch inside *bar: D
// outside: E

try {
  console.log('outside:', it.next(4).value);
}
catch (err) {
  console.log('error catch outside:', err);
}
// error catch outside: F
```

有几点需要强调的是：
  1. 当调用 `it.throw(2)` 的时候，数字 `2` 被作为 error message 传递到 `*bar()` 中，而后被 `catch` 捕获到。最终 `yield 'C'` 将 `C` 作为 `it.throw(2)` 的 `value` 返回；

  2. `* foo()` 最后的语句是 `throw 'D'`，而这个 error message 同样也被外层 `* bar` 的 `catch` 捕获到，而 `yield 'E'` 会继续将 `E` 作为 `it.next(3)` 的 `value` 值返回；

  3. 而后继续执行 `it.next(4)`，这一次代理了 `*baz`，而它直接 `throw 'F'`，并且这一路上没有任何的 `try…catch` 处理，因此最终冒泡到最外部的 `try…catch` 中，而此时 `*baz` 和 `*bar` 的状态都是执行完成，因此接下去再调用 `it.next()` 返回的 `value` 不会是 `G`，而只能是 `undefined`。

### yield 代理的递归(Delegation "Recursion")
在 generator 函数中，利用 `yield`-delegation 递归的调用自己，也是合理合法的一件事：

```js
function *foo(val) {
  if (val > 1) {
    val = yield *foo(--val);
  }

  return yield request('http://some.url?v=' + val)
}

function *bar() {
  var r1 = yield *foo(3);

  console.log(r1);
}

run(bar);
```

别看代码行数很短，但代码执行的过程十分复杂，详细的步骤分析如下：
  1. `run(bar)` 开始执行 `*bar`；

  2. `*foo(3)` 代理 generator `foo`，而后创建一个迭代器，并将 `3` 作为参数传递；

  3. `3 > 1` -> `foo(2)` -> 又新创建了一个迭代器，将 `2` 作为参数递归调用 `*foo(2)`；

  4. `2 > 1` -> `foo(1)` -> 又新创建了一个迭代器，将 `1` 作为参数递归调用 `*foo(1)`；

  5. `1 > 1` 为 `false`，因此调用 `request('http://some.url?v=' + 1)`，并返回一个 Promise；

  6. 这个 Promise 被 `yield` 返回，到了 `*foo(2)` 的手上；

  7. 继续下去，这个 Promise 再次被 `yield` 传递到 `*bar` 的 `*foo(3)` 手中，而此时 `*bar` 是作为 `run` 的参数被其实例化成一个 iterator 迭代器在自动执行。因此，这个 Promise 是作为某次调用 `next()` 的 `value` 被放到 `Promise.resolve(next().value)` 中等待 Ajax 请求完成，进而再执行后续的 `then()`；

  8. 当这个 Ajax 完成时，意味着 Promise 被 resolve 了，那么在 `run` 中会继续调用 `it.next()`，进而恢复 `*bar()` 的执行，并把网络请求的返回值作为参数传递到 `*foo(3)` 生成的 iterator 中，这个参数沿着代理链路一直传递到 `*foo(2)` 中，并作为 `yield` 的值赋值给 `val` 变量；

  9. 接下去第二个 Ajax 网络请求又会发起，同样这个 Promise 又会层层传递，先经过 `*foo(3)` 手中，最终到 `Promise.resolve(…)` 中，而后当这个请求返回时，它的返回值又会作为参数层层传递，并被赋值给 `val`，而后发起第三个 Ajax 请求；

  10. 最终，当第三个请求也完成后，它的返回值会被作为 `yield *foo(3)` 所生成的值赋值给 `r1`，而后被打印出来，最终结束整个 `run` 的运行。

## Generator 的并发(Generator Concurrency)