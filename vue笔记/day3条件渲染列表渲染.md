- ## 条件渲染

  1. #### v-if v-else v-else-if

     渲染原理：其渲染过程是惰性的。当条件为false时，其判断的内容完全不会被渲染或者会被销毁掉；为true时才会真正渲染条件快中的内容

  2. #### template元素

     可以将多个v-if嵌入到template代码块中，然后对template使用条件判断，最终template不会被渲染

  3. #### v-show与v-if的区别

     1. 用法上的区别

        v-show不支持template；

        v-show不能和v-else一起使用

     2. 本质区别

        - v-show无论是否显示，其元素都存在DOM中，在false时将样式设置为display:none;
        - v-if为false时，其对应元素不会被渲染到DOM中

     3. 选择

        - 当元素需要在显示与隐藏来回切换时，选择v-show
        - 当不会频繁地发生切换，使用v-if

- ## 列表渲染

  - #### 基本使用

    1. 遍历数组

       ```html
       //只有一个参数时，为数组值
       <template id="my-app">
       	<ul>
               <li v-for="item in movies">{{item}}</li>
           </ul>
       </template>
       //两个参数时，是值和索引
       <template id="my-app">
       	<ul>
               <li v-for="(item, index) in movies">{{index+1}}.{{item}}</li>
           </ul>
       </template>
       ```

    2. 遍历对象

       ```html
       //一个参数
       v-for="value in object"
       //两个参数
       v-for="(value, key) in object"
       //三个参数, index从零开始
       v-for="(value, key, index) in object"
       ```

    3. 遍历数字

       ```html
       //一个参数
       v-for="num in 10"
       //两个参数，num从1开始，index从0开始
       v-for="(num, index) in 10"
       ```

  - #### 配合template使用

    - 类似v-if，可以使用template来循环渲染一段包含多个元素的内容

      ```html
      <template id="my-app">
      	<ul>
              <template v-for="(value, key) in info">
              	<li>{{key}}</li>
                  <li>{{value}}</li>
              </template>
          </ul>
      </template>
      ```

  - #### 数组更新检测

    ##### vue将被侦听的数组的变更方法进行了包裹，所以他们会触发视图更新，包括：

    - push()
    - pop()
    - shift()
    - unshift()
    - splice()
    - sort()
    - reverse()

    ##### 上面的方法会直接修改原来的数组，但某些方法不会替换原来的数组，而是会生成新的数组，如filter()、concat()、slice()

  - #### v-for中key的作用

    ##### 在使用v-for进行列表渲染时，我们通常会给元素或者组件绑定一个key属性

    ```html
    <li v-for="(movie, index) in movies" :key="movie">{{movie}}</li>
    ```

    

    ##### 作用：

    - key属性主要用于Vue的虚拟DOM算法，在新旧nodes对比时辨识VNodes
    - 如果不使用key，Vue会使用一种最大限度减少动态元素并且尽可能地尝试就地修改/复用相同类型元素的算法
    - 使用key时，会基于key的变化重新排列元素顺序，并且移除/销毁key不存在的元素

  
  - #### 认识Vnode
  
    VNode即虚拟节点，无论是组件或者是元素，最终在Vue中都表示为VNode。VNode本质上是一个JavaScript对象
  
    ```html
    <div class="title" style="font-size: 30px; color:red;">哈哈哈</div>
    const vnode = {
    	type: "div",
    	props: {
    		class: "title",
    		style: {
    			"font-size": "30px",
                color: "red",
    		},
    	},
    	children: "哈哈哈",
    };
    ```
  
    大量VNode节点连接形成树结构，即虚拟DOM
  
    
