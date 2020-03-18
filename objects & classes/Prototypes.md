# 原型(Prototypes)
无论是之前提到的 `mixin` 还是 `[[Prototype]]`，本质上都是为了模拟传统面向对象语言中类的实现罢了。

## `[[Prototype]]`
JS中的大部分的对象都有一个内置的属性，在规范中注明为 `[[Prototype]]`，这是一个用来引用其他对象的属性。

```js
var myObj = {
  a: 2
};

myObj.a; // 2
```

在上面👆的代码块中，`myObj.a;` 激活了 `[[Get]]` 运算，它做的第一步事情就是检测 `a` 属性是否存在于 `myObj` 上，如果有，直接返回它。但如果 `myObj` 里没有属性 `a`，那么 `[[Get]]` 运算就会往 原型链 `[[Prototype]]` 中找：

```js
var otherObj = Object.create(myObj);

otherObj.a; // 2
```

`Object.create(…)` 这个方法可以暂时将它理解为创建一个对象，并且将它的 `[[Prototype]]` 关联到参数对象上。因此，`otherObj` 的 `[[Prototype]]` 就关联到了 `myObj` 上，所以即便 `otherObj` 中并没有属性 `a`，但 `[Get]]` 运算通过 `[[Prototype]]` 找到了 `myObj`，而后返回了 `myObj.a` 的值。

`[Get]]` 运算会一层层的查找每个对象的 `[[Prototype]]`，如果最终没有找到，则会返回 `undefined`。

`for…in` 循环和 `in` 操作符同样借用了 原型链(`[[Prototype]]` chain) 的机制，前者会遍历原型链上所有 可枚举(enumerable) 的属性，后者则会挨个检查 原型链 上是否存在某个属性。

```js
for (k in otherObj) {
  console.log('found: ', k);
}
// found:  a

'a' in otherObj; // true
```

### `Object.prototype`
`Object.prototype` 是原型链的尽头，这个对象包含了很多我们常见的方法，比如 `.toString()`、`.valueOf()`、`hasOwnProperty()` 等。通过原型链，常规的普通对象就能调用这些 `Object.prototype` 里面内置的方法。其中有一个 `isPrototypeOf(…)` 的方法接下去会重点谈一谈。

#### 设置/遮盖 属性(Setting & Shadowing Properties)
在原型链中，设置一个属性会经历比你想象的多得多的事情，比如👇看似简单的第一个赋值语句：

```js
myObj.foo = 'foo';
```

如果 `myObj` 已经有一个属性为 `foo`，那么这个赋值语句和期望一样，仅仅简单的将 `foo` 的属性值更变。

如果 `myObj` 没有 `foo` 属性，那么接下去就会遍历原型链(和 `[[Get]]` 运算类似)：
  - 这时候如果 `foo` 不存在于任何一层的原型链中，那么 `foo` 属性就会被直接加到 `myObj` 里；
  
  - 但如果 `foo` 属性在某一层原型链中存在，那情况就有些复杂了：
      1. 如果在原型链中的 `foo` 被设置为 **只读** (`writable: false`)，那么这次赋值操作就会失败，并且若是在在 严格模式 中，还会抛出错误；

      2. 如果在原型链中的 `foo` 定义了 setter 方法，那么这个 setter 方法在每次赋值语句中都会被调用，并且 `foo` 不会被加到 `myObj` 中，同时 setter 也没办被重新定义；

      3. 若原型链上只是一个常规的 `foo`，那么 `foo` 会被加到 `myObj` 上，作为 **遮盖的属性(shadowed property)**。


如果 `foo` 属性在 `myObj` 和原型链中同时存在，这时候就被称为 *遮盖(shadowing)* —— 在 `myObj` 上的 `foo` 会 遮盖 掉原型链上其余所有的 `foo` 属性，因为原型链查找永远只会找到最底层的 `foo`。

当然，`Object.defineProperty(…)` 能够成功绕开上面的 #1 和 #2 两条限制，直接把 `foo` 添加到 `myObj` 上。

