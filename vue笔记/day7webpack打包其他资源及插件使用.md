[TOC]

### 一、图片引用及file-loader

- 为了演示我们项目中可以加载图片，我们需要在项目中使用图片，比较常见的使用图片的方式是两种

  - img元素，设置src属性
  - 其他元素（比如div），设置background-image的css属性

- file-loader

  file-loader的作用就是帮助我们处理import/require()方式引入的一个文件资源，并且会将它放到我们输出的文件夹中

  - 安装file-loader

    ```
    npm install file-loader -D
    ```

  - 配置loader

    ```
    {
    	test: /\.(png|jpe?g|gif|svg)$/i,
    	use: {
    		loader: "file-loader"
    	}
    }
    ```

- 在配置了file-loader后，通过在单独css文件中的background-image属性引入的图片可以正常打包。但通过js创建img标签并设置路径的图片是不能打包的，因为路径已经被写死，而在打包完成后浏览器会根据相对于index.html的路径来寻找图片，但设置路径时的相对路径是相对于js文件的，所以会出错。

  ```
  //此时，可以通过将图片视为模块进行导入
  import img1 from "xxx";
  
  const imgel = document.createElement("img");
  imgel.src = imgl;
  //这样就能正常打包了
  ```

- 图片命名规则

  - 有时候我们处理后的文件名称按照一定的规则进行显示，比如保留原来的文件名、扩展名，同时为了防止重复，包含一个hash值等
  - 这个时候我们可以使用PlaceHolders来完成，webpack给我们提供了大量的PlaceHolders来显示不同的内容：
    - [ext]： 处理文件的扩展名
    - [name]：处理文件的名称
    - [hash]：文件的内容，使用MD4的散列函数处理，生成的一个128位的hash值（32个十六进制）
    - [contentHash]：在file-loader中和[hash]结果是一致的（在webpack的一些其他地方不一样，后面会讲到）
    - [hash:<length>]：截图hash的长度，默认32个字符太长了
    - [path]：文件相对于webpack配置文件的路径
    - ......

- 图片名称及路径设置

  ```
  {
  	test: /\.(jpe?g|png|gif|svg)$/i,
  	use: {
  		loader: "file-loader",
  		options: {
  			name: "img/[name]_[hash:8].[ext]" //前面的img是图片打包出口路径
  			//相当于 outputPath: 'img',
  			// name: "[name]_[hash:8].[ext]"
  		}
  	}
  }
  ```

### 二、url-loader

url-loader和file-loader的工作方式是相似的，但是可以将较小的文件，转成base64的URI

- 安装

  ```
  npm install url-loader -D
  ```

- 配置

  ```
  {
  	test: /\.(jpe?g|png|gif|svg)$/i,
  	use: {
  		loader: "url-loader",
  		options: {
  			limit: 100*1024 //设置小于100kb的图片会被转换
  		}
  	}
  }
  ```

### 三、asset module type的使用

在webpack5之前，加载这些资源我们需要使用一些loader，比如raw-loader 、url-loader、file-loader。在webpack5开始，我们可以直接使用资源模块类型（asset module type），来替代上面的这些loader，asset module type在webpack中内置，不需要下载。

- 资源模块类型(asset module type)，通过添加4 种新的模块类型，来替换所有这些loader
  - asset/resource 发送一个单独的文件并导出URL。之前通过使用file-loader 实现
  - asset/inline 导出一个资源的data URI。之前通过使用url-loader 实现
  - asset/source 导出资源的源代码。之前通过使用raw-loader 实现
  - asset 在导出一个data URI 和发送一个单独的文件之间自动选择。之前通过使用url-loader，并且配置资源体积限制实现

- 使用

  比如加载图片，可以用以下方式

  ```
  {
  	test: /\.(png|jpe?g|gif|svg)$/i,
  	type: "asset/resource"
  }
  ```

  - 定义文件输出路径和文件名

    1. 方法一：在rules中，添加generator属性，并设置filename

       ```
       {
       	test: /\.(png|jpe?g|gif|svg)$/i,
       	type: "asset/resource",
       	generator: {
       		filename: "img/[name]_[hash:8][ext]" //名字和拓展名间不需要加点
       	}
       }
       ```

    2. 方法二：在output中添加assetModuleFilename属性

       ```
       output: {
       	filename: "bundle.js",
       	path: path.resolve(__dirname, "./dist"),
       	assetModuleFilename: "img/[name]_[hash:8][ext]"
       }
       ```

  - asset类型使用

    asset是自动选择实现file-loader还是url-loader

    ```
    {
    	test: /\.(png|jpe?g|gif|svg)$/i,
    	type: "asset",
    	generator: {
    		filename: "img/[name]_[hash:8][ext]"
    	},
    	parser: {
    		dataUrlCondition: {
    			maxSize: 100 * 1024  //相当于url-loader中的limit
    		}
    	}
    }
    ```

### 四、加载字体文件

首先import字体库中的css文件，然后引用其中的类，最后使用file-loader或者asset/resource来打包

