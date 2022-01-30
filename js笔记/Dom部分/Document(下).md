### 一、 特性和属性

对于元素节点，大多数标准的 HTML 特性（attributes）会自动变成 DOM 对象的属性（properties）。（attribute 和 property 两词意思相近，为作区分，将 attribute 称为“特性”，property 称为“属性”。）

例如，如果标签是 `<body id="page">`，那么 DOM 对象就会有 `body.id="page"`。但特性—属性映射并不是一一对应的

#### 1.1 `DOM`属性

DOM 节点是常规的 JavaScript 对象。可以像普通对象一样对它们使用`alert`方法等。

例如，在 `document.body` 中创建一个新的属性：

```javascript
document.body.myData = {
  name: 'Caesar',
  title: 'Imperator'
};

alert(document.body.myData.title); // Imperator
```

也可以像下面这样添加一个方法：

```javascript
document.body.sayTagName = function() {
  alert(this.tagName);
};

document.body.sayTagName(); // BODY（这个方法中的 "this" 的值是 document.body）
```

还可以修改内建属性的原型，例如修改 `Element.prototype` 为所有元素添加一个新方法：

```javascript
Element.prototype.sayHi = function() {
  alert(`Hello, I'm ${this.tagName}`);
};

document.documentElement.sayHi(); // Hello, I'm HTML
document.body.sayHi(); // Hello, I'm BODY
```

所以，DOM 属性和方法的行为就像常规的 Javascript 对象一样：

- 它们可以有很多值。
- 它们是大小写敏感的（要写成 `elem.nodeType`，而不是 `elem.NoDeTyPe`）。

#### 1.2 `HTML`特性

在 HTML 中，标签可能拥有特性（attributes）。当浏览器解析 HTML 文本，并根据标签创建 DOM 对象时，浏览器会辨别 **标准的** 特性并以此创建 DOM 属性。所以，当一个元素有 `id` 或其他 **标准的** 特性，那么就会生成对应的 DOM 属性。但是非 **标准的** 特性则不会。

例如：

```html
<body id="test" something="non-standard">
  <script>
    alert(document.body.id); // test
    // 非标准的特性没有获得对应的属性
    alert(document.body.something); // undefined
  </script>
</body>
```

一个元素的标准的特性对于另一个元素可能是未知的。例如 `"type"` 是 `<input>` 的一个标准的特性（[HTMLInputElement](https://html.spec.whatwg.org/#htmlinputelement)），但对于 `<body>`（[HTMLBodyElement](https://html.spec.whatwg.org/#htmlbodyelement)）来说则不是。所以，如果一个特性不是标准的，那么就没有相对应的 DOM 属性。

当然。所有特性都可以通过使用以下方法进行访问：

- `elem.hasAttribute(name)` — 检查特性是否存在。
- `elem.getAttribute(name)` — 获取这个特性值。
- `elem.setAttribute(name, value)` — 设置这个特性值。
- `elem.removeAttribute(name)` — 移除这个特性。

这些方法操作的实际上是 HTML 中的内容。除此之外也可以使用 `elem.attributes` 读取所有特性：属于内建 [Attr](https://dom.spec.whatwg.org/#attr) 类的对象的集合，具有 `name` 和 `value` 属性。

下面是一个读取非标准的特性的示例：

```html
<body something="non-standard">
  <script>
    alert(document.body.getAttribute('something')); // 非标准的
  </script>
</body>
```

HTML 特性有以下几个特征：

- 它们的名字是大小写不敏感的（`id` 与 `ID` 相同）。
- 它们的值总是字符串类型的。

下面是一个使用特性的扩展示例：

```markup
<body>
  <div id="elem" about="Elephant"></div>

  <script>
    alert( elem.getAttribute('About') ); // (1) 'Elephant'，读取

    elem.setAttribute('Test', 123); // (2) 写入

    alert( elem.outerHTML ); // (3) 查看特性是否在 HTML 中（在）

    for (let attr of elem.attributes) { // (4) 列出所有
      alert( `${attr.name} = ${attr.value}` );
    }
  </script>
</body>
```

注意事项：

1. `getAttribute('About')` — 这里的第一个字母是大写的，但是在 HTML 中，它们都是小写的。但这没有影响：特性的名称是大小写不敏感的。
2. 可以将任何东西赋值给特性，但是这些东西会变成字符串类型的。所以这里的值为 `"123"`。
3. 所有特性，包括设置的那个特性，在 `outerHTML` 中都是可见的。
4. `attributes` 集合是可迭代对象，该对象将所有元素的特性（标准和非标准的）作为 `name` 和 `value` 属性存储在对象中。

#### 1.3 属性—特性同步

当一个标准的特性被改变，对应的属性也会自动更新，反之亦然（除了几个特例）。例如 `input.value` 只能从特性同步到属性，反过来则不行：

```html
<input>

<script>
  let input = document.querySelector('input');

  // 特性 => 属性
  input.setAttribute('value', 'text');
  alert(input.value); // text

  // 这个操作无效，属性 => 特性
  input.value = 'newValue';
  alert(input.getAttribute('value')); // text（没有被更新！）
</script>
```

在上面这个例子中：

- 改变特性值 `value` 会更新属性，`input`输入框中的值会发生改变。
- 但是属性的更改不会影响特性，但`input`输入框中的值还是会发生改变。

这个“功能”在实际中会派上用场，因为用户行为可能会导致 `value` 的更改，然后在这些操作之后，如果想从 HTML 中恢复“原始”值，那么该值就在特性中。

#### 1.4 `DOM`类型属性

DOM 属性不总是字符串类型的。例如，`input.checked` 属性（对于 `checkbox` 的）是布尔型的，特性值是一个空字符串。`style` 特性是字符串类型的，但 `style` 属性是一个对象：

```html
<input id="input" type="checkbox" checked> checkbox

<script>
  alert(input.getAttribute('checked')); // 特性值是：空字符串
  alert(input.checked); // 属性值是：true
</script>

<div id="div" style="color:red;font-size:120%">Hello</div>

<script>
  // 字符串
  alert(div.getAttribute('style')); // color:red;font-size:120%

  // 对象
  alert(div.style); // [object CSSStyleDeclaration]
  alert(div.style.color); // red
</script>
```

尽管大多数 DOM 属性都是字符串类型的。有一种非常少见的情况，即使一个 DOM 属性是字符串类型的，但它可能和 HTML 特性也是不同的。例如，`href` DOM 属性一直是一个 **完整的** URL，即使该特性包含一个相对路径或者包含一个 `#hash`。

这里有一个例子：

```markup
<a id="a" href="#hello">link</a>
<script>
  // 特性
  alert(a.getAttribute('href')); // #hello

  // 属性
  alert(a.href ); // http://site.com/page#hello 形式的完整 URL
</script>
```

