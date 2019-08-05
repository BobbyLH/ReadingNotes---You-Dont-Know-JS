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