对于 **只读** 属性没法办赋值成功的原因，是因为 JS 想加强它“拥有” 类的继承 的幻觉 —— 如果你从一个类中继承了某个属性，它是只读的，那么在子类中，继承而来的这个属性同样也应该是只读的。但如果你了解了JS继承的本质，就会觉得 `myObj` 无法设置 `foo` 属性，仅仅是因为某个其他的对象将 `foo` 属性设置为了只读，你就会感到十分的别扭。而 `=` 赋值操作符不起作用，但是 `Object.defineProperty(…)` 又不受限制的首尾不一致，更是让人啼笑皆非。

遮盖往往会在意料之外悄悄地发生：

```js
var myObj = {
  a: 2
};

var otherObj = Object.create(myObj);

myObj.a; // 2
otherObj.a; // 2
myObj.hasOwnProperty('a'); // true
otherObj.hasOwnProperty('a'); // false

otherObj.a++;
myObj.a; // 2
otherObj.a; // 3

otherObj.hasOwnProperty('a'); // true
```

从道理来讲，`otherObj.a++;` 语句的执行应该是先从原型链上找到 `myObj` 上的 `a`，而后对其进行 `++` 操作。但实际上 `otherObj.a++;` 等价于 `otherObj.a = otherObj.a + 1`，因此若你想改变 `myObj.a` 的值时，只能用 `myObj.a++;`。

## “类”("Class")
开始这一节之前，我们先问一个问题：对象关联到另一个对象到底有什么用？为了回答这个问题，我们先得明白 `[[Prototype]]` 到底是什么以及如何使用它，但在这之前，还得先搞懂它不是什么。

在 JS 中，根本没有所谓的“类”，其底层只是对象。事实上，JS 在市面上众多流行的语言中是唯一一个能真正配得上 “面向对象” 这个标签的语言。

**There's just the object.**

### “类”函数("Class" Functions)
JS 中，“类”的实现都离不开函数：所有的函数都有一个公开的，且不可遍历的属性叫做 `prototype`，它是一个任意的(arbitrary)对象。

无论我们怎样叫它(`prototype`)，这个对象到底是什么呢？简单来讲，所有通过 `new` 关键字创建的“实例”，都会自动关联(`Foo.prototype`)上这个对象：

```js
function Foo () {}

var a = new Foo();

Object.getPrototype(a) === Foo.prototpye; // true
```

👆当调用 `new Foo()` 创建出“实例” `a` 时，其内部的 `[[Prototype]]` 就自动关联到了 `Foo.prototype`。

仔细思考上面的行为到底意味着什么 —— 在面向类的语言中，将一个类实例化的过程，一定会伴随着复制拷贝的行为，但是在 JS 中，并没有发生这样的事情。当然你能创建出多个对象都同时关联到某一个对象上，但那并不是赋值拷贝，这些对象彼此间并没有独立分开，相反是关联到一块儿去了。

`new Foo()` 返回的结果是一个新的对象，而这个新对象内置的 `[[Prototype]]` 关联到了 `Foo.prototype` 上。事实上，`new Foo()` 明面上做的事情并没有 “关联(link)” 这个动作，这是意外的副作用，它采用了一种迂回的方式帮我们实现了我们的要求：**创建一个新对象并关联到其他对象**。

那么我们能直接做这件事而不是绕来绕去调用 `new` 来实现么？当然可以，这个英雄就是 `Object.create(…)`。

#### 名字真的有那么重要么？！(What' in the name)
上一章中我们看见过类似的图：

