### 一、setup中的生命周期及provide、inject

#### 1.1 生命周期钩子

setup 可以用来替代data 、methods 、computed 、watch 等等这些选项，也可以替代生命周期钩子，在setup中可以使用直接导入的onX函数注册生命周期钩子

| options API   | hook inside setup |
| ------------- | ----------------- |
| beforeCreate  | Not needed        |
| created       | Not needed        |
| beforeMount   | onBeforeMount     |
| mounted       | onMounted         |
| beforeUpdate  | onBeforeUpdate    |
| updated       | onUpdated         |
| beforeUnmount | onBeforeUnmount   |
| unmounted     | onUnmounted       |
| activated     | onActivated       |
| deactivated   | onDeactivated     |

因为`setup`是围绕`beforeCreate`和`created`生命周期钩子进行的，所以不需要显式定义他们。换句话说，在这些钩子中编写的代码都应该在`setup`函数中编写

#### 1.2 provide与inject

之前还学习过Provide和Inject，Composition API也可以替代之前的Provide 和Inject 的选项

- 我们可以通过provide来提供数据：
  - 可以通过provide 方法来定义每个Property
  - provide可以传入两个参数：
    - name：提供的属性名称
    - value：提供的属性值
      ```js
      import { provide } from 'vue';
      let counter = 100;
      let info = {
          name: "why",
          age: 10
      }
      
      provide("counter", counter);
      provide("info", info);
      ```

- 在后代组件中可以通过inject 来注入需要的属性和对应的值：
  - 可以通过inject 来注入需要的内容
  - inject可以传入两个参数：
    - 要inject 的property 的name
    - 默认值
      ```js
      import { inject } from 'vue';
      
      const counter = inject("counter");
      const info = inject("info");
      ```

- 数据的响应式

  为了增加provide 值和inject 值之间的响应性，我们可以在provide 值时使用ref 和reactive

  ```javascript
  let counter = ref(100);
  let info = reactive({
      name: "why",
      age: 18
  })
  
  provide("counter", counter);
  provide("info", info);
  ```

- 修改响应式Property

  如果我们需要修改可响应的数据，那么最好是在数据提供的位置来修改，所以在provide的时候最好提供的是readonly对象：

  ```js
  import { provide, ref, readonly } from 'vue';
  
  let counter = ref(100);
  let info = reactive({
      name: "why",
      age: 18
  })
  
  provide("counter", readonly(counter));
  provide("info", readOnly(info));
  ```

  或者，我们可以将修改方法进行共享，在后代组件中进行调用：

  ```js
  const changeInfo = () => {
      info.name = "coderwhy";
  }
  provide("changeInfo", changeInfo);
  ```

#### 1.3 composition API综合案例

- useCounter

  ```vue
  <template>
  	<div>{{counter}}</div>
  	<div>{{doubleCounter}}</div>
  	<button @click="increment">+1</button>
  	<button @click="decrement">-1</button>
  </template>
  
  <script>
      import { ref, computed } from 'vue';
  	setup() {
          const counter = ref(0);
          const doubleCounter = computed(() => counter*2);
          
          const increment = () => counter.value++;
          const decrement = () => counter.value--;
          
          return {
              counter,
              doubleCounter,
              increment,
              decrement
          }
      }
  </script>
  ```

  但在实际开发中，会将counter的代码逻辑抽取到一个独立的hook中：

  ```js
  //useCounter.js
  import { ref, computed } from 'vue';
  
  export function useCounter() {
      const counter = ref(0);
      const doubleCounter = computed(() => counter*2);
          
      const increment = () => counter.value++;
      const decrement = () => counter.value--;
          
      return {
      	counter,
      	doubleCounter,
      	increment,
      	decrement
      }
  }
  ```

  ```vue
  //App.vue
  import useCounter from './hooks/useCounter';
  
  export default {
  	setup() {
  		const { counter, doubleCounter, increment, decrement } = useCounter();
  		return {
  			counter,
  			doubleCounter,
  			increment,
  			decrement
  		}
  	}
  }
  ```

