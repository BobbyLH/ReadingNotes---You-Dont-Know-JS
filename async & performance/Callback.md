# 回调函数(Callback)
“回调函数” 是JS中异步编程的基础。有很多的JS程序都是以此来实现异步的，比如第一章中的定时器、网络请求等功能的实现。尽管它并非没有缺点，但更高级的抽象API比如 `Promise`，也是基于它来实现的。如果你不理解它，那就没办法有效的使用在它基础上建立的其他抽象工具。

## 续集(Continuations)
简单的回顾下第一章的 ajax 网络请求：

```js
// A
ajax('…', function () {
  // C
});
// B
```

👆 `// A` 和 `// B` 是程序执行的前半部分，`// C` 是后半部分 —— 前者会立即执行，而后者则要等待一段不确定的时间后，在未来的某个时刻，才会被执行。

换句话说，回调函数封装了一段程序的 *“续集”*。

再来一段更简单的：

```js
// A
setTimeout(function () {
  // C
}, 1000);
// B
```

👆如果让你用大白话把这段代码的执行描述出来，你也许会这么想：“先完成A，而后执行一个要等待1000毫秒的定时器，接下去完成B，而后等到1000毫秒之后，完成C”。

觉着差不多吧？但实际上不够准确。而这些差异正是我们理解 回调函数在表达和管理异步编程的*缺陷* 的核心所在。一旦我们以回调函数的形式引入了多个异步，我们的大脑就不够用了 —— 它的工作方式和代码的执行方式产生了巨大的分歧，而这会使得我的的代码变得难以理解、调试和维护。

## 大脑的顺序(Sequential Brain)
别以为你有一个 “多线程” 的大脑，我们的注意力是有限的。抛开无意识、潜意识、条件反射等功能(心跳、呼吸、眼跳)，我们的大脑是一个一次只能执行单一任务的“机器”。所有你认为你能同时并行处理好几个任务的幻觉，其实都是你大脑在快速的切换它的“上下文”，让你产生你能多任务处理的错觉 —— 这是否听上去有点像JS中是实现异步的事件循环机制呢？如果不是的话，建议回头再看看第一章的内容。

### 做比较(Doing Versus Planing)
我们写代码的时候，会仔细的按照计划的顺序来书写一组要执行的操作(A -> B -> C)，当我们逐条语句书写出同步执行的代码时，它和我们的 todo-list 就很类似：

```js
// 将 x 和 y 的值交换，并用 z 作为保存数据的临时变量
z = x;
x = y;
y = z;
```

👆它们的执行过程一定是一个接一个的执行，很符合我们大脑的计划工作模式。

虽然同步代码能够很好的和我们的大脑相互映射，但异步代码就难以做到这一点了。比如，你能想象一下让你的脑中出现了这些乱七八糟的计划安排时，你是什么感受么？：

> 我准备先去一下超市；但是路上接到了一个电话：“有啥事？老妈”；当她开始唠唠叨叨的时候，我准备先查一下超市的GPS定位；当等待查询结果的时候，我把收音机的音量调小了一些，以便于能够更清楚的听到老妈在说什么；这时候突然想起来我的外套忘带了，不过没关系，开车过去应该问题不大；继续和老妈聊着，突然安全带未系的提升声音响了，赶紧系上；“老妈我一直都系好安全带的，放心吧！” 而后GPS工作了，开始出发……

👆这些乱七八糟的计划安排本质上就是我们大脑工作的方式，并且要记住的是，这不是多任务并行处理，而只是快速的上下文切换罢了。

“意识流” 没有一点顺序可言，这也是为什么一旦用这种模式去实现异步代码，就会让它变得难以理解的原因。因为对于大多数的人来讲，标准的思考的方式是一步一步按计划来的，但是回调函数让我们编写的代码脱离了这个模式，变为了同步和异步的意识流模式。而这也是为什么用回调函数来实现的异步有时候会变得如此难以理解 —— 它和我们大脑的工作方式不符。

