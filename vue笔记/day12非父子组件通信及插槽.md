[TOC]

### 一、非父子组件通信

#### 1.1 provide/inject

- Provide/Inject用于非父子组件之间共享数据：

  - 比如有一些深度嵌套的组件，子组件想要获取父组件的部分内容
  - 在这种情况下，如果我们仍然将props沿着组件链逐级传递下去，就会非常的麻烦

- 对于这种情况下，我们可以使用Provide 和Inject：

  - 无论层级结构有多深，父组件都可以作为其所有子组件的依赖
    提供者
  - 父组件有一个provide 选项来提供数据
  - 子组件有一个inject 选项来开始使用这些数据

- 实际上，可以将依赖注入看作是“long range props”，除了：

  - 父组件不需要知道哪些子组件使用它provide 的property
  - 子组件不需要知道inject 的property 来自哪里

- 基本使用：

  ```vue
  //App.vue
  <template>
  	<div>
          <home></home>
      </div>
  </template>
  
  <script>
  	import Home from './Home.vue'
      
      export default {
          components: {
              Home
          },
          provide: {
              name: "why",
              age: 18
          }
      }
  </script>
  ...
  //Home.vue
  <template>
  	<div>
          <h2>HomeContent</h2>
          <h2>{{name}}-{{age}}</h2>
      </div>
  </template>
  
  <script>
  	export default {
          inject: ["name", "age"]
      }
  </script>
  ```

- 如果Provide中提供的一些数据是来自data，那么我们可能会想要通过this来获取：

  ```
  //App.vue
  provide: {
  	name: this.name
  },
  data() {
  	return {
  		name: "why"
  	}
  }
  ```

  此时代码会报错，因为此时provide中的this指向的是全局对象，但因为script中是模块，所以指向的是undefined

- 如果需要使用data中的数据则需要将provide包装为一个函数

  ```
  provide() {
  	return {
  		name: this.name
  	}
  },
  data() {
  	return {
  		name: "why"
  	}
  }
  ```

- 处理响应式数据
	
  - ```vue
    //App.vue
    data() {
  		return {
  			names: ["abc", "cba"]
  		}
  	},
  	provide() {
  		return {
  			length: this.name.length	
  		}
  	}
  	//Home.vue
  	<template>
  		<h2>
          	{{length}}
      	</h2>
  	</template>
  	
  	inject: ["length"]
  	```
  
  - 在这段代码中，在增加了names数组的内容后，在Home组件中渲染的length仍然不变，这是因为当我们修改了names之后，之前在provide中引入的this.names.length本身并不是响应式的
  
  - 那么怎么样可以让我们的数据变成响应式的呢
  
    - 非常的简单，我们可以使用响应式的一些API来完成这些功能，比如说computed函数
  
    - ```vue
      //App.vue
      provide() {
      	return {
      		length: computed(() => this.name.length)
      	}
      }
      //Home.vue
      <template>
      	{{length.value}}
      </template>
      ```
  
      注意：我们在使用length的时候需要获取其中的**value**，这是因为computed返回的是一个ref对象，需要取出其中的value来使用

#### 1.2 全局事件总线mitt库

- 首先，我们需要先安装这个库：

  ```
  npm install mitt
  ```

- 其次，我们可以封装一个工具eventbus.js：

  ```
  //新建一个eventbus.js文件
  import mitt from 'mitt';
  const emitter = mitt();
  export default emitter;
  ```

- 使用事件总线工具

  - 在项目中可以使用它们， 事件总线可在任意两个组件间传递事件：

    ```vue
    //App.vue
    import emitter from './eventBus';
    export default {
    	components: {
    		Home
    	},
    	methods: {
    		triggerEvent() {
    			emitter.emit("why", {name: "why", age: 18});//触发事件
    		}
    	}
    }
    //Home.vue
    import emitter from './evevntBus';
    export default {
    	//在生命周期内
    	created() {
    		emitter() {
    			emitter.on("why", (info) => { //使用emitter监听事件
    				console.log("why event:", info);
    			});
    			emitter.on("*", (type, info) => { //监听所有事件，回调函数第一个                 console.log("event listener:", type, info)//参数为事件类                                                     //型，第二个为事件参数
    			});
    		}
    	}
    }
    ```
    
  
- mitt的事件取消

  - 在某些情况下我们可能希望取消掉之前注册的函数监听

  - ```
    //取消emitter中所有的监听
    emitter.all.clear()
    
    //取消对foo的监听
    emitter.off("foo", onFoo) //onFoo是监听函数时的handler
    ```

### 二、插槽slot

#### 2.1 认识插槽Slot

- 在开发中，我们会经常封装一个个可复用的组件：
  - 前面我们会通过props传递给组件一些数据，让组件来进行展示
  - 但是为了让这个组件具备更强的通用性，我们不能将组件中的内容限制为固定的div、span等等这些元素
  - 比如某种情况下我们使用组件，希望组件显示的是一个按钮，某种情况下我们使用组件希望显示的是一张图片
  - 我们应该让使用者可以决定某一块区域到底存放什么内容和元素

#### 2.2 使用插槽slot

- 插槽的使用过程其实是抽取共性、预留不同

- 我们会将共同的元素、内容依然在组件内进行封装

- 同时会将不同的元素使用slot作为占位，让外部决定到底显示什么样的元素

- 如何使用slot呢

  - Vue中将`<slot>`元素作为承载分发内容的出口
  - 在封装组件中，使用特殊的元素`<slot>`就可以为封装组件开启一个插槽
  - 该插槽插入什么内容取决于父组件如何使用