如果需要的是`href` 特性的值，或者其他与 HTML 中所写的完全相同的特性，则可以使用 `getAttribute`。

#### 1.5 非标准的特性，dataset

当编写 HTML 时，会用到很多标准的特性。但是也会有许多非标准的自定义特性。

有时，非标准的特性常常用于将自定义的数据从 HTML 传递到 JavaScript，或者用于为 JavaScript “标记” HTML 元素。像这样：

```html
<!-- 标记这个 div 以在这显示 "name" -->
<div show-info="name"></div>
<!-- 标记这个 div 以在这显示 "age" -->
<div show-info="age"></div>

<script>
  // 这段代码找到带有标记的元素，并显示需要的内容
  let user = {
    name: "Pete",
    age: 25
  };

  for(let div of document.querySelectorAll('[show-info]')) {
    // 在字段中插入相应的信息
    let field = div.getAttribute('show-info');
    div.innerHTML = user[field]; // 首先 "name" 变为 Pete，然后 "age" 变为 25
  }
</script>
```

它们还可以用来设置元素的样式。例如，这里使用 `order-state` 特性来设置订单状态：

```html
<style>
  /* 样式依赖于自定义特性 "order-state" */
  .order[order-state="new"] {
    color: green;
  }

  .order[order-state="pending"] {
    color: blue;
  }

  .order[order-state="canceled"] {
    color: red;
  }
</style>

<div class="order" order-state="new">
  A new order.
</div>

<div class="order" order-state="pending">
  A pending order.
</div>

<div class="order" order-state="canceled">
  A canceled order.
</div>
```

为什么使用特性比使用 `.order-state-new`，`.order-state-pending`，`.order-state-canceled` 这些样式类要好？因为特性值更容易管理，可以轻松地更改状态：

```javascript
// 比删除旧的或者添加一个新的类要简单一些
div.setAttribute('order-state', 'canceled');
```

但是自定义的特性也存在问题。如果出于个人的目的使用了非标准的特性，之后它被引入到了标准中并有了其自己的用途，该怎么办？HTML 语言是在不断发展的，并且更多的特性出现在了标准中，以满足开发者的需求。在这种情况下，自定义的属性可能会产生意料不到的影响。

