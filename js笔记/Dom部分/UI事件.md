### 一、鼠标事件

此类事件不仅可能来自于“鼠标设备”，还可能来自于对此类操作进行了模拟以实现兼容性的其他设备，例如手机和平板电脑。

#### 1.1 鼠标事件类型

常见鼠标事件：

- `mousedown/mouseup`

  在元素上点击/释放鼠标按钮。

- `mouseover/mouseout`

  鼠标指针从一个元素上移入/移出。

- `mousemove`

  鼠标在元素上的每个移动都会触发此事件。

- `click`

  如果使用的是鼠标左键，则在同一个元素上的 `mousedown` 及 `mouseup` 相继触发后，触发该事件。

- `dblclick`

  在短时间内双击同一元素后触发。如今已经很少使用了。

- `contextmenu`

  在鼠标右键被按下时触发。还有其他打开上下文菜单的方式，例如使用特殊的键盘按键，在这种情况下它也会被触发，因此它并不完全是鼠标事件。

#### 1.2 事件顺序

从上面的列表中可以看到，一个用户操作可能会触发多个事件。

例如，点击鼠标左键，在鼠标左键被按下时，会首先触发 `mousedown`，然后当鼠标左键被释放时，会触发 `mouseup` 和 `click`。

在单个动作触发多个事件时，事件的顺序是固定的。也就是说，会遵循 `mousedown` → `mouseup` → `click` 的顺序调用处理程序。

#### 1.3 鼠标按钮

与点击相关的事件始终具有 `button` 属性，该属性允许获取确切的鼠标按钮。

通常不在 `click` 和 `contextmenu` 事件中使用这一属性，因为前者只在单击鼠标左键时触发，后者只在单击鼠标右键时触发。

不过，在 `mousedown` 和 `mouseup` 事件中则可能需要用到 `event.button`，因为这两个事件在任何按键上都会触发，所以可以使用 `button` 属性来区分是左键单击还是右键单击。

`event.button` 的所有可能值如下：

| 鼠标按键状态     | `event.button` |
| :--------------- | :------------- |
| 左键 (主要按键)  | 0              |
| 中键 (辅助按键)  | 1              |
| 右键 (次要按键)  | 2              |
| X1 键 (后退按键) | 3              |
| X2 键 (前进按键) | 4              |

大多数鼠标设备只有左键和右键，对应的值就是 `0` 和 `2`。触屏设备中的点按操作也会触发类似的事件。

另外，还有一个 `event.buttons` 属性，其中以整数的形式存储着当前所有按下的鼠标按键，每个按键一个比特位。在实际开发中，很少会用到这个属性。

**过时的 `event.which`**

一些老代码可能会使用 `event.which` 属性来获得按下的按键。这是一个古老的非标准的方式，具有以下可能值：

- `event.which == 1` —— 鼠标左键，
- `event.which == 2` —— 鼠标中键，
- `event.which == 3` —— 鼠标右键。

现在，`event.which` 已经被弃用了，不应再使用它。

#### 1.4 组合键：`shift,alt,ctrl,meta`

所有的鼠标事件都包含有关按下的组合键的信息。

事件属性：

- `shiftKey`：Shift
- `altKey`：Alt（或对于 Mac 是 Opt）
- `ctrlKey`：Ctrl
- `metaKey`：对于 Mac 是 Cmd

如果在事件期间按下了相应的键，则它们为 `true`。

比如，下面这个按钮仅在 Alt+Shift+click 时才有效：

```html
<button id="button">Alt+Shift+Click on me!</button>

<script>
  button.onclick = function(event) {
    if (event.altKey && event.shiftKey) {
      alert('Hooray!');
    }
  };
</script>
```

**在 Mac 上通常使用 `Cmd` 代替 `Ctrl`**

在 Windows 和 Linux 上有 `Alt`，`Shift` 和 `Ctrl`。在 Mac 上还有：`Cmd`，它对应于属性 `metaKey`。

在大多数情况下，当在 `Windows/Linux` 上使用 `Ctrl` 时，在 Mac 是使用 `Cmd`。

也就说：当 `Windows` 用户按下 `Ctrl+Enter` 或 `Ctrl+A` 时，`Mac` 用户会按下 `Cmd+Enter` 或 `Cmd+A`，以此类推。

因此，如果想支持 `Ctrl+click`，那么对于 `Mac` 应该使用 `Cmd+click`。对于 Mac 用户而言，这更舒适。

即使想强制 Mac 用户使用 `Ctrl+click` —— 这非常困难。问题是：在 `MacOS` 上左键单击和 `Ctrl` 一起使用会被解释为 **右键单击**，并且会生成 `contextmenu` 事件，而不是像 `Windows/Linux` 中的 `click` 事件。

因此，如果想让所有操作系统的用户都感到舒适，那么应该将 `ctrlKey` 与 `metaKey` 一起进行检查。对于 `JS` 代码，这意味着应该检查 `if (event.ctrlKey || event.metaKey)`。

#### 1.5 坐标

所有的鼠标事件都提供了两种形式的坐标：

1. 相对于窗口的坐标：`clientX` 和 `clientY`。
2. 相对于文档的坐标：`pageX` 和 `pageY`。

#### 1.6 防止在鼠标按下时的的选择

双击鼠标会有副作用，在某些界面中可能会出现干扰：它会选择文本。

比如，双击下面的文本，除了执行处理程序外，还会选择文本：

```markup
<span ondblclick="alert('dblclick')">Double-click me</span>
```

如果按下鼠标左键，并在不松开的情况下移动鼠标，这也常常会造成不必要的选择。

在这种情况下，最合理的方式是防止浏览器对 `mousedown` 进行操作。这样能够阻止刚刚提到的两种选择：

```markup
Before...
<b ondblclick="alert('Click!')" onmousedown="return false">
  Double-click me
</b>
...After
```

