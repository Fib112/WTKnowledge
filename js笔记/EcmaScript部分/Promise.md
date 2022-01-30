### 一、`Promise`基本使用

#### 1.1`reject`与异常

在Promise中显式抛出异常或`reject`都会在微任务中抛出异常，通过`setTimeout`抛出异常则会在宏任务中抛出异常。而`catch`只能捕捉微任务中的异常，蓑衣此时`catch`无法捕捉`setTimeout`所抛出的异常

#### 1.2 `Promise.resolve`

通过调用`Promise.resolve()`静态方法，可以实例化一个`fulfilled`的`Promise`。这个`fulfilled`的`Promise`的值对应着传给`Promise.resolve()`的第一个参数。使用这个静态方法，实际上可以把任何值都转换为一个期约，如：

```javascript
console.log(Promise.resolve()) // Promise<fulfilled>: undefined
console.log(Promise.resolve(3, 4, 5)) // Promise<fulfilled>: 3
console.log(Promise.resolve(new Error(1))) //Promise<fulfilled>: Error: 1
```

对这个静态方法而言，如果传入的参数本身是一个期约，那它的行为就类似于一个空包装。因此，`Promise.resolve()`可以说是一个幂等方法，如：

```javascript
let p = Promise.resolve(1)
let p1 = Promise.resolve(p)

console.log(p === p1) //true
```

#### 1.3 `Promise.reject`

与`Promise.resolve()`类似，`Promise.reject()`会实例化一个`rejected`的`Promise`并抛出一个异步错误（这个错误不能通过`try/catch`捕获，而只能通过拒绝处理程序捕获）。下面的两个`Promise`实例实际上是一样的：

```javascript
let p = new Promise((resolve, reject) => reject())

let p1 = Promise.reject()
```

这个`rejected`的`Promise`的理由就是传给`Promise.reject()`的第一个参数。这个参数也会传给后续的拒绝处理程序：

```javascript
let p = Promise.reject(1) // Promise<rejected>: 1

p.catch(err => console.log(err)) //1
```

关键在于，`Promise.reject()`并没有照搬`Promise.resolve()`的幂等逻辑。如果给它传一个`Promise`对象，则这个`Promise`会成为它返回的`rejected Promise`的理由：

```javascript
console.log(Promise.reject(Promise.resolve(1))) // Promise<rejected>: Promise<resolve>: 1
```

### 二、 `Promise`实例方法

#### 2.1 `then`方法

`Promise.prototype.then()`是为`Promise`实例添加处理程序的主要方法。这个`then()`方法接收最多两个参数：`onResolved`处理程序和`onRejected`处理程序。这两个参数都是可选的，如果提供的话，则会在`Promise`分别进入`fulfilled`和`rejected`状态时执行。

`Promise.prototype.then()`方法返回一个新的`Promise`实例，这个新`Promise`实例基于`onResovled`处理程序的返回值构建。换句话说，该处理程序的返回值会通过`romise.resolve()`包装来生成新期约。如果没有提供`onResolved`，则`Promise.resolve()`就会包装上一个`Promise fulfilled`之后的值。如果没有显式的返回语句，则`Promise.resolve()`会包装默认的返回值`undefined`：

```javascript
let p = Promise,resolve(1)
setTimeout(() => console.log(p.then())) // Promise<fulfilled>: 1   p.then() !== p
setTimeout(() => console.log(p.then(value => return 2))) // Promise<fulfilled>: 2
setTimeout(() => console.log(p,then(value => {}))) // Promise<fulfilled>: undefined
setTimeout(() => console.log(p.then(value => p) === p)) //false
```

如果有显式的返回值，则`Promise.resolve()`会包装这个值：

```javascript
let p = Promise.resolve(1)
setTimeout(() => console.log(p.then(value => 2))) // Promise<fulfilled>: 2
setTimeout(() => console.log(p.then(value => Promise.reject(3)))) // Promise<rejected>: 3
```

抛出异常会返回`rejected`的`Promise`：

```javascript
let p = Promise.resolve(1)
setTimeout(() => console.log(p.then(value => throw 2))) // Promise<rejected>: 2
```

