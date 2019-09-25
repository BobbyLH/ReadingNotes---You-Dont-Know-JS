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

在JS中，如果你将所有的变量和函数都暴露在全局作用域下，那么不仅违背了这一原则，并且还会让你代码质量变得很差！比如👇的代码：
```javascript
function foo (a) {
  b = b + bar (a * 2);
  console.log(b * 3);
};

function bar (a) {
  return a - 1;
};

var b = 2;
foo(2); //15
```

变量 `b` 和函数 `bar` 暴露在外部，任何对它们的修改都可能会导致函数 `foo` 产出的结果变化；正确的做法是把暴露在外面的变量和函数都放到函数 `foo` 的内部，如此一来 `foo` 所产生的结果不会受到潜在影响：
```javascript
function foo (a) {
  var b = 2;
  function bar (c) {
    return c - 1;
  };

  b = b + bar (a * 2);
  console.log(b * 3);
};

foo(2); //15
```

### 避免冲突(Collision Avoidance)
*隐藏* 变量的另一个好处是能够避免由于使用了相同的变量名而产生意外的冲突。
```javascript
function foo () {
  function bar (a) {
    i = 3;
    console.log(a + 1);
  };

  for (var i = 0; i < 10; i++) {
    bar(i * 2);
  };
};

foo();
```
👆函数 `bar` 能够访问在 `for` 循环模块中声明的变量 `i`，并且对它重新赋值；这就会导致一个 *无限的死循环(infinite loop)* —— 无论 `for` 循环进行到哪一步，其内部调用的 `bar` 都会重新赋值 `i = 3;`，因此永远也不能满足 `i < 10` 的终止循环的条件。

#### 全局命名空间(Global "Namespaces")
