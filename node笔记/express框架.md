### 一、express简单使用

Express整个框架的核心就是中间件，理解了中间件其他一切都非常简单

#### 1.1 express安装

express的使用过程有两种方式：

- 方式一：通过express提供的脚手架，直接创建一个应用的骨架；

  1. 安装脚手架
     npm install -g express-generator
  2. 创建项目
     express express-demo
  3. 进入项目目录安装依赖
     npm install
  4. 启动项目
     node bin/www

- 方式二：从零搭建自己的express应用结构；
  npm init -y

  npm install express

#### 1.2 express基本使用

我们来创建第一个express项目：

- 我们会发现，之后的开发过程中，可以方便的将请求进行分离：
- 无论是不同的URL，还是get、post等请求方式；
- 这样的方式非常方便开发者进行维护、扩展；

请求的路径中如果有一些参数，可以这样表达：

- /users/:userId；
- 在request对象中要获取可以通过req.params.userId;

返回数据，可以方便的使用json：

- res.json(数据)方式；
- 可以支持其他的方式，可以自行查看文档：https://www.expressjs.com.cn/guide/routing.html

```javascript
const express = require('express')

const app = express()

app.get('/home', (req, res) => {
    res.end('hello home')
})

app.post('/login', (req, res) => {
    res.end('hello login')
})

app.listen(8000, () => {
    console.log('服务器启动成功')
})
```

#### 1.3 中间件使用

Express是一个路由和中间件的Web框架，它本身的功能非常少：Express应用程序本质上是一系列中间件函数的调用；

中间件是什么呢？

- 中间件的本质是传递给express的一个回调函数；
- 这个回调函数接受三个参数：
  1. 请求对象（request对象）；
  2. 响应对象（response对象）；
  3. next函数（在express中定义的用于执行下一个中间件的函数）；

认识中间件

中间件中可以执行哪些任务呢？

- 执行任何代码；
- 更改请求（request）和响应（response）对象；
- 结束请求-响应周期（返回数据）；
- 调用栈中的下一个中间件；

如果当前中间件功能没有结束请求-响应周期，则必须调用next()将控制权传递给下一个中间件功能，否则，请求将被挂起。

中间件调用函数组成：

