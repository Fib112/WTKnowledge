### 一、精灵图

一个网页中往往会应用很多小的背景图像作为修饰，当网页中的图像过多时，服务器就会频繁地接收和发送请求图片，造成服务器请求压力过大，这将大大降低页面的加载速度。 
因此，为了有效地减少服务器接收和发送请求的次数，提高页面的加载速度，出现了 CSS 精灵技术（也称 CSS Sprites、CSS 雪碧）。 
核心原理：将网页中的一些小背景图像整合到一张大图中 ，这样服务器只需要一次请求就可以了。

使用精灵图核心： 

- 精灵技术主要针对于背景图片使用。就是把多个小背景图片整合到一张大图片中。 
- 这个大图片也称为 sprites  精灵图  或者 雪碧图 
- 移动背景图片位置， 此时可以使用 background-position 。 
- 移动的距离就是这个目标图片的 x 和 y 坐标。注意网页中的坐标有所不同(向右和向下是正值)
- 因为一般情况下都是往上往左移动，所以数值是负值。 
- 使用精灵图的时候需要精确测量，每个小背景图片的大小和位置。

### 二、字体图标

#### 2.1 字体图标应用场景

精灵图是有诸多优点的，但是缺点很明显。 

1. 图片文件还是比较大的。 
2. 图片本身放大和缩小会失真。 
3. 一旦图片制作完毕想要更换非常复杂。 

此时，有一种技术的出现很好的解决了以上问题，就是字体图标 iconfont。 
字体图标可以为前端工程师提供一种方便高效的图标使用方式，展示的是图标，本质属于字体。 
字体图标使用场景：  主要用于显示网页中通用、常用的一些小图标。 

#### 2.2 字体图标优点

- 轻量级：一个图标字体要比一系列的图像要小。一旦字体加载了，图标就会马上渲染出来，减少了服务器请求 
- 灵活性：本质其实是文字，可以很随意的改变颜色、产生阴影、透明效果、旋转等 
- 兼容性：几乎支持所有的浏览器，请放心使用 

注意： 字体图标不能替代精灵技术，只是对工作中图标部分技术的提升和优化。 
总结： 

- 如果遇到一些结构和样式比较简单的小图标，就用字体图标。 
- 如果遇到一些结构和样式复杂一点的小图片，就用精灵图。 

#### 2.3 字体图标使用

字体图标是一些网页常见的小图标，我们直接网上下载即可。 因此使用可以分为： 

1. 字体图标的下载  

   icomoon 字库  http://icomoon.io    推荐指数  ★★★★★ 
   阿里 iconfont 字库   http://www.iconfont.cn/   推荐指数   ★★★★★  

