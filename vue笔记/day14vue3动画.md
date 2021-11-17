### 一、vue3动画实现

在开发中，我们想要给一个组件的显示和消失添加某种过渡动画，可以很好的增加用户体验，Vue中为我们提供一些内置组件和对应的API来完成动画，利用它们我们可以方便的实现过渡动画效果

#### 1.1 transition动画

- Vue 提供了transition 的封装组件，在下列情形中，可以给任何元素和组件添加进入/离开过渡：

  - 条件渲染(使用v-if)条件展示(使用v-show)

  - 动态组件

  - 组件根节点

    ```vue
    <transition name="fade">
    	<h2 v-if="show">
            Hello World
        </h2>
    </transition>
    
    <style scoped>
    	.fade-enter-from,
        .fade-leave-to {
            opacity: 0;
        }
        
        .fade-enter-to,
        .fade-leave-from {
            opacity: 1;
        }
        
        .fade-enter-active,
        .fade-leave-active {
            transition: opacity 1s ease;
        }
    </style>
    ```

#### 1.2 transition组件原理

- 我们会发现，Vue自动给h2元素添加了动画，这是什么原因呢
- 当插入或删除包含在transition 组件中的元素时，Vue 将会做以下处理：
  - 自动嗅探目标元素是否应用了CSS过渡或者动画，如果有，那么在恰当的时机添加/删除CSS类名
  - 如果transition 组件提供了JavaScript钩子函数，这些钩子函数将在恰当的时机被调用
  - 如果没有找到JavaScript钩子并且也没有检测到CSS过渡/动画，DOM插入、删除操作将会立即执行

#### 1.3 过渡动画class

- 我们会发现上面提到了很多个class，事实上Vue就是帮助我们在这些class之间来回切换完成的动画：
  - v-enter-from：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除
  - v-enter-active：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数
  - v-enter-to：定义进入过渡的结束状态。在元素被插入之后下一帧生效(与此同时v-enter-from 被移除)，在过渡/动画完成之后移除
  - v-leave-from：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除
  - v-leave-active：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数
  - v-leave-to：离开过渡的结束状态。在离开过渡被触发之后下一帧生效(与此同时v-leave-from 被删除)，在过渡/动画完成之后移除
- class的name命名规则如下：
  - 如果我们使用的是一个没有name的transition，那么所有的class是以v-作为默认前缀
  - 如果我们添加了一个name属性，比如`<transtionname="why">`，那么所有的class会以why-开头

#### 1.4 过渡css动画

- 前面我们是通过transition来实现的动画效果，另外我们也可以通过animation来实现

  ```vue
  .bounce-enter-active {
  	animation: bounce-in 0.5s;
  }
  
  .bounce-leave-active {
  	animation: bounce-in 0.5s reverse;
  }
  
  @keyframes bounce-in {
  	0%: {
  		transform: scale(0);
  	}
  	50% {
  		transform: scale(1.2);
  	}
  	100% {
  		transform: scale(1);
  	}
  }
  ```

- 同时设置过渡和动画

  - Vue为了知道过渡的完成，内部是在监听transitionend或animationend，到底使用哪一个取决于元素应用的CSS规则：如果我们只是使用了其中的一个，那么Vue能自动识别类型并设置监听

  - 但是如果我们同时使用了过渡和动画呢，并且在这个情况下可能某一个动画执行结束时，另外一个动画还没有结束

    - 在这种情况下，我们可以设置type 属性为animation 或者transition来明确的告知Vue监听的类型

      ```
      <transition name="why"
      			type="tramsition">
      	<h2 v-if="show">Hello World</h2>
      </transition>
      ```

- 显示的指定动画时间

  - 我们也可以显示的来指定过渡的时间，通过duration 属性

  - duration可以设置两种类型的值：

    - number类型：同时设置进入和离开的过渡时间

    - object类型：分别设置进入和离开的过渡时间

      ```vue
      //number
      <transition name="why"
                  type="transition"
                  :duration="1000">
      	<h2>
              Hello World
          </h2>
      </transition>
      //object
      <transition name="why"
                  type="transition"
                  :duration="{enter: 800, leave: 1000}">
      	<h2>
              Hello World
          </h2>
      </transition>
      ```

- 过渡的模式mode

  - 动画在两个元素之间切换的时候两个元素是同时存在的，因为默认情况下进入和离开动画是同时发生的

  - 如果我们不希望同时执行进入和离开动画，那么我们需要设置transition的过渡模式：

    - in-out: 新元素先进行过渡，完成之后当前元素过渡离开

    - out-in: 当前元素先进行过渡，完成之后新元素过渡进入

    - mode属性适用于动态组件

      ```vue
      <div>
          <button @click="show = !show">
              Toggle
          </button>
      </div>
      
      <transition name="why" appear mode="out-in">
      	<component :is=""></component>
      </transition>
      ```

      

- appear初次渲染

  - 默认情况下，首次渲染的时候是没有动画的，如果我们希望给他添加上去动画，那么就可以增加另外一个属性appear：

    ```vue
    <div>
        <button @click="show = !show">
            Toggle
        </button>
    </div>
    
    <transition name="why" appear>
    	<component :is="show ? 'home' : 'about'"></component>
    </transition>
    ```

#### 1.5 认识animate.css

在开发中我们可能会引用一些第三方库的动画库，如果我们手动一个个来编写这些动画，那么效率是比较低的，比如animate.css

- 安装与使用

  - 安装：`npm install animate.css`

  - 在main.js中导入animate.css：`import "animate.css"`

  - 接下来在使用的时候我们有两种用法：

    - 用法一：直接使用animate库中定义的keyframes 动画

      ```vue
      //flip为库中定义的动画
      .why-enter-active {
      	animation: flip 1s;
      }
      .why-leave-active {
      	animation: flip 1s reverse;
      }
      ```

    - 用法二：直接使用animate库提供给我们的类

      ```vue
      <transition name="why" //animate__animated为必加前缀
                  enter-active-class="animate__animated animate__lightSpeedInRight"
                  leave-active-class="animate__animated animate__lightSpeedOutRight">
      	<h2 v-if="show">
              Hello World
          </h2>
      </transition>
      //如果还需要对动画进行自定义设置，比如方向反转，可以在style中进行设置
      <style>
          .animate_lightSpeedOutRight {
              aimation-diretio
          }
      </style>
      ```

      