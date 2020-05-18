# 异步：现在 & 未来(Asynchronv: Now & Later)
异步编程在 JS 中是个绕不开的话题，如何使用在整个程序的生命周期中合理的使用它，是一个非常重要但同时也会导致很多误区的地方。

**异步** 说的不是 `for` 循环中从开始到结束所花费的几微秒或者几毫秒，它指的是你的一部分程序在当前运行，而另一部分程序则会在未来某个确定或者不确定的时间中运行 —— 而且在这两块程序中，还有一段时间是你的程序不会运行的 *沟(gap)*。

无论是 *用户的输入*、*从文件系统或数据库中读取数据*、*网络请求* 亦或 *定时不间断的执行某个动画*……这些都要求你的程序能够很好的组织和管理这些跨越 *沟* 的状态，就如同伦敦地铁站里的那句提示一样："mind the gap"。

**现在和未来的关系管理，是异步编程的核心所在**。

在 ES6 之前，JS 中异步的实现只能通过 *回调函数(callback)* 来实现，虽然到今天还是有人认为这已经足够了，但是随着 JS 能够运行在各种浏览器、服务器、客户端，甚至是各种IoT的设备中的时候，异步的管理变得越来越艰难，各种呼吁采取更有力、更合理的异步管理方式不断被提出。

后面的章节中我们会一一探索 JS 中各种异步编程的技术，但在这之前，我们要回归本质，理解到底什么是异步。

## 一个程序块(A Program in Chunks)
哪怕你的 JS 脚本程序只有一个文件，但你的程序在大多数情况下都会有几个 *代码块(chunk)* 组成 —— 只有一个代码块是当前立即执行的，而其余的代码块在未来的某个时间执行。而组织这些代码块最常用的工具就是 `function` 无疑了。

大多数刚开始接触JS的开发者面对 异步 的疑惑，很可能是编写的异步代码没办法立即完成，从而获取到期望的结果：

```js
var data = ajax('http://some.url'); // 假定 ajax 是由某个第三方库提供的异步http请求方法

console.log(data); // data 不是期望的 ajax 返回的值
```

👆直觉上第一时间可能会以为 `data` 会接受到 `ajax` 请求的结果，但要实现这个的前提是 `ajax` 会阻塞后面的代码执行，直到获取到http网络请求的结果，而后进行赋值的动作。但实际上，Ajax 是异步的请求，即在执行了 `ajax(…)` 之后并不会立即返回请求的结果，而是需要提供一个函数，通常情况下这个函数被称为 *回调函数(callback function)*，当网络请求返回结果后，再调用这个函数：

```js
ajax('http://some.url', function callback (data) {
  console.log(data); // 正确获取到网络请求的数据
});
```

**Note**：即便是你能够让 Ajax 的网络请求变成同步代码也尽量别这么做，因为这种阻塞不仅会让浏览器中的 UI 渲染受阻，还会严重的影响用户交互体验，这是可怕的想法，快点把它从脑海中剔除掉！

如果想要避免各种回调函数把自己绕晕，一个不错的办法是让这些包装代码块的函数都有语义清晰名字：

```js
function now () {
  return 21;
}

function later () {
  answer = answer * 2;
  console.log('The answer is: ', answer);
}

var answer = now();

setTimeout(later, 1000);
```

👆上面的代码其实可以看做两块，同步执行的代码块：

```js
function now () {
  return 21;
}

function later () {//……}

var answer = now();

setTimeout(later, 1000);
```

1000毫秒后执行的异步的代码块：
```js
answer = answer * 2;
console.log('The answer is: ', answer);
```

一旦你将某部分代码块放入某个函数中，并将这个函数作为回调函数(比如定时器、事件、Ajax请求)调用时，你其实就使用了异步编程。

### 异步的控制台(Async Console)
针对于 `console.*` 这个宿主环境实现的对象(BOM对象)而言，并没有一个清晰且准确的说明或者文档来定义它到底如何工作 —— 因此不同的JS执行的宿主环境，要如何实现这个功能，真的是随他们的便了(as the please)。

在某些浏览器中的某些条件下，你可能会惊奇的发现 `console.log(…)` 居然不会立即执行，并输出你期望的结果。导致这个现象的主要原因是 I/O 操作是一件非常耗时且会阻塞程序运行的操作(不仅仅JS如此)。因此，出于对性能的考虑，浏览器将 `console.log(…)` 这种 I/O 实现为异步的也没什么好奇怪的。