为了避免冲突，存在 [data-*](https://html.spec.whatwg.org/#embedding-custom-non-visible-data-with-the-data-*-attributes) 特性。**所有以 “data-” 开头的特性均被保留供程序员使用。它们可在 `dataset` 属性中使用。**

例如，如果一个 `elem` 有一个名为 `"data-about"` 的特性，那么可以通过 `elem.dataset.about` 取到它：

```html
<body data-about="Elephants">
<script>
  alert(document.body.dataset.about); // Elephants
</script>
```

像 `data-order-state` 这样的多词特性可以以驼峰式进行调用：`dataset.orderState`。

这里是 “order state” 那个示例的重构版：

```html
<style>
  .order[data-order-state="new"] {
    color: green;
  }

  .order[data-order-state="pending"] {
    color: blue;
  }

  .order[data-order-state="canceled"] {
    color: red;
  }
</style>

<div id="order" class="order" data-order-state="new">
  A new order.
</div>

<script>
  // 读取
  alert(order.dataset.orderState); // new

  // 修改
  order.dataset.orderState = "pending"; // (*)
</script>
```

使用 `data-*` 特性是一种合法且安全的传递自定义数据的方式。数据不仅可以读取，还可以修改数据属性（data-attributes）。然后 `CSS` 会更新相应的视图：在上面这个例子中的最后一行 `(*)` 将颜色更改为了蓝色。

### 二、修改文档

#### 2.1 创建元素

要创建 DOM 节点，有两种方法：

- `document.createElement(tag)`

  用给定的标签创建一个新**元素节点（element node）**：`let div = document.createElement('div');`

- `document.createTextNode(text)`

  用给定的文本创建一个**文本节点**：`let textNode = document.createTextNode('Here I am');`

大多数情况下需要创建像 `div` 这样的元素节点。

#### 2.2 插入文档

为了让 `div` 显示出来，需要将其插入到 `document` 中的某处。例如，into `<body>` element, referenced by `document.body`.

对此有一个特殊的方法 `append`：`document.body.append(div)`。

完整代码如下：

```html
<style>
.alert {
  padding: 15px;
  border: 1px solid #d6e9c6;
  border-radius: 4px;
  color: #3c763d;
  background-color: #dff0d8;
}
</style>

<script>
  let div = document.createElement('div');
  div.className = "alert";
  div.innerHTML = "<strong>Hi there!</strong> You've read an important message.";

  document.body.append(div);
</script>
```

在这个例子中，对 `document.body` 调用了 `append` 方法。不过也可以在其他任何元素上调用 `append` 方法，以将另外一个元素放入到里面。例如，通过调用 `div.append(anotherElement)`，便可以在 `<div>` 末尾添加一些内容。

这里是更多的元素插入方法，指明了不同的插入位置：

- `node.append(...nodes or strings)` —— 在 `node` **末尾** 插入节点或字符串，
- `node.prepend(...nodes or strings)` —— 在 `node` **开头** 插入节点或字符串，
- `node.before(...nodes or strings)` —— 在 `node` **前面** 插入节点或字符串，
- `node.after(...nodes or strings)` —— 在 `node` **后面** 插入节点或字符串，
- `node.replaceWith(...nodes or strings)` —— 将 `node` 替换为给定的节点或字符串。

这些方法的参数可以是一个要插入的任意的 DOM 节点列表，或者文本字符串（会被自动转换成文本节点）。这些方法可以在单个调用中插入多个节点列表和文本片段。

例如，在这里插入了一个字符串和一个元素：

```html
<div id="div"></div>
<script>
  div.before('<p>Hello</p>', document.createElement('hr'));
</script>
```

但这里的文字都被“作为文本”插入，而不是“作为 HTML 代码”。因此像 `<`、`>` 这样的符号都会被作转义处理来保证正确显示。

所以，最终的 HTML 为：

```html
&lt;p&gt;Hello&lt;/p&gt;
<hr>
<div id="div"></div>
```

换句话说，字符串被以一种安全的方式插入到页面中，就像 `elem.textContent` 所做的一样。所以，这些方法只能用来插入 DOM 节点或文本片段。

#### 2.3 `insertAdjacentHTML/Text/Element`

如果想要将内容“作为 HTML 代码插入”，让内容中的所有标签和其他东西都像使用 `elem.innerHTML` 所表现的效果一样，可以使用另一个非常通用的方法：`elem.insertAdjacentHTML(where, html)`。

该方法的第一个参数是代码字（code word），指定相对于 `elem` 的插入位置。必须为以下之一：

- `"beforebegin"` — 将 `html` 插入到 `elem` 前插入，
- `"afterbegin"` — 将 `html` 插入到 `elem` 开头，
- `"beforeend"` — 将 `html` 插入到 `elem` 末尾，
- `"afterend"` — 将 `html` 插入到 `elem` 后。

第二个参数是 HTML 字符串，该字符串会被“作为 HTML” 插入。

例如：

```html
<div id="div"></div>
<script>
  div.insertAdjacentHTML('beforebegin', '<p>Hello</p>');
  div.insertAdjacentHTML('afterend', '<p>Bye</p>');
</script>
```

这样的效果是：

```html
<p>Hello</p>
<div id="div"></div>
<p>Bye</p>
```

这个方法有两个兄弟：

- `elem.insertAdjacentText(where, text)` — 语法一样，但是将 `text` 字符串“作为文本”插入而不是作为 HTML，
- `elem.insertAdjacentElement(where, elem)` — 语法一样，但是插入的是一个元素。

它们的存在主要是为了使语法“统一”。实际上，大多数时候只使用 `insertAdjacentHTML`。因为对于元素和文本，有 `append/prepend/before/after` 方法 — 它们也可以用于插入节点/文本片段，但写起来更短。

所以，下面是显示一条消息的一种变体：

```html
<style>
.alert {
  padding: 15px;
  border: 1px solid #d6e9c6;
  border-radius: 4px;
  color: #3c763d;
  background-color: #dff0d8;
}
</style>

<script>
  document.body.insertAdjacentHTML("afterbegin", `<div class="alert">
    <strong>Hi there!</strong> You've read an important message.
  </div>`);
</script>
```

#### 2.4 节点移除

想要移除一个节点，可以使用 `node.remove()`。让消息在一秒后消失：

```html
<style>
.alert {
  padding: 15px;
  border: 1px solid #d6e9c6;
  border-radius: 4px;
  color: #3c763d;
  background-color: #dff0d8;
}
</style>

<script>
  let div = document.createElement('div');
  div.className = "alert";
  div.innerHTML = "<strong>Hi there!</strong> You've read an important message.";

  document.body.append(div);
  setTimeout(() => div.remove(), 1000);
</script>
```

但如果要将一个元素 **移动** 到另一个地方，则无需将其从原来的位置中删除。**所有插入方法都会自动从旧位置删除该节点。**例如，进行元素交换：

```html
<div id="first">First</div>
<div id="second">Second</div>
<script>
  // 无需调用 remove
  second.after(first); // 获取 #second，并在其后面插入 #first
</script>
```

#### 2.5 克隆节点

如何再插入一条类似的消息？可以创建一个函数，并将代码放在其中。另一种方法是 **克隆** 现有的 `div`，并修改其中的文本（如果需要）。如果有一个很大的元素时，克隆的方式可能更快更简单。

调用 `elem.cloneNode(true)` 来创建元素的一个“深”克隆 — 具有所有特性（attribute）和子元素。如果调用 `elem.cloneNode(false)`，那克隆就不包括子元素。

一个拷贝消息的示例：

```html
<style>
.alert {
  padding: 15px;
  border: 1px solid #d6e9c6;
  border-radius: 4px;
  color: #3c763d;
  background-color: #dff0d8;
}
</style>

<div class="alert" id="div">
  <strong>Hi there!</strong> You've read an important message.
</div>

<script>
  let div2 = div.cloneNode(true); // 克隆消息
  div2.querySelector('strong').innerHTML = 'Bye there!'; // 修改克隆

  div.after(div2); // 在已有的 div 后显示克隆
</script>
```

#### 2.6 `DocumentFragment`

`DocumentFragment` 是一个特殊的 DOM 节点，用作来传递节点列表的包装器（wrapper）。可以向其附加其他节点，但是当将其插入某个位置时，则会插入其内容。

例如，下面这段代码中的 `getListContent` 会生成带有 `<li>` 列表项的片段，然后将其插入到 `<ul>` 中：

```html
<ul id="ul"></ul>

<script>
function getListContent() {
  let fragment = new DocumentFragment();

  for(let i=1; i<=3; i++) {
    let li = document.createElement('li');
    li.append(i);
    fragment.append(li);
  }

  return fragment;
}

ul.append(getListContent()); // (*)
</script>
```

在最后一行 `(*)` 附加了 `DocumentFragment`，但是它和 `ul` “融为一体（blends in）”了，所以最终的文档结构应该是：

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>
```

#### 2.7 老式的`insert/remove`方法

- `parentElem.appendChild(node)`

  将 `node` 附加为 `parentElem` 的最后一个子元素。下面这个示例在 `<ol>` 的末尾添加了一个新的 `<li>`：

  ```html
  <ol id="list">
    <li>0</li>
    <li>1</li>
    <li>2</li>
  </ol>
  
  <script>
    let newLi = document.createElement('li');
    newLi.innerHTML = 'Hello, world!';
  
    list.appendChild(newLi);
  </script>
  ```
  
- `parentElem.insertBefore(node, nextSibling)`

   在 `parentElem` 的 `nextSibling` 前插入 `node`。下面这段代码在第二个 `<li>` 前插入了一个新的列表项：

  ```html
  <ol id="list">
    <li>0</li>
    <li>1</li>
    <li>2</li>
  </ol>
  <script>
    let newLi = document.createElement('li');
    newLi.innerHTML = 'Hello, world!';
  
    list.insertBefore(newLi, list.children[1]);
  </script>
  ```

- `parentElem.replaceChild(node, oldChild)`

  将 `parentElem` 的后代中的 `oldChild` 替换为 `node`

- `parentElem.removeChild(node)`

  从 `parentElem` 中删除 `node`（假设 `node` 为 `parentElem` 的后代）。

所有这些方法都会返回插入/删除的节点。换句话说，`parentElem.appendChild(node)` 返回 `node`。

#### 2.8 `document.write`

还有一个非常古老的向网页添加内容的方法：`document.write`。

语法如下：

```html
<p>Somewhere in the page...</p>
<script>
  document.write('<b>Hello from JS</b>');
</script>
<p>The end</p>
```

调用 `document.write(html)` 意味着将 `html` “就地马上”写入页面。`html` 字符串可以是动态生成的，所以它很灵活。可以使用 JavaScript 创建一个完整的页面并对其进行写入。

这个方法来自于没有 DOM，没有标准的上古时期。但这个方法依被保留了下来，因为还有脚本在使用它。

由于以下重要的限制，在现代脚本中很少看到它：**`document.write` 调用只在页面加载时工作。**

如果稍后调用它，则现有文档内容将被擦除。例如：

```html
<p>After one second the contents of this page will be replaced...</p>
<script>
  // 1 秒后调用 document.write
  // 这时页面已经加载完成，所以它会擦除现有内容
  setTimeout(() => document.write('<b>...By this.</b>'), 1000);
</script>
```

因此，在某种程度上讲，它在“加载完成”阶段是不可用的，这与上面介绍的其他 DOM 方法不同。

这是它的缺陷。但还有一个好处。从技术上讲，当在浏览器正在读取（“解析”）传入的 HTML 时调用 `document.write` 方法来写入一些东西，浏览器会像它本来就在 HTML 文本中那样使用它。所以它运行起来出奇的快，因为它 **不涉及 DOM 修改**。它直接写入到页面文本中，而此时 DOM 尚未构建。

因此，如果需要向 HTML 动态地添加大量文本，并且正处于页面加载阶段，并且速度很重要，那么它可能会有帮助。但实际上，这些要求很少同时出现。

### 三、 样式和类

通常有两种设置元素样式的方式：

1. 在 CSS 中创建一个类，并添加它：`<div class="...">`
2. 将属性直接写入 `style`：`<div style="...">`。

JavaScript 既可以修改类，也可以修改 `style` 属性。相较于将样式写入 `style` 属性，应该首选通过 CSS 类的方式来添加样式。仅当类“无法处理”时，才应选择使用 `style` 属性的方式。

例如，如果想要动态地计算元素的坐标，并希望通过 JavaScript 来设置它们，那么使用 `style` 是可以接受的。对于其他情况，例如将文本设为红色，添加一个背景图标 — 可以在 CSS 中对这些样式进行描述，然后添加类（JavaScript 可以做到）。这样更灵活，更易于支持。

#### 3.1 `className`和`classList`

在很久以前，JavaScript 中有一个限制：像 `"class"` 这样的保留字不能用作对象的属性。这一限制现在已经不存在了，但当时就不能存在像 `elem.class` 这样的 `"class"` 属性。因此，对于类，引入了看起来类似的属性 `"className"`：`elem.className` 对应于 `"class"` 特性（attribute），它是包含所有类的字符串。

如果对 `elem.className` 进行赋值，它将替换类中的整个字符串。有时，这正是所需要的，但通常希望的是添加/删除单个类。

这里还有另一个属性：`elem.classList`。`elem.classList` 是一个特殊的对象，它具有 `add/remove/toggle` 单个类的方法。如：

```html
<body class="main page">
  <script>
    // 添加一个 class
    document.body.classList.add('article');

    alert(document.body.className); // main page article
  </script>
</body>
```

因此，既可以使用 `className` 对完整的类字符串进行操作，也可以使用使用 `classList` 对单个类进行操作。选择什么取决于我们的需求。

`classList` 的方法：

- `elem.classList.add/remove(class)` — 添加/移除类。
- `elem.classList.toggle(class)` — 如果类不存在就添加类，存在就移除它。
- `elem.classList.contains(class)` — 检查给定类，返回 `true/false`。

此外，`classList` 是可迭代的，因此可以像下面这样列出所有类：

```html
<body class="main page">
  <script>
    for (let name of document.body.classList) {
      alert(name); // main，然后是 page
    }
  </script>
</body>
```

#### 3.2 元素样式

`elem.style` 属性是一个对象，它对应于 `"style"` 特性（attribute）中所写的内容。`elem.style.width="100px"` 的效果等价于在 `style` 特性中有一个 `width:100px` 字符串。

对于多词（multi-word）属性，使用驼峰式 camelCase：

```javascript
background-color  => elem.style.backgroundColor
z-index           => elem.style.zIndex
border-left-width => elem.style.borderLeftWidth
```

例如：

```javascript
document.body.style.backgroundColor = prompt('background color?', 'green');
```

**前缀属性**

像 `-moz-border-radius` 和 `-webkit-border-radius` 这样的浏览器前缀属性，也遵循同样的规则：连字符 `-` 表示大写。例如：

```javascript
button.style.MozBorderRadius = '5px';
button.style.WebkitBorderRadius = '5px';
```

#### 3.3 重置样式属性

有时想要分配一个样式属性，稍后移除它。

例如，为了隐藏一个元素，我们可以设置 `elem.style.display = "none"`。

然后，稍后可能想要移除 `style.display`，就像它没有被设置一样。这里不应该使用 `delete elem.style.display`，而应该使用 `elem.style.display = ""` 将其赋值为空。

```javascript
// 如果运行这段代码，<body> 将会闪烁
document.body.style.display = "none"; // 隐藏

setTimeout(() => document.body.style.display = "", 1000); // 恢复正常
```

如果将 `display` 设置为空字符串，那么浏览器通常会应用 CSS 类以及内建样式，就好像根本没有这样的 `style` 属性一样。

**用** `style.cssText` **进行完全的重写**

通常，可以使用 `style.*` 来对各个样式属性进行赋值。但不能像这样的 `div.style="color: red; width: 100px"` 设置完整的属性，因为 `div.style` 是一个对象，并且它是只读的。

想要以字符串的形式设置完整的样式，可以使用特殊属性 `style.cssText`：

```html
<div id="div">Button</div>

<script>
  // 可以在这里设置特殊的样式标记，例如 "important"
  div.style.cssText=`color: red !important;
    background-color: yellow;
    width: 100px;
    text-align: center;
  `;

  alert(div.style.cssText);
</script>
```

一般很少使用这个属性，因为这样的赋值会删除所有现有样式：它不是进行添加，而是替换它们。有时可能会删除所需的内容。但是，当知道不会删除现有样式时，可以安全地将其用于新元素。

可以通过设置一个特性（attribute）来实现同样的效果：`div.setAttribute('style', 'color: red...')`。

#### 3.4 注意单位

不要忘记将 CSS 单位添加到值上。例如，不应该将 `elem.style.top` 设置为 `10`，而应将其设置为 `10px`。否则设置会无效

#### 3.5 计算样式：`getComputedStyle`

修改样式很简单。但是如何 **读取** 样式呢？例如，想知道元素的 size，margins 和 color。应该怎么获取？

**`style` 属性仅对 `"style"` 特性（attribute）值起作用，而没有任何 CSS 级联（cascade）。**因此无法使用 `elem.style` 读取来自 CSS 类的任何内容。例如，这里的 `style` 看不到 margin：

```html
<head>
  <style> body { color: red; margin: 5px } </style>
</head>
<body>

  The red text
  <script>
    alert(document.body.style.color); // 空的
    alert(document.body.style.marginTop); // 空的
  </script>
</body>
```

但如果需要，例如，将 margin 增加 20px 呢？那么就需要获取 margin 的当前值。

对于这个需求，这里有另一种方法：`getComputedStyle`。语法如下：

```javascript
getComputedStyle(element, [pseudo])
```

- element

  需要被读取样式值的元素。

- pseudo

  伪元素（如果需要），例如 `::before`。空字符串或无参数则意味着元素本身。

结果是一个具有样式属性的对象，像 `elem.style`，但现在对于所有的 CSS 类来说都是如此。例如：

```html
<head>
  <style> body { color: red; margin: 5px } </style>
</head>
<body>

  <script>
    let computedStyle = getComputedStyle(document.body);

    // 现在可以读取它的 margin 和 color 了

    alert( computedStyle.marginTop ); // 5px
    alert( computedStyle.color ); // rgb(255, 0, 0)
  </script>

</body>
```

**计算值和解析值**

在 [CSS](https://drafts.csswg.org/cssom/#resolved-values) 中有两个概念：

1. **计算 (computed)** 样式值是所有 CSS 规则和 CSS 继承都应用后的值，这是 CSS 级联（cascade）的结果。它看起来像 `height:1em` 或 `font-size:125%`。
2. **解析 (resolved)** 样式值是最终应用于元素的样式值值。诸如 `1em` 或 `125%` 这样的值是相对的。浏览器将使用计算（computed）值，并使所有单位均为固定的，且为绝对单位，例如：`height:20px` 或 `font-size:16px`。对于几何属性，解析（resolved）值可能具有浮点，例如：`width:50.5px`。

很久以前，创建了 `getComputedStyle` 来获取计算（computed）值，但事实证明，解析（resolved）值要方便得多，标准也因此发生了变化。所以，现在 `getComputedStyle` 实际上返回的是属性的解析值（resolved）。

**`getComputedStyle` 需要完整的属性名**

在`getComputedStyle`返回的对象中应该总是使用想要的确切的属性，例如 `paddingLeft`、`marginTop` 或 `borderTopWidth`。否则，就不能保证正确的结果。

例如，如果有 `paddingLeft/paddingTop` 属性，那么对于 `getComputedStyle(elem).padding`，会得到什么？什么都没有，或者是从已知的 padding 中“生成”的值？这里没有标准的规则。

还有其他不一致的地方。例如，在下面这个例子中，某些浏览器（Chrome）会显示 `10px`，而某些浏览器（Firefox）则没有：

```html
<style>
  body {
    margin: 10px;
  }
</style>
<script>
  let style = getComputedStyle(document.body);
  alert(style.margin); // 在 Firefox 中是空字符串
</script>
```

**应用于 `:visited` 链接的样式被隐藏了！**

可以使用 CSS 伪类 `:visited` 对被访问过的链接进行着色。

但 `getComputedStyle` 没有给出访问该颜色的方式，因为否则，任意页面都可以通过在页面上创建它，并通过检查样式来确定用户是否访问了某链接。

JavaScript 看不到 `:visited` 所应用的样式。此外，CSS 中也有一个限制，即禁止在 `:visited` 中应用更改几何形状的样式。这是为了确保一个不好的页面无法测试链接是否被访问，进而窥探隐私。

### 四、元素大小和滚动

JavaScript 中有许多属性可以读取有关元素宽度、高度和其他几何特征的信息。在 JavaScript 中移动或定位元素时，会经常需要它们。

示例元素：

```html
<div id="example">
  ...Text...
</div>
<style>
  #example {
    width: 300px;
    height: 200px;
    border: 25px solid #E8C48F;
    padding: 20px;
    overflow: auto;
  }
