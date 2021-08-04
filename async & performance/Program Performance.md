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