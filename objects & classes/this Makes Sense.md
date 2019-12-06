# `this` 看上去是那么回事了！(`this` All Makes Sense Now!)
之前提及 `this` 的绑定取决于函数调用时的调用条件，即函数如何被调用 —— **调用点(call-site)**。

## 调用点(Call-site)
所谓 *调用点(call-site)*，即代码中函数调用的位置(区别函数声明的位置)。但仅仅通过 *定位函数的调用位置* 来找到 *调用点* 并不那么有效，因为有些书写代码的模式会将其掩盖起来。

另一个重要的概念是 *调用栈(call-stack)* —— 当前函数 *执行时* 的已调用的栈。

👇 *调用点* 和 *调用栈*：
```js
function baz () {
  // call-stack is baz
  // that means call-site is in global scope
  console.log('baz');
  bar();
}

function bar () {
  // call-stack is in baz -> bar
  // that means call-site is in baz
  console.log('bar');
  foo();
}

function foo () {
  // call-stack is in baz -> bar -> foo
  // that means call-site is in bar
  console.log('foo');
}

baz(); // the bar call-site
```

从 *调用栈* 中分析出 *调用点* 是为了获取 `this` 的绑定，你可以使用浏览器自带的debugger工具，或者直接在代码里插入 `debugger`，来在 `foo` 函数的位置打断点，而后在可视化的图形界面中查看👆这段代码真实的调用栈(右侧第二行的Call Stack)：

![avatar](./assets/this_all_makes_sense_now_call_stack.png)

## 规定，不是乌龟的屁股(Nothing But Rules)
想要知道 `this` 的绑定指向，我们必须先查看 *调用点*，并根据 **4** 个既定的规则来知晓 `this` 的真实指向。

### 默认绑定(Default Binding)
第一个场景是我们最为熟知的，即 *单独地调用某个函数(standalone function invocation)*。你可以把这个规则理解成其他绑定规则都没生效时，对于 `this` 绑定的 缺省/默认规则。

```js
function foo () {
  console.log(this.a);
}

var a = 2;

foo(); // 2
```

👆 全局的变量声明 `var a = 2;` 实际上同等于在 *全局对象(global-object)* 中加入了一个属性值 `a` —— 它们不是互为复制品，它们就是彼此，它们是一个硬币的两面，区别仅在于看问题的角度不同而已。

紧接着，我们调用函数 `foo` 而后执行其中的代码块 `console.log(this.a);` 获取到了变量 `a` 的值并将结果打印在控制台上。为什么 `this.a` 能够获取变量 `a` 的值？ —— 我们先查看 *调用点(call-site)*，发现对于函数 `foo` 是 *单独调用* 的，则可以确认应用的是默认绑定规则，因此这时候的 `this` 绑定的是全局对象。

但如果，我们使用了 *严格模式(strict mode)* —— `"use strict";`，`this` 的默认绑定的 *全局对象* 则会被替换成 `undefined`。

```js
function foo () {
  "use strict";

  console.log(this.a);
}

var a = 2;

foo(); // TypeError
```

有个重要的细节点：`this` 的绑定规则虽然基于 *调用点*，全局对象也只会在默认绑定规则且没有运行在 *严格模式* 才会绑定到 `this` 上。但在 *调用点* 上声明的 *严格模式*，并不能决定在这个点上调用的函数是否都遵循 *严格模式*：

```js
function foo () {
  console.log(this.a);
}

var a = 2;

(function () {
  "use strict";

  foo(); // 2
})();
```

### 隐式绑定(Implicit Binding)
另一个 `this` 的绑定规则需要关注：函数的 *调用点* 是否有 *上下文对象(context object)*；换句话说，*调用点* 是否拥有或包含某个对象：
```js
function foo () {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo
}

obj.foo(); // 2
```

首先，无论函数 `foo` 是提前声明而后被加入到对象 `obj` 中的，还是 `foo` 伴随对象初始化时才一并声明，这都不是重点。重点是函数 `foo` 被 `obj` 所包含，当然你也可理解成对象 `obj` 保存了对函数 `foo` 的引用指针。

`obj.foo();` 在函数 `foo` 调用之前还有一个对于 `obj` 的引用，这个 `obj` 就是之前提到的 *上下文对象*。而这正好符合 `this` 的 *隐式绑定* 规则 —— `this` 绑定到函数引用的对象 `obj` 上，因此 `this.a` 指代的就是 `obj.a`。

