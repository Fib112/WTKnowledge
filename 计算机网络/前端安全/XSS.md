### 一、XSS攻击简介

#### 1.1 什么是XSS

Cross-Site Scripting（跨站脚本攻击）简称 XSS，是一种代码注入攻击。攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。利用这些恶意脚本，攻击者可获取用户的敏感信息如 Cookie、SessionID 等，进而危害数据安全。

XSS 的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

而由于直接在用户的终端执行，恶意代码能够直接获取用户的信息，或者利用这些信息冒充用户向网站发起攻击者定义的请求。

在部分情况下，由于输入的限制，注入的恶意脚本比较短。但可以通过引入外部的脚本，并由浏览器执行，来完成比较复杂的攻击策略。

这里有一个问题：用户是通过哪种方法“注入”恶意脚本的呢？

不仅仅是业务上的“用户的 UGC 内容”可以进行注入，包括 URL 上的参数等都可以是攻击的来源。在处理输入时，以下内容都不可信：

- 来自用户的 UGC 信息
- 来自第三方的链接
- URL 参数
- POST 参数
- Referer （可能来自不可信的来源）
- Cookie （可能来自其他子域注入）

#### 1.2 XSS分类

根据攻击的来源，XSS 攻击可分为存储型、反射型和 DOM 型三种。

|类型|存储区*|插入点*| ： |存储型 XSS|后端数据库|HTML| |反射型 XSS|URL|HTML| |DOM 型 XSS|后端数据库/前端存储/URL|前端 JavaScript|

- 存储区：恶意代码存放的位置。
- 插入点：由谁取得恶意代码，并插入到网页上。

##### 1.2.1 存储型 XSS

存储型 XSS 的攻击步骤：

1. 攻击者将恶意代码提交到目标网站的数据库中。
2. 用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

这种攻击常见于带有用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等。

##### 1.2.2 反射型 XSS

反射型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

反射型 XSS 跟存储型 XSS 的区别是：存储型 XSS 的恶意代码存在数据库里，反射型 XSS 的恶意代码存在 URL 里。

反射型 XSS 漏洞常见于通过 URL 传递参数的功能，如网站搜索、跳转等。

由于需要用户主动打开恶意的 URL 才能生效，攻击者往往会结合多种手段诱导用户点击。

POST 的内容也可以触发反射型 XSS，只不过其触发条件比较苛刻（需要构造表单提交页面，并引导用户点击），所以非常少见。

##### 1.2.3 DOM 型 XSS

DOM 型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL。
3. 用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

### 二、XSS攻击预防

通过前面的介绍可以得知，XSS 攻击有两大要素：

1. 攻击者提交恶意代码。
2. 浏览器执行恶意代码。

针对第一个要素：我们是否能够在用户输入的过程，过滤掉用户输入的恶意代码呢？

#### 2.1 输入过滤

在用户提交时，由前端过滤输入，然后提交到后端。这样做是否可行呢？

答案是不可行。一旦攻击者绕过前端过滤，直接构造请求，就可以提交恶意代码了。

那么，换一个过滤时机：后端在写入数据库前，对输入进行过滤，然后把“安全的”内容，返回给前端。这样是否可行呢？

我们举一个例子，一个正常的用户输入了 `5 < 7` 这个内容，在写入数据库前，被转义，变成了 `5 &lt; 7`。

问题是：在提交阶段，我们并不确定内容要输出到哪里。

这里的“并不确定内容要输出到哪里”有两层含义：

1. 用户的输入内容可能同时提供给前端和客户端，而一旦经过了 `escapeHTML()`，客户端显示的内容就变成了乱码( `5 &lt; 7` )。
2. 在前端中，不同的位置所需的编码也不同。

- 当 `5 &lt; 7` 作为 HTML 拼接页面时，可以正常显示：

  ```html
  <div title="comment">5 &lt; 7</div>
  ```

