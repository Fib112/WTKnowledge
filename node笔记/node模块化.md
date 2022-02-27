### 一、Node传参

正常情况下执行一个node程序，直接跟上对应的文件即可。但是，在某些情况下执行node程序的过程中，可能希望给node传递一些参数：

```javascript
node index.js development
```

如果这样来使用程序，就意味着需要在程序中获取到传递的参数：

- 获取参数其实是在process的内置对象中的；

- 如果直接打印这个内置对象，它里面包含特别的信息

- 其他的一些信息，比如版本、操作系统等大家可以自行查看

- 现在，先找到其中的argv属性：

  - 发现它是一个数组，里面包含了我们需要的参数：

    ![node参数](https://gitee.com/Topcvan//img-storage/raw/master//node/node%E5%8F%82%E6%95%B0.png)

  - `argv`数组中的前两项是固定的：第一项是node所在目录，第二项是js文件所在目录，剩下的就是用户在命令行传入的参数

为什么叫argv呢？在C/C++程序中的main函数中，实际上可以获取到两个参数：

- argc：argument counter的缩写，传递参数的个数；

- argv：argument vector的缩写，传入的具体参数。
  - vector翻译过来是矢量的意思，在程序中表示的是一种数据结构。
  - 在C++、Java中都有这种数据结构，是一种数组结构；
  - 在JavaScript中也是一个数组，里面存储一些参数信息；
  
- 我们可以在代码中，将这些参数信息遍历出来，使用：

  ```javascript
  process.argv.forEach(item => {
      console.log(item)
  })
  ```

### 二、全局对象

Node中给我们提供了一些全局对象，方便我们进行一些操作：

#### 2.1 特殊的全局对象

这些全局对象可以在模块中任意使用，但是在命令行交互中是不可以使用的；

包括：`__dirname、__filename、exports、module、require()`
`__dirname`：获取当前文件所在的路径：
注意：不包括后面的文件名
`__filename`：获取当前文件所在的路径和文件名称：
注意：包括后面的文件名称

![dirname&filename](https://gitee.com/Topcvan//img-storage/raw/master//node/dirname&filename.png)

#### 2.2 常见的全局对象

- process对象：process提供了Node进程中相关的信息：比如Node的运行环境、参数信息等；

- console对象：提供了简单的调试控制台。

- 定时器函数：在Node中使用定时器有好几种方式：

  - setTimeout(callback, delay[, ...args])：callback在delay毫秒后执行一次；

  - setInterval(callback, delay[, ...args])：callback每delay毫秒重复执行一次；
  - setImmediate(callback[, ...args])：callback I/O 事件后的回调的“立即”执行
  - process.nextTick(callback[, ...args])：添加到下一次tick队列中；

#### 2.3 global对象

global是一个全局对象，事实上前面提到的process、console、setTimeout等都有被放到global中。

**global和window的区别**
在浏览器中，全局变量都是在window上的，比如有document、setInterval、setTimeout、alert、console等等在Node中也有一个global属性，并且看起来它里面有很多其他对象。

但是在浏览器中执行的JavaScript代码，如果在顶级范围内通过var定义的一个属性，默认会被添加到window对象上。但是在node中，通过var定义一个变量，它只是在当前模块中有一个变量，不会放到全局中

### 三、模块化

#### 3.1 CommonJs

**exports导出**

exports是一个对象，可以在这个对象中添加很多个属性，添加的属性会导出：

```javascript
exports.name = name
exports.age = age
exports.sayHello = sayHello
```

另外一个文件中可以导入：

```javascript
const bar = require('./bar')
```

上面这行完成了什么操作呢？理解下面这句话，Node中的模块化一目了然

- 意味着main中的bar变量等于exports对象；
- 也就是require通过各种查找方式，最终找到了exports这个对象；
- 并且将这个exports对象赋值给了bar变量，bar变量就是exports对象了；

bar和exports是同一个对象：

在下面的代码中，`bar.js`中的改变会影响到`main.js`

```javascript
// bar.js
const name = 'coderwhy'
const age = 18

const sayHello = () => {
    console.log('Hello ' + name)
}

exports.name = name
exports.age = age
exports.sayHello = sayHello

setTimeout(() => exports.name = 'lilei', 1000)
```

```javascript
// main.js
const bar = require('./bar')

const name = bar.name
const age = bar.age
const sayHello = bar.sayHello

console.log(name) // coderwhy
console.log(age) // 18

sayHello() // Hello coderwhy

setTimeout(() => console.log(bar.name), 2000) // lilei
```

**module.exports**

CommonJS中是没有module.exports的概念的。但是为了实现模块的导出，Node中使用的是Module的类，每一个模块都是Module的一个实例，也就是module；所以在Node中真正用于导出的其实根本不是exports，而是module.exports；因为module才是导出的真正实现者；

但是，为什么exports也可以导出呢？这是因为node内部自动将exports对象赋值给了module对象的exports属性；也就是说module.exports = exports = main中的bar；

如果在js代码中手动将新的对象赋值给module.exports，那么导出的对象将是新对象：

```javascript
const name = 'coderwhy'

module.exports = {
    name
}
```

**exports赋值时机**

```javascript
// bar.js
// 如果export是在文件执行前赋值给module.exports，那么导出的应该是空对象
export = 123
// 如果是在执行后赋值的，那么应该是数字
```

```javascript
// main.js
console.log(require('./bar')) // {}
```

可见module.exports是在文件执行前赋值的

**require查找规则**

require是一个函数，可以引入一个文件（模块）中导出的对象。导入格式如下：require(X)

1. 情况一：X是一个核心模块，比如path、http，则直接返回核心模块，并且停止查找

2. 情况二：X是以./ 或../ 或/（根目录）开头的

   第一步：将X当做一个文件在对应的目录下查找；
   1.如果有后缀名，按照后缀名的格式查找对应的文件
   2.如果没有后缀名，会按照如下顺序：

   - 直接查找文件X
   - 查找X.js文件
   - 查找X.json文件
   - 查找X.node文件

   第二步：没有找到对应的文件，将X作为一个目录
   - 查找目录下面的index文件
   - 查找X/index.js文件
   - 查找X/index.json文件
   - 查找X/index.node文件

   如果没有找到，那么报错：not found
   
3. 情况三：直接是一个X（没有路径），并且X不是一个核心模块

   如：在/Users/coderwhy/Desktop/Node/TestCode/04_learn_node/05_javascript-module/02_commonjs/main.js中编写`require('abc')`，那么就会按照以下路径逐个查找：

   ```javascript
   paths: [
       '/Users/coderwhy/Desktop/Node/TestCode/04_learn_node/05_javascript-module/02_commonjs/node_modules',
       '/Users/coderwhy/Desktop/Node/TestCode/04_learn_node/05_javascript-module/node_modules',
       '/Users/coderwhy/Desktop/Node/TestCode/04_learn_node/node_modules',
       '/Users/coderwhy/Desktop/Node/TestCode/node_modules',
       '/Users/coderwhy/Desktop/Node/node_modules',
       '/Users/coderwhy/Desktop/node_modules',
       '/Users/coderwhy/node_modules',
       '/Users/node_modules',
       '/node_modules',
   ]
   ```

   如果上面的路径中都没有找到，那么报错：not found

   **模块加载过程**

   - 模块在被第一次引入时，模块中的js代码会被运行一次

   - 模块被多次引入时，会缓存，最终只加载（运行）一次
     为什么只会加载运行一次呢？
     这是因为每个模块对象module都有一个属性：loaded。
     loaded为false表示还没有加载，为true表示已经加载；

   - 如果有循环引入，那么加载顺序是什么？

     ![模块循环引用](https://gitee.com/Topcvan//img-storage/raw/master//node/%E6%A8%A1%E5%9D%97%E5%BE%AA%E7%8E%AF%E5%BC%95%E7%94%A8.png)

     如果出现上图模块的引用关系，那么加载顺序是什么呢？
     这个其实是一种数据结构：图结构；
     图结构在遍历的过程中，有深度优先搜索（DFS, depth first search）和广度优先搜索（BFS, breadth first search）；Node采用的是深度优先算法：main -> aaa -> ccc -> ddd -> eee -> bbb

**CommonJs规范缺点**

CommonJS加载模块是同步的：

   - 同步的意味着只有等到对应的模块加载完毕，当前模块中的内容才能被运行；
   - 这个在服务器不会有什么问题，因为服务器加载的js文件都是本地文件，加载速度非常快；

如果将它应用于浏览器呢？

   - 浏览器加载js文件需要先从服务器将文件下载下来，之后在加载运行；

   - 那么采用同步的就意味着后续的js代码都无法正常运行，即使是一些简单的DOM操作；

所以在浏览器中，我们通常不使用CommonJS规范：

- 当然在webpack中使用CommonJS是另外一回事；
- 因为它会将我们的代码转成浏览器可以直接执行的代码

#### 3.2 ES Module与CommonJs区别

**CommonJS的加载过程**

CommonJS模块加载js文件的过程是运行时加载的，并且是同步的：

- 运行时加载意味着是js引擎在执行js代码的过程中加载 模块；
- 同步的就意味着一个文件没有加载结束之前，后面的代码都不会执行；

CommonJS通过module.exports导出的是一个对象：
- 导出的是一个对象意味着可以将这个对象的引用在其他模块中赋值给其他变量；
- 但是最终他们指向的都是同一个对象，那么一个变量修改了对象的属性，所有的地方都会被修改；

**ES Module加载过程**

ES Module加载js文件的过程是编译（解析）时加载的，并且是异步的：

- 编译时（解析）时加载，意味着import不能和运行时相关的内容放在一起使用：
- 比如from后面的路径需要动态获取；
- 比如不能将import放到if等语句的代码块中；
- 所以有时候也称ES Module是静态解析的，而不是动态或者运行时解析的；

异步的意味着：JS引擎在遇到import时会去获取这个js文件，但是这个获取的过程是异步的，并不会阻塞主线程继续执行；
- 也就是说设置了type=module 的代码，相当于在script标签上也加上了async 属性；
- 如果后面有普通的script标签以及对应的代码，那么ES Module对应的js文件和代码不会阻塞它们的执行；

ES Module通过export导出的是变量本身的引用：

- export在导出一个变量时，js引擎会解析这个语法，并且创建模块环境记录（module environment record）；
- 模块环境记录会和变量进行绑定（binding），并且这个绑定是实时的；
- 而在导入的地方，我们是可以实时的获取到绑定的最新值的；

所以，如果在导出的模块中修改了变化，那么导入的地方可以实时获取最新的变量；
- 注意：在导入的地方不可以修改变量，因为它只是被绑定到了这个变量上（其实是一个常量）

#### 3.3 node对ES Module的支持

在最新的Current版本（v14.13.1）中，支持es module。需要进行如下操作：

- 方式一：在package.json中配置 type: module
- 方式二：文件以.mjs 结尾，表示使用的是ES Module；

**ES Module与CommonJs的交互**

通常情况下，CommonJS不能加载ES Module

- 因为CommonJS是同步加载的，但是ES Module必须经过静态分析等，无法在这个时候执行JavaScript代码；
- 但是这个并非绝对的，某些平台在实现的时候可以对代码进行针对性的解析，也可能会支持；
- Node当中是不支持的；

多数情况下，ES Module可以加载CommonJS
- ES Module在加载CommonJS时，会将其module.exports导出的内容作为default导出方式来使用；
- 这个依然需要看具体的实现，比如webpack中是支持的、Node最新的Current版本也是支持的；



​     