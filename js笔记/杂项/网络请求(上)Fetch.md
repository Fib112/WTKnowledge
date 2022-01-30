### 一、`Fetch`

#### 1.1 `Fetch`简介

JavaScript 可以将网络请求发送到服务器，并在需要时加载新信息。

例如，可以使用网络请求来：

- 提交订单，
- 加载用户信息，
- 从服务器接收最新的更新，
- ……等。

所有这些都没有重新加载页面！

对于来自 JavaScript 的网络请求，有一个总称术语 “AJAX”（**A**synchronous **J**avaScript **A**nd **X**ML 的简称）。但是，我们不必使用 XML：这个术语诞生于很久以前，所以这个词一直在那儿。

有很多方式可以向服务器发送网络请求，并从服务器获取信息。

`fetch()` 方法是一种现代通用的方法。旧版本的浏览器不支持它（可以 polyfill），但是它在现代浏览器中的支持情况很好。

基本语法：

```javascript
let promise = fetch(url, [options])
```

- **`url`** —— 要访问的 URL。
- **`options`** —— 可选参数：method，header 等。

没有 `options`，那就是一个简单的 GET 请求，下载 `url` 的内容。

浏览器立即启动请求，并返回一个该调用代码应该用来获取结果的 `promise`。

获取响应通常需要经过两个阶段。

