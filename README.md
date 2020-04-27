# 读书笔记 之《You Don't Know JS》

## 罗列了关于拜读《You Don't Know JS》之后的一些收获，仅作为参考 —— 强烈推荐[阅读原著](https://github.com/getify/You-Dont-Know-JS "You Don't Know JS")

### 简介
《You Don't Know JS》这本书不涉及到任何的框架和库, 只聚焦于原生的JS内容, 且大部分是作为JS开发者很容易忽略但却对于理解这门语言所产生的反应至关重要的内容。

- **up & going** -- 第一卷主要从较为基础的内容入手, 简单的罗列了一些常用的概念, 帮助我们快速的回忆起一些可能已经遗忘的JS基础概念, 为之后的几章内容做好铺垫；

- **scope & closures** -- 顾名思义, 这卷主要聚焦于JS的作用域以及由此衍生出来的闭包概念; 除此之外, 区分了词法作用域、块级作用域、动态作用域等让人混淆的概念, 如果你对于这部分内容有些困惑, 不妨抽出时间来逐字逐句的仔细阅读, 最好辅助代码实例的实现；

- **objects & classes** -- 这一卷其实可以分为两部分, 即 `this` 部分和 `JS原型继承` 部分, this部分实际上是对于前面一章的 `动态作用域` 的延伸, 而原型部分则是作者提出 `OLOO` 概念的地方, 这部将JS原型继承错综复杂的关系进行了逐项梳理, 让我们看到原型继承的本质, 并最终提出了 `Objects Linked to Other Objects` 的设计理念；

- **type & grammar** -- 你是否觉得JS的隐式转换就是一个噩梦?你是否已经投向了 `typescript` 或者 `flow` 的怀抱? 但你是否依然在写代码的过程中不知不觉的用上了隐式转换?! 任何事物存在就有其存在的合理性, 只要花点心思精华和糟粕总能被分辨, 是的，它们就在这一卷中呈现；

- **async & performance** -- 说到JS的异步编程，当然绕不开 ES6+ 提出的 `Promise`、`Generator`、`Async` 等众多新特性，不必再写回调函数的你是否觉得如获新生？而性能优化是编程领域中几乎永恒的话题 —— 只要世界在发展，科技在进步，无论是硬件还是软件都会不断的更新迭代，而你写的程序也要适应这种变化，不断提升性能和效率，而这一卷中提供了很多独到的见解；

- **ES6+** -- 进行中

### 导航
- [开始](/up%20%26%20going/README.md) (up & going)

- [作用域和闭包](/scope%20%26%20closures/README.md) (scope & closures)

- [对象和类](/objects%20%26%20classes/README.md) (objects & classes)

- [类型和语法](/types%20%26%20grammar/README.md) (type & grammar)

- [异步和性能](/async%20%26%20performance/README.md) (async & performance)

- [ES6+](/es6%20%26%20beyond/README.md) (ES6 & Beyond) --- 进行中

**News**：目前该书正在在线编辑最新版，新增加了不少内容，待完成后也会同步至读书笔记中，敬请期待