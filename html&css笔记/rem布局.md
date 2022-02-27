### 一、rem基础

rem 单位

rem (root em)是一个相对单位，类似于em，em是父元素字体大小。
不同的是rem的基准是相对于html元素的字体大小。
比如，根元素（html）设置font-size=12px; 非根元素设置width:2rem; 则换成px表示就是24px。

```css
/* 根html 为 12px */
html {
   font-size: 12px;
}
/* 此时 div 的字体大小就是 24px */       
div {
    font-size: 2rem;
}
```

rem的优势：父元素文字大小可能不一致， 但是整个页面只有一个html，可以很好来控制整个页面的元素大小

### 二、媒体查询

#### 2.1 什么是媒体查询

媒体查询（Media Query）是CSS3新语法。

- 使用 @media 查询，可以针对不同的媒体类型定义不同的样式
- @media 可以针对不同的屏幕尺寸设置不同的样式
- 当重置浏览器大小的过程中，页面也会根据浏览器的宽度和高度重新渲染页面 
- 目前针对很多苹果手机、Android手机，平板等设备都用得到多媒体查询

2.2 语法规范

```css
@media mediatype and|not|only (media feature) {
    CSS-Code;
}
```

- 用 @media 开头 注意@符号
- mediatype  媒体类型
- 关键字 and  not   only
- media feature 媒体特性 必须有小括号包含

##### 2.2.1 mediatype 查询类型

将不同的终端设备划分成不同的类型，称为媒体类型

| 值     | 解释说明                           |
| ------ | ---------------------------------- |
| all    | 用于所有设备                       |
| print  | 用于打印机和打印预览               |
| screen | 用于电脑屏幕，平板电脑，智能手机等 |

##### 2.2.2 关键字

关键字将媒体类型或多个媒体特性连接到一起做为媒体查询的条件。

- and：可以将多个媒体特性连接到一起，相当于“且”的意思。

- not：排除某个媒体类型，相当于“非”的意思，可以省略。
- only：指定某个特定的媒体类型，可以省略

##### 2.2.3 媒体特性

每种媒体类型都具体各自不同的特性，根据不同媒体类型的媒体特性设置不同的展示风格。

| 值        | 解释说明                           |
| --------- | ---------------------------------- |
| width     | 定义输出设备中页面可见区域的宽度   |
| min-width | 定义输出设备中页面最小可见区域宽度 |
| max-width | 定义输出设备中页面最大可见区域宽度 |

#### 2.2 用法示例

在宽度大于320px的屏幕上背景色为蓝色，在宽度大于540px的屏幕上背景色为绿色：

```css
@media screen and (min-width: 320px) {
    body {
        background-color: blue;
    }
}

@media screen and (min-width: 540px) {
    body {
        background-color: green;
    }
}
```

#### 2.3 媒体查询+rem

rem单位是跟着html来走的，有了rem页面元素可以设置不同大小尺寸。媒体查询可以根据不同设备宽度来修改样式。媒体查询+rem  就可以实现不同设备宽度，实现页面元素大小的动态变化

#### 2.4 引入资源

当样式比较繁多的时候，我们可以针对不同的媒体使用不同 stylesheets（样式表）。原理，就是直接在link中判断设备的尺寸，然后引用不同的css文件。

语法规范：

```css
<link rel="stylesheet" media="mediatype and|not|only (media feature)" href="mystylesheet.css">
```

示例：

```css
<link rel="stylesheet" href="styleA.css" media="screen and (min-width: 400px)">
```

