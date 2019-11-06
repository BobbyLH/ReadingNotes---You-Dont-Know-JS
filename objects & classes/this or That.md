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

