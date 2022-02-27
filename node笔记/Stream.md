### 一、文件读写的Stream

#### 1.1 认识Stream

什么是流呢？我们的第一反应应该是流水，源源不断的流动；

程序中的流也是类似的含义，我们可以想象当我们从一个文件中读取数据时，文件的二进制（字节）数据会源源不
断的被读取到程序中；而这个一连串的字节，就是我们程序中的流；

所以，可以这样理解流：是连续字节的一种表现形式和抽象概念；流应该是可读的，也是可写的；

在之前学习文件的读写时，可以直接通过`readFile`或者`writeFile`方式读写文件，为什么还需要流呢？

直接读写文件的方式，虽然简单，但是无法控制一些细节的操作；比如从什么位置开始读、读到什么位置、一次性读取多少个字节；读到某个位置后，暂停读取，某个时刻恢复读取等等；或者这个文件非常大，比如一个视频文件，一次性全部读取并不合适；

#### 1.2 文件读写的Stream

事实上Node中很多对象是基于流实现的：

- `http`模块的Request和Response对象；
- `process.stdout`对象；

官方：所有的流都是`EventEmitter`的实例：

`Node.js`中有四种基本流类型：

- Writable：可以向其写入数据的流（例如`fs.createWriteStream()`）
- Readable：可以从中读取数据的流（例如`fs.createReadStream()`）
- Duplex：同时为Readable和Writable的流（例如`net.Socket`）
- `Transform`：`Duplex`可以在写入和读取数据时修改或转换数据的流（例如`zlib.createDeflate()`）

### 二、Readable

之前读取一个文件的信息是通过`readFile`：

```javascript
fs.readFile('./foo.txt', (err, data) => {
    console.log(data)
})
```

这种方式是一次性将一个文件中所有的内容都读取到程序（内存）中，但是这种读取方式就会出现之前提到的很多问题：

- 文件过大、读取的位置、结束的位置、一次读取的大小；
- 这个时候，可以使用`createReadStream`，我们来看几个参数，更多参数可以参考官网：
  - `start`：文件读取开始的位置；
  - `end`：文件读取结束的位置；
  - `highWaterMark`：一次性读取字节的长度，默认是`64kb`；

#### 2.1 `Readable`

创建文件Readable：

```javascript
const read = fs.createReadStream('./foo.txt', {
    start: 3,
    end: 8,
    highWaterMark: 4
})
```

可以通过监听data事件，获取读取到的数据：

```javascript
read.on('data', (data) => {
    console.log(data)
})
```

也可以做一些其他的操作：监听其他事件、暂停或者恢复

```javascript
read.on('data', data => {
    console.log(data)
    read.pause() // 暂停读取
    setTimeout(() => {
        read.resume() // 1s后恢复读取
    }, 1000)
})

read.on('open', () => {
    console.log('file open')
})
read.on('end', () => {
    console.log('read end')
})
read.on('close', () => {
    console.log('file close')
})
```

#### 2.2 `Writable`

之前写入一个文件的方式是这样的：

```javascript
fs.writeFile('./test.txt', 'test', (err) => {})
```

这种方式相当于一次性将所有的内容写入到文件中，但是这种方式也有很多问题：

- 比如希望一点点写入内容，精确每次写入的位置等；

这个时候，可以使用`createWriteStream`，我们来看几个参数，更多参数可以参考官网：

- flags：默认是w，如果希望是追加写入，可以使用a或者a+；
- start：写入的位置；

**Writable**使用

```javascript
const writer = fs.createWriteStream('./test', {
    flags: 'a+',
    starts: 2 // 写入的文字会覆盖原来位置上的文字
})
```

Writable写入文件：

```javascript
writer.write('测试test', err => {
    console.log(err || '文件写入成功')
})
```

监听open与close事件：

```javascript
writer.on('open', () => {
    console.log('文件打开')
})
writer.on('close', () => {
    console.log('文件关闭')
})
```

但是会发现，close 事件的回调函数不会执行：

- 这是因为写入流在打开后是不会自动关闭的；
- 必须手动关闭，来告诉Node已经写入结束了；并且会发出一个finish 事件；

另外一个非常常用的方法是end：

```javascript
writer.end('Hello World') // Hello World追加到文档末尾
```

end方法相当于做了两步操作：write传入的数据和调用close方法；

#### 2.3 pipe方法

正常情况下可以将读取到的输入流，手动的放到输出流中进行写入：

```javascript
const reader = fs.createReadStream('./test.txt')
const writer = fs.createWriteStream('./foo.txt')

reader.on('data', (data) => {
    console.log(data)
    writer.write(data, err => {
        console.log(err)
    })
})
```

也可以通过pipe来完成这样的操作：

```
reader.pipe(writer)
```

