### 一、grid布局简介

跟Flexbox 类似，网格布局也是作用于两级的DOM 结构。设置为display: grid 的元素
成为一个网格容器（grid container）。它的子元素则变成网格元素（grid items）

```css
.grid { 
  display: grid;  
  grid-template-columns: 1fr 1fr 1fr;  
  grid-template-rows: 1fr 1fr;  
  grid-gap: 0.5em;  
} 
 
.grid > * { 
  background-color: darkgray; 
  color: white; 
  padding: 2em; 
  border-radius: 0.5em; 
} 
```

在支持网格布局的浏览器中，这段代码会渲染三列，共六个大小相等的盒子

首先，使用 display: grid 定义一个网格容器。容器会表现得像一个块级元素，100%填充可用宽度。也可以使用inline-grid（尽管这段代码没写），这样元素就会在行内流动，且宽度只能够包含子元素，不过inline-grid 的使用频率不高

接下来是新属性：grid-template-columns 和 grid-template-rows。这两个属性定义了网格每行每列的大小。本例使用了一种新单位 fr，代表每一列（或每一行）的分数单位（fraction  unit）。这个单位跟Flexbox 中flex-grow 因子的表现一样。grid-template-columns: 1fr 1fr 1fr 表示三列等宽。 

不一定非得用分数单位，可以使用其他的单位，比如px、em 或百分数。也可以混搭这几种
单位，例如，grid-template-columns: 300px 1fr 定义了一个固定宽度为 300px 的列，后
面跟着一个会填满剩余可用空间的列。2fr 的列宽是1fr 的两倍。 

最后，grid-gap 属性定义了每个网格单元之间的间距。也可以用两个值分别指定垂直和水
平方向的间距（比如grid-gap: 0.5em 1em）

### 二、网格剖析

理解网格的各个部分很重要。前面已经提及网格容器和网格元素，这些是网格布局的基本元
素。还有另外四个重要的概念：

-  网格线（grid line）——网格线构成了网格的框架。一条网格线可以水平或垂直，并且位于一行或一列的任意一侧。如果指定了grid-gap 的话，它就位于网格线上。 
- 网格轨道（grid track）——一个网格轨道是两条相邻网格线之间的空间。网格有水平轨道（行）和垂直轨道（列）。 
- 网格单元（grid cell）——网格上的单个空间，水平和垂直的网格轨道交叉重叠的部分。
- 网格区域（grid area）——网格上的矩形区域，由一个到多个网格单元组成。该区域位于两条垂直网格线和两条水平网格线之间