多层引用时，只有最后一层的对象引用才会绑定到 `this` 上：
```js
function foo () {
  console.log(this.a);
}

var obj2 = {
  a: 42,
  foo
};

var obj1 = {
  a: 2,
  obj2
};

obj1.obj2.foo(); // 42
```

#### 隐式绑定的丢失问题(Implicitly Lost)
一个常见的令人困惑在于 `this` 的隐式绑定会丢失，并退回到默认绑定规则，即 `this` 要么指向全局对象，要么是 `undefined`，这取决于是否应用了 *严格模式*👇：

```js
function foo () {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo
};

var bar = obj.foo;

var a = 'lost binding';

bar(); // "lost binding"
```

👆虽然变量 `bar` 显然是对 `obj.foo` 的引用，但是关键点还是在于 *调用点* —— `bar();` 显然是一个没有任何 *上下文对象(context object)* 的函数调用，因此使用了默认规则也不足为奇了。

*回调函数* 是另一个更微妙、更普遍、更反预期的丢失隐式绑定的场景👇：

```js
function foo () {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo
};

function doFoo (callback) {
  callback && callback();
}

var a = 'lost binding';

doFoo(obj.foo); // "lost binding"
```

👆向 `doFoo(obj.foo);` 传递参数时，虽然 `obj.foo` 是有对象的引用，但函数并没有被调用，此时并不是我们提到的 *调用点*。真正的 *调用点* 还是 `callback && callback();` —— 没有涉及任何的 *上下文对象(context object)*。

你可能会想，如果我使用内建的方法接受回调的方法，是否情况有所改变呢？：

```js
function foo () {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo
};

var a = 'lost binding';

setTimeout(obj.foo, 1000); // "lost binding"
```

![avatar](./assets/this_all_makes_sense_now_implicit_lose.png)

答案显然是 **依然会丢失** —— 关键依然是在 *调用点* 上。

### 显示绑定(Explicit Binding)
了解了默认绑定和隐式绑定的规则后，有一部分的 `this` 指向场景能满足我们的需求了，但如果我们想要强制将某个函数的 `this` 绑定到某个 *上下文对象* 上，而又不想将这个函数作为这个 *上下文对象* 的某个方法来调用，这时候就需要使用 *显示绑定* 规则了。

`call(…)` 和 `apply(…)` 是每个函数都具备的内置方法，它们的使用基本类似，第一个参数都是绑定到 `this` 的 *上下文对象*，后面的则是调用该方法(`call` 和 `apply`)的函数所需要的参数，完成这些后，立即执行这个函数：

```js
function foo () {
  console.log(this.a);
}

var obj = {
  a: 2
};

foo.call(obj); // 2
```

如果你将 *原始值(primitive value)*(`string`、`number`、`boolean`) 作为第一个参数传入 `call` 或 `apply`，那么这些原始值将会被隐式转换成 *包装对象*(`new String(…)`、`new Number(…)`、`new Boolean(…)`)。

但好像显示绑定也并没有解决隐式绑定丢失 `this` 的问题？！

#### 硬绑定(Hard Binding)
围绕着显示绑定实现的一种绑定模式，能够解决丢失 `this` 的问题：

```js
function foo () {
  console.log(this.a);
}

var obj = {
  a: 2
};

var a = 'lost binding';

function bar () {
  foo.call(obj);
}

setTimeout(bar, 1000);

bar.call(window);
```

👆函数 `bar` 无论是作为回调函数，还是自身的 `this` 被显示的绑定到全局对象(`window`)，调用它都只会在控制台打印出 `2` 即 `obj.a` 的值 —— 因为其内部只会将 `foo` 的 `this` 显示的绑定到 `obj` 上。这样的绑定模式我们称为 *硬绑定*。

改进一下，你可以将需要进行硬绑的函数和 *上下文对象* 作为参数传递：

```js
function foo (num) {
  console.log(this.a, num);

  return this.a + num;
}

function simpleBind (fn, ctx) {
  return function () {
    return fn.apply(ctx, arguments);
  }
}

var obj = {
  a: 2
};

var a = 'lost binding';

var bar = simpleBind(foo, obj);

var b = bar(5); // 2 5
console.log(b); // 7
```

