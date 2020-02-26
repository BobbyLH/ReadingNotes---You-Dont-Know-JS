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