`onRejected`处理程序也与之类似：`onRejected`处理程序返回的值也会被`Promise.resolve()`包装，如果没有`onRejected`处理程序，则会将上一个`rejected`的`Promise`包装并返回，如果抛出异常则会返回`rejected`的`Promise`。乍一看这可能有点违反直觉，但是想一想，`onRejected`处理程序的任务正是捕获异步错误。因此，拒绝处理程序在捕获错误后不抛出异常是符合`Promise`的行为，应该返回一个`fulfilled`的`Promise`

#### 2.2 `catch`方法

`Promise.prototype.catch()`方法用于给`Promise`添加拒绝处理程序。这个方法只接收一个参数：`onRejected`处理程序。事实上，这个方法就是一个语法糖，调用它就相当于调用`Promise.prototype.then(null, onRejected)`。在返回新`Promise`实例方面，`Promise.prototype.catch()`的行为与`Promise.prototype.then()`的`onRejected`处理程序是一样的。

**未处理的`rejection`**

当一个 error 没有被处理，如忘了在链的尾端附加 `.catch`，像这样：

```javascript
new Promise(function() {
  noSuchFunction(); // 这里出现 error（没有这个函数）
})
  .then(() => {
    // 一个或多个成功的 promise 处理程序（handler）
  }); // 尾端没有 .catch
```

如果出现`error`，`promise`的状态将变为 `rejected`，然后执行应该跳转至最近的`rejection`处理程序`（handler）`。但是上面这个例子中并没有这样的处理程序`（handler）`。因此 `error `会“卡住（stuck）”。没有代码来处理它。

发生一个常规的错误（error）并且未被 `try..catch` 捕获时，脚本会停止执行，并在控制台（console）中留下了一个信息。对于在`promise`中未被处理的`rejection`，也会发生类似的事儿。`JavaScript`引擎会跟踪此类 `rejection`，在这种情况下会生成一个全局的`error`。在浏览器中，可以使用 `unhandledrejection` 事件来捕获这类 error：

```javascript
window.addEventListener('unhandledrejection', function(event) {
  // 这个事件对象有两个特殊的属性：
  alert(event.promise); // [object Promise] - 生成该全局 error 的 promise
  alert(event.reason); // Error: Whoops! - 未处理的 error 对象
});

new Promise(function() {
  throw new Error("Whoops!");
}); // 没有用来处理 error 的 catch
```

这个事件是HTML 标准的一部分。如果出现了一个`error`，并且在这儿没有 `.catch`，那么 `unhandledrejection` 处理程序（handler）就会被触发，并获取具有 error 相关信息的 `event` 对象，所以此时就能做一些后续处理了。通常此类 error 是无法恢复的，所以最好的解决方案是将问题告知用户，并且可以将事件报告给服务器。

#### 2.3 `finally`方法

`Promise.prototype.finally()`方法用于给期约添加`onFinally`处理程序，这个处理程序在`Promise`转换为`fulfilled`或`rejected`状态时都会执行。这个方法可以避免`onResolved`和`onRejected`处理程序中出现冗余代码。但`onFinally`处理程序没有办法知道期约的状态是解决还是拒绝，所以这个方法主要用于添加清理代码。并且`onFinally`程序也是异步执行的。

`Promise.prototype.finally()`方法返回一个新的`Promise`实例，这个新`Promise`实例不同于`then()`或`catch()`方式返回的实例。因为`onFinally`被设计为一个状态无关的方法，所以在大多数情况下它将表现为父`Promise`的传递，对于`fulfilled`状态和`rejected`状态都是如此。

如果返回的是一个`pending`的`Promise`，或者`onFinally`处理程序抛出了错误（显式抛出或返回了一个`rejected`的`Promise`），则会返回相应的（`pending`或`rejected`的`Promise`），如下所示：

```javascript
let p = Promise.resolve(1)

// Promise<pending>
setTimeout(() => console.log(p.finally(() => new Promise((resolve, reject) => {})))
setTimeout(() => console.log(p.finally(() => throw 2))) // Promise<rejected>: 2
setTimeout(() => console.log(p.finally(() => Promise.reject(3)))) // Promise<rejected>: 3
```

返回`pending`的`Promise`的情形并不常见，这是因为只要`Promise`进入`fulfilled`后，`onFinally`仍然会原样后传初始的`Promise`：

```javascript
let p = Promise.resolve(1)
let p1 = p.finally(() => new Promise((resolve, reject) => setTimeout(resolve, 1000, 2))
setTimeout(console.log, 0, p1) // Promise<pending>
setTimeout(console.log, 1500, p1) // Promise<resolve>: 1
```

