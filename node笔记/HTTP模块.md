### 一、创建服务器

创建服务器有两种方式：

1. 通过`http.createServer`创建

   ```javascript
   const http = require('http')
   
   const server = http.createServer((req, res) => {
       res.end('hello world')
   })
   server.listen(8000, () => {
       console.log('服务器正在监听8000端口')
   })
   ```

2. 通过`new http.Server`创建

   ```javascript
   const server = new http.Server((req, res) => {
       res.end('hello world')
   })
   
   server.listen(8001, () => {
       console.log('服务器正在监听8001端口')
   })
   ```

#### 1.1 监听主机和端口号

Server通过listen方法来开启服务器，并且在某一个主机和端口上监听网络请求：

- 也就是当我们通过ip:port的方式发送请求到监听的Web服务器上时；就可以对其进行相关的处理；

listen函数有三个参数：

- 端口port: 可以不传, 系统会默认分配端, 后续项目会写入到环境变量中。如果不传，可以通过`server.address().port`获取分配的端口号；

- 主机host: 通常可以传入`localhost`、`ip`地址`127.0.0.1`、或者`ip`地址`0.0.0.0`。默认是0.0.0.0

  - localhost：本质上是一个域名，通常情况下会被解析成127.0.0.1；

  - 127.0.0.1：回环地址（Loop Back Address），表达的意思其实是我们主机自己发出去的包，直接被自己接收；

    正常的数据库包经常应用层-传输层-网络层-数据链路层-物理层；而回环地址，是在网络层直接就被获取到了，是不会经常数据链路层和物理层的；比如监听127.0.0.1时，在同一个网段下的主机中，通过`ip`地址是不能访问的；

  - 0.0.0.0：

    监听IPV4上所有的地址，再根据端口找到不同的应用程序；比如监听0.0.0.0时，在同一个网段下的主机中，通过ip地址是可以访问的；

- 回调函数：服务器启动成功时的回调函数；

#### 1.2 res对象

在向服务器发送请求时，会携带很多信息，比如：

- 本次请求的URL，服务器需要根据不同的URL进行不同的处理；
- 本次请求的请求方式，比如GET、POST请求传入的参数和处理的方式是不同的；
- 本次请求的headers中也会携带一些信息，比如客户端信息、接受数据的格式、支持的编码格式等；
- ......

这些信息，Node会封装到一个request的对象中，可以直接来处理这个request对象：

```javascript
const server = http.createServer((req, res) => {
    console.log(req.url)
    console.log(req.method)
    console.log(req.headers)
    
    res.end('hello world')
})
```

#### 1.3 URL的处理

客户端在发送请求时，会请求不同的数据，那么会传入不同的请求地址：

- 比如http://localhost:8000/login；
- 比如http://localhost:8000/products;

服务器端需要根据不同的请求地址，作出不同的响应：

```javascript
const server = http.createServer((req, res) => {
    const url = req.url
    console.log(url)
    
    if (url === '/login') {
        res.end('登录页面')
    } else if (url === '/products') {
        res.end('商品页面')
    } else {
        res.end('error message')
    }
})
```

#### 1.4 URL解析

那么如果用户发送的地址中还携带一些额外的参数呢？如：http://localhost:8000/login?name=why&password=123;这个时候，`url`的值是/login?name=why&password=123；

此时手动对`url`进行解析判断会十分复杂，所以需要使用内置`URL`类进行解析：

```javascript
const url = require('url')

const server = http.createServer((req, res) => {
   const parseInfo = url.parse(req.url)
   
   console.log(parseInfo)
})
```

`url.parse`方法返回对`url`进行解析后的对象，具体如下：

