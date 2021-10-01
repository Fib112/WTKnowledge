- ## 计算属性

  - #### 什么是计算属性

    计算属性被混入组件实例中，所有getter和setter的this上下文自动地绑定为组件实例

  - #### 计算属性的用法

    - 选项：computed

      ```javascript
      Vue.createApp({
      	template:...,
      	data() {
      		return {
      		...
      		}
      	},
      	computed: {
      	...
      	}
      })
      ```

    - 类型

      {[keystring]: Function | {get: Function, set: Function}}

  - #### 计算属性案例

    ```html
    <template id="my-app">
    	<h2>{{ fullName }}</h2>
        <h2>{{ result }}</h2>
        <h2>{{ reverseMessage }}</h2>
    </template>
    ...
    computed: {
    	fullName() {
    		return this.firstName + "" + this.lastName;
    	},
    	result() {
    		return this.score > 60 ? "及格" : "不及格";
    	},
    	reverseMessage() {
    		return this.message.split(" ").reverse().join(" ");
    	}
    },
    ```

  - #### 计算属性与methods的区别

    - 计算属性是有缓存的，当调用一次后后缓存结果，再次调用时直接返回结果。

      ```html
      //使用methods,console.log只调用一次
      <h2>{{getResult()}}</h2> //methods
      <h2>{{getResult()}}</h2> //methods
      <h2>{{getResult()}}</h2> //methods
      
      //使用computed,console.log只调用一次
      <h2>{{result}}</h2> //computed
      <h2>{{result}}</h2>
      <h2>{{result}}</h2>
      ...
      methods: {
      	getResult() {
      		consolr.log("methods");
      		return score > 60 ? "合格" : "不合格";
      	}
      },
      computed: {
      	result() {
      		console.log("computed");
      		return score > 60 ? "合格" : "不合格";
      	}
      }
      ```

    - 计算属性的缓存
    
      计算属性会基于它们的依赖关系进行缓存，在数据不发生变化时，计算属性不需要重新计算；但当依赖的数据发生变化时，在使用时计算属性依然会进行重新计算
    
  - #### 计算属性的getter和setter
  
    - 计算属性在大多数情况下只需要getter，所以直接写一个函数即可。如果确实需要setter时，可以设置一个对象
  
      ```javascript
      computed: {
          fullName: {
              get() {
                  ...
              },
              set() {
                  ...
              }
          }
      }
      ```
  
      在vue源码中，会对传入的fullName进行判断，如果传入的是函数则设置为getter，如果传入的是对象，则会取对象的get和set方法设置为getter和setter
  
- ## 侦听器watch

  - #### 基本用法

    用来侦听data中数据的变化

    选项：watch

    类型： {[key: string] | Function | Object | Array}

    ```javascript
    watch: {
    	info(newValue, oldValue){ //data中的数据
    		console.log(newValue, oldValue);
    	} 
    }
    ```

  - #### 配置选项

    - 当侦听的数据是对象等引用类型时，修改如info.name，watch是不会侦听到的。因为watch只是在侦听info的引用变化，对于内部属性的变化是不会做出相应的。此时，可以使用一个选项**deep**进行更深层的侦听

    - 还有另外一个属性，是希望一开始就会执行一次。这个时候使用**immediate**选项，这时无论后面数据是否变化，侦听函数都会有限执行一次

      ```vue
      watch: {
      	info: {
      		handler(newValue, oldValue) {
      			console.log(newValue, oldValue);
      		},
      		deep: true,
      		immediate:true
      	}
      }
      ```

    - **在进行深度监听时，进行的是浅拷贝，当一个属性为引用类型且该属性的属性变化时，由于是进行浅拷贝，所以oldValue是对newValue的同一个属性进行引用，所以oldValue中没有保存变化的属性的属性的旧值**

  - #### 其他侦听方式(一)

    ```vue
    //字符串方法名
    info: "someMethods" //someMethods为定义在methods中的方法
    //可以传入回调数组，它们会被逐一侦听
    info: [
    	"handle1",
    	function handle2(newValue, oldValue) {
    		...
    	},
    	{
    		handler: function handle3(val, oldVal) {
    			...
    		}
    	}
    ]
    ```

  - #### 其他侦听方式(二)

    - 侦听对象属性

      ```vue
      //深度监听是对所有属性变化起作用，而该监听只对指定属性起作用
      'info.name': function(newValue, oldValue) {
      	...
      }
      ```

    - 使用$watch的api：

      我们可以在created的生命周期中，使用this.$watch来侦听

      - 第一个参数是要侦听的源

      - 第二个参数是侦听的回调函数callback

      - 第三个参数是额外的其他选项，比如deep、immediate

        ```vue
        created() {
        	this.$watch('message', (newValue, oldValue) => {
        		...
        	},
        	{
        		deep: true,
        		immediate: true
        	})
        }
        ```

        