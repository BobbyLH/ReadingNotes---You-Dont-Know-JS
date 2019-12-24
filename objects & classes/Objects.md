# 对象(Objects)
前面两节的内容主要解释了 `this` 是如何根据规则，指向某个对象的 *上下文* 的。但对象到底是什么？为什么需要指向它呢？

## 语法(Syntax)
JS中创建 **对象(Objects)** 源于两种形式：*字面量声明(literal declarative)创建* 和 *构造(constructed)创建*。

字面量创建：
```js
var obj = {
  key: 'value'
};
```

构造函数创建：
```js
var obj = new Object();
obj.key = 'value';
```

两种模式创建出的对象都一样，唯一的区别在于 字面量创建 可以一次性的添加多个 *键值对(key/value pairs)*，而 构造函数创建 模式只能创建好对象后，再一个个的往里面添加。

## 类型(Type)
对象在JS这门语言中的重要性不言而喻，它是JS主要的语言类型中的一种：
- `string`

- `number`

- `boolean`

- `null`

- `undefined`

- `object`

`string`、`number`、`boolean`、`null` 和 `undefined` 都属于 简单的原始类型，特别要强调的是 `null`，在使用 `typeof null` 判断其类型时，得到的结果是 `"object"` —— 这是一个历史遗留的bug，实际上，`null` 是原始类型的值。

这句话：**“在JS中万事万物皆对象”** —— **显然是不对的**。

相反，有几种特殊的对象子类型，我们通常将它们视为是复杂类型。`function` 就是其中之一，它不仅是JS中的 *一等公民(first class)*，你可以像操作普通对象一样操作它；并且，它还被赋予了 callable behavior。

数组也是对象子类型之一。它能够让数据更轻量化的组合在一起。

## 内置对象(Built-in Objects)
下面几种 内置的对象子类型(也称为包装对象) 有一些和 原始类型 有密切的关联：
- `String`

- `Number`

- `Boolean`

- `Object`

- `Function`

- `Array`

- `Date`

- `RegExp`

- `Error`

在某些语言中，上述的这些类型会被定义为某种 类(class)，比如Java的 `String` 类。但在JS中，它们的本质只是 内置的函数(built-in function)，加上关键字 `new` 来调用它们构造出一个对象的子类型，例如：

```js
var str_primitive = 'string';
typeof str_primitive; // "string"
str_primitive instanceof String; // false

var str_object = new String('string');
typeof str_object; // "object"
str_object instanceof String; // true

Object.prototype.toString.call(str_object); // "[object String]"
```

👆由 `new String('string');` 构造出的 `str_object` 显然是 `object` 的子类型。而由字面量直接创建的 `str_primitive` 是货真价实的 `string` 原始类型。

想对 `str_primitive` 进行一些字符串的操作，比如获取其长度、截取某个字符等，需要将其转换成对象类型的 `String`。所幸的是，这些冗余的工作被JS自动完成了，而且相当的智能 —— 这也意味着用 `String` 构造函数创建一个字符串显得不仅多余，而且还不方便阅读和维护。**只有当你需要用到额外的选项时，才考虑使用构造函数的形式去创建。**

```js
var str_primitive = 'string';

str_primitive.length; // 6

str_primitive.charAt(3); // "i"
```

同样的规律也适用于 `Number`、`Boolean` 对象子类型中。

`null` 和 `undefined` 没有构造函数为其创建包装对象，只有字面量创建的方式；而 `Date` 只能通过构造函数创建，没有字面量创建的方式。

`Object`、`Array`、`Function` 和 `RegExp` 等，无论是通过字面量的形式还是构造函数的形式，创建出来的都是对象的子类型。

`Error` 对象很少显示的在代码中创建，一般在程序运行出错时会自动创建。

## 内容(Contents)