### 三、`Promise`合成

`Promise`类提供两个将多个`Promise`实例组合成一个`Promise`的静态方法：`Promise.all()`和`Promise.race()`。而合成后`Promise`的行为取决于内部`Promise`的行为。

#### 3.1 `Promise.all`

`Promise.all()`静态方法创建的`Promise`会在一组`Promise`全部`fulfilled`之后再进入`fulfilled`。这个静态方法接收一个可迭代对象，返回一个新`Promise`：

```javascript
let p1 = Promise.all([
    Promise.resolve(1),
    Promise.resolve(2)
]) // p1: Promise<fulfilled>: Array[1, 2]

// 可迭代对象中的元素会通过Promise.resolve转化为Promise
let p2 = Promise.all([3, 4]) // p2: Promise<fulfilled>: Array[3, 4]

//空的可迭代对象等价于Promise.resolve()
let p3 = Promise.all([]) // p3: Promise<fulfilled>: Array[]

// 不传入参数或传入不可迭代的对象会异步抛出TypeError
let p4 = Promise() // p4: Promise<rejected>: TypeError: ...
// 随后在微任务队列中抛出错误,该错误可被p4.catch(onRejected)捕捉
```

如果至少有一个包含的`Promise`处于`pending`，则合成的`Promise`也会处于`pending`。如果有一个包含的`Promise`处于`rejected`，则合成的`Promise`也会处于`rejected`。则第一个`rejected`的`Promise`会将自己的`reject reason`作为合成`Promise`的`reject Reason`。之后再`reject`的`Promise`不会影响最终`Promise`的`reject reason`。不过，这并不影响所有包含`Promise`正常的`reject`操作。合成的`Promise`会静默处理所有包含`Promise`的`reject`操作，如下所示：

```javascript
// 虽然只有第一个Promise的reject reason会进入拒绝处理程序,但第二个Promise会被静默处理,不会抛出异常
let p = Promise.all([
    Promise.resolve(1),
    new Promise((resolve, reject) => setTimeout(reject, 0))
])
p.catch(err => setTimeout(console.log, 0, err)) //打印1
```

如果所有`Promise`都成功进入`fulfilled`，则合成`Promise`的值就是所有包含`Promise`的`fulfilled`值的数组，按照迭代器顺序：

```javascript
let p = Promise.all([
    Promise.resolve(1),
    Promise.resolve(),
    Promise.resolve(3)
])
p.then(values => {
    for (let value of values) {
        console.log(value)
    }
}) // 1, undefined, 3
```

#### 3.2 `Promise.allSettled`

如果任意的`promise reject`，则 `Promise.all` 整个将会进入`reject`。当我们需要所有结果都成功时，它对这种“全有或全无”的情况很有用。而`Promise.allSettled` 等待所有的`promise`都被进入`settle`，无论结果如何,结果数组会包含：

- `{status:"fulfilled", value:result}` 对于成功的响应，
- `{status:"rejected", reason:error}` 对于 error

所以，对于每个`promise`，都得到了其状态（status）和 `value/reason`

#### 3.3 `Promise.race`

`Promise.race()`静态方法返回一个包装`Promise`，是一组集合中最先`fulfilled`或`rejected`的`Promise`的镜像。这个方法接收一个可迭代对象，返回一个新`Promise`。其他规则和`Promise.all`相同

#### 3.4 串行`Promise`合成

`Promise`的另一个主要特性：异步产生值并将其传给处理程序。后续`Promise`基于之前`Promise`的返回值来串联是`Promise`的基本功能。这很像函数合成，即将多个函数合成为一个函数，比如：

```javascript
let addTwo = x => x + 2
let addThree = x => x + 3
let addFive = x => x + 5
let addTen = x => {
    return Promise.resolve(x)
	.then(addFive)
    .then(addThree)
    .then(addTwo)
}
addTen(8).then(console.log) // 18
```

使用`Array.prototype.reduce()`可以写成更简洁的形式：

```javascript
let addTwo = x => x + 2
let addThree = x => x + 3
let addFive = x => x + 5
let addTen = x => {
    return [addTwo, addThree, addFive]
        .reduce((promise, fn) => promise.then(fn), Promise.resolve(x))
}
addTen(8).then(console.log) // 18
```

