# [相关资料](https://www.ibm.com/developerworks/cn/opensource/os-nodejs/)
# Node的本质是什么
* 服务器
# 和它处于同一作用的产品有哪些
*  Apache 和 Tomcat 等
# 为什么会产生node.js
* 传统服务器 每个连接都会生成一个新线，程每个新线程可能需要 2 MB 的配套内存。在一个拥有 8 GB RAM 的系统上，理论上最大的并发连接数量是 4,000 个用户
* 整个 Web 应用程序架构（包括流量、处理器速度和内存速度）中的瓶颈是：服务器能够处理的并发连接的最大数量。
* Node 解决这个问题的方法是：更改连接到服务器的方式。每个连接发射一个在 Node 引擎的进程中运行的事件；Node 还宣称，运行它的服务器能支持数万个并发连接。
# Node 的特性
* Node 本身运行 V8 JavaScript
* 事件驱动编程
## 什么是 V8 JavaScript
* V8 JavaScript 引擎是 Google 用于其 Chrome 浏览器的底层 JavaScript 引擎
* Google 使用 V8 创建了一个用 C++ 编写的超快解释器
* 该解释器拥有另一个独特特征；您可以下载该引擎并将其嵌入任何 应用程序
# Node 适用于哪些情况（高连接数，低逻辑处理）
* 在响应客户端之前，您预计可能有很高的流量，但所需的服务器端逻辑和处理不一定很多
*RESTful API
提供 RESTful API 的 Web 服务接收几个参数，解析它们，组合一个响应，并返回一个响应（通常是较少的文本）给用户。这是适合 Node 的理想情况，因为您可以构建它来处理数万条连接。它仍然不需要大量逻辑；它本质上只是从某个数据库中查找一些值并将它们组成一个响应。由于响应是少量文本，入站请求也是少量的文本，因此流量不高，一台机器甚至也可以处理最繁忙的公司的 API 需求。

* Twitter 队列
想像一下像 Twitter 这样的公司，它必须接收 tweets 并将其写入数据库。实际上，每秒几乎有数千条 tweet 达到，数据库不可能及时处理高峰时段所需的写入数量。Node 成为这个问题的解决方案的重要一环。如您所见，Node 能处理数万条入站 tweet。它能快速而又轻松地将它们写入一个内存排队机制（例如 memcached），另一个单独进程可以从那里将它们写入数据库。Node 在这里的角色是迅速收集 tweet，并将这个信息传递给另一个负责写入的进程。想象一下另一种设计（常规 PHP 服务器会自己尝试处理对数据库本身的写入）：每个 tweet 都会在写入数据库时导致一个短暂的延迟，因为数据库调用正在阻塞通道。由于数据库延迟，一台这样设计的机器每秒可能只能处理 2000 条入站 tweet。每秒处理 100 万条 tweet 则需要 500 个服务器。相反，Node 能处理每个连接而不会阻塞通道，从而能够捕获尽可能多的 tweets。一个能处理 50,000 条 tweet 的 Node 机器仅需 20 台服务器即可。

* 电子游戏统计数据
如果您在线玩过《使命召唤》这款游戏，当您查看游戏统计数据时，就会立即意识到一个问题：要生成那种级别的统计数据，必须跟踪海量信息。这样，如果有数百万玩家同时在线玩游戏，而且他们处于游戏中的不同位置，那么很快就会生成海量信息。Node 是这种场景的一种很好的解决方案，因为它能采集游戏生成的数据，对数据进行最少的合并，然后对数据进行排队，以便将它们写入数据库。使用整个服务器来跟踪玩家在游戏中发射了多少子弹看起来很愚蠢，如果您使用 Apache 这样的服务器，可能会 有一些有用的限制；但相反，如果您专门使用一个服务器来跟踪一个游戏的所有统计数据，就像使用运行 Node 的服务器所做的那样，那看起来似乎是一种明智之举。



# Node 模块概念 (个人理解类似第三方库概念)
## npm(Node Package Manager)  用于管理Node的模块（个人理解类似于Android中的gradle）npm是安装node本身就带的不需要额外安装
* yarn是facebook开发和npm功能相同的工具，可以加速 node 模块的下载，需要额外安装。