# 进入编程(Into Programming)
## 代码(Code)
所谓的编程，实际上就是在一定规则范围(通常叫做语法 syntax)内，用人类能够看懂的语言，组合拼装成计算机能够识别的指令，最终让计算机执行它们

通常，这些代码都被保存在文件中，而JS代码还能直接运行在浏览器的控制台里

### 语句(Statements)
在编程语言中，语句就是一组由字母、数字和操作符构成，而执行某项指定任务的东西：
```javascript
a = b * 2;
```

上述JS代码即为一个语句，字符 `a` 和 `b` 被称为变量，有点像是一个能储存东西的盒子，也像是一个象征着某个值的占位符

与之相反，`2` 代表的就是它本身，也叫做 *字面量值*

操作符则是 `=` 和 `*`，它们分别执行 *赋值* 和 *乘法运算* 的操作

最后的 `;` 分号标志着这段语句的结束

这段语句实际上告诉计算机：从变量 `b` 中取得值，然后将取得的值乘以 `2`，最后将结果储存在变量 `a` 中

程序就是将这些语言收集起来，它们在一起共同描述程序执行的每一个步骤，以满足编程者的最终的意图

### 表达式(Expressions)
语句是由一个或多个表达式组成，一个表达式涉及到了一个变量或值，或者是一组通过操作符结合在一起的变量和值：
```javascript
a = b * 2;
```

上述语句有四个表达式：
- `2` 是字面量值的表达式

- `b` 是变量表达式，意味着从该变量中取出值

- `b * 2` 是运算表达式，含义是做乘法运算

- `a = b * 2` 是赋值表达式，即将 `b * 2` 表达式的值绑定到变量 `a` 上

一般的表达式也能单独成为一个 **表达式语句** (expression statement)：
```javascript
b * 2;
```

但一般这样的表达式并不常见，因为它对程序来讲并没有任何副作用 —— 它只是从变量 `b` 中取出值然后乘以 `2`，但是对这个结果没有任何的后续操作

另一个常见的表达式语句叫做 **调用表达式语句** (call expression statement)，整个语句就是一个函数的调用：
```javascript
alert('test');
```

### 执行程序(Excuting a Program)
👆上面说的语句虽然告诉了计算机要做什么，但最终的实现要基于整个程序的执行。`a = b * 2` 虽然作为开发者能看懂，但是计算机并不知道这是什么。因此我们需要一个专门的工具(解释器 interpreter/编译器 compiler)将代码变成机器能够明白的命令。

- 在一些计算机语言中，编译的动作发生在程序运行的每时每刻，从头到尾，一行接一行不停的在做编译，这叫做解释器(interpreter)

- 在另一些语言中，编译动作发生在程序运行之前，在程序运行时的代码已经编译成能够被计算机识别的指令了，这叫编译器(compiler)

通常认为JS是解释型语言，但这并不完全准确，因为JS引擎会在运行前进行一次对优化代码的编译动作(比如函数和变量名的提升)，然后马上运行这些编译后的代码

---

## 亲自尝试一把(Try It Yourself)
浏览器的控制台通常是开始写自己第一段JS代码的地方，打开你熟悉的浏览器，用键盘快捷键(keyboard shortcut)快速地打开控制台(自行google)，输入自己的第一行代码：
```javascript
a = 21;

b = a * 2;

console.log(b);
```

### 输出(Output)
```javascript
console.log(b);
```
- 👆上面代码中，`log(b)` 的部分涉及到函数调用 —— 询问变量 `b` 的值，然后打印到控制台中

- `console.` 部分则是一个对象，它存储了 `log(...)` 函数

### 输入(Input)
```javascript
age = prompt("请输入你的年龄：");

console.log(age);
```
- `prompt(...)` 唤起了一个弹层，其中有一个输入框，当你输入年龄的数字后，点击确定，就能够在控制台看到你输入的数字被打印了出来

- `age` 变量保存了你输入的数字，这个数字是 `prompt(...)` 函数调用的返回值

---

## 操作符(Operators)
针对 `=` 赋值运算符，在其左手边(left-hand side)的是目标变量，右手边(right-hand side)的是源值 —— 这正好和我们熟悉的数学公式相反，但不幸的是，这种风格流行于目前绝大部分现代编程语言中，所以，适应它！

```javascript
var a = 20;

a = a + 1;
a = a * 2;

console.log(a); // 42
```

👆上述代码中，`var` 是声明 *变量(declare)* 的关键字，在使用一个变量前，首先要声明它，但只需要声明一次即可

一些常见的JS运算符：
- `=` 赋值运算符

- `+`、`-`、`*`、`/` 等数学运算符

- `+=`、`-=`、`*=`、`/=` 等组合赋值运算符

- 自增(Increment)`++`、自减(Decrement)`--` 运算符

- `.` (点)对象属性运算符

- 相等(Equality)`==`、严格相等(strict-equals)`===`、松不等(loose not-equals)`!=`、严格不等(strict not-equals)`!==` 运算符

- `<`、`>`、`<=`、`>=` 等比较运算符

- `&&`、`||` 等逻辑运算符

---

## 值和类型(Value & Types)
当你想在JS中表达某个值的时候，你应该基于意图选择不同的类型：
- 如果你需要数学计算，选择 `number`

- 如果你想将值打印在屏幕上，你需要的是 `string`

- 若你想要在程序中做决定，`boolean` 应该是你不二之选

**字面量(literals)** 就是直接在源码中展示值本身，即：
```javascript
"test message";
'test message';

50;

true;
false;
```

