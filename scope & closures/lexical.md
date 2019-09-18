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