👆上面代码中的 `simpleBind` 是内置的 `bind` 方法的部分功能简单实现，内置的 `bind` 方法不仅能够返回一个已经为 `this` 绑定了 *上下文对象* 的函数，并且这个函数还有一个 `name` 属性的值为 `bound foo`，即绑定了 `this` 的函数 `foo`。

#### API调用的 “上下文”(API Call "Context")
很多第三方库的方法，甚至是一些内置的方法，提供了可选参数，这个参数会让你传递 *上下文对象* 来绑定到回调函数 `this` 上：

```js
function foo (ele) {
  console.log(ele, this.id);
}

var obj = {
  id: 'awesome'
};

[1, 2, 3].forEach(foo, obj); // 1 "awesome" 2 "awesome" 3 "awesome"
```

从本质上来说，这些方法都采用了显示绑定的规则，即使用 `call` 和 `apply` 内置方法来帮你避免一些麻烦。

## `new` 绑定(`new` Binding)
第四个，也是最后一个 `this` 的绑定规则，需要我们重新思考JS中的函数和对象，以及关于它们常见的误区。

在传统的面向对象的语言中，通常使用关键字 `new` 来实例化一个类，这个关键字在JS中也同样存在。但JS中使用关键字 `new` 虽然看上去跟面向对象的语言一样，好像都实例化了一个类，但这只是一个假象。

首先，JS中所谓的 *构造器(constructor)* 只是一个函数，在其前面使用了关键字 `new` 来调用它而已。它并非一个类，也不是实例化这个类，它甚至都不是一个特殊的函数 —— 除了使用 `new` 来 *劫持(hijacked)* 它的调用，它本质上和普通的函数没什么两样。

比如，内置的 `Number(…)` 构造函数，在ES5的说明文档里，对其有清晰的定义：
> 15.7.2 The Number Constructor
> 
> When Number is called as part of a new expression it is a constructor: it initialises the newly created object.

JS中并没有所谓的 *构造器函数(constructor functions)*，只有 *函数的构造调用(construction calls of functions)*。

当函数前使用了 `new` 关键字时，发生了如下的事情：
1. 凭空创造了新对象，并将其关联到 *原型(prototype)* 上；
2. 这个新对象被绑定为这个函数中的 `this` 的 *上下文对象*；
3. 除非函数手动的返回一个对象，否则就这个新创建的对象会被返回。

```js
function foo (a) {
  this.a = a;
}

var bar = new foo(a);

console.log(bar.a); // 2
```

`new` 是函数调用时能够绑定 `this` 的最后一种方式，我们称其为 *new 绑定*。

## 一切都井然有序(Everything In Order)
如果只有一种 `this` 绑定的规则，那一切尽在不言中。而现实是我们往往会面临多种绑定规则一起出现，这时候次序，或者说权重就显得很重要了。

一个显而易见的情况是，*默认绑定规则(default binding)* 肯定是优先级最低的情况，因此先比较 *隐式绑定(implicit binding)* 和 *显示绑定(explicit binding)*：

```js
function foo () {
  console.log(this.a);
}

var obj1 = {
  a: 3,
  foo
};

var obj2 = {
  a: 2,
  foo
};

obj1.foo(); // 3
obj2.foo(); // 2

obj1.foo.call(obj2); // 2
obj2.foo.call(obj1); // 3
```

👆很显然，从 `obj1.foo.call(obj2);` 会输出 `3` 的结果来看，*显示绑定* 的优先级高于 *隐式绑定*。

👇接下去是 *隐式绑定(implicit binding)* 和 *new 绑定(new binding)*：

```js
function foo (num) {
  this.a = num;
}

var obj1 = {
  foo
};

var obj2 = {};

obj1.foo(2);
console.log(obj1.a); // 2

obj1.foo.call(obj2, 3);
console.log(obj2.a); // 3

var bar = new obj1.foo(4);
console.log(obj1.a); // 2
console.log(bar.a); // 4
```

很显然，`bar.a` 的值是 `4`，因此 *new 绑定* 优先级高于 *隐式绑定*。

**Note:** `new` 和 `call / apply` 不能够同时使用，因此 `new foo.call(obj1);` 是不能被允许的。为了能够测试 *new 绑定* 和 *显示绑定* 的优先级，我们可以借用 *硬绑定* 来实现。

