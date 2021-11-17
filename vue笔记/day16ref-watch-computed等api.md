### 一、Reactive判断的API

1. isProxy：检查对象是否是由 reactive 或 readonly创建的 proxy
2. isReactive：
   - 检查对象是否是由 reactive创建的响应式代理
   - 如果该代理是 readonly 建的，但包裹了由 reactive 创建的另一个代理，它也会返回 true：`let infoReadOnly = readonly(reactive({name: 'coderwhy'}))`
3. isReadonly：检查对象是否是由 readonly 创建的只读代理
4. toRaw：返回 reactive 或 readonly 代理的原始对象（不建议保留对原始对象的持久引用。谨慎使用）
5. shallowReactive：创建一个响应式代理，它跟踪其自身 property 的响应性，但不执行嵌套对象的深层响应式转换 (深层还是原生对象)
6. shallowReadonly：创建一个 proxy，使其自身的 property 为只读，但不执行嵌套对象的深度只读转换（深层还是可读、可写的）

### 二、ref API

#### 2.1 toRefs

如果我们使用ES6的解构语法，对reactive返回的对象进行解构获取值，那么之后无论是修改结构后的变量，还是修改reactive返回的state对象，数据都不再是响应式的：

```js
const state = reactive({
    name: "why",
    age: 18
});
const { name, age } = state //name和age不是响应式的，name和age变化时template不会发生变化
```

Vue为我们提供了一个toRefs的函数，可以将reactive返回的对象中的属性都转成ref，那么我们再次进行结构出来的name 和age 本身都是ref的

```js
const { name, age } = toRefs(state);
```

这种做法相当于已经在state.name和name.value之间建立了链接，任何一个修改都会引起另外一个变化

#### 2.2 toRef

如果我们只希望转换一个reactive对象中的属性为ref, 那么可以使用toRef的方法：

```js
const name = toRef(state, 'name')//第一个参数是待转换的响应式对象，第二个参数是需要转换的属性key值
//返回的对象不用大括号包裹
```

#### 2.3 ref其他API

unref：如果我们想要获取一个ref引用中的value，那么也可以通过unref方法：

- 如果参数是一个ref，则返回内部值，否则返回参数本身
- 这是val= isRef(val) ? val.value: val的语法糖函数

isRef：判断值是否是一个ref对象

shallowRef：创建一个浅层的ref对象

triggerRef：手动触发和shallowRef相关联的副作用，比如深层数据改变强制刷新：

```js
const info = shallowRef({anme:"why"});
const changeName = () => {
    info.value.name = "james"; //此时属于深层次数据改变，shallowRef不会响应
    triggerRef(info);//此时手动触发info刷新
}
```

#### 2.4 customRef

创建一个自定义的ref，并对其依赖项跟踪和更新触发进行显示控制：

- 它需要一个工厂函数，该函数接受track 和trigger 函数作为参数
- 并且应该返回一个带有get 和set 的对象

对双向绑定的属性进行debounce(节流)的操作案例：

```js
//useDebounceRef.js文件
import { customRef } from 'vue';
export function useDebounceRef(value, delay = 200) {
    let timeout = null;
    return customRef((track, trigger) => {
        return {
            get() {
                track(); //收集依赖
                return value;
            },
            set(newValue) {
                clearTimeout(timeout);
                timeout = setTimeout(() => {
                    value = newValue;
                    trigger();//更新触发
                }, delay);
            }
        }
    })
}
```

```vue
//需要使用节流函数的文件
<template>
	<div>
        <input v-model="message">
        <h2>
         {{message}}   
    	</h2>
    </div>
</template>

<script>
	import { useDebounceRef } from './useDebounceRef';
	
	export default {
        setup() {
            const message = useDebounceRef("Hello World");
            return {
                message
            }
        }
    }
</script>
```

### 三、computed

在前面我们讲解过计算属性computed：当我们的某些属性是依赖其他状态时，我们可以使用计算属性来处理