</style>
```

它有边框（border），内边距（padding）和滚动（scrolling）等全套功能。但没有外边距（margin），因为它们不是元素本身的一部分，并且它们没什么特殊的属性。

这个元素看起来就像这样：

![example](https://gitee.com/Topcvan//js-notes-img/raw/master/example.png)

**注意滚动条**

上图演示了元素具有滚动条这种最复杂的情况。一些浏览器（并非全部）通过从内容（上面标记为 “content width”）中获取空间来为滚动条保留空间。

因此，如果没有滚动条，内容宽度将是 `300 px`，但是如果滚动条宽度是 `16px`（不同的设备和浏览器，滚动条的宽度可能有所不同），那么还剩下 `300 - 16 ＝ 284px`，应该考虑到这一点。这就是为什么本章的例子总是假设有滚动条。如果没有滚动条，一些计算会更简单。

**文本可能会溢出到 `padding-bottom` 中**

在插图中的 padding 中通常显示为空，但是如果元素中有很多文本，并且溢出了，那么浏览器会在 `padding-bottom` 处显示“溢出”文本，这是正常现象。

**几何**

这是带有几何属性的整体图片：

![几何](https://gitee.com/Topcvan//js-notes-img/raw/master/%E5%87%A0%E4%BD%95.png)

这些属性的值在技术上讲是数字，但这些数字其实是“像素（pixel）”，因此它们是像素测量值。

#### 4.1 `offsetParent,offsetLeft/Top`

这些属性很少使用，但它们仍然是“最外面”的几何属性

`offsetParent` 是最接近的祖先（ancestor），在浏览器渲染期间，它被用于计算坐标。

最近的祖先为下列之一：

1. CSS 定位的（`position` 为 `absolute`，`relative` 或 `fixed`），
2. 或 `<td>`，`<th>`，`<table>`，
3. 或 `<body>`。

属性 `offsetLeft/offsetTop` 提供相对于 `offsetParent` 左上角的 x/y 坐标。

在下面这个例子中，内部的 `<div>` 有 `<main>` 作为 `offsetParent`，并且 `offsetLeft/offsetTop` 让它从左上角位移（`180`）：

```html
<main style="position: relative" id="main">
  <article>
    <div id="example" style="position: absolute; left: 180px; top: 180px">...</div>
  </article>
