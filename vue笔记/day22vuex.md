### 一、状态管理与vuex安装

#### 1.1 状态管理

- 在开发中，我们会的应用程序需要处理各种各样的数据，这些数据需要保存在我们应用程序中的某一个位置，对于这些数据的管理我们就称之为是状态管理

- 在前面我们是如何管理自己的状态呢？

  - 在Vue开发中，我们使用组件化的开发方式
  - 而在组件中我们定义data或者在setup中返回使用的数据，这些数据我们称之为state
  - 在模块template中我们可以使用这些数据，模块最终会被渲染成DOM，我们称之为View
  - 在模块中我们会产生一些行为事件，处理这些行为事件时，有可能会修改state，这些行为事件我们称之为actions

- 复杂的状态管理

  - JavaScript开发的应用程序，已经变得越来越复杂了：

    - JavaScript需要管理的状态越来越多，越来越复杂
    - 这些状态包括服务器返回的数据、缓存数据、用户操作产生的数据等等
    - 也包括一些UI的状态，比如某些元素是否被选中，是否显示加载动效，当前分页

  - 当我们的应用遇到多个组件共享状态时，单向数据流的简洁性很容易被破坏：

    - 多个视图依赖于同一状态
    - 来自不同视图的行为需要变更同一状态

  - 我们是否可以通过组件数据的传递来完成呢？

    - 对于一些简单的状态，确实可以通过props的传递或者Provide的方式来共享状态

    - 但是对于复杂的状态管理来说，显然单纯通过传递和共享的方式是不足以解决问题的，比如兄弟组件之间数据的共享

- Vuex的状态管理

  - 管理不断变化的state本身是非常困难的：
    - 状态之间相互会存在依赖，一个状态的变化会引起另一个状态的变化，View页面也有可能会引起状态的变化
    - 当应用程序复杂时，state在什么时候，因为什么原因而发生了变化，发生了怎么样的变化，会变得非常难以控制和追踪
  - 因此，我们可以考虑将组件的内部状态抽离出来，以一个全局单例的方式来管理：
    - 在这种模式下，我们的组件树构成了一个巨大的“视图View”
    - 不管在树的哪个位置，任何组件都能获取状态或者触发行为
    - 通过定义和隔离状态管理中的各个概念，并通过强制性的规则来维护视图和状态间的独立性，我们的代码边会变得更加结构化和易于维护、跟踪

#### 1.2 Vuex的安装与使用

- 要使用vuex，首先第一步需要安装vuex：
  - 这里使用的是vuex4.x，安装的时候需要添加next 指定版本`npm install vuex@next`

- 创建Store

  - 每一个Vuex应用的核心就是store（仓库）：store本质上是一个容器，它包含着你的应用中大部分的状态（state）

- Vuex和单纯的全局对象之间的区别：

  1. Vuex的状态存储是响应式的：当Vue组件从store中读取状态的时候，若store中的状态发生变化，那么相应的组件也会被更新

  2. 不能直接改变store中的状态，改变store中的状态的唯一途径就显示提交(commit)mutation，这样使得我们可以方便的跟踪每一个状态的变化，从而让我们能够通过一些工具帮助我们更好的管理应用的状态

- 使用步骤：

  1. 创建Store对象

     ```javascript
     //store文件夹下index.js
     //创建Store对象
     import {createStore} from 'vuex';
     const store = createStore({
     	state() {
             return {
                 counter: 100
             }
         },
         mutation() {
             increment(state) {
                 state.counter++;
             }
         }
     });
     export default store;
     ```

  2. 在app中通过插件安装

     ```
     //main.js
     import store from "./store";
     ...
     
     createApp(App).use(store).mount('#app');
     ```

  3. 组件中使用store

     在组件中使用store，我们按照如下的方式：

     1. 在模板中使用

        ```vue
        <template>
        	<div>$store.state.counter</div>
        </template>
        ```

     2. 在options api中使用，比如methods

        ```vue
        <script>
        	export default {
                methods: {
                    increment() {
                        //不要在methods中直接修改store中的state,而是提交mutation
                    	this.$store.commit("increment");
                    }
                }
            }
        </script>
        ```

     3. 在setup中使用

        ```vue
        <template><div>{{sCounter}}</div></template>
        
        <script>
        	//在setup中获取store需要使用useStore钩子函数
            import { useStore } from "vuex"
            import { computed } from 'vue';
            
            export default {
                setup() {
                    const store = useStore();
                    const sCounter = computed(() => store.state.counter);
                    
                    return {
                        sCounter
                    }
                }
            }
        </script>
        ```

