[TOC]

### 一、组件的通信

- 在开发过程中，我们会经常遇到需要组件之间相互进行通信
  - 比如App可能使用了多个Header，每个地方的Header展示的内容不同，那么我们就需要使用者传递给Header一些数据，让其进行展示
  - 又比如我们在Main中一次性请求了Banner数据和ProductList数据，那么就需要传递给它们来进行展示
  - 也可能是子组件中发生了事件，需要由父组件来完成某些操作，那就需要子组件向父组件传递事件
- 父子组件之间如何进行通信呢
  - 父组件传递给子组件：通过props属性
  - 子组件传递给父组件：通过$emit触发事件

### 二、父组件传递给子组件

- 在开发中很常见的就是父子组件之间通信，比如父组件有一些数据，需要子组件来进行展示。这个时候我们可以通过props来完成组件之间的通信
- 什么是Props呢
  - Props是你可以在组件上注册一些自定义的attribute
  - 父组件给这些attribute赋值，子组件通过attribute的名称获取到对应的值
- Props有两种常见的用法
  - 方式一：字符串数组，数组中的字符串就是attribute的名称
  - 对象类型，对象类型我们可以在指定attribute名称的同时，指定它需要传递的类型、是否是必须的、默认值等等

#### 2.1 Props的数组用法

- 首先在子组件中定义attributes

  ```vue
  <script>
  	export default {
          props: ["title", "content"]
      }
  </script>
  ```

- 在父组件中传递该值

  ```vue
  <template>
  	<div>
          <show-message title="hahaha" content="呵呵呵"></show-message>   //在子组件中传递值
          <show-message title="呵呵呵" content="hahaha"></show-message>   
      </div>
  </template>
  ```

#### 2.2 Props的对象用法

- 数组用法中我们只能说明传入的attribute的名称，并不能对其进行任何形式的限制

- 当使用对象语法的时候，我们可以对传入的内容限制更多

  - 比如指定传入的attribute的类型

  - 比如指定传入的attribute是否是必传的

  - 比如指定没有传入时，attribute的默认值

    ```
    export default {
    	props: {
    		//指定类型
    		title: String,
    		//指定类型，是否必选、默认值
    		content: {
    			type: String,
    			require: true,
    			default: "hahaha"
    		}
    	}
    }
    ```

    

- type的类型

  - String
  - Number
  - Boolean
  - Array
  - Object
  - Date
  - Function
  - Symbol
  - 自定义类型

- 对象类型的其他写法

  ```vue
  props: {
  	messageInfo: String,
  	//多个可能的类型
  	propB: [String, Number],
  	//该属性必填
  	propC: {
  		type: String,
  		require: true
  	},
  	//有默认值的数字
  	propD: {
  		type: Number,
  		default: 100
  	},
  	//带有默认值的对象
  	propF: {
  		type: Object,
  		default() {
  			return {message: "hello"} //类型为对象则默认值必须为工厂函数，不然多个组件                                       //对同一个对象引用可能会引起错误
  		}
  	},
  	//自定义验证函数
  	propG: {
  		validator(value) {
  			return["success", "warning", "danger"].includes(value)
  		}
  	},
      //与对象或数组不同，函数默认值不是一个工厂函数
  	propH: {
  		type: Function,
  		default(){
  			return "default function"
  		}
  	}
  }
  ```

- Prop的大小写命名

  - HTML 中的attribute 名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符
  - 这意味着当你使用DOM 中的模板时，camelCase (驼峰命名法) 的prop 名需要使用其等价的kebab-case (短横线分隔命名) 命名
  - 但在vue文件中不用，因为vue文件是vue-loader进行处理而非浏览器

- 非Prop的Attribute

  - 当我们传递给一个组件某个属性，但是该属性并没有定义对应的props或者emits时，就称之为非Prop的Attribute，常见的包括class、style、id属性等

  - Attribute继承

    - 当组件有单个根节点时，非Prop的Attribute将自动添加到根节点的Attribute中

  - 如果我们不希望组件的根元素继承attribute，可以在组件中设置inheritAttrs: false

    - 禁用attribute继承的常见情况是需要将attribute应用于根元素之外的其他元素

    - 我们可以通过$attrs来访问所有的非props的attribute
    
  - 多个根节点的attribute
  
    - 多个根节点的attribute如果没有显示的绑定，那么会报警告，我们必须手动的指定要绑定到哪一个属性上
    
### 三、子组件传递给父组件

- 什么情况下子组件需要传递内容到父组件呢

  - 当子组件有一些事件发生的时候，比如在组件中发生了点击，父组件需要切换内容
  - 子组件有一些内容想要传递给父组件的时候

- 我们如何完成上面的操作呢

  - 首先，我们需要在子组件中定义好在某些情况下触发的事件名称

  - 其次，在父组件中以v-on的方式传入要监听的事件名称，并且绑定到对应的方法中

  - 最后，在子组件中发生某个事件的时候，根据事件名称触发对应的事件

    ```vue
    //子组件中
    <template>
    	<div>
            <button @click="increment">+1</button>
            <button @click="decrement">+1</button>
        </div>
    </template>
    
    <script>
    	export default {
            emits: ["addOne", "subOne"],
            methods: {
                increment() {
                    this.$emit("addOne");
                },
                decrement() [
                    this.$emit("subOne");
                ]
            }
        }
    </script>
    ```

    ```vue
    //父组件中
    <template>
    	<div>{{conut}}</div>
    	<conuter-operator @addOne="add" @subOne="sub"></conuter-operator>
    </template>
    
    <script>
        import CounterOperator from "./CounterOperator"
    	export default {
            components: {
                CounterOperator
            },
            data() {
                return {
                    count: 0
                }
            },
            methods: {
                add() {
                    this.count++;
                },
                sub() {
                    this.count--
                }
            }
        }
    </script>
    ```

- 自定义事件的参数和验证

  - 自定义事件的时候，我们也可以传递一些参数给父组件

    ```
    increment() {
    	this.$emit("addTen", 10) //该参数可在父组件中事件的回调函数中取得
    }
    ```

  - 在vue3当中，我们可以对传递的参数进行验证

    ```vue
    emits: {
    	addOne: null,
    	sunOne: null,
    	addTen: function(payload) {
    		if(payload === 10) {
    			return true;
    		}
    		return false;
    	}
    }
    ```

    


​      