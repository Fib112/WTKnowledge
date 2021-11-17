### 一、动态组件与keep-alive

#### 1.1 动态组件

- 切换组件案例

  - 我们现在想要实现了一个功能：
    - 点击一个tab-bar，切换不同的组件显示
  - 这个案例我们可以通过两种不同的实现思路来实现
    - 方式一：通过v-if来判断，显示不同的组件
    - 方式二：动态组件的方式

- 动态组件实现

  - 动态组件是使用component 组件，通过一个特殊的attribute is 来实现

    ```vue
    //App.vue
    <component :is="currentTab"></component>
    ```

  - 这个currentTab的值需要是什么内容呢

    - 可以是通过component函数注册的组件
    - 在一个组件对象的components对象中注册的组件

- 动态组件的传值

  - 如果是动态组件我们可以给它们传值和监听事件吗
    - 也是一样的，只是我们需要将属性和监听事件放到component上来使用

#### 1.2 keep-alive

- 默认情况下，我们在切换组件后，之前的组件会被销毁掉，再次回来时会重新创建组件

- 但是，在开发中某些情况我们希望继续保持组件的状态，而不是销毁掉，这个时候我们就可以使用一个内置组件：keep-alive

  ```vue
  <keep-alive>
  	<component :is="currentBar"></component>
  </keep-alive>
  ```

- keep-alive有一些属性:

  - include -string | RegExp | Array。只有名称匹配的组件会被缓存

    ```vue
    //String
    <keep-alive include="a,b"> //ab间不要加空格
    	<component :is="currentBar"></component>
    </keep-alive>
    //RegExp
    <keep-alive :include="/a|b/"> //使用正则表达式需要用v-bind绑定
    	<component :is="currentBar"></component>
    </keep-alive>
    //Array
    <keep-alive :include="["a", "b"]"> //使用数组也需要用v-bind绑定
    	<component :is="currentBar"></component>
    </keep-alive>
    ```

  - exclude-string | RegExp | Array。任何名称匹配的组件都不会被缓存

  - max-number | string。最多可以缓存多少组件实例，一旦达到这个数字，那么缓存组件中最近没有被访问的实例会被销毁

- include 和exclude prop 允许组件有条件地缓存：

  - 二者都可以用逗号分隔字符串、正则表达式或一个数组来表示
  - 匹配首先检查组件自身的name 选项，所以需要在对应组件加上name属性

### 二、异步组件·

#### 2.1 Webpack的代码分包

- 默认的打包过程：

  - 默认情况下，在构建整个组件树的过程中，因为组件和组件之间是通过模块化直接依赖的，那么webpack在打包时就会将组件模块打包到一起（比如一个app.js文件中）
  - 这个时候随着项目的不断庞大，app.js文件的内容过大，会造成首屏的渲染速度变慢

- 打包时，代码的分包：

  - 所以，对于一些不需要立即使用的组件，我们可以单独对它们进行拆分，拆分成一些小的代码块chunk.js
  - 这些chunk.js会在需要时从服务器加载下来，并且运行代码，显示对应的内容

- 分包步骤

  ```vue
  //使用import函数加载模块文件，返回一个promise对象
  //通过import函数导入的模块，webpack在打包时会对它进行分包操作
  import("./utils/math").then(res => {
  	console.log(res.sum(20, 30));
  })
  ```

#### 2.2 vue中实现异步组件

- 如果我们的项目过大了，对于某些组件我们希望通过异步的方式来进行加载（目的是可以对其进行分包处理），那么Vue中给我们提供了一个函数：defineAsyncComponent

- defineAsyncComponent接受两种类型的参数：

  - 类型一：工厂函数，该工厂函数需要返回一个Promise对象，正好使用import函数

    ```vue
    //App.vue
    <script>
    	import { defineAsyncComponent } from 'vue';
        const AsyncHome = defineAsyncComponent(() => import("./AsyncHome.vue"))
        
        export default {
            components: {
                AsyncHome
            }
        }
    </script>
    ```

  - 类型二：接受一个对象类型，对异步函数进行配置

    ```vue
    const AsyncHome = defineAsyncComponent({
    	loader: () => import("./AsyncHome.vue"),
    	//加载过程中显示的组件
    	loadingComponent: Loading //Loading为注册组件
    	//加载失败时显示的组件
    	errorComponent: Error, //Error为注册组件
    	//在显示 loadingComponent 之前的延时 | 默认值：200(ms)
    	delay: 2000
    })
    ```

#### 2.3 异步组件与Suspense

- Suspense显示的是一个实验性的特性，API随时可能会修改

- Suspense是一个内置的全局组件，该组件有两个插槽：

  - default：如果default可以显示，那么显示default的内容

  - fallback：如果default无法显示，那么会显示fallback插槽的内容

    ```vue
    <suspense>
    	<template #default>
        	<async-home></async-home> //async-home为异步组件
        </template>
        <template #fallback>
        	<loading></loading> //loading为注册组件
        </template>
    </suspense>
    ```

    

