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