现在，在双击时，粗体元素不会被选中，并且在粗体元素上按下鼠标左键也不会开始选择。

请注意：其中的文本仍然是可选择的。但是，选择不应该开始于该文本自身，而应该在该文本之前或之后开始。通常，这对用户来说挺好的。

**防止复制**

如果想禁用选择以保护页面的内容不被复制粘贴，那么可以使用另一个事件：`oncopy`。

```markup
<div oncopy="alert('Copying forbidden!');return false">
  Dear user,
  The copying is forbidden for you.
  If you know JS or HTML, then you can get everything from the page source though.
</div>
```

如果试图在 `<div>` 中复制一段文本，这是行不通的，因为默认行为 `oncopy` 被阻止了。

当然，用户可以访问页面的 HTML 源码，并且可以从那里获取内容，但并不是每个人都知道如何做到这一点。

### 二、移动鼠标

#### 2.1 `mouseover/mouseout,relatedTarget`

当鼠标指针移到某个元素上时，`mouseover` 事件就会发生，而当鼠标离开该元素时，`mouseout` 事件就会发生。

这些事件很特别，因为它们具有 `relatedTarget` 属性。此属性是对 `target` 的补充。当鼠标从一个元素离开并去往另一个元素时，其中一个元素就变成了 `target`，另一个就变成了 `relatedTarget`。

对于 `mouseover`：

- `event.target` —— 是鼠标移过的那个元素。
- `event.relatedTarget` —— 是鼠标来自的那个元素（`relatedTarget` → `target`）。

`mouseout` 则与之相反：

- `event.target` —— 是鼠标离开的元素。
- `event.relatedTarget` —— 是鼠标移动到的，当前指针位置下的元素（`target` → `relatedTarget`）。

**`relatedTarget` 可以为 `null`**

`relatedTarget` 属性可以为 `null`。这是正常现象，仅仅是意味着鼠标不是来自另一个元素，而是来自窗口之外。或者它离开了窗口。当在代码中使用 `event.relatedTarget` 时，应该牢记这种可能性。如果访问`event.relatedTarget.tagName`，那么就会出现错误。

#### 2.2 跳过元素

当鼠标移动时，就会触发 `mousemove` 事件。但这并不意味着每个像素都会导致一个事件。浏览器会一直检查鼠标的位置。如果发现了变化，就会触发事件。这意味着，如果访问者非常快地移动鼠标，那么某些 DOM 元素就可能被跳过：

