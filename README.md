# Christmas-Proxi
Self-make a SOCKS5 proxy (so-called ladder)

用 socket 搭一个梯子（TCP/IP 代理，SOCKS5 协议），尽管翻的是自己的墙，落地时还发现这不是自己的地儿嘛。

本项目推荐在 Linux 系统，或者 Windows 的 WSL 上进行，因为配置客户端和测试代理会更加方便。

## Baseline

笔者使用 python 尝试了这个项目，因为确实方便。最关键的是，采用 python 的话 baseline 就十分简单，只需提供教程，而不需要提供代码框架。（哈

使用 C/C++, Go, Java 等主流语言乃至 Rust 等都是可以的，开发过程中有疑惑同样可以问助教，助教自会去学的。其中 Go 开发这个十分方便，也已经被各位学长学习验证过其优越性，欢迎去学。

第一个任务是写一个 socks5 协议的代理服务器（代理端），不需要鉴权认证，即鉴权查询和鉴权响应部分的 `METHOD` 字段值取 `0x00` 即可，当然实现任意认证方法也是十分欢迎的。欲彻底了解 socks5 协议，请上 [Wiki](https://zh.wikipedia.org/wiki/SOCKS#SOCKS5) 或仔细阅读文末的 socks5 协议文档。

注意，只做 baseline 得不了全部分数，记得去 bonus 看看。

如果对这两横线之间的内容有任何不解，请积极联系助教，助你尽快搞清楚怎么一回事儿。

------

还是简要介绍一下 socks5 协议。其步骤可以用这张图描述。

![](https://github.com/T-Eric/Christmas-Proxi/blob/main/pics/Snipaste_2024-09-19_09-32-34.png)

中间左侧那一栏就是你要完成的任务，简单说是：

- phase 1b：接收客户端认证请求和支持的认证方法
- phase 2a：判断自己是否支持这个认证方法，做出回应
- phase 3b：接收客户端连接请求，尝试连接目标网络（图中的百度）
- phase 4a：回应自己是否连接成功
- phase 5：转发阶段，双端已经建立连接，永久循环等待一方数据并转发数据给另一方

具体传输的信息中哪几个字节有什么含义看文档；如何截取收到的字节信息为若干变量，以及如何将若干变量打包为字节信息传输，python 玩家推荐用 `struct` 库。

任务中说的不用做的鉴权认证，指的正是图中“方法 B 子协商”这一步，选择使用 0 去协商，就是可以略过的。

对了，好生研究官方提供的 socket 文档。对 python 玩家，下面两个 docs 基本上是必读：

- [`socket` — Low-level networking interface](https://docs.python.org/3/library/socket.html)
- [`socketserver` — A framework for network servers](https://docs.python.org/3/library/socketserver.html)

如果你是新手，请了解一些做基础工作的泛用库，比如读字节数据的 `struct` 库等。自学新手教程大概能接触到这些。

同时为 python 玩家准备了这个入门文章：[Socket Programming in Python (Guide)](https://realpython.com/python-sockets/)

------

会了。怎么测试自己写得好不好呢？

既然实现了代理端，客户端和服务端就像评测机，必须齐备。以本机作为客户端，服务端不能再是本机（让一个端口发出数据给自己没有意义）。就笔者已知，有两种服务端实现方式：

- 一个搭载 Linux 系统的虚拟机。
- [Proxy SwitchyOmega 浏览器插件](https://proxy-switchyomega.com/)。在情景模式下，修改代理服务器为你写的代理，用浏览器访问网页当做服务端，这样也可以通过在本地监测流量的方式调试你的代理服务器。

ip 地址选用 127.0.0.1 这个回环地址即可，如果你在其他ip地址有服务器可用，也可以选那些。端口可以任选，比如 1145。

任务已经说完，标准自在心中！只需要最后将你的成果展示给助教看看就行。

如果对这两横线之间的内容有任何不解，或遇到代码调试上的困难，请积极联系助教，助你尽快搞清楚怎么一回事儿。

------

做这个项目相当于学习和实践两个知识：

1. 项目语言的 socket 编程（废话）。最重要的是理解你所用的函数的参数和返回值到底是什么，以及怎样将外部得到的数据转化成这些类。
2. 要实现的连接协议的连接过程，以及协商交流中各个参数的含义。

自学成才，加油加油。

------

一点开发提示：

- 代理代码运行后应当是无限循环的，同时等待客户端和服务端的数据包，并在二者任意一端发出终止信号后终止数据传输。
- 你可以在启动代理后，在 Linux 终端输入 `curl -v --proxy socks5 xxx.xxx.xxx.xxx:xxxx http://www.baidu.com` 快速检查自己的代理服务器是否成功，其中的 `xx` 段即 ip 和端口由你自己指定，`-v` 参数可换其他。如果是 Windows，请尝试 `set http_proxy=http://xxx.xxx.xxx.xxx:xxxx`，然后进行 `http` 网址访问。

## Bonus

千重水，万重山，过了代理第一关。你已经发现，整个任务是偏探索学习性质的，bonus 也是如此，不会直接教各位的。需要各位自行查阅资料，理解所选任务的概念，按各自的想法实现之，然后将这个任务的特色效果展示给助教看。既没有评测方式，也没有设置标准，因为各位的实现必将百花齐放。

- TLS 劫持
- 透明代理

分数上，上面两个任务几乎是必然二选一的，而且这两个也最简单了。

- 代理客户端
  - 多级代理
  - 分流规则
    - 按 ip 分流
    - HTTP/TLS 分流
    - 按程序分流
- 反向代理
- UDP 代理
- 你也可以自拟项目做出来，助教会根据你完成的任务量打分的。

## 参考资料

下面是本项目涉及的协议文档。阅读原始文档尽管艰难（笔者也不想读）但实际上非常重要，只有严格按照协议要求进行协商和数据传输才能保证流程的正常进行。

- [RFC 1928: SOCKS Protocol Version 5](https://www.rfc-editor.org/rfc/rfc1928)
- [RFC 9293: Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293)
- [RFC 768: User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768)
- [RFC 9112: HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html)
- [HTTP on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3](https://www.rfc-editor.org/rfc/rfc8446)
