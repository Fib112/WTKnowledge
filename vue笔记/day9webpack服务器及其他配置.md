[TOC]

### 一、搭建本地服务器

#### 1.1 为什么要搭建本地服务器

- 目前我们开发的代码，为了运行需要有两个操作
  1. 操作一：npm run build，编译相关的代码
  2. 操作二：通过live server或者直接通过浏览器，打开index.html代码，查看效果
- 这个过程经常操作会影响我们的开发效率，我们希望可以做到，当文件发生变化时，可以自动的完成编译和展示
- 为了完成自动编译，webpack提供了几种可选的方式：
  - webpack watch mode
  - webpack-dev-server（常用）
  - webpack-dev-middleware（比较少用）

#### 1.2 Webpack watch

- webpack给我们提供了watch模式

  - 在该模式下，webpack依赖图中的所有文件，只要有一个发生了更新，那么代码将被重新编译
  - 我们不需要手动去运行npm run build指令了

- 如何开启watch呢？两种方式：

  1. 方式一：在导出的配置中，添加watch: true

     ```
     //在webpack.config.js中
     module.exports = {
     	entry: xxx,
     	...
     	watch: true,
     	...
     }
     ```

  2. 方式二：在启动webpack的命令中，添加--watch的标识

     ```
     //在package.json中
     "scripts": {
     	"build": "webpack --watch"
     }
     ```

#### 1.3webpack-dev-server

- 上面的方式可以监听到文件的变化，但是事实上它本身是没有自动刷新浏览器的功能的

  - 当然，目前我们可以在VSCode中使用live-server来完成这样的功能
  - 但是，我们希望在不使用live-server的情况下，可以具备live reloading（实时重新加载）的功能

- 安装webpack-dev-server

  ```
  npm install webpack-dev-server -D
  ```

- 配置

  ```
  "scripts": {
  	"build": "webpack",
  	"serve": "webpack serve" //通过webpack-cli来解析执行
  }
  ```

- webpack-dev-server 在编译之后不会写入到任何输出文件，而是将bundle 文件保留在内存中：

  - 事实上webpack-dev-server使用了一个库叫memfs

- webpack-dev-server配置

  ```
  //在webpack.config.js中
  module.exports = {
  	entry: xxx,
  	...,
  	devServer: {
  		contentBase: "./public" //这个配置的作用是如果无法从webpack打包文件中获得资                                 //源，则从此文件夹加载资源，在开发中一般是public文件                                 //夹
  	}
  }
  ```

### 二、模块热替换

#### 2.1 认识模块热替换

- 什么是HMR呢
  - HMR的全称是Hot Module Replacement，翻译为模块热替换
  - 模块热替换是指在应用程序运行过程中，替换、添加、删除模块，而无需重新刷新整个页面
- HMR通过如下几种方式，来提高开发的速度
  - 不重新加载整个页面，这样可以保留某些应用程序的状态不丢失
  - 只更新需要变化的内容，节省开发的时间
  - 修改了css、js源代码，会立即在浏览器更新，相当于直接在浏览器的devtools中直接修改样式
- 如何使用HMR
  - 默认情况下，webpack-dev-server已经支持HMR，我们只需要开启即可
  - 在不开启HMR的情况下，当我们修改了源代码之后，整个页面会自动刷新，使用的是live reloading

#### 2.2 开启HMR

- 修改webpack配置

  ```
  //在webpack.config.js中
  devServer: {
  	contentBase: xxx,
  	hot: true
  },
  //一般情况下，为了稳定，还会设置target
  target: "web",  //设置运行环境，可选项为node或web
  ```

- 但是当我们修改了某一个模块的代码时，依然是刷新的整个页面

  - 这是因为我们需要去指定哪些模块发生更新时，进行HMR

    ```
    //在引用指定模块的文件中
    import "./js/element.js"
    
    if(module.hot) {
    	module.hot.accept("./js/element.js",() => {
    		... //回调函数为可选参数
    	})
    }
    ```

#### 2.3 框架的HMR

- 在开发其他项目时，我们是否需要经常手动去写入module.hot.accpet相关的API呢
  - 比如开发Vue、React项目，我们修改了组件，希望进行热更新，这个时候应该如何去操作呢
  - 事实上社区已经针对这些有很成熟的解决方案了
  - 比如vue开发中，我们使用vue-loader，此loader支持vue组件的HMR，提供开箱即用的体验
  - 比如react开发中，有React Hot Loader，实时调整react组件（目前React官方已经弃用了，改成使用react-refresh）

