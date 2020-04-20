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

4. `class` 语法没有提供静态属性的能力，只能实现方法的绑定，但这看似是一个限制，但却符合了大多数场景下 属性(状态) 最好是实例而非类所拥有 —— 如果修改类所拥有的静态属性，那势必会影响到所有它实例化的对象；

5. `extends` 关键字甚至允许你继承内置的对象子类型，比如 `Array`、`RegExp`、`String`、`Number` 等，如果没有 `class … extends …` 这样的语法，想要实现这样的功能是一件很繁琐且容易出问题的事情，有很多第三方库的作者长期为此困扰。但现在这个工作只需一行代码就能轻易完成。

公平的讲，这都是实质性的解决方案，并且让习惯于 类设计模式 的人很开心。

## 抓住你了，类(`class` Gotchas)