- useTitle

  编写一个可以重复修改title的Hook：

  ```js
  //useTitle.js
  import { ref, watch } from 'vue';
  
  export function useTitle(title) {
      const titleRef = ref(title);
      
      //因为需要进行重复修改，所以需要使用watch监听
      watch(titleRef, (newValue) => {
          document.title = newValue;
      }, {
          immediate: true; //因为watch是惰性的，所以需要立即执行一次将title赋值给document.title
      })
      
      return titleRef
  }
  ```

  ```vue
  //App.vue
  <script>
  	import useTitle from './hooks/useTitle';
  
  	export default {
  		setup() {
  			const title = useTitle("coderwhy");
  			setTimeout(() => {
  				title.value = 'kobe';  //两秒后修改标题
  			}, 2000)
  		}
  	}
  </script>
  ```

### 二、单文件组件`<script setup>`

`<script setup>`是在单文件组件 (SFC) 中使用组合式 API的编译时语法糖。相比于普通的 `<script>` 语法，它具有更多优势：

- 更少的样板内容，更简洁的代码
- 能够使用纯 Typescript 声明 props 和抛出事件
- 更好的运行时性能 (其模板会被编译成与其同一作用域的渲染函数，没有任何的中间代理)
- 更好的 IDE 类型推断性能 (减少语言服务器从代码中抽离类型的工作)

#### 2.1 基本语法

要使用这个语法，需要将 `setup` attribute 添加到 `<script>` 代码块上：

```vue
<script setup>
console.log('hello script setup')
</script>
```

里面的代码会被编译成组件 `setup()` 函数的内容。这意味着与普通的 `<script>` 只在组件被首次引入的时候执行一次不同，`<script setup>` 中的代码会在**每次组件实例被创建的时候执行**

#### 2.2 顶层的绑定会被暴露给模板

当使用 `<script setup>` 的时候，任何在 `<script setup>` 声明的顶层的绑定 (包括变量，函数声明，以及 import 引入的内容) 都能在模板中直接使用：

```vue
<template>
  <div @click="log">{{ msg }}</div>
</template>

<script setup>
// 变量
const msg = 'Hello!'

// 函数
function log() {
  console.log(msg)
}
</script>
```

import 导入的内容也会以同样的方式暴露。意味着可以在模板表达式中直接使用导入的 helper 函数，并不需要通过 `methods` 选项来暴露它：

```vue
<template>
  <div>{{ capitalize('hello') }}</div>
</template>

<script setup>
import { capitalize } from './helpers'
</script>
```

#### 2.3 响应式

响应式状态需要明确使用响应式 APIs 来创建。和从 `setup()` 函数中返回值一样，ref 值在模板中使用的时候会自动解包：

```vue
<template>
  <button @click="count++">{{ count }}</button>
</template>

<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>
```

#### 2.4 使用组件

`<script setup>` 范围里的值也能被直接作为自定义组件的标签名使用：

```vue
<template>
  <MyComponent />
</template>

<script setup>
	import MyComponent from './MyComponent.vue'
</script>
```

#### 2.5 动态组件

由于组件被引用为变量而不是作为字符串键来注册的，在 `<script setup>` 中要使用动态组件的时候，就应该使用动态的 `:is` 来绑定：

```vue
<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>

<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>
```

#### 2.6 defineProps和 defineEmits

在 `<script setup>` 中必须使用 `defineProps` 和 `defineEmits` API 来声明 `props` 和 `emits` ，它们具备完整的类型推断并且在 `<script setup>` 中是直接可用的：

```vue
<script setup>
const props = defineProps({
  foo: String
})

const emit = defineEmits(['change', 'delete'])
// setup code
</script>
```

