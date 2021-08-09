# 程序的性能(Program Performance)
前面关于JS中异步的长篇大论，都是围绕在如何让它变得更为高效进行的。但为何异步在JS的程序中如此重要，以至于要花费一整卷的内容来讲述呢 —— 因为它和程序的性能好坏，直接联系紧密。

比如你的程序要执行两个 Ajax 网络请求 —— 你可以让第一个请求完成后再执行第二个请求，而后完成任务；同样你也能两个请求一起执行，设置一个门槛保证在两个任务都完成后结束任务。很显然，第二种方式的性能更好，也能带来更好的用户体验(user experience)。

👆即便是这两种方式最终耗费的时间居然相差无几，而用户对性能的感知哪怕只提升一点点，也同样和实际的测量的性能数据的提升一样重要。

**Note**：至于 `a++` 还是 `++a` 更快，将会在下一章中有详细的解读。

## Web Workers
如果你的程序需要执行一系列的CPU密集任务，而你又不希望它们会阻塞主线程的UI渲染，JS单线程的特性并不会成为最终的阻碍。

想象一下你的程序被分为两个部分，其中一块跑在主线程中，而另一个运行在一个完全独立的线程中。这样的架构会带来啥呢？

首先，这个独立的线程的工作是否会阻塞主线程的运行，如果是的话，那这一切并没有任何意义 —— 直接用异步的方式显然更方便。

接着，要看这两个分开的线程，能否共享作用域和资源。如果是的话，就要和其他多线程语言(Java、C++)一样，得处理 **合作锁(cooperative locking)** 或者 **抢占锁(preemptive locking)**，背后有更多额外的工作，绝非轻松之事。

那如果不共享资源，程序之间要如何进行交互，这是不得不考虑的问题。

对于 JS 而言，在 HTML5 中新增的 "Web Workers" 就是这些问题的答案 —— 但这是基于你 JS 代码运行的宿主环境，即浏览器决定的，JS 本身并没有提供多线程运行的能力。

一般来讲，在你JS程序的主线程中初始化一个 Worker 很简单：

```js
var w1 = new Worker("http://some.url.com/myworker.js");
```

上面的 URL 指向的是你提供的JS文件，而后浏览器会自动帮你开一个线程单独的执行这个文件。

**Note**：这种提供一个外部文件 URL 所实例化的 Worker 被称为 **专用Worker(Dedicated Worker)*；若是将 URL 替换为一个 Blob URL，那它就是所谓的 *内联Worker(Inline Worker)*。

Workers 之间以及和主线程，都不会共享作用域或资源，它们只靠基本的事件机制来进行通信连接 —— 这把多线程编程中要处理的噩梦都简化了。

👆 `w1` 是一个事件监听和触发的对象，它能够订阅 Workers 发出的事件，也能够触发事件通知 Workers。

就比如在主线程中：

```js
// 监听 message 事件
w1.addEventListener('message', function (evt) {
  console.info(evt.data);
});

// 发送 message 事件
w1.postMessage('someting from main thread!');
```

相应地在 Worker 中：

```js
addEventListener('message', function (evt) {
  console.info(evt.data);
});

postMessage('someting from worker!');
```

在 专用Worker 模式中，主线程和 Worker 的关系都是一一对应的，他们的之间接收的事件只能来自对方，发送的事件也只能被对方接收。

和主线程一样，Worker 同样也能实例化另一个 Worker，即一个 subworker。

关闭 Worker 的方法也很简单，只需要调用 `w1` 对象上的 `terminate()` 方法就行。但是这样的关闭就好像直接关闭浏览器的 tab 栏一样，Worker 线程是来不及做任何清理工作的。

即便一个浏览器中有多个 tab，它们都用同样的 URL 初始化了 Worker，这些 Worker 也是独立运行互不干扰的。

**Note**：无论你有意还是无意地创建了几百上千个 Worker，这都不会造成DOS(denial-of-service)攻击的核心原因在于说，系统可以自由地决定实际使用的 线程/CPU核数 来运行这些 Worker —— 虽然目前没有任何方式能够保证 Worker 最终使用的实际资源数量，但至少不会在主线程中运行。

### Worker Environment
Worker 线程是一个完全独立的线程，即意味着说你完全没办法访问主线程中的任何资源 —— 无论是全局变量亦或DOM元素。当然，一些基本的诸如网络的 Ajax、Fetch、WebSockets 操作，以及定时器不受影响。而且，一些重要的全局对象，比如 `navigator`、`location`、`JSON` 等都有一份拷贝可供使用。