比如👇：

```js
var a = { index: 1 };
console.log(a);
a.index++;
```

若是将这些代码一股脑的贴到浏览器的控制台中，而后在某些浏览器下，打印出的内容很可能会让你大吃一惊 —— 展开对象 `a` 发现其中的属性 `index` 居然是 `2`。

大多数时候 `console.log(…)` 输出的结果都和我们预期的一致(比如输出的是一个基本数据类型)，不过有时候当浏览器认为你的代码需要被推迟在执行的时候，就如这个例子中输出的是一个对象的时候，当你兴致勃勃想要查看某个对象，从而debug的时候，突然发现这个结果居然已经被修改了……

**Note**：选用 *打断点(breakpoints)* 而不是用 `console.log(…)` 来 debug，因为后者某时候会不太靠谱。如果非得用 `console.log(…)`，记得把你要输出的对象先 “截个图(snapshot)”，比如用 `JSON.stringify(…)` 将其序列化，而后靠谱的输出结果。

## 事件循环(Event Loop)
尽管异步的代码在JS中随处可见，但直到 ES6 之前，Javascript 都没有一个内置的异步实现。JS 引擎本质上只会在 “被要求(when asked to)” 的时候执行单个的代码块。而 “被谁要求？” —— 这才是关键。

JS 引擎通常情况下并不能单独的运行，它需要搭配一套宿主环境才能运行，对大多数开发者来讲，它一般是浏览器。但随着最近这些年的发展，JS 已经能够运行通过比如 Node.js，运行在服务器上，甚至还被植入到各种各样的设备中，从机器人到电灯泡……

在所有的这些环境中运行的多个代码块，都是通过一个线程完成的，而处理这些代码块的机制被称为 “事件循环(event loop)”。

换句话讲，JS引擎本身对时间并不怎么敏感，它本质上只是按需执行宿主环境中交派给它的JS代码块，宿主环境的工作则是计划和安排 “事件”。

例如，当你想要程序执行一个网络请求从某个服务器获取数据的时候，你通常会设置一段代码置于某个函数中，此时JS引擎通知宿主环境这个函数是需要等到网络请求完毕，获取到数据后再调用的，而宿主环境(浏览器)则会监听网络请求的响应，一旦有数据回来，它就会将这个函数执行插入到事件循环中。

一段简单的伪代码有助于你理解 “事件循环”：

```js
var eventLoop= []; // 事件队列，满足先进先出
var event;

while(true) {
  // tick
  if (eventLoop.length > 0) {
    event = eventLoop.shift();
    // 执行事件
    try {
      event();
    } catch (err) {
      reportError(err);
    }
  }
}
```

👆我们假设 “事件循环” 是运行在一个无限循环的 `while` 循环体中，每次迭代则被称为 “tick”，每一次 tick 都会检查事件队列中是否有排队等候的事件(如同你定义的回调函数)，若是有，则按照先进先出的原则，一个个的执行这些事件。

需要强调的是，`setTimeout(…)` 并不会立马将你的传入的回调函数放入到事件队列中，而是先设置一个计时器，当计时器到点了，才会让宿主环境将你的回调函数加入到事件循环中，而后未来的某一次 tick 将会获取并执行这个回调函数。如果前面已经有排队的事件，你的回调函数必须等到这些事件都执行完毕，才轮到你 —— 没有办法能够插队。这也解释了为什么 `setTimeout(…)` 计时器可能不会精确的按照你设定的时间执行，只能保证说一定不会早于设定的时间，但执行的时间取决于事件队列的状态。

从这个角度来看，你的程序本质上被切分成一个个的小块，一些小块会在另一些小块执行完成之后才会执行。从技术上来看，其他一些和你程序不相干的事件，也可能通过事件队列和你的程序交错执行。

**Note**：ES6中明确的规定了 “事件循环” 应该如何工作，这也意味着以后 “事件循环” 应该属于 JS引擎的范畴，而不是宿主环境。而带来这些改变的源头是ES6中引入的 Promises 机制，它要求能够有更细的颗粒度对事件循环进行控制，第三节会详细讨论。

## 平行线程(Parallel Threading)
“异步(async)” 和 “并行(parallel)” 是两个会经常被混为一谈的术语，但实际上它们有相当的不同 —— “异步” 是关于 当前(now) 和 未来(later) 的时间差，而 “并行” 指的是同时发生的事情。