</main>
<script>
  alert(example.offsetParent.id); // main
  alert(example.offsetLeft); // 180（注意：这是一个数字，不是字符串 "180px"）
  alert(example.offsetTop); // 180
</script>
```

![offsetLeft](https://gitee.com/Topcvan//js-notes-img/raw/master/offsetLeft.png)

有以下几种情况下，`offsetParent` 的值为 `null`：

1. 对于未显示的元素（`display:none` 或者不在文档中）。
2. 对于 `<body>` 与 `<html>`。
3. 对于带有 `position:fixed` 的元素。

#### 4.2 `offsetWidth/Height`

这两个属性是最简单的。它们提供了元素的“外部” width/height。或者，换句话说，它的完整大小（包括边框）。

![offsetWidth](https://gitee.com/Topcvan//js-notes-img/raw/master/offsetWidth.png)

对于示例元素：

- `offsetWidth = 390` — 外部宽度（width），可以计算为内部 CSS-width（`300px`）加上 padding（`2 * 20px`）和 border（`2 * 25px`）。
- `offsetHeight = 290` — 外部高度（height）。

**对于未显示的元素，几何属性为 0/null**

仅针对显示的元素计算几何属性。

如果一个元素（或其任何祖先）具有 `display:none` 或不在文档中，则所有几何属性均为零（或 `offsetParent` 为 `null`）。

例如，当创建了一个元素，但尚未将其插入文档中，或者它（或它的祖先）具有 `display:none` 时，`offsetParent` 为 `null`，并且 `offsetWidth` 和 `offsetHeight` 为 `0`。

可以用它来检查一个元素是否被隐藏，像这样：

```javascript
function isHidden(elem) {
  return !elem.offsetWidth && !elem.offsetHeight;
}
```

请注意，对于屏幕上显示，但大小为零的元素（例如空的 `<div>`），它们的 `isHidden` 返回 `true`。

#### 4.3 `clientTop/Left`

在元素内部有边框（border）。为了测量它们，可以使用 `clientTop` 和 `clientLeft`。

在例子中：

- `clientLeft = 25` — 左边框宽度
- `clientTop = 25` — 上边框宽度

但准确地说 — 这些属性不是边框的 width/height，而是内侧与外侧的相对坐标。

有什么区别？

当文档从右到左显示（操作系统为阿拉伯语或希伯来语）时，影响就显现出来了。此时滚动条不在右边，而是在左边，此时 `clientLeft` 则包含了滚动条的宽度。

在这种情况下，`clientLeft` 的值将不是 `25`，而是加上滚动条的宽度 `25 + 16 = 41`。

#### 4.4 `clientWidth/Height`

这些属性提供了元素边框内区域的大小。它们包括了 “content width” 和 “padding”，但不包括滚动条宽度（scrollbar）：

![clientWidth](https://gitee.com/Topcvan//js-notes-img/raw/master/clientWidth.png)

在上图中，首先考虑 `clientHeight`。

这里没有水平滚动条，所以它恰好是 border 内的总和：CSS-width `200px` 加上顶部和底部的 padding（`2 * 20px`），总计 `240px`。

现在 `clientWidth` — 这里的 “content width” 不是 `300px`，而是 `284px`，因为被滚动条占用了 `16px`。所以加起来就是 `284px` 加上左侧和右侧的 padding，总计 `324px`。

**如果这里没有 padding，那么 `clientWidth/Height` 代表的就是内容区域，即 border 和 scrollbar（如果有）内的区域。**

![clientWidth1](https://gitee.com/Topcvan//js-notes-img/raw/master/clientWidth1.png)

因此，当没有 padding 时，可以使用 `clientWidth/clientHeight` 来获取内容区域的大小。

**CSS width 与 clientWidth 的不同点**

1. `clientWidth` 值是数值，而 `getComputedStyle(elem).width` 返回一个以 `px` 作为后缀的字符串。
2. `getComputedStyle` 可能会返回非数值的 width，例如内联（inline）元素的 `"auto"`。
3. `clientWidth` 是元素的内部内容区域加上 padding，而 CSS width（具有标准的 `box-sizing`）是内部内容区域，**不包括 padding**。
4. 如果有滚动条，并且浏览器为其保留了空间，那么某些浏览器会从 CSS width 中减去该空间（因为它不再可用于内容），而有些则不会这样做。`clientWidth` 属性总是相同的：如果为滚动条保留了空间，那么将减去滚动条的大小。

#### 4.5 `scrollWidth/Height`

这些属性就像 `clientWidth/clientHeight`，但它们还包括滚动出（隐藏）的部分：![scrollWidth](https://gitee.com/Topcvan//js-notes-img/raw/master/scrollWidth.png)

在上图中：

- `scrollHeight = 723` — 是内容区域的完整内部高度，包括滚动出的部分。
- `scrollWidth = 324` — 是完整的内部宽度，这里我们没有水平滚动，因此它等于 `clientWidth`。

可以使用这些属性将元素展开（expand）到整个 width/height。

像这样：

```javascript
// 将元素展开（expand）到完整的内容高度
element.style.height = `${element.scrollHeight}px`;
```

#### 4.6 `scrollLeft/Top`

属性 `scrollLeft/scrollTop` 是元素的隐藏、滚动部分的 width/height。在下图中，可以看到带有垂直滚动块的 `scrollHeight` 和 `scrollTop`。![scrollLeft](https://gitee.com/Topcvan//js-notes-img/raw/master/scrollLeft.png)

换句话说，`scrollTop` 就是“已经滚动了多少”。

**`scrollLeft/scrollTop` 是可修改的**

大多数几何属性是只读的，但是 `scrollLeft/scrollTop` 是可修改的，并且浏览器会滚动该元素。将 `scrollTop` 设置为 `0` 或一个大的值，例如 `1e9`，将会使元素滚动到顶部/底部。

#### 4.7 不要从`CSS`中获取`width/height`

DOM 元素的几何属性，它们可用于获得宽度、高度和计算距离。但是，正如在 [样式和类](https://zh.javascript.info/styles-and-classes) 一章所知道的那样，可以使用 `getComputedStyle` 来读取 CSS-width 和 height。那为什么不像这样用 `getComputedStyle` 读取元素的 width 呢？

```javascript
let elem = document.body;

