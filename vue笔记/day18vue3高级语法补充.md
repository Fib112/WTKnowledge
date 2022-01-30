### 一、自定义指令

在Vue的模板语法中我们学习过各种各样的指令：v-show、v-for、v-model等等，除了使用这些指令之外，Vue也允许我们来自定义自己的指令

- 注意：在Vue中，代码的复用和抽象主要还是通过组件
- 通常在某些情况下，需要对DOM元素进行底层操作，这个时候就会用到自定义指令

自定义指令分为两种：

- 自定义局部指令：组件中通过directives 选项，只能在当前组件中使用
- 自定义全局指令：app的directive 方法，可以在任意组件中被使用

#### 1.1 自定义指令简单应用

简单案例实现：当某个元素挂载完成后可以自动获取焦点：

- 实现方式一：如果我们使用默认的实现方式

  ```vue
  <template>
  	<div>
          <input type="text" ref="inputRef">
      </div>
  </template>
  
  <script>
  	import { ref, onMounted } from 'vue';
      
      export default {
          setup() {
              const inputRef = ref(null);
              
              onMounted(() => {
                  inputRef.value.focus();
              })
              
              return {
                  inputRef
              }
          }
      }
  </script>
  ```

  使用默认实现方式在多个页面应用时，需要编写大量重复代码

- 实现方式二：自定义一个v-focus 的局部指令

  - 这个自定义指令实现非常简单，我们只需要在组件选项中使用directives 即可

  - 它是一个对象，在对象中编写我们自定义指令的名称（注意：这里不需要加v-）

  - 自定义指令有一个生命周期，是在组件挂载后调用的mounted，我们可以在其中完成操作

    ```vue
    <template>
    	<div>
            <input type="text" v-focus>
        </div>
    </template>
    
    <script>
    	export deault {
            directives: {
                focus: {
                    mounted(el) {
                        el.focus();
                    }
                }
            }
        }
    </script>
    ```

    

- 实现方式三：自定义一个v-focus 的全局指令

  自定义一个全局的v-focus指令可以让我们在任何地方直接使用

  ```js
  //main.js
  const app = createApp(App);
  
  app.directive("focus", { //传入两个参数，分别是指令名称(string)以及指令内容(包裹在对象中)
      mounted(el) {
          el.focus();
      }
  });
  app.mount("#app");
  ```

#### 1.2 指令的生命周期

一个指令定义的对象，Vue提供了如下的几个钩子函数：

- created：在绑定元素的attribute 或事件监听器被应用之前调用
- beforeMount：当指令第一次绑定到元素并且在挂载父组件之前调用
- mounted：在绑定元素的父组件被挂载后调用
- beforeUpdate：在更新包含组件的VNode 之前调用
- updated：在包含组件的VNode 及其子组件的VNode 更新后调用
- beforeUnmount：在卸载绑定元素的父组件之前调用
- unmounted：当指令与元素解除绑定且父组件已卸载时，只调用一次

所有的钩子函数都会接收四个参数(el, bindings, vnode, preVnode)

#### 1.3 指令的参数和修饰符

自定义指令可以接受一些参数或者修饰符：

```vue
<button v-why:info.aaa.bbb="'coderwhy'">{{counter}}</button> //传入字符串需要使用单引号，如果是变量就只需包裹在双引号中，如 v-why:info.aaa.bbb="counter"
```

- info是参数的名称
- aaa bbb是修饰符的名称，值为Boolean，只要调用了这个修饰符就为true，所以aaa bbb都为true
- 后面是传入的具体的值

在生命周期中，可以通过参数中的bindings 获取到对应的内容：

- 参数在arg中，修饰符在modifiers中，传入的值在value中

#### 1.4 自定义指令综合案例

自定义指令案例：时间戳的显示需求：

- 在开发中，大多数情况下从服务器获取到的都是时间戳
- 我们需要将时间戳转换成具体格式化的时间来展示
- 在Vue2中我们可以通过过滤器来完成
- 在Vue3中我们可以通过计算属性（computed）或者自定义一个方法（methods）来完成
- 其实我们还可以通过一个自定义的指令来完成

在案例中通过编写自定义指令v-format-time实现：

- 时间格式化指令会在多处频繁使用，所以应该注册成全局指令：

  - 创建一个directives文件夹存放所有自定义指令文件，所有自定义指令的统一出口为index.js文件

    ```javascript
    //index.js
    import registerFormatTime from './format-time'
    
    export default function registerDirectives(app) {
        registerFormatTime(app);
    }
    ```