在计算机中处理并行常见的工具是 进程(processes) 和 线程(threads)，它们的执行过程是独立的，并且很有可能会同时发生，在不同的处理器，甚至是不同的计算机上，单个进程下的每个线程都能够共享其内存。

在事件循环的模式中，会将工作拆解成若干子任务，然后按照顺序执行，且在这种模式下不允许并行的访问或更改共享的内存。但无论并行还是串行的模式，都能单独的线程中以事件循环的形式共存。

```js
function later () {
  answer = answer * 2;
  console.log('The answer is:', answer);
}
```

👆可以将 `later()` 中的内容都视为是事件循环队列中的一个单独的条目。其中 `answer = answer * 2;` 是属于底层的操作(low-level operations)：首先先从 `answer` 中获取到当前它存储的值，而后将 `2` 放入到某块内存中，紧接着在执行乘法运算操作，最后将运算的结果保存到 `answer` 中。

在单线程的环境中，上面这一切都不是问题，因为这个过程没办法被打断。但是若这段代码运行在多线程的环境中，就会导致一些意料之外的结果：

```js
var a = 20;

function foo() {
	a = a + 1;
}

function bar() {
	a = a * 2;
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", foo);
ajax("http://some.url.2", bar);
```

根据 JS 单线程的行为可以推断出上面👆的代码块至多只会产生两种不同的结果：
  1. 如果 `foo()` 先运行，那么 `a` 的结果是 `42`；

  2. 如果 `bar()` 先运行，那么 `a` 的结果是 `41`；

但若是上面的代码运行在多线程的环境中，那就有可能 `foo()` 和 `bar()` 会同时运行，这就会产生非常多的可能性：

  线程1(`X` 和 `Y` 是两处临时的内存地址)：
    ```
    foo():
      a. 从 `X` 中获取到 `a` 的值
      b. 将 `1` 存到 `Y` 中
      c. 将 `X` 和 `Y` 的值相加，而后将结果储存到 `X` 中
      d. 将 `X` 的值存储到 `a` 中
    ```

  线程2(`X` 和 `Y` 是两处临时的内存地址)：
    ```
    bar():
      a. 从 `X` 中获取到 `a` 的值
      b. 将 `2` 存到 `Y` 中
      c. 将 `X` 和 `Y` 的值相乘，而后将结果储存到 `X` 中
      d. 将 `X` 的值存储到 `a` 中
    ```

👆问题的关键是两个线程在同一时间，都访问和修改了 `X` 和 `Y` 两处临时的内存地址。因此 `a` 的值会有多种情况：

  情况一，`a` 的值最终为 `44`：
    ```
    1a  (从 `X` 中取出 `a` 的值 ==> `20`)
    2a  (从 `X` 中取出 `a` 的值 ==> `20`)
    1b  (把 `1` 存到 `Y` 中 ==> `1`)
    2b  (把 `2` 存到 `Y` 中 ==> `2`)
    1c  (将 `X` 和 `Y` 的值相加，而后将结果储存到 `X` 中 ==> `22`)
    1d  (将 `X` 的值存储到 `a` 中 ==> `22`)
    2c  (将 `X` 和 `Y` 的值相乘，而后将结果储存到 `X` 中 ==> `44`)
    2d  (将 `X` 的值存储到 `a` 中 ==> `44`)
    ```

  情况二，`a` 的值最终为 `21`：
    ```
    1a  (从 `X` 中取出 `a` 的值 ==> `20`)
    2a  (从 `X` 中取出 `a` 的值 ==> `20`)
    2b  (把 `2` 存到 `Y` 中 ==> `2`)
    1b  (把 `1` 存到 `Y` 中 ==> `1`)
    2c  (将 `X` 和 `Y` 的值相乘，而后将结果储存到 `X` 中  ==> `20`)
    1c  (将 `X` 和 `Y` 的值相加，而后将结果储存到 `X` 中 ==> `21`)
    1d  (将 `X` 的值存储到 `a` 中 ==> `21`)
    2d  (将 `X` 的值存储到 `a` 中 ==> `21`)
    ```

这种无法确定的情况会让人很头疼，好在 JS 是单线程。不过依然会存在之前提及的 `race condition` 的情况。

### 一口气运行完(Run-to-Completion)
正因为JS是单线程语言，因此 `foo()` 和 `bar()` 具有原子性 —— 这也就意味着一旦它们开始执行时，除非完成，否则不会被打断。这也是为什么👇下面的代码至多只有两种结果的原因：

