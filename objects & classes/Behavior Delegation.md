# 行为代理(Behavior Delegation)
虽说在前面的一章已经介绍了不少有关于JS中的 `[[Prototype]]` 的机制，也探讨了为什么使用它去模拟 “类” 和 “继承” 是利大于弊的情况 —— 不仅是 `.prototype` 胡乱堆放，更有使用 `.constructor` 试图去解决令人恶心的伪多态语法，以及各种 “混合(mixin)” 的尝试……

你可能想说：到此为止了！但如果你不继续深入，而是将这些现象当做理所当然的 “黑盒(black box)”，那你可能会错过习得在JS中使用一种更为直观、间接的设计模式，它会让你的代码更清晰明了、易于维护。

再次回顾一下 “原型链(prototype chain)” 的机制：在JS的对象中，存在一个内部的连接，指向另外某个对象；当引用某个对象中并不存在的方法或属性时，通过这个内部的连接(即原型对象 prototype)，查找另外一个关联对象上是否存在这个属性或方法；如果没有找到，则会继续查找该对象上的原型对象，直到查询到结果或者抵达 `Object.prototype` 为止…

👆由此可见，关键的机制是 **对象能关联到其他对象** —— 这是帮助我们理解后面内容的基础和关键。

## 面向代理设计(Towards Delegation-Oriented Design)
磨刀不误砍柴工，为了能更好的聚焦于 `[[Prototype]]` 的使用，先让我们看看它和类的设计模式到底有什么不同。

**Note**：有一些面向类的设计理念依然很有用，比如 *封装(encapsulation)* 就能兼容于面向代理的设计模式 —— 别把所有的东西都扔掉。

### 类的原理(Class Theory)
假设我们有几个相似、重复的任务需要处理，这时候需要对这些任务的模型进行抽象。

如果使用类的设计模式，通常应该包含如下步骤：定义一个父(基)类 `Task`，它包含了能被其他子类共享的各种行为；接着定义子类 `XYZ` 和 `ABC`，它们都继承于父类 `Task`，并且都添加了自己的特定的行为去处理各自的任务。

关键在于，类设计模式会鼓励你尽可能多的去利用继承带来的便利。而当需要处理特定的行为时，使用多态(polymorphism)或者改写(override)的特性，如使用诸如 `super` 关键字，来实现。可以说 **抽象通用的行为** 和 **改写特定行为** 在面向类的设计模式中无处不在。

比如下面的伪代码：
```js
class Task {
	id;

	// constructor `Task()`
	Task(ID) { id = ID; }
	outputTask() { output( id ); }
}

class XYZ inherits Task {
	label;

	// constructor `XYZ()`
	XYZ(ID,Label) { super( ID ); label = Label; }
	outputTask() { super(); output( label ); }
}

class ABC inherits Task {
	// ...
}
```

用 `ABC` 或 `XYZ` 实例化后得到的实例，会从各自的类以及类继承的父类中复制拷贝行为；最终它们只能和自身进行交互而互不干涉，每个实例都有执行任务所需要的行为的全部副本。

### 代理的原理(Delegation Theory)