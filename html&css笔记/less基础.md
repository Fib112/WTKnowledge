### 一、Less介绍

#### 1.1 维护 css 的弊端

CSS 是一门非程序式语言，没有变量、函数、SCOPE（作用域）等概念。

- CSS 需要书写大量看似没有逻辑的代码，CSS 冗余度是比较高的。

- 不方便维护及扩展，不利于复用。

- CSS 没有很好的计算能力

- 非前端开发工程师来讲，往往会因为缺少 CSS 编写经验而很难写出组织良好且易于维护的 CSS 代码项目

#### 1.2 Less 介绍

Less （Leaner Style Sheets 的缩写） 是一门 CSS 扩展语言，也成为CSS预处理器。
做为 CSS 的一种形式的扩展，它并没有减少 CSS 的功能，而是在现有的 CSS 语法上，为CSS加入程序式语言的特性。
它在 CSS 的语法基础之上，引入了变量，Mixin（混入），运算以及函数等功能，大大简化了 CSS 的编写，并且降低了 CSS 的维护成本，就像它的名称所说的那样，Less 可以用更少的代码做更多的事情。
Less中文网址： http://lesscss.cn/
常见的CSS预处理器：Sass、Less、Stylus
一句话：Less 是一门 CSS 预处理语言，它扩展了CSS的动态特性。

#### 1.3 Less 使用

首先新建一个后缀名为less的文件， 在这个less文件里面书写less语句。

### 二、Less特性

Less主要有以下特性：

- Less 变量
- Less 编译
- Less 嵌套
- Less 运算

#### 2.1 Less变量

变量是指没有固定的值，可以改变的。因为我们CSS中的一些颜色和数值等经常使用。

```css
@变量名:值;
```

变量命名规范

- 必须有@为前缀
- 不能包含特殊字符
- 不能以数字开头
- 大小写敏感

如：

```less
@color: pink;
```

变量使用规范

```less
// 定义变量
@color: pink;
// 直接使用
body{
    color:@color;
}
a:hover{
    color:@color;
}
```

#### 2.2 Less 编译 

本质上，Less 包含一套自定义的语法及一个解析器，用户根据这些语法定义自己的样式规则，这些规则最终会通过解析器，编译生成对应的 CSS 文件。

所以，需要把less文件，编译生成为css文件，这样html页面才能使用。

vocode Less 插件：Easy LESS

#### 2.3 Less 嵌套

开发中经常用到选择器的嵌套

```css
#header .logo {
  width: 300px;
}
```

Less 嵌套写法

```less
#header {
    .logo {
       width: 300px;
    }
}
```

如果遇见 （交集|伪类|伪元素选择器） 

```css
a:hover{
    color:red;
}
```

Less 嵌套写法

```less
a{
  &:hover{
      color:red;
  }
}
```

- 内层选择器的前面没有 & 符号，则它被解析为父选择器的后代；
- 如果有 & 符号，它就被解析为父元素自身或父元素的伪类。

#### 2.4 Less 运算

任何数字、颜色或者变量都可以参与运算。Less提供了加（+）、减（-）、乘（\*）、除（/）算术运算。

```less
/*Less 里面写*/
@witdh: 10px + 5;
div {
    border: @witdh solid red;
}
/*生成的css*/
div {
  border: 15px solid red;
}
/*Less 甚至还可以这样 */
width: (@width + 5) * 2;
```

注意：

乘号（*）和除号（/）的写法   

- 运算符中间左右有个空格隔开 1px + 5
- 对于两个不同的单位的值之间的运算，运算结果的值取第一个值的单位 
- 如果两个值之间只有一个值有单位，则运算结果就取该单位