```js
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", foo);
ajax("http://some.url.2", bar);
```

将👆上面代码拆分成三个块，第 1 块是一个同步执行的块，第 2 和 3 块是异步执行的块(即它们执行会有时间差)：

Chunk 1：
```js
var a = 1;
var b = 2;
```

Chunk 2(`foo()`)：
```js
a++;
b = b * a;
a = b + 3;
```

Chunk 2(`bar()`)：
```js
b--;
a = 8 + b;
b = a * 2;
```

Chunk 2 和 Chunk 3 的执行顺序会对结果造成影响。但这个颗粒度已经是在 *函数层面* 而非 *语句表达式层面*，虽然 *函数层面* 的执行顺序不确定性也会造成 *race condition*，而只要能有效控制 *函数层面* 的执行顺序，就能解决这个问题。相关的讨论会放在后面的章节中。

## 并发(Concurrency)
试想一下有这样一个场景：一个网站实现了滚动懒加载后续内容，即当用户触发了浏览器的滚动事件之后，会发送Ajax请求获取到数据，从而渲染真实的UI。为了说明起见，我们将这个功能拆分成两个 “进程”，第一个 “进程” 用于响应 `onscroll` 事件，即根据用户的滚动位置从而发送 Ajax 请求；第二个 “进程” 用于接收 Ajax 请求的数据，从而渲染正确的内容。

**Note**：👆使用 “进程” 只是为了和后续的内容接洽，保证 “术语” 一致(terminology-wise)，而不是说这真的就是计算机系统层面的进程。你可以将它理解为是一个虚拟的进程，或者任务，说白了就是逻辑的连接以及各项操作按顺序的运行。

“并发” 在这个场景下很实用：用户滚动浏览器并触发Ajax请求 和 接受到Ajax请求并渲染视图 两者之间并不冲突，它们可以同时执行(当然这是在 任务/进程层面，而非 操作系统层面)：

“进程”一(`onscroll` 事件，并触发Ajax请求)：
```
onscroll, request 1
onscroll, request 2
onscroll, request 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
onscroll, request 7
```

“进程”二(响应Ajax请求，并渲染视图)：
```
response 1
response 2
response 3
response 4
response 5
response 6
response 7
```

而它们真实发生的顺序，完全有可能是下面这样：
```
onscroll, request 1
onscroll, request 2  response 1
onscroll, request 3  response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6  response 4
onscroll, request 7
response 6
response 5
response 7
```

但是回到 JS 的事件循环的特性，作为单线程脚本语言的它在同一时刻只能操作一个事件，因此无论 `onscroll, request 2 ` 和 `response 1` 哪一个先发生，它们都不能在同一时刻发生，而最多只能在同一个任务队列里面，排队等待处理。

因此，在 JS 中真实的顺序应该是这样：
```
onscroll, request 1   <--- Process 1 starts
onscroll, request 2
response 1            <--- Process 2 starts
onscroll, request 3
response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
response 4
onscroll, request 7   <--- Process 1 finishes
response 6
response 5
response 7            <--- Process 2 finishes
```

👆“进程”一 和 “进程”二 在任务层面是并发的，即会一起进入任务队列。随带一提的是 `response 4` 和 `response 5` 的返回的前后顺序并没有按照它们发送时的顺序进行，这就引入了后面要谈到的 *race condition* 问题。

### 非交互(Noninteracting)
如果上面提到的几个“进程”之间都没有任何的交互，那么它们发生的顺序是什么并不重要：

```js
var res = {};

function foo(results) {
	res.foo = results;
}

function bar(results) {
	res.bar = results;
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", foo);
ajax("http://some.url.2", bar);
```

👆 `foo()` 和 `bar()` 无论他们的执行顺序是怎么样的，都不会影响到彼此，即并没有发生所谓的 *race condition* 的问题，因此顺序的 *不确定性(nondeterminism)* 是可以被接受的。

### 有交互(Interaction)
不过在大多数实际的情况下，很难做到没有交互，例如对共享作用域下的变量或者 DOM 做出任何的变动，“进程” 发生的顺序都会影响到彼此：

```js
var res = [];

function response(data) {
	res.push( data );
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", response);
ajax("http://some.url.2", response);
```

`res[0]` 和 `res[1]` 到底是哪个请求的结果？！这其实就是不确定的执行顺序带来的 *race condition* 问题。解决这个问题的办法也很简单：