#### 2.3HMR原理解析

- HMR的原理是什么？如何可以做到只更新一个模块中的内容？
  - webpack-dev-server会创建两个服务：提供静态资源的服务（express）和Socket服务（net.Socket）
  - express server负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）
- HMR Socket Server，是一个socket的长连接
  - 长连接有一个最好的好处是建立连接后双方可以通信（服务器可以直接发送文件到客户端）
  - 当服务器监听到对应的模块发生变化时，会生成两个文件.json（manifest文件）和.js文件（update chunk）
  - 通过长连接，可以直接将这两个文件主动发送给客户端（浏览器）
  - 浏览器拿到两个新的文件后，通过HMR runtime机制，加载这两个文件，并且针对修改的模块进行更新

### 三、webpack-dev-server其他配置

#### 3.1 host配置

- host设置主机地址
  - 默认值是localhost，如果希望其他地方也可以访问，可以设置为0.0.0.0
- localhost 和0.0.0.0 的区别
  - localhost：本质上是一个域名，通常情况下会被解析成127.0.0.1
  - 127.0.0.1：回环地址(Loop Back Address)，表达的意思其实是我们主机自己发出去的包，直接被自己接收
    - 正常的数据库包经常应用层-传输层-网络层-数据链路层-物理层
    - 而回环地址，是在网络层直接就被获取到了，是不会经常数据链路层和物理层的
    - 比如我们监听127.0.0.1时，在同一个网段下的主机中，通过ip地址是不能访问的
  - 0.0.0.0：监听IPV4上所有的地址，再根据端口找到不同的应用程序
    - 比如我们监听0.0.0.0时，在同一个网段下的主机中，通过ip地址是可以访问的

#### 3.2 port、open、compress

- port设置监听的端口，默认情况下是8080
- open是否打开浏览器
  - 默认值是false，设置为true会打开浏览器
  - 也可以设置为类似于Google Chrome等值
- ompress是否为静态文件开启gzipcompression
  - 默认值是false，可以设置为true
  - html是不会进行压缩的，而js文件会被压缩

#### 3.4 proxy

- proxy是我们开发中非常常用的一个配置选项，它的目的设置代理来解决跨域访问的问题：

  - 比如我们的一个api请求是http://localhost:8888，但是本地启动服务器的域名是http://localhost:8000，这个时候发送网络请求就会出现跨域的问题
  - 那么我们可以将请求先发送到一个代理服务器，代理服务器和API服务器没有跨域的问题，就可以解决我们的跨域问题了

