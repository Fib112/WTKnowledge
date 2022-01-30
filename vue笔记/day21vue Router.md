### 一、认识前端路由

路由其实是网络工程中的一个术语：

- 在架构一个网络时，非常重要的两个设备就是路由器和交换机
- 事实上，路由器主要维护的是一个映射表，映射表会决定数据的流向

路由的概念在软件工程中出现，最早是在后端路由中实现的，原因是web的发展主要经历了这样一些阶段：

- 后端路由阶段
- 前后端分离阶段
- 单页面富应用（SPA）

#### 1.1 后端路由阶段：

- 早期的网站开发整个HTML页面是由服务器来渲染的，服务器直接生产渲染好对应的HTML页面，返回给客户端进行展示
- 但是, 一个网站, 这么多页面服务器如何处理呢
  - 一个页面有自己对应的网址, 也就是URL
  - URL会发送到服务器, 服务器会通过正则对该URL进行匹配, 并且最后交给一个Controller进行处理
  - Controller进行各种处理, 最终生成HTML或者数据, 返回给前端
- 上面的这种操作, 就是后端路由：
  - 当我们页面中需要请求不同的路径内容时, 交给服务器来进行处理, 服务器渲染好整个页面, 并且将页面返回给客户端
  - 这种情况下渲染好的页面, 不需要单独加载任何的js和css, 可以直接交给浏览器展示, 这样也有利于SEO的优化
- 后端路由的缺点：
  - 一种情况是整个页面的模块由后端人员来编写和维护的
  - 另一种情况是前端开发人员如果要开发页面, 需要通过PHP和Java等语言来编写页面代码
  - 而且通常情况下HTML代码和数据以及对应的逻辑会混在一起, 编写和维护都是非常糟糕的事情

#### 1.2 前后端分离阶段：

- 前端渲染的理解：
  - 每次请求涉及到的静态资源都会从静态资源服务器获取，这些资源包括HTML+CSS+JS，然后在前端对这些请求回来的资源进行渲染
  - 需要注意的是，客户端的每一次请求，都会从静态资源服务器请求文件
  - 同时可以看到，和之前的后端路由不同，这时后端只是负责提供API了
- 前后端分离阶段：
  - 随着Ajax的出现, 有了前后端分离的开发模式
  - 后端只提供API来返回数据，前端通过Ajax获取数据，并且可以通过JavaScript将数据渲染到页面中
  - 这样做最大的优点就是前后端责任的清晰，后端专注于数据上，前端专注于交互和可视化上
  - 并且当移动端(iOS/Android)出现后，后端不需要进行任何处理，依然使用之前的一套API即可
  - 目前比较少的网站采用这种模式开发（jQuery开发模式）

#### 1.3 SPA开发阶段：

- SPA最主要的特点就是在前后端分离的基础上加了一层前端路由：
  - 一般情况下，整个网站只有一个html页面
  - 在前后端分离阶段，静态资源服务器中放了好几套html+css+js，对应各个页面。但是在SPA里面，只有一个html+css+js
  - 当你输入网站，它将全部资源都加载到浏览器中
  - 比如你想进入你页面的首页，通过首页的url从浏览器获取的全部资源中抽取出需要的组件

#### 1.4 URL的hash

前端路由是如何做到URL和内容进行映射呢？监听URL的改变

URL的hash：

