# 附录A：ES6 类(Appendix A: ES6 Class)
类的设计模式是可选的，而非必须的 —— 这是 4-6 章节最有价值的内容之一，特别是当你不得不用 `[[Prototype]]` 来实现这种模式的时候。

语法的冗余 只是在JS中使用类的设计模式尴尬的地方之一，除此之外，`.constructor` 这个属性还会让你产生 “被构造” 的错觉；更为重要的是，借用 `[[Prototype]]` 实现的继承本质上并非 **拷贝(Copy)** —— 其本质只不过是代理链接。

OLOO 设计模式则充分利用了 行为代理的特点，将原型的机制摆在台面上，并且极大的简化 —— 没有复杂的父子层级，取而代之的是平级对象直接的关联。

## 类(`class`)
回顾下之前实现的前端组件的业务逻辑，用 ES6 class 来实现的版本：

```js
class Widget {
	constructor (width, height) {
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
	}

	render ($where) {
		if (this.$elem) {
			this.$elem.style.width = this.width + 'px';
			this.$elem.style.height = this.height + 'px';
			$where.appendChild(this.$elem);
		}
	}
}

class Button extends Widget {
	constructor (width, height, label) {
		super(width, height);
		this.label = label || 'Button';
		this.$elem = document.createElement('button');
		this.$elem.innerText = this.label;
	}

	render ($where) {
		super.render($where);
		this.$elem.onclick = this.onClick.bind(this);
	}

	onClick (evt) {
		console.log('Button ' + this.label + ' clicked!');
	}
}

const $body = $('body');
const btn1 = new Button(125, 30, "Hello");
const btn2 = new Button(150, 40, "World");

btn1.render($body);
btn2.render($body);
```

抛开语法层面的优势，ES6的 class 为我们解决了：

1. 不必在使用 `.prototype` 而让你的代码混乱不堪；

2. `Button` 组件借用关键字 `extends` 完成了 “继承” 的动作，避免了使用诸如 `Object.create(…)` 来替换 `.prototype` 关联到的对象，或是直接使用 `.__proto__` 或 `Object.setPrototypeOf(…)`；

3. `super(…)` 提供了使用 **相对性多态(relative polymorphism)** 的能力 —— 任意层级的类的方法都能方便的引用上层类同名的方法，这也解决了 ES5 写法中，构造函数不属于它们的类的奇怪现象(`Parent.call(…)` vs `super(…)`)；

4. `class` 语法没有提供定义 属性 的能力，只能实现方法的绑定，但这看似是一个限制，但却符合了大多数场景下，属性(状态) 最好是实例而非类所拥有的 “最佳实践” —— 如果修改类所拥有的某个属性，那势必会影响到所有它实例化的对象；

5. `extends` 关键字甚至允许你继承内置的对象子类型，比如 `Array`、`RegExp`、`String`、`Number` 等，如果没有 `class … extends …` 这样的语法，想要实现这样的功能是一件很繁琐且容易出问题的事情，有很多第三方库的作者长期为此困扰。但现在这个工作只需一行代码就能轻易完成。

公平的讲，这都是实质性的解决方案，并且让习惯于 类设计模式 的人很开心。

## `class`的坑(`class` Gotchas)
外表光鲜亮丽的 `class` 语法糖，其内部的实现依然是一团糟：

1. 首先，`class` 语法并没有从根本上改变什么，依然只是原型继承的语法糖罢了。这意味着从底层来看，类的继承、实例化等操作，并不会有复制拷贝的动作发生。因此只要你修改了(无论故意还是无意)父类上的某个方法，不仅是其子类，包括子类实例化的对象，可能都会遭受到影响：

```js
class C {
	constructor() {
		this.num = Math.random();
	}
	rand() {
		console.log( "Random: " + this.num );
	}
}

var c1 = new C();
c1.rand(); // "Random: 0.xxxx..."

C.prototype.rand = function() {
	console.log( "Random: " + Math.round( this.num * 1000 ));
};

var c2 = new C();
c2.rand(); // "Random: xxx"

c1.rand(); // "Random: xxx"

```

