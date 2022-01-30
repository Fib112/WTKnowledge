### 一、Generator

常规函数只会返回一个单一值（或者不返回任何值）。而`Generator`可以按需一个接一个地返回(`yield`)多个值。它们可与`iterable`完美配合使用，从而可以轻松地创建数据流。

#### 1.1 `Generator`基本使用

要创建一个 generator，我们需要一个特殊的语法结构：`function*`，即所谓的 `generator function`。如：

```javascript
function* gen() {
    yield 1
    yield 2
    return 3
}
```

Generator 函数与常规函数的行为不同。在此类函数被调用时，它不会运行其代码。而是返回一个被称为 `generator object”`的特殊对象，来管理执行流程。如：

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

// "generator function" 创建了一个 "generator object"
let generator = generateSequence();
alert(generator); // [object Generator]
```

到目前为止，上面这段代码中的 **函数体** 代码还没有开始执行。一个`generator`的主要方法就是 `next()`。当被调用时（指 `next()` 方法），它会恢复函数体的运行，执行直到最近的 `yield <value>` 语句（`value` 可以被省略，默认为 `undefined`）。然后函数执行暂停，并将产出的`(yielded)`值返回到外部代码。`next()` 的结果始终是一个具有两个属性的对象：

- `value`: 产出的`(yielded)`的值。
- `done`: 如果`generator`函数已执行完成则为 `true`，否则为 `false`。

基本使用：

```javascript
function* gen() {
    yield 1
    yield 2
    return 3
}

let generator = gen()

console.log(generator.next()) // {value: 1, done: false}
console.log(generator.next()) // {value: 2, done: false}
console.log(generator.next()) // {value: 3, done: true}
console.log(generator.next()) // 生成器执行完毕,此时调用next方法只会返回 {done: true}
```

#### 1.2 可迭代的`Generator`

看到`next`方法，可知道`generator`是可迭代`(iterable)`的，可以使用 `for..of` 循环遍历它所有的值：

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

for(let value of generator) {
  alert(value); // 1，然后是 2
}
```

上面这个例子会先显示 1，然后是 2，然后就没了。它不会显示 3！这是因为当 `done: true` 时，`for..of` 循环会忽略最后一个 `value`。因此，如果想要通过 `for..of` 循环显示所有的结果，必须使用 `yield` 返回它们：

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}

let generator = generateSequence();

for(let value of generator) {
  alert(value); // 1，然后是 2，然后是 3
}
```

因为`generator`是可迭代的，所以可以使用`iterator`的所有相关功能，例如：`spread`语法`...`：

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}

let sequence = [0, ...generateSequence()]
alert(sequence) // 0, 1, 2, 3
```

**使用`Generator`进行迭代**

可以通过提供一个`generator`函数作为对象的 `Symbol.iterator`，来使用 generator 进行迭代：

```javascript
let range = {
  from: 1,
  to: 5,

  *[Symbol.iterator]() { // [Symbol.iterator]: function*() 的简写形式
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

alert( [...range] ); // 1,2,3,4,5
```

之所以代码正常工作，是因为 `range[Symbol.iterator]()` 现在返回一个`generator`，而`generator`方法正是 `for..of` 所期望的：

- 它具有 `.next()` 方法
- 它以 `{value: ..., done: true/false}` 的形式返回值

当然，这不是巧合。`Generator`被添加到`JavaScript`语言中是有对`iterator`的考量的，以便更容易地实现 `iterator`。带有`generator`的变体比原来的 `range` 迭代代码简洁得多，并且保持了相同的功能。

#### 1.3 `Generator`组合

`Generator`组合`（composition）`是 `generator` 的一个特殊功能，它允许透明地（transparently）将 `generator` 彼此“嵌入（embed）”到一起。对于 `generator` 而言，我们可以使用 `yield*` 这个特殊的语法来将一个 `generator` “嵌入”（组合）到另一个 generator 中：

```javascript
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generatePasswordCodes() {

  // 0..9
  yield* generateSequence(48, 57);

  // A..Z
  yield* generateSequence(65, 90);

  // a..z
  yield* generateSequence(97, 122);

}

let str = '';

for(let code of generatePasswordCodes()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```

