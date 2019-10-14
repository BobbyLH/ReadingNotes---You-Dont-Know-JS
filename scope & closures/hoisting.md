# 提升(Hoisting)
对于函数作用域和块级作用域的规则 —— *任何在这些作用域中声明的变量，都会附定(attach)到这个作用域上* —— 上一章已经讨论较为充分了。但还有一些关于作用域中如何处理附定工作的细节，还需要再看看。

## 鸡还是蛋?(Chicken Or The Egg?)
如果你认为JS运行代码是一行接一行，自上而下的顺序，虽然基本没错，但可能会带来认知的误区：
```javascript
a = 2;

var a;

console.log(a); // 2
```
👆很多开发者都以为打印的结果应该是 `undefined`，但实际上会输出 `2`。

👇下面的输出结果中，有人认为是 `2`，还有人认为是 `ReferenceError`，但实际上是 `undefined`：
```javascript
console.log(a); // undefined

var a = 2;
```

## 编译器又来了(The Compiler strikes Again)