2. 其次，`class` 语法并不支持定义属性(只支持方法定义)，如果需要在子类之间共享某个属性(状态)，你不得不回退到使用丑陋了 `.prototype` 语法：

```js
class C {
	constructor() {
		C.prototype.count++;

		console.log("Hello: " + this.count);
	}
}

C.prototype.count = 0;

var c1 = new C();
// Hello: 1

var c2 = new C();
// Hello: 2

c1.count === 2; // true
c1.count === c2.count; // true
```

当然，`this.count++` 可以实现在状态的隔离，能在实例化的对象中各自保存各自的状态。但 `class` 语法之所以不提供定义类的属性，很有可能是暗示你别使用这个特性，因为这可能会导致意料之外的Bug发生。

3. 再之，还有更恶心的情况：

```js
class C {
	constructor(id) {
		this.id = id;
	}
	id() {
		console.log("Id: " + this.id);
	}
}

var c1 = new C("c1");
c1.id(); // TypeError
```

👆如果 类的方法名 和 实例化对象的属性名 一旦有冲突，那么结果可想而知 —— `c1.id` 此时是一个字符串而非函数，用函数调用表达式来访问它，肯定会抛出 `TypeError` 的错误。

4. 这些都不算，`super` 关键字的工作机制一定会让你大跌眼镜：

```js
class P {
	foo() { console.log("P.foo"); }
}

class C extends P {
	foo() {
		super.foo();
	}
}

var c1 = new C();
c1.foo(); // "P.foo"

var D = {
	foo: function() { console.log("D.foo"); }
};

var E = {
	foo: C.prototype.foo
};

Object.setPrototypeOf(E, D);

E.foo(); // "P.foo"
```

👆发现哪里不对了么？`E.foo();` 居然输出的是 `"P.foo"` 而不是意料中的 `"D.foo"`！

`super` 关键字和 `this` 的工作机制不一样，后者是动态绑定执行的上下文，而前者则是在语句创建时就静态绑定好了其指向的上下文 `C.prototype` 即 `P`，因此 `E.foo();` 的执行结果是 `"P.foo"` 而非 `"D.foo"`。

在使用 `super` 的时候，你要自己留意它的执行上下文，有时候你甚至得手动改变它的指向，或者重写这个方法，Ugh!

## 静态 > 动态?(Static > Dynamic?)
ES6 的 `class` 带来的最大的问题其实是让你产生了一种心理暗示 —— 你正在用 类的设计模式 来书写代码，这一切都好像是静态定义的规范，就好像是传统的面向类的语言一样，你甚至失去了对其本质的洞察力 —— 在JS中，一切都是对象，`class P…` 本质上也是一个对象，它是流动的存在，你可以在任意时间和它动态的交互。

但 `class` 语法好像在告诉你千万别这么做(虽然这是一个强大但充满危险的能力)，否则你只能用丑陋的 `.prototype` 的语法来实现动态的交互，又或者用 `super` 来限制你动态改变上下文的自由……

换句话说，`class` 语法是在告诉你：“动态的能力很危险，让我们来假装这是门静态语言”。但这样只会把水搅浑，让JS越来越难以被真正的理解。

**注意**：如果 `extends` 关键字后是一个使用 `bind(…)` 绑定后的函数，那它的行为和普通的函数有很大的差异。

## 回顾(Review)
`class` 的语法糖看上去外表光鲜亮丽，但其本质却是 **包藏了大部分的缺陷和问题，但却引入了更微妙且危险一个语法**。

`class` 并没有从根本上解决JS的继承方式，依然是依靠原型继承，来伪装自己显得像是和其他语言一样，拥有漂亮的语法的来帮你完成继承、实例化的各种操作。

那么，如果 ES6 的 `class` 并不能带来本质上的改变，而是让你的代码更复杂，更脆弱，那么我们为什么不看一看其依赖的底层能力 —— **动态的联接其他对象** 实际上是一个更为出色的设计模式呢？

当然，这本书只是提供了一些信息，当你深入思考这些问题的时候，你会找到属于你自己的答案。