一般我们用 `Function.prototype.bind(…)` 来对某个函数进行一层 *包裹(wrapper)*，这个包裹会忽视掉其他的 `this` 绑定规则，而使用我们手动提供的 *上下文对象* 来绑定到 `this` 上。

因此，*显示绑定* 优先于 *new 绑定* 是一个很轻易就被推断出来的结论，但实际上并非如此：

```js
function foo (num) {
  this.a = num;
}

var obj1 = {};
 
var bar = foo.bind(obj1);

bar(2);
console.log(obj1.a); // 2

var baz = new bar(3);

console.log(obj1.a); // 2
console.log(baz.a); // 3
```

👆 `bar` 是作为函数 `foo` 硬绑定其 `this` 到对象 `obj1` 的结果，但是 `new bar(3);` 并没有直接改变 `obj1.a` 的值，而是创建了一个新的对象(`baz`)，并将函数 `foo` 的 `this` 绑定到这个对象上，从而得到 `baz.a` 的结果是 `3`。

但如果回顾之前我们写的一个简单的模拟 *硬绑定* 的方法，你会惊奇的发现，`new` 好像没有办法重写 *硬绑定* 的规则：
```js
function simpleBind (fn, ctx) {
  return function () {
    return fn.apply(ctx, arguments);
  }
}

function foo (num) {
  this.a = num;
}

var obj1 = {};
 
var bar = simpleBind(foo, obj1);

bar(2);
console.log(obj1.a); // 2

var baz = new bar(3);

console.log(obj1.a); // 3
console.log(baz.a); // undefined
```

但实际上，在ES5中出现的内置的方法 `Function.prototype.bind(…)` 是更完善的，比如拿 MDN 中对其进行 [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) 的代码为例(能够使用 `new` 关键字的版本)：
```js
// It does work with `new funcA.bind(thisArg, args)`
if (!Function.prototype.bind) {
  var ArrayPrototypeSlice = Array.prototype.slice;
  Function.prototype.bind = function (oThis) {
    if (typeof this !== 'function') {
      throw new TypeError('Function.prototype.bind - ' + 'what is trying to be bound is not callable');
    }

    var aArgs = ArrayPrototypeSlice.call(arguments, 1),
    aArgsLength = aArgs.length,
    fToBind = this,
    fNOP = function () {},
    fBound = function () {
      aArgs.length = aArgsLength; // reset to default base arguments
      aArgs.push.apply(aArgs, arguments);
      return fToBind.apply(fNOP.prototype.isPrototypeOf(this) ? this : oThis, aArgs);
    };

    if (this.prototype) {
      fNOP.prototype = this.prototype;
    }
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

**Note:** 👆MDN提供的 `bind(…)` 的 polyfill 和ES5中内置的 `bind(…)` 方法在硬绑定再后调用 `new`，有些地方有区别。因为 polyfill 会创建一个具有 `prototype` 属性的函数 `fBound` 来作为硬绑定的返回函数，而内置的方法返回的函数则没有。
> The partial implementation creates functions that have a prototype property. (Proper bound functions have none.)

👇这段代码是能够使用 `new` 关键字的核心：
```js
fNOP.prototype.isPrototypeOf(this) ? this : oThis

// 以及

if (this.prototype) {
  fNOP.prototype = this.prototype;
}
fBound.prototype = new fNOP();
```

其中，`fNOP` 的 *原型对象(prototype)* 如果和该函数 *调用点* 时绑定的 *上下文对象* 一致时①，硬绑定所返回的函数的 `this` 指向则会使用这个新的 `this` 作为 *上下文对象*，而非之前预设且保存在变量 `oThis` 中的 *上下文对象*。

**①:** 该场景等于使用关键字 `new` 来调用这个函数，因此时的 `this` 指代的是其函数自身，即`fBound`。而 `fNOP.prototype = this.prototype;` 这行代码将 `fNOP` 的 *原型对象* 提前保存为进行硬绑定的函数的 *原型对象*；随后将 `fBound` 的 *原型对象* 赋值为 `new fNOP();` 进行了 *原型链* 的关联操作。因此，如果使用 `new` 来调用这个函数，那么 `fBound` 的 *原型对象* 其实就是 `fNOP.prototype`。

另一个 `bind(…)` 所具备的能力则是将第一次进行硬绑定过程中，除开第一个参数(即绑定上下文的 `oThis`)的剩余参数 `aArgs`，作为下次调用这个硬绑定函数的默认参数传递进去，从技术角度将，这个特性叫做 *部分应用(partial application)*，也称为 *柯里化(currying)*：

```js
function foo (p1, p2) {
  this.val = p1 + p2;
}