- 当 `5 &lt; 7` 通过 Ajax 返回，然后赋值给 JavaScript 的变量时，前端得到的字符串就是转义后的字符。这个内容不能直接用于 Vue 等模板的展示，也不能直接用于内容长度计算。不能用于标题、alert 等。

所以，输入侧过滤能够在某些情况下解决特定的 XSS 问题，但会引入很大的不确定性和乱码问题。在防范 XSS 攻击时应避免此类方法。

当然，对于明确的输入类型，例如数字、URL、电话号码、邮件地址等等内容，进行输入过滤还是必要的。

既然输入过滤并非完全可靠，我们就要通过“防止浏览器执行恶意代码”来防范 XSS。这部分分为两类：

- 防止 HTML 中出现注入。
- 防止 JavaScript 执行时，执行恶意代码。

#### 2.2 预防存储型和反射型 XSS 攻击

存储型和反射型 XSS 都是在服务端取出恶意代码后，插入到响应 HTML 里的，攻击者刻意编写的“数据”被内嵌到“代码”中，被浏览器所执行。

预防这两种漏洞，有两种常见做法：

- 改成纯前端渲染，把代码和数据分隔开。
- 对 HTML 做充分转义。

##### 2.2.1 纯前端渲染

纯前端渲染的过程：

1. 浏览器先加载一个静态 HTML，此 HTML 中不包含任何跟业务相关的数据。
2. 然后浏览器执行 HTML 中的 JavaScript。
3. JavaScript 通过 Ajax 加载业务数据，调用 DOM API 更新到页面上。

在纯前端渲染中，我们会明确的告诉浏览器：下面要设置的内容是文本（`.innerText`），还是属性（`.setAttribute`），还是样式（`.style`）等等。浏览器不会被轻易的被欺骗，执行预期外的代码了。

但纯前端渲染还需注意避免 DOM 型 XSS 漏洞（例如 `onload` 事件和 `href` 中的 `javascript:xxx` 等）。

在很多内部、管理系统中，采用纯前端渲染是非常合适的。但对于性能要求高，或有 SEO 需求的页面，我们仍然要面对拼接 HTML 的问题。

##### 2.2.2 转义 HTML

如果拼接 HTML 是必要的，就需要采用合适的转义库，对 HTML 模板各处插入点进行充分的转义。

常用的模板引擎，如 doT.js、ejs、FreeMarker 等，对于 HTML 转义通常只有一个规则，就是把 `& < > " ' /` 这几个字符转义掉，确实能起到一定的 XSS 防护作用，但并不完善：

| XSS 安全漏洞      | 简单转义是否有防护作用 |
| ----------------- | ---------------------- |
| HTML 标签文字内容 | 有                     |
| HTML 属性值       | 有                     |
| CSS 内联样式      | 无                     |
| 内联 JavaScript   | 无                     |
| 内联 JSON         | 无                     |
| 跳转链接          | 无                     |

所以要完善 XSS 防护措施，我们要使用更完善更细致的转义策略。