![原型继承](./assets/prototype_what's_name.png)

👆唯一的区别是箭头的方向反过来了 —— 这当然是因为JS中类的继承和实例化并没有真正进行赋值拷贝的动作，而仅仅是在不同对象间，通过 `[[Prototype]]` 建立了连接。因而箭头的方向指向的是关联到的对象。

这样的机制常常被称为 “原型继承(prototypal inheritance)”，理解成经典的 “类继承(classical inheritance)” 的动态版。这样的称谓试图让我们将其理解成常见的 类 和 继承 中的一种，但实际上却会让我们扭曲其底层真正的实现。

在编程世界中，“继承” 这个概念有很明确的定义(详见前一章节)，而自从在其前面加上 “原型” 二字，但实际上执行的截然不同的行为时，一晃已经二十几年时间了。这就好比我们一手拿着橙子，一手拿着苹果，非得说苹果是 “红色的橙子” 一样荒谬 —— 无论给这个苹果贴上什么标签，也不能改变苹果和橙子有着本质的区别，而非仅仅是颜色上不一致。

对于JS而言，根本不应该使用 “继承” 这个字眼，更为贴切的说法是 “代理(Delegation)” —— 两个对象通过原型对象关联起来，其中一个对象能够代理另外一个对象上的属性、方法。

另一个比较常见的术语是 “差异继承(differential inheritance)”，这样的叫法是认为我们当描述一个对象的行为时，是基于一个通用的对象，而后在其基础上做差异化的定制。但其本质上还是在假设你的 思维模式(mental model) 比 实际的发生(physically happening) 更为重要 —— 因为在本质上并没有发生复制拷贝的行为。

### “构造函数”("Constructor")
回顾一下之前的代码：

```js
function Foo () {}

var a = new Foo();
```

👆究竟是什么在作祟，使得我们认为 `Foo` 是一个类呢？

一方面，当我们看见关键字 `new` 的时候，就自然而然的联想起了其他面向类的语言中，类的实例化过程，并且函数 `Foo` 在这个过程的确被调用了，就如同类的构造函数在实例化的时候会被调用一样。

更有趣的是，JS 还“想尽一切办法”让我们相信确实有一个构造函数的存在：

```js
function Foo () {}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

`Foo.prototype` 有一个公共的、不能枚举的属性叫做 `constructor`，这个属性指向的是 `prototype` 关联到的函数，即 `Foo`。而且，实例 `a` 看上去好像有一个属性也叫 `constructor`，但实际上 `a` 并没有这个属性，这就回到了之前我们提到的 *原型链查找* 的机制上去了。退一万步讲，就算 `a.constructor` 解析到了 `Foo`，这也不意味 `a` 就是 *构造函数(constructor)* “构造” 出来的。

另一方面，一个以大写字母开头的函数(`Foo` 而非 `foo`)，难道不正暗示这个函数是一个类么？ —— 这可是社区规范！

**Note**：甚至有些 linter 工具会生成提示信息或者报错，如果你在关键字 `new` 的后面调用一个小写字母开头的函数，或者以普通调用方式调用一个以大写字母开头的函数 —— 虽然这对 JS引擎 来说没有任何区别。

#### 构造函数调用？(Constructor or Call?)
在上面的代码片段中，我们很容易认为 `Foo` 是一个构造函数，不仅因为关键字 `new`、以大写字母开头的 `Foo`，更是由于最终的实例化结果生成了一个对象。

实际上，`new` 关键字只是 劫持(hijack) 了函数调用，让其最终返回一个新对象(在函数没有自定义 `return` 的情况下)。在任何函数之前放上 `new`，都有相同的效果 —— 这只是 “构造函数调用(constructor call)”，而并不意味着这个函数是 “构造函数”。

```js
function NothingSpecial() {
  console.log('just a function call');
}

var a = new NothingSpecial();
// "just a function call"

a; // NothingSpecial {}
```

👆 `NothingSpecial` 是一个非常普通的函数，但是当我们在它前面加上 `new` 之后，它就会构造出一个对象(这其实是一个副作用)，而后绑定到变量 `a` 上。这是一次 “构造函数调用”，但就 `NothingSpecial` 而言，它并不是一个构造函数。

### 机制(Mechanics)
```js
function Foo (name) {
  this.name = name;
}

Foo.prototype.myName = function () {
  return this.name;
}

var a = new Foo('a');
var b = new Foo('b');

a.myName(); // "a"
b.myName(); // "b"
```

👆上面的代码块体现了 JS 试图模拟类的实现的两个方面：
  1. `this.name = name;` 为每一个实例化的对象添加了一个 `name` 属性，这和实例化类的过程中，单独为每个实例封装数据的思路如出一辙；

  2. `Foo.prototype.myName = …` 在 `Foo.prototype` 对象上添加了 `myName` 方法，并在其中用 `this` 动态改变包含属性 `name` 的上下文；

即便是 `a` 或 `b` 中并没有方法 `myName`，但通过原型链的查找，调用默认的 `[[Get]]` 算法，依然能够在 `Foo.prototype` 上获取到这个方法，并且通过 `this` 的隐式绑定规则，将上下文指向自身。但其底层的原理并不是将 `Foo.prototype` 这个对象分别拷贝到 `a` 或 `b` 上，而是通过连接。

#### "Constructor" Redux
上面的代码块中，`a.constructor` 指向的是函数 `Foo`，看上去好像是说 `a` 是由 `Foo` 构造出来的，但这是假象！`a.constructor` 之所以能够指向 `Foo`，完全是归功于原型代理(`[[Prototype]]` delegation)，如果你不小心这一点看上去微不足道的区别，到时候掉进坑里也不知道怎么回事：

```js
function Foo () {}

Foo.prototype = {};

var a1 = new Foo();

a1.constructor === Foo; // false
a1.constructor === Object; // true
```

看到了哈👆！`Foo.prototype` 是可以随意修改的，而且很多时候为了模拟类继承的特性，也不得不修改原型对象。不过这时候 `a.constructor` 就不再指向 `Foo` 了，因为新赋值的对象的 `constructor` 属性指向的是内置的 `Object(…)` 函数。

#### 一错再错(Misconception, busted)
当然，你可以手动重新赋值 `constructor` 属性，特别是你想继续错下去的时候：

```js
function Foo () {}

Foo.prototype = {};

Object.defineProperty(Foo.prototype, 'constructor', {
  enumerable: false,
  writable: true,
  configurable: true,
  value: Foo
});
```

当你信心满满的调用 `Object.defineProperty(…)`，从而尽可能模拟 `constructor` 的可写、可配置，但不可枚举的属性特性时，你已经在某个实例就是被其 “构造函数” 所 “构造” 的道路上越走越远了。

`constructor` 不是银弹，你能随意的重写它 —— 多么随意啊！如果你的代码中依赖于 `constructor` 做一些事情，那它带来的副作用很可能是产生bug的罪魁祸首！这种不可靠、不安全的引用，一定要尽量避免！


## “原型继承”("Prototypal Inheritance")
通常情况下，我们认为的 “类的继承” 应该发生在父子类之间，而非类和实例之间。不过在JS中，之所以能够实现的 “原型继承”，反而是因为 `a` 能够 “继承” `Foo.prototype` 而实现的。

![原型继承](./assets/prototype_prototypal_inheritance.png)

👆上图体现出不仅仅 `a1` 代理了 `Foo.prototype`，并且 `Bar.prototype` 也做了一样的事情。看上去类似父子类继承的概念，但实际上除了箭头之外，只有代理联接(delegation links)，没有复制拷贝。

一个典型的 “原型继承” 风格的代码：
```js
function Foo (name) {
  this.name = name;
}

Foo.prototype.myName = function () {
  return this.name;
}

function Bar (name, label) {
  Foo.call(this, name);
  this.label = label;
}

Bar.prototype = Object.create(Foo.prototype);

Bar.prototype.myLabel = function () {
  return this.name;
}

var a = new Bar('a', 'obj a');

a.myName(); // "a"
a.myLabel(); // "obj a"
```

👆上面实现 “原型继承” 的代码，最关键的一步是 `Bar.prototype = Object.create(Foo.prototype);`，`Object.create(…)` 会创建一个新对象，并且将这个对象的 `[[Prototype]]` 关联到指定的对象上(上面的例子是`Foo.prototype`)，并最终返回这个对象。换句话说，就是将 `Bar.prototype` 关联到 `Foo.prototype` 上。

“原型继承” 的实现过程中，常常会产生一些个常见的错误：

```js
Bar.prototype = Foo.prototype;

Bar.prototype = new Foo();
```

就 `Bar.prototype = Foo.prototype;` 而言，关键的问题是你一旦往 `Bar.prototype` 上新增或删除某个属性或方法，这都会影响到所有 “继承” 自 `Foo` 的类，或者实例化后的实例。如果你发现你不得不这么做的时候，不妨想想是否 `Bar` 有存在的必要？直接用 `Foo` 不就完事了？

而 `Bar.prototype = new Foo();` 虽然看上去好像没什么大问题，但一旦函数 `Foo` 写了一些副作用的时候(日志输入、装改改变、指定`return`返回值、**在 this 上添加新的属性或方法**…)，一旦你调用了 “构造函数”，这些副作用就会产生意想不到的问题。

当然，`Object.create(…)` 也不是没缺点 —— 它会创建一个新的对象，消耗一定的资源。不过在ES6之前，`__proto__` 虽然能访问到原型对象，但并没有标准化，因此 `Object.create(…)` 是唯一靠谱的解决方案。不过在 ES6 来临时，不仅规范化了 `__proto__`，而且还提供了一个标准的API `Object.setPrototypeOf(…)`，帮助实现 “原型继承”：

```js
Object.setPrototypeOf(Bar.prototype, Foo.prototype);
```

如果忽略 `Object.create(…)` 掉性能上的一些影响(替换掉旧的对象会被垃圾回收掉)，它其实更为简洁，但从语义角度来看，`Object.setPrototypeOf(…)` 最终会胜出也说不定。

### 检查“类”的关系(Inspecting "Class" Relationships)
在JS中，如果你想要检查一个对象是否代理了另外一个对象，使用 `instanceof` 关键字检查所谓某个对象是否是某个类的实例，通常被称为 *内检(introspection / reflection)*，看山去是一个常见的方案：

```js
function Foo () {}

Foo.prototype.name = 'Foo Function';
var a = new Foo();

a instanceof Foo; // true
```

👆`instanceof` 运算符左侧接受一个对象，右侧接受一个函数；但它并不是的去判断某个对象是否是某个函数的“实例”，而是按照算法去遍历左侧 `a` 的原型链，然后看看其中是否有一个任意的原型对象，指向的是 `Foo.prototype`。

因此，如果你想要检查两个对象直接是否存在代理关系，`instanceof` 显然不能满足需求。

**Note**：用内置的 `bind(…)` 方法进行 `this` 强绑定返回的函数没有 `prototype` 属性，这也意味着使用 `instanceof` 检查用此类函数“实例化”的对象时，其实并不是检查通过 `bind(…)` 绑定的返回的原函数，而是 `bind(…)` 之前的原函数：

```js
var obj1 = {
  fn: function (name) { this.name = name; }
};

var obj2 = {
  name: 'object'
};

var bindFn = obj1.fn.bind(obj2);

var a1 = new bindFn();

a1 instanceof bindFn; // true
a1 instanceof obj1.fn; // true

bindFn.prototype; // undefined
obj1.fn.prototype.isPrototypeOf(a1); // true
```

👇这段代码用一种 “神奇” 的办法试着解决检查两个对象直接是否存在关联：

```js
function isRelatedTo(o1, o2) {
  function F () {}
  F.prototype = o2;
  return o1 instanceof F;
}

var c = {};
var b = Object.create(c);

isRelatedTo(b, c); // true
```

👆问题不是在于能不能检查出两个对象直接的关联，而是归根于在 JS 中一定要去模拟出类以及类相应的语法上。比如上面例子使用的 `instanceof` 间接揭示的语义。

另一种检查的办法是使用 `isPrototype(…)` 内置方法：

```js
Foo.prototype.isPrototypeOf(a); // true
```

`Foo.prototype` 虽然是一个对象，并且参数也传递的是一个对象，看上去好像完成了两个对象是否关联的检查，但 `isPrototypeOf(…)` 的本质依然是在问：`a` 的原型链中，是否存在 `Foo.prototype`。

普通对象都能引用 `isPrototypeOf(…)`，因此用它来代替上面使用的 `isRelatedTo` 再好不过：

```js
c.isPrototypeOf(b); // true
```

如果要获取 `prototype`，在ES5中早已提供了支持：

```js
Object.getPrototypeOf(a);

Object.getPrototypeOf(a) === Foo.prototype; // true
```

在ES6之前，`__proto__`(社区为`__`专门创建了一个术语叫做 dunder，因此 `__proto__` 又被称为 dunder proto) 是一个非标准的实现(虽然大部分浏览器都支持)，直到ES6来临。虽然其看上去是一个属性，但正确的理解方式应该是将其理解为 getter/setter，比如下面👇这样：

```js
Object.defineProperty(Object.prototype, '__proto__', {
  get: function () {
    return Object.getPrototypeOf(this);
  },
  set: function (o) {
    Object.setPrototypeOf(this, o);
    return 0;
  }
})
```

换句话说，当我们直接使用 `__proto__` 的时候，其实是调用了 `Object.getPrototypeOf(…)`。而为其赋值的时候，调用的是 `Object.setPrototypeOf(…)`。这里强调一点，虽然 `__proto__` 可以改变其值，但最好别随意更变，将其视为 **read-only** 对大家都好。

## 对象连接(Object Links)
`[[Prototype]]` 机制的本质就是对象内部存在一个能引用其他对象的连接。这个连接存在的意义就是当对象中不存在某个属性或方法的时候，能通过一层层的查找机制，引用其他对象中的同名属性或方法，这样 *对象的原型对象的原型对象…* 的一系列连接被称为 “原型链(prototype chain)”。

### `Create()`ing link
为什么我们作为 JS 的开发者一定要绕来绕去，非得使用什么所谓的原型继承，然后把自己绕晕进去呢？有什么好的办法能让我们既使用了对象连接的优势，又避免了被伪装类而产生的错综复杂的关系所困扰呢？英雄是 `Object.create(…)`：

```js
var foo = {
  something: function () {
    console.log('Tell some stories');
  }
}

var bar = Object.create(foo);

bar.something(); // "Tell some stories"
```

👆使用 `Object.create(…)` 让我们能够使用 “原型链” 机制带来的好处 —— 直接在 `bar` 调用自身并不存在的方法 `bar.something();`，并且不用被 `new`、`.prototype`、`.constructor` 等相关问题所困扰。

**Note**：`Object.create(null)` 会创建一个被俗称为 *字典(dictionaries)* 的对象 —— 它是一个干净的空对象，没有代理任何其他的对象，因而对其使用 `instanceof` 运算符的结果总会返回 `false`。它做为一个纯净的数据存储器在合适不过了，因为它不会产生因为原型链代理而来的副作用。

#### `Object.create()` Polyfilled
作为 ES5 规范提出来的 `Object.create()`，若要兼容之前的环境，通常采用简单的 **部分polyfill** 的做法：

```js
if (!Object.create) {
  Object.create = function (o) {
    function F () {}
    F.prototype = o;
    return new F();
  }
}
```

👆上面采用取巧的办法，利用 `new` 的运行机制，将一个用完即抛的函数的 `prototype` 属性指向目标对象 `o`，而后通过 `new F();` 返回一个原型链上绑定了 `o` 的新对象。

但需要注意的是，标准的 `Object.create(…)` 的实现远不止这些，并且并不能完全被 ployfill：

```js
var anotherObj = {
  a: 2
};

var myObj = Object.create(anotherObj, {
  b: {
    enumerable: false,
    writable: true,
    configurable: false,
    value: 3
  },
  c: {
    enumerable: true,
    writable: false,
    configurable: false,
    value: 4
  }
});

myObj.hasOwnProperty('a'); // false
myObj.hasOwnProperty('b'); // true
myObj.hasOwnProperty('c'); // true

myObj.a; // 2
myObj.b; // 3
myObj.c; // 4

myObj; // {c: 4, b: 3}
```

`Object.create(…)` 的第二个参数接受一个对象，这个对象会被添加到新创建的对象中，并且能使用对象属性描述器定义每个属性 —— 这是没办法被 ployfill 的部分。

虽然通常我们使用 `Object.create(…)` 时，只会用到 polyfill 后的子级的功能，但是有的开发者持有这样一种观念：如果一个新的特性不能被全方位无死角的 ployfill，那就不要 polyfill，而是自定义一个不同名称的 utility：

```js
function createAndLinkObject (o) {
    function F () {}
    F.prototype = o;
    return new F();
  }
}

