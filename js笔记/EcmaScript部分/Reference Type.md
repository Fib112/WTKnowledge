### Reference Type

一个动态执行的方法调用可能会丢失 `this`。如：

```javascript
let user = {
  name: "John",
  hi() { alert(this.name); },
  bye() { alert("Bye"); }
};

user.hi(); // 正常运行

// 现在让我们基于 name 来选择调用 user.hi 或 user.bye
(user.name == "John" ? user.hi : user.bye)(); // undefined
```

在最后一行有个在 `user.hi` 和 `user.bye` 中做选择的条件（三元）运算符。当前情形下的结果是 `user.hi`。

接着该方法被通过 `()` 立刻调用。但是并不能正常工作！

此处调用导致了一个错误，因为在该调用中 `"this"` 的值变成了 `undefined`。

这样是能工作的（对象.方法）：

```javascript
user.hi()
```

这就无法工作了：

```javascript
(user.name == "John" ? user.hi : user.bye)(); // undefined
```

### Reference type 解读

 在`obj.method()` 语句中的两个操作：

1. 首先，点 `'.'` 取了属性 `obj.method` 的值。
2. 接着 `()` 执行了它。

那么，`this` 的信息是怎么从第一部分传递到第二部分的呢？如果将这些操作放在不同的行，`this` 必定是会丢失的：

```javascript
let hi = user.hi
hi()
```

这里 `hi = user.hi` 把函数赋值给了一个变量，接下来在最后一行它是完全独立的，所以这里没有 `this`。

为确保 `user.hi()` 调用正常运行，JavaScript 玩了个小把戏 —— 点 `'.'` 返回的不是一个函数，而是一个特殊的 [Reference Type](https://tc39.github.io/ecma262/#sec-reference-specification-type) 的值

`Reference Type` 是 `ECMA` 中的一个“规范类型”。不能直接使用它，但它被用在 JavaScript 语言内部。

`Reference Type` 的值是一个三个值的组合 `(base, name, strict)`，其中：

- `base` 是对象。
- `name` 是属性名。
- `strict` 在 `use strict` 模式下为 true。

对属性 `user.hi` 访问的结果不是一个函数，而是一个 Reference Type 的值。对于 `user.hi`，在严格模式下是：

```javascript
(user, "hi", true)
```

当 `()` 被在 Reference Type 上调用时，它们会接收到关于对象和对象的方法的完整信息，然后可以设置正确的 `this`（在此处 `=user`）。

`Reference Type` 是一个特殊的“中间人”内部类型，目的是从 `.` 传递信息给 `()` 调用。

任何例如赋值 `hi = user.hi` 等其他的操作，都会将 Reference Type 作为一个整体丢弃掉，而会取 `user.hi`（一个函数）的值并继续传递。所以任何后续操作都“丢失”了 `this`。

因此，`this` 的值仅在函数直接被通过点符号 `obj.method()` 或方括号 `obj['method']()` 语法（此处它们作用相同）调用时才被正确传递。还有很多种解决这个问题的方式，例如 [func.bind()](https://zh.javascript.info/bind#solution-2-bind)。

**Exercise**

1. ```javascript
   let user = {
     name: "John",
     go: function() { alert(this.name) }
   }
   
   (user.go)()
   ```

   在上面代码中会提示`user`还未初始化，**出现此错误是因为在 `user = {...}` 后面漏了一个分号。**

   JavaScript 不会在括号 `(user.go)()` 前自动插入分号，所以解析的代码如下：

   ```javascript
   let user = { go:... }(user.go)()
   ```

   然后可以看到，这样的联合表达式在语法上是将对象 `{ go: ... }` 作为参数为 `(user.go)` 的函数。这发生在 `let user` 的同一行上，因此 `user` 对象是甚至还没有被定义，因此出现了错误。

   如果插入该分号，一切都变得正常：

   ```javascript
   let user = {
       name: "john",
       go: function() { alert(this.name) }
   };
   
   (user.go)()
   ```

   要注意的是，`(user.go)` 外边这层括号在这没有任何作用。通常用它们来设置操作的顺序，但在这里点符号 `.` 总是会先执行，所以并没有什么影响。分号是唯一重要的。

2. 解释 "this" 的值

   在下面的代码中，试图连续调用 `obj.go()` 方法 4 次。但是前两次和后两次调用的结果不同

   ```javascript
   let obj, method;
   
   obj = {
     go: function() { alert(this); }
   };
   
   obj.go();               // (1) [object Object]
   
   (obj.go)();             // (2) [object Object]
   
   (method = obj.go)();    // (3) undefined
   
   (obj.go || obj.stop)(); // (4) undefined
   ```

   因为第一个是一个常规的方法调用。第二个同样，括号没有改变执行的顺序，点符号总是先执行。在第三个这里有一个更复杂的 `(expression)()` 调用。这个调用就像被分成了两行（代码）一样：

   ```javascript
   f = obj.go; // 计算函数表达式
   f();        // 调用
   ```

   这里的 `f()` 是作为一个没有（设定）`this` 的函数执行的。`(4)`与 `(3)` 相类似，在括号 `()` 的左边也有一个表达式。要解释 `(3)` 和 `(4)` 得到这种结果的原因，只需回顾一下属性访问器（点符号或方括号）返回的是引用类型的值。除了方法调用之外的任何操作（如赋值 `=` 或 `||`），都会把它转换为一个不包含允许设置 `this` 信息的普通值。