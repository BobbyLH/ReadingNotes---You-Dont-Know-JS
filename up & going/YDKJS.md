# 进入YDKJS(Into YDKJS)
这本书到底在讲什么呢？简单来讲，它不是所谓的 *语言精粹(the good parts)*，也不是能让你马上解决工作中遇到问题的工具书；它是严肃的学习关于JS所有的底层原理的一本书。

和其他语言的开发者需要从头到脚的将这个语言吃透彻不同，JS的开发者只需要较少的学习就能从事工作 —— 这显然不是什么好事情，而且更不应该成为常态。

*You Don't Know JS* 会挑战你的 *舒适区(comfort zone)*，深入的对每个JS代码所产生的行为 *问为什么(why)*。这一节是简短的总结这本书的大致内容，为后面的学习夯实基础。

## 作用域和闭包(Scope & Closures)
**变量是如何在作用域中工作** 应该是JS中基础的基础。

*Scope & Closures* 这一节内容一开始就戳穿了JS是 *解释性语言(interpreted language)* 因此不需要编译的误区。JS引擎在一开始的时候就会编译你的代码，*提升(Hoisting)* 是一个JS对于变量作用域管理的典型例子。

*词法作用域(lexical scope)* 是理解 *闭包(closure)* 的基础，闭包的一个重要应用是 *模块设计模式(module pattern)*，这是一个流行的JS代码组织模式，深入理解它应该是你的最高优先级。

## this和对象的原型(this & Object Prototype)
一个关于 `this` 的常见的误区是某个函数内部如果有它的话，它指向的就是这个函数。

实际上，`this` 关键字是会根据调用的方式，动态的绑定上下文的。和 `this` 紧密相关的是 *原型对象机制(object prototype mechanism)*，和词法作用域查找变量类似，该机制实现了一个自动查找对象属性的链条(chain)。

关于JS模拟的实现 *类(classes)*，也叫 *原型继承(prototypal inheritance)*，实际上是一个巨大的误会 —— 关于这个机制更合适的名称应该叫 *行为代理(behavior delegation)*，更多的细节[点这里](../this%20%26%20object%20prototype/README.md)。

## 类型和语法(Types & Grammer)
这一章的关注点主要是在 *类型转换(type coercion)* 上面，特别是 *隐式的类型转换(implicit coercion)*，这或许是JS开发者最为困扰的东西。

大多数人都认为 *隐式转换* 是一个坑，是JS的毒瘤，因此有很多工具不干别的，只会时时刻刻扫描并提醒你的代码里出现了可能的类型转换。

实际上，类型转换是一个被低估的很有价值的工具。只要你深入了解，并且恰当的使用它，它不仅会提升让你的工作效率，更让你的代码质量得到升级，[戳此深究](../types%20%26%20grammar/README.md)。

## 异步和性能(Async & Performance)
前三章的内容聚焦在语言的核心机制上，但第四章的内容集中在管理 *异步编程(asynchronous programming)* 上。

对于一些异步的概念，比如 *async*，*parallel*，*concurrent*……进行了深入的探究和解说。对基于 *回调函数(callback function)* 的 *IoC 控制反转(Inversion of Control)* 异步编程的缺点：*信任缺失(trust loss)* 和 不具备 *线性逻辑能力(linear reason-ability)* 也进行了深入的探讨。

为了解决基于回调函数的异步编程的不足，ES6新增了 *promise* 和 *generator* 两种新的机制。

**Promises** 是一个基于 *未来值* 与时间无关的包装，它能让你不用关心 *未来值* 是否准备好。而且，它有效的解决了 *IoC* 的信任问题。

**Generators** 是JS的一种函数运行的新模式，它能打断函数的运行，在 *generator* 函数内部遇到 `yield` 关键字时，函数会暂停，并将控制权交移出。基于这样的特性，我们能将异步的非线性的代码整合成有顺序且线性的代码。

除此之外，Web Worker、用SIMD实现的并型数据、低水准的性能优化AMS.js都会有所探讨，并且在第六节会基于正确的基准技术对性能优化的点做更多的涉及。

## ES6+(ES6 & Beyond)
这一章涉及到JS未来的规范和特性，虽然本书更多的焦点在ES5中，但是ES6、ES7的新特性才是未来JS的发展方向。

无论你认为自己现在有多了解JS，但是JS的发展不会就此停止，所以千万别停下你前进的脚步。

## 总结(Review)
"You Don't Know JS" 不是批评更不是侮辱，它是我们JS开发者对自我的认知和反省，也是我们必须接受的现实。关键是，虽然现在我们不知道JS，但是我们终将会知道！