- 在前面的Options API中，我们是使用computed选项来完成的
- 在Composition API中，我们可以在setup 函数中使用computed 方法来编写一个计算属性

computed使用：

- 方式一：接收一个getter函数，并为getter 函数返回的值，返回一个不变的ref 对象
- 方式二：接收一个具有get 和set 的对象，返回一个可变的（可读写）ref 对象

```vue
<template>
	<div>
        {{fullName}}
    </div>
	<button @click="changeName">修改名字</button>
</template>

<script>
	import { computed, ref } from 'vue';
    export default {
        setup() {
            let firstName = ref('Kobe');
            let lastName = ref('Bryant');
            const fullName = computed({
                get() {
                    return firstName.value + ' ' + lastName.value;
                },
                set(newValue) {
                    const names = newValue.split(" ");
                    firstName.value = names[0];
                    lastName.value = names[1];
                }
            })
        }
    }
</script>
```

### 四、侦听数据的变化

在前面的Options API中，我们可以通过watch选项来侦听data或者props的数据变化，当数据变化时执行某一些操作

在Composition API中，我们可以使用watchEffect和watch来完成响应式数据的侦听：

- watchEffect用于自动收集响应式数据的依赖
- watch需要手动指定侦听的数据源

#### 4.1 watchEffect

当侦听到某些响应式数据变化时，我们希望执行某些操作，这个时候可以使用watchEffect

watchEffect使用：

```js
import {watchEffect, ref} from 'vue';
const name = ref("why");
const age = ref(18);

watchEffect(() => {
    console.log("watchEffect执行", name);
})
```

- 首先，watchEffect传入的函数会被立即执行一次，并且在执行的过程中会收集依赖
- 其次，只有收集的依赖发生变化时，watchEffect传入的函数才会再次执行，比如上面的代码中只收集了name依赖，所以当age发生变化时函数不会执行

#### 4.2 watchEffect停止侦听

如果在发生某些情况下，我们希望停止侦听，这个时候我们可以获取watchEffect的返回值函数，调用该函数即可

比如在下面的案例中，在age达到20的时候就停止侦听：

```js
const stopWatch = watchEffect(() => {
	console.log("watchEffect执行", name.value);
})

const changeAge = () => {
    age.value++;
    if(age.value > 20) {
        stopWatch();
    }
}
```

#### 4.3 watchEffect清除副作用

什么是清除副作用呢？

- 比如在开发中我们需要在侦听函数中执行网络请求，但是在网络请求还没有达到的时候，我们停止了侦听器，或者侦听器侦听函数被再次执行了
- 那么上一次的网络请求应该被取消掉，这个时候我们就可以清除上一次的副作用

在我们给watchEffect传入的函数被回调时，其实可以获取到一个参数：onInvalidate

- 当副作用即将重新执行或者侦听器被停止时会执行该函数传入的回调函数
- 我们可以在传入的回调函数中，执行一些清除工作

```js
const stopWatch = watchEffect((onInvalidate) => {
    ... //监听到变化时的回调函数代码
    onInvalidate(() => {
        ... // 在侦听器副作用重新执行或者侦听器被停止时回调的清除副作用代码
    })
})
```

#### 4.4 setup中使用ref

setup中不能使用this，那在setup中如何使用ref或者元素或者组件？

其实非常简单，我们只需要定义一个ref对象，绑定到元素或者组件的ref属性上即可

```vue
<template>
	<div>
        <h2 ref="titleRef">我是标题</h2>
    </div>
</template>

<script>
	import { ref, watchEffect } from 'vue';
    
    export default {
        setup() {
            const titleRef = ref(null);
            watchEffect((invalidate) => {
                console.log(titleRef.value);
            })
            return {
                titleRef
            }
        }
    }
</script>
```

在vue将`h2`标签挂载成功后会赋值到titleRef.value中，所以watchEffect会打印两次，一次是null，另一次就是`h2`标签

#### 4.5 watchEffect的执行时机

默认情况下，组件的更新会在副作用函数执行之前。如果我们希望在副作用函数中获取到元素，是否可行呢。在上面ref的代码中watchEffect会打印两次：

