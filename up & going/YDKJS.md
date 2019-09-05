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