- 基本使用

  - 我们一个组件MySlotCpn.vue：该组件中有一个插槽，我们可以在插槽中放入需要显示的内容

    ```vue
    //MySlotCpn.vue
    <template>
    	<div>
            <h2>MySlotCpn开始</h2>
            <slot></slot>
            <h2>MySlotCpn结尾</h2>
        </div>
    </template>
    ```

  - 我们在App.vue中使用它们：我们可以插入普通的内容、html元素、组件元素，都可以是可以的

    ```vue
    //App.vue
    <template>
    	<div>
            <my-slot-cpn>
                //普通的内容
                //Hello World
                //html元素
                //<button>按钮</button>
                //自定义组件
                <my-button></my-button>
        	</my-slot-cpn>
        </div>
    </template>
    ```

#### 2.3 插槽默认内容

- 有时候我们希望在使用插槽时，如果没有插入对应的内容，那么我们需要显示一个默认的内容，当然这个默认的内容只会在没有提供插入的内容时，才会显示

  ```vue
  //MySlotCpn.vue
  <template>
  	<div>
          <h2>MySlotCpn开始</h2>
          <slot>
              <h2>我是默认显示的内容</h2>
      	</slot>
          <h2>MySlotCpn结尾</h2>
      </div>
  </template>
  ```

  ```vue
  //App.vue
  <template>
  	<div>
          <my-slot-cpn></my-slot-cpn> //没有插入内容，默认显示<h2>中的内容
      </div>
  </template>
  ```

#### 2.4 多个插槽的效果

- 如果一个组件中含有多个插槽，我们会发现默认情况下每个插槽都会获取到我们插入的内容来显示

  ```vue
  //navBar.vue
  <template>
  	<div>
          <div>
              <slot></slot>
      	</div>
          <div>
              <slot></slot>
      	</div>
          <div>
              <slot></slot>
      	</div>
      </div>
  </template>
  //App.vue
  <template>
  	<nav-bar>
      	<button></button>  //在3个插槽中都有<button>、<h2>和<i>
          <h2></h2>
          <i></i>
      </nav-bar>
  </template>
  ```

- 使用具名插槽

  - 事实上，我们希望达到的效果是插槽对应的显示，这个时候我们就可以使用具名插槽

    - 具名插槽顾名思义就是给插槽起一个名字，`<slot> `元素有一个特殊的attribute：name

    - 一个不带name 的slot，会带有隐含的名字default

      ```vue
      //NavBar.vue
      <template>
      	<div>
              <div>
                  <slot name="left"></slot>
          	</div>
              <div>
                  <slot name="center"></slot>
          	</div>
              <div>
                  <slot name="right"></slot>
          	</div>
          </div>
      </template>
      
      //App.vue
      <template>
      	<nav-bar>
          	<template v-slot:left>
      			<button>左边按钮</button>
      		</template>
      		<template v-slot:center>
      			<h2>中间标题</h2>
      		</template>
      		<template v-slot:right>
      			<i>右边i元素</i>
      		</template>
          </nav-bar>
      </template>
      ```

  - 具名插槽使用时缩写

    - 跟v-on 和v-bind 一样，v-slot 也有缩写，即把参数之前的所有内容(v-slot:) 替换为字符#

      ```vue
      <template #left> //相当于v-slot:left
      	<button>左边按钮</button>
      </template>
      ```

#### 2.5 动态插槽名

- 目前我们使用的插槽名称都是固定的

- 比如v-slot:left、v-slot:center等等

- 我们可以通过v-slot:[dynamicSlotName]方式动态绑定一个名称

  ```vue
  //App.vue
  <template>
  	<div>
          <nav-bar :name="name">
      		<template v-slot:[name]></template> //使用data中的name属性
      	</nav-bar>
      </div>
  </template>
  ...
  data() {
  	return {
  		name: "why"
  	}
  }
  
  //NavBar.vue
  <slot :name="name"></slot> //通过组件通信动态绑定name属性
  ...
  props: {
  	name: String
  }
  ```

#### 2.6 渲染作用域及作用域插槽

- 渲染作用域

  - 父级模板里的所有内容都是在父级作用域中编译的
  - 子模板里的所有内容都是在子作用域中编译的
  - 所以在父组件传给子组件中的插槽实例是不能访问子组件中的数据的

- 作用域插槽

  - 但是有时候我们希望插槽可以访问到子组件中的内容是非常重要的

    - 但是有时候我们希望插槽可以访问到子组件中的内容是非常重要的
    - Vue给我们提供了作用域插槽

  - 案例

    ```vue
    //ShowNames.vue
    <template>
    	<div>
            <slot :item="item" :index="index"></slot> //绑定数据到属性
        </div>
    </template>
    
    data() {
    	return {
    		item: "why",
    		index: 0
    	}
    }
    
    //App.vue
    <template>
    	<div>
            <show-names>
        		<template v-slot:default="slotProps">//绑定name为default的插                                                  //槽数据到slotProps对象
    				<span>{{slotProps.item}}-{{slotProps.index}}</span>
    			</template>
        	</show-names>
        </div>
    </template>
    ```

- 独占默认插槽的缩写

  - 如果我们的插槽是默认插槽default，那么在使用的时候v-slot:default="slotProps"可以简写为v-slot="slotProps"：

    ```vue
    <show-names>
    	<template v-slot="slotProps">
        	<span>{{slotProps.item}}-{{slotProps.index}}</span>
        </template>
    </show-names>
    ```

  - 并且如果我们的插槽只有默认插槽时，组件的标签可以被当做插槽的模板来使用，这样，我们就可以将v-slot 直接用在组件上：

    ```vue
    <show-names v-slot="slotProps">//直接将数据绑定放在子组件标签上无需使用                                      //template包裹插槽实例
    	<span>{{slotProps.item}}-{{slotProps.index}}</span>
    </show-names>
    ```

  - 但是，如果我们有默认插槽和具名插槽，那么按照完整的template来编写