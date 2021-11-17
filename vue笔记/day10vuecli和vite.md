[TOC]



### 一、vue-cli使用

- 安装

  使用全局安装，这样在任何时候都可以通过vue的命令来创建项目

  ```
  npm install @vue/cli -g
  ```

- 升级vue

  ```
  npm update @vue/cli -g
  ```

- 通过vue的命令来创建项目

  ```
  vue create 项目名称
  ```

- 项目运行及打包

  ```
  //在本地运行
  npm run serve
  //打包项目
  npm run build
  ```

### 二、vite使用

#### 2.1vite构造

vite主要由两部分组成：

- 一个开发服务器，它基于原生ES模块提供了丰富的内建功能，HMR的速度非常快速
- 一套构建指令，它使用rollup打开我们的代码，并且它是预配置的，可以输出生成环境的优化过的静态资源

#### 2.2 vite的安装与使用

- 安装

  ```
  npm install vite -g //全局安装
  npm install vite -D //局部安装
  ```

- 启动项目

  ```
  npx vite
  ```

#### 2.3 vite对css及less的支持

- vite可以直接支持css的处理，直接导入css文件即可

- vite可以支持css预处理器

  - 直接导入less文件

  - 安装less编译器

    ```
    npm install less -D
    ```

- vite还支持postcss的转换

  - 安装postcss及预设

    ```
    npm install postcss postcss-preset-env -D
    ```

  - 配置预设

    - 创建postcss.config.js配置文件

    - 进行配置

      ```
      module.exports = {
      	plugin: [
      		require("postcss-preset-env")
      	]
      }
      ```

#### 2.4 vite对typescript的支持

- vite对TypeScript是原生支持的，它会直接使用ESBuild来完成编译，只需要直接导入即可
- 如果我们查看浏览器中的请求，会发现请求的依然是ts的代码
  - 这是因为vite中的服务器Connect会对我们的请求进行转发
  - 获取ts编译后的代码，给浏览器返回，浏览器可以直接进行解析

#### 2.5 vite对vue的支持

- vite对vue提供第一优先级支持

  - Vue 3 单文件组件支持：@vitejs/plugin-vue
  - vue3 jsx支持：@vitejs/plugin-vue-jsx
  - vue2支持：underfin/vite-plugin-vue2

- 安装支持vue的插件

  ```
  npm install @vitejs/plugin-vue -D
  //安装vue插件
  npm install @vue/compiler-sfc -D
  ```

- 创建vite.config.js文件，进行配置

  ```
  import vue from '@vitejs/plugin-vue'
  
  module.exports = {
  	plugins: [
  		vue()
  	]
  }
  ```

#### 2.6 vite打包项目

- vite在打包时会预打包项目的依赖到node_modules中的.vite文件中，在再次执行打包的时候由于已经有缓存，所以打包速度会快很多

- 可以直接通过vitebuild来完成对当前项目的打包

  ```
  npx vite build
  ```

- 我们可以通过preview的方式，开启一个本地服务来预览打包后的效果

  ```
  npx vite preview
  ```

- 在真实开发中，一般会配置script来更改命令

  ```
  //在package.json中
  "scripts": {
  	"serve": "vite",
  	"build": "vite build",
  	"preview": "vite preview"
  }
  ```

  在需要运行项目时，使用`npm run serve`，在打包时使用`npm run build`，在预览时使用`npm run preview`

#### 2.7ESBuild

- vite之所以这么快，其中一个原因是因为使用了ESBuild而非babel
- ESBuild的特点
  - 超快的构建速度，并且不需要缓存
  - 支持ES6和CommonJS的模块化
  - 支持ES6的Tree Shaking
  - 支持Go、JavaScript的API
  - 支持TypeScript、JSX等语法编译
  - 支持SourceMap
  - 支持代码压缩
  - 支持扩展其他插件
- ESBuild为什么这么快
  - 使用Go语言编写的，可以直接转换成机器代码，而无需经过字节码
  - ESBuild可以充分利用CPU的多内核，尽可能让它们饱和运行
  - ESBuild的所有内容都是从零开始编写的，而不是使用第三方，所以从一开始就可以考虑各种性能问题

#### 2.8 使用vite脚手架创建项目

- 在开发中，我们不可能所有的项目都使用vite从零去搭建，比如一个react项目、Vue项目，这个时候vite还给我们提供了对应的脚手架工具

- 所以Vite实际上是有两个工具的

  - vite：相当于是一个构件工具，类似于webpack、rollup
  - @vitejs/create-app：类似vue-cli、create-react-app

- 安装与使用

  ```
  npm install @vitejs/create-app -g
  create-app 项目名称
  //由于创建项目时脚手架不会安装相关依赖，所以需要手动安装一下
  npm install
  ```

- 打包

  ```
  //运行项目
  npm run dev
  //打包项目
  npm run build
  //预览项目
  npm run serve
  ```

  