**第一阶段，当服务器发送了响应头（response header），`fetch` 返回的 `promise` 就使用内建的 [Response](https://fetch.spec.whatwg.org/#response-class) class 对象来对响应头进行解析。**

在这个阶段，可以通过检查响应头，来检查 HTTP 状态以确定请求是否成功，当前还没有响应体（response body）。

如果 `fetch` 无法建立一个 HTTP 请求，例如网络问题，亦或是请求的网址不存在，那么 promise 就会 reject。异常的 HTTP 状态，例如 404 或 500，不会导致出现 error。

可以在 response 的属性中看到 HTTP 状态：

- **`status`** —— HTTP 状态码，例如 200。
- **`ok`** —— 布尔值，如果 HTTP 状态码为 200-299，则为 `true`。

例如：

```javascript
let response = await fetch(url);

if (response.ok) { // 如果 HTTP 状态码为 200-299
  // 获取 response body（此方法会在下面解释）
  let json = await response.json();
} else {
  alert("HTTP-Error: " + response.status);
}
```

**第二阶段，为了获取 response body，需要使用一个其他的方法调用。**

`Response` 提供了多种基于 promise 的方法，来以不同的格式访问 body：

- **`response.text()`** —— 读取 response，并以文本形式返回 response，
- **`response.json()`** —— 将 response 解析为 JSON，
- **`response.formData()`** —— 以 `FormData` 对象的形式返回 response，
- **`response.blob()`** —— 以 [Blob](https://zh.javascript.info/blob)（具有类型的二进制数据）形式返回 response，
- **`response.arrayBuffer()`** —— 以 [ArrayBuffer](https://zh.javascript.info/arraybuffer-binary-arrays)（低级别的二进制数据）形式返回 response，
- 另外，`response.body` 是 [ReadableStream](https://streams.spec.whatwg.org/#rs-class) 对象，它允许逐块读取 body。

例如，从 GitHub 获取最新 commits 的 JSON 对象：

```javascript
let url = 'https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits';
let response = await fetch(url);

let commits = await response.json(); // 读取 response body，并将其解析为 JSON

alert(commits[0].author.login);
```

也可以使用纯 promise 语法，不使用 `await`：

```javascript
fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits')
  .then(response => response.json())
  .then(commits => alert(commits[0].author.login));
```

要获取响应文本，可以使用 `await response.text()` 代替 `.json()`：

```javascript
let response = await fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits');

let text = await response.text(); // 将 response body 读取为文本

alert(text.slice(0, 80) + '...');
```

作为一个读取为二进制格式的演示示例，fetch 并显示一张 [“fetch” 规范](https://fetch.spec.whatwg.org/) 中的图片：

```javascript
let response = await fetch('/article/fetch/logo-fetch.svg');

let blob = await response.blob(); // 下载为 Blob 对象

// 为其创建一个 <img>
let img = document.createElement('img');
img.style = 'position:fixed;top:10px;left:10px;width:100px';
document.body.append(img);

// 显示它
img.src = URL.createObjectURL(blob);

setTimeout(() => { // 3 秒后将其隐藏
  img.remove();
  URL.revokeObjectURL(img.src);
}, 3000);
```

**重要：**

只能选择一种读取 body 的方法。

如果已经使用了 `response.text()` 方法来获取 response，那么如果再用 `response.json()`，则不会生效，因为 body 内容已经被处理过了。

```javascript
let text = await response.text(); // response body 被处理了
let parsed = await response.json(); // 失败（已经被处理过了）
```

#### 1.2 `Response header`

Response header 位于 `response.headers` 中的一个类似于 Map 的 header 对象。

它不是真正的 Map，但是它具有类似的方法，可以按名称（name）获取各个 header，或迭代它们：

```javascript
let response = await fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits');

// 获取一个 header
alert(response.headers.get('Content-Type')); // application/json; charset=utf-8

// 迭代所有 header
for (let [key, value] of response.headers) {
  alert(`${key} = ${value}`);
}
```

#### 1.3 `Request header`

要在 `fetch` 中设置 request header，可以使用 `headers` 选项。它有一个带有输出 header 的对象，如下所示：

```javascript
let response = fetch(protectedUrl, {
  headers: {
    Authentication: 'secret'
  }
});
```

但是有一些无法设置的 header（详见 [forbidden HTTP headers](https://fetch.spec.whatwg.org/#forbidden-header-name)）：

- `Accept-Charset`, `Accept-Encoding`
- `Access-Control-Request-Headers`
- `Access-Control-Request-Method`
- `Connection`
- `Content-Length`
- `Cookie`, `Cookie2`
- `Date`
- `DNT`
- `Expect`
- `Host`
- `Keep-Alive`
- `Origin`
- `Referer`
- `TE`
- `Trailer`
- `Transfer-Encoding`
- `Upgrade`
- `Via`
- `Proxy-*`
- `Sec-*`

这些 header 保证了 HTTP 的正确性和安全性，所以它们仅由浏览器控制。

#### 1.4 `POST`请求

要创建一个 `POST` 请求，或者其他方法的请求，需要使用 `fetch` 选项：

- **`method`** —— HTTP 方法，例如 `POST`，
- **`body`**—— request body，其中之一：
  - 字符串（例如 JSON 编码的），
  - `FormData` 对象，以 `form/multipart` 形式发送数据，
  - `Blob`/`BufferSource` 发送二进制数据，
  - [URLSearchParams](https://zh.javascript.info/url)，以 `x-www-form-urlencoded` 编码形式发送数据，很少使用。

JSON 形式是最常用的。

例如，下面这段代码以 JSON 形式发送 `user` 对象：

```javascript
let user = {
  name: 'John',
  surname: 'Smith'
};

let response = await fetch('/article/fetch/post/user', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json;charset=utf-8'
  },
  body: JSON.stringify(user)
});

let result = await response.json();
alert(result.message);
```

注意，如果请求的 `body` 是字符串，则 `Content-Type` 会默认设置为 `text/plain;charset=UTF-8`。

但是，当要发送 JSON 时，会使用 `headers` 选项来发送 `application/json`，这是 JSON 编码的数据的正确的 `Content-Type`。

#### 1.5 发送图片

同样可以使用 `Blob` 或 `BufferSource` 对象通过 `fetch` 提交二进制数据。

例如，这里有一个 `<canvas>`，可以通过在其上移动鼠标来进行绘制。点击 “submit” 按钮将图片发送到服务器：

```html
<body style="margin:0">
  <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

  <input type="button" value="Submit" onclick="submit()">

  <script>
    canvasElem.onmousemove = function(e) {
      let ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };

    async function submit() {
      let blob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));
      let response = await fetch('/article/fetch/post/image', {
        method: 'POST',
        body: blob
      });

      // 服务器给出确认信息和图片大小作为响应
      let result = await response.json();
      alert(result.message);
    }

  </script>
</body>
```

注意，这里没有手动设置 `Content-Type` header，因为 `Blob` 对象具有内建的类型（这里是 `image/png`，通过 `toBlob` 生成的）。对于 `Blob` 对象，这个类型就变成了 `Content-Type` 的值。

可以在不使用 `async/await` 的情况下重写 `submit()` 函数，像这样：

```javascript
function submit() {
  canvasElem.toBlob(function(blob) {
    fetch('/article/fetch/post/image', {
      method: 'POST',
      body: blob
    })
      .then(response => response.json())
      .then(result => alert(JSON.stringify(result, null, 2)))
  }, 'image/png');
}
```

### 二、`FormData`

这一章是关于发送 HTML 表单的：带有或不带文件，带有其他字段等。

[FormData](https://xhr.spec.whatwg.org/#interface-formdata) 对象可以提供帮助，它是表示 HTML 表单数据的对象。

构造函数是：

```javascript
let formData = new FormData([form]);
```

如果提供了 HTML `form` 元素，它会自动捕获 `form` 元素字段。

`FormData` 的特殊之处在于网络方法（network methods），例如 `fetch` 可以接受一个 `FormData` 对象作为 body。它会被编码并发送出去，带有 `Content-Type: multipart/form-data`。

从服务器角度来看，它就像是一个普通的表单提交。

#### 2.1 发送表单

发送一个简单的表单，几乎就是一行代码：

```html
<form id="formElem">
  <input type="text" name="name" value="John">
  <input type="text" name="surname" value="Smith">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user', {
      method: 'POST',
      body: new FormData(formElem)
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

#### 2.2 `FormData`方法

可以使用以下方法修改 `FormData` 中的字段：

- `formData.append(name, value)` —— 添加具有给定 `name` 和 `value` 的表单字段，
- `formData.append(name, blob, fileName)` —— 添加一个字段，就像它是 `<input type="file">`，第三个参数 `fileName` 设置文件名（而不是表单字段名），因为它是用户文件系统中文件的名称，
- `formData.delete(name)` —— 移除带有给定 `name` 的字段，
- `formData.get(name)` —— 获取带有给定 `name` 的字段值，
- `formData.has(name)` —— 如果存在带有给定 `name` 的字段，则返回 `true`，否则返回 `false`。

从技术上来讲，一个表单可以包含多个具有相同 `name` 的字段，因此，多次调用 `append` 将会添加多个具有相同名称的字段。

还有一个 `set` 方法，语法与 `append` 相同。不同之处在于 `.set` 移除所有具有给定 `name` 的字段，然后附加一个新字段。因此，它确保了只有一个具有这种 `name` 的字段，其他的和 `append` 一样：

- `formData.set(name, value)`，
- `formData.set(name, blob, fileName)`。

可以使用 `for..of` 循环迭代 formData 字段：

```javascript
let formData = new FormData();
formData.append('key1', 'value1');
formData.append('key2', 'value2');

// 列出 key/value 对
for(let [name, value] of formData) {
  alert(`${name} = ${value}`); // key1=value1，然后是 key2=value2
}
```

#### 2.3 发送带有文件的表单

表单始终以 `Content-Type: multipart/form-data` 来发送数据，这个编码允许发送文件。因此 `<input type="file">` 字段也能被发送，类似于普通的表单提交。

这是具有这种形式的示例：

```html
<form id="formElem">
  <input type="text" name="firstName" value="John">
  Picture: <input type="file" name="picture" accept="image/*">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user-avatar', {
      method: 'POST',
      body: new FormData(formElem)
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

#### 2.4 发送具有`Blob`数据的表单

以 `Blob` 发送一个动态生成的二进制数据，例如图片，是很简单的。可以直接将其作为 `fetch` 参数的 `body`。

但在实际中，通常更方便的发送图片的方式不是单独发送，而是将其作为表单的一部分，并带有附加字段（例如 “name” 和其他 metadata）一起发送。

并且，服务器通常更适合接收多部分编码的表单（multipart-encoded form），而不是原始的二进制数据。

下面这个例子使用 `FormData` 将一个来自 `<canvas>` 的图片和一些其他字段一起作为一个表单提交：

```html
<body style="margin:0">
  <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

  <input type="button" value="Submit" onclick="submit()">

  <script>
    canvasElem.onmousemove = function(e) {
      let ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };

    async function submit() {
      let imageBlob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));

      let formData = new FormData();
      formData.append("firstName", "John");
      formData.append("image", imageBlob, "image.png");

      let response = await fetch('/article/formdata/post/image-form', {
        method: 'POST',
        body: formData
      });
      let result = await response.json();
      alert(result.message);
    }

  </script>
</body>
```

注意图片 `Blob` 是如何添加的：

```javascript
formData.append("image", imageBlob, "image.png");
```

就像表单中有 `<input type="file" name="image">` 一样，用户从他们的文件系统中使用数据 `imageBlob`（第二个参数）提交了一个名为 `image.png`（第三个参数）的文件。

服务器读取表单数据和文件，就好像它是常规的表单提交一样。

### 三、`Fetch`：下载进度

`fetch` 方法允许去跟踪 **下载** 进度。

注意：到目前为止，`fetch` 方法无法跟踪 **上传** 进度。对于这个目的，请使用 [XMLHttpRequest](https://zh.javascript.info/xmlhttprequest)。

要跟踪下载进度，可以使用 `response.body` 属性。它是 `ReadableStream` —— 一个特殊的对象，它可以逐块（chunk）提供 body。在 [Streams API](https://streams.spec.whatwg.org/#rs-class) 规范中有对 `ReadableStream` 的详细描述。

与 `response.text()`，`response.json()` 和其他方法不同，`response.body` 给予了对进度读取的完全控制，可以随时计算下载了多少。

这是从 `response.body` 读取 response 的示例代码：

```javascript
// 代替 response.json() 以及其他方法
const reader = response.body.getReader();

// 在 body 下载时，一直为无限循环
while(true) {
  // 当最后一块下载完成时，done 值为 true
  // value 是块字节的 Uint8Array
  const {done, value} = await reader.read();

  if (done) {
    break;
  }

  console.log(`Received ${value.length} bytes`)
}
```

`await reader.read()` 调用的结果是一个具有两个属性的对象：

- **`done`** —— 当读取完成时为 `true`，否则为 `false`。
- **`value`** —— 字节的类型化数组：`Uint8Array`。

**注意：**

Streams API 还描述了如果使用 `for await..of` 循环异步迭代 `ReadableStream`，但是目前为止，它还未得到很好的支持（参见 [浏览器问题](https://github.com/whatwg/streams/issues/778#issuecomment-461341033)），所以使用了 `while` 循环。

我们在循环中接收响应块（response chunk），直到加载完成，也就是：直到 `done` 为 `true`。

要将进度打印出来，只需要将每个接收到的片段 `value` 的长度（length）加到 counter 即可。

这是获取响应，并在控制台中记录进度的完整工作示例，下面有更多说明：

```javascript
// Step 1：启动 fetch，并获得一个 reader
let response = await fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits?per_page=100');

const reader = response.body.getReader();

// Step 2：获得总长度（length）
const contentLength = +response.headers.get('Content-Length');

// Step 3：读取数据
let receivedLength = 0; // 当前接收到了这么多字节
let chunks = []; // 接收到的二进制块的数组（包括 body）
while(true) {
  const {done, value} = await reader.read();

  if (done) {
    break;
  }

  chunks.push(value);
  receivedLength += value.length;

  console.log(`Received ${receivedLength} of ${contentLength}`)
}

// Step 4：将块连接到单个 Uint8Array
let chunksAll = new Uint8Array(receivedLength); // (4.1)
let position = 0;
for(let chunk of chunks) {
  chunksAll.set(chunk, position); // (4.2)
  position += chunk.length;
}

// Step 5：解码成字符串
let result = new TextDecoder("utf-8").decode(chunksAll);

// 我们完成啦！
let commits = JSON.parse(result);
alert(commits[0].author.login);
```

让我们一步步解释下这个过程：

1. 像往常一样执行 `fetch`，但不是调用 `response.json()`，而是获得了一个流读取器（stream reader）`response.body.getReader()`。

   请注意，不能同时使用这两种方法来读取相同的响应。要么使用流读取器，要么使用 reponse 方法来获取结果。

2. 在读取数据之前，可以从 `Content-Length` header 中得到完整的响应长度。

   跨源请求中可能不存在这个 header（请参见 [Fetch：跨源请求](https://zh.javascript.info/fetch-crossorigin)），并且从技术上讲，服务器可以不设置它。但是通常情况下它都会在那里。

3. 调用 `await reader.read()`，直到它完成。

   将响应块收集到数组 `chunks` 中。这很重要，因为在使用完（consumed）响应后，将无法使用 `response.json()` 或者其他方式（你可以试试，将会出现 error）去“重新读取”它。

4. 最后，有了一个 `chunks` —— 一个 `Uint8Array` 字节块数组。需要将这些块合并成一个结果。但不幸的是，没有单个方法可以将它们串联起来，所以这里需要一些代码来实现：

   1. 创建 `chunksAll = new Uint8Array(receivedLength)` —— 一个具有所有数据块合并后的长度的同类型数组。
   2. 然后使用 `.set(chunk, position)` 方法，从数组中一个个地复制这些 `chunk`。

5. 结果现在储存在 `chunksAll` 中。但它是一个字节数组，不是字符串。

   要创建一个字符串，需要解析这些字节。可以使用内建的 [TextDecoder](https://zh.javascript.info/text-decoder) 对象完成。然后，可以 `JSON.parse` 它，如果有必要的话。

   如果需要的是二进制内容而不是字符串呢？这更简单。用下面这行代码替换掉第 4 和第 5 步，这行代码从所有块创建一个 `Blob`：

   ```javascript
   let blob = new Blob(chunks);
   ```

最后得到了结果（以字符串或 blob 的形式表示，什么方便就用什么），并在过程中对进度进行了跟踪。这不能用于 **上传** 过程（现在无法通过 `fetch` 获取），仅用于 **下载** 过程。

### 四、`Fetch`：中止`(Abort)`

正如我们所知道的，`fetch` 返回一个 promise。JavaScript 通常并没有“中止” promise 的概念。那么怎样才能取消一个正在执行的 `fetch` 呢？例如，如果用户在网站上的操作表明不再需要 `fetch`。

为此有一个特殊的内建对象：`AbortController`。它不仅可以中止 `fetch`，还可以中止其他异步任务。用法非常简单。

#### 4.1 `AbortController`对象

创建一个控制器（controller）：

```javascript
let controller = new AbortController();
```

控制器是一个极其简单的对象。

- 它具有单个方法 `abort()`，
- 和单个属性 `signal`，可以在这个属性上设置事件监听器。

当 `abort()` 被调用时：

- `controller.signal` 就会触发 `abort` 事件。
- `controller.signal.aborted` 属性变为 `true`。

通常，处理分为两部分：

1. 一部分是一个可取消的操作，它在 `controller.signal` 上设置一个监听器。
2. 另一部分是取消：在需要的时候调用 `controller.abort()`。

这是完整的示例（目前还没有 `fetch`）：

```javascript
let controller = new AbortController();
let signal = controller.signal;

// 可取消的操作这一部分
// 获取 "signal" 对象，
// 并将监听器设置为在 controller.abort() 被调用时触发
signal.addEventListener('abort', () => alert("abort!"));

// 另一部分，取消（在之后的任何时候）：
controller.abort(); // 中止！

// 事件触发，signal.aborted 变为 true
alert(signal.aborted); // true
```

正如我们所看到的，`AbortController` 只是在 `abort()` 被调用时传递 `abort` 事件的一种方式。

可以自己在代码中实现相同类型的事件监听，而根本不需要 `AbortController` 对象。但是有价值的是，`fetch` 知道如何与 `AbortController` 对象一起工作，它们俩是集成在一起的。

#### 4.2 与`fetch`一起使用

为了能够取消 `fetch`，将 `AbortController` 的 `signal` 属性作为 `fetch` 的一个可选参数（option）进行传递：

```javascript
let controller = new AbortController();
fetch(url, {
  signal: controller.signal
});
```

`fetch` 方法知道如何与 `AbortController` 一起工作。它会监听 `signal` 上的 `abort` 事件。

现在，想要中止 `fetch`，调用 `controller.abort()` 即可：

```javascript
controller.abort();
```

现在`fetch` 从 `signal` 获取了事件并中止了请求。

当一个 fetch 被中止，它的 promise 就会以一个 error `AbortError` reject，因此应该对其进行处理，例如在 `try..catch` 中。

这是完整的示例，其中 `fetch` 在 1 秒后中止：

```javascript
// 1 秒后中止
let controller = new AbortController();
setTimeout(() => controller.abort(), 1000);

try {
  let response = await fetch('/article/fetch-abort/demo/hang', {
    signal: controller.signal
  });
} catch(err) {
  if (err.name == 'AbortError') { // handle abort()
    alert("Aborted!");
  } else {
    throw err;
  }
}
```

#### 4.3 `AbortController`是可伸缩的

`AbortController` 是可伸缩的，它允许一次取消多个 fetch。

这是一个代码草稿，该代码并行 fetch 很多 `urls`，并使用单个控制器将其全部中止：

```javascript
let urls = [...]; // 要并行 fetch 的 url 列表

let controller = new AbortController();

// 一个 fetch promise 的数组
let fetchJobs = urls.map(url => fetch(url, {
  signal: controller.signal
}));

let results = await Promise.all(fetchJobs);

// 如果 controller.abort() 被从其他地方调用，
// 它将中止所有 fetch
```

如果有自己的与 `fetch` 不同的异步任务，可以使用单个 `AbortController` 中止这些任务以及 fetch。

在我们的任务中，只需要监听其 `abort` 事件：

```javascript
let urls = [...];
let controller = new AbortController();

let ourJob = new Promise((resolve, reject) => { // 我们的任务
  ...
  controller.signal.addEventListener('abort', reject);
});

let fetchJobs = urls.map(url => fetch(url, { // fetches
  signal: controller.signal
}));

// 等待完成我们的任务和所有 fetch
let results = await Promise.all([...fetchJobs, ourJob]);

// 如果 controller.abort() 被从其他地方调用，
// 它将中止所有 fetch 和 ourJob
```

### 五、跨源请求

如果向另一个网站发送 `fetch` 请求，则该请求可能会失败。

例如，尝试向 `http://example.com` 发送 `fetch` 请求：

```javascript
try {
  await fetch('http://example.com');
} catch(err) {
  alert(err); // fetch 失败
}
```

正如所料，获取失败。

这里的核心概念是 **源（origin）**—— 域（domain）/端口（port）/协议（protocol）的组合。

跨源请求 —— 那些发送到其他域（即使是子域）、协议或端口的请求 —— 需要来自远程端的特殊 header。

这个策略被称为 “CORS”：跨源资源共享（Cross-Origin Resource Sharing）。

#### 5.1 跨源请求简史

CORS 的存在是为了保护互联网免受黑客攻击。

**多年来，来自一个网站的脚本无法访问另一个网站的内容。**

这个简单有力的规则是互联网安全的基础。例如，来自 `hacker.com` 的脚本无法访问 `gmail.com` 上的用户邮箱。基于这样的规则，人们感到很安全。

在那时候，JavaScript 并没有任何特殊的执行网络请求的方法。它只是一种用来装饰网页的玩具语言而已。

但是 Web 开发人员需要更多功能。人们发明了各种各样的技巧去突破该限制，并向其他网站发出请求。

**使用表单**

其中一种和其他服务器通信的方法是在那里提交一个 `<form>`。人们将它提交到 `<iframe>`，只是为了停留在当前页面，像这样：

```html
<!-- 表单目标 -->
<iframe name="iframe"></iframe>

<!-- 表单可以由 JavaScript 动态生成并提交 -->
<form target="iframe" method="POST" action="http://another.com/…">
  ...
</form>
```

因此，即使没有网络方法，也可以向其他网站发出 GET/POST 请求，因为表单可以将数据发送到任何地方。但是由于禁止从其他网站访问 `<iframe>` 中的内容，因此就无法读取响应。

确切地说，实际上有一些技巧能够解决这个问题，这在 iframe 和页面中都需要添加特殊脚本。因此，与 iframe 的通信在技术上是可能的。

**使用`script`**

另一个技巧是使用 `script` 标签。`script` 可以具有任何域的 `src`，例如 `<script src="http://another.com/…">`。也可以执行来自任何网站的 `script`。

如果一个网站，例如 `another.com` 试图公开这种访问方式的数据，则会使用所谓的 “JSONP (JSON with padding)” 协议。

这是它的工作方式。

假设在我们的网站，需要以这种方式从 `http://another.com` 网站获取数据，例如天气：

1. 首先，先声明一个全局函数来接收数据，例如 `gotWeather`。

   ```javascript
   // 1. 声明处理天气数据的函数
   function gotWeather({ temperature, humidity }) {
     alert(`temperature: ${temperature}, humidity: ${humidity}`);
   }
   ```

2. 然后创建一个特性（attribute）为 `src="http://another.com/weather.json?callback=gotWeather"` 的 `<script>` 标签，使用函数名作为它的 `callback` URL-参数。

   ```javascript
   let script = document.createElement('script');
   script.src = `http://another.com/weather.json?callback=gotWeather`;
   document.body.append(script);
   ```

3. 远程服务器 `another.com` 动态生成一个脚本，该脚本调用 `gotWeather(...)`，发送它想让我们接收的数据。

   ```javascript
   // 我们期望来自服务器的回答看起来像这样：
   gotWeather({
     temperature: 25,
     humidity: 78
   });
   ```

4. 当远程脚本加载并执行时，`gotWeather` 函数将运行，并且因为它是我们的函数，我们就有了需要的数据。

这是可行的，并且不违反安全规定，因为双方都同意以这种方式传递数据。而且，既然双方都同意这种行为，那这肯定不是黑客攻击了。现在仍然有提供这种访问的服务，因为即使是非常旧的浏览器它依然适用。

不久之后，网络方法出现在了浏览器 JavaScript 中。

起初，跨源请求是被禁止的。但是，经过长时间的讨论，跨源请求被允许了，但是任何新功能都需要服务器明确允许，以特殊的 header 表述。

#### 5.2 简单的请求

有两种类型的跨源请求：

1. 简单的请求。
2. 所有其他请求。

顾名思义，简单的请求很简单，所以先从它开始。

一个 [简单的请求](http://www.w3.org/TR/cors/#terminology) 是指满足以下两个条件的请求：

1. [简单的方法](http://www.w3.org/TR/cors/#simple-method)：GET，POST 或 HEAD
2. 简单的 header—— 仅允许自定义下列 header：
   - `Accept`，
   - `Accept-Language`，
   - `Content-Language`，
   - `Content-Type` 的值为 `application/x-www-form-urlencoded`，`multipart/form-data` 或 `text/plain`。

任何其他请求都被认为是“非简单请求”。例如，具有 `PUT` 方法或 `API-Key` HTTP-header 的请求就不是简单请求。

**本质区别在于，可以使用 `<form>` 或 `<script>` 进行“简单请求”，而无需任何其他特殊方法。**

因此，即使是非常旧的服务器也能很好地接收简单请求。

与此相反，带有非标准 header 或者例如 `DELETE` 方法的请求，无法通过这种方式创建。在很长一段时间里，JavaScript 都不能进行这样的请求。所以，旧的服务器可能会认为此类请求来自具有特权的来源（privileged source），“因为网页无法发送它们”。

当尝试发送一个非简单请求时，浏览器会发送一个特殊的“预检（preflight）”请求到服务器 —— 询问服务器，你接受此类跨源请求吗？

并且，除非服务器明确通过 header 进行确认，否则非简单请求不会被发送。

#### 5.3 用于简单请求的`CORS`

如果一个请求是跨源的，浏览器始终会向其添加 `Origin` header。

例如，如果从 `https://javascript.info/page` 请求 `https://anywhere.com/request`，请求的 header 将会如下：

```http
GET /request
Host: anywhere.com
Origin: https://javascript.info
...
```

`Origin` 包含了确切的源（domain/protocol/port），没有路径。

服务器可以检查 `Origin`，如果同意接受这样的请求，就会在响应中添加一个特殊的 header `Access-Control-Allow-Origin`。该 header 包含了允许的源（在示例中是 `https://javascript.info`），或者一个星号 `*`。然后响应成功，否则报错。

浏览器在这里扮演受被信任的中间人的角色：

1. 它确保发送的跨源请求带有正确的 `Origin`。

2. 它检查响应中的许可 `Access-Control-Allow-Origin`，如果存在，则允许 JavaScript 访问响应，否则将失败并报错。

   ![跨源请求过程](https://gitee.com/Topcvan//js-notes-img/raw/master/%E8%B7%A8%E6%BA%90%E8%AF%B7%E6%B1%82%E8%BF%87%E7%A8%8B.png)

这是一个带有服务器许可的响应示例：

```http
200 OK
Content-Type:text/html; charset=UTF-8
Access-Control-Allow-Origin: https://javascript.info
```

#### 5.4 `Response header`

对于跨源请求，默认情况下，JavaScript 只能访问“简单” response header：

- `Cache-Control`
- `Content-Language`
- `Content-Type`
- `Expires`
- `Last-Modified`
- `Pragma`

访问任何其他 response header 都将导致 error。

**注意：**

列表中没有 `Content-Length` header！

该 header 包含完整的响应长度。因此，如果正在下载某些内容，并希望跟踪进度百分比，则需要额外的权限才能访问该 header。

要授予 JavaScript 对任何其他 response header 的访问权限，服务器必须发送 `Access-Control-Expose-Headers` header。它包含一个以逗号分隔的应该被设置为可访问的非简单 header 名称列表。

例如：

```http
200 OK
Content-Type:text/html; charset=UTF-8
Content-Length: 12345
API-Key: 2c9de507f2c54aa1
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Expose-Headers: Content-Length,API-Key
```

有了这种 `Access-Control-Expose-Headers` header，此脚本就被允许读取响应的 `Content-Length` 和 `API-Key` header。

#### 5.5 “非简单”请求

我们可以使用任何 HTTP 方法：不仅仅是 `GET/POST`，也可以是 `PATCH`，`DELETE` 及其他。

之前，没有人能够设想网页能发出这样的请求。因此，可能仍然存在有些 Web 服务将非标准方法视为一个信号：“这不是浏览器”。它们可以在检查访问权限时将其考虑在内。

因此，为了避免误解，任何“非标准”请求 —— 浏览器不会立即发出在过去无法完成的这类请求。即在它发送这类请求前，会先发送“预检（preflight）”请求来请求许可。

预检请求使用 `OPTIONS` 方法，它没有 body，但是有两个 header：

- `Access-Control-Request-Method` header 带有非简单请求的方法。
- `Access-Control-Request-Headers` header 提供一个以逗号分隔的非简单 HTTP-header 列表。

如果服务器同意处理请求，那么它会进行响应，此响应的状态码应该为 200，没有 body，具有 header：

- `Access-Control-Allow-Origin` 必须为 `*` 或进行请求的源（例如 `https://javascript.info`）才能允许此请求。

- `Access-Control-Allow-Methods` 必须具有允许的方法。

- `Access-Control-Allow-Headers` 必须具有一个允许的 header 列表。

- 另外，header `Access-Control-Max-Age` 可以指定缓存此权限的秒数。因此，浏览器不是必须为满足给定权限的后续请求发送预检。

  ![非简单请求响应过程](https://gitee.com/Topcvan//js-notes-img/raw/master/%E9%9D%9E%E7%AE%80%E5%8D%95%E8%AF%B7%E6%B1%82%E5%93%8D%E5%BA%94%E8%BF%87%E7%A8%8B.png)

用一个例子来一步步看一下它是怎么工作的，对于一个跨源的 `PATCH` 请求（此方法经常被用于更新数据）：

```javascript
let response = await fetch('https://site.com/service.json', {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
    'API-Key': 'secret'
  }
});
```

这里有三个理由解释为什么它不是一个简单请求（其实一个就够了）：

- 方法 `PATCH`
- `Content-Type` 不是这三个中之一：`application/x-www-form-urlencoded`，`multipart/form-data`，`text/plain`。
- “非简单” `API-Key` header。

**Step 1 预检请求（preflight request）**

在发送请求前，浏览器会自己发送如下所示的预检请求：

```http
OPTIONS /service.json
Host: site.com
Origin: https://javascript.info
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: Content-Type,API-Key
```

- 方法：`OPTIONS`。
- 路径 —— 与主请求完全相同：`/service.json`。
- 特殊跨源头：
  - `Origin` —— 来源。
  - `Access-Control-Request-Method` —— 请求方法。
  - `Access-Control-Request-Headers` —— 以逗号分隔的“非简单” header 列表。

**Step 2 预检响应（preflight response）**

服务应响应状态 200 和 header：

- `Access-Control-Allow-Origin: https://javascript.info`
- `Access-Control-Allow-Methods: PATCH`
- `Access-Control-Allow-Headers: Content-Type,API-Key`。

这将允许后续通信，否则会触发错误。

如果服务器将来期望其他方法和 header，则可以通过将这些方法和 header 添加到列表中来预先允许它们。

例如，此响应还允许 `PUT`、`DELETE` 以及其他 header：

```http
200 OK
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Allow-Methods: PUT,PATCH,DELETE
Access-Control-Allow-Headers: API-Key,Content-Type,If-Modified-Since,Cache-Control
Access-Control-Max-Age: 86400
```

现在，浏览器可以看到 `PATCH` 在 `Access-Control-Allow-Methods` 中，`Content-Type,API-Key` 在列表 `Access-Control-Allow-Headers` 中，因此它将发送主请求。

如果 `Access-Control-Max-Age` 带有一个表示秒的数字，则在给定的时间内，预检权限会被缓存。上面的响应将被缓存 86400 秒，也就是一天。在此时间范围内，后续请求将不会触发预检。假设它们符合缓存的配额，则将直接发送它们。

**Step 3 实际请求（actual request）**

预检成功后，浏览器现在发出主请求。这里的算法与简单请求的算法相同。

主请求具有 `Origin` header（因为它是跨源的）：

```http
PATCH /service.json
Host: site.com
Content-Type: application/json
API-Key: secret
Origin: https://javascript.info
```

**Step 4 实际响应（actual response）**

服务器不应该忘记在主响应中添加 `Access-Control-Allow-Origin`。成功的预检并不能免除此要求：

```http
Access-Control-Allow-Origin: https://javascript.info
```

然后，JavaScript 可以读取主服务器响应了。

**注意：**

预检请求发生在“幕后”，它对 JavaScript 不可见。

JavaScript 仅获取对主请求的响应，如果没有服务器许可，则获得一个 error。

#### 5.6 凭据（Credentials）

默认情况下，由 JavaScript 代码发起的跨源请求不会带来任何凭据（cookies 或者 HTTP 认证（HTTP authentication））。

这对于 HTTP 请求来说并不常见。通常，对 `http://site.com` 的请求附带有该域的所有 cookie。但是由 JavaScript 方法发出的跨源请求是个例外。

例如，`fetch('http://another.com')` 不会发送任何 cookie，即使那些 (!) 属于 `another.com` 域的 cookie。

为什么？

这是因为具有凭据的请求比没有凭据的请求要强大得多。如果被允许，它会使用它们的凭据授予 JavaScript 代表用户行为和访问敏感信息的全部权力。

服务器真的这么信任这种脚本吗？是的，它必须显式地带有允许请求的凭据和附加 header。

要在 `fetch` 中发送凭据，需要添加 `credentials: "include"` 选项，像这样：

```javascript
fetch('http://another.com', {
  credentials: "include"
});
```

现在，`fetch` 将把源自 `another.com` 的 cookie 和请求发送到该网站。

如果服务器同意接受 **带有凭据** 的请求，则除了 `Access-Control-Allow-Origin` 外，服务器还应该在响应中添加 header `Access-Control-Allow-Credentials: true`。

例如：

```http
200 OK
Access-Control-Allow-Origin: https://javascript.info
Access-Control-Allow-Credentials: true
```

注意：对于具有凭据的请求，禁止 `Access-Control-Allow-Origin` 使用星号 `*`。如上所示，它必须有一个确切的源。这是另一项安全措施，以确保服务器真的知道它信任的发出此请求的是谁。

### 六、`Fetch API`

这是所有可能的 `fetch` 选项及其默认值（注释中标注了可选值）的完整列表：

```javascript
let promise = fetch(url, {
  method: "GET", // POST，PUT，DELETE，等。
  headers: {
    // 内容类型 header 值通常是自动设置的
    // 取决于 request body
    "Content-Type": "text/plain;charset=UTF-8"
  },
  body: undefined // string，FormData，Blob，BufferSource，或 URLSearchParams
  referrer: "about:client", // 或 "" 以不发送 Referer header，
  // 或者是当前源的 url
  referrerPolicy: "no-referrer-when-downgrade", // no-referrer，origin，same-origin...
  mode: "cors", // same-origin，no-cors
  credentials: "same-origin", // omit，include
  cache: "default", // no-store，reload，no-cache，force-cache，或 only-if-cached
  redirect: "follow", // manual，error
  integrity: "", // 一个 hash，像 "sha256-abcdef1234567890"
  keepalive: false, // true
  signal: undefined, // AbortController 来中止请求
  window: window // null
});
```

#### 6.1 referrer，referrerPolicy

这些选项决定了 `fetch` 如何设置 HTTP 的 `Referer` header。

通常来说，这个 header 是被自动设置的，并包含了发出请求的页面的 url。在大多数情况下，它一点也不重要，但有时出于安全考虑，删除或缩短它是有意义的。

**`referer` 选项允许设置在当前域的任何 `Referer`，或者移除它。**

要不发送 referer，可以将 `referer` 设置为空字符串：

```javascript
fetch('/page', {
  referrer: "" // 没有 Referer header
});
```

设置在当前域内的另一个 url：

```javascript
fetch('/page', {
  // 假设我们在 https://javascript.info
  // 我们可以设置任何 Referer header，但必须是在当前域内的
  referrer: "https://javascript.info/anotherpage"
});
```

**`referrerPolicy` 选项为 `Referer` 设置一般的规则。**

请求分为 3 种类型：

1. 同源请求。
2. 跨源请求。
3. 从 HTTPS 到 HTTP 的请求 (从安全协议到不安全协议)。

与 `referrer` 选项允许设置确切的 `Referer` 值不同，`referrerPolicy` 告诉浏览器针对各个请求类型的一般的规则。

可能的值在 [Referrer Policy 规范](https://w3c.github.io/webappsec-referrer-policy/)中有详细描述：

- **`"no-referrer-when-downgrade"`** —— 默认值：除非从 HTTPS 发送请求到 HTTP（到安全性较低的协议），否则始终会发送完整的 `Referer`。
- **`"no-referrer"`** —— 从不发送 `Referer`。
- **`"origin"`** —— 只发送在 `Referer` 中的域，而不是完整的页面 URL，例如，只发送 `http://site.com` 而不是 `http://site.com/path`。
- **`"origin-when-cross-origin"`** —— 发送完整的 `Referer` 到相同的源，但对于跨源请求，只发送域部分（同上）。
- **`"same-origin"`** —— 发送完整的 `Referer` 到相同的源，但对于跨源请求，不发送 `Referer`。
- **`"strict-origin"`** —— 只发送域，对于 HTTPS→HTTP 请求，则不发送 `Referer`。
- **`"strict-origin-when-cross-origin"`** —— 对于同源情况下则发送完整的 `Referer`，对于跨源情况下，则只发送域，如果是 HTTPS→HTTP 请求，则什么都不发送。
- **`"unsafe-url"`** —— 在 `Referer` 中始终发送完整的 url，即使是 HTTPS→HTTP 请求。

这是一个包含所有组合的表格：

| 值                                             | 同源       | 跨源       | HTTPS→HTTP |
| :--------------------------------------------- | :--------- | :--------- | :--------- |
| `"no-referrer"`                                | -          | -          | -          |
| `"no-referrer-when-downgrade"` 或 `""`（默认） | 完整的 url | 完整的 url | -          |
| `"origin"`                                     | 仅域       | 仅域       | 仅域       |
| `"origin-when-cross-origin"`                   | 完整的 url | 仅域       | 仅域       |
| `"same-origin"`                                | 完整的 url | -          | -          |
| `"strict-origin"`                              | 仅域       | 仅域       | -          |
| `"strict-origin-when-cross-origin"`            | 完整的 url | 仅域       | -          |
| `"unsafe-url"`                                 | 完整的 url | 完整的 url | 完整的 url |

假如有一个带有 URL 结构的管理区域（admin zone），它不应该被从网站外看到。

如果发送了一个 `fetch`，则默认情况下，它总是发送带有页面完整 url 的 `Referer` header（从 HTTPS 向 HTTP 发送请求的情况除外，这种情况下没有 `Referer`）。

例如 `Referer: https://javascript.info/admin/secret/paths`。

如果想让其他网站只知道域的部分，而不是 URL 路径，可以这样设置选项：

```javascript
fetch('https://another.com/page', {
  // ...
  referrerPolicy: "origin-when-cross-origin" // Referer: https://javascript.info
});
```

可以将其置于所有 `fetch` 调用中，也可以将其集成到项目的执行所有请求并在内部使用 `fetch` 的 JavaScript 库中。

与默认行为相比，它的唯一区别在于，对于跨源请求，`fetch` 只发送 URL 域的部分（例如 `https://javascript.info`，没有路径）。对于同源请求，仍然可以获得完整的 `Referer`（可能对于调试目的是有用的）。

**Referrer policy 不仅适用于 `fetch`**

在 [规范](https://w3c.github.io/webappsec-referrer-policy/) 中描述的 referrer policy，不仅适用于 `fetch`，它还具有全局性。

特别是，可以使用 `Referrer-Policy` HTTP header，或者为每个链接设置 `<a rel="noreferrer">`，来为整个页面设置默认策略（policy）。

#### 6.2 `mode`

`mode` 选项是一种安全措施，可以防止偶发的跨源请求：

- **`"cors"`** —— 默认值，允许跨源请求，
- **`"same-origin"`** —— 禁止跨源请求，
- **`"no-cors"`** —— 只允许简单的跨源请求。

当 `fetch` 的 URL 来自于第三方，并且想要一个“断电开关”来限制跨源能力时，此选项可能很有用。

#### 6.3 credentials

`credentials` 选项指定 `fetch` 是否应该随请求发送 cookie 和 HTTP-Authorization header。

- **`"same-origin"`** —— 默认值，对于跨源请求不发送，
- **`"include"`** —— 总是发送，需要来自跨源服务器的 `Accept-Control-Allow-Credentials`，才能使 JavaScript 能够访问响应，详细内容在 [Fetch：跨源请求](https://zh.javascript.info/fetch-crossorigin) 一章有详细介绍，
- **`"omit"`** —— 不发送，即使对于同源请求。

#### 6.4 cache

默认情况下，`fetch` 请求使用标准的 HTTP 缓存。就是说，它遵从 `Expires`，`Cache-Control` header，发送 `If-Modified-Since`，等。就像常规的 HTTP 请求那样。

使用 `cache` 选项可以忽略 HTTP 缓存或者对其用法进行微调：

- **`"default"`** —— `fetch` 使用标准的 HTTP 缓存规则和 header，
- **`"no-store"`** —— 完全忽略 HTTP 缓存，如果我们设置 header `If-Modified-Since`，`If-None-Match`，`If-Unmodified-Since`，`If-Match`，或 `If-Range`，则此模式会成为默认模式，
- **`"reload"`** —— 不从 HTTP 缓存中获取结果（如果有），而是使用响应填充缓存（如果 response header 允许），
- **`"no-cache"`** —— 如果有一个已缓存的响应，则创建一个有条件的请求，否则创建一个普通的请求。使用响应填充 HTTP 缓存，
- **`"force-cache"`** —— 使用来自 HTTP 缓存的响应，即使该响应已过时（stale）。如果 HTTP 缓存中没有响应，则创建一个常规的 HTTP 请求，行为像正常那样，
- **`"only-if-cached"`** —— 使用来自 HTTP 缓存的响应，即使该响应已过时（stale）。如果 HTTP 缓存中没有响应，则报错。只有当 `mode` 为 `same-origin` 时生效。

#### 6.5 redirect

通常来说，`fetch` 透明地遵循 HTTP 重定向，例如 301，302 等。

`redirect` 选项允许对此进行更改：

- **`"follow"`** —— 默认值，遵循 HTTP 重定向，
- **`"error"`** —— HTTP 重定向时报错，
- **`"manual"`** —— 不遵循 HTTP 重定向，但 `response.url` 将是一个新的 URL，并且 `response redirectd` 将为 `true`，以便能够手动执行重定向到新的 URL（如果需要的话）。

#### 6.6 integrity

`integrity` 选项允许检查响应是否与已知的预先校验和相匹配。

正如 [规范](https://w3c.github.io/webappsec-subresource-integrity/) 所描述的，支持的哈希函数有 SHA-256，SHA-384，和 SHA-512，可能还有其他的，这取决于浏览器。

例如，下载一个文件，并且知道它的 SHA-256 校验和为 “abcdef”（当然，实际校验和会更长）。

可以将其放在 `integrity` 选项中，就像这样:

```javascript
fetch('http://site.com/file', {
  integrity: 'sha256-abcdef'
});
```

然后 `fetch` 将自行计算 SHA-256 并将其与我们的字符串进行比较。如果不匹配，则会触发错误。

#### 6.7 keepalive

`keepalive` 选项表示该请求可能会在网页关闭后继续存在。

例如，收集有关当前访问者是如何使用我们的页面（鼠标点击，他查看的页面片段）的统计信息，以分析和改善用户体验。

当访问者离开网页时 —— 我们希望能够将数据保存到服务器上。

可以使用 `window.onunload` 事件来实现：

```javascript
window.onunload = function() {
  fetch('/analytics', {
    method: 'POST',
    body: "statistics",
    keepalive: true
  });
};
```

通常，当一个文档被卸载时（unloaded），所有相关的网络请求都会被中止。但是，`keepalive` 选项告诉浏览器，即使在离开页面后，也要在后台执行请求。所以，此选项对于我们的请求成功至关重要。

它有一些限制：

- 无法发送兆字节的数据：`keepalive`请求的 body 限制为 64kb。

  - 如果需要收集有关访问的大量统计信息，则应该将其定期以数据包的形式发送出去，这样就不会留下太多数据给最后的 `onunload` 请求了。
  - 此限制是被应用于当前所有 `keepalive` 请求的总和的。换句话说，可以并行执行多个 `keepalive` 请求，但它们的 body 长度之和不得超过 64KB。

- 如果文档（document）已卸载（unloaded），就无法处理服务器响应。因此，在示例中，因为`keepalive`，所以`fetch`会成功，但是后续的函数将无法正常工作。

  - 在大多数情况下，例如发送统计信息，这不是问题，因为服务器只接收数据，并通常向此类请求发送空的响应。