这种模式可以提炼出一个通用函数，可以把任意多个函数作为处理程序合成一个连续传值的`Promise`连锁。这个通用的合成函数可以这样实现：

```javascript
let addTwo = x => x + 2
let addThree = x => x + 3
let addFive = x => x + 5
let compose = (...fns) => {
    return (x) => fns.reduce((promise, fn) => {
        return promise.then(fn)
    }, Promise.resolve(x))
}

let addTen = compose(addTwo, addThree, addFive)
addTen(8).then(console.log)
```

### 四、异步函数

异步函数，也称为`async/await`（语法关键字），是`ES6 Promise`模式在`ECMAScript`函数中的应用。`async/await`是`ES8`规范新增的。这个特性从行为和语法上都增强了`JavaScript`，让以同步方式写的代码能够异步执行。

#### 4.1 `async`

`async`关键字用于声明异步函数。这个关键字可以用在函数声明、函数表达式、箭头函数和方法上：

```javascript
async function foo() {}
let bar = async function() {}
let baz = async () => {}
class Qux {
    async qux() {}
}
```

使用`async`关键字可以让函数具有异步特征，但总体上其代码仍然是同步求值的。而在参数或闭包方面，异步函数仍然具有普通`JavaScript`函数的正常行为。正如下面的例子所示，`foo()`函数仍然会在后面的指令之前被求值：

```javascript
async function foo() {
    console.log(1)
}
foo()
console.log(2)
// 1
// 2
```

不过，异步函数如果使用`return`关键字返回了值（如果没有`return`则会返回`undefined`），这个值会被`Promise.resolve()`包装成一个`Promise`对象。异步函数始终返回`Promise`对象。在函数外部调用这个函数可以得到它返回的`Promise`，当然，也可以直接返回一个`Promise`对象。

异步函数的返回值期待（但实际上并不要求）一个实现`thenable`接口的对象，但常规的值也可以。如果返回的是实现`thenable`接口的对象，则这个对象可以由提供给`then()`的处理程序“解包”。如果不是，则返回值就被当作`fulfilled`的`Promise`。如：

```javascript
// 返回原始值
async function foo() {
    return 'foo'
}
foo().then(console.log) // foo

// 返回没有实现thenable接口的对象
async function bar() {
    return ['bar']
}
bar().then(console.log) // ['bar']

// 返回实现thenable接口的对象
async function baz() {
    return {
        then(callback) {
            callback('baz')
        } 
    }
}
baz().then(console.log) // baz then中的callback仍然是异步执行
console.log(baz()) // Promise<fulfilled>: 'baz'
```

与在`Promise`处理程序中一样，在异步函数中抛出错误会返回`rejected`的`Promise`：

```javascript
async function foo() {
    console.log(1)
    throw 3
}
foo().then(console.log)
console.log(2)
// 1
// 2
// 3
```

不过，`rejected`的`Promise`的错误不会被异步函数捕获：

```javascript
async function foo() {
    console.log(1)
    Promise.reject(3)
}
foo().catch(console.log)
console.log(2)
// 1
// 2
// Uncaught (in Promise) 3
```

但返回`rejected`的`Promise`会被异步函数捕捉到错误：

```javascript
async function foo() {
    console.log(1)
    return Promise.reject(3)
}
foo().then(console.log)
console.log(2)
// 1
// 2
// 3
```

#### 4.2 `await`

因为异步函数主要针对不会马上完成的任务，所以自然需要一种暂停和恢复执行的能力。使用`await`关键字可以暂停异步函数代码的执行，等待`Promise`进入`fulfilled`。如：

```javascript
async function foo() {
    let p = new Promise((resolve, reject) => {
        setTimeout(resolve, 1000, 2)
    })
    console.log(await p)
}
foo()
console.log(1)
// 1
// 两秒后
// 2
```

`await`关键字会暂停执行异步函数后面的代码，让出`JavaScript`运行时的执行线程。这个行为与生成器函数中的`yield`关键字是一样的。`await`关键字同样是尝试“解包”对象的值，然后将这个值传给表达式，再异步恢复异步函数的执行。

`await`关键字的用法与`JavaScript`的一元操作一样。它可以单独使用，也可以在表达式中使用。如：