![url解析对象](https://gitee.com/Topcvan//img-storage/raw/master//node/url%E8%A7%A3%E6%9E%90%E5%AF%B9%E8%B1%A1.png)

但返回的`parseInfo`对象的`query`是一整串的查询字符串，手动解析还是十分复杂。所以，需要使用内置模块`qs`对`queryString`进行解析。如：

```javascript
const qs = require('querystring')

const { pathname, query } = url.parse(req.url)

const queryObj = qs.parse(query)
```

`qs.parse`方法会返回解析后的查询字符串，字符串的`key`作为对象的`key`，`value`作为对象对应属性的`value`

#### 1.5 method处理

在Restful规范（设计风格）中，对于数据的增删改查应该通过不同的请求方式：

- GET：查询数据；
- POST：新建数据；
- PATCH：更新数据；
- DELETE：删除数据；

所以，可以通过判断不同的请求方式进行不同的处理。比如创建一个用户：

- 请求接口为/users；
- 请求方式为POST请求；
- 携带数据`username`和`password`

判断请求方法并解析`body`中的数据：

```javascript
const server = http.createServer((req, res) => {
    const { pathname } = url.parse(req.url)
    
    if (pathname === '/register') {
        if (req.method === 'POST') {
            req.setEncoding('utf-8') // 设置body传过来的数据解码格式
            req.on('data', data => { // body不能直接通过req.body获取,需要监听data事件获取Stream
                const { username, password } = JSON.parse(data) // 将String转换成对象
                console.log(username, password)
            })
            req.on('end', () => { // 监听body传输结束事件
                console.log('传输完成')
                req.end('create user success')
            })
        }
    }
})
```

#### 1.6 headers属性

在request对象的header中也包含很多有用的信息，客户端会默认传递过来一些信息：

```http
{
	'content-type': 'application/json',
	'user-agent': 'PostmanRuntime/7.26.5',
	accept: '*/*',
	'postman-token': 'afe4b8fe-67e3-49cc-bd6f-f61c95c4367b',
	host: 'localhost:8080',
	'accept-encoding': 'gzip, deflate, br',
	connection: 'keep-alive',
	'content-length': '48'
}
```

content-type是这次请求携带的数据的类型：

- application/json表示是一个json类型；
- text/plain表示是文本类型；
- application/xml表示是xml类型；
- multipart/form-data表示是上传文件；

content-length：文件的大小和长度

keep-alive：

- http是基于TCP协议的，但是通常在进行一次请求和响应结束后会立刻中断；
- 在http1.0中，如果想要继续保持连接：浏览器需要在请求头中添加connection: keep-alive；服务器需要在响应头中添加connection:keep-alive；当客户端再次放请求时，就会使用同一个连接，直接一方中断连接；
- 在http1.1中，所有连接默认是connection: keep-alive的；
- 不同的Web服务器会有不同的保持keep-alive的时间；Node中默认是5s中；

accept-encoding：告知服务器，客户端支持的文件压缩格式，比如js文件可以使用gzip编码，对应.gz文件；

accept：告知服务器，客户端可接受文件的格式类型；

user-agent：客户端相关的信息

#### 1.7 响应结果

如果希望给客户端响应的结果数据，可以通过两种方式：

- write方法：这种方式是直接写出数据，但是并没有关闭流；

- end方法：这种方式是写出最后的数据，并且写出后会关闭流；

  ```javascript
  res.write('hello world')
  res.end('meaasge end')
  ```

如果没有调用end，客户端将会一直等待结果：所以客户端在发送网络请求时，都会设置超时时间。

**响应状态码**

Http状态码（Http Status Code）是用来表示Http响应状态的数字代码：

- Http状态码非常多，可以根据不同的情况，给客户端返回不同的状态码；

- 常见的状态码是下面这些：

  ![http状态码](https://gitee.com/Topcvan//img-storage/raw/master//node/http%E7%8A%B6%E6%80%81%E7%A0%81.png)

设置状态码常见的有两种方式：

```javascript
res.statusCode = 400;

res.writeHead(200, {
    ... // 响应Headers
})
```

**响应头文件**

返回头部信息，主要有两种方式：

- res.setHeader：一次写入一个头部信息；

- res.writeHead：同时写入header和status；

  ```javascript
  res.setHeader('Content-Type', 'application/json;charset=utf8')
  
  res.writeHead(200, {
      'Content-Type': 'application/json;charset=utf8'
  })
  ```

Header设置Content-Type有什么作用呢？

- 默认客户端接收到的是字符串，客户端会按照自己默认的方式进行处理；

- 比如如果是text/*，就会将响应结果当作页面文档进行渲染，如果是application/\*就会当作是请求的文件

  ```javascript
  res.writeHead(200, {
      'Content-Type': 'text/plain' // 当作字符串渲染
      'Content-Type': 'text/html'  // 当作html文档渲染
      'Content-Type': 'application/html' // 当作请求的html文件并下载
  })
  res.end('<h2>Hello World</h2>')
  ```

### 二、http请求

node既可以作为服务器处理网络请求，也可以主动发送请求。主要有两个方法：

- `http.get`

  ```javascript
  const http = require('http')
  
  http.get('http://localhost:8000', (res) => {
      // res的类型与服务器回调函数中的req一致,均为IncomingMessage
      res.on('data', data => {
          console.log(data.toString())
          console.log(JSON.parse(data.toString()))
      })
  })
  ```

- `http.request`

  ```javascript
  const http = require('http')
  
  const req = http.request({ // 返回Writable Stream
      method: 'POST',
      hostname: 'localhost',
      port: 8000
  }, res => {
      res.on('data', data => {
          console.log(data.toString())
          console.log(JSON.parse(data.toString()))
      })
  })
  req.write(postData) // 将数据写入Writable作为请求body
  req.end() // 主动调用end表示请求结束,否则会一直等待,get方法不用手动调用
  ```

### 三、文件上传

#### 3.1 错误示例

如果是一个很大的文件需要上传到服务器端，服务器端进行保存应该如何操作呢？以下是错误的操作实例：

```javascript
const http = require('http')
const fs = require('fs')

const fileWriter = fs.createWriteStream('./foo.png')

const server = http.createServer((req, res) => {
    if (req.url === 'upload') {
        if (req.method === 'POST') {
            req.on('data', data => {
        		fileWriter.write(data)
    		})
    
    		req.on('end', () => {
        		res.end('文件上传完成')
    		})
        }
    }
})

server.listen('8000', () => {
    console.log('服务器启动成功')
})
```

但是如果打开`foo.png`是会发现，并不能显示出图片，因为`req`的`data`中并不是只有图片的二进制数据，还有分隔符，文件名等控制信息，所以如果不将这些信息剔除直接将数据写入文件，图片是不能正常显示的

#### 3.2 正确示例

```javascript
const http = require('http')
const fs = require('fs')
const qs = require('querystring')
const buffer = require('Buffer')

const fileWriter = fs.createWriteStream('./foo.png')

const server = http.createServer((req, res) => {
    if (req.url === 'upload') {
        if (req.method === 'POST') {
            // 图片文件必须设置为binaryString格式编码
            req.setEncoding('binary')
            // 获取分隔符
            let boundary = req.header['content-type'].split(': ')[1]
            										 .replace('boundary=', '')
            let body = '' // body用于存储二进制数据解码后生成的字符串
            res.on('data', data => {
                body += data // data 会自动转成字符串
            })
            
            req.on('end', () => {
                const payload = qs.parse(body) // 解析数据字符串以进行分割
                // 获取文件类型
                const fileType = payload['Content-Type'].substring(1)
                // 获取截取长度
                const fileTypePosition = body.indexOf(fileType) + fileType.length
                let binaryData = body.subString(fileTypePosition)
                // 去掉开头的空格
                binaryData = binaryData.replace(/^\s\s?'/, '')
                
                const finalData = binaryData.substring(0, binaryData.indexOf('--' + boundary + '--'))
                
                fileWriter.write(buffer.from(finalData))
            })
        }
    }
}
```

