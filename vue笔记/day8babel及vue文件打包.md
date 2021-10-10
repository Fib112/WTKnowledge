[TOC]

### 一、babel

- Babel是一个工具链，主要用于旧浏览器或者环境中将ECMAScript 2015+代码转换为向后兼容版本的JavaScript；包括：语法转换、源代码转换等

#### 1.1 安装

```
npm install @babel/core -D
//安装babel预设
npm install @babel/preset-env -D
```

#### 1.2 babel底层原理

事实上我们可以将babel看成就是一个编译器，Babel编译器的作用就是将我们的源代码，转换成浏览器可以直接识别的另外一段源代码，Babel也拥有编译器的工作流程

- 解析阶段
- 转换阶段
- 生成阶段

#### 1.3 babel-loader

在实际开发中，我们通常会在构建工具中通过配置babel来对其进行使用的，比如在webpack中

安装

```
npm install babel-loader
```

在webpack.config.js中进行配置

```
module: {
	rules: [
		{
			test: /\.js$/,
			use: {
				loader: "babel"
			}
		}
	]
}
```

添加预设

常见的预设有三种：env、react、typescript

配置预设

```
{
	test: /\.js$/,
	use: {
		loader: "babel",
		options: {
			presets: [
				["@babel/preset-env"]
			]
		}
	}
}
```

像之前一样，我们可以将babel的配置信息放到一个独立的文件中，在根目录下新建babel.config.js

```
module.exports = {
	presets: [
		["@babel/preset-env"]
	]
}
```

### 二、vue源码打包

#### 2.1 vue源码打包

vue打包后有几种不同的源码包，在执行`npm install vue@next`后，vue源码包包括以下几种类型的包

- vue(.runtime).global(.prod).js
  - 通过浏览器中的`<script src="...">`直接使用
  - 我们之前通过CDN引入和下载的Vue版本就是这个版本
  - 会暴露一个全局的Vue来使用
- vue(.runtime).esm-browser(.prod).js
  - 用于通过原生ES 模块导入使用(在浏览器中通过`<script type="module">`来使用)
- vue(.runtime).esm-bundler.js
  - 用于webpack，rollup 和parcel 等构建工具
  - 构建工具中默认是vue.runtime.esm-bundler.js
  - 如果我们需要解析模板template，那么需要手动指定vue.esm-bundler.js
- vue.cjs(.prod).js
  - 服务器端渲染使用
  - 通过require()在Node.js中使用

#### 2.2 运行时+编译器 vs 仅运行时

- 在Vue的开发过程中我们有三种方式来编写DOM元素

  1. 方式一：template模板的方式（之前经常使用的方式）
  2. 方式二：render函数的方式，使用h函数来编写渲染的内容
  3. 方式三：通过.vue文件中的template来编写模板
- 它们的模板分别是如何处理的呢

  - 方式二中的h函数可以直接返回一个虚拟节点，也就是Vnode节点

  - 方式一和方式三的template都需要有特定的代码来对其进行解析

    - 方式三.vue文件中的template可以通过在vue-loader对其进行编译和处理
    - 方式一中的template我们必须要通过源码中一部分代码来进行编译
- 所以，Vue在让我们选择版本的时候分为运行时+编译器 vs仅运行时
  - 运行时+编译器包含了对template模板的编译代码，更加完整，但是也更大一些
  - 仅运行时没有包含对template版本的编译代码，相对更小一些

#### 2.3sfc文件

- VSCode对SFC的支持

  - 插件一：Vetur，从Vue2开发就一直在使用的VSCode支持Vue的插件
  - 插件二：Volar，官方推荐的插件（后续会基于Volar开发官方的VSCode插件）

- 编写App.vue代码

  - 在src文件夹下新建vue文件夹，在vue文件夹下新建App.vue文件，vue文件夹下可以放其他组件

  - 在main.js（入口文件）处先import vue，`import {createApp} from "vue"`，因为在sfc文件下vue-loader会使用@vue/compiler-sfc来对template进行解析，所以可以这样引入vue

  - 然后在.vue文件中写template、script和css代码

    ```vue
    <template>
    	<h2>
            {{title}}
        </h2>
    	<p>
            {{content}}
        </p>
    </template>
    
    <script>
    	export default { //导出
            data() {
                return {
                    xxx
                }
            },
            methods: {
                xxx
            }
        }
    </script>
    
    <style>
        h2 {
            color: red;
        }
        p {
            color: blue;
        }
    </style>
    ```

  - 在main.js中引用

    ```
    import App from "./vue/App.vue";
    
    createApp(App).mount("#app");
    ```

- 编写其他组件代码

  - 在vue文件夹下添加xxx.vue文件，文件结构与App.vue文件一致

  - 在App.vue文件中注册

    ```vue
    <template>
    	<h2>xxxxxxx</h2>
    	<xxx> </xxx> //使用xxx组件
    </template>
    
    <script>
    	import xxx from "./xxx.vue";
        export default = {
            components: {
                xxx
            }
        }
    </script>
    ```

- 打包.vue文件

  - 在对vue文件进行打包时会报错，提醒我们使用合适的loader来处理文件

  - 此时，我们需要使用vue-loader:

    ```
    npm install vue-loader@next -D
    ```

    配置

    ```
    {
    	test: /\.vue$/,
    	loader: "vue-loader"
    }
    ```

  - 在配置好后，打包依然会报错，这是因为vue-loader还需要另一个插件——@vue/compiler-sfc来对template进行解析

    安装

    ```
    npm install @vue/compiler-sfc -D
    ```

    配置

    ```
    const {VueLoaderPlugin} = require('vue-loader/dist/index');
    
    //在plugins处
    new VueLoaderPlugin()
    ```

    之后，就可以正常打包了

- 全局标识的配置

  在vue3.0.0-rc.3版本后，esm-bundler会有两个全局标识：

  1. `__VUE_OPTIONS_API__`: 对vue2做适配，默认是true，如果在代码中没有使用options-api可以关闭以减少代码
  2. `__VUE_PROD_DEVTOOLS__`：在production模式下是否支持dev-tools工具，默认是关闭

  我们可以在DefinePlugin进行配置：

  ```
  new DefinePlugin({
  	__VUE_OPTIONS_API__: true,
  	__VUE_PROD_DEVTOOLS__: false
  })
  ```

  
