- ## 为何methods不能使用箭头函数？

  - 因为箭头函数不能绑定this，会直接绑定上级域中的this，即window  



- ## methods中的this指向？

  - 在vue源码中通过bind绑定publicThis

-  ## mustache双大括号语法

  1. 可以插入data中的属性

  2. 还可以是表达式甚至是三元运算符

  3. 但不能是语句，比如赋值语句，判断语句等

- ## v-once指令

  ​	用于指定元素或者组件只渲染一次

  ​    用法：

  		```html
  		<h2 v-once> {{counter}} </h2>
  		```

   - 当数据发生变化时，元素或者组件以及所有子元素将视为静态内容并跳过

- ## v-text指令

   - 用于更新元素的textContent：

     ```html
     <span v-text="msg"></span>
     //等价于
     <span> {{msg}}</span>
     ```

- ## v-html指令

   - 将数据作为html元素渲染

     ​	

     ```html
     <div v-html='info'></div>
     ...
     data() {
     	return {
     		info: `<span>hello</span>    
     	}
     }
     ```

  - 如果不适用v-html，而是直接{{info}}，将会将span标签视为普通字符串渲染

- ## v-pre指令

   - 用于跳过元素及其子元素编译，加快编译速度

     ​	

     ```html
     <div v-pre>{{msg}}</div>
     ```

  - 浏览器显示为{{msg}}，而不使用mustache语法

- ## v-cloak

   - 这个指令保持在元素上直到关联组件实例结束编译

   - 和css规则如 [v-cloak] { display: none }一起用时，这个指令可以隐藏未编译的mustache标签直到组件实例准备完毕

     ​	

     ```html
     <style>
         [v-cloak] {
             display: none;
         }
     </style>
     ...
     <template>
     	<div v-cloak>
             {{msg}}
         </div>
     </template>
     ```

  - `<div>`不会显示，直到编译结束

- ## **v-bind绑定属性** 

   1. 绑定基本属性

      - 比如图片链接src、网站href、动态绑定类、样式等

      ```html
      <img v-bind:src="src">
      //简写
      <img :src="src">
      ```

      

   2. 绑定class

      1. 对象语法

         - 传给:class一个对象，以动态切换class

         ```html
         //普通绑定
         <div :class="calssName"></div>
         //对象绑定 对象值为boolean，可以引用data中的属性(boolean)
         //键名有空格或连字符要用引号
         <div :class="{nba: true,"ja-mes": true}"></div>
         //绑定对象
         <div :class="classObj"></div>
         //从methods中获取
         <div :class="getClassObj()"></div>
         ```

      2. 数组语法

         - 将一个数组传给:class，以应用一个class列表

           ```html
           //直接传入一个数组,不加引号的是使用data中的属性
           <div :class="['why', nba]"></div>
           //还可以使用三元运算符
           <div :class="['why', nba, isActive? 'active': '']">				</div>
           //还可以使用对象语法
           <div :class="['why', nba, {'active': isActive}]">				</div>
           ```

     3. 绑定style

        1. 对象语法

        	```html
        	//基本使用
        	<div :style="{color: 'red', 'font-size': '30px'}">				</div>
        	//传入一个对象，值来自data
        	<div :style="{color: 'red', 'font-size': size+'px'}">			</div>
        	//直接在data中定义好对象在这里使用
        	<div :style="styleObj"></div>
        	```

        2. 数组语法

           可以将多个样式对象应用到同一个元素上

        	```html
        	<div :style="[styleOBj1, styleObj2]"></div>
        	```

     4. 动态绑定属性

      	某些情况下，属性名称可能不固定，可以通过v-bind动态绑定属性
      	
      	```html
      	<div :[name]="value">{{msg}}</div>
      	```

   

   5. 将一个对象的所有属性，绑定到元素上的所有属性

      	```html
      	<div v-bind="info"></div>
      	 ...
      	data() {
      		return {
      			info: {
      				name: "why",
      				age: 18,
      				height: 1.80
      			}
      		}
      	}
      	//绑定后的效果
      	<div name="why" age="18" height="1.80"></div>
      	```

- ## v-on绑定事件

  1. 基本使用

     ```html
     //绑定事件
     <button @click="btnClick">按钮</button>
     <div @mouseMove="mouseMove">div区域</div>
     //绑定表达式
     <button @click="counter++">
         {{counter}}
     </button>
     //传入对象绑定多个事件,此处需用v-on，而不是@(不够直观)
     <button v-on="{click: btnClick, mouseMove: mouseMove}">
         特殊按钮
     </button>
     ```

  2. 参数传递
  
     1. 如果该方法不需要额外参数(无参数或参数为event)，那么方法可以不加()，如果方法本身有一个参数，那么会默认将事件event传递进去
  
        ```html
        <button @click="btnClick">按钮</button>
        ...
        methods: {
        	btnClick() {
        		console.log("点击了按钮");
        	}
        }
        //or
        methods: {
        	btnClick(event) {
        		console.log(event.target);
        	}
        }
        ```
  
     2. 如果需要传入某个参数，同时需要event时，可通过$event传入事件
  
        ```html
        <button @click="btn1Click($event, 'why')">按钮1</button>
        ...
        btn5Click(event, msg) {
        	console.log(event, msg);
        }
        ```

  3. 修饰符

     相当于对事件进行一些特殊处理：

     .stop --调用event.stopPropagation()

     .prevent --调用event.preventDefault()
  
     ...
  
     ```html
     <div @click="divClick">
         //阻止冒泡
         <button @click.stop="btnClick">
             按钮6
         </button>
     </div>
     ```
  
     