### 一、 `eval`

#### 1.1 基本使用

内建函数 `eval` 允许执行一个代码字符串。语法如下：

```javascript
let result = eval(code);
```

代码字符串可能会比较长，包含换行符、函数声明和变量等。

#### 1.2 返回值

`eval` 的结果是最后一条语句的结果。如：

```javascript
let value = eval('1+1');
alert(value); // 2
```

#### 1.3 词法环境

`eval` 内的代码在当前词法环境（lexical environment）中执行，因此它能访问外部变量：

```javascript
let a = 1;

function f() {
  let a = 2;

  eval('alert(a)'); // 2
}

f();
```

它也可以更改外部变量：

```javascript
let x = 5;
eval("x = 10");
alert(x); // 10，值被更改了
```

严格模式下，`eval` 有属于自己的词法环境。因此我们不能从外部访问在 `eval` 中声明的函数和变量：

```javascript
eval("let x = 5; function f() {}");

alert(typeof x); // undefined（没有这个变量）
// 函数 f 也不可从外部进行访问
```

如果不启用严格模式，`eval` 没有属于自己的词法环境，这时就可以从外部访问变量 `x` 和函数 `f`。

### 二、 使用`"eval"`

现代编程中，已经很少使用 `eval` 了。因为它会破坏代码结构，而且还会产生一些副作用。

代码压缩工具将局部变量重命名为更短的变量（例如 `a` 和 `b` 等），以使代码体积更小。这通常是安全的，但在使用了 `eval` 的情况下就不一样了，因为局部变量可能会被 `eval` 中的代码访问到。因此压缩工具不会对所有可能会被从 `eval` 中访问的变量进行重命名。这样会导致代码压缩率降低。在 `eval` 中使用外部局部变量也被认为是一个坏的编程习惯，因为这会使代码维护变得更加困难。

有两种方法可以完全避免此类问题。

1. **如果 `eval` 中的代码没有使用外部变量，以 `window.eval(...)` 的形式调用 `eval`。**通过这种方式，该代码便会在全局作用域内执行：

   ```javascript
   let x = 1;
   {
     let x = 5;
     window.eval('alert(x)'); // 1（全局变量）
   }
   ```

2. **如果 `eval` 中的代码需要访问局部变量，可以使用 `new Function` 替代 `eval`，并将它们作为参数传递：**

   ```javascript
   let f = new Function('a', 'alert(a)');
   
   f(5); // 5
   ```

   `new Function` 从字符串创建一个函数，并且也是在全局作用域中的。所以它无法访问局部变量。但是，正如上面的示例一样，将它们作为参数进行显式传递要清晰得多。