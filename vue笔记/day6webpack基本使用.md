[TOC]



### 一、创建局部webpack

1. 在项目目录下创建package.json文件，用于管理项目信息、库依赖等，使用以下命令创建

   ```
   npm init
   ```

2. 安装局部webpack

   ```
   npm install webpack webpack-cli -D
   ```

3. 使用局部webpack打包代码

   此时直接在终端执行`webpack`命令会使用全局webpack进行打包，而使用局部webpack打包有两种方式

   - 第一种：执行`npx webpack`命令，会优先寻找modules中的局部webpack进行打包
   - 第二种：在package.json中的script属性下添加`"build": "webpack"`属性，然后执行`npm run build`执行打包

### 二、webpack配置文件

- 在通常情况下，webpack需要打包的项目都是非常复杂的，并且需要一系列配置来满足要求，所以需要一个配置文件

- 我们可以在根目录下创建一个webpack.config.js文件来作为webpack配置文件

  1. 首先使用common.js语法来导出配置信息

     ```
     module.exports = {
     	entry: "./src/main.js" //指定打包入口
     }
     ```

  2. 然后是配置出口文件夹

     ```
     module.exports = {
     	entry: "./src/main.js", //指定打包入口
     	output: {
     		filename: "bundle.js" // 指定打包文件名字
     		path: "..." //此处需要使用绝对路径
     	}
     }
     ```

  3. 由于出口文件路径需要使用绝对路径，此处可以用nodejs的语法来简化

     ```
     const path = require('path');
     
     module.exports = {
     	entry: "./src/main.js", //指定打包入口
     	output: {
     		filename: "bundle.js" // 指定打包文件名字
     		path: path.resolve(__dirname, "./dist") //__dirname是当前配置文件所处的路径
     	}
     }
     ```

  4. 指定配置文件

     配置文件不一定要叫webpack.config.js，可以在webpack.json指定配置文件

     ```
     "script": {
     	"build": "webpack --config xxx.config.js" //在build处指定xxx.config.js作为配置文件
     }
     ```

### 三、webpack依赖图

- 事实上webpack在处理应用程序时，它会根据命令或者配置文件找到入口文件
- 从入口开始，会生成一个依赖关系图，这个依赖关系图会包含应用程序中所需的所有模块（比如.js文件、css文件、图片、字体等）
- 然后遍历图结构，打包一个个模块（根据文件的不同使用不同的loader来解析）
- 所以，如果某个文件需要被打包进该项目，就必须加入到依赖图中，被依赖图中某个文件关联

### 四、loader

#### 4.1 loader是什么

- loader可以用于对模块的源代码进行转换

- 我们可以将css文件也看成是一个模块，我们是通过import来加载这个模块的

- 在加载这个模块时，webpack其实并不知道如何对其进行加载，我们必须制定对应的loader来完成这个功能

- 对于css来说，最常用的loader是css-loader

  ```
  //css loader安装
  npm install css-loader -D
  ```

#### 4.2 css-loader使用方案

1. 内联方式

   在引入的样式前加上使用的loader，并且用!分割

   ```
   import "css-loader!../css/style.css";
   ```

2. 配置方式

   配置方式表示的意思是在我们的webpack.config.js文件中写明配置信息。module.rules中允许我们配置多个loader（因为我们也会继续使用其他的loader，来完成其他文件的加载）。这种方式可以更好的表示loader的配置，也方便后期的维护，同时也让你对各个Loader有一个全局的概览

   module.rules配置如下：

   - rules属性对应的值是一个数组：[Rule]

   - 数组中存放的是一个个的Rule，Rule是一个对象，对象中可以设置多个属性

     - test属性：用于对 resource（资源）进行匹配的，通常会设置成正则表达式
     - use属性：对应的值是一个数组：[UseEntry]
       - UseEntry是一个对象，可以通过对象的属性来设置一些其他属性
         - loader：必须有一个 loader属性，对应的值是一个字符串
         - options：可选的属性，值是一个字符串或者对象，值会被传入到loader中
         - query：目前已经使用options来替代
       - 传递字符串（如：use: [ 'style-loader' ]）是 loader 属性的简写方式（如：use: [ { loader: 'style-loader'} ]）
     - loader属性： Rule.use: [ { loader } ] 的简写

     ```
     module.exports = {
     	entry: "./src/main.js", 
     	output: {
     		filename: "bundle.js" 
     		path: "path.resolve(__dirname, "./dist")" 
     	},
     	modules: {
     		rules: [
     			{
     				test: /\.css$/,
     				//写法一: "css-loader"
     				//写法二: use: ["css-loader"]
     				//写法三
     				use: [
     					{loader: "css-loader"}
     				]
     			}
     		]
     	}
     }
     ```

#### 4.3 style-loader使用

在使用css-loader后css在代码中并没有生效，因为css-loader只是负责将.css文件进行解析，并不会将解析之后的css插入到页面中。如果我们希望再完成插入style的操作，那么我们还需要另外一个loader，就是style-loader

1. 安装

   ```
   npm install style-loader -D
   ```

2. 配置

   在配置文件中，添加style-loader

   ```
   use: [
   	{loader: "style-loader"}, //因为loader的执行顺序是从下往上的，所以style-                                   //loader要写在css-loader上面
   	{loader: "css-loader"}
   ]
   ```

#### 4.4 less文件处理

在项目中less文件需要对应的loader来解析文件，这个loader就是less-loader

```
npm install less-loader -D
```

实际上less-loader需要一个叫lessc的工具，来将less文件转化为css文件，但在安装less-loader时会自动安装lessc

配置less-loader

```
//与css-loader配置的位置相同
{
	test: /\.less$/,
	use: [
		loader: "style-loader",
		loader: "css-loader",
		loader: "less-loader"
	]
}
```

#### 4.5 PostCSS

- 什么是PostCSS

  - PostCSS是一个通过JavaScript来转换样式的工具
  - 这个工具可以帮助我们进行一些CSS的转换和适配，比如自动添加浏览器前缀、css样式的重置
  - 但是实现这些功能，我们需要借助于PostCSS对应的插件

- 如何使用PostCSS呢？主要就是两个步骤：

  1. 查找PostCSS在构建工具中的扩展，比如webpack中的postcss-loader
  2. 选择可以添加你需要的PostCSS相关的插件

- 在webpack中使用postcss-loader

  - 安装

    ```
    npm install postcss-loader -D
    //安装插件
    npm install postcss-preset-env -D
    ```

  - 配置

    ```
    test: /\.css$/,
    use: [
    	loader: "style-loader",
    	loader: "css-loader",
    	{
    		loader: "postcss-loader",
    		options: {
    			postcssOptions: {
    				plugins: [
    					require('postcss-preset-env')
    				]
    			}
    		}
    	}
    ]
    ```

    - 还可以将这些配置信息放到一个单独的文件中进行管理

      在根目录下创建postcss.config.js

      ```
      module.exports = {
      	plugin: [
      		require('postcss-preset-env')
      	]
      }
      ```

      在webpack.config.json中

      ```
      test: /\.css$/,
      use: [
      	loader: "style-loader",
      	loader: "css-loader",
      	loader: "postcss-loader"
      ]
      ```

      