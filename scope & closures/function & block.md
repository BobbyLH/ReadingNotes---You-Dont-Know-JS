# 函数和块级作用域(Function vs. Block Scope)
之前提到过，JS中除了全局作用域之外，还有由函数创建的局部作用域，并且这些作用域之间还能相互嵌套。那么，除了函数之外，是否还有其他机制能够创建作用域呢？

## 函数作用域(Scope From Function)
```javascript
function foo (a) {
  var b = 2;

  function bar () {};

  var c = 3;
}
```
👆无论是参数 `a`，变量`b`、`c` 还是函数 `bar`，它们都属于函数 `foo` 创建的作用域中。并且函数 `bar` 也创建了属于自己的作用域。如果想在全局作用域中访问这几个标识符，就会产生 `ReferenceError` 的错误：
```javascript
bar();

console.log(a, b, c);
```
![avatar](./assets/function_block_referenceError.png)

当然，在 `foo` 和 `bar` 内的作用域，都是能够访问这些标识符的。

## 把代码藏在作用域中(Hiding In Plain Scope)
由函数创建的局部作用域有一个妙用：将任意的代码块放入其中后，能有效的 *"隐藏"* 它们。

软件设计中有一项 *最小特权原则(Principle of Least Privilege)*，又称为 *最小权利/暴露(Least Authority/Exposure)* —— 你应该只暴露最需要的 *API*，而 *隐藏* 所有的其他细节。

在JS中，如果你将所有的变量和函数都暴露在全局作用域下，那么不仅违背了这一原则，并且还会让你代码质量变得很差！