值得一提的是，`importScripts(…)` 这个在 Worker 中的全局 API，能帮助你加载额外的 JS 资源：

```js
// Worker 中
importScripts('a.js', 'b.js');
```

不过，`importScripts(…)` 的加载过程是同步执行的，即意味着它会阻塞后续代码的执行，直到所有的资源都加载完毕。

Web Worker 常见的用途包括但不限于以下这些：

- 处理密集型的计算

- 为较大的数据组做排序

- 数据相关的操作(压缩、音频分析、图片像素操作……)

- 大流量的网络通讯 

### 数据传输(Data Transfer)
你可能注意到在 主线程 和 Worker 之间会有大量的，通过事件机制来进行的信息传递。在早期的时间，唯一传递的数据类型只有字符串 —— 这不仅在序列化和反序列化的过程中，会带来性能上的损失；而且还会消耗掉双倍的内存，以及随之而来的垃圾回收的开销。

所幸的是，现在有了更好的选项 —— [“结构化拷贝算法(Structured Cloning Algorithm)”](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) 可以用来拷贝复杂对象等数据结构，即便是存在循环引用也不足为惧。不过依然要消耗双倍的内存，并且只有在 IE10 以上的现代浏览器才支持。

更牛的另一个方案是使用 [“可转移对象(Transferable Object)”](https://developers.google.com/web/updates/2011/12/Transferable-Objects-Lightning-Fast)，它能将对象的 “所有权(ownership)” 转移而不改其包含的变数据。譬如一旦将对象所有权转移给 Worker 后，主线程就无法通过内存地址访问到原来的数据了，反过来也是一样。这样一来，就消除了线程之间共享作用域的危害。

比如，类型数组 `Uint8Array` 就一个 “可转移(Transferables)” 的对象，而它配合 `postMessage(…)` 就能完成所有权转移的操作：

```js
const uint8 = new Uint8Array([1, 2]);
postMessage(uint8.buffer, [...uint8]);
```

👆🏻 第一个参数是一个原始的buffer，第二个参数则是一个需要传递的数据列表。

如果当浏览器不支持 *Transferable Object* 时，可以降级到 *Structured Cloning*。

### 共享Workers(Shared Workers)
如果你的网站会在同一个页面中打开多个标签页(tab)，而它们都使用了 Web Worker，此时你一定会考虑如何才能避免创建重复的 Worker，以减少资源的开销。无论是对于 socket 网络连接的个数限制，亦或同一个域名的并发个数连接限制，都是处于相同的道理。而这样的限制也有助于减少后端服务器的压力。

一个能被所有页面共享的中心化的 Worker 恰好能适用于这样的场景：

```js
var w1 = new SharedWorker('http://some.url.com/center-worker.js');
```

显然一个共享的 Worker 必须要知道接收的消息是谁发送的。好在 Worker 中的 `port` 对象，已经提供了这套机制来满足唯一身份识别的需求 —— 这有点像 socket 网络中的端口号一样：

```js
w1.port.addEventListener('message', handleMessage);

w1.port.postMessage('some messages!');
```

port 的连接必须有个初始化的过程：

```js
w1.port.start();
```

**Note**：`SharedWorker` 目只有 Chrome、Firefox、Opera 等几个浏览器支持。

而在共享的 Worker 内部，还需要处理额外的 `"connect"` 事件。用这个事件提供的 `port` 对象，配合闭包的机制，就能够完成事件的监听和发送：

```js
addEventListener('connect', function (evt) {
  const port = evt.ports[0];
  port.addEventListener('message', function () {
    port.postMessage('some msg!');
  });

  port.start();
});
```

除此之外，共享的和专有的 Worker 在别的操作上都一模一样。

**Note**：共享的 Worker 只要有一个 port 连接还继续存在，那它就会一直存在。而专有的 Worker 会在连接它的程序终止后关闭。

### Web Workers 的兼容(Polyfilling Web Workers)
Web Workers 为 JS 的程序提供了真正意义上的并行能力，在某些场景下对于程序性能的提升十分明显。但是在老旧的浏览器中，只能通过扩展 API 的方式来兼容这个特性，而且也没办法做到真正意义上的并行。

比如通过 Iframes 来模拟并行的环境，或者是通过事件循环的异步机制(`setTimeout(…)`)来模拟。但其实在现代的浏览器中，Iframe 都是运行在同一个线程中，而采用异步机制模拟的 “时间切片” 虽然相对看起来稍微好一些，但有些 API 比如 `importScripts(…)` 就没办法做到同步阻塞的特性。

## SIMD