alert( getComputedStyle(elem).width ); // 显示 elem 的 CSS width
```

为什么应该使用几何属性呢？这里有两个原因：

1. 首先，CSS `width/height` 取决于另一个属性：`box-sizing`，它定义了“什么是” CSS 宽度和高度。出于 CSS 的目的而对 `box-sizing` 进行的更改可能会破坏此类 JavaScript 操作。

2. 其次，CSS 的 `width/height` 可能是 `auto`，例如内联（inline）元素：

   ```html
   <span id="elem">Hello!</span>
   
   <script>
     alert( getComputedStyle(elem).width ); // auto
   </script>
   ```

   从 CSS 的观点来看，`width:auto` 是完全正常的，但在 JavaScript 中，需要一个确切的 `px` 大小，以便在计算中使用它。因此，这里的 CSS 宽度没什么用。

还有另一个原因：滚动条。有时，在没有滚动条的情况下代码工作正常，当出现滚动条时，代码就出现了 bug，因为在某些浏览器中，滚动条会占用内容的空间。因此，可用于内容的实际宽度小于 CSS 宽度。而 `clientWidth/clientHeight` 则会考虑到这一点。

……但是，使用 `getComputedStyle(elem).width` 时，情况就不同了。某些浏览器（例如 Chrome）返回的是实际内部宽度减去滚动条宽度，而某些浏览器（例如 Firefox）返回的是 CSS 宽度（忽略了滚动条）。这种跨浏览器的差异是不使用 `getComputedStyle` 而依靠几何属性的原因。

### 五、`Window`大小和滚动

#### 5.1 窗口的`width/height`

为了获取窗口（window）的宽度和高度，可以使用 `document.documentElement` 的`clientWidth/clientHeight`：

![窗口宽度](https://gitee.com/Topcvan//js-notes-img/raw/master/%E7%AA%97%E5%8F%A3%E5%AE%BD%E5%BA%A6.png)

**不是 `window.innerWidth/innerHeight`**

浏览器也支持像 `window.innerWidth/innerHeight` 这样的属性。它们看起来像想要的属性，那为什么不使用它们呢？

如果这里存在一个滚动条，并且滚动条占用了一些空间，那么 `clientWidth/clientHeight` 会提供没有滚动条（减去它）的 width/height。换句话说，它们返回的是可用于内容的文档的可见部分的 width/height。

`window.innerWidth/innerHeight` 包括了滚动条。

如果这里有一个滚动条，它占用了一些空间，那么这两行代码会显示不同的值：

```javascript
alert( window.innerWidth ); // 整个窗口的宽度
alert( document.documentElement.clientWidth ); // 减去滚动条宽度后的窗口宽度
```

在大多数情况下，需要的是 **可用** 的窗口宽度以绘制或放置某些东西。也就是说，在滚动条内（如果有）。所以，应该使用 `documentElement.clientHeight/clientWidth`。

**`DOCTYPE` 很重要**

请注意：当 HTML 中没有 `<!DOCTYPE HTML>` 时，顶层级（top-level）几何属性的工作方式可能就会有所不同。可能会出现一些稀奇古怪的情况。

在现代 HTML 中，始终都应该写 `DOCTYPE`。

#### 5.2 文档的`width/height`

从理论上讲，由于根文档元素是 `document.documentElement`，并且它包围了所有内容，因此可以通过使用 `documentElement.scrollWidth/scrollHeight` 来测量文档的完整大小。

但是在该元素上，对于整个文档，这些属性均无法正常工作。在 Chrome/Safari/Opera 中，如果没有滚动条，`documentElement.scrollHeight` 甚至可能小于 `documentElement.clientHeight`！

为了可靠地获得完整的文档高度，应该采用以下这些属性的最大值：

```javascript
let scrollHeight = Math.max(
  document.body.scrollHeight, document.documentElement.scrollHeight,
  document.body.offsetHeight, document.documentElement.offsetHeight,
  document.body.clientHeight, document.documentElement.clientHeight
);