![网格组成部分](https://gitee.com/Topcvan/img-storage/raw/master/css/%E7%BD%91%E6%A0%BC%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86.png)

构建网格布局时会涉及这些组成部分。比如声明grid-template-columns: 1fr 1fr 1fr就会定义三个等宽且垂直的网格轨道，同时还定义了四条垂直的网格线：一条在网格最左边，两条在每个网格轨道之间，还有一条在最右边。

使用网格布局：

```css
.container { 
  display: grid; 
  grid-template-columns: 2fr 1fr;  
  grid-template-rows: repeat(4, auto);  
  grid-gap: 1.5em; 
  max-width: 1080px; 
  margin: 0 auto; 
}

header, 
nav { 
  grid-column: 1 / 3;  
  grid-row: span 1;  
} 
 
.main { 
  grid-column: 1 / 2;  
  grid-row: 3 / 5; 
} 
 
.sidebar-top { 
  grid-column: 2 / 3; 
  grid-row: 3 / 4; 
} 
 
.sidebar-bottom { 
  grid-column: 2 / 3; 
  grid-row: 4 / 5;  
}
```

代码首先设置了网格容器，并用 grid-template-columns 和 grid-template-rows 定义了网格轨道。因为列的分数单位分别是2fr 和1fr，所以第一列的宽度是第二列的两倍。定义行的时候用到了一个新方法：repeat()函数。它在声明多个网格轨道的时候提供了简写方式。

grid-template-rows: repeat(4, auto);定义了四个水平网格轨道，高度为auto，这等价于grid-template-rows: auto auto auto auto。轨道大小设置为auto，轨道会根据自身内容扩展。 

用 repeat()符号还可以定义不同的重复模式，比如 repeat(3, 2fr 1fr)会重复三遍这个模式，从而定义六个网格轨道，重复的结果是2fr 1fr 2fr 1fr 2fr 1fr。

还可以将repeat()作为一个更长的模式的一部分。比如grid-template-columns: 1fr repeat(3, 3fr) 1fr 定义了一个 1fr 的列，接着是三个 3fr 的列，最后还有一个 1fr 的列（可以用1fr 3fr 3fr 3fr 1fr 表示）。可以看出来因为展开的写法无法一目了然，所以才产生了repeat()这种简写方式。

#### 2.1网格线的编号

网格轨道定义好之后，要将每个网格元素放到特定的位置上。浏览器给网格里的每个网格线
都赋予了编号，如图所示。CSS 用这些编号指出每个元素应该摆放的位置：

![网格线的编号](https://gitee.com/Topcvan/img-storage/raw/master/css/%E7%BD%91%E6%A0%BC%E7%BA%BF%E7%9A%84%E7%BC%96%E5%8F%B7.png)

可以在grid-column 和grid-row 属性中用网格线的编号指定网格元素的位置。如果想要一个网格元素在垂直方向上跨越1 号网格线到3 号网格线，就需要给元素设置grid-column: 1 / 3。或者设置grid-row: 3 / 5 让元素在水平方向上跨越3 号网格线到5 号网格线。这两个属性一起就能指定一个元素应该放置的网格区域。

这些属性实际上是简写属性：grid-column 是 grid-column-start 和grid-column-end 的简写；grid-row 是grid-row-start 和grid-row-end 的简写。中间的斜线只在简写属性里用于区分两个值，斜线前后的空格不作要求

定位header 和nav 的规则集稍有变化。以下代码片段用相同的规则集同时布局这两者。 

```css
header, 
nav { 
  grid-column: 1 / 3; 
  grid-row: span 1; 
} 
```

代码里使用之前介绍的grid-column 写法，让网格元素占满网格的宽度。其实还可以用一个特别的关键字span 来指定grid-row 和grid-column 的值（这里用在了grid-row 上）。这个关键字告诉浏览器元素需要占据一个网格轨道。因为这里没有指出具体是哪一行，所以会根据网格元素的布局算法（placement  algorithm）自动将其放到合适的位置。布局算法会将元素放在网格上可以容纳该元素的第一处可用空间，本例中是第一行和第二行。

#### 2.2 与flexbox配合

Flexbox和Grid是互补的。二者几乎是一起开发出来的，虽然它们的功能有一些重叠的地方，但是它们各自擅长的场景不一样。在一个设计场景里，要根据特定的需求来做出选择。这两种布局方式有以下两个重要区别： 

- Flexbox 本质上是一维的，而网格是二维的。 
- Flexbox 是以内容为切入点由内向外工作的，而网格是以布局为切入点从外向内工作的。 

因为 Flexbox 是一维的，所以它很适合用在相似的元素组成的行（或列）上。它支持用flex-wrap 换行，但是没法让上一行元素跟下一行元素对齐。相反，网格是二维的，旨在解决一个轨道的元素跟另一个轨道的元素对齐的问题。

按照CSS  WG 的成员Rachel  Andrew 的说法，它们的第二个区别在于，Flexbox 以内容为切
入点由内向外工作，而网格以布局为切入点由外向内工作。Flexbox 让你在一行或一列中安排一系列元素，但是它们的大小不需要明确指定，每个元素占据的大小根据自身的内容决定。而在网格中，首先要描述布局，然后将元素放在布局结构中去。虽然每个网格元素的内容都能影响其网格轨道的大小，但是这同时也会影响整个轨道的大小，进而影响这个轨道里的其他网格元素的大小。 

用网格给网页的主区域定位是因为我们希望内容能限制在它所在的网格内，但是对于网页上
的其他元素，比如导航菜单，则允许内容对布局有更大的影响。也就是说，文字多的元素可以宽一些，文字少的元素则可以窄一些。同时这还是一个水平（一维）布局。因此，用Flexbox 来处理这些元素更合适。

当设计要求元素在两个维度上都对齐时，使用网格。当只关心一维的元素排列时，使用Flexbox。在实践中，这通常（并非总是）意味着网格更适合用于整体的网页布局，而Flexbox 更适合对网格区域内的特定元素布局。

### 三、替代语法

布局网格元素还有另外两个替代语法：命名的网格线和命名的网格区域。至于选择哪个纯属个人偏好。在某些设计中，一种语法会比另一种语法更好理解。

#### 3.1 命名的网格线

有时候记录所有网格线的编号实在太麻烦了，尤其是在处理很多网格轨道时。为了能简单点，可以给网格线命名，并在布局时使用网格线的名称而不是编号。声明网格轨道时，可以在中括号内写上网格线的名称，如下代码片段所示：

```css
grid-template-columns: [start] 2fr [center] 1fr [end]; 
```

这条声明定义了两列的网格，三条垂直的网格线分别叫作 start、center 和 end。之后定义网格元素在网格中的位置时，可以不用编号而是用这些名称来声明，如下代码所示： 

```css
grid-column: start / center; 
```

这条声明将网格元素放在1 号网格线（start）到 2 号网格线（center）之间的区域。还可以给同一个网格线提供多个名称，比如下面的声明（为了可读性，这里将代码换行了）。 

```css
grid-template-columns: [left-start] 2fr 
                       [left-end right-start] 1fr 
                       [right-end]; 
```

在这条声明里，2 号网格线既叫作 left-end 也叫作 right-start，之后可以任选一个名称使用。
这里还有一个彩蛋：将网格线命名为left-start 和left-end，就定义了一个叫作left 的区域，这个区域覆盖两个网格线之间的区域。-start 和-end 后缀作为关键字，定义了两者之间的区域。如果给元素设置grid-column: left，它就会跨越从left-start 到left-end 的区域。

用命名的网格线实现网格布局：

```css
.container { 
  display: grid; 
  grid-template-columns: [left-start] 2fr  
                         [left-end right-start] 1fr 
                         [right-end];  
  grid-template-rows: repeat(4, [row] auto);  
  grid-gap: 1.5em; 
  max-width: 1080px; 
  margin: 0 auto; 
} 
 
header, 
nav { 
  grid-column: left-start / right-end; 
  grid-row: span 1; 
} 
 
.main { 
  grid-column: left;  
  grid-row: row 3 / span 2;  
} 
 
.sidebar-top { 
  grid-column: right;  
  grid-row: 3 / 4; 
} 
 
.sidebar-bottom { 
  grid-column: right; 
  grid-row: 4 / 5; 
}
```

这个例子用命名的网格线将每个元素放在的相应网格列上，并且在repeat()里声明了一条命名的水平网格线，于是每条水平网格线被命名为row（除了最后一条）。这看起来很不可思议，但是重复使用同一个名称完全合法。然后将main 元素放在从row  3（第三个叫row 的网格线）开始的地方，并跨越两个网格轨道。 

可以以各种方式命名的网格线。它们在网格里的用法也是五花八门，这取决于每个网格特定
的结构，比如可以实现如图所示的布局：

![特殊的命名网格线](https://gitee.com/Topcvan/img-storage/raw/master/css/%E7%89%B9%E6%AE%8A%E7%9A%84%E5%91%BD%E5%90%8D%E7%BD%91%E6%A0%BC%E7%BA%BF.png)

这个场景展示了一种重复模式：每两个网格列为一组，在每组的两个网格轨道之前命名一条
网格线（grid-template-columns: repeat(3, [col] 1fr 1fr)）。然后就可以借助命名的网格线将一个元素定位到第二组网格列上（grid-column: col 2 / span 2）。

#### 3.2 命名网格区域 

另一个方式是命名网格区域。不用计算或者命名网格线，直接用命名的网格区域将元素定位到网格中。实现这一方法需要借助网格容器的 grid-template-areas 属性和网格元素的grid-area 属性。 

用命名的网格区域 ：

```css
.container { 
  display: grid; 
  grid-template-areas: "title title"  
                       "nav   nav" 
                       "main  aside1" 
                       "main  aside2"; 
  grid-template-columns: 2fr 1fr;  
  grid-template-rows: repeat(4, auto);  
  grid-gap: 1.5em; 
  max-width: 1080px; 
  margin: 0 auto; 
} 
 
header { 
  grid-area: title;  
} 
 
nav { 
  grid-area: nav; 
} 
 
.main { 
  grid-area: main; 
} 
 
.sidebar-top { 
  grid-area: aside1; 
} 
 
.sidebar-bottom { 
  grid-area: aside2;  
}
```

grid-template-areas 属性使用了一种ASCII art 的语法，可以直接在CSS 中画一个可视化的网格形象。该声明给出了一系列加引号字符串，每一个字符串代表网格的一行，字符串内用空格区分每一列。 在这个例子中，第一行完全分配给了网格区域title，第二行则分配给了nav。接下来两行的左列分配给了main，侧边栏的板块分别分配给了aside1 和aside2。用 grid-area 属性将每个网格元素放在这些命名区域中。

注意：每个命名的网格区域必须组成一个矩形。不能创造更复杂的形状，比如L 或者U 型

还可以用句点（.）作为名称，这样便能空出一个网格单元。比如，以下代码定义了四个网格区域，中间围绕着一个空的网格单元。 

```css
grid-template-areas: "top  top    right" 
                     "left .      right" 
                     "left bottom bottom"; 
```

当你构建一个网格时，选择一种舒适的语法即可。网格布局共设计了三种语法：编号的网格
线、命名的网格线、命名的网格区域

### 四、显式和隐式网格

在某些场景下，你可能不清楚该把元素放在网格的哪个位置上。当处理大量的网格元素时，挨个指定元素的位置未免太不方便。当元素是从数据库获取时，元素的个数可能是未知的。在这些情况下，以一种宽松的方式定义网格可能更合理，剩下的交给布局算法来放置网格元素。

这时需要用到隐式网格（implicit grid）。使用grid-template-* 属性定义网格轨道时，创建的是显式网格（explicit  grid），但是有些网格元素仍然可以放在显式轨道外面，此时会自动创建隐式轨道以扩展网格，从而包含这些元素。 

下图中的网格只在每个方向上指定了一个网格轨道。当把网格元素放在第二个轨道（2 号
和3 号网格线之间）时，就会自动创建轨道来包含该元素。

![隐式网格](https://gitee.com/Topcvan/img-storage/raw/master/css/%E9%9A%90%E5%BC%8F%E7%BD%91%E6%A0%BC.png)

隐式网格轨道默认大小为 auto，也就是它们会扩展到能容纳网格元素内容。可以给网格容器设置 grid-auto-columns 和 grid-auto-rows，为隐式网格轨道指定一个大小（比如，grid-auto-columns: 1fr）

注意：在指定网格线的时候，隐式网格轨道不会改变负数的含义。负的网格线编号仍然是从显式网格的右下开始的

#### 4.1 autofill与minmax

有时候我们不想给一个网格轨道设置固定尺寸，但是又希望限制它的最小值和最大值。这时候需要用到minmax()函数。它指定两个值：最小尺寸和最大尺寸。浏览器会确保网格轨道的大小介于这两者之间。（如果最大尺寸小于最小尺寸，最大尺寸就会被忽略。）通过指定minmax(200px, 1fr)，浏览器确保了所有的轨道至少宽200px。

repeat()函数里的auto-fill 关键字是一个特殊值。设置了之后，只要网格放得下，浏览器就会尽可能多地生成轨道，并且不会跟指定大小（minmax()值）的限制产生冲突。 auto-fill 和minmax(200px, 1fr)加在一起，就会让网格在可用的空间内尽可能多地产生网格列，并且每个列的宽度不会小于200px。因为所有轨道的大小上限都为1fr（最大值），所以所有的网格轨道都等宽。

如果网格元素不够填满所有网格轨道，auto-fill 就会导致一些空的网格轨道。如果不希望出现空的网格轨道，可以使用auto-fit 关键字代替auto-fill。它会让非空的网格轨道扩展，填满可用空间。具体选择auto-fill 还是auto-fit 取决于你是想要确保网格轨道的大小，还是希望整个网格容器都被填满

#### 4.2 autoflow与dense

当不指定网格上元素的位置时，元素会按照其布局算法自动放置。默认情况下，布局算法会按元素在标记中的顺序将其逐列逐行摆放。当一个元素无法在某一行容纳（也就是说该元素占据了太多网格轨道）时，算法会将它移动到下一行，寻找足够大的空间容纳它。

网格布局模块规范提供了另一个属性 grid-auto-flow，它可以控制布局算法的行为。它的初始值是row，上一段描述的就是这个值的行为。如果值为column，它就会将元素优先放在网格列中，只有当一列填满了，才会移动到下一行。

还可以额外加一个关键字 dense（比如，grid-auto-flow: column dense）。它让算法紧凑地填满网格里的空白，尽管这会改变某些网格元素的顺序。加上这个关键字，小元素就会“回填”大元素造成的空白区域

#### 4.3 object-fit

默认情况下，一个`<img>`的 object-fit 属性值为fill，也就是说整个图片会缩放，以填满<img>元素。你也可以设置其他值改变默认行为。比如，object-fit 属性的值还可以是cover 和contain。这些值告诉浏览器，在渲染盒子里改变图片的大小，但是不要让图片变形

- cover：扩展图片，让它填满盒子（导致图片一部分被裁剪）。 
- contain：缩放图片，让它完整地填充盒子（导致盒子里出现空白）。 

![object-fit](https://gitee.com/Topcvan//img-storage/raw/master//css/object-fit.png)

这里有两个概念要区分清楚：盒子（由`<img>`元素的宽和高决定）和渲染的图片。默认情况下，这二者大小相等。object-fit 属性让我们能在盒子内部控制渲染图片的大小，同时保持盒子的大小不变。因为用flex-grow 属性拉伸了图片，所以应该给它加上object-fit: cover 防止渲染的图片变形。作为妥协，图片的边缘会被裁掉一部分。

### 五、特性查询

CSS 最近添加了一个叫作特性查询（feature  query）的功能，该功能有助于解决这个问题，
如下代码片段所示。 

```css
@supports (display: grid) { 
  ... 
} 
```

@supports 规则后面跟着一个小括号包围的声明。如果浏览器理解这个声明（在本例中，
浏览器支持网格），它就会使用大括号里面的所有样式规则。如果它不理解小括号里的声明，就不会使用这些样式规则。 也就是说可以提供一份样式规则，它使用较旧的布局技术，比如浮动。这些样式不一定完美（需要做出妥协），但是能实现基本的布局。然后在特性查询中，用网格补全剩下的样式。 

@supports 规则可以用来查询所有的CSS 特性。比如，用@supports (display: flex)
来查询是否支持Flexbox，用@supports (mix-blend-mode: overlay) 来查询是否支持混合
模式

注意：IE 不支持@supports 规则。它忽略了特性查询里的任何规则，不管是否真的支持该特性。通常情况下这是可以接受的，因为让旧版的浏览器渲染回退布局也是情理之中的事情。

特性查询还有以下几种写法。 

- @supports not(<declaration>)——只有当不支持查询声明里的特性时才使用里面的样式规则。
- @supports (<declaration>) or (<declaration>)——查询声明里的两个特性只要有一个支持就使用里面的样式规则。
- @supports (<declaration>) and (<declaration>)——查询声明里的两个特性都支持才使用里面的样式规则。

这些写法还可以结合起来查询更复杂的情况。关键字or 适合查询带浏览器前缀的属性（如下声明所示）。 

```css
@supports (display: grid) or (display: -ms-grid) 
```

这句声明既指定了支持非前缀版本属性的浏览器，也指定了要求用-ms-前缀的旧版Edge 浏
览器。需要注意的是，旧版Edge 对网格的部分支持不如现代浏览器稳健。在旧版Edge 中用特性查询来支持网格布局，可能麻烦比收益更大。因此最好是忽略它，让旧版Edge 渲染为回退布局

### 六、对齐

CSS 给网格布局提供了三个调整属性：justify-content 、justify-items 、justify-self。这些属性控制了网格元素在水平方向上的位置。

还有三个对齐属性：align-content、align-items、align-self。这些属性控制网格
元素在垂直方向上的位置。

![网格对齐属性](https://gitee.com/Topcvan/img-storage/raw/master/css/%E7%BD%91%E6%A0%BC%E5%AF%B9%E9%BD%90%E5%B1%9E%E6%80%A7.png)

可以用justify-content 和align-content 设置网格容器内的网格轨道在水平方向和垂
直方向上的位置，特别是当网格元素的大小无法填满网格容器时。参考以下代码：

```css
.grid { 
  display: grid; 
  height: 1200px; 
  grid-template-rows: repeat(4, 200px); 
} 
```

它明确指定了网格容器的高度为 1200px，但是只定义了高 800px 的有效水平网格轨道。align-content 属性指定了网格轨道如何在剩下的400px 空间内分布。它可以设为以下值

- start——将网格轨道放到网格容器的上/左（Flexbox 里则是flex-start）。 
- end——将网格轨道放在网格容器的下/右（Flexbox 里则是flex-end）。 
- center——将网格轨道放在网格容器的中间。 
- stretch——将网格轨道拉伸至填满网格容器。 
- space-between——将剩余空间平均分配到每个网格轨道之间（它能覆盖任何 grid-gap值）。 
- space-around——将空间分配到每个网格轨道之间，且在两端各加上宽度为间隙宽度一半的间距
- space-evenly——将空间分配到每个网格轨道之间，且在两端各加上同等大小的间距