- 配置

  - target：表示的是代理到的目标地址，比如/api-hy/moment会被代理到http://localhost:8888/api-hy/moment

  - pathRewrite：默认情况下，我们的/api-hy也会被写入到URL中，如果希望删除，可以使用pathRewrite

  - secure：默认情况下不接受在 HTTPS 上运行且证书无效的后端服务器，如果希望支持，可以设置为false

  - changeOrigin：它表示是否更新代理后请求的headers中host地址

    ```
    //在webpack.config.json中
    module.exports = {
    	//...
    	devServer: {
    		proxy: {
    			'/api': {
    				target: 'http://localhost:8888'，
    				pathRewrite: {
    					'^/api': ''
    				},
    				secure: false,
    				changeOrigin: true
    			}
    		}
    	}
    }
	
- changeOrigin的解析

  - 这个changeOrigin官方说的非常模糊，通过查看源码发现其实是要修改代理请求中的headers中的host属性
    - 因为我们真实的请求，其实是需要通过http://localhost:8888来请求的
    - 但是因为使用了代码，默认情况下它的值时http://localhost:8000
    - 如果我们需要修改，那么可以将changeOrigin设置为true即可

### 四、其他配置

#### 4.1 resolve模块解析

- resolve用于设置模块如何被解析
  - 在开发中我们会有各种各样的模块依赖，这些模块可能来自于自己编写的代码，也可能来自第三方库
  - resolve可以帮助webpack从每个require/import 语句中，找到需要引入到合适的模块代码
  - webpack 使用enhanced-resolve 来解析文件路径
- webpack能解析三种文件路径：
  - 绝对路径
    - 由于已经获得文件的绝对路径，因此不需要再做进一步解析
  - 相对路径
    - 在这种情况下，使用import 或require 的资源文件所处的目录，被认为是上下文目录
    - 在import/require 中给定的相对路径，会拼接此上下文路径，来生成模块的绝对路径
  - 模块路径
    - 在resolve.modules中指定的所有目录检索模块，默认值是['node_modules']，所以默认会从node_modules中查找文件
    - 我们可以通过设置别名的方式来替换初识模块路径

#### 4.2确实文件还是文件夹

- 如果是一个文件：
  - 如果文件具有扩展名，则直接打包文件
  - 否则，将使用resolve.extensions选项作为文件扩展名解析
- 如果是一个文件夹
  - 会在文件夹中根据resolve.mainFiles配置选项中指定的文件顺序查找
    - resolve.mainFiles的默认值是['index']
    - 再根据resolve.extensions来解析扩展名

#### 4.3extensions和alias配置

- extensions是解析到文件时自动添加扩展名

  - 默认值是['.wasm', '.mjs', '.js', '.json']
  - 所以如果我们代码中想要添加加载.vue 或者jsx 或者ts等文件时，我们必须自己写上扩展名

- 另一个非常好用的功能是配置别名alias

  - 特别是当我们项目的目录结构比较深的时候，或者一个文件的路径可能需要../../../这种路径片段
  - 我们可以给某些常见的路径起一个别名

  ```
  //在webpack.config.js中
  module.exports = {
  	...
  	resolve: {
  		extensions: [".wasm", ".mjs", ".js", ".json", ".jsx", ".ts", ".vue"],
  		alias: {
  			"@": resolveApp("./src") // @/js即代表./src/js文件夹
  		}
  	}
  }
  ```

### 五、生产、开发环境分离

#### 5.1 如何区分开发环境

- 目前我们所有的webpack配置信息都是放到一个配置文件中的：webpack.config.js

  - 当配置越来越多时，这个文件会变得越来越不容易维护
  - 并且某些配置是在开发环境需要使用的，某些配置是在生成环境需要使用的，当然某些配置是在开发和生成环境都会使用的
  - 所以，我们最好对配置进行划分，方便我们维护和管理

- 那么，在启动时如何可以区分不同的配置呢

  - 方案一：编写两个不同的配置文件，开发和生成时，分别加载不同的配置文件即可

  - 方式二：

  - 在项目目录下新建一个config文件夹，建立三个文件，分别为webpack.comm.config.js（公共配置信息文件）、webpack.prod.config.js （生产环境所需配置文件）、webpack.dev.config.js（开发环境所需配置文件）

  - 通过一个插件将comm文件与生产、开发配置文件合并

    - 安装

      ```
      npm install webpack-merge -D
      ```

    - 合并

      ```
      //在prod、dev文件中
      const {merge} = require("webpack-merge");
      
      //引入comm文件
      const commonConf = require("./webpack.comm.config.js");
      
      //合并
      module.exports = merge(commonConf, {
      	// dev/prod的内容
      	...
      })
      ```

    - 修改打包命令

      ```
      //在package.json中
      "script": {
      	"build": "webpack --config ./config/webpack.prod.config.js",
      	"serve": "webpack serve --config. /config/webpack.config.dev.js"
      	//npm run build是生产环境
      	//npm run serve是开发环境
      }
      ```
    

#### 5.2 入口文件解析

- 我们之前编写入口文件的规则是这样的：./src/index.js，但是如果我们的配置文件所在的位置变成了config 目录，我们是否应该变成../src/index.js呢

  - 如果我们这样编写，会发现是报错的，依然要写成./src/index.js
  - 这是因为入口文件其实是和另一个属性时有关的context

- context的作用是用于解析入口（entry point）和加载器（loader）

  - 官方说法：默认是当前路径（但是经过测试，默认应该是webpack的启动目录）

  - 另外推荐在配置中传入一个值

    ```
    //context是配置文件所在目录
    module.exports = {
    	context: path.resolve(__diename, "./")，
    	entry: "../src/index.js"
    }
    ```

    ```
    //context是配置文件上一层目录
    module.exports = {
    	context: path.resolve(__dirname, "../"),
    	entry: "./src/index.js"
    }
    ```

    