alert('Full document height, with scrolled out part: ' + scrollHeight);
```

这些不一致来源于远古时代，而不是“聪明”的逻辑。

#### 5.3 获得当前滚动

DOM 元素的当前滚动状态在其 `scrollLeft/scrollTop` 属性中。

对于文档滚动，在大多数浏览器中，可以使用 `document.documentElement.scrollLeft/scrollTop`，但在较旧的基于 WebKit 的浏览器中则不行，例如在 Safari（bug [5991](https://bugs.webkit.org/show_bug.cgi?id=5991)）中，应该使用 `document.body` 而不是 `document.documentElement`。

幸运的是，根本不必记住这些特性，因为滚动在 `window.pageXOffset/pageYOffset` 中可用：

```javascript
alert('Current scroll from the top: ' + window.pageYOffset);
alert('Current scroll from the left: ' + window.pageXOffset);
```

这些属性是只读的。

#### 5.4 滚动：`scrollTo,scrollBy,scrollIntoView`

可以通过更改 `scrollTop/scrollLeft` 来滚动常规元素。也可以使用`document.documentElement.scrollTop/scrollLeft` 对页面进行相同的操作（Safari 除外，而应该使用 `document.body.scrollTop/Left` 代替）。

或者，有一个更简单的通用解决方案：使用特殊方法 [window.scrollBy(x,y)](https://developer.mozilla.org/zh/docs/Web/API/Window/scrollBy) 和 [window.scrollTo(pageX,pageY)](https://developer.mozilla.org/zh/docs/Web/API/Window/scrollTo)。

- 方法 `scrollBy(x,y)` 将页面滚动至 **相对于当前位置的 `(x, y)` 位置**。例如，`scrollBy(0,10)` 会将页面向下滚动 `10px`。
- 方法 `scrollTo(pageX,pageY)` 将页面滚动至 **绝对坐标**，使得可见部分的左上角具有相对于文档左上角的坐标 `(pageX, pageY)`。就像设置了 `scrollLeft/scrollTop` 一样。要滚动到最开始，可以使用 `scrollTo(0,0)`。
- 对 `elem.scrollIntoView(top)` 的调用将滚动页面以使 `elem` 可见。它有一个参数：
  - 如果 `top=true`（默认值），页面滚动，使 `elem` 出现在窗口顶部。元素的上边缘将与窗口顶部对齐。
  - 如果 `top=false`，页面滚动，使 `elem` 出现在窗口底部。元素的底部边缘将与窗口底部对齐。

这些方法适用于所有浏览器。

此外，必须在 DOM 完全构建好之后才能通过 JavaScript 滚动页面。例如，如果尝试通过 `<head>` 中的脚本滚动页面，它将无法正常工作。

#### 5.5 禁止滚动

有时候需要使文档“不可滚动”。例如，当需要用一条需要立即引起注意的大消息来覆盖文档时，希望访问者与该消息而不是与文档进行交互。

要使文档不可滚动，只需要设置 `document.body.style.overflow = "hidden"`。该页面将“冻结”在其当前滚动位置上。还可以使用相同的技术来冻结其他元素的滚动，而不仅仅是 `document.body`。

这个方法的缺点是会使滚动条消失。如果滚动条占用了一些空间，它原本占用的空间就会空出来，那么内容就会“跳”进去以填充它。

这看起来有点奇怪，但是可以对比冻结前后的 `clientWidth`。如果它增加了（滚动条消失后），那么可以在`document.body` 中滚动条原来的位置处通过添加 `padding`，来替代滚动条，这样这个问题就解决了。保持了滚动条冻结前后文档内容宽度相同。

### 六、坐标

要移动页面的元素，应该先熟悉坐标。大多数 JavaScript 方法处理的是以下两种坐标系中的一个：

1. **相对于窗口** — 类似于 `position:fixed`，从窗口的顶部/左侧边缘计算得出。
   - 这些坐标表示为 `clientX/clientY`
2. **相对于文档** — 与文档根（document root）中的 `position:absolute` 类似，从文档的顶部/左侧边缘计算得出。
   - 将它们表示为 `pageX/pageY`。

当页面滚动到最开始时，此时窗口的左上角恰好是文档的左上角，它们的坐标彼此相等。但是，在文档移动之后，元素的窗口相对坐标会发生变化，因为元素在窗口中移动，而元素在文档中的相对坐标保持不变。

在下图中，在文档中取一点，并演示了它滚动之前（左）和之后（右）的坐标：![相对坐标](https://gitee.com/Topcvan//js-notes-img/raw/master/%E7%9B%B8%E5%AF%B9%E5%9D%90%E6%A0%87.png)

当文档滚动了：

- `pageY` — 元素在文档中的相对坐标保持不变，从文档顶部（现在已滚动出去）开始计算。
- `clientY` — 窗口相对坐标确实发生了变化（箭头变短了），因为同一个点越来越靠近窗口顶部。

#### 6.1 元素坐标：`getBoundingClientRect`

方法 `elem.getBoundingClientRect()` 返回最小矩形的窗口坐标，该矩形将 `elem` 作为内建 [DOMRect](https://www.w3.org/TR/geometry-1/#domrect) 类的对象。

主要的 `DOMRect` 属性：

- `x/y` — 矩形原点相对于窗口的 X/Y 坐标，
- `width/height` — 矩形的 width/height（可以为负）。

此外，还有派生（derived）属性：

- `top/bottom` — 顶部/底部矩形边缘的 Y 坐标，
- `left/right` — 左/右矩形边缘的 X 坐标。

下面这张是 `elem.getBoundingClientRect()` 的输出的示意图：

![DOMRect](https://gitee.com/Topcvan//js-notes-img/raw/master/DOMRect.png)

`x/y` 和 `width/height` 对矩形进行了完整的描述。可以很容易地从它们计算出派生（derived）属性：

- `left = x`
- `top = y`
- `right = x + width`
- `bottom = y + height`

注意事项：

- 坐标可能是小数，例如 `10.5`。这是正常的，浏览器内部使用小数进行计算。在设置 `style.left/top` 时，不是必须对它们进行舍入。
- 坐标可能是负数。例如滚动页面，使 `elem` 现在位于窗口的上方，则 `elem.getBoundingClientRect().top` 为负数。

**为什么需要派生（derived）属性？如果有了 `x/y`，为什么还要还会存在 `top/left`？**

从数学上讲，一个矩形是使用其起点 `(x,y)` 和方向向量 `(width,height)` 唯一定义的。因此，其它派生属性是为了方便起见。

从技术上讲，`width/height` 可能为负数，从而允许“定向（directed）”矩形，例如代表带有正确标记的开始和结束的鼠标选择。

负的 `width/height` 值表示矩形从其右下角开始，然后向左上方“增长”。

这是一个矩形，其 `width` 和 `height` 均为负数（例如 `width=-200`，`height=-100`）：

![left和x的区别](https://gitee.com/Topcvan//js-notes-img/raw/master/left%E5%92%8Cx%E7%9A%84%E5%8C%BA%E5%88%AB.png)

**坐标的 right/bottom 与 CSS position 属性不同**

相对于窗口（window）的坐标和 CSS `position:fixed` 之间有明显的相似之处。

但是在 CSS 定位中，`right` 属性表示距右边缘的距离，而 `bottom` 属性表示距下边缘的距离。

如果再看一下上面的图片，可以看到在 JavaScript 中并非如此。窗口的所有坐标都从左上角开始计数，包括这些坐标。

#### 6.2 `elementFromPoint(x,y)`

对 `document.elementFromPoint(x, y)` 的调用会返回在窗口坐标 `(x, y)` 处嵌套最多（the most nested）的元素。

语法如下：

```javascript
let elem = document.elementFromPoint(x, y);
```

例如，下面的代码会高亮显示并输出现在位于窗口中间的元素的标签：

```javascript
let centerX = document.documentElement.clientWidth / 2;
let centerY = document.documentElement.clientHeight / 2;