例如 Java 工程里，常用的转义库为 `org.owasp.encoder`。以下代码引用自 [org.owasp.encoder 的官方说明](https://link.juejin.cn?target=https%3A%2F%2Fwww.owasp.org%2Findex.php%2FOWASP_Java_Encoder_Project%23tab%3DUse_the_Java_Encoder_Project)。

```jsp
<!-- HTML 标签内文字内容 -->
<div><%= Encode.forHtml(UNTRUSTED) %></div>

<!-- HTML 标签属性值 -->
<input value="<%= Encode.forHtml(UNTRUSTED) %>" />

<!-- CSS 属性值 -->
<div style="width:<= Encode.forCssString(UNTRUSTED) %>">

<!-- CSS URL -->
<div style="background:<= Encode.forCssUrl(UNTRUSTED) %>">

<!-- JavaScript 内联代码块 -->
<script>
  var msg = "<%= Encode.forJavaScript(UNTRUSTED) %>";
  alert(msg);
</script>

<!-- JavaScript 内联代码块内嵌 JSON -->
<script>
var __INITIAL_STATE__ = JSON.parse('<%= Encoder.forJavaScript(data.to_json) %>');
</script>

<!-- HTML 标签内联监听器 -->
<button
  onclick="alert('<%= Encode.forJavaScript(UNTRUSTED) %>');">
  click me
</button>

<!-- URL 参数 -->
<a href="/search?value=<%= Encode.forUriComponent(UNTRUSTED) %>&order=1#top">

<!-- URL 路径 -->
<a href="/page/<%= Encode.forUriComponent(UNTRUSTED) %>">

<!--
  URL.
  注意：要根据项目情况进行过滤，禁止掉 "javascript:" 链接、非法 scheme 等
-->
<a href='<%=
  urlValidator.isValid(UNTRUSTED) ?
    Encode.forHtml(UNTRUSTED) :
    "/404"
%>'>
  link
</a>
```

可见，HTML 的编码是十分复杂的，在不同的上下文里要使用相应的转义规则。

#### 2.3 预防 DOM 型 XSS 攻击

DOM 型 XSS 攻击，实际上就是网站前端 JavaScript 代码本身不够严谨，把不可信的数据当作代码执行了。

在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 `.textContent`、`.setAttribute()` 等。

如果用 Vue/React 技术栈，并且不使用 `v-html`/`dangerouslySetInnerHTML` 功能，就在前端 render 阶段避免 `innerHTML`、`outerHTML` 的 XSS 隐患。

DOM 中的内联事件监听器，如 `location`、`onclick`、`onerror`、`onload`、`onmouseover` 等，`<a>` 标签的 `href` 属性，JavaScript 的 `eval()`、`setTimeout()`、`setInterval()` 等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。

```html
<!-- 内联事件监听器中包含恶意代码 -->
<img onclick="UNTRUSTED" onerror="UNTRUSTED" src="data:image/png,">

<!-- 链接内包含恶意代码 -->
<a href="UNTRUSTED">1</a>

<script>
// setTimeout()/setInterval() 中调用恶意代码
setTimeout("UNTRUSTED")
setInterval("UNTRUSTED")

// location 调用恶意代码
location.href = 'UNTRUSTED'

// eval() 中调用恶意代码
eval("UNTRUSTED")
</script>
```

如果项目中有用到这些的话，一定要避免在字符串中拼接不可信数据。

### 三、其他XSS防范措施

虽然在渲染页面和执行 JavaScript 时，通过谨慎的转义可以防止 XSS 的发生，但完全依靠开发的谨慎仍然是不够的。以下介绍一些通用的方案，可以降低 XSS 带来的风险和后果。

#### 3.1 Content Security Policy

严格的 CSP 在 XSS 的防范中可以起到以下的作用：

- 禁止加载外域代码，防止复杂的攻击逻辑。
- 禁止外域提交，网站被攻击后，用户的数据不会泄露到外域。
- 禁止内联脚本执行（规则较严格，目前发现 GitHub 使用）。
- 禁止未授权的脚本执行（新特性，Google Map 移动版在使用）。
- 合理使用上报可以及时发现 XSS，利于尽快修复问题。

#### 3.2 输入内容长度控制

对于不受信任的输入，都应该限定一个合理的长度。虽然无法完全防止 XSS 发生，但可以增加 XSS 攻击的难度。

#### 3.3 其他安全措施

- HTTP-only Cookie: 禁止 JavaScript 读取某些敏感 Cookie，攻击者完成 XSS 注入后也无法窃取此 Cookie。
- 验证码：防止脚本冒充用户提交危险操作。

### 四、CSP

CSP 通过告诉浏览器一系列规则，严格规定页面中哪些资源允许有哪些来源， 不在指定范围内的统统拒绝。相比同源策略，CSP 可以说是很严格了。

其实施有两种途径：

- 服务器添加 `Content-Security-Policy` 响应头来指定规则
- HTML 中添加 标签来指定 `Content-Security-Policy` 规则

#### 4.1 CSP 规则

无论是 header 中还是 `<meta>` 标签中指定，其值的格式都是统一的，由一系列 CSP 指令（directive）组合而成。

示例：

```
Content-Security-Policy: <policy-directive>; <policy-directive>…
```

这里 directive，即指令，是 CSP 规范中规定用以详细详述某种资源的来源，比如 `script-src`，指定脚本可以有哪些合法来源，`img-src` 则指定图片，以下是常用指令：

- `base-uri` 限制可出现在页面 `<base>` 标签中的链接。
- `child-src` 列出可用于 worker 及以 frame 形式嵌入的链接。 譬如: `child-src https://youtube.com` 表示只能从 Youtube 嵌入视频资源。
- `connect-src` 可发起连接的地址 (通过 XHR, WebSockets 或 EventSource)。
- `font-src` 字体来源。譬如，要使用 Google web fonts 则需要添加 `font-src https://themes.googleusercontent.com` 规则。
- `form-action` `<form>` 标签可提交的地址。
- `frame-ancestors` 当前页面可被哪些来源所嵌入（与 `child-src` 正好相反）。作用于 `<frame>`, `<iframe>`, `<embed>` 及 `<applet>`。 该指令不能通过 `<meta>` 指定且只对非 HTML文档类型的资源生效。
- `frame-src` 该指令已在 level 2 中废弃但会在 level 3 中恢复使用。未指定的情况下回退到 `tochild-src` 指令。
- `img-src` 指定图片来源。
- `media-src` 限制音视频资源的来源。
- `object-src` Flash 及其他插件的来源。
- `plugin-types` 限制页面中可加载的插件类型。
- `report-uri` 指定一个可接收 CSP 报告的地址，浏览器会在相应指令不通过时发送报告。不能通过 `<meta>` 标签来指定。
- `style-src` 限制样式文件的来源。
- `upgrade-insecure-requests` 指导客户端将页面地址重写，HTTP 转 HTTPS。用于站点中有大量旧地址需要重定向的情形。
- `worker-src` CSP Level 3 中的指令，规定可用于 worker, shared worker, 或 service worker 中的地址。

> `child-src` 与 `frame-ancestors` 看起来比较像。前者规定的是页面中可加载哪些 iframe，后者规定谁可以以 iframe 加载本页。 比如来自不同站点的两个网页 A 与 B，B 中有 iframe 加载了 A。那么
>
> - A 的 frame-ancestors 需要包含 B
> - B 的 child-src 需要包含 A

默认情况下，这些指令都是最大条件开放的，可以理解为其默认值为 `*`。比如 `img-src`，如果不明确指定，则可以从所有地方加载图片资源。

还有种特殊的指令 `default-src`，如果指定了它的值，则相当于改变了这些未指定的指令的默认值。可以理解为，上面 `img-src` 如果没指定，本来其默认值是 `*`，可以加载所有来源的图片，但设置 `default-src` 后，默认值就成了 `default-src` 指定的值。

常见的做法会设置 `default-src ‘self’`，这样所有资源都被限制在了和页面同域下。如果此时想要加载从 CDN 来的图片，将图片来源单独添加上即可。

```
Content-Security-Policy: default-src ‘self’; img-src https://cdn.example.com
```

这里的 `self` 及后来改成的 `none` 是预设值，需用引号包裹，否则会当成 URI 来解析。这里的 CSP 规则表示页面中脚本只能从同域及 `https://unpkg.com` 加载。假如我们把后者去掉， React 库会加载失败，同时控制台中会有加载失败的日志及被触发的规则列出来。`script-src`改成 `none` 之后表示页面不加载任何脚本，即使自己站点上的脚本都无法被加载执行

#### 4.2 指令可接受的值

指令后面跟的来源，有两种写法

- 预设值
- URI 通配符

##### 4.2.1 预设值

其中预设值有以下这些：

- `none` 不匹配任何东西。
- `self` 匹配当前域，但不包括子域。比如 example.com 可以，api.example.com 则会匹配失败。
- `unsafe-inline` 允许内嵌的脚本及样式。是的，没看错，对于页面中内嵌的内容也是有相应限制规则的。
- `unsafe-eval` 允许通过字符串动态创建的脚本执行，比如 eval，setTimeout 等。

特别地，在 CSP 的严格控制下，页面中内联脚本及样式也会受影响，在没有明确指定的情况下，其不能被浏览器执行。

考虑下面的代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
+   <meta http-equiv="Content-Security-Policy" content="default-src 'self'">
    <title>CSP Test</title>
    <style>
        body{
            color:red;
        }
    </style>
</head>
<body>
    <h1>Hello, World!</h1>
    <script>
        window.onload=function(){
            alert('hi jack!')
        }
    </script>
</body>
</html>
```

配置站点默认只信息同域的资源，但注意，这个设置并不包含内联的情况，所以结果会如下图：

![浏览器拒绝执行内联脚本](https://gitee.com/Topcvan/img-storage/raw/master/network/%E6%B5%8F%E8%A7%88%E5%99%A8%E6%8B%92%E7%BB%9D%E6%89%A7%E8%A1%8C%E5%86%85%E8%81%94%E8%84%9A%E6%9C%AC.png)

如何修复它呢。如果我们想要允许页面内的内联脚本或样式，则需要明确地通过 script-src 和 style-src 指出来，或者在`default src`中添加`unsafe-inline`。

通常是不建议使用 `unsafe-inline` 的（同样也不推荐使用 `unsafe-eval`），因为内联的脚本和样式维护不便，也不利用良好地组织代码。最佳实践是样式抽离到样式文件，脚本放到单独的 js 文件中加载，让 HTML 文件纯粹一点才是好的做法。即使是 `onclick=“myHandler”` 或 `href=“javascript:;”` 这种平时常见的写法，也属于内联的脚本，是需要改造的。

如果页面中非得用内联的写法，还有种方式。即页面中这些内联的脚本或样式标签，赋值一个加密串，这个加密串由服务器生成，同时这个加密串被添加到页面的响应头里面。

```html
<script nonce="EDNnf03nceIOfn39fn3e9h3sdfa">
  // 这里放置内联在 HTML 中的代码
</script>
```

页面 HTTP 响应头的 `Content-Security-Policy` 配置中包含相同的加密串：

```
Content-Security-Policy: script-src 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa'
```

注意这里的 `nonce-` 前缀。

`<style>` 标签也是类似的处理。

这里的加密串一定是随机不可预测的，否则达不到安全效果，且每次页面被访问时重新生成。

除了使用 `nonce` 指定加密串，还可以通过混淆的 hash 值来达到目的。这种做法不需要在标签上加 `nonce` 而是将需要内嵌的代码本身使用加密算法生成 hash 后放入 CSP 指令中作为值使用，这里的加密算法支持 sha256, sha384 和 sha512。此时 CSP 中使用的前缀为相应的算法名。

hash 方式的示例：

```html
<script>alert('Hello, world.');</script>
```

```http
Content-Security-Policy: script-src 'sha256-qznLcsROx4GACP2dm0UCKCzCG-HiZ1guq6ZZDob_Tng
```

##### 4.2.2 URI

除了上面的预设值，还可通过提供完整的 URI 或带通配符 `*` 的地址来匹配，以指定资源的合法来源。这里 URI 的规则和配置服务器的跨域响应头是一样的，参考 [Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)。

- `*://*.example.com:*` 会匹配所有 `example.com` 的子域名，但不包括 `example.com`。
- `http://example.com` 和 `http://www.example.com` 是两个不同的 URI。
- `http://example.com:80` 和 `http://example.com` 也是是两个不同的 URI，虽然网站默认端口就是 80

> 根据维基百科 [Uniform Resource Identifier](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) 页面 给出的解释，一个完整的 URI 由以下部分组成：
> `URI = scheme:[//authority]path[?query][#fragment]`
>
> 其中 `authority` 又包含：
> `authority = [userinfo@]host[:port]`
>
> 所以可以认为其中某一项不同，那都是两个 URI。了解这点很重要，一如上面列出的第一条例子 `*.example.com`， 我们很容易先入为主地认为既然已经允许了该域名的所有子域名，那必然 `example.com` 也是合法的。

因为 URI 是进行动态匹配的，所以解释了上面提到的预设值缘何要加引号。因为如果不加引号的话， `self` 会表示 `host` 是 `self` 的资源地址，而不会表示原有的意思。

#### 4.3 优先级

CSP 的配置是很灵活的。每条指令可指定多个来源，空格分开。而一条 CSP 规则可由多条指令组成，指令间用分号隔开。各指令间没有顺序的要求，因为每条指令都是各司其职。甚至一次响应中， `Content-Security-Policy` 响应头都可以重复设置。

我们来看这些情形下 CSP 的表现。

- 对于设置了多次响应头的情况，最严格的规则会生效。比如下面两条响应头中，虽然 第二条中设置 `connect-src` 允许 `http://example.com/`，但第一条里面设置了 `connect-src` 为 `none`，所以更加严格的 `none` 会生效。参见 [Multiple content security policies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#Multiple_content_security_policies)。

  ```http
  Content-Security-Policy: default-src 'self' http://example.com;
                           connect-src 'none';
  Content-Security-Policy: connect-src http://example.com/;
                           script-src http://example.com/
  ```

- 同一指令多次指定，以第一个为准，后续的会被忽略。

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta http-equiv="Content-Security-Policy" content="default-src 'self';default-src 'unsafe-inline';">
      <title>CSP Test</title>
      <style>
          body{
              color:red;
          }
      </style>
  </head>
  <body>
      <h1>Hello, World!</h1>
      <script>
          window.onload=function(){
              alert('hi jack!')
          }
      </script>
  </body>
  </html>
  ```

  ![多条重复指令仅第一条生效](https://gitee.com/Topcvan/img-storage/raw/master/network/%E5%A4%9A%E6%9D%A1%E9%87%8D%E5%A4%8D%E6%8C%87%E4%BB%A4%E4%BB%85%E7%AC%AC%E4%B8%80%E6%9D%A1%E7%94%9F%E6%95%88.png)

  很智能地， 浏览器不仅会将检测不过的资源及指令打印出来，重复配置时被忽略的指令也会提示出来。

- 指定 `default-src` 的情况下，它会充当 `Fetch` 类指令 的默认值。即 `default-src` 并不对所有指令生效，其他指令默认值仍是 `*`。

#### 4.4 CSP报告

##### 4.4.1 发送报告

当检测到非法资源时，除了控制台看到的报错信息，也可以让浏览器将日志发送到服务器以供后续分析使用。接收报告的地址可在 `Content-Security-Policy` 响应头中通过 `report-uri` 指令来配置。当然，服务端需要编写相应的服务来接收该数据。

```http
Content-Security-Policy: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
```

服务端拿到的是以 JSON 形式传来的数据。

```json
{
  "csp-report": {
    "document-uri": "http://example.org/page.html",
    "referrer": "http://evil.example.com/",
    "blocked-uri": "http://evil.example.com/evil.js",
    "violated-directive": "script-src 'self' https://apis.google.com",
    "original-policy": "script-src 'self' https://apis.google.com; report-uri http://example.org/my_amazing_csp_report_parser"
  }
}
```

##### 4.4.2 报告模式

CSP 提供了一种报告模式，该模式下资源不会真的被限制加载，只会对检测到的问题进行上报 ，以 `JSON` 数据的形式发送到 `report-uri` 指定的地方。

通过指定 `Content-Security-Policy-Report-Only` 而不是 `Content-Security-Policy`，则开启了报告模式：

```http
Content-Security-Policy-Report-Only: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
```

当然，你也可以同时指定两种响应头，各自里的规则还会正常执行，不会互相影响。比如：

```http
Content-Security-Policy: img-src *;
Content-Security-Policy-Report-Only: img-src ‘none’; report-uri http://reportcollector.example.com/collector.cgi
```

这里图片还是会正常加载，但是 `img-src ‘none’` 也会检测到并且发送报告。

报告模式对于测试非常有用。在开启 CSP 之前肯定需要对整站做全面的测试，将发现的问题及时修复后再真正开启，比如上面提到的对内联代码的改造。

##### 4.4.3 推荐的做法

这样的安全措施当然是能尽快启用就尽快。以下是推荐的做法：

- 先只开启报告模式，看影响范围，修改问题。
- 添加指令时从 `default-src ‘none’` 开始，查看报错，逐步添加规则直至满足要求。
- 上线后观察一段时间，稳定后再由报告模式转到强制执行。

### 五、XSS攻击总结

1. XSS 防范是后端 RD 的责任，后端 RD 应该在所有用户提交数据的接口，对敏感字符进行转义，才能进行下一步操作。

   > 不正确。因为：
   >
   > - 防范存储型和反射型 XSS 是后端 RD 的责任。而 DOM 型 XSS 攻击不发生在后端，是前端 RD 的责任。防范 XSS 是需要后端 RD 和前端 RD 共同参与的系统工程。
   > - 转义应该在输出 HTML 时进行，而不是在提交用户输入时。

2. 所有要插入到页面上的数据，都要通过一个敏感字符过滤函数的转义，过滤掉通用的敏感字符后，就可以插入到页面中。

   > 不正确。 不同的上下文，如 HTML 属性、HTML 文字内容、HTML 注释、跳转链接、内联 JavaScript 字符串、内联 CSS 样式表等，所需要的转义规则不一致。 业务 RD 需要选取合适的转义库，并针对不同的上下文调用不同的转义规则。

整体的 XSS 防范是非常复杂和繁琐的，我们不仅需要在全部需要转义的位置，对数据进行对应的转义。而且要防止多余和错误的转义，避免正常的用户输入出现乱码。

虽然很难通过技术手段完全避免 XSS，但我们可以总结以下原则减少漏洞的产生：

- **利用模板引擎** 开启模板引擎自带的 HTML 转义功能。例如： 在 ejs 中，尽量使用 `<%= data %>` 而不是 `<%- data %>`； 在 doT.js 中，尽量使用 `{{! data }` 而不是 `{{= data }`； 在 FreeMarker 中，确保引擎版本高于 2.3.24，并且选择正确的 `freemarker.core.OutputFormat`。
- **避免内联事件** 尽量不要使用 `onLoad="onload('{{data}}')"`、`onClick="go('{{action}}')"` 这种拼接内联事件的写法。在 JavaScript 中通过 `.addEventlistener()` 事件绑定会更安全。
- **避免拼接 HTML** 前端采用拼接 HTML 的方法比较危险，如果框架允许，使用 `createElement`、`setAttribute` 之类的方法实现。或者采用比较成熟的渲染框架，如 Vue/React 等。
- **时刻保持警惕** 在插入位置为 DOM 属性、链接等位置时，要打起精神，严加防范。
- **增加攻击难度，降低攻击后果** 通过 CSP、输入长度配置、接口安全措施等方法，增加攻击的难度，降低攻击的后果。
- **主动检测和发现** 可使用 XSS 攻击字符串和自动扫描工具寻找潜在的 XSS 漏洞。