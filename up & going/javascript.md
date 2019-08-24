# 进入JavaScript(Into JavaScript)
这一章节的内容会对聚焦在JavaScript这门语言中一些特别东西，并且对此做一个大致的梳理过程，但是并不会深入其中。

深度探究JavaScript的起点，是从这里开始

## 值和类型(Values & Types)
JavaScript(后简称JS)，变量没有类型，只有值才有类型，下面是一些内置的类型：
- `string` - 字符串

- `number` - 数字

- `boolean` - 布尔值

- `null` 和 `undefined`

- `object` 对象类型

- `symbol`

JS 也内置了 `typeof` 运算符，能够以字符串的形式返回某个值的类型：
```javascript
var a;
typeof a; // "undefined"

a = 'test';
typeof a; // "string"

a = 2;
typeof a; // "number"

a = true;
typeof a; // "boolean"

a = null;
typeof a; // "object"

a = undefined;
typeof a; // "undefined"

a = {};
typeof a; // "object"

a = Symbol(1);
typeof a; // "symbol"
```

__Tips__: 👆注意上面的 `typeof a` 并不是询问变量 `a` 的类型，而是询问当前在变量 `a` 中的值是什么类型。

__Warning__：`typeof null` 返回的是一个 `"object"`，虽然这是一个 _bug_，但它经历了很长历史包袱，因此若是将其修复，甚至会导致更多更严重的问题。

`a = undefined` 是我们显示的给变量 `a` 赋值 `undefined`；在JS中，有以下几种能设置变量为 `undefined` 的方式：
- `var a;` —— 不设置任何值

- `a = undefined;` —— 显示的赋予`undefined`

- `var a = (function () {})()` —— 函数不返回任何值

- `void(undefined)` —— 使用 `void` 运算符

### 对象(Object)
对象是一个键值对的形式的集合，即由字符串类型的 *属性名*(properties)包含 *任意* 的复合值的集合：
```javascript
var obj = {
  a: "text",
  b: 22,
  c: false
};

obj.a; // "text"
obj.b; // 22
obj.c; // false

obj[a]; // "text"
obj[b]; // 22
obj[c]; // false
```

从对象中获取属性可以用 `.` 点运算符(dot notation) 或者 `[]` 括号运算符(bracket notation) —— 前者更常用也更方便阅读，后者在属性名含有 *特殊字符* 或者用 *变量* 时使用：
```javascript
var obj = {
  a: "text",
  b: 22,
  'test string': false
};
var b = 'a';

obj['test string']; // fasle
obj[b]; // "text"
obj['b']; // 22
```

#### 数组(Arrays)
数组是对象的一个子类型，但它的键是由数字类型的 *索引*(indexed) 组成的，并且内置了一个 `length` 的属性表示数组的长度：
```javascript
var arr = ["text", 22, false];

arr[0]; // "text"
arr[1]; // 22
arr[2]; // false
arr.length; // 3

typeof arr; // "object"
```

`length` 属性会自动根据数组的长度进行更新，但如果你自定义了非数字类型的属性名，那么新增的不会计入到 `length` 中；因此正常的使用数组的方式还是用数字类型的索引

#### 函数(Functions)
函数也是对象的一个子类型：
```javascript
function foo () {
  return 22;
}

foo.bar = 'text';

typeof foo; // "function"
typeof foo(); // "number"
typeof foo.bar; // "string"
```

虽然 `typeof foo;` 生成的结果是 `"function"`，然而函数依然可以像对象一样拥有属性，这也意味着函数是对象的一个子类型(subtype)

### 内置类型的方法(Built-In Type Methods)
内置(built-in)类型以及子(subtype)类型暴露出来的一些属性和方法十分有用：
```javascript
var a = 'text';
var b = 3.14159;

a.length; // 4
a.toUpperCase(); // "TEXT"
b.toFixed(4); // "3.1416"
```

👆`a.toUppercase()` 之所以能够这样被调用，在其底层有较为复杂的逻辑；简单讲，有一个 `String` 包装对象，通常称为内置对象(native)，与原始类型的 `string` 配对，在其原型对象`prototype`上，定义了这个叫做`toUpperCase` 的方法。当你像对象一样查询 `"text"` 的属性或者方法时，JS会自动帮我们 *包装(boxes)* 一层与之配对的对象(该例中就是 `String` 对象) —— 对于 `number` 类型的就是 `Number` 对象，`boolean` 类型的就是 `Boolean` 对象。

### 比较值(Comparing Values)
在JS中，有两种比较的类型：*相等(equality)* 和 *不相等(inequality)* 比较。无论比较的值是什么类型，比较的最终结果都是布尔值。

#### 类型转换(Coercion)
JS中有两种形式的类型转化：*显示的(explicit)* 和 *隐式的(implicit)* —— 前者是你能够明显在代码中看到的，后者是指在某些不注意的地方发生的副作用的操作产生的转换。

类型转换不是魔鬼，相反在很多地方它都很有用 —— 对于写出可读的、合理的、易于理解的代码有很大帮助：
```javascript
var a = "22";
// 显示转换 explicit
var b = Number(a);

a; // "22"
b; // 22
```

```javascript
var a = "22";
// 隐式转换 implicit
var b = a * 1;

a; // "22"
b; // 22
```

### Turthy 和 Falsy