- 这是因为setup函数在执行时就会立即执行传入的副作用函数，这个时候DOM并没有挂载，所以打印为null
- 而当DOM挂载时，会给title的ref对象赋值新的值，副作用函数会再次执行，打印出来对应的元素

果我们希望在第一次的时候就打印出来对应的元素，这个时候我们需要改变副作用函数的执行时机，此时就需要提供watchEffect函数的第二个参数：

- 它的默认值是pre，它会在元素挂载或者更新之前执行，所以我们会先打印出来一个空的，当依赖的title发生改变时，就会再次执行一次，打印出元素

- 我们可以设置副作用函数的执行时机：

  ```js
  watchEffect(() => {
      ...
  }, {
      flush: "post" //此时watchEffect函数会在元素挂载或者更新之后执行
  })
  ```

- flush 选项还接受sync，这将强制效果始终同步触发。然而，这是低效的，应该很少需要

#### 4.6 Watch的使用

watch的API完全等同于组件watch选项的Property：

- watch需要侦听特定的数据源，并在回调函数中执行副作用
- 默认情况下它是惰性的，只有当被侦听的源发生变化时才会执行回调

与watchEffect的比较，watch允许我们：

- 懒执行副作用（第一次不会直接执行）
- 更具体的说明当哪些状态发生变化时，触发侦听器的执行
- 访问侦听状态变化前后的值

#### 4.7 侦听单个数据源

watch侦听函数的数据源有两种类型：

- 一个getter函数：但是该getter函数必须引用可响应式的对象（比如reactive或者ref）

  ```js
  import {watch, reactive} from 'vue';
  
  const state = reactive({
      name: "why",
      age: 18
  })
  //监听具体某个对象属性
  watch(() => state.name, (newValue, oldValue) => {
      console.log(newValue, oldValue);
  })
  ```

- 直接写入一个可响应式的对象，reactive或者ref（比较常用的是ref）

  ```js
  //监听reactive对象
  const state = reactive({
      name: "why",
      age: 18
  })
  watch(state, (newValue, oldValue) => {
      console.log(newValue, oldValue); //newValue、oldValue均为reactive对象
  })
  
  //监听reactive对象返回普通对象
  const state = reactive({
      name: "why",
      age: 18
  })
  watch(() => {
      return {...state}; //传入getter将reactive对象解构
  }, (newValue, oldValue) => {
      console.log(newValue, oldValue); //此时newValue、oldValue均为普通对象
  })
  
  //监听ref对象，oldValue与newValue均为普通值
  const name = ref("why");
  watch(name, (newValue, oldValue) => {
      console.log(newValue, oldValue) //均为string类型
  })
  ```

#### 4.8 侦听多个数据源

侦听器还可以使用数组同时侦听多个源：使用数组

```js
const name = ref("why");
const age = ref(18);

watch([name, age], (newValue, oldValue) => {
    console.log(newValue, oldValue); //newValue是[newName,newAge]的数组
})

//可以对结果数组进行解构
watch([name, age], ([newName, newAge], [olaName, oldAge]) => {
    console.log(newName, newAge, olaName, oldAge)
} 
```

如果我们希望侦听一个数组或者对象，那么可以使用一个getter函数，并且对reactive可响应对象进行解构

```js
const names = reactive(["abc", "cba", "nba"]);
watch(() => [...names], (newValue, oldValue) => {
	console.log(newValue, oldValue);
})
```

watch的选项

- 监听对象为reactive对象时本身是深度监听的，如果我们希望对reactive对象解构并且进行深层的侦听，那么依然需要设置 deep 为true

- 还可以可以传入 immediate 立即执行

  ```js
  const state = reactive({
      name: "why",
      age: 18,
      friends: {
          name: "kobe"
      }
  });
  watch(() => ({...state}), (newValue, oldValue) => {
      console.log(newValue, oldValue);
  },{
      deep: true,
      immediate: true
  });
  ```

  