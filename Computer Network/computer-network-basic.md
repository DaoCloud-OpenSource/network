# Concepts
- 网卡 MAC 地址唯一，每个主机在一个网络里都会有一个 IP 地址
- 路由器可以把一个 IP 分配给很多个主机使用，这些主机对外只表现出一个IP。交换机可以把很多主机连起来，这些主机对外各有各的IP。
- 例如办公室无线网慢，走有线，桌子上放的是交换机，大家电脑接上去。交换机位于路由器和主机之间。交换机将这些主机连起来，对外（即路由器）各有各的IP。
- 路由器根据 IP 地址寻址，交换机根据 MAC 地址寻址。交换机中有一张记录着局域网主机MAC地址与交换机接口的对应关系的表。交换机按照表来发送数据，一个端口可能对应多个 MAC 地址。
- MAC 地址为物理地址，IP 地址为逻辑地址







## TCP/IP
### Layers
- The *application layer* in the TCP/IP protocol suite comprises of the application, presentation and the sessions layer of the ISO OSI model.
- The *socket layer* acts as the interface to and from the application layer to the transport layer. This layer is also called as the *Transport Layer Interface*. There are two kinds of sockets which operate in this layer, namely the connection oriented (streaming sockets) and the connectionless (datagram sockets).
- The next layer which exists in the stack is the *Transport Layer* which encapsulates the TCP and UDP functionality within it.
- The Network Layer in the TCP/IP protocol suite is called *IP layer*. In addition to IP, the ICMP and IGMP also go hand in hand with the IP layer.
- Link Layer
- Physical Layer
