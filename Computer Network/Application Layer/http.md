## HTTP
### HTTP 1.0
在 HTTP 头里面，Cache-Control 用来控制缓存。当 Cache-Control 为 max-age=delta-seconds 时，如果缓存层中资源的缓存时间数值比指定时间的数值小，那么客户端可以接受缓存的资源；当指定 max-age 值为 0，那么缓存层通常需要将请求转发给应用集群。

服务端在返回资源时，会将该资源的最后更改时间通过 Last-Modified 字段返回给客户端。客户端下次请求时通过 If-Modified-Since 带上 Last-Modified，服务端检查该时间是否与服务器的最后修改时间一致：如果一致，则返回 304 状态码 Not Modified，不返回资源，可以节省带宽；如果不一致则返回 200 和修改后的资源，并带上新的时间。


### HTTP 1.1
目前使用的 HTTP 协议大部分都是 1.1。

在 HTTP 1.0 中主要使用 If-Modified-Since、Expires 来做为缓存判断的标准，HTTP 1.1 则引入了更多的缓存控制策略例如 Entity tag、If-Unmodified-Since、If-Match、If-None-Match等更多可供选择的缓存头来控制缓存策略。

HTTP 1.0 中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP 1.1 则在请求头引入了 Range 头，它允许只请求资源的某个部分，即返回码是 206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

在 HTTP 1.1 中新增了 24 个错误状态响应码，如 409（Conflict） 表示请求的资源与资源的当前状态发生冲突；410（Gone） 表示服务器上的某个资源被永久性的删除。

Host头处理，在HTTP 1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP 1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

HTTP 1.1 支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个 TCP 连接上可以传送多个 HTTP 请求和响应，减少了建立和关闭连接的消耗和延迟，在 HTTP 1.1 中默认开启 Connection： Keep-alive，一定程度上弥补了 HTTP 1.0 每次请求都要创建连接的缺点。这样建立的 TCP 连接，就可以在多次请求中复用。


### SPDY
SPDY（SPDY是 Speedy 的昵音，意为更快），是 Google 开发的基于 TCP 协议的应用层协议。它在 HTTP 的基础上，结合 HTTP 1.x 的多个痛点进行改进和升级。

SPDY 协议的目标是优化 HTTP 协议的性能，通过压缩、多路复用和优先级等技术，缩短网页的加载时间并提高安全性。协议核心思想是尽量减少 TCP 连接数，而对于 HTTP 的语义未做太大修改（比如，HTTP 的 GET 和 POST 消息格式保持不变），基本上兼容 HTTP 协议。

SPDY 通过复用在单个 TCP 连接上的多次请求，而非为每个请求单独开放连接。

SPDY 的多路复用可以设置优先级，而不像传统 HTTP 那样严格按照先入先出一个一个处理请求，它会选择性的先传输 CSS 这样更重要的资源，然后再传输网站图标之类不太重要的资源，可以避免让非关键资源占用网络通道的问题，提升 TCP 的性能。

SPDY 舍弃掉了不必要的头信息，经过压缩之后可以节省多余数据传输所带来的等待时间和带宽。


### HTTP 2.0
HTTP 2 借鉴了很多 SPDY 的特性。

HTTP 2 采用二进制格式传输数据，而非 HTTP 1 的文本格式，二进制协议解析起来更高效。

HTTP 2.0 对 HTTP 的头进行一定的压缩，将原来每次都要携带的大量 key value 在两端建立一个索引表，对相同的头只发送索引表中的索引。

HTTP 2.0 协议将一个 TCP 的连接中，切分成多个流。流是连接中的一个虚拟信道，可以承载双向的消息。每个流都有一个唯一的整数标识符。流是有优先级的。

HTTP 2.0 还将所有的传输信息分割为更小的消息和帧，并对它们采用二进制格式编码。Header 帧，用于传输 Header 内容，并且会开启一个新的流。Data 帧，用来传输正文实体。多个 Data 帧属于同一个流。 

通过这两种机制，客户端可以将多个请求分到不同的流中，然后将请求内容拆成帧，发送到一个 TCP 连接中。这些帧可以打散乱序发送，然后根据每个帧首部的流标识符重新组装，并且可以根据优先级，决定优先处理哪个流的数据。


### QUIC
HTTP 2.0 虽然大大增加了并发性，但 HTTP 2.0 也是基于 TCP 协议，TCP 协议在处理包时是有严格顺序的。在出现丢包的情况下，整个 TCP 都要开始等待重传，也就导致了后面的所有数据都被阻塞了。对于 HTTP/1.1 来说，可以开启多个 TCP 连接，出现这种情况反到只会影响其中一个连接，剩余的 TCP 连接还可以正常传输数据。

Google 的 QUIC 协议基于 UDP 协议，自定义了连接机制、重传机制，实现了无阻塞的多路复用和自定义的流量控制。

#### 连接机制
一条 TCP 连接由源 IP、源端口、目的 IP、目的端口的四元组标识。其中一个元素发生变化就需要断开重连。在移动互联情况下，当手机信号不稳定或者在 WIFI 和 移动网络之间切换时，都会导致重连，需要再次三次握手，导致一定的时延。基于 UDP 可以在 QUIC 自己的逻辑里面维护连接的机制，不再以四元组标识，而是以一个 64 位的随机数作为 ID 来标识，而且 UDP 是无连接的，当 IP 或者端口变化时，只要 ID 不变，就不需要重新建立连接。

#### 重传机制
TCP 为了保证可靠性，通过使用序号和应答机制，来解决顺序问题和丢包问题。超时重传机制采用了自适应重传算法，但超时时间存在不准确的问题。QUIC 有个递增的序列号。任何一个序列号的包只发送一次，RTT 计算相对准确。为了识别不同序列号但是内容相同的包，QUIC 定义了 offset。QUIC 像 TCP 一样面向连接，也是一个数据流，发送的数据在这个数据流里面有个偏移量 offset，可以通过 offset 查看数据发送到了哪里，只要这个 offset 的包没有来，就要重发。

#### 无阻塞的多路复用
同 HTTP 2.0 一样，同一条 QUIC 连接上可以创建多个 stream，来发送多个 HTTP 请求。但是，QUIC 是基于 UDP 的，一个连接上的多个 stream 之间没有依赖。例如 stream2 丢了一个 UDP 包，后面跟着 stream3 的一个 UDP 包，虽然 stream2 丢失的包需要重传，但是 stream3 的包无需等待，就可以发给用户。

#### 流量控制
TCP 的流量控制是通过滑动窗口来实现的。QUIC 的流量控制也是通过 window_update，来告诉对端它可以接受的字节数。但是 QUIC 的窗口是适应自己的多路复用机制的，不但在一个连接上控制窗口，还在 一个连接中的每个 stream 控制窗口。 

在 TCP 协议中，接收端的窗口的起始点是下一个要接收并且之前的都已 ACK 的包，即便后来的包都到了，放在缓存里面，窗口也不能右移，因为 TCP 的 ACK 机制是基于序列号的累计应答。这会导致后面的到了，也有可能超时重传，浪费带宽。 

QUIC 的 ACK 是基于 offset 的，每个 offset 的包来了，进了缓存，就可以应答，应答后就不会重发，中间的空挡会等待到来或者重发即可，而窗口的起始位置为当前收到的最大 offset，从这个 offset 到当前的 stream 所能容纳的最大缓存，是真正的窗口大小，这样更加准确。整个连接的窗口，需要对于所有的 stream 的窗口做一个统计。