- URL的hash也就是锚点(#), 本质上是改变window.location的href属性

- 我们可以通过直接赋值location.hash来改变href, 但是页面不发生刷新

  ```html
  <div>
      <a href="#/home">home</a>
      <a href="#/about">about</a>
      <div class="router-view">default</div>
  </div>
  
  <script>
      //1.获取内容展示模块
  	const routerViewEl = document.querySelector(".router-view");
      
      //2.监听hashchange
      window.addEventListener("hashchange", () => {
          switch(location.hash) {
              case "#/home": 
                  routerViewEl.innerHTML = "home";
                  break;
              case "#/about": 
                  routerViewEl.innerHTML = "about";
                  break;
              default: 
                  routerViewEl.innerHTML = "default";
          }
      })
  </script>
  ```

hash的优势就是兼容性更好，在老版IE中都可以运行，但是缺陷是有一个#，显得不像一个真实的路径

#### 1.5 HTML5的History模式

history接口是HTML5新增的, 它有六种模式改变URL而不刷新页面：

- replaceState：替换原来的路径
- pushState：使用新的路径
- popState：路径的回退
- go：向前或向后改变路径
- forward：向前改变路径
- back：向后改变路径

然后可以调用事件监听，当路径变化时切换到不同页面

```html
<div>
    <a href="/home">home</a>
    <a href="/about">about</a>
    <div class="router-view">default</div>
</div>

<script>
	const aEls = document.getElementByTagName("a");
    const routerViewEl = document.querySelector(".router-view");
    
    const contentChange = function() {
        switch(location.href) {
            case "home": 
                routerViewEl.innerHTML = "home";
                break;
            case "about": 
                routerViewEl.innerHTML = "about";
                break;
            default:
                routerViewEl.innerHTML = "default";
        }
    }
    for(let aEl of aEls) {
        aEl.addEventListener("click", e => {
            e.preventDefault();
            const href = aEl.getAttribute("href");
            history({}, "", href);
            contentChange();
        })
    }
    //监听回退，调用事件
    window.addEventListener("popState", contentChange);
</script>
```

popState会将路径压入记录栈中，使用popState会有前进回退功能，而replaceState则是替换路径而不会压入记录栈，所以没有前进回退

### 二、Vue-Router

#### 2.1 认识Vue-Router

目前前端流行的三大框架, 都有自己的路由实现：

- Angular的ngRouter
- React的ReactRouter
- Vue的vue-router

Vue Router 是Vue.js 的官方路由。它与Vue.js 核心深度集成，让用Vue.js 构建单页应用变得非常容易，目前Vue路由最新的版本是4.x版本。

vue-router是基于路由和组件的：

- 路由用于设定访问路径, 将路径和组件映射起来
- 在vue-router的单页面应用中, 页面的路径的改变就是组件的切换

安装Vue Router：`npm install vue-router@4`

#### 2.2 路由的使用步骤

使用vue-router的步骤：

- 第一步：创建路由组件的组件

  ```vue
  //pages文件夹下的Home.vue
  <template>
  	<div>
          Home组件
      </div>
  	<h2>{{message}}</h2>
  </template>
  
  <script>
  	export default {
          data() {
              return {
                  message: "Hello World"
              }
          }
      }
  </script>
  ```

  

- 第二步：配置路由映射: 组件和路径映射关系的routes数组

  ```javascript
  //router文件夹下index.js
  import { createRouter, createWebHashHistory } from 'vue-router';
  
  import Home from '../pages/Home.vue';
  
  const routes = [ //配置路由映射
      {
          path: '/home',
          component: Home
      },
      {
          path: '/about',
          component: About
      }
  ];
  
  const router = createRouter({ //创建router对象
      routes,
      history: createWebHashHistory() //选择hash模式/history模式
  });
  
  export default router
  ```

  

- 第三步：通过createRouter创建路由对象，并且传入routes和history模式，并使用路由

  ```javascript
  //main.js
  import router from './router'
  
  createApp(App).use(router).mount("#app"); //通过use注册路由插件
  ```

  

- 第四步：使用路由: 通过`<router-link>`和`<router-view>`

  ```vue
  //App.vue
  <template>
  	//通过to设置路径，点击router-link会改变href
  	<router-link to="/home">首页</router-link>
  	<router-link to="/about">关于</router-link>
  	
  	<router-view></router-view> //当href发生变化时，router-view会展示不同组件
  </template>
  ```

#### 2.3 路由默认路径及router-link参数

- 默认情况下, 进入网站的首页, 我们希望`<router-view>`渲染首页的内容，但是我们的实现中, 默认没有显示首页组件, 必须让用户点击才可以，如何可以让路径默认跳到到首页, 并且<router-view>渲染首页组件呢？我们可以在routes中配置一个映射：

  ```javascript
  const routes = [
      {path: '/', redirect: Home},
      ...
  ]
  ```

  path配置的是根路径: /，redirect是重定向, 也就是我们将根路径重定向到/home的路径下, 这样就可以得到我们想要的结果了

- router-link

  router-link事实上有很多属性可以配置：

  - to属性：是一个字符串，或者是一个对象
  - replace属性：设置replace 属性的话，当点击时，会调用router.replace()，而不是router.push()
  - active-class属性：设置激活a元素后应用的class，默认是router-link-active。利用这个class可以自定义a元素激活后的样式
  - exact-active-class属性：链接精准激活时，应用于渲染的<a> 的class，默认是router-link-exact-active

#### 2.4 懒加载

当打包构建应用时，JavaScript 包会变得非常大，影响页面加载：

- 如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就会更加高效，也可以提高首屏的渲染效率

这里属于webpack的分包知识，而Vue Router默认就支持动态来导入组件：

- 这是因为component可以传入一个组件，也可以接收一个函数，该函数需要放回一个Promise
- 而import函数就是返回一个Promise

```javascript
//router文件夹下的index.js

const routes = [
    {path: "/", redirect: "./home"}
    {path: "/home", component: import("../pages/Home.vue")},
    {path: "/about", component: () => import("../pages/About.vue")}
]
```

在对打包效果分析后发现分包是没有一个很明确的名称的，其实webpack从3.x开始支持对分包进行命名（chunk name）：

```javascript
const routes = [
    {path: "/", redirect: "./home"}
    {path: "/home", component: import(/* webpackChunkName: "home-chunk"*/"../pages/Home.vue")},
    {path: "/about", component: () => import("../pages/About.vue")}
]
```

#### 2.5 路由的其他属性

- name属性：路由记录独一无二的名称

- meta属性：自定义的数据

  ```javascript
  //router文件夹下index.js
  const routes = [
      {
          path: "/home",
          name: "home-router",
          component: () => import("../pages/Home.vue"),
          meta: {
              name: "why",
              age: 18
          }
      }
  ]
  ```

  

#### 2.6 动态路由基本匹配

很多时候我们需要将给定匹配模式的路由映射到同一个组件：

- 例如，我们可能有一个User 组件，它应该对所有用户进行渲染，但是用户的ID是不同的，在不同用户访问User页面时，我们希望能根据不同ID显示不同href而不是同一href为`/user`

- 在Vue Router中，我们可以在路径中使用一个动态字段来实现，我们称之为路径参数

  ```javascript
  //router文件夹下index.js
  const routes = [
      {
          path: "/user: id",
          component: () => import("../pages/User.vue")
      }
  ]
  ```

- 在router-link中进行如下跳转：

  ```vue
  <router-link to="/user/123"></router-link>
  ```

获取动态路由的值：

- 那么在User中如何获取到对应的值呢

  - 在template中，直接通过$route.params获取值

  - 在created中，通过this.$route.params获取值

  - 在setup中，我们要使用vue-router库给我们提供的一个hook：useRoute。该Hook会返回一个Route对象，对象中保存着当前路由相关的值

    ```vue
    //User.vue
    <template>
    	<div>{{$route.params.id}}</div>
    </template>
    
    <script>
    	import { useRoute } from 'vue-router';
        
        export default {
            created() {
                console.log(this.$route.params.id);
            },
            setup() {
                console.log(useRoute().params.id);
            }
        }
    </script>
    ```

匹配多个参数

在routes的route对象中可以传入多个动态参数：

```javascript
const routes = [
    {
        path: "user/:id/info/:name",
        component: () => import("../pages/User.vue")
    }
]
```

在`<router-link>`中匹配：

| 匹配模式                       | 匹配路径                                             | $route.params            |
| ------------------------------ | ---------------------------------------------------- | ------------------------ |
| `path: "/user/:id"`            | `<router-link to="/user/123"><router-link>`          | `{id: 123}`              |
| `path: "/user/:id/info/:name"` | `<router-link to="/user/123/info/why"><router-link>` | `{id: 123, name: "why"}` |

#### 2.7 NotFound页面的路径匹配

对于那些没有匹配到的路由，我们通常会匹配到固定的某个页面，以提示用户路径输入错误。比如NotFound的错误页面中，这个时候我们可编写一个动态路由用于匹配所有不存在的路径

```vue
//NotFound.vue
<template>
	<h2>
        Page Not Found
    </h2>
</template>
```

```javascript
const routes = [
    {...},
    {...},
    {
        //固定写法,当路由匹配不到上面的页面路径时,会在这里匹配
        path: "/:pathMatch(.*)", 
        component: () => import("../pages/NotFound.vue")
    }
]
```

我们可以通过$route.params.pathMatch获取到传入的参数：

```vue
//NotFound.vue
<template>
	<h2>
        Page Not Found
    </h2>
	<h2>{{$route.params.pathMatch}}</h2>
</template>
```

匹配规则加*：

```
 path: "/:pathMatch(.*)*", 
        component: () => import("../pages/NotFound.vue")
```

它们的区别在于解析的时候，是否解析”/“：

| 路径           | 匹配规则                  | 解析结果                                         |
| -------------- | ------------------------- | ------------------------------------------------ |
| user/hahah/123 | `path: "/:pathMatch(.*)`  | params.pathMatch(string):"user/hahah/123"        |
| user/hahah/123 | `path: "/:pathMatch(.*)*` | params.pathMatch(array):["user", "hahah", "123"] |

#### 2.8 路由的嵌套

目前我们匹配的Home、About、User等都属于底层路由，我们在它们之间可以来回进行切换。但是呢，我们Home页面本身，也可能会在多个组件之间来回切换：

- 比如Home中包括Product、Message，它们可以在Home内部来回切换
- 这个时候我们就需要使用嵌套路由，在Home中也使用router-view 来占位之后需要渲染的组件

在Home组件中：

```vue
<template>
  <div>
    <h2>Home</h2>
    <ul>
      <li>home的内容1</li>
      <li>home的内容2</li>
      <li>home的内容3</li>
    </ul>
    <router-view/>

    <router-link to="/home/message">消息</router-link>
    <router-link to="/home/shops">商品</router-link>
  </div>
</template>

<script>
  export default {
    
  }
</script>

<style scoped>

</style>
```

配置嵌套路由：

```javascript
//router文件夹下index.js文件中
const routes = [
    {
        path: "/home",
        component: () => import("../pages/Home.vue"),
        children: [
            {
                path: "/home/message" //重定向需要写完整路径
            }
            {
                path: "message", //不需要加斜杠
                component: () => import("../pages/HomeMessage.vue")
            },
            {
                path: "shops",
                component: () => import("../pages/HomeShops.vue")
            }
        ]
    }
]
```

router-link中的exact-active-class属性：

- 当路径跳转到home组件中的message中时，首页的router-link标签和message的router-link标签都会加上active-class类属性
- 而exact-active-class类属性则会精准匹配到message的router-link标签

#### 2.9 编程式导航

之前的路由跳转都是通过内置的router-link标签进行路径修改，如果希望通过自定义元素进行条转，则需要编程式导航：

```vue
<template>
	<button @click="jumpToHome">
        首页
    </button>
</template>

<script>
	export default {
        methods: {
            jumpToHome() {
                this.$router.push("/home");
            }
        }
    }
</script>
```

push方法还可以传入一个对象：

```javascript
this.$router.push({
	path: "/home"
})
```

在setup中通过useRouter获取router对象：

```javascript
import {useRouter} from "vue-router"

export default {
    setup() {
        const router = useRouter();
        jumpToHome = function() {
            router.push("/home");
        }
        return {
            jumpToHome
        }
    }
}
```

query方式的参数：

```vue
<script>
	import {useRouter} from "vue-router";
    
    export default {
        setup() {
            const router = useRouter();
            function jumpToHome() {
                router.push({
                    path: "/home",
                    query: {
                        name: "coderwhy",
                        age: 18
                    }
                })
            }
        }
    }
</script>
```

跳转到home组件时，url后面会带有查询参数：`xxxxx/home?name=coderwhy&age=18`，且home组件可以获得查询参数：

```vue
//home.vue组件
<h2>
    {{$route.query.name}}-{{$route.query.age}}
</h2>
```

除了push方法外，router对象还有replace、go、forward、back等方法

#### 2.10 router-link的v-slot

`<router-link>`里面可以放其他标签甚至自定义组件作为插槽，使用v-slot作用域插槽可以将某些参数传递到插槽中

```vue
<router-link v-slot="props">
	<h2>关于</h2>
    <button>about</button>
</router-link>
```

props中具有很多属性：

- href：解析后的URL

- route：解析后的规范化的route对象

- navigate：触发导航的函数

  在`<router-link>`中如果使用custom标志，则内部的元素或自定义组件不会包裹在`<router-link>`标签中，此时如果需要实现路由跳转需要使用编程式导航，但vue-router提供了更简便的方式，就是传入navigate导航函数：

  ```vue
  <router-link v-slot="props" custom>
  	<button @click="props.navigate">首页</button>
  </router-link>
  ```

- isActive：布尔值，是否匹配的状态

- isExactActive：布尔值，是否是精准匹配的状态；

#### 2.11 router-view的v-slot

router-view也提供给我们一个插槽，可以用于<transition> 和<keep-alive> 组件来包裹你的路由组件：

- Component：要渲染的组件

- route：解析出的标准化路由对象

  ```vue
  <router-view v-slot="props">
  	<transition>
      	<component :is="props.Component"></component>
      </transition>
  </router-view>
  
  <style>
  	//书写动画样式
  </style>
  ```

#### 2.12 动态添加路由

某些情况下我们可能需要动态的来添加路由：

- 比如根据用户不同的权限，注册不同的路由这个时候我们可以使用一个方法addRoute

  ```javascript
  const categoryRoute = {
  	path: "/category",
      name: "category",
      component: () => import("../pages/Category.vue")
  }
  
  router.addRoute(categoryRoute);
  ```

- 如果我们是为route添加一个children路由，那么可以传入对应的name：

  ```javascript
  const categoryMonentRoute = {
      path: "monent",
      component: () => import("../pages/categoryMonent.vue")
  }
  
  router.addRoute("category", categoryMonentRoute);
  ```

动态删除路由：

- 方式一：添加一个name相同的路由

  ```javascript
  router.addRoute({path: "/home", name: "home", component: () => import("../pages/Home.vue")});
  //这会删除此前已经添加的路由，因为他们具有相同的名字并且名字必须是唯一的
  router.addRoute({path: "/other", name: "home", component: () => import("../pages/Other.vue")})
  ```

- 方式二：通过removeRoute方法，传入路由的名称

  ```javascript
  router.addRoute({path: "/home", name: "home", component: () => import("../pages/Home.vue")});
  router.removeRoute("home");
  ```

- 方式三：通过addRoute方法的返回值回调

  ```javascript
  const removeRoute = router.addRoute({path: "/home", component: () => import("../pages/Home.vue")});
  removeRoute();
  ```

路由的其他方法补充：

- router.hasRoute()：检查路由是否存在
- router.getRoutes()：获取一个包含所有路由记录的数组

#### 2.13 路由导航守卫

vue-router 提供的导航守卫主要用来通过跳转或取消的方式守卫导航

全局的前置守卫beforeEach是在导航触发时会被回调的：

- 它有两个参数：
  - to：即将进入的路由Route对象
  - from：即将离开的路由Route对象
- 它有返回值：
    - false：取消当前导航
    - 不返回或者undefined：进行默认导航
    - 返回一个路由地址：
      - 可以是一个string类型的路径
      - 可以是一个对象，对象中包含path、query、params等信息，类似于router.push()的操作
- 可选的第三个参数：next
    - 在Vue2中我们是通过next函数来决定如何进行跳转的
    - 但是在Vue3中我们是通过返回值来控制的，不再推荐使用next函数，这是因为开发中很容易调用多次next

登录守卫功能：

只有登录后才能看到其他页面

```javascript
//router文件夹下的index.js
router.beforeEach(from, to) {
    if(to.path !== "/login") {
        const token = window.localStorage.getItem("token");
        if(!token) {
            return "/login";
        }
    }
}
```

其他导航守卫：

Vue还提供了很多的其他守卫函数，目的都是在某一个时刻给予我们回调，让我们可以更好的控制程序的流程或者功能：<a href="https://next.router.vuejs.org/zh/guide/advanced/navigation-guards.html">https://next.router.vuejs.org/zh/guide/advanced/navigation-guards.html</a>

完整的导航解析流程：

- 导航被触发
- 在失活的组件里调用beforeRouteLeave守卫
- 调用全局的beforeEach守卫
- 在重用的组件里调用beforeRouteUpdate守卫(2.2+)
- 在路由配置里调用beforeEnter
- 解析异步路由组件
- 在被激活的组件里调用beforeRouteEnter
- 调用全局的beforeResolve守卫(2.5+)
- 导航被确认
- 调用全局的afterEach钩子
- 触发DOM 更新
- 调用beforeRouteEnter守卫中传给next 的回调函数，创建好的组件实例会作为回调函数的参数传入

