### 一、Mixin与extends

#### 1.1 mixin简介

目前我们是使用组件化的方式在开发整个Vue的应用程序，但是组件和组件之间有时候会存在相同的代码逻辑，我们希望对相同的代码逻辑进行抽取

在Vue2和Vue3中都支持的一种方式就是使用Mixin来完成：

- Mixin提供了一种非常灵活的方式，来分发Vue组件中的可复用功能
- 一个Mixin对象可以包含任何组件选项
- 当组件使用Mixin对象时，所有Mixin对象的选项将被混合进入该组件本身的选项中

#### 1.2 mixin使用

先建立mixin的js模块对象文件，然后在需要使用的组件中import，然后在mixin属性中注册，以数组的形式

#### 1.3 Mixin的合并规则

如果Mixin对象中的选项和组件对象中的选项发生了冲突，分成不同的情况来进行处理：

1. 如果是data函数的返回值对象
   - 返回值对象默认情况下会进行合并
   - 如果data返回值对象的属性发生了冲突，那么会保留组件自身的数据
2. 如果是生命周期钩子函数
   - 生命周期的钩子函数会被合并到数组中，都会被调用
3. 值为对象的选项，例如methods、components 和directives，将被合并为同一个对象
   - 比如都有methods选项，并且都定义了方法，那么它们都会生效
   - 但是如果对象的key相同，那么会取组件对象的键值对

#### 1.4 全局混入Mixin

如果组件中的某些选项，是所有的组件都需要拥有的，那么这个时候我们可以使用全局的mixin：

- 全局的Mixin可以使用应用app的方法mixin 来完成注册

- 一旦注册，那么全局混入的选项将会影响每一个组件

  ```js
  //在main.js中
  const app = creatApp(App)
  app.mixin({
      //全局mixin对象
  })
  app.mount("#app");
  ```

#### 1.5 extends

另外一个类似于Mixin的方式是通过extends属性：

- 允许声明扩展另外一个组件，类似于Mixins

- 先编写一个vue组件，再在另一个组件import之后在extends中注册

  ```js
  import BasePage from './BasePage.vue'
  
  export default {
      extends: BasePage //只能使用BasePage中export default中的东西
  }
  ```


### 二、compositionAPI

#### 2.1 OptionsAPI的弊端

在Vue2中，我们编写组件的方式是Options API：

- Options API的一大特点就是在对应的属性中编写对应的功能模块
- 比如data定义数据、methods中定义方法、computed中定义计算属性、watch中监听属性改变，也包括生命周期钩子

但是这种代码有一个很大的弊端：

- 当我们实现某一个功能时，这个功能对应的代码逻辑会被拆分到各个属性中
- 当我们组件变得更大、更复杂时，逻辑关注点的列表就会增长，那么同一个功能的逻辑就会被拆分的很分散
- 尤其对于那些一开始没有编写这些组件的人来说，这个组件的代码是难以阅读和理解的（阅读组件的其他人）

#### 2.2 认识CompositionAPI

为了开始使用Composition API，我们需要有一个可以实际使用它（编写代码）的地方,在Vue组件中，这个位置就是setup函数

setup其实就是组件的另外一个选项：

- 只不过这个选项强大到我们可以用它来替代之前所编写的大部分其他选项,比如methods、computed、watch、data、生命周期等等

#### 2.3 setup函数的参数

setup函数的参数，主要有两个：

- 第一个参数：props
- 第二个参数：context

props非常好理解，它其实就是父组件传递过来的属性会被放到props对象中，我们在setup中如果需要使用，那么就可以直接通过props参数获取：

- 对于定义props的类型，我们还是和之前的规则是一样的，在props选项中定义
- 并且在template中依然是可以正常去使用props中的属性，比如message
- 如果我们在setup函数中想要使用props，那么不可以通过this 去获取
- 因为props有直接作为参数传递到setup函数中，所以我们可以直接通过参数来使用即可

另外一个参数是context，我们也称之为是一个SetupContext，它里面包含三个属性：

- attrs：所有的非prop的attribute
- slots：父组件传递过来的插槽（这个在以渲染函数返回时会有作用，后面会讲到）
- emit：当我们组件内部需要发出事件时会用到emit（因为我们不能访问this，所以不可以通过this.$emit发出事件）

#### 2.4 setup函数的返回值

setup既然是一个函数，那么它也可以有返回值：

