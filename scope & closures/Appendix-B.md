# 附录B：兼容块级作用域(Polyfilling Block Scope)

拥有块级作用域功能的 `with` 和 `catch` 语句早在ES3中就被引入了，但是直到ES6，块级作用域的概念才被基于 `let` 和 `const` 以及 `{}` 实现出来。

```js
{
  let a = 2;
  console.log(a); // 2
}

console.log(a); // ReferenceError
```

👆这段代码在ES6环境中运行良好，但是如果想要在 *前ES6(pre-ES6)* 的环境中运行呢？答案是 `catch` 语句：

```js
try {
  throw 2;
} catch (a) {
  console.log(a); // 2
}

console.log(a); // ReferenceError
```

👆的确是让人很恶心的代码，但这不是重点，重点是它能让块级作用域在 *前ES6(pre-ES6)* 环境中运行通畅无阻 —— 你只管写出你优雅的块级作用域代码，而这些 *肮脏* 的工作就交给构建工具去完成好了！

## Traceur
一个由Google维护的开源项目叫 *Traceur*，它要解决的问题就是上面我们讨论的将ES6新特性的语法编译成 *前ES6(pre-ES6)* 的语法，以此来确保你的代码能在大多数环境中运行。话说就连 TC39 委员会也用这个工具来测试他们发布的新特性。