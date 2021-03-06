# 词法作用域(Lexical Scope)
编程语言中，和 *作用域* 有关的模型一般分为：
- 词法作用域(Lexical Scope)

- 动态作用域(Dynamic Scope)

前者是本章重点讲述的内容，后者会在附录A中呈现。

## Lex-time
之前一章提到过，*编译器* 在编译阶段第一件事就是做词法分析(lexing / tokenizing)。在这个过程中产生了 *词法作用域*，换句话说，*词法作用域* 是基于你书写代码(变量和块)时就已经确定的，静态的作用域。

**Note**：虽然JS中有一些方法能够动态的修改 *词法作用域*，但这些方法非常危险；对于 *词法作用域* 最佳实践是保持原汁原味 —— 代码写(编译)的时候是什么样就是什么样。

```javascript
function foo (a) {
  var b = a * 2;
  
  function bar (c) {
    console.log(a, b, c);
  };

  bar (b * 3);
};

foo (2); // 2 4 12
```
👆上面代码有三个嵌套的作用域：
1. 全局作用域，这个作用域中能访问到函数 `foo`；

2. 函数 `foo` 的作用域，这个作用域能访问变量 `b`、参数 `a`、函数`bar`，以及外层的全局作用域；

3. 函数 `bar` 的作用域，该作用域能访问参数 `c`，以及外层的 `bar` 作用域能访问的内容。

可以看出，这些作用域是在代码书写时就确定的。

### 查询(Look-ups)
**作用域查找会在找到第一个满足的值后停止(Scope look-up stops once it finds the first match)**。如果 *不同层级的作用域* 存在同名的变量，就会产生 *遮盖(shadowing)* —— 内部的变量会 *遮挡(shadow)* 外层的变量。

`window.a` 可以避免全局变量 `a` 被 *遮挡* 的情况：
```javascript
var a = 5;
function foo () {
  var a = 6;
  console.log(window.a);
};

foo(); // 5
```

无论函数在何处被调用、如何调用，*词法作用域* 仅会在函数声明的时候定义。

## 欺骗作用域(Cheating Lexical)
JS中有两个机制能够在运行的过程中，动态的“欺骗”作用域：`eval` 和 `with`。

社区内普遍认为这是两个应该被弃绝的东西，因为它们不仅让你的代码难以维护，更要命的是，它们会严重的影响性能！

### `eval`
`eval(…)` 接受字符串作为参数，并将这些字符串作为可执行的程序执行。这意味着，你能够在编写 *由程序本身生成的代码*。

从这个角度来看，`eval` 能让你 *欺骗词法作用域*，让它以为这段代码是在你写的(实际上你确实写了，只不过它是一个字符串，然后在代码运行的时候由程序生成了可执行的代码)。

```javascript
function foo (str, a) {
  eval(str);
  console.log(a, b);
};

var b = 2;
foo('var b = 3;', 1); // 1, 3
```
👆`'var b = 3;'` 作为字符串传递到 `eval` 中，然后生成并执行了这段代码；在随后的 `console.log(a, b);` 中，对于 RHS `b` 的查询在函数 `foo` 的作用域内就完成了，直接无视了全局作用域中声明的 `var b = 2;`。

`eval` 的本质是用来动态创建代码，但带来的 *副作用* 就是也同时能修改在 *代码写作时(author-time)* 确定的 *词法作用域*。

在严格模式中，`eval` 不能修改作用域：
```javascript
function foo (str) {
  "use strict";
  eval(str);
  console.log(a);
};

var a = 2;
foo('var a = 3;'); // 2
```

除了 `eval` 之外，`setTimeout` 和 `setInterval` 的第一个参数也能接受一个字符串，并随后动态的生成执行的回调函数 —— 这是一个古老的特性，别用它！

另外，构造函数 `new Function(…)` 的最后一个参数也能接受字符串，并动态生成函数 —— 别用它！

动态生成代码在实际生产中的用处不太大，并且由于带来的性能问题，弊大于利！

### `with`
使用 `with` 关键字是另外一种能够 *欺骗* 词法作用域的机制。虽然对于 `with` 能有很多种解释，但这里主要聚焦于它是如何影响词法作用域的。

`with` 在引用对象属性时，能够省略对象，从而实现所谓的 *简写(short-hand)*：
```javascript
var obj = {
  a: 1,
  b: 2,
  c: 3
};

// 正常的 LHS -> 改变对象属性的值
obj.a = 2;
obj.b = 3;
obj.c = 4;
obj; // {a: 2, b: 3, c: 4}

// 使用with，能够省略对象
with (obj) {
  a = 3;
  b = 4;
  c = 5;
};
obj; // {a: 3, b: 4, c: 5}
```

但是惊喜就发生在你对某个对象上，并不存在的属性赋值的时候：
```javascript
function handleWith (obj) {
  with (obj) {
    a = 2;
  };
};

var o1 = {
  a: 3
};

var o2 = {
  b: 3
};

handleWith(o1);
console.log(o1.a); // 2

handleWith(o2);
console.log(o2.a); // undefined
console.log(a); // 2 (惊喜!)
```
👆因为 `o2` 中并没有属性 `a`，而 `with` 会将传入的对象认为是一个完全独立的词法作用域，因此 `with` 会直接赋值，而由此产生的副作用就是污染了全局作用域。

**Note**：虽然 `with` 将对象视为独立的词法作用域，但在其中用 `var` 声明的变量不属于这个作用域范畴，而是属于外部包裹 `with` 的作用域范畴。

和 `eval` 通过传入字符串生成代码，从而修改已有的作用域相比，`with` 是直接从你传入的对象中，凭空创建一个新的词法作用域 —— 上例中传入的 `o1`，会被 `with` 视为一个新的词法作用域，该作用域内包含了 *标识符(identifier)* `a`，与之对应的就是属性 `o1.a`；而 `o2` 没有对应的标识符 `a`，因此会使用常规的 *LHS* 的查找机制。

在严格模式下，`with` 会直接被禁用，`eval` 禁用了大部分有害功能，只保留了少数的核心功能。

### 性能(Performance)
`eval` 和 `with` 通过对词法作用域的修改，或是在运行创建一个新的词法作用域，来实现欺骗*author-time* 时定义的作用域 —— 更灵活的代码并不总意味着是一件好事。

*JS引擎* 在代码编译阶段实现了很多性能上的优化，其中绝大部分是基于词法作用域的静态特征 —— 比如预声明 *变量和函数*，就是所谓的 *提升(hoist)*，这会使得在执行阶段时解析这些 *变量和函数* 的标识符更快。

但如果 `eval` 或者 `with` 出现在代码中，*JS引擎* 就会认为它没办法提前知道执行到什么地方的时候，词法作用域可能会被 `eval` 修改，或者干脆 `with` 创建了一个新的作用域 —— 这会导致大部分的性能优化动作都变得毫无意义！无论 *JS引擎* 有多么智能，一个无法回避的事实就是你的代码运行速度肯定会变慢！

## 回顾(Review)
*词法作用域(Lexical Scope)* 就是该作用域在你 *书写代码(author-time)* 时就已经确定的作用域。这样的作用域在编译阶段能够确定关于 *变量和函数的标识符* 是在什么位置被定义的，因此能够提前预声明这些标识符。

`eval` 和 `with` 能够欺骗作用域，前者能够动态的修改作用域，后者能则直接在代码运行过程中创建一个新的作用域。别用它们！它们会让你的代码运行更慢！
