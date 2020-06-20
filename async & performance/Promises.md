# Promises
通过前面一节的深入探讨，我们已经很清楚 回调函数 所面临的问题了 —— 书写的反人性和信任问题。显然最为头疼的是由于 *控制反转(inversion of control)* 所带来的信任问题。幸好在 ES6 中已经提供了内置的 API 来解决这个问题 —— Promise。

## Promise是什么？(What Is a Promise?)
许多开发者学习一个新技术的第一步通常是直接撸代码，搞出一个 “Hello World” 什么的。当然这没什么不好，不过想要深入理解 Promise，以及它背后所隐含的理论，不妨先从两个类比开始：

### 未来的值(Future Value)
想象一个在现实世界中常见的场景：当你走进一家快餐店(KFC啥的)的前台，准备买一个汉堡套餐来解决午饭时，你通常是先付钱，而后拿到一张小票，上面有一个数字，代表的就是“我欠你一份汉堡套餐”的 *承诺(promise)* —— 通常情况下，你最终会凭此拿到一份汉堡套餐。

在等待汉堡套餐的时间里，你会干什么呢？打开手机翻翻朋友圈？给某人发个消息说待会吃完中饭去找他聊聊天？或者浏览下最新的热点新闻？……反正不会瞎等在那里就对了 —— 而在这自信满满的等着的汉堡套餐，就是所谓的 **future value**。

时间溜走了，当 “113号，请取餐” 的声音响起，你充满期待的去前台准备接下去的大快朵颐时，此时会有两个情况等着你，第一种会按照你的期待进行，自不必说；另外一种小概率但未必不会发生的事件是：“很抱歉，我们的汉堡卖完了，您看要不来份三明治怎么样？” —— 每次当你想买汉堡套餐时，你都得做好失败的准备。

**Note**：这里的类比是个简化的场景，显然在写代码的时候，要考虑更多的事情，比如你的号码一直都没被叫(很有可能是被忘记了)，这与之对应的就是一直 “悬而未决” 的状态。

#### 现在和将来的值(Values Now and Later)
直接来段代码：

```js
var x, y = 2;

console.log(x + y); // NaN
```

👆 `x + y` 最终产生的是一个与预期结果不符的 `NaN` —— 难道你还指望 `+` 帮你监视着 `x` 和 `y` 都已经是 `number` 类型的值了，而后再帮你完成加法运算？

为了解决这种问题，你得靠自己，比如用回调函数：

```js
function add (getX, getY, cb) {
  var x, y;
  getX(function (xVal) {
     x = xVal;
     if (y != undefined) {
       cb(x + y);
     }
  }),
  getY(function (yVal) {
     y = yVal;
     if (x != undefined) {
       cb(x + y);
     }
  })
}

// `fetchX()` 和 `fetchY()` 是同步或异步的获取对应 x 和 y 值的函数
add(fetchX, fetchY, function (sum) {
  console.log(sum);
})
```

👆先别管它是否优雅，至少它能让行为可控，即在`fetchX()` 和 `fetchY()` 工作正常的情况下，无论它们是同步还是异步，`x + y` 都能正确的返回值期待的值，而非 `NaN`。

#### 承诺的值(Promise Value)
若是把上面的 `add` 用 Promise 来改写的话：

```js
function add (xPromise, yPromise) {
  return Promise.all([xPromise, yPromise])
    .then(function (values) {
      return values[0] + values[1];
    });
}

add(fetchX(), fetchY())
  .then(function (sum) {
    console.log(sum);
  });
```

👆`fetchX()` 和 `fetchY()` 都返回了 Promise 对象，而后被作为参数传入了 `add(…)` 中。

在 `add(…)` 内部，`Promise.all([…])` 接收了它们，随后它也返回一个 Promise 对象，而后在其链接的 `then(…)` 方法中，完成了 `x + y`(`values[0] + values[1]`) 的动作，并作为立即 resolve 的值返回。

最终，在调用 `add(…)` 返回的 Promise 对象后面，链接上 `then(…)` 方法，并将 `sum` 输出到了控制台上。

关键的地方是 Promise 对象本身包含了 `then` 方法，而调用 `then(…)` 方法本身又会返回一个 Promise 对象，因此这个链可以无限进行下去。

和买汉堡套餐的例子一样，我们得对意外的情况做好准备，在 Promise 中，对应的就是 `then(…)` 方法的第二个参数：

```js
add(fetchX(), fetchY())
  .then(
    function (sum) {
      console.log(sum);
    },
    function (err) {
      console.error(err);
    }
  );
```

因为 Promise 是独立于时间的，因此它可以很好的预测和组合各种情况和结果。并且更重要的是，一旦 Promise 的状态 resolve 掉了，就不能更改了 —— 尽管这个状态的结果能够被无限次的访问。

**Note**：这种 *不可更改(immutable)* 的特性，使其能够被任意的传入到各种第三方的代码中，而不必担心被无意或是恶意的篡改。而这是 Promise 设计之初，殚精竭虑完成的最基础、最重要的功能。

Promise 是一种能够很容易的封装、组合 *未来的值(fature value)* 的机制。

### 完成事件(Completion Event)