### 二、单一状态树和mapState

#### 2.1 单一状态树

Vuex 使用单一状态树：

- 用一个对象就包含了全部的应用层级的状态
- 采用的是SSOT，Single Source of Truth，也可以翻译成单一数据源
- 这也意味着，每个应用将仅仅包含一个store 实例
- 单状态树和模块化并不冲突

单一状态树的优势：

- 如果状态信息是保存到多个Store对象中的，那么之后的管理和维护等等都会变得特别困难
- 所以Vuex也使用了单一状态树来管理应用层级的全部状态
- 单一状态树能够以最直接的方式找到某个状态的片段，而且在之后的维护和调试过程中，也可以非常方便的管理和维护

#### 2.2 mapState使用

在前面已经学习过如何在组件中获取状态了，如果觉得那种方式有点繁琐（表达式过长），可以使用计算属性：

```vue
<template>
	<h2>{{sCounter}}</h2>
</template>

<script>
	export default {
        computed: {
            sCounter() {
                return this.$store.state.counter;
            }
        }
    }
</script>
```

但是，如果我们有很多个状态都需要获取话，可以使用mapState的辅助函数：

```vue
<template>
	<h2>counter</h2>
	<h2>age</h2>
	<h2>height</h2>
</template>

<script>
	import {mapState} from 'vuex';
    
    export default {
        computed: {
            //mapState返回的是对象,需要进行结构
            ...mapState(["counter", "age", "height"])//数组写法
            ...mapState({
            	//对象写法,可以重命名数据
            	sCounter: state => state.counter; 
        	})
        }
    }
</script>
```

在setup API中使用：

```vue
<template>
	<div>{{counter}}</div>
	<div>{{height}}</div>
	<div>{{age}}</div>
</template>

<script>
	import { useStore, mapState } from "vuex";
    import { computed } from "vue";
    
    export default {
        setup() {
            const store = useStore();
            //mapState返回的是函数,所以才能在computed中注册并在模板进行显示
            //但这些函数不能直接传进computed进行注册
            //因为这些函数内部是通过this.$store访问store拿到数据的,而setup中并没有this
            const storeStateFns = mapState(["counter", "height", "age"]);
            //创建空对象储存数据
            const storeState = {};
            Object.keys(storeStateFns).forEach(fnKey => {
                绑定一个能通过this.$store访问到store的对象
                const fn = storeStateFns[fnKey].bind({$store:store});
                storeState[fnKey] = computed(fn)//computed返回一个ref对象
            });
            
            return {
        		...storeState
    		}
        }
    }
</script>
```

封装以上方法：

```javascript
//hooks文件夹下useState.js
import { useStore, mapState } from "vuex";
import { computed } from "vue";

export function useState(mapper) {
    const store = useStore();
    const storeStateFns = mapState(mapper);
    const storeState = {};
    Object.keys(storeStateFns).forEach(fnKey => {
        const fn = storeStateFns[fnKey].bind({$store:store});
        storeState[fnKey] = computed(fn);
    });
    return storeState;
}
```

在setup中使用：

```vue
<template>
	<div>
        {{counter}}-{{sCounter}}
    </div>
</template>

<script>
    import {useState} from "./hooks/useState.js"
	export default {
        setup() {
            const storeState = useState(["counter"]);
            const storeState1 = useState({
                sCounter: state => state.counter
            });
            
            return {
                ...storeState,
                ...storeState1
            }
        }
    }
</script>
```

#### 2.3 getter

某些属性我们可能需要进行变化后来使用，这个时候可以使用getters：

- 基本使用：

  ```javascript
  //store文件夹下index.js
  const store = createStore({
      state() {
          return {
              books: [
                  {name: "vuejs", count: 2, price: 100},
                  {name: "webpack", count: 3, price: 200},
                  {name: "react", count: 4, price: 300}
              ]
          }
      },
      getters: {
          totalPrice(state) {
              let price = 0;
              for(let book of state.books) {
                  price += book.price * book.count;
              }
              return price;
          }
      }
  })
  ```

  在模板中使用：

  ```vue
  <template>
  	<div>
          {{$state.getters.totalPrice}}
      </div>
  </template>
  ```

