## 传输层
传输层里有比较重要的两个协议，TCP 和 UDP。TCP 是面向连接的，UDP 是面向无连接的。

所谓的建立连接，是为了在客户端和服务端之间维护连接，建立一定的数据结构来维护双方交互的状态， 以此来保证所谓的面向连接的特性。

TCP 提供可靠交付。通过 TCP 连接传输的数据，无差错、不丢失、不重复、并且按序到达。UDP 继承了 IP 包的特性，不保证不丢失，不保证按顺序到达。

TCP 是面向字节流的，发送的时候发的是一个流，没头没尾。IP 层的数据传输单元是数据包，之所以变成了流，也是通过状态维护达到的。而 UDP 继承了 IP 的特性，基于数据报，一个一个发送与接受。

此外，TCP 还有 UDP 所不具备的系列特性，例如拥塞控制、有状态服务等等。