### 一、koa简单使用

koa注册的中间件提供了两个参数：

- ctx：上下文（Context）对象；
  - koa并没有像express一样，将req和res分开，而是将它们作为ctx的属性；
  - ctx代表依次请求的上下文对象；
  - ctx.request：获取请求对象；
  - ctx.response：获取响应对象；
- next：本质上是一个dispatch，类似于之前的next；

```javascript
const Koa = require('koa') // 导出的是一个类

const app = new Koa()

app.use((ctx, next) => {
    console.log('middleware 01')
    next()
})

app.use((ctx, next) => {
    console.log('middleware 02')
    ctx.response.body = 'Hello World'
})

app.listen(8000, () => {
    console.log('服务器启动成功')
})
```

#### 1.1 koa中间件

koa通过创建的app对象，注册中间件只能通过use方法：

- Koa并没有提供methods的方式来注册中间件；
- 也没有提供path中间件来匹配路径；
- 而且一次use只能注册一个中间件函数，不支持连续注册

但是真实开发中如何将路径和method分离呢？

- 方式一：像原生`http`模块一样根据`ctx.request`手动来判断。如：

  ```javascript
  app.use((ctx, next) => {
      if(ctx.request.path === '/users') {
          if(ctx.request.method === 'POST') {
              ctx.response.body = 'Create User Success'
          } else {
              ctx.response.body = 'Users Lists'
          }
      } else {
          ctx.response.body = 'Other Request Response'
      }
  })
  ```

- 方式二：使用第三方路由中间件；

除此之外，koa的`ctx.response`也没有`end`方法，返回数据需要将数据放到`ctx.response.body`上，koa内部会在最后一个中间件函数结束后调用`end`方法并将`body`中的数据返回给客户端

#### 1.2 路由的使用

koa官方并没有提供路由的库，我们可以选择第三方库`koa-router`：`npm install koa-router`

- 我们可以先在`"./router"`文件夹下封装一个`user.js`的文件：

  ```javascript
  const Router = require('koa-router')
  
  const userRouter = new Router({prefix: '/users'}) // 匹配/users下的请求
  
  userRouter.get('/', (ctx, next) => { // 在koa-router中路由一次可以连续注册多个中间件函数
      ctx.response.body = 'user list'
  })
  
  userRouter.post('/', (ctx, next) => {
      ctx.response.body = 'create user info'
  })
  
  module.exports = userRouter
  ```

- 在app中将router.routes()注册为中间件：

  ```javascript
  const userRouter = require('./router/user.js')
  
  app.use(userRouter.routes())
  ```

- 注意：koa-router提供了allowedMethods，用于判断某一个method是否支持：

  ```javascript
  app.use(userRouter.routes())
  app.use(userRouter.allowedMethods())
  ```

  这个中间件会拦截对应路由路径下没有实现的请求：

  - 如果请求get，那么是正常的请求，因为路由中有实现get；
  - 如果请求put、delete、patch，那么就自动报错：Method Not Allowed，状态码：405；
  - 如果请求link、copy、lock，那么就自动报错：Not Implemented，状态码：501；

#### 1.3 参数解析：params -query

在原生的koa中间件中，只能获取`query`参数而不能获取`params`参数：

请求地址http://localhost:8000/users/123?name=why&age=18

```javascript
app.use((ctx, next) => {
    console.log(ctx.request.url) // /users/123?name=why&age=18
    console.log(ctx.request.query)// {name: 'why', age: '18'}
    console.log(ctx.request.params)// undefined
})
```

只有在`koa-router`中的路由中间件才能同时获取`query`和`params`：

```javascript
const Router = require('koa-router')

const userRouter = new Router({prefix: '/users'})

userRouter.get('/:id', (ctx, next) => {
    console.log(ctx.request.url) // /users/123?name=why&age=18
    console.log(ctx.request.query)// {name: 'why', age: '18'}
    console.log(ctx.request.params)// {id: '123'}
})

app.use(userRouter.routes())
```

#### 1.4 参数解析：json、urlencoded、form-data

koa原生并不提供这三种数据的解析，需要依赖第三方库`koa-bodyparser`：`npm install koa-bodyparser`

