# Syntax(语法)

[Babel 在线转译工具](https://babeljs.io/repl/) 是一个能够在线完成新语法转译的工具 —— 你甚至不需要在自己的项目中添加任何编译的步骤，只需要将它转译后的代码直接使用即可。

## 块级作用域声明(Block-Scoped Declarations)
早在 ES6 之前，为了解决变量名污染的问题，IIFE 被广泛采用，比如大名鼎鼎的 JQuery —— 利用闭包的特性，运行一个立即执行的函数，返回一个对象，这个对象能访问这个函数内部的变量，而又不会造成全局变量的污染。

### `let` 声明(`let` Declaration)
ES6中，在于一对花括号 `{…}` 之内，且使用 `let` 而不是 `var` 声明的变量，就是块级作用域下的变量：

```js
var a = 2;
{
  let a = 3;
  a; // 3
}
a; // 2
```

为了避免踩坑，一个建议是在块级作用域中，只使用一个 `let`，在一开始就将要申明的变量罗列出来：
```js
{
  let a = 1, b = 2, c;
  // …
}
```

这样做的好处，是避免了和之前 *变量提升* 的特性搞混淆，让我们交智商税：

```js
{
  console.log(a); // undefined
  console.log(b); // ReferenceError: b is not defined

  var a;
  let b;
}
```

`ReferenceError` 出现的原因，是因为 `let` 声明的变量，没有所谓的变量提升的特性 —— 这被称为 *暂时性的死区 Temporal Dead Zone(TDZ)* —— 在声明它之前，不能访问这个变量。

另一个要当心 TDZ 的场景是在使用 `typeof` 做判断的时候：

```js
{
  if (typeof a === 'undefined') {
    // …
  }

  if (typeof b === 'undefined') { // ReferenceError
    // …
  }

  let b;
}
```

👆这个时候，没有声明的 `a` 没任何问题，但用 `let` 声明的变量 `b`，反而会报错。

#### `let` + `for`
```js
const funcs = [];

for (let i = 0; i < 5; i++) {
	funcs.push(function(){
		console.log(i);
	});
}

funcs[3](); // 3
```

`funcs[3]()` 之所以输出 `3` 而不是 `5`，是因为在 `for` 循环顶部用 `let` 声明的变量 `i`，它的作用域是在每一次循环体内，`funcs[3]` 中的值是 `3`。

同样，`for…in`、`for…of` 循环也遵守这个规则。

### `const` 声明(`const` Declaration)
用 `const` 声明的常量，一旦创建后，就是只读(read-only)的存在，不能再次进行修改。

并且用 `const` 声明的常量，必须要进行赋值，哪怕就想要一个 `undefined`，那也得 `const a = undefined`。

常量所限制的，并不是值本身，而是值的引用 —— 即当赋值是一个复杂数据的时候，比如对象、数组，只要引用不发生改变，那么其中的值如何变化，也不会有任何影响：

```js
{
  const a = [1, 2, 3];
  a.push(4);
  a; // [1, 2, 3, 4]

  a = 42; // TypeError
}
```

当然，如果要限制值的改变，可以修改值的属性描述符，比如修改 `writable` 、`configurable`，或者直接使用 `Object.preventExtensions()`、`Object.seal()`、`Object.freeze()` 等 API 来达成目的，具体可参考关于 [对象的属性描述符](https://github.com/BobbyLH/ReadingNotes---You-Dont-Know-JS/blob/master/objects%20%26%20classes/Objects.md#%E5%B1%9E%E6%80%A7%E6%8F%8F%E8%BF%B0%E7%AC%A6property-descriptors)。

#### `const` Or Not
有江湖传闻，`const` 的性能要优于 `var` 和 `let`，因为 JS 引擎可以提前预知这个变量的类型不会改变，因此可以节省一些可能的性能开销。

有的开发者，喜欢一开始将变量全都用 `const` 来声明，只有当需要改变其引用的时候，才把它的声明改成 `let`。

对于这种开发偏好，这里不做过多评论，但一个建议是：不要将 `const` 作为防止变量被更改的手段(因为只需要简单将 `const` 变成 `let` 就能破除这个规则)，而是将其作为一个信号传递给未来的你和你的同事 —— 这就是一个常量。

### 块级作用域下的函数(Blocked-scoped Functions)
```js
{
  foo();

  function foo() {
    console.log('foo');
  }
}

foo(); // ReferenceError
```

👆可以发现，函数声明在块级作用域中，没有 TDZ 的困扰，而且在块级作用域之外，理所当然的无法被访问。

值得注意的是，如果是在 pre-ES6 的环境下，完成块级作用域的函数声明，可能会遇到一些问题：

```js
if (true) {
	function foo() {
		console.log(1);
	}
} else {
	function foo() {
		console.log(2);
	}
}

foo(); // ??
```

你会发现，在 pre-ES6 的环境下，`foo()` 返回的是 `2` —— 这是因为函数声明提升的特性，导致第二个函数 `console.log(2);` 会覆盖第一个函数。

## Spread/Rest

