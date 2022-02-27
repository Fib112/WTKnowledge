### 一、buffer和字符串

Buffer相当于是一个字节的数组，数组中的每一项对于一个字节的大小：

```javascript
const buffer01 = new Buffer('why') // 将字符串转化到Buffer字节,此用法已废弃
const buffer = Buffer.from('why') // 新版本推荐的用法
```

它是怎么样的过程呢？

![字符串转Buffer](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%AD%97%E7%AC%A6%E4%B8%B2%E8%BD%ACBuffer.png)

如果传入`Buffer.from`的是中文字符串呢？

```javascript
const buffer = Buffer.from('王红元')

console.log(buffer)
```

此时会默认启用`utf-8`编码，大多数中文以3个字节表示，所以打印出来的`buffer`会有9个字节

`Buffer.prototype`有`toString`方法，会将buffer解码，默认解码格式是`utf-8`，如：

```javascript
const buffer = Buffer.from('王红元')

console.log(buffer.toString()) // 打印 王红元
```

如果编码和解码不同，就会打印出乱码：

```javascript
const buffer = Buffer.from('王红元', 'utf16le') // utf16le会将中文编码成两个字节

console.log(buffer.toString('utf8')) // 打印 �s�~CQ
```

### 二、Buffer的其他创建

![Buffer的其他创建](https://gitee.com/Topcvan//img-storage/raw/master//node/Buffer%E7%9A%84%E5%85%B6%E4%BB%96%E5%88%9B%E5%BB%BA.png)

### 三、`Buffer.alloc`和文件读取

#### 3.1 `Buffer.alloc`

`Buffer.alloc`会创建一个定长的Buffer，里面所有的数据默认为00，如：

```javascript
const buffer = Buffer.alloc(8)

console.log(buffer) // <Buffer 00 00 00 00 00 00 00 00>
```

可以对其进行操作，如：

```javascript
buffer[0] = 'w'.charCodeAt()
buffer[1] = 100
buffer[2] = 0x66

console.log(buffer.toString()) // 打印 wdf
```

#### 3.2 文件读取

`fs.readFile`如果不传入编码格式参数，在读取文件时会返回`Buffer`实例，如：

```javascript
const fs = require('fs')

fs.readFile('./foo.txt', (err, data) => {
    console.log(data) // data为Buffer实例 <Buffer 48 65 6c 6c 6f 20 57 6f 72 6c 64>
    console.log(data.toString()) // Hello World
})
```

读取图片：

```javascript
fs.readFile('./img.png', (err, data) => {
    // 由于node不支持图片png、jpg等格式的解码,只支持文本解码,所以图片只能读取为buffer
    console.log(data)
})
```

处理图片可使用第三方库`sharp.js`：

```javascript
const fs = require('fs')

const sharp = require('sharp')

sharp('./test.png').resize(1000, 1000).toBuffer()
  .then(data => {
    fs.writeFileSync('./test_copy.png', data)
})
```

#### 3.3 `Buffer`创建

事实上，创建Buffer实例时，并不会频繁的向操作系统申请内存，它会默认先申请一个8 * 1024个字节大小的内存，也就是`8kb`

### 四、事件循环

#### 4.1 浏览器的事件循环

浏览器的事件循环维护着两个队列：

- 宏任务队列（`macrotaskqueue`）：`ajax`、`setTimeout`、`setInterval`、`DOM监听`、`UI Rendering`等
- 微任务队列（`microtask queue`）：`Promise`的`then`回调、`Mutation Observer API`、`queueMicrotask()`等

那么事件循环对于两个队列的优先级是怎么样的呢？

1. main script中的代码优先执行（编写的顶层script代码）；
2. 在执行任何一个宏任务之前（不是队列，是一个宏任务），都会先查看微任务队列中是否有任务需要执行
   - 也就是宏任务执行之前，必须保证微任务队列是空的；
   - 如果不为空，那么久优先执行微任务队列中的任务（回调）；

#### 4.2 Node架构分析

浏览器中的EventLoop是根据HTML5定义的规范来实现的，不同的浏览器可能会有不同的实现，而Node中是由
libuv实现的。

Node架构图：

![node架构](https://gitee.com/Topcvan//img-storage/raw/master//node/node%E6%9E%B6%E6%9E%84.png)

我们会发现libuv中主要维护了一个`EventLoop`和`worker threads`（线程池）；`EventLoop`负责调用系统的一些其他操作：文件的IO、Network、child-processes等

`libuv`是一个多平台的专注于异步IO的库，它最初是为Node开发的，但是现在也被使用到`Luvit`、`Julia`、`pyuv`等其他地方；

#### 4.3 阻塞IO和非阻塞IO

如果希望在程序中对一个文件进行操作，那么就需要打开这个文件：通过文件描述符。

- 我们思考：JavaScript可以直接对一个文件进行操作吗？
- 看起来是可以的，但是事实上任何程序中的文件操作都是需要进行系统调用（操作系统的文件系统）；
- 事实上对文件的操作，是一个操作系统的IO操作（输入、输出）；

操作系统提供了阻塞式调用和非阻塞式调用：

- 阻塞式调用：调用结果返回之前，当前线程处于阻塞态（阻塞态CPU是不会分配时间片的），调用线程只有在得到调用结果之后才会继续执行。
- 非阻塞式调用：调用执行之后，当前线程不会停止执行，只需要过一段时间来检查一下有没有结果返回即可。

所以开发中的很多耗时操作，都可以基于这样的非阻塞式调用：

- 比如网络请求本身使用了Socket通信，而Socket本身提供了select模型，可以进行非阻塞方式的工作；
- 比如文件读写的IO操作，可以使用操作系统提供的基于事件的回调机制；

**非阻塞IO的问题**
但是非阻塞IO也会存在一定的问题：我们并没有获取到需要读取（我们以读取为例）的结果

- 那么就意味着为了可以知道是否读取到了完整的数据，需要频繁的去确定读取到的数据是否是完整的；
- 这个过程称之为轮询操作；

那么这个轮询的工作由谁来完成呢？

- 如果主线程频繁的去进行轮训的工作，那么必然会大大降低性能；
- 并且开发中可能不只是一个文件的读写，可能是多个文件；
- 而且可能是多个功能：网络的IO、数据库的IO、子进程调用；

libuv提供了一个线程池（Thread Pool）：

- 线程池会负责所有相关的操作，并且会通过轮询等方式等待结果；
- 当获取到结果时，就可以将对应的回调放到事件循环（某一个事件队列）中；
- 事件循环就可以负责接管后续的回调工作，告知JavaScript应用程序执行对应的回调函数；

**阻塞和非阻塞，同步和异步的区别**
阻塞和非阻塞是对于被调用者来说的；

- 在这里就是系统调用，操作系统提供了阻塞调用和非阻塞调用；

同步和异步是对于调用者来说的；

- 在这里就是自己的程序；
- 如果在发起调用之后，不会进行其他任何的操作，只是等待结果，这个过程就称之为同步调用；
- 如果在发起调用之后，并不会等待结果，继续完成其他的工作，等到有回调时再去执行，这个过程就是异步调用；

`Libuv`采用的就是非阻塞异步IO的调用方式；

#### 4.4 node事件循环的阶段

事件循环像是一个桥梁，是连接着应用程序的JavaScript和系统调用之间的通道：

- 无论是文件IO、数据库、网络IO、定时器、子进程，在完成对应的操作后，都会将对应的结果和回调函数放到事件循环（任务队列）中；
- 事件循环会不断的从任务队列中取出对应的事件（回调函数）来执行；

但是一次完整的事件循环Tick分成很多个阶段：

1. 定时器（Timers）：本阶段执行已经被`setTimeout() `和`setInterval() `的调度回调函数。
2. 待定回调（Pending Callback）：对某些系统操作（如TCP错误类型）执行回调，比如TCP连接时接收到
   `ECONNREFUSED`。
3. idle, prepare：仅系统内部使用。
4. 轮询（Poll）：检索新的 I/O 事件;执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，那些由计时器和 `setImmediate()` 调度的之外），其余情况 node 将在适当的时候在此阻塞。
5. 检测：`setImmediate() `回调函数在这里执行。
6. 关闭的回调函数：一些关闭的回调函数，如：`socket.on('close', ...)`。

图解如下：

![事件循环阶段图解](https://gitee.com/Topcvan//img-storage/raw/master//node/%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%E9%98%B6%E6%AE%B5%E5%9B%BE%E8%A7%A3.png)

**Node的微任务和宏任务**

从一次事件循环的Tick来说，Node的事件循环更复杂，它也分为微任务和宏任务：

- 宏任务（`macrotask`）：`setTimeout`、`setInterval`、`IO`事件、`setImmediate`、`close`事件；
- 微任务（`microtask`）：`Promise`的`then`回调、`process.nextTick`、`queueMicrotask`；

但是，Node中的事件循环不只是微任务队列和宏任务队列：

- 微任务队列：
  - next tick queue：process.nextTick；
  - other queue：Promise的then回调、queueMicrotask；
- 宏任务队列：
  - timer queue：setTimeout、setInterval；
  - poll queue：IO事件；
  - check queue：setImmediate；
  - close queue：close事件；

#### 4.5 setTimeout与setImmediate

setTimeout(回调函数, 0)、setImmediate(回调函数)执行顺序分析

- 情况一：setTimeout、setImmediate
- 情况二：setImmediate、setTimeout

为什么呢？

在Node源码的`deps/uv/src/timer.c`中141行，有一个 `uv__next_timeout`的函数；这个函数决定了，`poll`阶段要不要阻塞在这里；阻塞在这里的目的是当有异步IO被处理时，尽可能快的让代码被执行；

情况一：如果事件循环开启的时间(ms)是小于`setTimeout`函数的执行时间的(不是回调事件，而是`setTimeout`函数执行所花费的时间)；

- 也就意味着先开启了event-loop，但是这个时候执行到timer阶段，并没有定时器的回调被放到入 timer queue中；所以没有被执行，后续开启定时器和检测到有`setImmediate`时，就会跳过poll阶段，向后继续执行；这个时候是先检测 `setImmediate`队列执行回调，第二次的tick中执行了timer中的`setTimeout`回调；
- 情况二：如果事件循环开启的时间(ms)是大于`setTimeout`函数的执行时间的；这就意味着在第一次tick中，已经准备好了`timer queue`；所以会直接按照顺序执行即可；