var anotherObj = {
  a: 2
};

var myObj = createAndLinkObject(anotherObj);

myObj.a; // 2
```

虽然书中作者认为👆这是一种狭隘的观点(narrower perspective)，但最终怎么选，还是要我们自定拿主意。

### 备胎？(Links As Fallbacks?)
把 `[[Prototype]]` 的工作方式，当做某个对象 “缺失” 某个属性或者方式时，用来充当缺省方案(“备胎”)，会让你觉着是一条 “妙计”：

```js
var anotherObj = {
  cool: function () {
    console.log('cool!');
  }
};

var myObj = Object.create(anotherObj);

myObj.cool(); // "cool!"
```

👆 `myObj.cool();` 之所以能正常工作，都要归功于 `[[Prototype]]`，但如果你是存心这样做 —— 将 `anotherObj` 作为 `myObj` 的 “备胎”，以便在 `myObj` 上引用某些其本身不存在的方法或属性。这样的 “魔法” 不仅难懂，更难维护。虽然并不是说这种做法完全不可取，但确实在JS中，不是一种常见且符合习惯的方式。

**Note**：ES6 中新增的 `proxy` 能够更好的处理这种 *“404(method/property not found)”* 的问题，后面会讲到。

一种比较合理的利用 `[[Prototype]]` 的优势，又让你的代码看起来更合理的的写法：

```js
var anotherObj = {
  cool: function () {
    console.log('cool!');
  }
};