```javascript
// 异步打印"foo"
async function foo() {
    console.log(await Promise.resolve('foo'))
}
foo()

// 异步打印"bar"
async function bar() {
    return await Promise.resolve('bar')
}
bar().then(console.log)

// 1000毫秒后异步打印"baz"
async function baz() {
    awiat new Promise((resolve, reject) => {
        resolve('baz')
    }, 1000)
}
```

`await`关键字期待（但实际上并不要求）一个实现`thenable`接口的对象，但常规的值也可以。如果是实现`thenable`接口的对象，则这个对象可以由`await`来“解包”。如果不是，则这个值就被当作已经`fulfilled`的`Promise`。如：

```javascript
// 等待一个原始值
async function foo() {
    console.log(await 'foo')
}
foo() // foo

// 等待一个没有实现thenable接口的对象
async function bar() {
    console.log(await ['bar'])
}
bar() // ['bar']

// 等待一个实现了thenable接口的非Promise对象
async function baz() {
    const thenable = {
        then(callback) {
            callback('baz')
        }
    }
    console.log(await thenable)
}
baz() // baz
```

等待会抛出错误的同步操作，会返回`rejected`的`Promise`，此错误可以通过`try...catch`捕捉：

```javascript
async function foo() {
    console.log(1)
    await (() => { throw 3 })()
}
// 给返回的Promise添加一个拒绝处理程序
foo().catch(console.log)
console.log(2)
// 1
// 2
// 3
console.log(foo()) 
// Promise<rejected>: 3
// Uncaught (in promise) 3

async function bar() {
    try {
        console.log(1)
        await(() => throw 3)()
    } catch (err) {
        console.log(err)
    }
}
let p = foo()
console.log(2)
setTimeout(console.log, 0, p)
// 1
// 3
// 2
// Promise<fulfilled>: undefined
```

如前面的例子所示，单独的`Promise.reject()`不会被异步函数捕获，而会抛出未捕获错误。不过，对`rejected`的`Promise`使用`await`则会释放（unwrap）错误值（将`rejected Promise`返回）：

```javascript
async function foo() {
    console.log(1)
    await Promise.reject(3)
    console.log(4) //这行不会执行,因为在上一行隐式执行了return
}
foo().catch(console.log)
console.log(2)
// 1
// 2
// 3 (异步执行)
console.log(foo())
// Promise<rejected>: 3
// Uncaught (in promise) 3
```

没有`await`的`throw error`：在`async`函数中没有`await`操作符的情况下抛出异常，函数会返回一个`rejected`的`Promise`，并且这个异常可以通过`then`中的`onRejected`或`catch`捕捉。如果将`throw`操作包裹在`try...catch`中，则可以通过`catch`捕捉，此时错误不会导致函数返回一个`rejected`的`Promise`：

```javascript
async function foo() {
    throw 1
}
let p = foo()
console.log(p)
p.catch(console.log)
// Promise<rejected>: 1
// 1
```

没有`await`的`rejected Promise`：在`async`函数中没有`await`操作符的情况下生成`rejected`的`Promise`，此时无法通过`try...catch`捕捉错误，也无法在函数返回的`Promise`上的`catch`捕捉，只能通过这个`rejected`的`Promise`的`catch`捕捉。该`Promise`不会导致`async`函数生成`rejected`的`Promise`：

```javascript
async function foo() {
    Promise.reject('foo')
    .catch(console.log)
}
let p = foo()
setTimeout(console.log, 0, p)
// foo
// Promise<fulfilled>: undefined
```

**`await`的限制**

await关键字必须在异步函数中使用，不能在顶级上下文如`<script>`标签或模块中使用。不过，定义并立即调用异步函数是没问题的。(新特性：从 `V8`引擎 8.9+ 版本开始，顶层 `await`可以在模块中工作)

#### 4.3 停止和恢复执行

使用`await`关键字之后的区别其实比看上去的还要微妙一些。比如，下面的例子中按顺序调用了3个函数，但它们的输出结果顺序是相反的：

```javascript
async function foo() {
    console.log(await Promise.resolve('foo'))
}
async function bar() {
    console.log(await 'bar')
}
async function baz() {
    console.log('baz')
}
foo()
bar()
baz()
// baz
// foo
// bar
```

`await`并非只是等待一个值可用那么简单。`JavaScript`运行时在碰到`await`关键字时，会记录在哪里暂停执行。等到`await`右边的值可用了，`JavaScript`运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行。因此，即使`await`后面跟着一个立即可用的值，函数的其余部分也会被异步求值。