比你不知道为什么代码挂了还要衰的事情是 —— 你不知道为什么这段代码居然会第一时间执行。Sartre 说：'Hell is other people'，程序员们可能会说：'Hell is other people's code'，但我更相信：'Hell is not understanding my own code'。而这一切的万恶之源就是回调函数。

### 嵌套/链式 回调函数(Nested/Chained Callback)
嵌套的回调函数只是“红鲱鱼” —— “回调地狱(callback hell)” 才是主题。

```js
window.addEventListener('click', function handler(evt){
	setTimeout(function request(){
		ajax('http://some.url.1', function response(text){
			if (text == 'hello') {
				handler();
			} else if (text == 'world') {
				request();
			}
		});
	}, 500);
});
```

👆上面一共有三层嵌套，是 “回调地狱” 的常见形式，也被称为 “末日金字塔(pyramid of doom)” —— 呈现了一种根据嵌套缩进的 “侧面朝向的三角形” 图形，塔尖在最右侧。

但 “回调地狱” 最大的问题可不是在于 “形状”，不过先让我们来看看上述例子的整个流程：

首先，我们等待 “点击事件” 的触发：

```js
window.addEventListener('click', function handler(evt){
  // ...
})
```

而后，触发一个定时器：

```js
setTimeout(function request(){
		// ...
}, 500);
```

接下去发起一个 ajax 请求：

```js
ajax('http://some.url.1', function response(text){
	// ...
});
```

最后，进入 `if...else...` 的条件判断：

```js
if (text == 'hello') {
	// ...
} else if (text == 'world') {
	// ...
}
```

这样一步步的分析下来看上去还行？！好吧，那再看看下面的一段伪代码：

```js
doA(function(){
	doB();

	doC(function(){
		doD();
	})

	doE();
});

doF();
```

你能瞟(glanced)一眼就说出它的运行顺序么？反正我至少要在脑子里过一遍：

- `doA()`

- `doF()`

- `doB()`

- `doC()`

- `doE()`

- `doD()`

好吧，为了减少所谓的字母干扰，我们在来一遍：

```js
doA(function(){
	doC();

	doD(function(){
		doF();
	})

	doE();
});

doB();
```

嗯，现在的顺序成了：`A -> B -> C -> D -> E -> F`...但你的眼睛肯定要来回瞟吧？！

就算你已经很熟练且自然了，那若是这些都是同步而非异步呢？最终的顺序就得是 `A -> C -> D -> F -> E -> B`。

回到一开始的 event/timeout/Ajax 嵌套来：

```js
window.addEventListener('click', handler);

function handler() {
	setTimeout(request, 500);
}

function request(){
	ajax('http://some.url.1', response);
}

function response(text){
	if (text == 'hello') {
		handler();
	}
	else if (text == 'world') {
		request();
	}
}
```

👆上面这样一拆分，好像清晰优雅了很多？！但真实的情况是嵌套往往超过三层，而且一旦某一层出了问题，这样的连续就会被中断。而若是想要新增一层嵌套的代价往往是巨大的。而且从我们大脑的工作模式来看(按顺序且阻塞)，这样的 “异步回调模式” 的代码很是让人费解 —— 因为我们先得把它转换成同步的代码。

## 信任问题(Trust Issues)
信任问题，这显然是更为严重的问题。比如回到上面例子中的 ajax 网络请求一段：

```js
// A
ajax('http://some.url.1', function response(text){
	// B
});
// C
```

`// A` 和 `// C` 显然在 `// B` 之前就执行完毕了，但 `// B` 到底什么时候执行是一个未知数 —— 通常情况这种将控制权交出去的做法不会导致什么大问题，但这正是回调模式驱动的最大问题所在 —— 这种被称为 “控制反转(IoC inversion of control)” 的情况一旦发生在脱离你控制的第三方库手中，那么你们之间就只剩下一份薄薄的 “没被承认过的协约”，你只能期待事情会按照你的预期进行下去。