var bar = foo.bind(null, 'your ');

var baz = new bar('name');

console.log(baz.val); // 'your name'
```

### `this` 的决定时刻(Determining `this`)
总结一下绑定 `this` 的规则，从上往下代表优先级的从高到底：
1. 若函数通过 `new` 关键字调用，则 `this` 指向新创建的这个对象：
  `var bar = new foo();`

2. 函数调用 `apply` 或 `call` 来显示的绑定 `this`：
  `var bar = foo.call(obj2);`

3. 函数的 *调用点* 包含了某个 *上下文对象* 来隐式的绑定 `this`：
  `var bar = obj1.foo();`

4. 上述情况都不满足，则使用默认绑定规则 —— *严格模式* 下 `this` 是 `undefined`，否则 `this` 指向全局对象：
  `var bar = foo();`

👆这些基本是所有的 `this` 绑定规则。

## 绑定规则之外(Binding Exceptions)
凡事都有例外，`this` 的绑定也不能幸免。

### 被忽略的`this`(Ignored `this`)
如果你传入 `null` 或 `undefined` 作为 `call`、`apply` 或 `bind` 的第一个参数，`null` 或 `undefined` 会被忽略，取而代之的是默认绑定规则：

```js
function foo () {
  console.log(this.a);
}

var a = 2;

foo.call(null); // 2
```

你可能会问，为什么要用 `null` 或 `undefined` 作为 `this` 的 *上下文对象*？因为一般我们可以使用 `apply` 第二个参数是数组的特性，把一个拥有很多参数(形参)的函数的参数(实参)，放进一个数组中(便于管理)，然后使用 `apply` 来调用这个函数；或者使用 `bind` 的柯里化特性，预先保留一些参数，以便在后面使用的时候自动带上这些参数：

```js
function foo (a, b, c) {
  console.log(a + b + c);
}

foo.apply(null, [1, 2, 3]); // 6

var bar = foo.bind(null, 1, 2);

bar(3); // 6

bar(4); // 7
```

`apply` 和 `bind` 都需要第一个参数作为指定 `this` 的 *上下文对象*，如果你调用的函数根本不关心 `this` 的指向，那么 `null` 就是一个很好的占位符。

**Note:** 虽然 ES6 中已经有 `...` 扩展运算符，能够解构数组，但是 `bind` 方法自带的 *柯里化(currying)* 特性还未有替代物，因此这种情况仍然需要关注。

最危险的情况往往是在你未曾预料的时候发生，就比如你是用了一个第三方的开源的库，你不清楚它的具体实现细节而贸然将它提供的某个方法，用 `null` 或 `undefined` 作为 `this` 的 *上下文对象*，那有可能会导致一些很难追踪到的bug。

#### 安全的`this`(Safer `this`)
创建一个没有任何委托，且完全是一个空对象，作为 `this` 绑定的 *隔离带(DMZ de-militarized zone)*，是一个常见的确保不会产生副作用(比如 `this` 根据默认规则绑定到全局对象上，而后对全局对象产生污染等意外的结果)的办法之一。

用 `ø` 来指代这个空对象更具语义化，使用 `Object.create(null)` 创建出一个没有代理 `Object.prototype` 的空对象：

```js
function foo (a, b, c) {
  console.log(a + b + c);
}

var ø = Object.create(null);

foo.apply(ø, [1, 2, 3]); // 6

var bar = foo.bind(ø, 1, 2);

bar(3); // 6

bar(4); // 7
```

### 间接绑定了`this`(Indirection)
另外一个需要警惕的情况是在赋值的时候发生的 *间接引用(indirect references)*，在这种情形下，一旦调用了该函数引用，执行的是默认的绑定规则：

```js
function foo () {
  console.log(this.a);
}

var a = 2;
var o = { a: 3 };
var p = { a: 4, foo };