- getters第二个参数：

  getters可以接收第二个参数：getters，来调用getters中的函数

  ```javascript
  //store文件夹下index.js
  const store = createStore({
      state() {
          return {
              books: [
                  {name: "vuejs", count: 2, price: 100},
                  {name: "webpack", count: 3, price: 200},
                  {name: "react", count: 4, price: 300}
              ],
              discount: 0.9
          }
      },
      getters: {
          totalPrice(state, getters) {
              let price = 0;
              for(let book of state.books) {
                  price += book.price * book.count;
              }
              return price * getters.currentDiscount;
          }
          currentDiscount(state) {
      		return state.discount * 0.8;
  		}
      }
  })
  ```

- getters的返回函数

  getters中的函数本身，可以返回一个函数，那么在使用的地方相当于可以调用这个函数：

  ```javascript
  //store文件夹下index.js
  const store = createStore({
      state() {
          return {
              books: [
                  {name: "vuejs", count: 2, price: 100},
                  {name: "webpack", count: 3, price: 200},
                  {name: "react", count: 4, price: 300}
              ],
              discount: 0.9
          }
      },
      getters: {
          totalPriceCountGreaterN(state, getters) {
              return function(n) {
                  let price = 0;
              	for(let book of state.books) {
                  	if(book.count > n) {
                          price += book.count * book.price
                      }
              	}
                  return price * getters.currentDiscount;
              }
          }
          currentDiscount(state) {
      		return state.discount * 0.8;
  		}
      }
  })
  ```

  在模板中使用：

  ```vue
  <template>
  	<div>
          数量大于2的总价格:{{$store.getters.totalPriceCountGreaterN(2)}}
      </div>
  </template>
  ```

#### 2.4 map辅助函数封装

在模板中使用getters需要写一长串的`$store.getters.xxx`，vuex同样提供了类似于mapState的映射函数：mapGetters，在setup中使用mapGetters需要像mapState一样进行封装，但二者封装过程十分类似，只是调用的map函数不一样，所以可以对公共代码进行抽取再进行一层封装：

```javascript
//hooks文件夹下useMapper.js
import { useStore } from "vuex";
import { computed } from "vue";

export default function(mapFn) {
    return function useMapper(mapper) {
        const store = useStore();
    	const storeFns = mapFn(mapper);
    	const storeData = {};
    	Object.keys(storeFns).forEach(fnKey => {
        	const fn = storeFns[fnKey].bind({$store:store});
        	storeData[fnKey] = computed(fn);
    	});
    	return storeData;
    }
}
```

在useState中使用：

```javascript
import { mapState } from "vuex";
import useMapper from "./useMapper";

const useState = useMapper(mapState);

export default useState;
```

在useGetters中使用：

```javascript
import { mapGetters } from "vuex";
import useMapper from "./useMapper";

const useGetters = useMapper(mapGetters);

export default useGetters;
```

在setup API中使用：

```vue
<template>
	<div>
        {{totalPriceCountGreaterN(2)}}-{{sTPCGN}}
    </div>
</template>

<script>
	import useGetters from "../hooks/useGetters.js";
    
    export default {
        setup() {
            const storeGetters = useGetters(["totalPriceCountGreaterN"]);
            const storeGetters1 = useGetters({
                //与mapState不同,mapGetters对象写法直接传入getter对应key就可以了
                sTPCGN: "totalPriceCountGreaterN"
            });
            return {
                ...storeGetters,
                ...storeGetters1
            }
        }
    }
</script>
```

#### 2.5 mutations基本使用

更改Vuex 的store 中的状态的唯一方法是提交mutation：

```javascript
//store文件夹下index.js
const store = createStore({
    state() {
        return {
            counter: 100
        }
    },
    mutations: {
        increment(state) {
            state.counter++;
        }
    }
})
```

```vue
<template>
	<div>{{$store.state.counter}}</div>
	<button @click="addOne">+1</button>
</template>

<script>
	export default {
        methods: {
            addOne() {
                this.$store.commit("increment");
            }
        }
    }
</script>
```

mutations携带数据

很多时候我们在提交mutation的时候，会携带一些数据，这个时候我们可以使用参数：

```javascript
//store文件夹下index.js
const store = createStore({
    state() {
        return {
            counter: 100
        }
    },
    mutations: {
        increment(state，payLoad) {
            state.counter += payLoad.count;
        }
    }
})
```