- `defineProps` 和 `defineEmits` 都是只在 `<script setup>` 中才能使用的**编译器宏**。他们不需要导入且会随着 `<script setup>` 处理过程一同被编译掉
- `defineProps` 接收与 [`props` 选项](https://v3.cn.vuejs.org/api/options-data.html#props)相同的值，`defineEmits` 也接收 [`emits` 选项](https://v3.cn.vuejs.org/api/options-data.html#emits)相同的值
- `defineProps` 和 `defineEmits` 在选项传入后，会提供恰当的类型推断
- 传入到 `defineProps` 和 `defineEmits` 的选项会从 setup 中提升到模块的范围。因此，传入的选项不能引用在 setup 范围中声明的局部变量。这样做会引起编译错误。但是，它*可以*引用导入的绑定，因为它们也在模块范围内
- 如果使用了 Typescript，使用纯类型声明来声明 prop 和 emits也是可以的

#### 2.7 useSlots和 useAttrs

在 `<script setup>` 使用 `slots` 和 `attrs` 的情况应该是很罕见的，因为可以在模板中通过 `$slots` 和 `$attrs` 来访问它们。在你的确需要使用它们的罕见场景中，可以分别用 `useSlots` 和 `useAttrs` 两个辅助函数：

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</script>
```

`useSlots` 和 `useAttrs` 是真实的运行时函数，它会返回与 `setupContext.slots` 和 `setupContext.attrs` 等价的值，同样也能在普通的组合式 API 中使用

#### 2.8 与普通的 `<script>` 一起使用

`<script setup>`可以和普通的 `<script>` 一起使用。普通的` <script>` 在有这些需要的情况下或许会被使用到：

- 无法在 `<script setup>` 声明的选项，例如 `inheritAttrs` 或通过插件启用的自定义的选项
- 声明命名导出
- 运行副作用或者创建只需要执行一次的对象

```vue
<script>
// 普通 <script>, 在模块范围下执行(只执行一次)
runSideEffectOnce()

// 声明额外的选项
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// 在 setup() 作用域中执行 (对每个实例皆如此)
</script>
```

#### 2.9使用类型声明时的默认 props 值

仅限类型的 `defineProps` 声明的不足之处在于，它没有可以给 props 提供默认值的方式。为了解决这个问题，提供了 `withDefaults` 编译器宏：

```ts
interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```

上面代码会被编译为等价的运行时 props 的 `default` 选项。此外，`withDefaults` 辅助函数提供了对默认值的类型检查，并确保返回的 `props` 的类型删除了已声明默认值的属性的可选标志

#### 2.10 限制：没有 Src 导入

由于模块执行语义的差异，`<script setup>` 中的代码依赖单文件组件的上下文。当将其移动到外部的 `.js` 或者 `.ts` 文件中的时候，对于开发者和工具来说都会感到混乱。因而 **`<script setup>`** 不能和 `src` attribute 一起使用

### 三、render函数

Vue推荐在绝大数情况下使用模板来创建HTML，然后一些特殊的场景，真的需要JavaScript的完全编程的
能力，这个时候可以使用渲染函数，它比模板更接近编译器

#### 3.1 认识h函数

VNode和VDOM的改变：

- Vue在生成真实的DOM之前，会将节点转换成VNode，而VNode组合在一起形成一颗树结构，就是虚拟DOM（VDOM）
- 事实上，之前编写的template中的HTML最终也是使用渲染函数生成对应的VNode
- 那么，如果想充分的利用JavaScript的编程能力，可以自己来编写createVNode函数，生成对应的
  VNode

h()函数：

- h() 函数是一个用于创建vnode的一个函数
- 其实更准确的命名是createVNode() 函数，但是为了简便在Vue将之简化为h()函数

#### 3.2 h函数的使用

h()函数接受三个参数：

- {String | Object | Function} 一个 HTML 标签名、一个组件、一个异步组件、或一个函数式组件，是必需的。比如'div'

- {Object} 与 attribute、prop 和事件相对应的对象，这会在模板中用到，是可选的。比如{class: "why"}

- {String | Array | Object} children子 VNodes, 使用 `h()` 构建,或使用字符串获取 "文本 VNode"或者有插槽的对象，是可选的。比如：

  ​	

  ```js
   [
        'Some text comes first.',
        h('h1', 'A headline'),
        h(MyComponent, {
          someProp: 'foobar'
        })
      ]
  ```

如果没有 prop，那么通常可以将 children 作为第二个参数传入。如果会产生歧义，可以将 `null` 作为第二个参数传入，将 children 作为第三个参数传入

h函数可以在两个地方使用：

- render函数选项中

  ```vue
  //由于使用了render函数，所以不需要编写template代码了
  <script>
  	import { h } from 'vue';
      
      export default {
          render() {
              return h("h2", {class: "title"}, "Hello Render");
          }
      }
  </script>
  ```

- setup函数选项中（setup本身需要是一个函数类型，函数再返回h函数创建的VNode）

  ```vue
  <script>
  	import { h } from 'vue';
      
      export default {
          setup() {
              return () => h("h2", {class: "title"}, "Hello Render")
          }
      }
  </script>
  ```

#### 3.3 计数器案例

使用render编写计数器：

```vue
//版本一：render搭配options API
<script>
	import { h } from 'vue';
    
    export default {
        data() {
            return {
                counter: 0
            }
        },
        render() {
            return h(
            'div',
            {class: "app"},
            [
                h("h2", null, `当前计数:${this.counter}`), //在render中可以使用this访问实例
                h("button", {
                    onClick: () => this.counter++;
                }, "+1"),
                h("button", {
                    onClick: () => this.counter--;
                }, "-1")
            ])
        }
    }
</script>
```

```vue
//版本二：render搭配setup
<script>
	import { ref, h } from 'vue';
    
    export default {
        setup() {
            const counter = ref(0);
            return {
                count
            }
        },
        render() {
            return h(
            'div',
            {class: "app"},
            [
                h("h2", null, `当前计数:${this.counter}`), //在render中同样可以使用this访问setup返回的数据
                h("button", {
                    onClick: () => this.counter++;
                }, "+1"),
                h("button", {
                    onClick: () => this.counter--;
                }, "-1")
            ])
        }
    }
</script>
```

```vue
//版本三： setup代替render
<script>
	import { ref, h } from 'vue';
    
    export default {
        setup() {
            const counter = ref(0);
            return () => {
                return h(
            	'div',
            	{class: "app"},
            	[
                	h("h2", null, `当前计数:${counter.value}`),//此时访问counter不需要this指针，但在setup中需要手动解包ref对象
                	h("button", {
                    	onClick: () => counter.value++;
                	}, "+1"),
                	h("button", {
                    	onClick: () => counter.value--;
                	}, "-1")
            	])
            }
        }
    }
</script>
```

#### 3.4 h函数渲染组件及插槽

在h函数中还可以渲染组件：

```vue
<script>
	import { h } from 'vue';
    import HelloWorld from './HelloWorld.vue'
    
    export default {
        render() {
            return h(HelloWorld, {}, "") //不需要注册组件
        }
    }
</script>
```

h函数中传入插槽：

```vue
//App.vue
<script>
	import { h } from 'vue';
    import HelloWorld from './HelloWorld.vue'
    
    export default {
        render() {
            return h("div", {}, {
                //传入一个返回VNode插槽元素的函数到default插槽中
                default: props => h("span", null, `app传入到HelloWorld中的内容:${props.info}`) //通过箭头函数参数访问子组件中的数据
            })
        }
    }
</script>
```

```vue
//HelloWorld.vue
<script>
	import { h } from 'vue';
    
    export default {
        render() {
            return h(
            'div',
            {class: "hello-world"},
            [
                h("h2", null, "Hello World"),
                this.$slots.default ? this.$slots.default({info: "hahaha"})
                					: h("span", null, "我是默认值")
            ])
        }
    }
</script>
```

### 四、jsx使用

#### 4.1 jsx的babel配置

如果我们希望在项目中使用jsx，那么我们需要添加对jsx的支持：

- jsx我们通常会通过Babel来进行转换（React编写的jsx就是通过babel转换的）
- 对于Vue来说，我们只需要在Babel中配置对应的插件即可

安装Babel支持Vue的jsx插件：

- `npm install @vue/babel-plugin-jsx -D`

在babel.config.js配置文件中配置插件：

```js
module.exports = {
    presets: [
        '@vue/cli-plugin-babel/preset'
    ],
    plugins: [
        "@vue/babel-plugin-jsx"
    ]
}
```

#### 4.2 jsx简单使用

jsx实现计数器案例：

```vue
<script>
	import { ref } from 'vue';
    
    export default = {
        setup() {
        	const counter = ref(0);
        	const increment = () => counter.value++;
        	const decrement = () => counter.value--;
        
        	return {
            	counter,
            	increment,
            	decrement
        	}
    	},
        render() {
            return (
            <div>
            	<h2>当前计数: {this.counter} //jsx引用变量是单括号
        		<button onClick={this.increment}>+1</button>
    			<button onClick={this.decrement}>+1</button>
    		</div>
    		)
        }
    }
</script>
```

jsx使用组件插槽：

```vue
//App.vue
<script>
    import HelloWorld from './HelloWorld.vue';
    
    export default {
        render() {
            return (
            	<div>
                	<HelloWolrd>
                		//在单括号中传入对象，属性为插槽名，value为函数
                		{{default: props => <button>{props.name}</button>}}
                	</HelloWorld>
                </div>
            )
        }
    }
</script>
```

```vue
//HelloWorld.vue
<script>
	export default {
        render() {
            return (
            	<div>
                	<h2>Hello World</h2>
                	{this.$slots.default ?
                	  this.$slots.default({name: "coderwhy"})
    				  : <span>我是默认值</span>}
                </div>
            )
        }
    }
</script>
```

