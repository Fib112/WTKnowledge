[TOC]

### vue3的v-model

#### 基本使用

1. 表单提交是开发中非常常见的功能，也是和用户交互的重要手段

   - 比如用户在登录、注册时需要提交账号密码

   - 在检索、创建、更新信息时，需要提交一些数据

2. 这些都要求我们可以在逻辑代码中获取到用户提交的数据，我们通常会用v-model指令来完成

   - v-model指令可以在表单input、textarea以及select元素上创建双向数据绑定

   - 它会根据控件类型自动选择正确的方法来更新元素

   - v-model本质上不过是语法糖，负责监听用户的输入事件来更新数据，并在某种极端场景下进行一些特殊处理

     ```html
     <input type="text" v-model="message">
     <h2>{{message}}</h2>
     //等价于
     <input type="text" :value="message" @input="changemsg">
     <h2>{{message}}</h2>
     ...
     methods: {
     	changemsg(event) {
     		this.message = event.target.value;
     	}
     }
     ```

#### v-model绑定checkbox

- 单个勾选框

  - v-model为布尔值

  - input的value并不影响v-model的值

    ```html
    <div>
        <label for="agreement">
            <input id="agreement" type="checkbox" v-model="isAgree">同意协议
        </label>
        <h2>
            isAgree当前的值是：{{isAgree}}
        </h2>
    </div>
    ```

- 多个复选框

  - 当是多个复选框时，因为可以选中多个，所以对应的data属性是一个数组

  - 当选中某一个时，就会将input的value添加到数组中

    ```html
    <div>
        <label for="basketball">
            <input id="basketball" type="checkbox" value="basketball" v-model="hobbies">篮球
        </label>
        <label for="football">
            <input id="football" type="checkbox" value="football" v-model="hobbies">足球
        </label>
        <label for="tennis">
            <input id="tennis" type="checkbox" value="tennis" v-model="hobbies">网球
        </label>
    </div>
    ...
    data() {
    	return {
    		hobbies: []
    	}
    }
    ```

#### v-model绑定radio

v-model绑定radio，用于选择其中一项

```html
<div>
    <label for="male">
        <input type="radio" id="male" v-model="gender" value="male">男
    </label>
    <label for="female">
        <input type="radio" id="female" v-model="gender" value="female">女
    </label>
    <h2>
        gender当前的值为：{{gender}}
    </h2>
</div>
...
data() {
	return {
		gender: ''
	}
}
```

#### v-model绑定select

和checkbox一样，select也分单选和多选两种情况

- 单选：只能选中一个值

  - v-model绑定的是一个值

  - 当我们选中option中的一个时，，会将它对应的value复制到绑定的data属性中

    ```html
    <div>
        <select v-model="fruit">
            <option value="apple">苹果</option>
            <option value="orange">橘子</option>
            <option value="banana">香蕉</option>
        </select>
        <h2>
            fruit当前的值是：{{fruit}}
        </h2>
    </div>
    ```

- 多选：可以选中多个值

  - v-model绑定的是一个数组；

  - 当选中多个值时，就会将选中的option对应的value添加到数组fruit中

    ```html
    <div>
        <select v-model="fruit" multiple>
            <option value="apple">苹果</option>
            <option value="orange">橘子</option>
            <option value="banana">香蕉</option>
        </select>
        <h2>
            fruit当前的值是：{{fruit}}
        </h2>
    </div>
    ```


#### v-model的值绑定

之前的案例中大部分的值都是在template中固定好的，比如gender中的两个输入值male、female；hobbies的三个输入框值basketball、football、tennis。

但在真实开发中，数据可能来自服务器，那么我们可以先将值请求下来，绑定到data返回的对象中，再通过v-bind来进行值的绑定

#### v-model修饰符

- lazy

  默认情况下，v-model在进行双向绑定时，绑定的是input事件，那么会在每次内容输入后就将最新的值和绑定的属性进行同步；

  如果我们在v-model后跟上lazy修饰符，那么会将绑定的事件切换为change事件，只有在提交时（比如回车）才会触发

  ```html
  <template id="my-app">
  	<input type="text" v-model.lazy="message">
      <h2>
          {{message}}
      </h2>
  </template>
  ```

- number

  v-model返回的值总是string类型，即使绑定的data属性是其他类型，v-model的返回值也会变为string类型（单个复选框和radio返回的Boolean除外）

  如果我们希望转换为数字类型，那么可以使用.number修饰符

  ```html
  <input type="text" v-model.number="score">
  ```

