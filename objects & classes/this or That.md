# `this` Or That
`this` 被一些JS的程序员视为是最让人困惑的机制，这个关键字会在每个函数的内部作用域中自动被定义。

## Why `this`?
在我们问如何使用 `this` 之前，我们应该先问为什么要有 `this`？

```js
function identify () {
  return this.name;
}

function speak () {
  var greeting = 'Hello, I am ' + identify.call(this);

  console.log(greeting);
}

var a = {
  name: 'Lee'
};

var b = {
  name: 'Bob'
};

identify.call(a); // Lee
identify.call(b); // Bob

speak.call(a); // Hello, I am Lee
speak.call(b); // Hello, I am Bob
```

👆这段代码借助 `this`，能让我们在 `a` 和 `b` 两个不同的 *上下文(context)* 之间自由的切换，复用已有的逻辑。

当然，你能选择使用显示的将 *上下文* 作为参数传入，而后完全回避 `this` 的问题：

```js
function identify (context) {
  return context.name;
}

function speak (context) {
  var greeting = 'Hello, I am ' + identify(context);

  console.log(greeting);
}

var a = {
  name: 'Lee'
};

var b = {
  name: 'Bob'
};

identify(a); // Lee
identify(b); // Bob

speak(a); // Hello, I am Lee
speak(b); // Hello, I am Bob
```

嗯，👆看上去好像是可行的方案。但一旦你的程序越来越复杂，代码越来越多时，你会发现使用 `this` 来自由的切换 *上下文* 所带来的便利远超显示的将 *上下文* 作为参数来使用。

## 困惑(Confusions)
在正式开始之前，我们还需要消除两个常见的误区，特别是当我们从字面的意思去理解 `this`时，通常会有两种设想，但都是不对的。

### Itself
第一种常见的假设是认为 `this` 指代的就是函数本身。虽然从语义上来讲貌似合理，且不说能从函数名直接获取对函数的引用，但为什么我们需要用 `this` 来指代函数自身呢？是要进行 *递归(recursion)* 操作还是在某个事件的回调函数中，操作它自身以便解除事件的绑定？

很多开发者都认同在JS中 *一切皆是对象* 的观念 —— 即便是函数，也不过是一种数据储存的对象形式而已 —— 如果从这样的角度去认知和使用函数，那就太局限了。

下面的代码打破了我们的假设：
```js
function foo (num) {
  console.log('foo: '+ num);

  this.count++;
}

foo.count = 0;

for (let i = 0; i < 5; i++) {
  foo(i);
}

console.log(foo.count); // 0

console.log(count); // NaN
```

👆 `foo.count` 仍旧是 `0`，即便在我们执行了一次 `for` 循环，调用了5次 `foo` 后，`this.count++;` 并没有对 `foo.count` 产生一丝一毫的影响。

那到底 `this.count++;` 对谁造成了影响呢？答案是全局作用域下的变量 `count`。你可能疑惑了 —— 我都没声明这个变量，而且为什么它的值是 `NaN`？那你可能需要回到 [作用域和闭包](../scope%20%26%20closures/README.md) 这一章再仔细阅读了。

大部分的开发者可能采取回避的措施，借用 *词法作用域* 的机制来完成实际的需求：
```js
var data = {
  count: 0
};

function foo (num) {
  console.log('foo: '+ num);

  data.count++;
}

for (let i = 0; i < 5; i++) {
  foo(i);
}

console.log(data.count); // 5
```

这种回避 `this` 到底如何工作的鸵鸟策略，只会让你待在舒适区止步不前！想要在函数内部获取自身，单单靠 `this` 是不够的，你还是需要词法作用域的帮助：
```js
function foo () {
  foo.count = 5;
}

foo();
console.log(foo.count); // 5
```

但如果是一个匿名函数，那就有点麻烦了：
```js
setTimeout(function () {
  // cannot refer to itself
});
```

一个 *非常不赞成(deprecated and frowned-upon)* 使用的非常老旧的API能够解决👆匿名函数的问题 —— `arguments.callee`。它能够在函数内部指向函数自身，这也是通常匿名函数用来访问自身的唯一途径。然而，正确的姿势不是使用匿名函数而后调用 `arguments.callee` —— 你应该直接为需要调用自身的函数绑定一个函数名。

所以，另一种解决办法是直接获取对 `foo` 函数的引用，然后对 `foo.count` 进行操作：
```js
function foo (num) {
  console.log('foo: '+ num);

  foo.count++;
}

foo.count = 0;

for (let i = 0; i < 5; i++) {
  foo(i);
}

console.log(foo.count); // 5
```

不过，还有一种办法能让你使用了 `this` 的同时，还能修改 `foo.count` —— 用 `call` 内置方法显示的绑定 `this` 的指向：
```js
function foo (num) {
  console.log('foo: '+ num);

  this.count++;
}

foo.count = 0;

for (let i = 0; i < 5; i++) {
  foo.call(foo, i);
}

console.log(foo.count); // 5
```

### Its Scope
另一个关于 `this` 的误区是认为它指代 *函数的作用域*，这是个棘手的问题(tricky question)，因为有的场景中是正确的，但有的场景中反而会成为误导。

`this` 无论从哪个方面来讲，都不能指代 *词法作用域(lexical scope)* —— 因为词法作用域虽然犹如一个对象一般具有各种属性能够被访问，但其本质是JS引擎的内部实现，而非真实的能够被JS代码方位的一个对象。

下面的代码显然以为 `this` 可以访问词法作用域：
```js
function foo () {
  var a = 2;
  this.bar();
}

function bar () {
  console.log('bar: ', this.a);
}

foo(); // bar: undefined
```

显然，`this.bar();` 的确调用了函数 `bar`，但显然这种调用的思路是基于 `this` 能够访问全局作用域 —— 那干嘛不直接使用词法作用域来调用 `bar();` 呢？其次，在函数 `bar` 中使用 `this.a` 的企图是通过 `this` 来搭个桥 —— 在函数 `foo` 中写出 `this.bar();`，想让 `this` 来指代 `foo` 的函数作用域，以便访问到变量 `a` —— 想太多！！

`this` 不能作为词法作用域的指代！！`this.bar();` 能够访问只不过是将全局作用域作隐式的为上下文绑定到 `this` 上而已，仅此而已！

## What's `this`?