# NodeJS入门笔记

# NodeJs Learning Notes

## 2021 年 5 月 26 日

## May 26 2021



### 参考

- [深入浅出Node.js](https://book.douban.com/subject/25768396/)



> Web的含义是网，Node的表现就如它的名字一样，是网络中灵活的一个节点。



### 2.2 Node的模块实现

在Node中引入模块，需要经历如下3个步骤。

1. **路径分析**
2. **文件定位**
3. **编译执行**

模块分为两类：一类是Node提供的模块，称为核心模块；另一类是用户编写的模块，称为文件模块。

- **核心模块**部分在Node源代码的编译过程中，编译进了二进制执行文件。在Node进程启动时，部分核心模块就被<u>直接加载进内存中</u>，所以这部分核心模块引入时，文件定位和编译执行这两个步骤可以省略掉，并且在路径分析中优先判断，所以它的加载速度是最快的。
- **文件模块**则是在运行时动态加载，需要完整的路径分析、文件定位、编译执行过程，速度比核心模块慢。

加载顺序方面，

- **优先从缓存加载**，与前端浏览器会缓存静态脚本文件以提高性能一样，Node对引入过的模块都会进行缓存。不同的地方在于，浏览器仅仅缓存文件，而Node缓存的是编译和执行之后的对象。

- 然后从**核心模块**加载，其优先级仅次于缓存加载。
- **文件模块**给Node指明了确切的文件位置，所以在查找过程中可以节约大量时间，其加载速度慢于核心模块。

- **自定义模块**指的是非核心模块，也不是路径形式的标识符。

> 这里的自定义模块通常是npm下载的包，通过`module.paths`可以看到node加载模块的路径。

在文件定位方面，需要做如下事情，

- **文件拓展名分析**：CommonJS模块规范允许在标识符中不包含文件扩展名，这种情况下，Node会按．js、.json、.node的次序补足扩展名，依次尝试。
- **目录分析**：Node会将目录当做一个包来处理，首先在该目录下查找package.json。如果main属性指定的文件名错误，或者压根没有package.json文件，Node会将index当做默认文件名，然后依次查找index.js、index.json、index.node。

如果在目录分析的过程中没有定位成功任何文件，则自定义模块进入下一个模块路径进行查找。如果模块路径数组都被遍历完毕，依然没有查找到目标文件，则会抛出查找失败的异常。

### 2.7 前后端公用模块

浏览器端的JavaScript需要经历从同一个服务器端分发到多个客户端执行，服务器端JavaScript则是相同的代码需要多次执行。**客户端的瓶颈在于带宽，服务器端的瓶颈则在于CPU和内存等资源。**前者需要通过网络加载代码，后者从磁盘中加载，两者的加载速度不在一个数量级上。

### 2.8 模块化总结

除了CommonJS以外，还有AMD（Asynchronous Module Definition）和CMD规范，ES6模块吸收了AMD异步加载的精华，与CJS的同步加载`require`不同，ES6默认是异步加载的，这极大提升了模块的适用性。

### 3. 1 为什么要异步I/O

解决资源分配问题的两种常用模型，

- **单线程同步编程模型**会因阻塞I/O导致硬件资源得不到更优的使用。

- **多线程编程模型**也因为编程中的死锁、状态同步等问题让开发人员头疼。

Node在两者之间给出了它的方案：利用单线程，远离多线程死锁、状态同步等问题；利用异步I/O，让单线程远离阻塞，以更好地使用CPU。

### 3.2.3 现实的异步I/O

现实比理想要骨感一些，但是要达成异步I/O的目标，并非难事。前面我们将场景限定在了单线程的状况下，多线程的方式会是另一番风景。通过让部分线程进行阻塞I/O或者非阻塞I/O加轮询技术来完成数据获取，让一个线程进行计算处理，通过线程之间的通信将I/O得到的数据进行传递，这就轻松实现了异步I/O（尽管它是模拟的）。

![io](https://cescdf.com/image/ioioio.png)

**我们时常提到Node是单线程的，这里的单线程仅仅只是JavaScript执行在单线程中罢了。在Node中，无论是*nix还是Windows平台，内部完成I/O任务的另有线程池。**

### 3.3 Node的异步I/O

在每个Tick的过程中，如何判断是否有事件需要处理呢？这里必须要引入的概念是<u>观察者</u>。每个事件循环中有一个或者多个观察者，而判断是否有事件要处理的过程就是向这些观察者询问是否有要处理的事件。

事件循环是一个典型的<u>生产者/消费者模型</u>。异步I/O、网络请求等则是事件的生产者，源源不断为Node提供不同类型的事件，这些事件被传递到对应的观察者那里，事件循环则从观察者那里取出事件并处理。

一个典型的异步任务`fs.open()`的步骤如下：

**请求对象**

从JavaScript调用Node的核心模块，核心模块调用C++内建模块，内建模块通过libuv进行系统调用，这是Node里经典的调用方式。

这里libuv作为封装层，实质上是调用了`uv_fs_open()`方法。在`uv_fs_open()`的调用过程中，我们创建了一个FSReqWrap请求对象。从JavaScript层传入的参数和当前方法都被封装在这个请求对象中。

以Windows平台为例，产生请求的方法`QueueUserWorkItem()`接受3个参数：第一个参数是将要执行的方法的引用，第二个参数是方法运行时所需要的参数；第三个参数是执行的标志。

至此，JavaScript调用立即返回，由JavaScript层面发起的异步调用的第一阶段就此结束。

请求对象是异步I/O过程中的重要中间产物，所有的状态都保存在这个对象中，包括送入线程池等待执行以及I/O操作完毕后的回调处理。

**执行回调**

如下图，

![ioproce](https://cescdf.com/image/ioproce.png)

> 输入输出完成端口（Input/Output Completion Port，*IOCP*）, 是Window下支持多个同时发生的异步I/O操作的应用程序编程接口。

### 3.3.5 小结

从前面实现异步I/O的过程描述中，我们可以提取出异步I/O的几个关键词：单线程、事件循环、观察者和I/O线程池。这里单线程与I/O线程池之间看起来有些悖论的样子。由于我们知道JavaScript是单线程的，所以按常识很容易理解为它不能充分利用多核CPU。事实上，在Node中，除了JavaScript是单线程外，Node自身其实是多线程的，只是I/O线程使用的CPU较少。另一个需要重视的观点则是，除了用户代码无法并行执行外，所有的I/O（磁盘I/O和网络I/O等）则是可以并行起来的。

### 3.4 非I/O的异步API

尽管我们在介绍Node的时候，多数情况下都会提到异步I/O，但是Node中其实还存在一些与I/O无关的异步API，这一部分也值得略微关注一下，它们分别是`setTimeout()`、`setInterval()`、`setImmediate()`和`process.nextTick()`。

**定时器**

`setTimeout()`和`setInterval()`与浏览器中的API是一致的，它们的实现原理与异步I/O比较类似，只是不需要I/O线程池的参与。调用`setTimeout()`或者`setInterval()`创建的定时器会被插入到定时器观察者内部的一个红黑树中。每次Tick执行时，会从该红黑树中迭代取出定时器对象，检查是否超过定时时间，如果超过，就形成一个事件，它的回调函数将立即执行。

如下图，

![iosettime](https://cescdf.com/image/iosettime.png)

**process.nextTick**

对于`process.nextTick()`方法，与`setTimeout(fn, 0)`类似，然而后者较为浪费性能。实际上，`process.nextTick()`方法的操作相对较为轻量，具体代码如下：

```javascript
process.nextTick = function(callback) {
  if (process._existing) return;
  
  if (tickDepth >= process.maxTickDepth)
    	maxTickWarn();
  
  var tock = {callback: callback};
  if (process.domain) tock.domain = process.domain;
  
  nextTickQueue.push(tock);
  if (nextTickQueue.length) {
    process._needTickCallback();
  }
};
```

每次调用`process.nextTick()`方法，只会将回调函数放入队列中，在下一轮Tick时取出执行。定时器中采用红黑树的操作时间复杂度为`O(log(n))`,` nextTick()`的时间复杂度为`O(1)`。相较之下，`process.nextTick()`更高效。

**setImmediate**

`setImmediate()`方法与`process.nextTick()`方法十分类似，都是将回调函数延迟执行。`process.nextTick()`中的回调函数执行的优先级要高于setImmediate()。这里的原因在于事件循环对观察者的检查是有先后顺序的，process.nextTick()属于idle观察者，setImmediate()属于check观察者。在每一个轮循环检查中，idle观察者先于I/O观察者，I/O观察者先于check观察者。

另外，`setImmediate`方法属于事件循环的宏任务，一次只执行一个，而`process.nextTick()`方法属于微任务，一次执行完所有的，包括本轮添加的。`process.nextTick()`的回调函数保存在一个数组中，`setImmediate()`的结果则是保存在链表中。在行为上，`process.nextTick()`在每轮循环中会将数组中的回调函数全部执行完，而`setImmediate()`在每轮循环中执行链表中的一个回调函数。

### 4.3 异步编程解决方案

**事件发布/订阅模式**

Node自身提供的[events模块](http://nodejs.org/docs/latest/api/events.html)是发布/订阅模式的一个简单实现，

```js
const EventEmitter = require('events');
```

Node中部分模块都继承自它，这个模块比前端浏览器中的大量DOM事件简单，不存在事件冒泡，也不存在`preventDefault()`、`stopPropagation()`和`stopImmediatePropagation()`等控制事件传递的方法。

它具有`addListener/on()`、`once()`、`removeListener()`、`removeAllListeners()`和`emit()`等基本的事件监听模式的方法实现。事件发布/订阅模式的操作极其简单，示例代码如下：

```javascript
// 订阅
emitter.on('event1', message => {
  console.log(message);
})
// 发布
emitter.emit('event1', "I am message!");
```

通过`emit()`发布事件后，消息会立即传递给当前事件的所有侦听器执行。侦听器可以很灵活地添加和删除，使得事件和具体处理逻辑之间可以很轻松地关联和解耦。

**事件发布/订阅模式常常用来解耦业务逻辑，事件发布者无须关注订阅的侦听器如何实现业务逻辑，甚至不用关注有多少个侦听器存在，数据通过消息的方式可以很灵活地传递。**

值得一提的是，Node对事件发布/订阅的机制做了一些额外的处理，这大多是基于健壮性而考虑的。下面为两个具体的细节点。

- 如果对一个事件添加了超过10个侦听器，将会得到一条警告。这一处设计与Node自身单线程运行有关，设计者认为侦听器太多可能导致内存泄漏，所以存在这样一条警告。另一方面，由于事件发布会引起一系列侦听器执行，如果事件相关的侦听器过多，可能存在过多占用CPU的情景。
- 为了处理异常，EventEmitter对象对error事件进行了特殊对待。如果运行期间的错误触发了error事件，EventEmitter会检查是否有对error事件添加过侦听器。如果添加了，这个错误将会交由该侦听器处理，否则这个错误将会作为异常抛出。如果外部没有捕获这个异常，将会引起线程退出。一个健壮的EventEmitter实例应该对error事件做处理。

实现一个**继承EventEmitter的类**是十分简单的，以下代码是Node中Stream对象继承EventEmitter的例子：

```javascript
var event = require('events');

function Stream() {
  events.EventEmitter.call(this);
}
util.inherits(Stream, event.EventEmitter);
```

Node在util模块中封装了继承的方法，所以此处可以很便利地调用。开发者可以通过这样的方式轻松继承EventEmitter类，利用事件机制解决业务问题。在Node提供的核心模块中，有近半数都继承自EventEmitter。

**Promise**

略，详见[JavaScript期约Promise专题](https://cescdf.com/blog/202105/javascript-promise-discuss)。

**流程控制库**

除了事件和Promise外，还有一类方法是需要手工调用才能持续执行后续调用的，我们将此类方法叫做尾触发，常见的关键词是next。事实上，尾触发目前应用最多的地方是Connect的中间件。

略。

### 5.3 高效使用内存

**主动释放全局变量**

如果变量是全局变量（不通过var声明或定义在global变量上），由于全局作用域需要直到进程退出才能释放，此时将导致引用的对象常驻内存（常驻在老生代中）。如果需要释放常驻内存的对象，可以通过delete操作来删除引用关系。或者将变量重新赋值，让旧的对象脱离引用关系。在接下来的老生代内存清除和整理的过程中，会被回收释放。

**闭包**

主动接触闭包引用，否则由于闭包一直会引用中间变量，所以导致无法被自动释放。

### 5.6 大内存应用

Node提供了stream模块用于处理大文件。stream模块是Node的原生模块，直接引用即可。stream继承自EventEmitter，具备基本的自定义事件功能，同时抽象出标准的事件和方法。它分可读和可写两种。Node中的大多数模块都有stream的应用，比如fs的createReadStream()和createWriteStream()方法可以分别用于创建文件的可读流和可写流，process模块中的stdin和stdout则分别是可读流和可写流的示例。

由于V8的内存限制，我们无法通过fs.readFile()和fs.writeFile()直接进行大文件的操作，而改用fs.createReadStream()和fs.createWriteStream()方法通过流的方式实现对大文件的操作。下面的代码展示了如何读取一个文件，然后将数据写入到另一个文件的过程：

```javascript
var reader = fs.createReadStream('in.txt');
var writer = fs.createWriteStream('out.txt');
reader.on('data', chunk => {
  writer.write(chunk);
})
reader.on('end', () => {
  writer.end();
})
```

由于读写模型固定，上述方法有更简洁的方式，具体如下所示：

```javascript
var reader = fs.createReadStream('in.txt');
var writer = fs.createWriteStream('out.txt');
reader.pipe(writer);
```

可读流提供了管道方法pipe()，封装了data事件和写入操作。通过流的方式，上述代码不会受到V8内存限制的影响，有效地提高了程序的健壮性。

如果不需要进行字符串层面的操作，则不需要借助V8来处理，可以尝试进行纯粹的Buffer操作，这不会受到V8堆内存的限制。但是这种大片使用内存的情况依然要小心，即使V8不限制堆内存的大小，物理内存依然有限制。

### 6. 理解Buffer

略。

### 7.4 构建WebSocket服务

WebSocket与传统HTTP有如下好处。

- 客户端与服务器端**只建立一个TCP连接**，可以使用更少的连接。
- WebSocket服务器端可以推送数据到客户端，实现**双向通信**。
- 有更**轻量级的协议头**，减少数据传送量。

Node没有内置WebSocket的库，但是社区的ws模块封装了WebSocket的底层实现。socket.io即是在它的基础上构建实现的。