### 五、插件

#### 5.1认识plugin

Loader是用于特定的模块类型进行转换，而Plugin可以用于执行更加广泛的任务，比如打包优化、资源管理、环境变量注入等

#### 5.2 CleanWebpackPlugin

每次修改了一些配置，重新打包时，都需要手动删除dist文件夹,我们可以借助于一个插件来帮助我们完成，这个插件就是CleanWebpackPlugin

- 安装

	```
	npm install clean-webpack-plugin -D
	```

- 配置

	```
	//在webpack.config.js中
	const {CleanWebpackPlugin} = require('clean-webpack-plugin');
	
	module.exports = {
		...
		plugins: [
			new CleanWebpackPlugin()
		]
	}
	```

#### 5.3 HtmlWebpackPlugin

我们的HTML文件是编写在根目录下的，而最终打包的dist文件夹中是没有index.html文件的.在进行项目部署的时，必然也是需要有对应的入口文件index.html,所以我们也需要对index.html进行打包处理,对HTML进行打包处理我们可以使用另外一个插件：HtmlWebpackPlugin

- 安装

	```
	npm install html-webpack-plugin -D
	```

- 配置

	```
	const HtmlWebpackPlugin = require('html-webpack-plugin');
	...
	module.exports = {
		...
		plugins: [
			...//其他插件
			new HtmlWebpackPlugin()
		]
	}
	```
	
	现在自动在dist文件夹中，生成了一个index.html的文件，该文件中也自动添加了我们打包的bundle.js文件。这个文件是如何生成的呢？默认情况下是根据ejs的一个模板来生成的，在html-webpack-plugin的源码中，有一个default_index.ejs模块。
	
- 自定义HTML模板

  如果我们想在自己的模块中加入一些比较特别的内容，比如添加一个noscript标签，在用户的JavaScript被关闭时，给予响应的提示，这个我们需要一个属于自己index.html模块。<br>在配置HtmlWebpackPlugin时，我们可以添加如下配置

  - template：指定我们要使用的模块所在的路径
  - title：在进行htmlWebpackPlugin.options.title读取时，就会读到该信息

  ```
  plugins: [
  	new HtmlWebpackPlugin({
  		title: "webpack项目",
  		template: "./public/index.html"
  	})
  ]
  ```

#### 5.4 DefinePlugin

在完成以上配置时，在打包复制vue-cli下的html模板还是报错，这是因为在编译template模块时，有一个BASE_URL，`<link rel="icon" href="<%= BASE_URL %>favicon.ico">`， 但是我们并没有设置过这个常量值，所以会出现没有定义的错误。这个时候我们可以使用DefinePlugin插件

DefinePlugin允许在编译时创建配置的全局常量，是一个webpack内置的插件（不需要单独安装）

配置

```
const {DefinePlugin} = require('webpack');

plugins: [
	new DefinePlugin({
		BASE_URL: '"./"'
		//也可以这样写
		//BASE: 'value' value是当前文件定义的一个变量
	})
]
```

#### 5.5 CopyWebpackPlugin

在vue的打包过程中，如果我们将一些文件放到public的目录下，那么这个目录会被复制到dist文件夹中，这个复制的功能，我们可以使用CopyWebpackPlugin来完成

安装

```
npm install copy-webpack-plugin -D
```

配置

复制的规则在patterns中设置

- from：设置从哪一个源中开始复制
- to：复制到的位置，可以省略，会默认复制到打包的目录下
- globOptions：设置一些额外的选项，其中可以编写需要忽略的文件
  - .DS_Store：mac目录下回自动生成的一个文件
  - index.html：也不需要复制，因为我们已经通过HtmlWebpackPlugin完成了index.html的生成

```
const CopyWebpackPlugin = require('copy-webpack-plugin');

plugins: [
	new CopyWebpackPlugin({
		patterns: [  //复制的规则在patterns中设置
			{
				from: "public", //设置从哪一个源中开始复制
				to: "./abc", //会复制到打包文件夹下的abc文件夹中
				globOptions: {
					ignore: [
						'**/index.html', //该文件目录下所有index.html都不进行复制
						'**/.DS_Store'
					]
				}
			},
			{
			//第二个需要复制的文件夹设置
			}
		]
	})
]
```

### 六、Mode配置

Mode配置选项，可以告知webpack使用响应模式的内置优化

- 默认值是production（什么都不设置的情况下）
- 可选值有：'none' | 'development' | 'production'
- 其中none是不使用任何默认优化选项

在进行调试时报错位置是打包后的bundle.js文件中的位置，难以调试，所以需要设置mode

设置mode

```
//在webpack.config.js文件下
module.exports = {
	mode: 'development'
}
```

设置devtool

- 在设置mode为development后devtool默认为eval，需要更改方便在开发时调试

	```
	module.exports = {
		mode: 'development'，
		devtool: 'source-map'
	}
	```

- 设置为source-map后在控制台报错会映射到打包前的文件中的位置