- trim

  如果要过滤用户输入的首尾空白字符，可以给v-model添加trim修饰符

  ```html
  <input type="text" v-model.trim="message">
  ```

### 组件化开发

- 在开发中如果将一个页面中所有的处理逻辑全部放在一起，处理起来就会变得非常复杂，不利于后续管理与维护

- 如果将一个页面拆分成一个个小的功能块，每个功能块完成属于自己这部分独立的功能，那么页面的管理和维护就变得容易了

#### 注册组件的方式

注册组件分为两种：全局组件和局部组件。其中全局组件在任何其他的组件中都可以使用，而局部组件只有在注册的组件中才能使用

#### 全局组件

- 注册

  全局组件需要使用我们全局创建的app来注册组件。

  通过component方法传入组件名称、组件对象即可注册一个全局组件了

  之后，我们可以在App组件的template中直接使用这个全局组件

  ```javascript
  const app = Vue.createApp({
      ...
  });
  app.component("my-cpn", {     //第一个参数为组件名称，第二个为组件对象
      ...
  });
  ```

- 逻辑

  组件本身可以有自己的代码逻辑，比如自己的data、computed、methods等

  ```javascript
  app.component("my-cpn", {
      template: "#my-cpn",     //template可以像app一样提出来写在html中再引入
      data() {
          return {
              title: ...,
              message: ...
          }
      },
      methods: {
          btnClick() {
              console.log("btnClick");
          }
      }
  })
  ```

- 名称

  定义组件名的方式有两种：

  - 使用kebab-case（短横线分隔符）

    当使用短横线分割符定义一个组件时，也必须在引用这个自定义元素时使用短横线分隔符

    ```javascript
    app.component('my-component-name', {
        ...
    });
    //使用时
    <my-component-name></my-component-name>
    ```

  - 使用PascalCase

    当使用PascalCase定义一个组件时，在引用这个自定义元素时两种命名法都可以用。也就是说`<my-component-name>`和`<MyComponentName>`都可以用

#### 局部组件

- 全局组件往往是在应用程序一开始就会全局组件完成，那么意味着如果某些组件我们并没有用到，也会被一起注册，这样用户在下载对应的JavaScript包也会增加包的大小

- 所以，在开发中我们通常使用组件的时候采用的都是局部注册；

  - 局部注册是在我们需要使用的组件中，通过components属性选项来进行注册
  - 比如之前的App组件中，我们有data、computed、methods等选项，事实上，还可以有一个components选项
  - 该components选项对应的是一个对象，对象中的键值对是**组件的名称：组件对象**；

  ```javascript
  const ComponentA = {
      template: "#component-a",
      data() {
          return {
              title: "我是ComponentA标题",
              message: "我是ComponentA内容"
          }
      }
  };
  const ComponentB = {
      template: "#component-b",
      data() {
          return {
              title: "我是ComponentB标题",
              message: "我是ComponentB内容"
          }
      }
  }
  ...
  const App = {
      template: "#my-app",
      component: {
          'component-a': ComponentA,
          'component-b': ComponentB,
      },
      data() {
          return {
              message: "Hello World"
          }
      }
  }
  ```

#### vue的开发模式

- 目前使用vue的过程都是在html文件中，通过template编写自己的模板、脚本逻辑、样式等

- 但随着项目越来越复杂，我们会采用组件化的方式来进行开发：

  - 这就意味着每个组件都会有自己的模板、脚本逻辑、样式等
  - 我们依然可以把它们抽离到单独的js、css文件中，但它们还是会分离开来
  - 也包括我们的script是在一个全局的作用域下，很容易出现命名冲突的问题
  - 并且我们的代码为了适配一些浏览器，必须使用ES5的语法
  - 在我们编写代码完成之后，依然需要通过工具对代码进行构建、代码

- 所以在真实开发中，我们可以通过一个后缀名为.vue的single-file components（单文件组件）来解决，并且可以通过webpack或者vite或者rollup等构建工具来对其进行处理

- 单文件特点

  在这个组件中我们可以获得非常多特性

  - 代码的高亮
  - ES6、CommonJS的模块化能力
  - 组件作用域的CSS
  - 可以使用预处理器来构建更加丰富的组件，如Typescript、Babel、Less、Sass等

#### 如何支持SFC

如果要使用这一SFC的.vue文件，比较常见的是两种方式：

1. 使用Vue CLI来创建项目，项目会默认帮助我们配置好所有配置选项，可以在其中直接使用.vue文件
2. 自己使用webpack或vite、rollup这类打包工具，对其进行打包处理