### 类型之间的转换(Converting Between Types)
为了将不同类型的值转换成合适的类型，JS内部有一种叫 **"coercion"** 的机制为我们提供了便捷：
```javascript
var a = '20';
var b = Number(a);

console.log(a); // "20"
console.log(b); // 20
```

更多关于类型转换的内容，请查阅 [*types & grammer*](https://github.com/BobbyLH/ReadingNotes---You-Dont-Know-JS/tree/master/types%20%26%20grammar) 一章

---

## 注释(Code Comments)
你写的若干行代码，计算机执行也许还不到一秒钟，但其他的开发者想要完全明白，也许要用上相当长的时间。你不仅应该写出能够正常工作的代码，更应该写出可读性高的代码：
- 选择合理的变量名和函数名

- 更重要的是，给你的代码加上注释 —— 为人所书写的且计算机会忽略的文字内容

- 关于注释的一些指导性建议：
  - 没有注释的代码是次等品

  - 太多的注释只能说明你代码写的太烂

  - 注释应该解释 *为什么(why)* 而不是 *是什么(what)*，也可对容易产生疑惑的特殊部分做出说明

在JS中，有两种类型的注释：
```javascript
// 单行注释

/*
  多行注释
  注释1...
  注释2...
*/
```
- 单行注释，顾名思义，只能注释一行文本：
  ```javascript
  var age = 20; // 20 is value of age
  ```

- 多行注释，既能做单行，也能做多行注释：
  ```javascript
  /*
    The age variables accept number value
    20 是某人的年龄
  */
  var age = /* 20 is value of age */20;
  ```

---

## 变量(Variables)
所谓变量，就是一个存储值的 **符号容器(symbolic container)**，它包含的值在你程序运行的过程中是可以变化的

在某些编程语言中，你声明的变量在一开始就指定好了容纳值的类型，它们关注的是变量的类型，术语称 *静态类型(static typing)*，也叫做 *强制类型(type enforcement)*；通常认为这样的特性有助于预防无意间的值转换

而另一些语言强调的是值的类型，允许变量在任何时候都能容纳任何类型的值，术语称 *动态类型(dynamic typing)*，也叫做 *弱类型(weak typing)*；通常认为这样的特性更有灵活性

JS是一种弱类型语言，它申明变量时没有涉及任何关于类型的信息，并且在任何时候都能轻松的使用不同类型的值替代当前容纳的值：
```javascript
var num = 99;

num = num * 2;

console.log(num); // 198

num = '$' + num;

console.log(num); // "$198"
```

除了变量，JS中也有常量，ES6中的 `const` 关键字用于常量；安装习惯，常量通常是大写的，并且用 `-` 来分隔多个字符：
```javascript
const TAX_RATE = 0.05;
```

更多的关于类型的内容，详见[Types & Grammer](../types%20%26%20grammar/README.md)一章

---

## 块(Blocks)
在JS中，通常使用花括号 `{ ... }` 来定义一个块：
```javascript
var num = 99;

{
  num = num * 2;
  console.log(num); // 198
}
```

块出现的典型场景的是在控制语句中，比如 `if(...) {}` 条件判断或 `for (...) {}` 循环语句中

和大多数的语句不同，块语句在结尾处可以不用带上分号 `;`

---

## 条件(Conditionals)
在JS程序中，想要表达 *条件*(做决定)，通常使用 `if` 语句：
```javascript
var a = 100;
var b = 50;

if (a > b) {
  console.log('a is bigger than b');
} else {
  console.log('a is smaller than b')
}
```

需要注意的是，在 `if` 语句的圆括号里，期望的值是 `布尔`(boolean)类型的值，如果传入的不是这种类型的值，会进行相应的转换(coercion)，具体规则详见[Types & Grammer - coercion](../types%20%26%20grammar/coercion.md)一章

---

## 循环(Loop)
所谓的循环，即 **重复的执行某个操作，直到条件不满足为止**，换言之即 **仅在条件满足的时候重复某项操作**

一个标准的循环包含了 *测试条件*(test condition) 和 *块*(block) —— `{...}`；每一次循环执行块时，叫做 *迭代*(iteration)

下面👇是 `while` 和 `do..while` 循环的例子：
```javascript
while (num > 0) {
  console.log('log some message');

  num = num - 1;
}

do {
  console.log('log some message');

  num = num - 1;
} while (num > 0)
```

`do..while` 和 `while` 循环唯一的区别是前者无论条件 `(num > 0)` 满足与否，都会先执行第一次的迭代，后者必须满足条件才能执行第一次的迭代

关键字 `break` 能够打断循环，例如在下面👇的无限循环中，可以通过 `break` 终止循环：
```javascript
var i = 0;

while(true) {
  if (i >= 9) {
    break;
  }

  console.log(i);
  i = i + 1;
}
// 0 1 2 3 4 5 6 7 8 9
```

**Tips**: 由于各种历史原因，编程语言中的计数通常以 `0` 开始，比如 `var i = 0;` 而不是 `var i = 1;`

除了 `while` 循环，还有 `for` 循环也能完成上面的工作：
```javascript
for (var i = 0; i <= 9; i++) {
  console.log(i);
}
// 0 1 2 3 4 5 6 7 8 9
```

`for` 循环的 `(...)` 中有三个分句：
- 初始化分句 `var i = 0`

- 测试条件分句 `i <= 9`

- 更新分句 `i++`

---

### 函数(Function)