### 五次调用的故事(Tale of Five Callbacks)
这可不是什么好故事，不过若是你提前知道了你也可能会发生这种情况的话，就能尽量避免一些不必要的Bug，故事是这样的：

话说假设你是一家电商购物的程序员，在你们平台上最后的交易环节，依赖一个第三方的大数据分析商家提供的数据追踪工具，你写的代码可能长这样：

```js
analytics.trackPurchase(purchaseData, function(){
	chargeCreditCard(); // 扣款的方法
	displayThankyouPage(); // 支付成功页面
});
```

代码上线几个月平稳运行，皆大欢喜。

但6个月后的某天早上，当你正享受你的拉铁咖啡，悠闲的开始一天的工作时，你的老板突然急急忙忙的打了一个电话来，说有一件紧急情况需要处理。于是你赶紧跑到他的办公室，里面还有客户专员。情况原来是有一个用户在购买一台很贵的 iphone 时，他的银行卡被刷了五次？！可是他只买了一台手机...

虽然用户收到了退款，而且还赠送了一些小礼物，但你的老板坚持要你查一查扣款的日志记录。你很不情愿的连接好了后台管理系统，打开后惊出一身冷汗 —— 那段数据分析的回调居然被调用了五次！！！你迫不及待的联系了供应商，而他们给的反馈居然是：“很抱歉，之前有个新人将测试环境的代码发布到生产环境了，我们已经第一时间修复了，感谢您的支持，祝您生活愉快，再见！”

WTF？你简直不能忍受，于是决定将它们 “纳入黑名单”：

```js
var tracked = false;
analytics.trackPurchase(purchaseData, function(){
  if (tracked) return;
  tracked = true;
	chargeCreditCard(); // 扣款的方法
	displayThankyouPage(); // 支付成功页面
});
```

哼哼，看你小子还敢给我搞什么妖蛾子！ —— 但是...和你搭档的测试妹子突然说：“要是他们不调用这个函数怎么办？！”

于是你突然发现有很多种会导致Bug的情况：
  - 过早地调用了 callback

  - 过迟地调用了 callback

  - 调用了太多次

  - 回调参数不正确

  - 错误情况被“吃”了

  - ...

而后你突然意识到，程序里面这样的地方还有很多...这已经不单单是 “回调地狱” 的问题那么简单了！

### 不仅仅是别人的代码(Not Just Other's Code)
“永远不要信任别人的代码” 是一个解决信任问题的办法。

但认真反思一个问题 —— 你真的能信任能由你控制的代码吗？

太自信了：
```js
function addNumbers(x, y) {
	return x + y;
}

addNumbers(21, 21);	// 42
addNumbers(21, '21'); // '2121'
```

加一层 “防御”：
```js
function addNumbers(x, y) {
  if (typeof x !== 'number' || typeof y !== 'number') {
		throw Error('Bad parameters');
	}

	return x + y;
}

addNumbers(21, 21);	// 42
addNumbers(21, '21'); // Error 'Bad parameters'
```

或者更友好一些：
```js
function addNumbers(x, y) {
  typeof x !== 'number' && (x = Number(x));
  typeof y !== 'number' && (y = Number(y));
	return x + y;
}

addNumbers(21, 21);	// 42
addNumbers(21, '21'); // 42
```

但是难道说我们每一个函数都得加上这些检查吗？！是的！不过回调函数在这件事情上毫无建树 —— 都得靠你自己。特别是在你的代码依赖外部不受控制的第三方库时 —— 潜在的 Bug 难道就不是 Bug 了？！

## 回调函数：救救孩子吧(Trying to Save Callbacks)
在ES6之前，社区里有尝试多种解决回调函数信任问题的方案，但它们注定不会成功是因为它们都试着从回调函数的内部来解决问题，比如将 “成功” 或者 “失败” 的回调分开：

