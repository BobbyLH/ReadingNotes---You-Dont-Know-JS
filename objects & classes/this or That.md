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

### 它自己(Itself)
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
```

👆 `foo.count` 仍旧是 `0`，即便在我们执行了一次 `for` 循环，调用了5次 `foo` 后，`this.count++;` 并没有对 `foo.count` 产生一丝一毫的影响。