let elem = document.elementFromPoint(centerX, centerY);

elem.style.background = "red";
alert(elem.tagName);
```

因为它使用的是窗口坐标，所以元素可能会因当前滚动位置而有所不同。

**对于在窗口之外的坐标，`elementFromPoint` 返回 `null`**

方法 `document.elementFromPoint(x,y)` 只对在可见区域内的坐标 `(x,y)` 起作用。如果任何坐标为负或者超过了窗口的 width/height，那么该方法就会返回 `null`。

#### 6.3 用于`fixed`定位

为了显示元素附近的东西，可以使用 `getBoundingClientRect` 来获取其坐标，然后使用 CSS `position` 以及 `left/top`（或 `right/bottom`）。

例如，下面的函数 `createMessageUnder(elem, html)` 在 `elem` 下显示了消息：

```javascript
let elem = document.getElementById("coords-show-mark");

function createMessageUnder(elem, html) {
  // 创建 message 元素
  let message = document.createElement('div');
  // 在这里最好使用 CSS class 来定义样式
  message.style.cssText = "position:fixed; color: red";

  // 分配坐标，不要忘记 "px"！
  let coords = elem.getBoundingClientRect();

  message.style.left = coords.left + "px";
  message.style.top = coords.bottom + "px";

  message.innerHTML = html;

  return message;
}

// 用法：
// 在文档中添加 message 保持 5 秒
let message = createMessageUnder(elem, 'Hello, world!');
document.body.append(message);
setTimeout(() => message.remove(), 5000);
```

#### 6.4 文档坐标

文档相对坐标从文档的左上角开始计算，而不是窗口。在 CSS 中，窗口坐标对应于 `position:fixed`，而文档坐标与顶部的 `position:absolute` 类似。

可以使用 `position:absolute` 和 `top/left` 来把某些内容放到文档中的某个位置，以便在页面滚动时，元素仍能保留在该位置。但是首先需要正确的坐标。

这里没有标准方法来获取元素的文档坐标。但是写起来很容易。

这两个坐标系统通过以下公式相连接：

- `pageY` = `clientY` + 文档的垂直滚动出的部分的高度。
- `pageX` = `clientX` + 文档的水平滚动出的部分的宽度。

函数 `getCoords(elem)` 将从 `elem.getBoundingClientRect()` 获取窗口坐标，并向其中添加当前滚动：

```javascript
// 获取元素的文档坐标
function getCoords(elem) {
  let box = elem.getBoundingClientRect();

  return {
    top: box.top + window.pageYOffset,
    right: box.right + window.pageXOffset,
    bottom: box.bottom + window.pageYOffset,
    left: box.left + window.pageXOffset
  };
}
```