```vue
<template>
	<div>{{$store.state.counter}}</div>
	<button @click="addTen">+10</button>
</template>

<script>
	export default {
        methods: {
            addTen() {
                this.$store.commit("increment", {count: 10});
                //另一种提交方式
                this.$store.commit({
                    type: "increment",
                    count: 10
                })
            }
        }
    }
</script>
```

有时候可能会将commit的mutation事件写错，在调试的时候可能很麻烦，所以可以将函数名定义为常量：

```
//store文件夹下的mutation-type.js
export const INCREMENT = "increment"
```

定义mutation：

```javascript
import { INCREMENT } from "./mutation-type.js"

const store = createStore({
    state() {
        return {
            counter: 100
        }
    },
    mutations: {
        [INCREMENT](state，payLoad) {
            state.counter += payLoad.count;
        }
    }
})
```

提交mutation：

```vue
import { INCREMENT } from "../mutation-type.js"
methods: {
	addTen(){
		this.$store.commit(INCREMENT, {count: 10});
		//另一种写法
		this.$store.commit({
			type: INCREMENT,
			count: 10
		})
	}
}
```

#### 2.6 mapMutation辅助函数

我们也可以借助于辅助函数，帮助我们快速映射到对应的方法中：

- 在methods中使用：

  ```vue
  <template>
  	<div>{{$store.state.counter}}</div>
  	//在模板中不能使用导入的常量,所以需要写mutation中原本的名字
  	<button @click="increment(10)"> 
          +10
      </button>
  	<button @click="dec">
          -1
      </button>
  </template>
  
  <script>
  	import { mapMutations } from "vuex";
      import { INCREMENT, DECREMENT } from "./store/mutations-type.js"
      
      export default {
          methods: {
              ...mapMutations({
                  dec: DECREMENT
              });
              ...mapMutations([INCREMENT])
          }
      }
  </script>
  ```

- 在setup中使用：

  ```vue
  <template>
  	<div>{{$store.state.counter}}</div>
  	//在模板中不能使用导入的常量,所以需要写mutation中原本的名字
  	<button @click="increment(10)"> 
          +10
      </button>
  	<button @click="dec">
          -1
      </button>
  </template>
  
  <script>
  	import { mapMutations } from "vuex";
      import { INCREMENT, DECREMENT } from "./store/mutations-type.js"
      
      export default {
          setup() {
              //因为mapMutations返回的是包裹函数的对象,可以直接使用,所以不需要进行封装
              const mutation = mapMutations([INCREMENT]);
              const mutation1 = mapMutations({
                  dec: DECREMENT
              })
              
              return {
                  ...mutation,
                  ...mutation1
              }
          }
      }
  </script>
  ```


#### 2.7 actions

action类似于mutation，不同在于：

- Action提交的是mutation，而不是直接变更状态
- Action可以包含任意异步操作

action接受一个重要的参数context：

- context是一个和store实例均有相同方法和属性的context对象
- 所以可以从其中获取到commit方法来提交一个mutation，或者通过context.state 和context.getters 来获取state 和getters，还可以通过context.dispatch来调用actions中的其他函数

进行action的分发：

- 分发使用的是store 上的dispatch函数

  ```javascript
  mutations: {
      increment(state) {
          state.counter++;
      }
  },
  actions: {
      incrementAction(context) {
          context.commit("increment");
      }
  }
  
  //在组件中使用
  methods: {
      increment() {
          this.$store.dispatch("incrementAction");
      }
  }
  ```

- 它也可以携带参数：

  ```javascript
  actions: {
      incrementAction(context, payload) {
          console.log(payload);
          context.commit("increment");
      }
  }
  ```

- n也可以以对象的形式进行分发：

  ```javascript
  methods: {
      increment() {
          this.$store.dispatch({
              type: "incrementAction",
              count: 10
          })
      }
  }
  ```

actions的异步操作：

Action 通常是异步的，那么如何知道action 什么时候结束呢？我们可以通过让action返回Promise，在Promise的then中来处理完成后的操作：

```javascript
actions: {
    incrementAction(context) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                context.commit("increment");
                resolve('异步完成');
            }, 1000)
        })
    }
}
```

```javascript
methods: {
    increment() {
        const res = this.store.dispatch("incrementAction");
        res.then(res => console.log(res, "异步完成"));
    }
}
```

vuex同样提供了actions的辅助函数mapActions，用法与mapMutations一致

#### 2.8 modules

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象，当应用变得非常复杂时，store 对象就有可能变得相当臃肿。为了解决以上问题，Vuex 允许我们将store 分割成模块（module）每个模块拥有自己的state、mutation、action、getter、甚至是嵌套子模块