```js
var res = [];

function response(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", response);
ajax("http://some.url.2", response);
```

👆虽然很不优雅，但是确实能够解决实际的问题。

👇下面的更好玩，无论哪个请求先响应，都会让程序挂掉：
```js
var a, b;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(y) {
	b = y * 2;
	baz();
}

function baz() {
	console.log(a + b);
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", foo);
ajax("http://some.url.2", bar);
```

只要一个请求响应，必然会调用(invocation) `baz()`，这时候`a` 和 `b` 中肯定有一个是 `undefined`……

解决的办法有很多，比如用一种叫做 “门槛”(gate) 的方式：

```js
var a, b;

function foo(x) {
	a = x * 2;
	if (a && b) {
		baz();
	}
}

function bar(y) {
	b = y * 2;
	if (a && b) {
		baz();
	}
}

function baz() {
	console.log(a + b);
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", foo);
ajax("http://some.url.2", bar);
```

`if(a && b)` 的条件限制就是所谓的 “门槛” —— 只有当两个请求都返回的时候，才能调用 `baz()`。

另一种解决方式是采用所谓的 “门闩”(latch) —— “先到先得，后到没有” / “赢家通吃”，比如👇 `a == undefined` 这个 “门闩”：

```js
var a;

function foo(x) {
	if (a == undefined) {
		a = x * 2;
		baz();
	}
}

function bar(x) {
	if (a == undefined) {
		a = x / 2;
		baz();
	}
}

function baz() {
	console.log(a);
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", foo);
ajax("http://some.url.2", bar);
```

**Note**：上面的诸多例子中，采用的都是全局变量，但这并不是一种好的方式，除了变量污染之外，还会让你的代码凌乱不堪，而 词法局部作用域 可能是一种更佳的解决方式。

### 协同(Cooperation)
“协同并发”(cooperative concurrency) 是另一种并发的模式。它的聚焦点在于将一段很长的任务，借用事件循环的机制，拆解成多个步骤，避免由于当前任务耗时较长会造成阻塞。

假设有一个Ajax请求会接受到大量的数据，而后会对这些数据进行处理：

```js
var res = [];

function response(data) {
  // 将接受到的数据处理后，加入到一个数据中
	res = res.concat(
    // 处理数据
		data.map(function(val){
			return val * 2;
		})
	);
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", response);
ajax("http://some.url.2", response);
```

👆假设 `data` 是一个多达百万级的数据，想要一次性的处理完这些数据，势必会阻塞掉诸如 UI渲染、用户的各种交互 等功能，而这是难以接受的。

因此，一个做法是将这些数据切片，每次只处理一定量的数据，并借助事件循环的机制处理未完成的内容：

```js
var res = [];

function response(data) {
	// 每次至多只处理1000条数据
	var chunk = data.splice(0, 1000);

	res = res.concat(
		chunk.map( function(val){
			return val * 2;
		} )
	);

	if (data.length > 0) {
		// 利用计时器实现
		setTimeout(function(){
			response(data);
		}, 0);
	}
}

// ajax() 是某个第三方库提供的网络请求的能力
ajax("http://some.url.1", response);
ajax("http://some.url.2", response);
```

除了按照数量来估算单次任务量，也可用按照时间消耗估算，比如利用所谓的 “时间切片” 的概念，以 50ms 为长任务的边界，借助 ES6 的 Generator，实现一个通用的方案：

```js
const longTaskThreshold = 50;
const singleTaskThreshold = longTaskThreshold / 2;

function timeslice (genF) {
    if (!genF || typeof genF !== 'function')  return genF;

    const gen = genF();

    if (typeof gen.next !== 'function') return gen;

    return new Promise((resolve, reject) => {
      function next () {
        try {
          const start = performance?.now() || getTs();
          let res = null;
    
          do {
            res = gen.next();
          } while (!res.done && (performance?.now() || getTs()) - start < singleTaskThreshold);
    
          if (res.done) {
            resolve(res.value);
            return;
          }

          setTimeout(next);
        } catch (err) {
          reject(err);
        }
      }

      next();
    });
  };
```

**Note**：当有多个 `setTimeout(…, 0)` 时，并不能保证它的回调函数会按照预设的顺序，在下一个事件循环中有序的执行。因此，若是在 Node.js 环境中，使用 `process.nextTick(…)` 会是一个相对靠谱的方案。

## 任务(Jobs)