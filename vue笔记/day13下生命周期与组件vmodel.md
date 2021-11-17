### 一、$refs、$parent和$root

- $refs的使用

  - 某些情况下，我们在组件中想要直接获取到元素对象或者子组件实例：

    - 在Vue开发中我们是不推荐进行DOM操作的

    - 这个时候，我们可以给元素或者组件绑定一个ref的attribute属性

      ```html
      <h2 :ref="title">
          哈哈哈
      </h2>
      ```

      

  - 组件实例有一个$refs属性：

    - 它是一个对象Object，持有注册过ref attribute 的所有DOM 元素和组件实例

      ```vue
      visitElement() {
      	console.log(this.$refs.title)
      }
      ```

- $parent和$root

  - 我们可以通过$parent来访问父元素，通过$root访问根元素

### 二、生命周期与组件v-model

#### 2.1 认识生命周期

- 什么是生命周期呢

  - 每个组件都可能会经历从创建、挂载、更新、卸载等一系列的过程
  - 在这个过程中的某一个阶段，用于可能会想要添加一些属于自己的代码逻辑（比如组件创建完后就请求一些服务器数据）
  - 但是我们如何可以知道目前组件正在哪一个过程呢？Vue给我们提供了组件的生命周期函数

- 生命周期函数

  - 生命周期函数是一些钩子函数，在某个时间会被Vue源码内部进行回调

  - 通过对生命周期函数的回调，我们可以知道目前组件正在经历什么阶段

  - 那么我们就可以在该生命周期中编写属于自己的逻辑代码了

  - 生命周期的流程：

    beforeCreate->created->beforeMount->mounted->(beforeUpdate->updated)->beforeUnmount->unmounted(update是指template中发生更新)
  
- keep-alive与生命周期

  keep-alive内的组件在切换时并没有卸载，而是缓存了起来，所以切换时没有beforeUnmount与unmounted事件，但vue提供了另一组回调函数：deactivated和activated

#### 2.2 组件的v-model

- 在input中可以使用v-model来完成双向绑定：

  - 这个时候往往会非常方便，因为v-model默认帮助我们完成了两件事：v-bind:value的数据绑定和@input的事件监听

- 如果我们现在封装了一个组件，其他地方在使用这个组件时，是否也可以使用v-model来同时完成这两个功能呢

  - 也是可以的，vue也支持在组件上使用v-model

  - 我们在组件上使用v-model的时候，等价于如下的操作：

    ```vue
    <my-input v-model="message"></my-input>
    //相当于
    <my-input :model-value="message" @undate:model-value="message = $event"></my-input>
    ```

- 组件v-model的实现

  - 那么，为了我们的MyInput组件可以正常的工作，这个组件内的` <input> `必须：

    - 将其value attribute 绑定到一个名叫modelValue的prop 上

    - 在其input 事件被触发时，将新的值通过自定义的update:modelValue 事件抛出

      ```vue
      //myInput.vue
      <template>
      	<div>
              <input :value="modelValue" @input="inputChange">
          </div>
      </template>
      
      <script>
      	export default {
              props: ["modelValue"],
              emits: ["update:modelValue"],
              methods: {
                  inputChange(event) {
                      this.$emit("update:modelValue", event.target.value);
                  }
              }
          }
      </script>
      ```

- computed实现

  - 既然在组件内使用了input，不免会想到使用v-model，但v-model只能修改组件内数据，无法与父组件通信，所以行不通

  - 我们依然希望在组件内部按照双向绑定的做法去完成，应该如何操作呢？我们可以使用计算属性的setter和getter来完成

    ```vue
    <template>
    	<div>
            <input v-model="value">
        </div>
    </template>
    
    <script>
    	export default {
            props: ["modelValue"],
            emits: ["update:modelValue"],
            computed: {
                value: {
                    get() {
                        return this.modelValue;
                    },
                    set(value) {
                        this.emit("update:modelValue", value);
                    }
                }
            }
        }
    </script>
    ```

- 绑定多个属性

  - 我们现在通过v-model是直接绑定了一个属性，如果我们希望在一个组件上使用多个v-model是否可以实现呢

    - 我们知道，默认情况下的v-model其实是绑定了modelValue 属性和@update:modelValue的事件

    - 如果我们希望绑定更多，可以给v-model传入一个参数，那么这个参数的名称就是我们绑定属性的名称

    - 注意：这里我是绑定了两个属性的

      ```vue
      <my-input v-model="message" v-model:title="title"></my-input>
      ```

  - v-model:title相当于做了两件事：

    - 绑定了title属性

    - 监听了@update:title的事件

      ```vue
      export default {
      	props: ["modelValue", "title"],
      	emits: ["update:modelValue", "update:title"],
      	methods: {
      		input1Change() {
      			this.$emit("update:modelValue", event.target.value);
      		}
      		input2Change() {
      			this.$emit("update:title", event.target.value);
      		}
      	}
      }
      ```

      