var myObj = Object.create(anotherObj);

myObj.doCool = function () {
  this.cool();
};

myObj.doCool(); // "cool!"
```

`myObj.doCool();` 是确确实实存在于 `myObj` 上的方法，而其内部的实现则是利用了 `[[Prototype]]` 的 **代理设计模式(delegation design pattern)**，用 `this` 指向 `myObj`，最终通过原型链调用 `anotherObj.cool()`。

总体来看，将代理模式包裹在方法的内部来实现比直接暴露整个API，要更清晰易懂。

## 回顾(Review)
`[[Prototype]]` 的机制，就是配合内部实现的 `[[Get]]` 操作，当访问某个对象上不存在的属性或方法时，层层遍历 “原型链(prototype chain)”，直至最顶层的 `Object.prototype` 上。若是在某层原型链中找到了则直接返回，若是没有找到，则返回 `undefined`。

最顶层的 `Object.prototype` 包含了很多内置的昂发，比如 `toString()`、`valueOf()`……这也是为什么所有的对象都能访问到这些方法(`Object.create(null)` 创建的对象除外)。

将两个对象关联起来最常用的方式是使用关键字 `new` —— 通过 `.prototype` 属性，将一个对象关联到这个新创建的对象上。函数调用前加上 `new` 通常被称为 “构造(constructor)”，尽管其本质上没有发生传统的面向类语言实例化某个类的那种过程。

虽然 JS 中类的实例化和类的继承都像模像样，但它缺少最为关键的复制拷贝的动作，取而代之的是通过原型链将对象直接连接起来。因此，当了解到其本质后，所有的关于面向对象的术语在JS中看上去都怪怪的，除非你非得强迫自己在心理上使用这种描述模型。

“代理(delegation)” 可能是更为恰当的术语 —— 因为在JS中没有复制拷贝，只有**连接(links)**。