[TOC]

## 一个问题

```javascript
console.log(1);
setTimeout(function() {
  console.log(2);
}, 1000);
new Promise(function(resolve){
    console.log(3);
    resolve();
})
  .then(console.log(4));
  console.log(5);
```

这段代码的正确输出应该是：1 3 5 4 2，为啥呢？

   我们知道JavaScript是单线程的，这对于同步代码还好，但对于异步阻塞代码，如果依然"一行一行执行"，就会阻塞主线程。为了避免主线程被阻塞，使用了Event Loop方案，事件循环。

根据HTML的规范，Event Loop的大概过程是：

1. 执行最旧的task（一次）
2. 检查是否存在microtask，然后不停执行，直到清空队列（多次）
3. 执行render

**task** 和 **microtask** 是两个异步任务的分类，不同的异步任务会被分配到不同的任务队列，然后等待Event Loop将它们压入执行栈执行。

task主要包含：包括整体代码，setTimeout, setInterval,I/O,UI 交互，setImmediate,

microtask主要包含： Promise，process.nextTick(node环境)、MutaionObserver

最基本的事件循环就是：

1. 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。
2. 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
3. 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
4. 上述过程会不断重复。

![](http://ojv7mano6.bkt.clouddn.com/18-2-8/33505129.jpg)

明白了Event Loop的大致流程，就能就是上面的代码了：

1.  整段代码进入主线程，先执行log(1)，输出1，


2. 遇到setTimeout，将其回调函数注册后分发给task Event Quene,
3. 遇到promise，new Promise立即执行，执行log(3),输出3，then函数注册进入microtask Event Quene，
4. 执行log(5)，输出5
5. 至此整体代码作为第一个task执行完毕，然后去看microtask，在microtask Event Quene中有一个then任务，将其压入执行栈，执行log(4),输出4
6. 第一轮事件循环结束，开始第二轮循环，仍然先从task开始，发现task Event Quene有一个setTimeout注册的回调，立即执行log(2)，输出2
7. 最后的结果就是1 3 5 4 2