2. 字体图标的引入 （引入到我们html页面中） 

   - 把下载包里面的 fonts 文件夹放入页面根目录下

     ![字体图标引入1](https://gitee.com/Topcvan//img-storage/raw/master//css/%E5%AD%97%E4%BD%93%E5%9B%BE%E6%A0%87%E5%BC%95%E5%85%A51.png)

   - 字体文件格式 

     不同浏览器所支持的字体格式是不一样的，字体图标之所以兼容，就是因为包含了主流浏览器支持的字体文件。 

     1. TureType(.ttf)格式.ttf字体是Windows和Mac的最常见的字体，支持这种字体的浏览器有IE9+、Firefox3.5+、Chrome4+、Safari3+、Opera10+、iOS Mobile、Safari4.2+； 
     2. Web Open Font Format(.woff)格式woff字体，支持这种字体的浏览器有IE9+、Firefox3.5+、Chrome6+、Safari3.6+、Opera11.1+； 
     3. Embedded Open Type(.eot)格式.eot字体是IE专用字体，支持这种字体的浏览器有IE4+； 
     4. SVG(.svg)格式.svg字体是基于SVG字体渲染的一种格式，支持这种字体的浏览器有Chrome4+、Safari3.1+、Opera10.0+、iOS Mobile Safari3.2+； 

   - 在 CSS 样式中全局声明字体： 简单理解把这些字体文件通过css引入到页面中。一定注意字体文件路径的问题。(可以在下载的文件夹中的style.css文件中复制)  

     ```css
     @font-face { 
        font-family: 'icomoon'; 
        src:  url('fonts/icomoon.eot?7kkyc2'); 
        src:  url('fonts/icomoon.eot?7kkyc2#iefix') format('embedded-opentype'), 
          url('fonts/icomoon.ttf?7kkyc2') format('truetype'), 
          url('fonts/icomoon.woff?7kkyc2') format('woff'), 
          url('fonts/icomoon.svg?7kkyc2#icomoon') format('svg'); 
        font-weight: normal; 
        font-style: normal; 
      }
     ```

   - html 标签内添加小图标

     打开demo.html，并复制字体对应代码

     ![字体图标引入2](https://gitee.com/Topcvan//img-storage/raw/master//css/%E5%AD%97%E4%BD%93%E5%9B%BE%E6%A0%87%E5%BC%95%E5%85%A52.png)

     然后添加到要使用的标签内

   - 给标签定义字体。 

     ```css
     span { 
        font-family: "icomoon"; 
      }
     ```

     务必保证这个字体和上面@font-face里面的字体保持一致。除此之外，可以通过font属性以及color等对字体图标进行自定义样式设置

3. 字体图标的追加 （以后添加新的小图标）

   如果工作中，原来的字体图标不够用了，我们需要添加新的字体图标到原来的字体文件中。 把压缩包里面的 selection.json 重新上传字体图标的网站中，然后选中自己想要新的图标，从新下载压缩包，并替换原来的文件即可

### 三、css三角

网页中常见一些三角形，使用 CSS 直接画出来就可以，不必做成图片或者字体图标。

生成三角形对应css代码：

```css
div { 
     width: 0; 
     height: 0; 
     line-height: 0; 
     font-size: 0; 
     border: 50px solid transparent; 
     border-left-color: pink; 
 } 
```

调整各边框的粗细可得到不同边长的三角形

### 四、用户界面样式

所谓的界面样式，就是更改一些用户操作样式，以便提高更好的用户体验。 

- 更改用户的鼠标样式  
- 表单轮廓 
- 防止表单域拖拽

#### 4.1 鼠标样式 cursor

```css
li {cursor: pointer; }
```

设置或检索在对象上移动的鼠标指针采用何种系统预定义的光标形状。

| 属性值      | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| default     | ![指针形状之default](https://gitee.com/Topcvan//img-storage/raw/master//css/%E6%8C%87%E9%92%88%E5%BD%A2%E7%8A%B6%E4%B9%8Bdefault.png) |
| pointer     | ![指针形状之pointer](https://gitee.com/Topcvan//img-storage/raw/master//css/%E6%8C%87%E9%92%88%E5%BD%A2%E7%8A%B6%E4%B9%8Bpointer.png) |
| move        | ![指针形状之move](https://gitee.com/Topcvan//img-storage/raw/master//css/%E6%8C%87%E9%92%88%E5%BD%A2%E7%8A%B6%E4%B9%8Bmove.png) |
| text        | ![指针形状之text](https://gitee.com/Topcvan//img-storage/raw/master//css/%E6%8C%87%E9%92%88%E5%BD%A2%E7%8A%B6%E4%B9%8Btext.png) |
| not-allowed | ![指针形状之notallowed](https://gitee.com/Topcvan//img-storage/raw/master//css/%E6%8C%87%E9%92%88%E5%BD%A2%E7%8A%B6%E4%B9%8Bnotallowed.png) |

#### 4.2 轮廓线 outline 

给表单添加 outline: 0;   或者  outline: none; 样式之后，就可以去掉默认的蓝色边框。

```css
input {
    outline: none; 
}
```

#### 4.3 防止拖拽文本域 resize

实际开发中，我们文本域右下角是不可以拖拽的。

```css
textarea{
    resize: none;
}
```

#### 4.4 取消列表圆点

```css
li {
    list-style: none;
}
```

### 五、vertical-align

CSS 的 vertical-align 属性使用场景： 经常用于设置图片或者表单(行内块元素）和文字垂直对齐。 
官方解释： 用于设置一个元素的垂直对齐方式，但是它只对行内元素、行内块元素和表格单元格元素生效，不能用它垂直对齐块级元素

语法：

```css
selector {
    vertical-align : baseline | top | middle | bottom
}
```

| 值       | 描述                                 |
| -------- | ------------------------------------ |
| baseline | 默认，元素放置在父元素基线上         |
| top      | 把元素的顶端与行中最高元素的顶端对齐 |
| middle   | 把此元素放置在父元素的中部           |
| bottom   | 把元素的顶端与行中最低的元素顶端对齐 |

![baseline](https://gitee.com/Topcvan//img-storage/raw/master//css/baseline.png)

#### 5.1 图片、表单和文字对齐 

图片、表单都属于行内块元素，默认的 vertical-align 是基线对齐。

![图片与文字基线对齐](https://gitee.com/Topcvan//img-storage/raw/master//css/%E5%9B%BE%E7%89%87%E4%B8%8E%E6%96%87%E5%AD%97%E5%9F%BA%E7%BA%BF%E5%AF%B9%E9%BD%90.png)

此时可以给图片、表单这些行内块元素的 vertical-align 属性设置为 middle 就可以让文字和图片垂直居中对齐了

#### 5.2 解决图片底部默认空白缝隙

bug：图片底侧会有一个空白缝隙，原因是行内块元素会和文字的基线对齐。 
主要解决方法有两种： 

1. 给图片添加 vertical-align:middle | top| bottom 等。 （提倡使用的） 
2. 把图片转换为块级元素  display: block;  

### 六、溢出的文字省略号显示

- 单行文本溢出显示省略号--必须满足三个条件：

  ```css
  /*1. 先强制一行内显示文本*/ 
  white-space: nowrap;  (默认 normal 自动换行)
  /*2. 超出的部分隐藏*/ 
  overflow: hidden; 
  /*3. 文字用省略号替代超出的部分*/ 
  text-overflow: ellipsis;
  ```

- 多行文本溢出显示省略号

  ```css
  overflow: hidden; 
  text-overflow: ellipsis; 
  /* 弹性伸缩盒子模型显示 */ 
  display: -webkit-box; 
  /* 限制在一个块元素显示的文本的行数 */ 
  -webkit-line-clamp: 2; 
  /* 设置或检索伸缩盒对象的子元素的排列方式 */ 
  -webkit-box-orient: vertical; 
  ```