(o.foo = p.foo)(); // 2
```

`o.foo = p.foo` 仅仅是对底层函数对象的引用，真正执行的 *调用点* 是 `foo()` 而不是 `p.foo()` 或 `o.foo()`，因此按照规则，默认绑定会被执行。

### 软绑定(Softening Binding)
虽然使用硬绑定能够解决 *不经意间丢失 `this` 绑定而导致默认绑定规则生效* 的情况，但硬绑定让函数失去了灵活度 —— 除非你使用 `new` 关键字，否则任何的隐式绑定，甚至是显示绑定规则都没效果。

*软绑定* 是解决这种问题的一个办法：

```js
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function (presetCtx) {
    var fn = this,
    curried = [].slice.call(arguments, 1),
    bound = function bound () {
      return fn.apply(
        (!this || (typeof window !== 'undefined' && this === window) || (typeof global !== 'undefined' && this === global))
          ? presetCtx
          : this,
        curried.concat.apply(curried, arguments)
      );
    };
    bound.prototype = Object.create(fn.prototype);

    return bound;
  }
}
```

除了检测 *调用点* 的 `this` 是否启用的是默认绑定规则之外，`softBind` 基本和 ES5 中的 `bind` 方法类似。其核心就是预设一个 `presetCtx` 作为替代默认绑定规则的 *上下文对象*，而后返回的函数判断其 *调用点* 的 `this` 指向，只要不是全局对象，那么就应用当前的 `this` 指向，否则将 `this` 绑定到预设的 *上下文对象* 上。

```js
function foo (a, b, c) {
  console.log(a + b + this.c);
}

var bar = foo.softBind(obj1, 1, 2);

var c = 3;
var obj1 = { c: 4 };
var obj2 = { c: 5, bar };
var obj3 = { c: 6 };

obj2.bar(); // 8

bar.call(obj3); // 9

bar(); // 7
```

可以看到，`bar();` 控制台打印的值是 `7`，即为 `1+2+4`，而其他的 `obj2.bar();` 和 `bar.call(obj3);` 都应用上了隐式绑定规则和显示绑定规则，唯一不生效的是默认绑定规则。

## 词法`this`(Lexical `this`)
正常的函数的 `this` 绑定，都遵守👆提到的四种规则，除了 ES6 中新增的 *箭头函数(arrow-function)* 外。

箭头函数 的声明和普通函数不同，它不使用关键字 `function`，而是用 `=>` 这样的胖箭头(fat-arrow)操作符替代。箭头函数的 `this` 绑定规则采用的是 *绑定到箭头函数所处的封闭作用域(enclosing scope)的 `this`上*，该作用域可以是函数作用域，也可以是全局作用域：

```js
function foo () {
  return () => {
    console.log(this.a);
  }
}

const obj1 = {
  a: 2
};

const obj2 = {
  a: 3
};

const bar = foo.call(obj1);

bar.call(obj2); // 2
```

与之类似的做法就是使用 *词法作用域* 来绕过 `this` 的问题：
```js
function foo () {
  const self = this;
  return function () {
    console.log(self.a);
  }
}

const obj1 = {
  a: 2
};

const obj2 = {
  a: 3
};

const bar = foo.call(obj1);

bar.call(obj2); // 2
```

箭头函数的 `this` 绑定和普通函数的有很大区别，它要求你更为熟悉词法作用域。按照作者的观点，如果你发现自己经常使用 *箭头函数* 或者 *词法作用域* 来解决 `this` 的问题时，你应该考虑：
1. 彻底放弃使用 `this`，用 词法作用域 就好了；
2. 拥抱 `this`，完全的理解其机制并熟练使用它解决问题，避免使用 `self = this` 或者 箭头函数 来绕开问题。

千万别在同一个函数中混用 词法作用域 和 `this`，这不仅会造成代码难以维护，而且也会让你自己的工作举步维艰。

## 回顾(Review)
普通函数的 `this` 绑定规则是基于 调用点(call-site) 的，安装从上往下的优先级依次排序：

1. `new` 关键字构造新对象；

2. `call`、`apply`、`bind` 显示绑定；

3. 包含 **上下文对象** 的隐式绑定；

4. 默认绑定规则，在 严格模式`strict mode` 下绑定到 `undefined`，非严格模式绑定到全局对象上。

小心各种 `this` 丢失，而后使用默认绑定规则的情况。在不需要指定 `this` 绑定的前提下，`ø = Object.create(null)` 是一个很好的占位符避免绑定到全局对象上造成全局污染。

ES6 的箭头函数和 词法作用域 能够绕开 `this` 绑定的困扰，在某些情况下，这也是不是办法的办法。