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
除了代表 “未来的值” 之外，对 Promise 的另一个洞见是把它视为是一个 “流程控制” 的机制 —— 将异步的任务拆解为多个步骤用来执行。

假设有一个函数 `foo` 会执行一些任务，但我们并不知道它执行的细节，以及它到底是立即执行还是延迟一会后再执行，我们只知道它会通知我们它已经完成，而后我们可以接下去做后续的任务 —— 在 JS 的编程范式中，你通常需要监听这些 “通知”，就好像事件触发的机制一样，我们能够自定义一些需求，打包成一个回调函数，并将回调函数传递给 `foo`；最终 `foo` 会根据情况触发这个回调函数，以此来完成我们的需求。

不过在 Promise 的机制下，这样的关系被反转了 —— 我们的回调函数不再是被 `foo` 触发的，而是通过 Promise 这个中间层来触发，比如下面的一段伪代码：

```js
foo(x) {
	// 异步操作……
}

foo(42)

on (foo 'success') {
	// foo 完成了，接着做下一步事情
}

on (foo 'error') {
	// foo 发生了点问题，处理一下
}
```

`foo(…)` 不必关心，也不用要处理一些订阅它的事件，这些都交给了 `on(…)`，这是一种很好的 "责任分离(separation of concerns)" 的实现。

但在 Promise 出来之前，在 JS 中想要实现这样的效果，只能 “魔改” 一把：

```js
function foo(x) {
	// 异步操作……

	// 返回一个 listener 对象，它有一个 `on` 方法，能够实现事件的绑定
	return listener;
}

const evt = foo(42);

evt.on('success', function(){
	// foo 完成了，接着做下一步事情
});

evt.on('error', function(err){
  // foo 发生了点问题，处理一下
});
```

👆 `foo(…)` 返回了一个对象能够处理事件的订阅，如此一来我们就能像事件机制一样来完成对 `foo` 后续任务的处理。但说到底，`listener` 依然是由 `foo` 内部实现，并没有颠覆 “控制反转” 的底层逻辑。

当然，你还能将监听 `foo` 的 `success` 和 `error` 事件分开，单独封装成 `bar` 和 `baz`：

```js
// `bar` 监听 `foo` 的 success
bar(evt);

// `baz` 监听 `foo` 的 error
baz(evt);
```

若是 `evt` 或者说 `listener` 是由一个可靠的第三方机制来生成的话，那的的确确实现了 “反控制反转(Uninversion of control)”，并且加上 “责任分离” 的效果，看上去真的很 nice。

#### Promise“事件”(Promise "Events")
其实 `evt` 就是对 Promise 的一个类比，在 Promise 的机制下，`foo(…)` 会返回一个 Promise 的实例，而后这个实例会作为参数传递给 `bar(…)` 和 `baz(…)`。

**Note**：需要注意的是之前对于事件的措辞，严格来讲，Promise 的实现并不是通过事件机制，更没有 `success` 和 `error` 之类的事件监听；不过你也能将调用其实例上的 `then(…)` 和 `catch(…)` 方法，理解成注册了 `fulfillment` 或 `rejection` 事件 —— 虽然我们不会在真实的代码中展示出来：

```js
function bar (fooPromise) {
	fooPromise.then(
		function(){
			// resolve 监听
		},
		function(){
			// reject 监听
		}
	);
}

// baz 同上

function foo(x) {
	// 异步操作……

	// 返回一个 Promise 的实例
	return new Promise(function(resolve, reject){
		// 内部调用 `resolve` 或则 `reject`，最终会触发绑定的回调
	});
}

var p = foo(42);

bar(p);

baz(p);
```

👆 `new Promise(function (resolve, reject) {})` 这种模式被称为 [revealing constructor](https://blog.domenic.me/the-revealing-constructor-pattern/) —— 被传入 Promise 构造函数的函数会被立即执行，并接受两个参数，一个是 `resolve`，调用它会产生 fulfillment 的状态；另一个是 `reject`，调用它会产生 rejection 的状态。

另一种更优雅实现的方式，将操作 `resolve` 和 `reject` 的代码分离，而不是耦合在一起：

```js
function bar() {
	// resolve 监听
}

function oopsBar() {
	// reject 监听
}

// `baz()` 和 `oopsBaz()` 同上

var p = foo(42);

p.then(bar, oopsBar);

p.then(baz, oopsBaz);
```

**Note**：`p.then(…).then(…)` 和 `p.then(…);p.then(…)` 是两个完全不一样的代码，前者的链式调用中，前后两个 `then(…)` 是不同的 Promise 实例上的方法；后者则都是出自同一个 Promise 实例的 `then(…)` 方法，并且它的状态一旦确定，就无法被改变了。


## Then的鸭式类型(Thenable Duck Typing)
使用 Promise 面临的最头疼的问题是如何确定某个对象是不是真正的 Promise。

`p instanceof Promise` 这样的检测方式没办法完全满足需求的主要原因是，浏览器窗口之间(iframe)的交互，若是它们使用的 Promise实例 是不同的构造函数创建的，那这样的检测就会失败。

再者，若是你使用了某个三方的库，它的 Promise 并非是基于原生的 Promise 来实现，那么也无法满足需求。

因此，社区内广泛流行的一种 类型检测 机制是基于 “鸭式类型(duck typing)” —— 如果它长得像鸭子，并且能像鸭子一样呱呱的叫，那么它就是鸭子。这种基于 “值的形状” 的检查，被用于 “Promise的真假美猴王” 之中，即所谓的 thenable：

```js
function isPromise (p) {
  if (p !== null && (typeof p === 'object' || typeof p === 'function') && typeof p.then === 'function') {
    // thenable!
    return true;
  }
  return false;
}
```

好像还行？立马露馅：

```js
var o = { then: function() {} };

var v = Object.create(o);

v.hasOwnProperty('then'); // false

isPromise(v); // true
```

而若是有人 无意/故意/恶意 在一些内置的类的原型对象上面添加了 `then` 方法的话：

```js
Object.prototype.then = function(){};
Array.prototype.then = function(){};

var v1 = { hello: 'world' };
var v2 = [ 'Hello', 'World' ];

isPromise(v1); // true
isPromise(v2); // true
```

👆要是你把这些对象作为参数传递给之前的 `baz`、`bar` 的话，那它们会被永远的挂起……

你可能会觉得，这是危言耸听吧？！遇到这种情况的概率实在是太小了。但你不得不防的是有一些你依赖的第三方的库，它们很可能是在 ES6 之前的产物，并且有 `then` 方法挂载在它的某个对象上。遇到这种情况，`isPromise` 就不好使了。

## Promise 和 信任问题(Promise Trust)