`yield*` 指令将执行 **委托** 给另一个 `generator`。这个术语意味着 `yield* gen` 在 `generator gen` 上进行迭代，并将其产出（yield）的值透明地（transparently）转发到外部。就好像这些值就是由外部的 `generator yield` 的一样。

#### 1.4 `yield`输出与输入

目前看来，`generator`和可迭代对象类似，都具有用来生成值的特殊语法。但实际上，`generator` 更加强大且灵活。这是因为 `yield` 是一条双向路（two-way street）：它不仅可以向外返回结果，而且还可以将外部的值传递到 `generator` 内。调用 `generator.next(arg)`，我们就能将参数 `arg` 传递到 generator 内部。这个 `arg` 参数会变成 `yield` 的结果。如：

```javascript
function* gen() {
  // 向外部代码传递一个问题并等待答案
  let result = yield "2 + 2 = ?"; // (*)

  alert(result);
}

let generator = gen();

let question = generator.next().value; // <-- yield 返回的 value(str: '2 + 2 = ?')

generator.next(4); // --> 将结果传递到 generator 中,成为yield的结果并赋值给result
```

1. 第一次调用 `generator.next()` 应该是不带参数的（如果带参数，那么该参数会被忽略）。它开始执行并返回第一个 `yield "2 + 2 = ?"` 的结果。此时，`generator`执行暂停，而停留在 `(*)` 行上。
2. 然后，`yield` 的结果进入调用代码中的 `question` 变量。
3. 在 `generator.next(4)`，`generator`恢复执行，并获得了 `4` 作为上一个`yield`的结果：`let result = 4`。然后执行`alert(result)`并返回`{done: true}`

#### 1.5 抛出异常

外部代码可能会将一个值传递到`generator`，作为 `yield` 的结果。但是它也可以在那里发起（抛出）一个 `error`。这很自然，因为`error`本身也是一种结果。

如果要向 `yield` 传递一个`error`，应该调用 `generator.throw(err)`。在这种情况下，`err` 将被抛到对应的 `yield` 所在的那一行。例如，`"2 + 2?"` 的 yield 导致了一个 error：

```javascript
function* gen() {
    try {
        let result = yield '2 + 2 = ?' // (1)
        alert("这行代码不会被执行,因为在上面抛出了异常,代码转向catch分支");
    } catch(err) {
        alert(err)
    }
}

let generator = gen()
let question = generator.next().value
generator.throw(new Error("The answer is not found in my database")); // (2)
```

在 `(2)` 行引入到`generator`的`error`导致了在 `(1)` 行中的 `yield` 出现了一个异常。在上面这个例子中，`try..catch` 捕获并显示了这个`error`。如果没有捕获它，那么就会像其他的异常一样，它将从`generator`“掉出”到调用代码中。调用代码的当前行是 `generator.throw` 所在的那一行，标记为 `(2)`。所以可以在这里捕获它，就像这样：

```javascript
function* gen() {
    let result = yield '2 + 2 = ?'
}

let generator = gen()
let question = generator.next().value
try {
    generator.throw(new Error("The answer is not found in my database"))
} catch(err) {
    alert(err)
}
```

如果没有在那里捕获这个`error`，那么，通常，它会掉入外部调用代码（如果有），如果在外部也没有被捕获，则会杀死脚本。

### 二、异步迭代&`generator`

异步迭代允许我们对按需通过异步请求而得到的数据进行迭代。例如，通过网络分段（chunk-by-chunk）下载数据时。异步生成器`(generator)`使这一步骤更加方便。

#### 2.1 异步可迭代对象

当值是以异步的形式出现时，例如在 `setTimeout` 或者另一种延迟之后，就需要异步迭代。最常见的场景是，对象需要发送一个网络请求以传递下一个值，要使对象异步迭代：

1. 使用 `Symbol.asyncIterator` 取代 `Symbol.iterator`

2. `next()`方法应该返回一个`promise`(带有下一个值，并且状态为`fulfilled`)
   - 关键字 `async` 可以实现这一点，可以简单地使用 `async next()`
   
3. 我们应该使用`for await (let item of iterable)`循环来迭代这样的对象。

   - 注意关键字 `await`

创建一个可迭代的 `range` 对象，现在它将异步地每秒返回一个值：