- 然后编写自定义指令文件

  - 具体转换格式通过第三方库dayjs来实现，所以需要在项目中安装依赖：`npm install dayjs`

  - 在使用指令时，还应该接收参数来让用户自定义时间格式，具体代码如下：

    ```javascript
    import dayjs from 'dayjs';
    
    export function(app) {
        app.directive("format-time", {
            created(el, bindings) {
                bindings.formatString = "YYYY-MM-DD HH:mm:ss";
                if(bindings.value) {
                    bindings.formatString = bindings.value;
                }
            },
            mounted(el, bindings) {
                const textContent = el.textContent;
                let timeStamp = parseInt(textContent);
                if(textContent === 10) {
                    timeStamp *= 1000;
                }
                el.textContent = dayjs(timeStamp).format(bindings.formatString);
            }
        })
    }
    ```

- 在main.js中注册：

  ```javascript
  import registerDirectives from './directives'
  
  const app = createApp(App);
  
  registerDirectives(app);
  app.mount("#app");
  ```

### 二、teleport使用

#### 2.1 认识Teleport

在组件化开发中，我们封装一个组件A，在另外一个组件B中使用：

- 那么组件A中template的元素，会被挂载到组件B中template的某个位置
- 最终我们的应用程序会形成一颗DOM树结构

但是某些情况下，我们希望组件不是挂载在这个组件树上的，可能是移动到Vue app之外的其他位置：

- 比如移动到body元素上，或者我们有其他的div#app之外的元素上
- 这个时候我们就可以通过teleport来完成

Teleport是什么呢

- 它是一个Vue提供的内置组件，类似于react的Portals
- teleport翻译过来是心灵传输、远距离运输的意思，它有两个属性：
  - to：指定将其中的内容移动到的目标元素，可以使用选择器
  - disabled：是否禁用 teleport 的功能

简单应用：

```vue
//App.vue
<template>
	<div>
        <teleport to="#why">
    		<h2>coderwhy</h2>
    	</teleport>
    </div>
</template>
```

此时teleport中的`<h2>`标签会挂载到外层id为why的元素上，而不是App.vue中的`<div>`标签

#### 2.2 和组件结合使用

teleport也可以和组件结合一起来使用，我们可以在teleport 中使用组件，并且也可以给他传入一些数据：

```vue
<template>
	<div>
        <teleport to="#why">
            <hello-world message="我是App中的message"></hello-world>
    	</teleport>
    </div>
</template>

<script>
	import HelloWorld from './HelloWorld.vue'
    
    export default {
        components: {
            HelloWorld
        }
    }
</script>
```

#### 2.3 多个teleport

如果我们将多个teleport应用到同一个目标上（to的值相同），那么这些目标会进行合并：

```vue
<template>
	<teleport to="#why">
    	<h2>coderwhy</h2>
    </teleport>
	<teleport to="#why">
    	<hello-world message="我是来自App的message"></hello-world>
    </teleport>
</template>

<script>
	import HelloWorld from './HelloWorld.vue';
    
    export default {
        components: {
            HelloWorld
        }
    }
</script>
```

### 三、vue插件

#### 3.1 认识vue插件

通常我们向Vue全局添加一些功能时，会采用插件的模式，它有两种编写方式：

- 对象类型：一个对象，但是必须包含一个install 的函数，该函数会在安装插件时执行
- 函数类型：一个function，这个函数会在安装插件时自动执行

插件可以完成的功能没有限制，比如下面的几种都是可以的：

- 添加全局方法或者property，通过把它们添加到config.globalProperties上实现
- 添加全局资源：指令/过滤器/过渡等
- 通过全局mixin来添加一些组件选项
- 一个库，提供自己的API，同时提供上面提到的一个或多个功能

#### 3.2 插件的编写方式

建立一个plugins文件夹，里面放插件文件

- 对象类型写法：

  ```javascript
  //plugin_object.js
  export default {
      install(app) { //vue会自动调用install方法并将app实例作为参数传进来
          //设置全局属性，为避免与组件中数据名冲突，一般采用$开头
          app.config.golbalProperties.$name = "coderwhy"
      }
  }
  ```

  ```javascript
  //main.js
  import pluginObject from './plugins/plugins_object' //导入插件对象
  
  const app = createApp(App);
  
  app.use(pluginObject); //使用use方法注册插件
  ```

  ```vue
  //App.vue optionsAPI中使用全局属性
  <script>
  	export default {
          mounted() {
              console.log(this.$name); //生命周期中可以通过this指针获取app
          }
          methods: {
              foo() {
                  console.log(this.$name); //在methods中的方法也可以通过this访问app从而拿到globalProperties
              }
          }
      }
  </script>
  ```

  ```vue
  //App.vue setupAPI中使用全局属性
  <script>
  	import { getCurrentInstance } from 'vue'; //需要使用这个API访问组件实例
      
      export default {
          setup() {
              const instance = getCurrentInstance();
              console.log(instance.appContext.config.globalProperties.$name);
          }
      }
  </script>
  ```

- 函数类型写法：

  ```javascript
  //plugins_Function.js
  export default function(app) {
      //函数类型插件在参数列表可以获取app，同样是通过app.use注册
  }
  ```

  