```js
function success(data) {
	console.info(data);
}

function failure(err) {
	console.error(err);
}

ajax('http://some.url.1', success, failure);
```

👆 `failure` 一般会被设计成可选项，若是发生了错误但没提供，那么错误会被忽视掉。

另一种常用的模式是 “错误优先(error-first-style)” 模式，也被称为 “Node Style” 模式 —— 因为在 nodejs 中的绝大部分API都是采用了这种模式。这种模式的回调函数的第一个参数是用于接受错误的 —— 若是成功则是 falsy 类型的值(其他参数紧随其后)，若是有错误就是 truty 类型的值(其他参数通常为空)：

```js
function response(err,data) {
	if (err) {
		console.error(err);
	}
	else {
		console.info(data);
	}
}

ajax('http://some.url.1', response);
```

 但无论是那种对回调函数进行的改良，依然没能解决本质的问题：对于 IoC 的信任问题，没有哪怕提供一个工具或是过滤器来避免重复的调用，并且你依然还得同时编写处理“成功”和“失败”的代码。同时这样的 “模板” 会让你一遍又一遍的重复这些动作，你甚至不得不自己写一个工具来确保信任问题，比如使用 timeout 的机制：

 ```js
function timeoutify(fn, delay) {
	var interval = setTimeout(function () {
    interval = null;
    fn(new Error('Timeout!'));
  }, delay);

	return function () {
		// 只有未超时才会被调用
		if (interval) {
			clearTimeout(interval);
      interval = null;
			fn.apply(this, [ null ].concat([].slice.call(arguments)));
		}
	};
}
```

而后这样使用：
```js
function foo(err,data) {
	if (err) {
		console.error(err);
	}
	else {
		console.info(data);
	}
}

// 这样一来就能确保回调函数只会被调用一次
ajax('http://some.url.1', timeoutify( foo, 500 ));
```

针对另外一个可能出现的 “过早调用” 的信任问题，即你不能确定交由第三方的回调函数到底是 “同步” 还是 “异步” 被触发时，可以写另一种“规范行为”的工具来避免：

```js
function asyncify(fn) {
	var orig_fn = fn;
	var interval = setTimeout(function(){
    interval = null;
    if (fn) fn();
	}, 0);

	fn = null;

	return function() {
		// 若是 interval 存在，即意味着说此时的回调函数是同步调用的
		if (interval) {
			// 将this、参数通过 bind 绑定到即将在setTimeout触发的回调函数上
			fn = orig_fn.bind.apply(
				orig_fn,
				[this].concat([].slice.call(arguments))
			);
		}
		// 异步调用，正常触发回调
		else {
			orig_fn.apply(this, arguments);
		}
	};
}
```

而后若是碰到了👇这样的情况，就不必担心 `console.info(a);` 输出的会是 `0` 还是 `1` 了：

```js
function result (data) {
	console.info(a);
  // ...
}

var a = 0;

ajax('..pre-cached-url..', asyncify(result));
a++;
```

👆好像信任问题解决了？！但你一旦想到你必须一遍又一遍的将这些工具应用到你的代码中时，你一定会很期待有原生就支持的 API 来解决这些个 “八股文” 吧？！

## 回顾(Review)
回调函数是 JS 异步编程的基础，但它本身的局限，不能很好的满足和支持日渐庞大的异步程序的开发。

究其根本是因为我们人类的大脑工作模式就和回调函数相违背，换句话讲，回调函数就是“反人性”的存在。我们需要一种和我们大脑工作模式接近的异步API来解决这些问题。

另一方面，也是更为关键的，回调函数受到了 *控制反转(inversion of control)* 的荼毒，从而导致了一系列的信任问题产生，虽然使用一系列临时的措施来解决这些问题是可行的，但这会让你的代码变得臃肿和难以维护。

显然这些痛点逼迫着一些更好用的内置 API 的到来，而这就是接下去要涉及到的部分。