- setup的返回值可以在模板template中被使用
- 也就是说我们可以通过setup的返回值来替代data选项

甚至是我们可以返回一个执行函数来代替在methods中定义的方法：

```js
//setup函数中
let counter = 100;
const increment = () => {
    counter++;
}
return {
    counter,
    increment
}
```

但是，如果我们将counter 在increment 或者 decrement进行操作时,不可以实现界面的响应式,这是因为对于一个定义的变量来说，默认情况下，Vue并不会跟踪它的变化，来引起界面的响应式操作

#### 2.5 setup不可以使用this

setup在调用的时候并没有绑定组件实例，所以使用this并不能访问组件实例

#### 2.6 Reactive API

如果想为在setup中定义的数据提供响应式的特性，那么我们可以使用reactive的函数：

```js
import {reactive} from 'vue' //先导入reactive函数
const state = reactive({
    name: "coderwhy",
    counter: 100
})
const increment = () => {
    state.counter++
}
return {
    state,
    increment
}

//在使用时，需要从state对象中取值
{{state.counter}}
```

这是什么原因呢？为什么数据可以变成响应式?

- 这是因为当我们使用reactive函数处理我们的数据之后，数据再次被使用时就会进行依赖收集
- 当数据发生改变时，所有收集到的依赖都是进行对应的响应式操作（比如更新界面）
- 事实上，我们编写的data选项，也是在内部交给了reactive函数将其变成响应式对象的

#### 2.7 ref API

reactive API对传入的类型是有限制的，它要求我们必须传入的是一个对象或者数组类型，如果我们传入一个基本数据类型（String、Number、Boolean）会报一个警告

这个时候Vue3给我们提供了另外一个API：ref API

- ref 会返回一个可变的响应式对象，该对象作为一个响应式的引用维护着它内部的值，这就是ref名称的来源

- 它内部的值是在ref的 value 属性中被维护的

  ```js
  import {ref} from 'vue' //导入ref API
  const message = ref("Hello")
  ```

两个注意事项：

- 在模板中引入ref的值时，Vue会自动帮助我们进行解包操作，所以我们并不需要在模板中通过 ref.value 的方式来使用
- 但是在setup 函数内部，它依然是一个ref引用，所以对其进行操作时，我们依然需要使用ref.value的方式

#### 2.8 Ref自动解包

模板中的解包是浅层的解包：

- 当给ref对象再包裹一层普通对象时，在template中不能进行正确解包

  ```js
  const msg = ref("hello,world");
  const info = {
      msg
  }
  return {
      info
  }
  //在template中
  {{info.msg}}//这样使用并不能解包
  {{info.msg.value}}//正确用法
  ```

- 当给ref对象包裹一层reactive对象时，在template中可以正确解包

  ```js
  const msg = ref("hello,world");
  const info = reactive({
      msg
  })
  return {
      info
  }
  //在template中
  {{info.msg}}//可以正确解包
  ```

#### 2.9 readonly

我们通过reactive或者ref可以获取到一个响应式的对象，但是某些情况下，我们传入给其他地方（组件）的这个响应式对象希望在另外一个地方（组件）被使用，但是不能被修改，Vue3为我们提供了readonly的方法：

- readonly会返回原生对象的只读代理（也就是它依然是一个Proxy，这是一个proxy的set方法被劫持，并且不能对其进行修改）
- 在开发中常见的readonly方法会传入三个类型的参数：
  - 类型一：普通对象
  - 类型二：reactive返回的对象
  - 类型三：ref的对象

在readonly的使用过程中，有如下规则：

- readonly返回的对象都是不允许修改的
- 但是经过readonly处理的原来的对象是允许被修改的：
  - 比如const info = readonly(obj)，info对象是不允许被修改的
  - 当obj被修改时，readonly返回的info对象也会被修改
  - 但是我们不能去修改readonly返回的对象info
- 其实本质上就是readonly返回的对象的setter方法被劫持了而已

####  2.10 注册子组件

在setup中注册子组件需要引入另一个API：defineComponent

```vue
//在father.vue中注册使用child.vue
<template>
	<h2>我是父组件</h2>
	<child></child>
</template>

<script>
	import { defineComponent } from 'vue';
	import child from './child.vue'
    
    export default defineComponent({
        components: {
            child
        },
        setup() {
            ...
        }
    })
</script>
```

