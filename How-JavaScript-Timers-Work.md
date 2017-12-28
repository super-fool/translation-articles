
# How JavaScript Timers Work

![原文地址](https://johnresig.com/blog/how-javascript-timers-work/)

在学习JavaScript的基础水平上, 清楚JavaScript的时间器如何工作是很重要的. 由于它们(时间器)都是处在各自的单线程中二导致行为不一致.我们来看三个函数,它们每一个函数都可以进行构建和操作时间器.

- `var id = setTimeout(fn, delay)`, 初始化一个构造器,它会在`delay`秒后去调用`fn`函数. 并且返回一个唯一ID,使用这个ID可以取消这个时间器.
- `var id =setInterval(fn,delay)`, 和`setTimeout`相似,但是会持续调用`fn`(每次调用都是相隔`delay`秒),直到被取消.
- `clearInterval(id); clearTimeout(id)`, 接受一个时间器的ID(上述任意一个函数的返回ID),然后从事件(occurring n.)中停止时间器的回调.

为了了解时间器如何在内部工作,其中有一个重要的概念需要去探究(explored):<b>时间器的`delay`是不能保证的</b>. 这是因为在浏览器中所有的JavaScript只有在单线程中开放执行的异步事件(例如点击事件,时间器)运行时才会执行. 
<i>Since All JavaScript in a browser executes on a single thread asynchronous event only run when there's been opening execution.</i> -- <b>比较难翻译,所以贴出来</b>
向下面的这站图表就是最好的展示(或者叫论证):
![](https://johnresig.com/files/Timers.png)
[查看原图](https://johnresig.com/files/Timers.png)

在这个图表(figure)中有许多信息需要消化(digest).理解它可以让你更好的明白JavaScript如何异步的执行工作. 这个图表是一个有维度的图表:我们有一个垂直的时间(wall clock=正常时间),它以毫秒为单位.蓝色区域代表JavaScript正在执行的部分.例如第一个JavaScript的蓝色块大约执行了18毫秒,鼠标点击事件大约执行了11毫秒,等等.

由于JavaScript在一个时间内只能执行一部分代码(基于单线程机制),每一个块代码都会锁住其他异步事件的进度.这就意味着当一个异步事件发生时就会加入事件队列中进行排队稍后执行（由于线程模型是不能这样做的），而实际上，延时调用程序将会被插入队列，等待可调用时序时，被顺序执行。

其次，我们在第一个代码块中，我们触发了一次点击操作。这个异步事件相关的回调函数，和定时器一样，也不会立即被执行，同样进入队列等待执行。

当第一个Javascript代码块执行完成后，浏览器就会去问队列：接下来要执行什么？然而此时此刻，鼠标事件的句柄函数和定时器的延时调用函数都在等待。浏览器会在二者中选择一个（鼠标事件）立即执行。定时器的回调会等待下个时机，被按顺序调用。

注意图中，在鼠标事件的回掉执行时，interval延时回掉被执行了。但是需要注意的时，当interval再次被出发时（当一个定时器的延时处理在执行的时候），这时候程序的处理将会被丢弃。假设当有大块的代码正在执行时，你又有一堆的interval延时调用在排队，你希望结果很可能就是这个大块的js代码执行完毕后，interval的延时调用会一个接一个的被触发，而且在执行时没有延时时间，也就是会被连续的调用。可是相反，浏览器往往只是等待，直到没有更多的interval处理程序进行排队。

事实上，我们也可以看到，第三个interval回掉触发的时候，这个interval本身也在执行中。这就像我们展示了一个很重要的现象就是：interval 并不在乎当前谁正在执行，他们不分青红皂白地将排队，即使这意味着回调之间的时间将被牺牲。

最后，当第二个interval回掉执行完成后，我们能看到，对于js引擎来说，没有需要去执行的东东了。这就意味着，浏览器在等待新的异步事件发生了。到第50秒时，这个interval被再次触发，这时候没有东西在阻塞执行，因此他会被立即调用。

我们来用几行代码来更好的去分辨setInterval和setTimeout之间的区别:
```
setTimeout(function(){
    /* Some long block of code... */ 
    setTimeout(arguments.callee, 10); 
}, 10);

setInterval(function(){
    /* Some long block of code... */ 
}, 10);
```
这两段代码乍一看似乎差不多，但事实上相差很多。有一点值得注意的是，在这里面的setTimeout，两个回掉执行的时间间隔至少会是10毫秒；而setInterval将尝试每10秒去执行一次，不去考虑上一次回掉是否已经完成。

There’s a lot that we’ve learned here, let’s recap:

- Javascript是一个单线程执行的东东，迫使异步事件排队等待执行。
- setTimeout 与 setInterval执行代码的原理是完全不同的。
- 当一个定时器执行被阻塞时，他会等待下一个可能执行的时机去执行，所以这个延时可能会比预先设定的时间要长。
- 如果回调函数执行时间过长（长于定时器的延迟时间），“间隔定时器”有可能会一个接一个无间隔的执行

补充的例子
```
var die = false; 
setTimeout(function() { 
    die = true; 
}, 100);

while(!die) { }

console.log('done');
```
如果你认为在100毫秒后，会打针done，说明你没有看懂此篇文章。你一定会觉得在100毫秒后，die的值变成true，然后console会被执行，如果你这样想那你就错了。记住setTimeout的准则是尽快执行，而不是立即执行。只有当主事件循环结束是，有时间片供setTimeout去执行时，定时器才会被执行。