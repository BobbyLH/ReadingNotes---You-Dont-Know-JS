# 附录：混杂不堪的JS运行环境(Appendix A: Mixed Environment Javascript)

## ECMAScript 的附录(Annex B)
现代的JS能够在多种环境中运行：浏览器环境、node环境、rhino环境...而 **Annex B** 中主要讨论的是在浏览器环境中的一些兼容性问：
- 在 **非严格模式中(non-strict mode)** 允许使用八进制(Octal)字面量，比如 `0123`(转换成十进制decimal是 `83`)

- `window.escape(...)` 和 `window.unsecape(...)` 能让你将不满足 *ASCII码、数字、`"`、`@`、`*`、`_`、`+`、`-`、`.`、`/`、* 之外的字符串用 `%` 作为分隔符将其转换成十六进制的转义序列(hexadecimal escape sequences)

- `String.prototype.substr` 和 `String.prototype.substring` 比较相似，除了前者的第二个参数表示 **截取字符串的长度**，后者的第二个参代表 **截取结束的索引位置**

### Web ECMAScript
- [Web ECMAScript](https://github.com/tc39/ecma262) 是ECMA官方关于ECMAScript规范和浏览器实现差异的说明；这是现代浏览器厂商需要遵循以便做到兼容其他浏览器的必备条款：
  - `<!--` 和 `-->` 是有效的单行注释分隔符

  - `String.prototype` 包含了一些专门将string格式化处理带有HTML Tag的方法：
    - `anchor(...)`
    - `big(...)`
    - `blink(...)`
    - `bold(...)`
    - `fixed(...)`
    - `fontcolor(...)`
    - `fontsize(...)`
    - `italics(...)`
    - `link(...)`
    - `small(...)`
    - `strike(...)`
    - `sub(...)`
  
  - `RegExp` 正则扩展：
    - `RegExp.$1` - `RegExp.$9` 匹配组

    - `RegExp.lastMatch` 最后一个匹配项 和 `RegExp['$&']` 最近的匹配项

  - `Function.prototype` 中的 `Function.prototype.arguments` 和 `Function.caller`，尽管它们已经被宣布弃用(deprecated)，但仍然有不少的老代码中存在，依然要小心

---

## 宿主对象(Host Object)
宿主对象指的是：**创建并提供运行你的JS代码的环境**；通常来讲都有一些内置的对象和函数，例如：
```javascript
const div = document.createElement('div');

typeof div; // "object"
Object.prototype.toString.call(a); // "[object HTMLDivElement]"

a.tagName; // "DIV"
```

宿主对象内建的变量通常具有的规律：
  - 可能没办法调用一些内置的方法，比如 `toString()`

  - 不能够被重写

  - 有某些预定义的只读的属性

  - 有一些不能够被改写 `this` 指向的方法

  - ...

---

## 全局DOM变量(Global DOM Variables)
你可能会意识到，在全局作用域中声明一个变量(无论是否带了 `var`)，都会在 `window`(浏览器) 或者 `global`(node) 对象中存在一个与之名字对应的变量

但是鲜为人知的是你创建一个DOM元素，并且为其赋值一个 `id` 属性，那么就能够在全局获取和 `id` 的值同名的变量：
```html
<div id='test'>test</div>
```

```javascript
typeof window['test']; // "object"
'test' in window; //true

console.log( window['test']); // <div id='test'>test</div>
```

这也是为什么你应该避免在全局作用域下创建变量的原因，如果不得不那么做，至少也应该选择一个不会冲突的名称

---

## 内置原型对象(Native Prototypes)
这一部分作者讲述了自己的一段亲身经历：当他将自己开发的依赖于JQuery的一个插件运用到某个网站时，这个网站挂了；明显的是，他开发的插件能在绝大多数站点上运行，于是他花了大约一周的时间，在这个站点的一处很古老的文件中，终于找到了罪魁祸首：

```javascript
// Netscape 4 dosen't have Array.push
Array.prototype.push = function (item) {
  this.[this.length] = item;
};
```

显然，写这段代码的人故作聪明的想给网景浏览器4添加数组的 `push` shim —— 首先，先不说网景公司早就关门了，就说这个年代谁还会关心这样的浏览器？！其次，内建的`push` 可以一次性传入多个参数并推进数组中，就是这个只shim了一部分的半吊子push方法，让作者的代码崩溃了。

所以，对于内置的原型对象，唯一可行的诀窍是 —— 永远不要对它进行拓展，即便是现在看上去非常有用、设计完美、并且有一个合理的名字 —— 你可以选择向 **TC39** 提交申请，让官方来标准化它们

### 垫片(Shims/Polyfills)
社区中关于部分垫片(partial-polyfill/)是否应该被使用的讨论持续了很久，每个人都有自己的决定，[ES5-Shim](https://github.com/es-shims/es5-shim) 以及 [ES6-Shim](https://github.com/es-shims/es6-shim) 提供了很多新的API的垫片，Babel、Traceur等工具也能够方便将你使用了新特性的代码优雅降级

---

## `<Script>`标签
通常一个JS应用都会包含若干个 `<script src=...></script`(external file) 或 `<script></script>`(inline) 标签，无论是外链还是内联的JS代码，它们到底是被视为分隔的还是整体的代码？

一个共识是全局作用域是能被共享的，即 `window` 对象(浏览器环境)能被所有的代码块访问和交互

但需要注意的是，全局的变量提升(global variable scope hoisting)不会跨边界：
```html
<script>
  foo(); // ReferenceError
</script>
<script>
  function foo() {}; 
</script>
```
只能这样：

```html
<script>
  function foo() {};
</script>
<script>
  foo(); // undefined
</script>
```

如果在某个 `<script></script>` 内发生了错误，只会阻止这块JS代码的运行，其余的不会受到阻碍

关于内联和外链的 `script`的区别，最重要的是解释它们内容的字符集(character) —— 前者是由HTML页面的 `meta` 标签的 `charset` 确定的，后者则是根据Script标签中的 `charset` 或其默认值决定：
```html
<meta charset="UTF-8">

<script type="text/javascript" src="pathTo/index.js" charset="UTF-8"></script>
```

---

## 保留字(Reserved Word)
有四类 **保留字** 不能够用于变量名：
- 关键字(keywords)，比如 `function`、`switch`

- 未来的保留字(future reserved words)，比如 `enum`

- `null`

- `true` / `false`

ES5以前，保留字甚至不能够作为对象的键名，所幸现在这样的限制已经不存在了：
```javascript
const obj = { import: 'test' };

console.log(obj.import);
```

---

## 执行时的限制(Implementation Limits)
不同的浏览器使用了不同的JS引擎，它们对于一些极端情况的处理各有不同限制：
- 字符串字面量允许的最大字符长度的不同限制

- 函数传入参数(实参)的大小的不同限制，即栈大小(stack size)的不同限制

- 函数声明时的形参个数的不同限制

- 最大递归深度的不同限制(how long a chain of function calls from one to the other can be)

- JS代码阻塞主线程最长秒数的不同限制

- 变量名的最大长度的不同限制

- ...