```javascript
let range = {
    from: 1,
    to: 5,
    [Symbol.asyncIterator]() { // (1)
        return {
            current: this.from,
            last: this.to,
            async next() {  // (2)
                await new Promise(resolve => setTimeout(resolve, 1000)) (3)
                
                if(this.current <= this.last) {
                    return {done: false, value: this.current++}
                } else {
                    return {done: true}
                }
            }
        }
    }
}

(async () => {
    for await (let value of range) { // (4)
        alert(value)
    }
})
```

其结构与常规的 iterator 类似:

1. 为了使一个对象可以异步迭代，它必须具有方法 `Symbol.asyncIterator` `(1)`。
2. 这个方法必须返回一个带有 `next()` 方法的对象，`next()` 方法会返回一个 promise `(2)`。
3. 这个 `next()` 方法可以不是 `async` 的，它可以是一个返回值是一个 `promise` 的常规的方法，但是使用 `async` 关键字可以允许在方法内部使用 `await`，所以会更加方便。这里只是用于延迟 1 秒的操作 `(3)`。
4. 我们使用 `for await(let value of range)` `(4)` 来进行迭代，也就是在 `for` 后面添加 `await`。它会调用一次 `range[Symbol.asyncIterator]()` 方法一次，然后调用它的 `next()` 方法获取值。

这是一个对比 Iterator 和异步 iterator 之间差异的表格：

|                          | Iterator          | 异步iterator           |
| ------------------------ | ----------------- | ---------------------- |
| 提供 iterator 的对象方法 | `Symbol.iterator` | `Symbol.asyncIterator` |
| `next()` 返回的值是      | 任意值            | `Promise`              |
| 要进行循环，使用         | `for..of`         | `for await..of`        |

**Spread 语法 `...` 无法异步工作**

需要常规的同步 iterator 的功能，无法与异步 iterator 一起使用。例如，spread 语法无法工作：

```javascript
alert( [...range] ); // Error, no Symbol.iterator
```

这很正常，因为它期望找到 `Symbol.iterator`，而不是 `Symbol.asyncIterator`。`for..of` 的情况和这个一样：没有 `await` 关键字时，则期望找到的是 `Symbol.iterator`。

#### 2.2 异步 generator (finally)

对于大多数的实际应用程序，当想要创建一个异步生成一系列值的对象时，都可以使用异步`generator`。

语法很简单：在 `function*` 前面加上 `async`。这即可使 generator 变为异步的。然后使用 `for await (...)` 来遍历它，像这样：

```javascript
async function* generateSequence(start, end) {

  for (let i = start; i <= end; i++) {

    // 哇，可以使用 await 了！
    await new Promise(resolve => setTimeout(resolve, 1000));

    yield i;
  }

}

(async () => {

  let generator = generateSequence(1, 5);
  for await (let value of generator) {
    alert(value); // 1，然后 2，然后 3，然后 4，然后 5（在每个 alert 之间有延迟）
  }

})();
```

常规`generator`与异步`generator`的差异：对于异步 generator，`generator.next()` 方法是异步的，它返回 `promise`。在一个常规的`generator`中，使用 `result = generator.next()` 来获得值。但在一个异步 `generator`中，应该添加 `await` 关键字，像这样：

```javascript
result = await generator.next(); // result = {value: ..., done: true/false}
```

就是为什么异步 generator 可以与 `for await...of` 一起工作。

**异步可迭代对象`range`**

常规的 generator 可用作 `Symbol.iterator` 以使迭代代码更短。与之类似，异步 generator 可用作 `Symbol.asyncIterator` 来实现异步迭代。例如，可以通过将同步的 `Symbol.iterator` 替换为异步的 `Symbol.asyncIterator`，来使对象 `range` 异步地生成值，每秒生成一个：

```javascript
let range = {
    from: 1,
    to: 5,
    
    async *[Symbol.asyncIterator]() {
        for (let i = this.from; i <= this.to, i++) {
            await new Promise(resolve => setTimeout(reslve, 1000))
            yield i
        }
    }
}

(async () => {
    for await (let value of range) {
        alert(value) // 1，然后 2，然后 3，然后 4，然后 5
    }
})
```

现在，value 之间的延迟为 1 秒。

从技术上讲，可以把 `Symbol.iterator` 和 `Symbol.asyncIterator` 都添加到对象中，因此它既可以是同步的（`for..of`）也可以是异步的（`for await..of`）可迭代对象。但是实际上，这将是一件很奇怪的事情。