```javascript
// store/modules/home.js
const homeModules = {
    state() {
        return {
            homeCounter: 100
        }
    },
    getters: {
        ...
    },
    mutaions: {
        ...
    },
    actions: {
        ...
    }
}
        
export default homeModule
```

```javascript
// store/index.js
import { createStore } from 'vuex'
import home from './modules/home'

const store = createStore({
    state() {
        return {
            
        }
    },
    getters: {
        
    },
    mutaions: {
        
    },
    actions: {
        
    },
    modules: {
        home
    }
})
```

module的局部状态：

对于模块内部的mutation 和getter，接收的第一个参数是模块的局部状态对象

```javascript
getters: {
    totalPrice(state) { // 该state是模块局部的state
        ...
    }
}
mutaions: {
    increment(state) {
        state.homeCounter++
    }
}
```

module的命名空间：

- 默认情况下，模块内部的action和mutation仍然是注册在全局的命名空间中的：

  - 这样使得多个模块能够对同一个action 或mutation 作出响应
  - Getter 同样也默认注册在全局命名空间

- 如果我们希望模块具有更高的封装度和复用性，可以添加`namespaced: true` 的方式使其成为带命名空间的模块：

  - 当模块被注册后，它的所有getter、action 及mutation 都会自动根据模块注册的路径调整命名

  ```javascript
  const homeModule = {
      namespaced: true,
      state() {
          return {
              counter: 100
          }
      },
      getters: {
          doubleCounter(state, getters, rootState, rootGetters) {
              return state.counter * 2;
          }
      }
      mutations: {
          increment(state, payload) {
              state.counter += payload.count;
          }
      },
      actions: {
          incrementAction({dispatch, commit, state, rootState, getters, rootGetters}, payload) {
              commit("increment");
          }
      }
  }
  ```

  ```vue
  <template>
  	模块内的状态已经是嵌套的了，使用 `namespaced` 属性不会对其产生影响
  	<div>{{$store.state.home.counter}}</div>
  	<div>{{$store.getters["home/doubleCounter"]}}</div>
  	<button @click="add">home+1</button>
  	<button @click="aDD">home: +1</button>
  </template>
  
  <script>
      import { useStore } from "vuex";
  	export default {
          setup() {
              const store = useStore();
              add = () => {
                  store.commit("home/increment");
              };
              aDD = () => {
                  store.dispatch("home/incrementAction")
              }
          }
      }
  </script>
  ```

module修改或派发根组件

如果我们希望在action中修改root中的state，那么有如下的方式：

```javascript
actions: {
    incrementAction(context) {
        //commit根store中的increment
        context.commit("increment", null, {root: true});
        //dispatch根store中的incrementAction
        context.dispatch("incrementAction", null, {root:  true});
    }
}
```

#### 2.9 module的辅助函数

module的辅助函数有三种使用方法：

- 方式一：通过完整的模块空间名称来查找

  ```vue
  <script>
  	import { mapState, mapGetters, mapMutations, mapActions } from "vuex";
      
      export default {
          computed: {
              ...mapState({
                  counter: state => state.home.counter
              });
              ...mapGetters({
              	doubleCounter: "home/doubleCounter"
          	});
          },
          methods: {
              ...mapMutations(["home/increment"]),
              ...mapActions(["home/incrementAction"])
          }
      }
  </script>
  ```

- 方式二：第一个参数传入模块空间名称，后面写上要使用的属性

  ```vue
  <script>
  	import { mapState, mapGetters, mapMutations, mapActions } from "vuex";
      
      export default {
          computed: {
              ...mapState("home", ["counter"]),
              ...mapGetters("home", ["doubleCounter"])
          },
          methods: {
              ...mapMutations("home", ["increment"]),
              ...mapActions("home", ["incrementActions"])
          }
      }
  </script>
  ```

- 方式三：通过createNamespacedHelpers 生成一个模块的辅助函数

  ```vue
  <script>
  	import { createNamespacedHelpers } from "vuex";
      const { mapState, mapGetters, mapMutations, mapActions } = createNamespacedHelpers("home")
      
      export default {
          computed: {
              ...mapState(["counter"]),
              ...mapGetters(["doubleCounter"])
          },
          methods: {
              ...mapMutations(["increment"]),
              ...mapActions(["increment"])
          }
      }
  </script>
  ```
