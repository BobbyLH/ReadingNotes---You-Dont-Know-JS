# 附录：混杂不堪的JS运行环境
## ECMAScript 的附录(Annex)B
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

## 全局DOM变量
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