### 一、TLS握手过程

HTTP 由于是明文传输，所谓的明文，就是说客户端与服务端通信的信息都是肉眼可⻅的，随意使用一个抓包工具都可以截获通信的内容。
所以安全上存在以下三个⻛险：

- 窃听⻛险，比如通信链路上可以获取通信内容
- 篡改⻛险，比如强制植入垃圾广告，视觉污染
- 冒充⻛险，比如冒充淘宝网站

HTTPS 在 HTTP 与 TCP 层之间加入了 TLS 协议，来解决上述的⻛险。

在进行 HTTP 通信前，需要先进行 TLS 握手。TLS 的握手过程，如下图：

![TLS握手过程](https://gitee.com/Topcvan/img-storage/raw/master/network/TLS%E6%8F%A1%E6%89%8B%E8%BF%87%E7%A8%8B.png)

上图简要概述来 TLS 的握手过程，其中每一个「框」都是一个记录（record），记录是 TLS 收发数据的基本单位，类似于 TCP 里的 segment。多个记录可以组合成一个 TCP 包发送，所以通常经过「四个消息」就可以完成 TLS 握手，也就是需要 2个 RTT 的时延，然后就可以在安全的通信环境里发送 HTTP 报文，实现 HTTPS 协议。

所以可以发现，HTTPS 是应用层协议，需要先完成 TCP 连接建立，然后走 TLS 握手过程后，才能建立通信安全的连接。

事实上，不同的密钥交换算法，TLS 的握手过程可能会有一些区别。
这里先简单介绍下密钥交换算法，因为考虑到性能的问题，所以双方在加密应用信息时使用的是对称加密密钥，而对称加密密钥是不能被泄漏的，为了保证对称加密密钥的安全性，所以使用非对称加密的方式来保护对称加密密钥的协商，这个工作就是密钥交换算法负责的。

接下来，就以最简单的 RSA  密钥交换算法，来看看它的 TLS 握手过程

### 二、RSA握手过程

传统的 TLS 握手基本都是使用 RSA 算法来实现密钥交换的，在将 TLS 证书部署服务端时，证书文件中包含一对公私钥，其中公钥会在 TLS 握手阶段传递给客户端，私钥则一直留在服务端，一定要确保私钥不能被窃取。

在 RSA 密钥协商算法中，客户端会生成随机密钥，并使用服务端的公钥加密后再传给服务端。根据非对称加密算法，公钥加密的消息仅能通过私钥解密，这样服务端解密后，双方就得到了相同的密钥，再用它加密应用消息。

用 Wireshark 工具抓了用 RSA 密钥交换的 TLS 握手过程，可以从下面看到，一共经历来四次握手：

![tls四次握手](https://gitee.com/Topcvan/img-storage/raw/master/network/tls%E5%9B%9B%E6%AC%A1%E6%8F%A1%E6%89%8B.png)

#### 2.1 LS 第一次握手 

客户端首先会发一个「Client Hello」消息，字面意思我们也能理解到，这是跟服务器「打招呼」。

![tls第一次握手](https://gitee.com/Topcvan/img-storage/raw/master/network/tls%E7%AC%AC%E4%B8%80%E6%AC%A1%E6%8F%A1%E6%89%8B.png)

消息里面有客户端使用的 TLS 版本号、支持的密码套件列表，以及生成的随机数（Client Random），这个随机数会被服务端保留，它是生成对称加密密钥的材料之一。

#### 2.2 TLS 第二次握手

当服务端收到客户端的「Client Hello」消息后，会确认 TLS 版本号是否支持，和从密码套件列表中选择一个密码套件，以及生成随机数（Server Random）。

接着，返回「Server Hello」消息，消息里面有服务器确认的 TLS 版本号，也给出了随机数（Server Random），然后从客户端的密码套件列表选择了一个合适的密码套件。

![tls第二次握手](https://gitee.com/Topcvan/img-storage/raw/master/network/tls%E7%AC%AC%E4%BA%8C%E6%AC%A1%E6%8F%A1%E6%89%8B.png)

可以看到，服务端选择的密码套件是：

```http
Cipher Suite: TLS_RSA_WITH_AES_128_GCM_SHA256
```

密码套件有固定格式和规范，基本的形式是「密钥交换算法 + 签名算法 + 对称加密算法 + 摘要算法」，一般 WITH 单词前面有两个单词，第一个单词是约定密钥交换的算法，第二个单词是约定证书的验证算法。比如刚才的密码套件的意思就是：

- 由于 WITH 单词只有一个 RSA，则说明握手时密钥交换算法和签名算法都是使用 RSA；
- 握手后的通信使用 AES 对称算法，密钥⻓度 128 位，分组模式是 GCM；
- 摘要算法 SHA256 用于消息认证和产生随机数；

就前面这两个客户端和服务端相互「打招呼」的过程，客户端和服务端就已确认了 TLS 版本和使用的密码套件，而且客户端和服务端都会各自生成一个随机数，并且还会把随机数传递给对方。

那这个随机数有啥用呢？其实这两个随机数是后续作为生成「会话密钥」的条件，所谓的会话密钥就是数据传输时，所使用的对称加密密钥。

然后，服务端为了证明自己的身份，会发送「Server Certificate」给客户端，这个消息里含有数字证书。

![数字证书内容](https://gitee.com/Topcvan/img-storage/raw/master/network/%E6%95%B0%E5%AD%97%E8%AF%81%E4%B9%A6%E5%86%85%E5%AE%B9.png)

随后，服务端发了「Server Hello Done」消息，目的是告诉客户端：我已经把该给你的东⻄都给你了，本次打招呼完毕。

![server_hello_done](https://gitee.com/Topcvan//img-storage/raw/master//network/server_hello_done.png)

#### 2.3 客户端验证证书

户端拿到了服务端的数字证书后，要怎么校验该数字证书是真实有效的呢？

##### 2.3.1 数字证书和 CA 机构

在说校验数字证书是否可信的过程前，先来看看数字证书是什么，一个数字证书通常包含了：

- 公钥；
- 持有者信息；
- 证书认证机构（CA）的信息；
- CA 对这份文件的数字签名及使用的算法；
- 证书有效期；
- 还有一些其他额外信息；

那数字证书的作用，是用来认证公钥持有者的身份，以防止第三方进行冒充。说简单些，证书就是用来告诉客户端，该服务端是否是合法的，因为只有证书合法，才代表服务端身份是可信的。

我们用证书来认证公钥持有者的身份（服务端的身份），那证书又是怎么来的？又该怎么认证证书呢？

为了让服务端的公钥被大家信任，服务端的证书都是由 CA （Certificate Authority，证书认证机构）签名的，CA 就是网络世界里的公安局、公证中心，具有极高的可信度，所以由它来给各个公钥签名，信任的一方签发的证书，那必然证书也是被信任的。

之所以要签名，是因为签名的作用可以避免中间人在获取证书时对证书内容的篡改。

##### 2.3.2 数字证书签发和验证流程

数字证书签发和验证流程如下图所示：

![数字证书签发流程](https://gitee.com/Topcvan/img-storage/raw/master/network/%E6%95%B0%E5%AD%97%E8%AF%81%E4%B9%A6%E7%AD%BE%E5%8F%91%E6%B5%81%E7%A8%8B.png)

CA 签发证书的过程，如上图左边部分：

- 首先 CA 会把持有者的公钥、用途、颁发者、有效时间等信息打成一个包，然后对这些信息进行 Hash 计算，得到一个 Hash 值；
- 然后 CA 会使用自己的私钥将该 Hash 值加密，生成 Certificate Signature，也就是 CA对证书做了签名；
- 最后将 Certificate Signature 添加在文件证书上，形成数字证书；

客户端校验服务端的数字证书的过程，如上图右边部分：

- 首先客户端会使用同样的 Hash 算法获取该证书的 Hash 值 H1；
- 通常浏览器和操作系统中集成了 CA 的公钥信息，浏览器收到证书后可以使用 CA 的公钥解密 Certificate Signature 内容，得到一个 Hash 值 H2 ；
- 最后比较 H1 和 H2，如果值相同，则为可信赖的证书，否则则认为证书不可信。

##### 2.3.3 证书链

但事实上，证书的验证过程中还存在一个证书信任链的问题，因为我们向 CA 申请的证书一般不是根证书签发的，而是由中间证书签发的，比如百度的证书，从下图可以看到，证书的层级有三级：

![证书链](https://gitee.com/Topcvan/img-storage/raw/master/network/%E8%AF%81%E4%B9%A6%E9%93%BE.png)

对于这种三级层级关系的证书的验证过程如下：

- 客户端收到 baidu.com 的证书后，发现这个证书的签发者不是根证书，就无法根据本地已有的根证书中的公钥去验证 baidu.com 证书是否可信。于是，客户端根据 baidu.com 证书中的签发者，找到该证书的颁发机构是 “GlobalSign Organization Validation CA - SHA256 - G2”，然后向 CA 请求该中间证书。
- 请求到证书后发现 “GlobalSign Organization Validation CA - SHA256 - G2” 证书是由 “GlobalSign Root CA” 签发的，由于 “GlobalSign Root CA” 没有再上级签发机构，说明它是根证书，也就是自签证书。应用软件会检查此证书有否已预载于根证书清单上，如果有，则可以利用根证书中的公钥去验证 “GlobalSign Organization Validation CA - SHA256 - G2” 证书，如果发现验证通过，就认为该中间证书是可信的。
- “GlobalSign Organization Validation CA - SHA256 - G2” 证书被信任后，可以使用 “GlobalSign Organization Validation CA - SHA256 - G2” 证书中的公钥去验证baidu.com 证书的可信性，如果验证通过，就可以信任 baidu.com 证书。

在这四个步骤中，最开始客户端只信任根证书 GlobalSign Root CA 证书的，然后 “GlobalSign Root CA” 证书信任 “GlobalSign Organization Validation CA - SHA256 - G2” 证书，而 “GlobalSign Organization Validation CA - SHA256 - G2” 证书又信任 baidu.com 证书，于是客户端也信任 baidu.com 证书。

总括来说，由于用户信任 GlobalSign，所以由 GlobalSign 所担保的 baidu.com 可以被信任，另外由于用户信任操作系统或浏览器的软件商，所以由软件商预载了根证书的GlobalSign 都可被信任。

什么需要证书链这么麻烦的流程？Root CA 为什么不直接颁发证书，而是要搞那么多中间层级呢？
这是为了确保根证书的绝对安全性，将根证书隔离地越严格越好，不然根证书如果失守了，那么整个信任链都会有问题。

#### 2.4 TLS 第三次握手

客户端验证完证书后，认为可信则继续往下走。接着，客户端就会生成一个新的随机数  (pre-master)，用服务器的 RSA 公钥加密该随机数，通过「Change Cipher Key Exchange」消息传给服务端。

![premaster](https://gitee.com/Topcvan//img-storage/raw/master//network/premaster.png)

服务端收到后，用 RSA 私钥解密，得到客户端发来的随机数 (pre-master)。至此，客户端和服务端双方都共享了三个随机数，分别是 Client Random、Server Random、pre-master。

于是，双方根据已经得到的三个随机数，生成会话密钥（Master Secret），它是对称密钥，用于对后续的 HTTP 请求/响应的数据加解密。

生成完会话密钥后，然后客户端发一个「Change Cipher Spec」，告诉服务端开始使用加密方式发送消息。

![change_cipher_spec](https://gitee.com/Topcvan//img-storage/raw/master//network/change_cipher_spec.png)

然后，客户端再发一个「Encrypted Handshake Message（Finishd）」消息，把之前所有发送的数据做个摘要，再用会话密钥（master secret）加密一下，让服务器做个验证，验证加密通信是否可用和之前握手信息是否有被中途篡改过。

![会话摘要](https://gitee.com/Topcvan/img-storage/raw/master/network/%E4%BC%9A%E8%AF%9D%E6%91%98%E8%A6%81.png)

可以发现，「Change Cipher Key Exchange」之前传输的 TLS 握手数据都是明文，之后都是对称密钥加密的密文。

#### 2.5 TLS 第四次握手

服务器也是同样的操作，发「Change Cipher Spec」和「Encrypted Handshake Message」消息，如果双方都验证加密和解密没问题，那么握手正式完成。

最后，就用「会话密钥」加解密 HTTP 请求和响应了。

### 三、RSA算法缺陷

使用 RSA 密钥协商算法的最大问题是不支持前向保密。因为客户端传递随机数（用于生成对称加密密钥的条件之一）给服务端时使用的是公钥加密的，服务端收到到后，会用私钥解密得到随机数。所以一旦服务端的私钥泄漏了，过去被第三方截获的所有 TLS 通讯密文都会被破解。

为了解决这一问题，于是就有了 DH 密钥协商算法，这里简单介绍它的工作流程。

![DH密钥交换](https://gitee.com/Topcvan/img-storage/raw/master/network/DH%E5%AF%86%E9%92%A5%E4%BA%A4%E6%8D%A2.png)

客户端和服务端各自会生成随机数，并以此作为私钥，然后根据公开的 DH 计算公示算出各自的公钥，通过 TLS 握手双方交换各自的公钥，这样双方都有自己的私钥和对方的公钥，然后双方根据各自持有的材料算出一个随机数，这个随机数的值双方都是一样的，这就可以作为后续对称加密时使用的密钥。

DH 密钥交换过程中，即使第三方截获了 TLS 握手阶段传递的公钥，在不知道的私钥的情况下，也是无法计算出密钥的，而且每一次对称加密密钥都是实时生成的，实现前向保密。

但因为 DH 算法的计算效率问题，后面出现了 ECDHE 密钥协商算法，我们现在大多数网站使用的正是 ECDHE 密钥协商算法