- json解析：

  body是json格式：

  ```json
  {
      'username': 'coderwhy',
      'password': '123'
  }
  ```

  ```javascript
  const bodyParser = require('koa-bodyparser')
  
  app.use(bodyParser())
  
  app.use((ctx, next) => {
      console.log(ctx.request.body) // {username: 'coderwhy', password: '123'}
  })
  ```

- urlencoded解析：与json解析一样，先将`bodyParser()`返回的函数注册到中间件进行处理，结果可在下一个中间件的`ctx.request.body`中获取：`{username: 'coderwhy', password: '123'}`

- form-data(无文件，仅数据上传)：解析form-data中的数据，需要使用与`express`中的`multer`相似的`koa-multer`：`npm install koa-multer`：

  body是form-data格式：

  ![formdata类型数据](https://gitee.com/Topcvan//img-storage/raw/master//node/formdata%E7%B1%BB%E5%9E%8B%E6%95%B0%E6%8D%AE.png)

  ```javascript
  const multer = require('multer')
  
  const upload = multer()
  
  app.use(upload.any())
  
  app.use((ctx, next) => {
      console.log(ctx.req.body) // {username: lilei, password: '8888'}
  })
  ```

  与`json`、`urlencoded`等形式的参数不同，`form-data`类型的数据需要在`ctx.req.body`中进行获取，`ctx.req`与`ctx.request`是两个不同的对象：`ctx.request`是`koa`定义的对象，而`ctx.req`等同于原生`http`中的`req`对象

#### 1.5 文件上传

对`form-data`上传的文件进行处理同样是利用`koa-multer`，如：

请求地址：`localhost:8080/upload/avatar`

```javascript
const Router = require('koa-router') // 由于文件上传都是在特定路径下进行的,所以此处用路由示例
const multer = require('koa-multer')

const uploadRouter = new Router({prefix: '/upload'})

const upload = multer({dest: './avatar'})

uploadRouter.post('/avatar', upload.single('avatar'), (ctx, next) => {
    console.log(ctx.req.file)
    ctx.response.body = '头像上传成功'
})

app.use(uploadRouter.routes())
```

`koa-multer`处理文件上传的步骤与`express`的`multer`类似，而且文件信息和`form-data`文本信息处理一样，都是存储在`ctx.req`对象上

#### 1.6 数据响应

`ctx.response`的body将响应主体设置为以下之一：

- string ：字符串数据
- Buffer ：Buffer数据
- Stream ：流数据
- Object|| Array：对象或者数组
- null ：不输出任何内容

如果response.status尚未设置，Koa会自动将状态设置为200或204(在body没有数据时)

```javascript
app.use((ctx, next) => {
    ctx.status = 200
    ctx.body = {   // 在发送时会转成JSON格式
        name: 'why',
        age: 18,
        height: 1.88
    }
})
```

在上面的代码中，将body和status直接设置到了ctx上而不是ctx.response上，但数据依然能正常返回。这是因为koa内部对ctx的某些属性做了代理，当在ctx设置这些属性时，会将这些属性写入到ctx.response上

#### 1.7 静态服务器

在koa中没有内置静态服务器功能，所以需要依赖第三方库`koa-static`：`npm i koa-static`

部署的过程类似于`express`：

```javascript
const path = require('path')

const staticAssets = require('koa-static')

app.use(staticAssets(path.resolve(__dirname, './build')))
```

#### 1.8 koa错误处理

koa错误处理一般按照以下流程：

1. 在某个中间件中发现错误，并将错误抛出：

   ```javascript
   app.use((ctx, next) => {
       let isLogin = false
       if (!isLogin) {
           ctx.app.emit('error', new Error('未登录'), ctx)
       }
   })
   ```

   在上面的代码中，发现错误后通过`ctx.app`发出错误事件，将`Error`对象和`ctx`传递给错误处理程序。由于真实开发中会将路由抽取到独立文件，此时在中间件中无法直接获取全局`app`对象，所以通过`ctx`来获取`app`对象，此处的`ctx.app`会被代理到全局`app`对象上。

2. 监听错误事件，注册错误处理中间件函数：

   ```javascript
   app.on('error', (err, ctx) => {
       ctx.status = 401;
       ctx.body = err.message
   })
   ```

### 二、koa源码解析

#### 2.1 导入的`koa`类

对koa进行导入的是一个类：

![koa类](https://gitee.com/Topcvan//img-storage/raw/master//node/koa%E7%B1%BB.png)

#### 2.2 进行监听

在调用`app.listen`时，执行的是以下函数：

![koaAppListen](https://gitee.com/Topcvan//img-storage/raw/master//node/koaAppListen.png)

#### 2.3 注册中间件

在调用`app.use`方法时，执行以下函数，将回调函数加入`middleware`数组中：

![koaAppUse](https://gitee.com/Topcvan//img-storage/raw/master//node/koaAppUse.png)

#### 2.4 监听回调

在`listen`方法中，koa调用`app.callback()`，将返会的函数传入`http.createServer`中，当接收到网络请求时会调用此方法。而调用`app.callback()`时，执行的是以下函数：

![koaAppCallback](https://gitee.com/Topcvan//img-storage/raw/master//node/koaAppCallback.png)

所以，在接收到网络请求时，调用的是`handleRequest`方法。

由于`handleRequest`是一个箭头函数，所以它内部的`this`会被绑定到上一层词法环境中的`this`，也就是`app`对象。在`handleRequest`中，将原生`http`模块中的`req, res`传入到`app.createContext`中，然后返回一个`ctx`对象，然后又调用`app.handleRequest`方法并传入`ctx`和`fn`。所以`app.use`中的`ctx`对象就是这个`ctx`。`fn`是将储存着中间件函数的`app.middleware`传入`compose`方法返回的对象

#### 2.5 `compose`方法

`callback`中的`compose`方法并不在`koa`源码中，而是在`koa`所依赖的第三方库`koa-compose`中，`compose`方法具体如下：

![compose](https://gitee.com/Topcvan//img-storage/raw/master//node/compose.png)

`compose`返回以下函数：

![compose返回值](https://gitee.com/Topcvan//img-storage/raw/master//node/compose%E8%BF%94%E5%9B%9E%E5%80%BC.png)

而`app.use`中的`next`就是上图中的`dispatch`

#### 2.6 `app.handleRequest`

在接收到网络请求时，会执行`handleRequest`函数，而这个函数又会调用`app.handleRequest`方法并将`ctx`对象和`compose`返回的`fn`作为参数传入。`app.handleRequest`方法具体如下：

![koaAppHandleRequest](https://gitee.com/Topcvan//img-storage/raw/master//node/koaAppHandleRequest.png)

上图中的`fnMiddleware`就是`compose`返回的对象。在`app.handleRequest`中会调用`fnMiddleware`并将`ctx`继续传入。

在`fnMiddleware`中：

![compose返回值](https://gitee.com/Topcvan//img-storage/raw/master//node/compose%E8%BF%94%E5%9B%9E%E5%80%BC.png)

会将`app.middleware`中的中间件函数取出执行，将`ctx`对象以及绑定`this`为`null`，`i`为当前`i+1`的`dispatch`函数传入，然后将中间件函数执行返回结果赋值到`Promise`中返回。

然后`app.handleRequest`将`handleResponse`函数推入微任务队列，在中间件函数执行完时会在微任务队列调用此函数，该函数会将`ctx`作为参数传入`respond`函数并调用，该方法会处理`body`中的结果，比如转化成`JSON`格式，最后调用`http`模块中的`res.end(body)`将`body`作为响应结果返回给客户端

### 三、express与koa的中间件执行顺序

express和koa框架的核心其实都是中间件：

但是事实上，它们的中间件的执行机制是不同的，特别是针对某个中间件中包含异步操作时；

案例实现：

假如有三个中间件会在一次请求中匹配到，并且按照顺序执行；

我希望最终实现的方案是：

- 在middleware1中，在req.message中添加一个字符串aaa；
- 在middleware2中，在req.message中添加一个字符串bbb；
- 在middleware3中，在req.message中添加一个字符串ccc；
- 当所有内容添加结束后，在middleware1中，通过res返回最终的结果；

实现方案：

Express同步数据的实现；

Express异步数据的实现；

Koa同步数据的实现；

Koa异步数据的实现；

#### 3.1 express同步实现

利用`express`同步中间件实现代码如下：

```javascript
const middleware1 = (req, res, next) => {
    req.message = 'aaa'
    next()
    res.end(req.message)
}

const middleware2 = (req, res, next) => {
    req.message += 'bbb'
    next()
}

const middleware3 = (req, res, next) => {
    req.message += 'ccc'
}

app.use(middleware1, middleware2, middleware3)
```

在`express`同步中间件中，以上需求可以实现，因为`express`中间件的执行过程是这样的：

![express中间件执行](https://gitee.com/Topcvan//img-storage/raw/master//node/express%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%89%A7%E8%A1%8C.png)

`express`内部调用`next`方法去栈中匹配中间件并调用，如果在该中间件中调用了`next()`，则会在栈中匹配下一个中间件并执行，该中间件的`req`与`res`对象与上一个中间件中的`req`、`res`对象相同，所以在`middleware3`中`+='ccc'`时会将所有数据拼接到一起。在`middleware3`执行完毕后后回退到`middleware`中，而`middleware`此时也执行完毕，所以回退到`middleware1`中，执行`res.end(req.message)`将所有数据返回

#### 3.2 express异步实现

但有时，数据并不能直接在本地进行获取，而是需要通过网络请求等手段异步获取。此时，如果使用异步中间件，则会缺失最后的异步请求数据，如：

![express异步中间件](https://gitee.com/Topcvan//img-storage/raw/master//node/express%E5%BC%82%E6%AD%A5%E4%B8%AD%E9%97%B4%E4%BB%B6.png)

在`middleware3`中，`axios`通过`get`方法发送网络请求后随即退出，中间件一直回退到`middleware1`中，然后`middleware1`执行`res.end(res.message)`将不完整的数据返回。在执行完这些同步代码后，才会进入微任务队列将网络请求返回的数据拼接到`req.message`中，而此时`res.end()`早已调用，所以这部分数据将会缺失，客户端接收到的响应数据只有`aaabbb`。

如果想要异步请求数据，只能将`middleware3`当作普通函数调用而不是中间件，如：

```javascript
const middleware1 = async (req, res, next) => {
    req.message = 'aaa'
    await middleware3(req)
    res.end(req.message)
}
```

#### 3.3 koa同步实现

在koa中，同步中间件实现如下：

```javascript
const Koa = require('koa')

const app = new Koa()

const middleware1 = (ctx, next) => {
    ctx.message = 'aaa'
    next()
    ctx.body = ctx.message 
}

const middleware2 = (ctx, next) => {
    ctx.message += 'bbb'
    next()
}

const middleware3 = (ctx, next) => {
    ctx.message += 'ccc'
}
```

最终，客户端可以成功接收到完整数据`aaabbbccc`，因为koa中的同步中间件执行顺序如下：

![koa同步实现](https://gitee.com/Topcvan//img-storage/raw/master//node/koa%E5%90%8C%E6%AD%A5%E5%AE%9E%E7%8E%B0.png)

在调用`next`方法时，实际上是调用`dispatch`去调用栈中下一个匹配的中间件函数，虽然中间件函数被包裹在`Promise.resolve()`中，但该函数还是会同步执行而不是加入微任务队列，中间件函数完成调用后会将其返回结果包装进`Promise`中并返回，所以koa的中间件中的`next()`返回的是一个`Promise`，其值是下一个中间件函数执行所返回的结果

#### 3.4 koa异步实现

如果koa像express一样书写异步中间件，那么同样会返回不完整的数据，如：

```javascript
const Koa = require('koa')

const app = new Koa()

const middleware1 = (ctx, next) => {
  ctx.message = "aaa";
  next();
  ctx.body = ctx.message;
}

const middleware2 = (ctx, next) => {
  ctx.message += "bbb";
  next();
}

const middleware3 = async (ctx, next) => {
  const result = await axios.get('http://123.207.32.32:9001/lyric?id=167876');
  ctx.message += result.data.lrc.lyric;
}
app.use(middleware1)
app.use(middleware2)
app.use(middleware3)
```

因为代码是同步调用的，所以`dispatch`不会等待`middleware`调用完成，而是直接返回一个`<pending>`的`Promise`作为`Promise.resolve`的参数并完成`dispatch`(也就是`next`)的调用，所以在进入微任务队列进行数据拼接前，响应数据就已经发出

如果确实需要通过异步中间件来完成需求，可以利用`async/await`来完成：

```javascript
const middleware1 = async (ctx, next) => {
  ctx.message = "aaa";
  await next();
  ctx.body = ctx.message;
}

const middleware2 = async (ctx, next) => {
  ctx.message += "bbb";
  await next();
}

const middleware3 = async (ctx, next) => {
  const result = await axios.get('http://123.207.32.32:9001/lyric?id=167876');
  ctx.message += result.data.lrc.lyric;
}
app.use(middleware1)
app.use(middleware2)
app.use(middleware3)
```

由于`next`返回的是`Promise`，所以在添加`await`后，会等待返回的`Promise`进入`Resolve`状态后，后续代码才会继续执行，所以在中间件的执行过程中，会像同步代码一样等待数据请求成功后再回退到上一个中间件中，最后执行`middleware1`中的`ctx.body = ctx.message`后，将`body`中的数据通过微任务队列中的`respond`函数将数据返回

#### 3.5 express异步中间件与koa异步中间件

以上需求，通过`koa`异步中间件可以完成，但是`express`异步中间件则不行，是因为`express`和`koa`中的`next`函数返回不同类型的值：

- express的`next`函数接到调用请求，会到栈中匹配封装了中间件函数的`layer`，在匹配到`layer`后会执行`layer.handleRequest(res, req, next)`并将返回值进一步返回作为`next()`的返回值，而这个返回值并不是`Promise`(即使将中间件设置为`async`函数，因为`layer.handleRequest()`的返回值并不是对应`middleware`的返回值)。

  `express`的中间件执行过程与如下代码类似：

  ```javascript
  const fn2 = async () => {
      await new Promise(resolve => {
          setTimeout(resolve, 1000)
      })
      console.log('完成异步请求', Date.now())
  }
  const next1 = () => {
      fn2()
      return console.log('sync next1 end', Date.now())
  }
  
  const fn1 = async () => {
      await next1()
      console.log('async fn1 end', Date.now())
  }
  const next = () => {
      fn1()
      return console.log('request end', Date.now())
  }
  next()
  ```

  其中`async`函数是开发者编写的中间件回调函数，而普通函数是`next`，在以上代码中，`async`函数并没有`await`进行阻塞，所以在执行`async`函数直到`await`进行阻塞后会让出线程，控制权回到普通函数中执行同步代码并完成调用；

  而`await`对普通函数进行阻塞只会将剩下的代码立即推进异步队列然后进入微任务队列执行。所以在异步中间件请求到数据并将异步函数推入微任务队列时，前面的中间件函数早已进入微任务队列并完成调用了，所以异步请求的数据并不能发送给客户端

  所以`express`需要异步执行的函数不能作为中间件使用，而是作为中间件函数中的异步调用，唯一允许作为中间件调用的异步函数就是第一个匹配的中间件，如：

  ```javascript
  const fn2 = () => {
      // 同步数据拼接
  }
  const next1 = () => {
      fn2()
      return console.log('中间件2完成')
  }
  const fn1 = async () => {
      // 同步数据拼接
      next1()
      await asyncFn() // 最后的异步数据拼接
      // 发送响应
  }
  const next = () => {
      fn1()
      return console.log('中间件1完成')
  }
  ```

- 而在koa中的中间件可以通过`async/await`完成需求，因为koa中的`next`返回的是以中间件调用返回值为`[[PromiseResult]]`的`Promise`，所以添加`await`会阻塞，直至`Promise`进入`<resolved>`状态，而这个`Promise`进入`<resolved>`状态当且仅当`async middleware`返回的`Promise`进入`<resolve>`状态；如此层层阻塞，就能等待异步数据请求完成后再进行拼接并返回给客户端
