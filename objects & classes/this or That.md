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