![鼠标掠过](https://gitee.com/Topcvan//js-notes-img/raw/master/%E9%BC%A0%E6%A0%87%E6%8E%A0%E8%BF%87.png)

如果鼠标从上图所示的 `#FROM` 快速移动到 `#TO` 元素，则中间的 `<div>`（或其中的一些）元素可能会被跳过。`mouseout` 事件可能会在 `#FROM` 上被触发，然后立即在 `#TO` 上触发 `mouseover`。

这对性能很有好处，因为可能有很多中间元素。但并不真的想要处理每一个移入和离开的过程。

另一方面，应该记住，鼠标指针并不会“访问”所有元素。它可以“跳过”一些元素。

特别是，鼠标指针可能会从窗口外跳到页面的中间。在这种情况下，`relatedTarget` 为 `null`，因为它是从石头缝里蹦出来的（nowhere）：

![mousemove](https://gitee.com/Topcvan//js-notes-img/raw/master/mousemove.png)

**如果 `mouseover` 被触发了，则必须有 `mouseout`**

在鼠标快速移动的情况下，中间元素可能会被忽略，但是可以肯定一件事：如果鼠标指针“正式地”进入了一个元素（生成了 `mouseover` 事件），那么一旦它离开，就会得到 `mouseout`。

#### 2.3 移动到子元素

`mouseout` 的一个重要功能 —— 当鼠标指针从元素移动到其后代时触发，例如在下面的这个 HTML 中，从 `#parent` 到 `#child`：

```html
<div id="parent">
  <div id="child">...</div>
</div>
```

如果在 `#parent` 上，然后将鼠标指针更深入地移入 `#child`，在 `#parent` 上会得到 `mouseout`！

![移动到子元素](https://gitee.com/Topcvan//js-notes-img/raw/master/%E7%A7%BB%E5%8A%A8%E5%88%B0%E5%AD%90%E5%85%83%E7%B4%A0.png)

这听起来很奇怪，但很容易解释。

**根据浏览器的逻辑，鼠标指针随时可能位于单个元素上 —— 嵌套最多的那个元素（z-index 最大的那个）。**因此，如果它转到另一个元素（甚至是一个后代），那么它将离开前一个元素。请注意事件处理的另一个重要的细节。后代的 `mouseover` 事件会冒泡。因此，如果 `#parent` 具有 `mouseover` 处理程序，它将被触发：

![嵌套mousemove](https://gitee.com/Topcvan//js-notes-img/raw/master/%E5%B5%8C%E5%A5%97mousemove.png)

当鼠标指针从 `#parent` 元素移动到 `#child` 时，会在父元素上触发两个处理程序：`mouseout` 和 `mouseover`：

```javascript
parent.onmouseout = function(event) {
  /* event.target: parent element */
};
parent.onmouseover = function(event) {
  /* event.target: child element (bubbled) */
};
```

**如果我们不检查处理程序中的 `event.target`，那么似乎鼠标指针离开了 `#parent` 元素，然后立即回到了它上面。**

但是事实并非如此！鼠标指针仍然位于父元素上，它只是更深入地移入了子元素。

如果离开父元素时有一些行为（action），例如一个动画在 `parent.onmouseout` 中运行，当鼠标指针深入 `#parent` 时，并不希望发生这种行为。

为了避免它，可以在处理程序中检查 `relatedTarget`，如果鼠标指针仍在元素内，则忽略此类事件。

另外，可以使用其他事件：`mouseenter` 和 `mouseleave`，它们没有此类问题。

#### 2.4 `mouseenter&mouseleave`

事件 `mouseenter/mouseleave` 类似于 `mouseover/mouseout`。它们在鼠标指针进入/离开元素时触发。

但是有两个重要的区别：

1. 元素内部与后代之间的转换不会产生影响。
2. 事件 `mouseenter/mouseleave` 不会冒泡。

这些事件非常简单。

当鼠标指针进入一个元素时 —— 会触发 `mouseenter`。而鼠标指针在元素或其后代中的确切位置无关紧要。

当鼠标指针离开该元素时，事件 `mouseleave` 才会触发。

这个例子和上面的例子相似，但是现在最顶部的元素有 `mouseenter/mouseleave` 而不是 `mouseover/mouseout`。

#### 2.5 事件委托

事件 `mouseenter/leave` 非常简单且易用。但它们不会冒泡。因此，不能使用它们来进行事件委托。

假设要处理表格的单元格的鼠标进入/离开。并且这里有数百个单元格。

通常的解决方案是 —— 在 `<table>` 中设置处理程序，并在那里处理事件。但 `mouseenter/leave` 不会冒泡。因此，如果类似的事件发生在 `<td>` 上，那么只有 `<td>` 上的处理程序才能捕获到它。

`<table>` 上的 `mouseenter/leave` 的处理程序仅在鼠标指针进入/离开整个表格时才会触发。无法获取有关其内部移动的任何信息。

因此，需要使用 `mouseover/mouseout`。

从高亮显示鼠标指针下的元素的简单处理程序开始：

```javascript
// 高亮显示鼠标指针下的元素
table.onmouseover = function(event) {
  let target = event.target;
  target.style.background = 'pink';
};

table.onmouseout = function(event) {
  let target = event.target;
  target.style.background = '';
};
```

现在它们已经激活了。当鼠标在下面这个表格的各个元素上移动时，当前位于鼠标指针下的元素会被高亮显示：![八卦1](https://gitee.com/Topcvan//js-notes-img/raw/master/%E5%85%AB%E5%8D%A61.png)

在这个例子中，需要处理表格的单元格 `<td>` 之间的移动：进入一个单元格并离开它。但对其他移动并不感兴趣，例如在单元格内部或在所有单元格的外部。

可以这样做将它们过滤：

- 在变量中记住当前被高亮显示的 `<td>`，称它为 `currentElem`。
- `mouseover` —— 如果仍然在当前的 `<td>` 中，则忽略该事件。
- `mouseout` —— 如果没有离开当前的 `<td>`，则忽略。

这是说明所有可能情况的代码示例：

```javascript
// 现在位于鼠标下方的 <td>（如果有）
let currentElem = null;

table.onmouseover = function(event) {
  // 在进入一个新的元素前，鼠标总是会先离开前一个元素
  // 如果设置了 currentElem，那么我们就没有鼠标所悬停在的前一个 <td>，
  // 忽略此事件
  if (currentElem) return;

  let target = event.target.closest('td');

  // 我们移动到的不是一个 <td> —— 忽略
  if (!target) return;

  // 现在移动到了 <td> 上，但在处于了我们表格的外部（可能因为是嵌套的表格）
  // 忽略
  if (!table.contains(target)) return;

  // 给力！我们进入了一个新的 <td>
  currentElem = target;
  onEnter(currentElem);
};


table.onmouseout = function(event) {
  // 如果我们现在处于所有 <td> 的外部，则忽略此事件
  // 这可能是一个表格内的移动，但是在 <td> 外，
  // 例如从一个 <tr> 到另一个 <tr>
  if (!currentElem) return;

  // 我们将要离开这个元素 —— 去哪儿？可能是去一个后代？
  let relatedTarget = event.relatedTarget;

  while (relatedTarget) {
    // 到父链上并检查 —— 我们是否还在 currentElem 内
    // 然后发现，这只是一个内部移动 —— 忽略它
    if (relatedTarget == currentElem) return;

    relatedTarget = relatedTarget.parentNode;
  }

  // 我们离开了 <td>。真的。
  onLeave(currentElem);
  currentElem = null;
};

// 任何处理进入/离开一个元素的函数
function onEnter(elem) {
  elem.style.background = 'pink';

  // 在文本区域显示它
  text.value += `over -> ${currentElem.tagName}.${currentElem.className}\n`;
  text.scrollTop = 1e6;
}

function onLeave(elem) {
  elem.style.background = '';

  // 在文本区域显示它
  text.value += `out <- ${elem.tagName}.${elem.className}\n`;
  text.scrollTop = 1e6;
}
```

再次，重要的功能是：

1. 它使用事件委托来处理表格中任何 `<td>` 的进入/离开。因此，它依赖于 `mouseover/out` 而不是 `mouseenter/leave`，`mouseenter/leave` 不会冒泡，因此也不允许事件委托。
2. 额外的事件，例如在 `<td>` 的后代之间移动都会被过滤掉，因此 `onEnter/Leave` 仅在鼠标指针进入/离开 `<td>` 整体时才会运行。

### 三、鼠标拖放事件

拖放（Drag’n’Drop）是一个很赞的界面解决方案。取某件东西并将其拖放是执行许多东西的一种简单明了的方式，从复制和移动文档（如在文件管理器中）到订购（将物品放入购物车）。

在现代 HTML 标准中有一个 [关于拖放的部分](https://html.spec.whatwg.org/multipage/interaction.html#dnd)，其中包含了例如 `dragstart` 和 `dragend` 等特殊事件。

这些事件能够支持特殊类型的拖放，例如处理从 OS 文件管理器中拖动文件，并将其拖放到浏览器窗口中。之后，JavaScript 便可以访问此类文件中的内容。

但是，原生的拖放事件也有其局限性。例如，无法阻止从特定区域的拖动。并且，无法将拖动变成“水平”或“竖直”的。还有很多其他使用它们无法完成的拖放任务。并且，移动设备对此类事件的支持非常有限。

因此需要使用鼠标事件来实现拖放。

#### 3.1 拖放算法

基础的拖放算法如下所示：

1. 在 `mousedown` 上 —— 根据需要准备要移动的元素（也许创建一个它的副本，向其中添加一个类或其他任何东西）。
2. 然后在 `mousemove` 上，通过更改 `position:absolute` 情况下的 `left/top` 来移动它。
3. 在 `mouseup` 上 —— 执行与完成的拖放相关的所有行为。

下面是拖放一个球的实现代码：

```javascript
ball.onmousedown = function(event) {
  // (1) 准备移动：确保 absolute，并通过设置 z-index 以确保球在顶部
  ball.style.position = 'absolute';
  ball.style.zIndex = 1000;

  // 将其从当前父元素中直接移动到 body 中
  // 以使其定位是相对于 body 的
  document.body.append(ball);

  // 现在球的中心在 (pageX, pageY) 坐标上
  function moveAt(pageX, pageY) {
    ball.style.left = pageX - ball.offsetWidth / 2 + 'px';
    ball.style.top = pageY - ball.offsetHeight / 2 + 'px';
  }

  // 将我们绝对定位的球移到指针下方
  moveAt(event.pageX, event.pageY);

  function onMouseMove(event) {
    moveAt(event.pageX, event.pageY);
  }

  // (2) 在 mousemove 事件上移动球
  document.addEventListener('mousemove', onMouseMove);

  // (3) 放下球，并移除不需要的处理程序
  ball.onmouseup = function() {
    document.removeEventListener('mousemove', onMouseMove);
    ball.onmouseup = null;
  };

};
```

如果运行这段代码，会发现一些奇怪的事情。在拖放的一开始，球“分叉”了：拖动的是它的“克隆”。

这是因为浏览器有自己的对图片和一些其他元素的拖放处理。它会在我们进行拖放操作时自动运行，并与自定义的拖放处理产生了冲突。

禁用它：

```javascript
ball.ondragstart = function() {
  return false;
};
```

现在一切都正常了。

另一个重要的方面是 —— 在 `document` 上跟踪 `mousemove`，而不是在 `ball` 上。乍一看，鼠标似乎总是在球的上方，我们可以将 `mousemove` 放在球上。

`mousemove` 会经常被触发，但不会针对每个像素都如此。因此，在快速移动鼠标后，鼠标指针可能会从球上跳转至文档中间的某个位置（甚至跳转至窗口外）。

因此，应该监听 `document` 以捕获它。

#### 3.2 修正定位

在上述示例中，球在移动时，球的中心始终位于鼠标指针下方：

```javascript
ball.style.left = pageX - ball.offsetWidth / 2 + 'px';
ball.style.top = pageY - ball.offsetHeight / 2 + 'px';
```

不错，但这存在副作用。要启动拖放，可以在球上的任意位置 `mousedown`。但是，如果从球的边缘“抓住”球，那么球会突然“跳转”以使球的中心位于鼠标指针下方。

如果能够保持元素相对于鼠标指针的初始偏移，那就更好了。

例如，按住球的边缘处开始拖动，那么在拖动时，鼠标指针应该保持在一开始所按住的边缘位置上。

![拖拽定位](https://gitee.com/Topcvan//js-notes-img/raw/master/%E6%8B%96%E6%8B%BD%E5%AE%9A%E4%BD%8D.png)

更新算法：

1. 当访问者按下按钮（`mousedown`）时 —— 可以在变量 `shiftX/shiftY` 中记住鼠标指针到球左上角的距离。我们应该在拖动时保持这个距离。

   我们可以通过坐标相减来获取这个偏移：

   ```javascript
   // onmousedown
   let shiftX = event.clientX - ball.getBoundingClientRect().left;
   let shiftY = event.clientY - ball.getBoundingClientRect().top;
   ```

2. 然后，在拖动球时，将鼠标指针相对于球的这个偏移也考虑在内，像这样：

   ```javascript
   // onmousemove
   // 球具有 position:absoute
   ball.style.left = event.pageX - shiftX + 'px';
   ball.style.top = event.pageY - shiftY + 'px';
   ```

能够更好地进行定位的最终代码：

```javascript
ball.onmousedown = function(event) {

  let shiftX = event.clientX - ball.getBoundingClientRect().left;
  let shiftY = event.clientY - ball.getBoundingClientRect().top;

  ball.style.position = 'absolute';
  ball.style.zIndex = 1000;
  document.body.append(ball);

  moveAt(event.pageX, event.pageY);

  // 移动现在位于坐标 (pageX, pageY) 上的球
  // 将初始的偏移考虑在内
  function moveAt(pageX, pageY) {
    ball.style.left = pageX - shiftX + 'px';
    ball.style.top = pageY - shiftY + 'px';
  }

  function onMouseMove(event) {
    moveAt(event.pageX, event.pageY);
  }

  // 在 mousemove 事件上移动球
  document.addEventListener('mousemove', onMouseMove);

  // 放下球，并移除不需要的处理程序
  ball.onmouseup = function() {
    document.removeEventListener('mousemove', onMouseMove);
    ball.onmouseup = null;
  };

};

ball.ondragstart = function() {
  return false;
};
```

#### 3.3 潜在的放置目标

在前面的示例中，球可以被放置（drop）到“任何地方”。在实际中，通常是将一个元素放到另一个元素上。例如，将一个“文件”放置到一个“文件夹”或者其他地方。

抽象地讲，取一个 “draggable” 的元素，并将其放在 “droppable” 的元素上。

这就需要知道：

- 在拖放结束时，所拖动的元素要放在哪里 —— 执行相应的行为
- 并且，最好知道所拖动到的 “droppable” 的元素的位置，并高亮显示 “droppable” 的元素。

这个解决方案很有意思，只是有点麻烦，所以在这儿对此进行介绍。

第一个想法是什么？可能是将 `onmouseover/mouseup` 处理程序放在潜在的 “droppable” 的元素中？

但这行不通。

问题在于，当拖动时，可拖动元素一直是位于其他元素上的。而鼠标事件只发生在顶部元素上，而不是发生在那些下面的元素。

例如，下面有两个 `<div>` 元素，红色的在蓝色的上面（完全覆盖）。这里，在蓝色的 `<div>` 中没有办法来捕获事件，因为红色的 `<div>` 在它上面：

```html
<style>
  div {
    width: 50px;
    height: 50px;
    position: absolute;
    top: 0;
  }
</style>
<div style="background:blue" onmouseover="alert('never works')"></div>
<div style="background:red" onmouseover="alert('over red!')"></div>
```

与可拖动的元素相同。球始终位于其他元素之上，因此事件会发生在球上。所以无论在较低的元素上设置什么处理程序，它们都不会起作用。

这就是一开始的那个想法，将处理程序放在潜在的 “droppable” 的元素，在实际操作中不起作用的原因。它们不会运行。

那么，该怎么办？

有一个叫做 `document.elementFromPoint(clientX, clientY)` 的方法。它会返回在给定的窗口相对坐标处的嵌套的最深的元素（如果给定的坐标在窗口外，则返回 `null`）。

可以在任何鼠标事件处理程序中使用它，以检测鼠标指针下的潜在的 “droppable” 的元素，就像这样：

```javascript
// 在一个鼠标事件处理程序中
ball.hidden = true; // (*) 隐藏我们拖动的元素

let elemBelow = document.elementFromPoint(event.clientX, event.clientY);
// elemBelow 是球下方的元素，可能是 droppable 的元素

ball.hidden = false;
```

请注意：需要在调用 `(*)` 之前隐藏球。否则，通常会在这些坐标上有一个球，因为它是在鼠标指针下的最顶部的元素：`elemBelow=ball`。

可以使用该代码来检查正在“飞过”的元素是什么。并在放置（drop）时，对放置进行处理。

基于 `onMouseMove` 扩展的代码，用于查找 “droppable” 的元素：

```javascript
// 我们当前正在飞过的潜在的 droppable 的元素
let currentDroppable = null;

function onMouseMove(event) {
  moveAt(event.pageX, event.pageY);

  ball.hidden = true;
  let elemBelow = document.elementFromPoint(event.clientX, event.clientY);
  ball.hidden = false;

  // mousemove 事件可能会在窗口外被触发（当球被拖出屏幕时）
  // 如果 clientX/clientY 在窗口外，那么 elementfromPoint 会返回 null
  if (!elemBelow) return;

  // 潜在的 droppable 的元素被使用 "droppable" 类进行标记（也可以是其他逻辑）
  let droppableBelow = elemBelow.closest('.droppable');

  if (currentDroppable != droppableBelow) {
    // 我们正在飞入或飞出...
    // 注意：它们两个的值都可能为 null
    //   currentDroppable=null —— 如果我们在此事件之前，鼠标指针不是在一个 droppable 的元素上（例如空白处）
    //   droppableBelow=null —— 如果现在，在当前事件中，我们的鼠标指针不是在一个 droppable 的元素上

    if (currentDroppable) {
      // 处理“飞出” droppable 的元素时的处理逻辑（移除高亮）
      leaveDroppable(currentDroppable);
    }
    currentDroppable = droppableBelow;
    if (currentDroppable) {
      // 处理“飞入” droppable 的元素时的逻辑
      enterDroppable(currentDroppable);
    }
  }
}
```

### 四、指针事件

指针事件（Pointer Events）是一种用于处理来自各种输入设备（例如鼠标、触控笔和触摸屏等）的输入信息的现代化解决方案。

**一段简史**

- 很早以前，只存在鼠标事件。

  后来，触屏设备开始普及，尤其是手机和平板电脑。为了使现有的脚本仍能正常工作，它们生成（现在仍生成）鼠标事件。例如，轻触屏幕就会生成 `mousedown` 事件。因此，触摸设备可以很好地与网页配合使用。

  但是，触摸设备比鼠标具有更多的功能。例如，可以同时触控多点（多点触控）。然而，鼠标事件并没有相关属性来处理这种多点触控。

- 因此，引入了触摸事件，例如 `touchstart`、`touchend` 和 `touchmove`，它们具有特定于触摸的属性（这里不再赘述这些特性，因为指针事件更加完善）。

  不过这还是不够完美。因为很多其他输入设备（如触控笔）都有自己的特性。而且同时维护两份分别处理鼠标事件和触摸事件的代码，显得有些笨重了。

- 为了解决这些问题，人们引入了全新的规范「指针事件」。它为各种指针输入设备提供了一套统一的事件。

目前，各大主流浏览器已经支持了 [Pointer Events Level 2](https://www.w3.org/TR/pointerevents2/) 标准，版本更新的 [Pointer Events Level 3](https://w3c.github.io/pointerevents/) 已经发布，并且大多数情况下与 Pointer Events Level 2 兼容。

因此，除非代码需要兼容旧版本的浏览器，例如 IE 10 或 Safari 12 或更低的版本，否则无需继续使用鼠标事件或触摸事件 —— 可以使用指针事件。

这样，代码就可以在触摸设备和鼠标设备上都能正常工作了。

话虽如此，指针事件仍然有一些重要的奇怪特性，应当对它们有所了解以正确使用指针事件，并避免一些意料之外的错误。

#### 4.1指针事件类型

指针事件的命名方式和鼠标事件类似：

| 指针事件             | 类似的鼠标事件 |
| :------------------- | :------------- |
| `pointerdown`        | `mousedown`    |
| `pointerup`          | `mouseup`      |
| `pointermove`        | `mousemove`    |
| `pointerover`        | `mouseover`    |
| `pointerout`         | `mouseout`     |
| `pointerenter`       | `mouseenter`   |
| `pointerleave`       | `mouseleave`   |
| `pointercancel`      | -              |
| `gotpointercapture`  | -              |
| `lostpointercapture` | -              |

不难发现，每一个 `mouse<event>` 都有与之相对应的 `pointer<event>`。同时还有 3 个额外的事件没有相应的 `mouse...`。

**在代码中用 `pointer<event>` 替换 `mouse<event>`**

可以把代码中的 `mouse<event>` 都替换成 `pointer<event>`，程序仍然正常兼容鼠标设备。

替换之后，程序对触屏设备的支持会“魔法般”地提升。但是，可能需要在 CSS 中的某些地方添加 `touch-action: none`。

#### 4.2 指针事件属性

指针事件具备和鼠标事件完全相同的属性，包括 `clientX/Y` 和 `target` 等，以及一些其他属性：

- `pointerId` —— 触发当前事件的指针唯一标识符。

  浏览器生成的。使我们能够处理多指针的情况，例如带有触控笔和多点触控功能的触摸屏（下文会有相关示例）。

- `pointerType` —— 指针的设备类型。必须为字符串，可以是：“mouse”、“pen” 或 “touch”。可以使用这个属性来针对不同类型的指针输入做出不同响应。

- `isPrimary` —— 当指针为首要指针（多点触控时按下的第一根手指）时为 `true`。

有些指针设备会测量接触面积和点按压力（例如一根手指压在触屏上），对于这种情况可以使用以下属性：

- `width` —— 指针（例如手指）接触设备的区域的宽度。对于不支持的设备（如鼠标），这个值总是 `1`。
- `height` —— 指针（例如手指）接触设备的区域的长度。对于不支持的设备，这个值总是 `1`。
- `pressure` —— 触摸压力，是一个介于 0 到 1 之间的浮点数。对于不支持压力检测的设备，这个值总是 `0.5`（按下时）或 `0`。
- `tangentialPressure` —— 归一化后的切向压力（tangential pressure）。
- `tiltX`, `tiltY`, `twist` —— 针对触摸笔的几个属性，用于描述笔和屏幕表面的相对位置。

大多数设备都不支持这些属性，因此它们很少被使用。如果需要使用它们，可以在 [规范文档](https://w3c.github.io/pointerevents/#pointerevent-interface) 中查看更多有关它们的详细信息。

#### 4.3 多点触控

多点触控（用户在手机或平板上同时点击若干个位置，或执行特殊手势）是鼠标事件完全不支持的功能之一。

指针事件能够通过 `pointerId` 和 `isPrimary` 属性的帮助，处理多点触控。

当用户用一根手指触摸触摸屏的某个位置，然后将另一根手指放在该触摸屏的其他位置时，会发生以下情况：

1. 第一个手指触摸：
   - `pointerdown` 事件触发，`isPrimary=true`，并且被指派了一个 `pointerId`。
2. 第二个和后续的更多个手指触摸（假设第一个手指仍在触摸）：
   - `pointerdown` 事件触发，`isPrimary=false`，并且每一个触摸都被指派了不同的 `pointerId`。

请注意：`pointerId` 不是分配给整个设备的，而是分配给每一个触摸的。如果 5 根手指同时触摸屏幕，会得到 5 个 `pointerdown` 事件和相应的坐标以及 5 个不同的 `pointerId`。

和第一个触摸相关联的事件总有 `isPrimary=true`。

利用 `pointerId`，可以追踪多根正在触摸屏幕的手指。当用户移动或抬起某根手指时，会得到和 `pointerdown` 事件具有相同 `pointerId` 的 `pointermove` 或 `pointerup` 事件。

#### 4.4 `pointercancel`

`pointercancel` 事件将会在一个正处于活跃状态的指针交互由于某些原因被中断时触发。也就是在这个事件之后，该指针就不会继续触发更多事件了。

导致指针中断的可能原因如下：

- 指针设备硬件在物理层面上被禁用。
- 设备方向旋转（例如给平板转了个方向）。
- 浏览器打算自行处理这一交互，比如将其看作是一个专门的鼠标手势或缩放操作等。

例如，想要实现一个像 [鼠标拖放事件](https://zh.javascript.info/mouse-drag-and-drop) 中开头提到的那样的一个对球的拖放操作。

用户的操作流和对应的事件如下：

1. 用户按住了一张图片，开始拖拽
   - `pointerdown` 事件触发
2. 用户开始移动指针（从而拖动图片）
   - `pointermove` 事件触发，可能触发多次
3. 然后意料之外的情况发生了！浏览器有自己原生的图片拖放操作，接管了之前的拖放过程，于是触发了`pointercancel`事件。
   - 现在拖放图片的操作由浏览器自行实现。用户甚至可能会把图片拖出浏览器，放进他们的邮件程序或文件管理器。
   - 不会再得到 `pointermove` 事件了。

这里的问题就在于浏览器”劫持“了这一个互动操作：在“拖放”过程开始时触发了 `pointercancel` 事件，并且不再有 `pointermove` 事件会被生成。

**阻止浏览器的默认行为来防止 `pointercancel` 触发。**

需要做两件事：

1. 阻止原生的拖放操作发生：
   - 正如在 [鼠标拖放事件](https://zh.javascript.info/mouse-drag-and-drop) 中描述的那样，可以通过设置 `ball.ondragstart = () => false` 来实现这一需求。
   - 这种方式也适用于鼠标事件。
2. 对于触屏设备，还有其他和触摸相关的浏览器行为（除了拖放）。为了避免它们所引发的问题：
   - 可以通过在 CSS 中设置 `#ball { touch-action: none }` 来阻止它们。
   - 之后代码便可以在触屏设备中正常工作了。

经过上述操作，事件将会按照预期的方式触发，浏览器不会劫持拖放过程，也不会触发 `pointercancel` 事件。

#### 4.5 指针捕获

指针捕获（Pointer capturing）是针对指针事件的一个特性。

这个想法很简单，但是乍一看可能感觉很奇怪，因为在其他任何事件类型中都没有这种东西。

主要的方法是：

- `elem.setPointerCapture(pointerId)` —— 将给定的 `pointerId` 绑定到 `elem`。在调用之后，所有具有相同 `pointerId` 的指针事件都将 `elem` 作为目标（就像事件发生在 `elem` 上一样），无论这些 `elem` 在文档中的实际位置是什么。

换句话说，`elem.setPointerCapture(pointerId)` 将所有具有给定 `pointerId` 的后续事件重新定位到 `elem`。

绑定会在以下情况下被移除：

- 当 `pointerup` 或 `pointercancel` 事件出现时，绑定会被自动地移除。
- 当 `elem` 被从文档中移除后，绑定会被自动地移除。
- 当 `elem.releasePointerCapture(pointerId)` 被调用，绑定会被移除。

**指针捕获可以被用于简化拖放类的交互。**

`setPointerCapture` 适用的场景。

- 可以在 `pointerdown` 事件的处理程序中调用 `thumb.setPointerCapture(event.pointerId)`，
- 这样接下来在 `pointerup/cancel` 之前发生的所有指针事件都会被重定向到 `thumb` 上。
- 当 `pointerup` 发生时（拖动完成），绑定会被自动移除，不需要关心它。

因此，即使用户在整个文档上移动指针，事件处理程序也将仅在 `thumb` 上被调用。尽管如此，事件对象的坐标属性，例如 `clientX/clientY` 仍将是正确的 —— 捕获仅影响 `target/currentTarget`。

**指针捕获事件**

还有两个与指针捕获相关的事件：

- `gotpointercapture` 会在一个元素使用 `setPointerCapture` 来启用捕获后触发。
- `lostpointercapture` 会在捕获被释放后触发：其触发可能是由于 `releasePointerCapture` 的显式调用，或是 `pointerup`/`pointercancel` 事件触发后的自动调用。

### 五、键盘事件

当想要处理键盘行为时，应该使用键盘事件（虚拟键盘也算）。例如，对方向键 Up 和 Down 或热键（包括按键的组合）作出反应。

#### 5.1 `keydown&keyup`

事件对象的 `key` 属性允许获取字符，而事件对象的 `code` 属性则允许获取“物理按键代码”。

例如，同一个按键 Z，可以与或不与 `Shift` 一起按下。我们会得到两个不同的字符：小写的 `z` 和大写的 `Z`。

`event.key` 正是这个字符，并且它将是不同的。但是，`event.code` 是相同的：

| Key     | `event.key` | `event.code` |
| :------ | :---------- | :----------- |
| Z       | `z`（小写） | `KeyZ`       |
| Shift+Z | `Z`（大写） | `KeyZ`       |

如果用户使用不同的语言，那么切换到另一种语言将产生完全不同的字符，而不是 `"Z"`。它将成为 `event.key` 的值，而 `event.code` 则始终都是一样的：`"KeyZ"`。

**“KeyZ” 和其他按键代码**

每个按键的代码都取决于该按键在键盘上的位置。[UI 事件代码规范](https://www.w3.org/TR/uievents-code/) 中描述了按键代码。

例如：

- 字符键的代码为 `"Key<letter>"`：`"KeyA"`，`"KeyB"` 等。
- 数字键的代码为：`"Digit<number>"`：`"Digit0"`，`"Digit1"` 等。
- 特殊按键的代码为按键的名字：`"Enter"`，`"Backspace"`，`"Tab"` 等。

有几种广泛应用的键盘布局，该规范给出了每种布局的按键代码。

有关更多按键代码，参见 [规范的字母数字部分](https://www.w3.org/TR/uievents-code/#key-alphanumeric-section)。

**大小写敏感：`"KeyZ"`，不是 `"keyZ"`**

它是 `KeyZ`，而不是 `keyZ`。像 `event.code=="keyZ"` 这样的检查不起作用：`"Key"` 的首字母必须大写。

如果按键没有给出任何字符呢？例如，Shift 或 F1 或其他。对于这些按键，它们的 `event.key` 与 `event.code` 大致相同：

| Key       | `event.key` | `event.code`                |
| :-------- | :---------- | :-------------------------- |
| F1        | `F1`        | `F1`                        |
| Backspace | `Backspace` | `Backspace`                 |
| Shift     | `Shift`     | `ShiftRight` 或 `ShiftLeft` |

`event.code` 准确地标明了哪个键被按下了。例如，大多数键盘有两个 Shift 键，一个在左边，一个在右边。`event.code` 会准确地告诉我们按下了哪个键，而 `event.key` 对按键的“含义”负责：它是什么（一个 “Shift”）。

假设，要处理一个热键：Ctrl+Z（或 Mac 上的 Cmd+Z）。大多数文本编辑器将“撤销”行为挂在其上。可以在 `keydown` 上设置一个监听器，并检查哪个键被按下了。

这里有个难题：在这样的监听器中，应该检查 `event.key` 的值还是 `event.code` 的值？

一方面，`event.key` 的值是一个字符，它随语言而改变。如果访问者在 OS 中使用多种语言，并在它们之间进行切换，那么相同的按键将给出不同的字符。因此检查 `event.code` 会更好，因为它总是相同的。

像这样：

```javascript
document.addEventListener('keydown', function(event) {
  if (event.code == 'KeyZ' && (event.ctrlKey || event.metaKey)) {
    alert('Undo!')
  }
});
```

另一方面，`event.code` 有一个问题。对于不同的键盘布局，相同的按键可能会具有不同的字符。

例如，下面是美式布局（“QWERTY”）和德式布局（“QWERTZ”）：

![键盘](https://gitee.com/Topcvan//js-notes-img/raw/master/%E9%94%AE%E7%9B%98.png)

对于同一个按键，美式布局为 “Z”，而德式布局为 “Y”（字母被替换了）。

从字面上看，对于使用德式布局键盘的人来说，但他们按下 Y 时，`event.code` 将等于 `KeyZ`。

如果在代码中检查 `event.code == 'KeyZ'`，那么对于使用德式布局键盘的人来说，当他们按下 Y 时，这个测试就通过了。

听起来确实很怪，但事实确实如此。[规范](https://www.w3.org/TR/uievents-code/#table-key-code-alphanumeric-writing-system) 中明确提到了这种行为。

因此，`event.code` 可能由于意外的键盘布局而与错误的字符进行了匹配。不同键盘布局中的相同字母可能会映射到不同的物理键，从而导致了它们有不同的代码。幸运的是，这种情况只发生在几个代码上，例如 `keyA`，`keyQ`，`keyZ`，而对于诸如 `Shift` 这样的特殊按键没有发生这种情况。可以在 [规范](https://www.w3.org/TR/uievents-code/#table-key-code-alphanumeric-writing-system) 中找到该列表。

为了可靠地跟踪与受键盘布局影响的字符，使用 `event.key` 可能是一个更好的方式。

另一方面，`event.code` 的好处是，即使访问者更改了语言，绑定到物理键位置的 `event.code` 会始终保持不变。因此，即使在切换了语言的情况下，依赖于它的热键也能正常工作。

如果想要处理与布局有关的按键，那么 `event.key` 是必选的方式。如果希望一个热键即使在切换了语言后，仍能正常使用，那么 `event.code` 可能会更好。

#### 5.2 自动重复

如果按下一个键足够长的时间，它就会开始“自动重复”：`keydown` 会被一次又一次地触发，然后当按键被释放时，我们最终会得到 `keyup`。因此，有很多 `keydown` 却只有一个 `keyup` 是很正常的。

对于由自动重复触发的事件，`event` 对象的 `event.repeat` 属性被设置为 `true`。

#### 5.3 默认行为

默认行为各不相同，因为键盘可能会触发很多可能的东西。

例如：

- 出现在屏幕上的一个字符（最明显的结果）。
- 一个字符被删除（Delete 键）。
- 滚动页面（PageDown 键）。
- 浏览器打开“保存页面”对话框（Ctrl+S）
- ……等。

阻止对 `keydown` 的默认行为可以取消大多数的行为，但基于 OS 的特殊按键除外。例如，在 Windows 中，Alt+F4 会关闭当前浏览器窗口。并且无法通过在 JavaScript 中阻止默认行为来阻止它。

例如，下面的这个 `<input>` 期望输入的内容为一个电话号码，因此它不会接受除数字，`+`，`()` 和 `-` 以外的按键：

```html
<script>
function checkPhoneKey(key) {
  return (key >= '0' && key <= '9') || ['+','(',')','-'].includes(key);
}
</script>
<input onkeydown="return checkPhoneKey(event.key)" placeholder="请输入手机号" type="tel">
```

这里 `onkeydown` 的处理程序使用 `checkPhoneKey` 来检查被按下的按键。如果它是有效的（`0..9` 或 `+-()` 之一），那么将返回 `true`，否则返回 `false`。

像上面那样，从事件处理程序返回 `false` 会阻止事件的默认行为，所以如果按下的按键未通过按键检查，那么 `<input>` 中什么都不会出现（从事件处理程序返回 `true` 不会对任何行为产生影响，只有返回 `false` 会产生对应的影响）。但像 Backspace，Left，Right，Ctrl+V 这样的特殊按键在输入中也会无效。这是严格过滤器 `checkPhoneKey` 的副作用。

将过滤条件放松一下：

```html
<script>
function checkPhoneKey(key) {
  return (key >= '0' && key <= '9') ||
    ['+','(',')','-','ArrowLeft','ArrowRight','Delete','Backspace'].includes(key);
}
</script>
<input onkeydown="return checkPhoneKey(event.key)" placeholder="Phone, please" type="tel">
```

现在方向键和删除键都能正常使用了。

但仍然可以使用鼠标右键单击 + 粘贴来输入任何内容。因此，这个过滤器并不是 100% 可靠。可以让它就这样吧，因为大多数情况下它是有效的。或者，另一种方式是跟踪 `input` 事件 —— 在任何修改后触发，这样就可以检查新值，并在其无效时高亮/修改它。

#### 5.4 遗存

过去曾经有一个 `keypress` 事件，还有事件对象的 `keyCode`、`charCode` 和 `which` 属性。

大多数浏览器对它们都存在兼容性问题，以致使该规范的开发者不得不弃用它们并创建新的现代的事件（本文上面所讲的这些事件），除此之外别无选择。旧的代码仍然有效，因为浏览器还在支持它们，但现在完全没必要再使用它们。

### 六、滚动

`scroll` 事件允许对页面或元素滚动作出反应。可以在这里做一些有用的事情。

例如：

- 根据用户在文档中的位置显示/隐藏其他控件或信息。
- 当用户向下滚动到页面末端时加载更多数据。

这是一个显示当前滚动的小函数：

```javascript
window.addEventListener('scroll', function() {
  document.getElementById('showScroll').innerHTML = window.pageYOffset + 'px';
});
```

`scroll` 事件在 `window` 和可滚动元素上都可以运行。

#### 6.1 防止滚动

如何使某些东西变成不可滚动？

不能通过在 `onscroll` 监听器中使用 `event.preventDefault()` 来阻止滚动，因为它会在滚动发生 **之后** 才触发。

但是可以在导致滚动的事件上，例如在 pageUp 和 pageDown 的 `keydown` 事件上，使用 `event.preventDefault()` 来阻止滚动。

如果向这些事件中添加事件处理程序，并向其中添加 `event.preventDefault()`，那么滚动就不会开始。

启动滚动的方式有很多，使用 CSS 的 `overflow` 属性更加可靠。