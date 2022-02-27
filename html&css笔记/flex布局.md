### 一、布局原理

flex 是 flexible Box 的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性，任何一个容器都可以指定为 flex 布局。当父盒子设为 flex 布局以后，子元素的 float、clear 和 vertical-align 属性将失效。

采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。

![flex布局](https://gitee.com/Topcvan//img-storage/raw/master//css/flex%E5%B8%83%E5%B1%80.png)

### 二、flex布局父项常见属性

#### 2.1 常见父项属性

以下由6个属性是对父元素设置的

- flex-direction：设置主轴的方向
- justify-content：设置主轴上的子元素排列方式
- flex-wrap：设置子元素是否换行  
- align-content：设置侧轴上的子元素的排列方式（多行）
- align-items：设置侧轴上的子元素排列方式（单行）
- flex-flow：复合属性，相当于同时设置了 flex-direction 和 flex-wrap

#### 2.2 flex-direction

在 flex 布局中，是分为主轴和侧轴两个方向，同样的叫法有 ： 行和列、x 轴和y 轴。`flex-direction`用于设置主轴的方向。

- 默认主轴方向就是 x 轴方向，水平向右
- 默认侧轴方向就是 y 轴方向，水平向下

![flex布局主轴](https://gitee.com/Topcvan//img-storage/raw/master//css/flex%E5%B8%83%E5%B1%80%E4%B8%BB%E8%BD%B4.png)

**属性值**

flex-direction 属性决定主轴的方向（即项目的排列方向）
注意： 主轴和侧轴是会变化的，就看 flex-direction 设置谁为主轴，剩下的就是侧轴。而子元素是跟着主轴来排列的

| 属性值         | 说明             |
| -------------- | ---------------- |
| row            | 默认值，从左到右 |
| row-reverse    | 从右到左         |
| column         | 从上到下         |
| column-reverse | 从下到上         |

#### 2.3 justify-content

justify-content属性定义了项目在主轴上的对齐方式。使用这个属性之前一定要确定好主轴是哪个

| 属性值        | 说明                                          |
| ------------- | --------------------------------------------- |
| flex-start    | 默认值。从头部开始，如果主轴是x轴，则从左到右 |
| flex-end      | 从尾部开始排列(子盒子顺序不变，只是靠右对齐)  |
| center        | 在主轴居中对齐（如果主轴是x轴则 水平居中）    |
| space-around  | 平分剩余空间                                  |
| space-between | 先两边贴边 再平分剩余空间                     |

#### 2.4  flex-wrap

默认情况下，项目都排在一条线（又称”轴线”）上。flex-wrap属性定义，flex布局中默认是不换行的，如果一行放不下会缩小子盒子宽度。

| 属性值   | 说明           |
| -------- | -------------- |
| `nowrap` | 默认值，不换行 |
| `wrap`   | 换行           |

#### 2.5 align-items

该属性是控制子项在侧轴（默认是y轴）上的排列方式，在子项为单项（单行）的时候使用

| 属性值     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| flex-start | 从上到下，贴上沿显示                                         |
| flex-end   | 从下到上，贴下沿显示                                         |
| center     | 挤在一起居中（垂直居中）                                     |
| stretch    | 拉伸填满侧轴  （默认值，在子盒子不设置侧轴对应的宽度/高度时生效 ） |

#### 2.6 align-content

设置子项在侧轴上的排列方式 并且只能用于子项出现 换行 的情况（多行），在单行下是没有效果的

| 属性值        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| flex-start    | 默认值在侧轴的头部开始排列                                   |
| flex-end      | 在侧轴的尾部开始排列                                         |
| center        | 在侧轴中间显示                                               |
| space-around  | 子项在侧轴平分剩余空间                                       |
| space-between | 子项在侧轴先分布在两头，再平分剩余空间                       |
| stretch       | 设置子项元素高度平分父元素高度(默认值，子盒子不设置宽/高生效) |

#### 2.7 align-content 和 align-items 区别

- align-items  适用于单行情况下， 只有上对齐、下对齐、居中和 拉伸
- align-content 适应于换行（多行）的情况下（单行情况下无效）， 可以设置 上对齐、 下对齐、居中、拉伸以及平均分配剩余空间等属性值。 
- 总结就是单行找 align-items  多行找 align-content

#### 2.8 flex-flow

 flex-flow 属性是 flex-direction 和 flex-wrap 属性的复合属性

```css
flex-flow:row wrap;
```

- flex-direction：设置主轴的方向

- justify-content：设置主轴上的子元素排列方式

- flex-wrap：设置子元素是否换行  

- align-content：设置侧轴上的子元素的排列方式（多行）

- align-items：设置侧轴上的子元素排列方式（单行）

- flex-flow：复合属性，相当于同时设置了 flex-direction 和 flex-wrap

### 三、flex布局子项常见属性

#### 4.1  flex 属性 

flex 属性定义子项目分配剩余空间，用flex来表示占多少份数。

```css
.item {
    flex: <number>; /* default 0 */
}
```

#### 4.2 align-self

align-self 属性允许单个项目有与其他项目不一样的对齐方式，可覆盖 align-items 属性。
默认值为 auto，表示继承父元素的 align-items 属性，如果没有父元素，则等同于 stretch。

```css
span:nth-child(2) {
      /* 设置自己在侧轴上的排列方式 */
      align-self: flex-end;
 }
```

align-self取值与align-items属性一致

#### 4.3 order

order属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。
注意：和 z-index 不一样。

```css
.item {
    order: <number>;
}
```

如：

```html
<style>
    section {
        display: flex;
    }
    section span:nth-child(2) {
        order: -1
    }
</style>

<body>
    <section>
    	<span>1</span>
        <span>2</span>
    </section>
</body>
```

效果如下：

![flex-order](https://gitee.com/Topcvan//img-storage/raw/master//css/flex-order.png)

### 四、linear-gradient()

CSS **`linear-gradient()`** 函数用于创建一个表示两种或多种颜色线性渐变的图片。

如：

```css
/* 渐变轴为45度，从蓝色渐变到红色 */
linear-gradient(45deg, blue, red);

/* 从右下到左上、从蓝色渐变到红色 */
linear-gradient(to left top, blue, red);

/* 从下到上，从蓝色开始渐变、到高度40%位置是绿色渐变开始、最后以红色结束 */
linear-gradient(0deg, blue, green 40%, red);
```

值：

- `<side-or-corner>`

  描述渐变线的起始点位置。它包含to和两个关键词：第一个指出水平位置left or right，第二个指出垂直位置top or bottom。关键词的先后顺序无影响，且都是可选的。
  to top, to bottom, to left 和 to right这些值会被转换成角度0度、180度、270度和90度。其余值会被转换为一个以向顶部中央方向为起点顺时针旋转的角度。渐变线的结束点与其起点中心对称。

- `<angle>`

  用角度值指定渐变的方向（或角度）。角度顺时针增加。 

- `<linear-color-stop>`

  由一个`color`值组成，并且跟随着一个可选的终点位置（可以是一个百分比值或者是沿着渐变轴的`length`）