![中间件函数调用组成](https://gitee.com/Topcvan//img-storage/raw/master//node/%E4%B8%AD%E9%97%B4%E4%BB%B6%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E7%BB%84%E6%88%90.png)

**编写中间件**

那么，如何将一个中间件应用到我们的应用程序中呢？

- express主要提供了两种方式：app/router.use和app/router.methods；
- 可以是app，也可以是router
- methods指的是常用的请求方式，比如：app.get或app.post等；

先来学习use的用法，因为methods的方式本质是use的特殊情况；

案例一：最普通的中间件

```javascript
app.use((req, res, next) => {
    res.end('hello common middleware')
})
```

这个中间件会匹配所有请求方法和路径

案例二：path匹配中间件

```javascript
app.use('/user', (req, res, next) => {
    res.end('hello path middleware')
})
```

这个中间件会匹配`/user`路径下的所有请求

案例三：path和method匹配中间件

```javascript
app.get('/', (req, res, next) => {
    res.end('hello get middleware') // 这个中间件会匹配根路径下所有get请求
})

app.get('/login', (req, res, next) => {
    res.end('hello get & path middleware') // 这个中间件会匹配login路径下所有get请求
})
```

案例四：注册多个中间件

```javascript
app.use((req, res, next) => {
    console.log('hello common middleware')
    next()
})

app.use('/user', (req, res, next) => {
    console.log('hello path middleware')
    next()
})

app.get('/user', (req, res, next) => {
    console.log('hello get and path middleware')
    res.end('hello express middleware')
})
```

在处理网络请求时，会调用第一个匹配的中间件函数，如果不显式调用`res.end`或者`next`，请求就会一直阻塞在这个处理函数中，不会返回结果或者移交控制权，调用栈后面的匹配的中间件函数也不会调用。此时，如果调用`next()`方法，就会进入到下一个匹配的中间件函数

`app.use`和`app.methods`都可以传入多个回调，这些回调与调用多个单回调的`app.use`和`app.methods`一样，都需要调用`next()`方法才能获得控制权

#### 1.4 中间件应用

在客户端发送post请求时，会将数据放到body中：客户端可以通过json的方式传递；也可以通过form表单的方式传递；如果在每个路径和方法都写对应的body解析方法就太复杂了，可以写在一个统一的拦截中间件中，如：

```javascript
app.use((req, res, next) => {
    if (req.headers['Content-Type'] === 'application/json') {
        req.on('data', data => {
            const userInfo = JSON.parse(data.toString())
            req.body = userInfo
        })
        req.on('end', () => {
            next()
        })
    } else {
        next()
    }
})
```

但是，事实上对body的解析可以使用expres内置的中间件或者使用body-parser来完成：

```javascript
app.use(express.json()) // express.json()返回的就是对JSON数据做解析的中间件函数

app.post('/login', (req, res, next) => {
    console.log(req.body)
    res.end('登录成功')
})
```

如果客户端上传的是application/x-www-form-urlencoded数据：

```javascript
app.use(express.urlencoded({extended: true}))

app.post('/login', (req, res, next) => {
    console.log(req.body)
    req.end('登录成功')
})
```

### 二、form-data解析

form-data一般用于上传文件。处理上传文件，可以使用express的第三方库multer来完成：

multer安装：`npm install multer`

使用：

```javascript
const multer = require('multer')

const upload = multer()
```

#### 2.1 multer函数参数配置

`multer`函数可接收一个包含多个参数的对象进行配置：

```javascript
{
    dest?: 文件储存路径 // 或者可以传入一个storage对象代替dest选项,
    fileFilter?: 文件过滤函数,
    limits?: 限制上传的数据的对象,
    preservePath?: 保存包含文件名的完整文件路径
}
```

如果`form-data`中没有文件，可以传递该对象作为参数

#### 2.2 upload中间件方法

upload对象中有多种中间件函数用以处理不同类型的`form-data`：

- `.single(fieldname)`，接受一个字段为`filedname`的文件，文件信息保存在`req.file`：

  ```javascript
  app.post('/upload', upload.single('avatar'), (req, res, next) => {
      console.log(req.file)
  })
  ```

- `.array(fieldname[, maxCount])`，接受一个字段为`filedname`的文件数组，可以设置`maxCount`来限制数量，文件信息保存在`req.files`中：

  ```javascript
  app.post('/upload', upload.array('books', 3), (req, res, next) => {
      console.log(req.files)
  })
  ```

- `.fields(fields)`，接受指定 `fields` 的混合文件。这些文件的信息保存在 `req.files`。`fields` 应该是一个对象数组，应该具有 `name` 和可选的 `maxCount` 属性：

  ```javascript
  [
    { name: 'avatar', maxCount: 1 },
    { name: 'gallery', maxCount: 8 }
  ]
  ```

  此时`req.files`是一个对象，不同字段的文件信息可通过字段名作为属性获取：

  ```javascript
  (req, res, next) => {
      console.log(req.files['avatar']) // 包含avatar文件信息的数组
      console.log(req.files['gallery']) // 包含gallery文件信息的数组
  }
  ```

- `.none()`，只接受文本域。如果任何文件上传到这个模式，将发生 `LIMIT_UNEXPECTED_FILE` 错误。这和 `upload.fields([])` 的效果一样。

- `.any()`，接受一切上传的文件。文件信息数组将保存在 `req.files`。

 永远不要将 multer 作为全局中间件使用，因为恶意用户可以上传文件到一个你没有预料到的路由，应该只在需要处理上传文件的路由上使用。

#### 2.3 storage对象配置

在配置`multer`函数是如果传入的是`dest`，那么multer会创建对应文件夹(如果不存在)并随机为文件生成名字并且没有拓展名。如果需要对文件储存路径以及文件名做自定义设置，可使用storage对象。

storage对象可分为两种：

- `DiskStorage`，磁盘存储引擎可以控制文件在磁盘进行存储：

  ```javascript
  const path = require('path')
  
  const storage = multer.diskStorage({
      destination: function(req, file, cb) {
          cb(null, '/tmp/my-uploads')
      },
      filename: function (req, file, cb) {
          cb(null, Date.now() + path.extname(file.originalname))
      }
  })
  
  const upload = multer({
      storage: storage
  })
  ```

  有两个选项可用，`destination` 和 `filename`。它们都是用来确定文件存储位置的函数。`destination` 是用来确定上传的文件应该存储在哪个文件夹中。也可以提供一个 `string` (例如 `'/tmp/uploads'`)。如果没有设置 `destination`，则使用操作系统默认的临时文件夹。

  **注意:** 如果提供的 `destination` 是一个函数，你需要负责创建文件夹。当提供一个字符串，multer在文件夹不存在时会创建对应的文件夹。

  `filename` 用于确定文件夹中的文件名的确定。 如果没有设置 `filename`，每个文件将设置为一个随机文件名，并且是没有扩展名的。

  **注意:** Multer 不会为你添加任何扩展名，你的程序应该返回一个完整的文件名。

  每个函数都传递了请求对象 (`req`) 和一些关于这个文件的信息 (`file`)，有助于你的决定。

  注意 `req.body` 可能还没有完全填充，这取决于向客户端发送字段和文件到服务器的顺序。
  
- `MemoryStorage`，内存存储引擎将文件存储在内存中的 `Buffer` 对象，它没有任何选项。

  ```javascript
  const storage = multer.memoryStorage()
  const upload = multer({ storage: storage })
  ```

  当使用内存存储引擎，文件信息将包含一个 `buffer` 字段，里面包含了整个文件数据。

  **警告**: 当使用内存存储，上传非常大的文件，或者非常多的小文件，会导致应用程序内存溢出。

#### 2.4 multer函数的其他参数

- `limits`，

- 一个对象，指定一些数据大小的限制。Multer 通过这个对象使用 busboy，详细的特性可以在 [busboy's page](https://github.com/mscdex/busboy#busboy-methods) 找到。

  可以使用下面这些:

  | Key             | Description                                              | Default   |
  | --------------- | -------------------------------------------------------- | --------- |
  | `fieldNameSize` | field 名字最大长度                                       | 100 bytes |
  | `fieldSize`     | field 值的最大长度                                       | 1MB       |
  | `fields`        | 非文件 field 的最大数量                                  | 无限      |
  | `fileSize`      | 在 multipart 表单中，文件最大长度 (字节单位)             | 无限      |
  | `files`         | 在 multipart 表单中，文件最大数量                        | 无限      |
  | `parts`         | 在 multipart 表单中，part 传输的最大数量(fields + files) | 无限      |
  | `headerPairs`   | 在 multipart 表单中，键值对最大组数                      | 2000      |

  设置 limits 可以帮助保护站点抵御拒绝服务 (DoS) 攻击。

- `fileFilter`

  设置一个函数来控制什么文件可以上传以及什么文件应该跳过，这个函数应该看起来像这样：

  ```javascript
  function fileFilter (req, file, cb) {
  
    // 这个函数应该调用 `cb` 用boolean值来
    // 指示是否应接受该文件
  
    // 拒绝这个文件，使用`false`，像这样:
    cb(null, false)
  
    // 接受这个文件，使用`true`，像这样:
    cb(null, true)
  
    // 如果有问题，你可以总是这样发送一个错误:
    cb(new Error('I don\'t have a clue!'))
  
  }
  ```

#### 2.5 错误处理机制

当遇到一个错误，multer 将会把错误发送给 express。你可以使用一个比较好的错误展示页 ([express标准方式](http://expressjs.com/guide/error-handling.html))。

如果想要捕捉 multer 发出的错误，可以自己调用中间件程序。如果想捕捉 [Multer 错误](https://github.com/expressjs/multer/blob/master/lib/multer-error.js)，可以使用 `multer` 对象下的 `MulterError` 类 (即 `err instanceof multer.MulterError`)。

```javascript
const multer = require('multer')
const upload = multer().single('avatar')

app.post('/profile', function (req, res) {
  upload(req, res, function (err) {
    if (err instanceof multer.MulterError) {
      // 发生错误
    } else if (err) {
      // 发生错误
    }

    // 一切都好
  })
})
```

### 三、日志、参数、路由、静态服务器与错误处理

#### 3.1 输出日志

如果我们希望将请求日志记录下来，那么可以使用express官网开发的第三方库：morgan

安装：`npm install morgan`

使用：

```javascript
const fs = require('fs')

const morgan = require('morgan')

const loggerWriter = fs.createWriteStream('./log/access.log', {
    flags: 'a+'
})

app.use(morgan('combined', {stream: loggerWriter}))
```

这样每次请求都会被记录到`./log/access.log`文件中

#### 3.2 GET请求参数传递

客户端通过`GET`方法传递到服务器参数的方法常见的是2种：

- 方式一：通过get请求中的URL的params；

  请求地址：http://localhost:8000/login/abc/why

  获取参数：

  ```javascript
  app.get('/login/:id/:name', (req, res, next) => {
      console.log(req.params) // 以id和name为key的对象
      res.end('请求成功')
  })
  ```

- 方式二：通过get请求中的URL的query

  请求地址：http://localhost:8000/login?username=why&password=123

  获取参数：

  ```javascript
  app.get('/login', (req, res, next) => {
      console.log(req.query) // 以username和password为key的对象
      res.end('请求成功')
  })
  ```

#### 3.3 响应数据

除了使用`res.end`来返回数据和关闭请求外，还有一些方法可用：

- json方法：json方法中可以传入很多的类型：object、array、string、boolean、number、null等；`json`方法会将传入的数据转化成`JSON`格式，并且设置对应的`Content-Type`，然后关闭请求。如：

  ```javascript
  app.use((req, res, next) => {
      res.json([{name: 'abc', age: 18}, {name: 'why', age: 19}])
      next()
  })
  ```

- status方法：用于设置状态码

  ```javascript
  app.use((req, res, next) => {
      res.status(200)
      res.json([{name: 'abc', age: 18}, {name: 'why', age: 19}])
      next()
  })
  ```

- type方法：将`Content-Type`HTTP 标头设置为由指定的`type`.；如果`type`包含“/”字符，则将设置`Content-Type`为确切值，否则假定为文件扩展名并使用`express.static.mime.lookup()`方法在映射中查找对应的 MIME 类型。

  ```javascript
  app.get('user', (req, res, next) => {
      res.type('json')
      res.status(200)
      const user = {name: 'why', age: 18}
      res.end(JSON.stringify(user))
  })
  ```

#### 3.4 路由

如果将所有的代码逻辑都写在app中，那么app会变得越来越复杂：

- 一方面完整的Web服务器包含非常多的处理逻辑；
- 另一方面有些处理逻辑其实是一个整体，应该将它们放在一起：比如对users相关的处理
  - 获取用户列表；
  - 获取某一个用户信息；
  - 创建一个新的用户；
  - 删除一个用户；
  - 更新一个用户；
- 可以使用express.Router来创建一个路由处理程序：
  - 一个Router实例拥有完整的中间件和路由系统；
  - 因此，它也被称为迷你应用程序（mini-app）；

如：

```javascript
// index.js
const express = require('express')

const userRouter = require('./routers/user')

const app = express()

app.use('/users', userRouter)
```

```javascript
// ./routers/user.js
const express = require('express')

const userRouter = express.Router()

userRouter.get('/', (req, res, next) => {
    res.end('用户列表')
})

userRouter.post('/', (req, res, next) => {
    res.end('创建用户')
})

userRouter.delete('/', (req, res, next) => {
    res.end('删除用户')
})

module.exports = userRouter
```

#### 3.5 静态资源服务器

部署静态资源我们可以选择很多方式：Node也可以作为静态资源服务器，并且express给我们提供了方便部署静态资源的方法。以下是部署一个打包完成的`react`项目示例：

```javascript
const path = require('path')

const express = require('express')

const app = express()

app.use(express.static(path.resolve(__dirname, '/bulid'))) // bulid目录是react项目的根目录

app.listen(8000, () => {
    console.log('服务器启动成功')
})
```

#### 3.6 错误处理

向`next()`函数传递任何内容（`'route'`字符串除外），Express会将当前请求视为错误，并将跳过任何剩余的非错误处理路由和中间件函数，所以错误处理函数一般放到最后进行注册，如：

```javascript
app.get((req, res, next) => {
    next(new Error('user already exists'))
})

app.use((err, req, res, next) => {
    const message = err.message
    if (message === 'user already exists') {
        res.status(400).json({message})
    }
})
```

### 四、express源码解析

#### 4.1 express函数与listen

通过`require('express')`导入的express函数的本质其实是createApplication：

![createApplication](https://gitee.com/Topcvan//img-storage/raw/master//node/createApplication.png)

`express()`返回的`app`对象其实就是在原生`http`模块的`http.createServer(cb)`中的回调函数，在图中，通过`mixin`的方式将`listen`等方法挂载到`app`上：

![appListen](https://gitee.com/Topcvan//img-storage/raw/master//node/appListen.png)

#### 4.2 注册中间件

比如我们通过use来注册一个中间件，源码中发生了什么？

- 我们会发现无论是app.use还是app.methods都会注册一个主路由；
- app本质上会将所有的函数，交给这个主路由去处理的

`app.use`：

![appUse](https://gitee.com/Topcvan//img-storage/raw/master//node/appUse.png)

`lazyrouter`方法是在`app`对象上创建`_router`对象

`router.user`：

![routerUse](https://gitee.com/Topcvan//img-storage/raw/master//node/routerUse.png)

`router.use`将中间件函数包装到`layer`对象中，然后压入栈中

#### 4.3 请求的处理过程

如果有一个请求过来，是从`app`函数被调用开始的：

![app函数被调用](https://gitee.com/Topcvan//img-storage/raw/master//node/app%E5%87%BD%E6%95%B0%E8%A2%AB%E8%B0%83%E7%94%A8.png)

在请求到达时，会调用`http.createServer(cb)`中的回调函数，在`express`中，传给`http.createServer`的就是`app`函数，在`app`函数中会调用`app.handle`方法并将参数传入

`app.handle`：

![apphandle](https://gitee.com/Topcvan//img-storage/raw/master//node/apphandle.png)

而在`app.handle`中会调用`router.handle`进行处理并将参数传入

#### 4.4 `router.handle`

`router.handle`：

![routerHandle](https://gitee.com/Topcvan//img-storage/raw/master//node/routerHandle.png)

在`router.handle`方法中，会将储存着中间件回调函数的栈取出，并执行`next`方法：

![routerNext](https://gitee.com/Topcvan//img-storage/raw/master//node/routerNext.png)

在`next`方法中会取出栈中函数并进行匹配，如果匹配则进行调用，如果中间件函数调用过程中执行了`next`方法，则继续将栈中函数弹出进行匹配并调用