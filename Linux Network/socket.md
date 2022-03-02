服务器端先初始化/创建Socket，然后与端口绑定/绑定地址(bind)，对端口进行监听(listen)，调用accept阻塞/等待连续，等待客户端连接。在这时如果有个客户端初始化一个Socket，然后连接服务器(connect)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束。



socket函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而socket()用于创建一个socket描述符（socket descriptor），它唯一标识一个socket。这个socket描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。

address family
protofamily：即协议域，又称为协议族（family）。常用的协议族有，AF_INET(IPV4)、AF_INET6(IPV6)、AF_LOCAL（或称AF_UNIX，Unix域socket）、AF_ROUTE等等。协议族决定了socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用ipv4地址（32位的）与端口号（16位的）的组合、AF_UNIX决定了要用一个绝对路径名作为地址。

　type：指定socket类型。常用的socket类型有，SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等等。


bind()函数把一个地址族中的特定地址赋给socket，也可以说是绑定ip端口和socket。例如对应AF_INET、AF_INET6就是把一个ipv4或ipv6地址和端口号组合赋给socket。


AF_PACKET
网卡驱动将数据包转为 skb 之后，如果有 AF_PACKET 类型的 socket，拷贝一份数据给它。tcpdump 抓包就是抓的这里的包。
数据包发往网卡的时候，拷贝一份给 AF_PACKET 类型的 socket。tcpdump 抓包就是抓的这里的包。


NF_INET_PRE_ROUTING 指 netfilter IPv4 PRE_ROUTING 钩子函数

数据包进入传输层，根据目的IP和端口找对应的socket，如果没有找到相应的socket，那么该数据包将会被丢弃，否则继续

udp_rcv： udp_rcv函数是UDP模块的入口函数，它里面会调用其它的函数，主要是做一些必要的检查，其中一个重要的调用是__udp4_lib_lookup_skb，该函数会根据目的IP和端口找对应的socket，如果没有找到相应的socket，那么该数据包将会被丢弃，否则继续
sock_queue_rcv_skb： 主要干了两件事，一是检查这个socket的receive buffer是不是满了，如果满了的话，丢弃该数据包，然后就是调用sk_filter看这个包是否是满足条件的包，如果当前socket上设置了filter，且该包不满足条件的话，这个数据包也将被丢弃（在Linux里面，每个socket上都可以像tcpdump里面一样定义filter，不满足条件的数据包将会被丢弃）
__skb_queue_tail： 将数据包放入socket接收队列的末尾
sk_data_ready： 通知socket数据包已经准备好


应用层一般有两种方式接收数据，一种是recvfrom函数阻塞在那里等着数据来，这种情况下当socket收到通知后，recvfrom就会被唤醒，然后读取接收队列的数据；另一种是通过epoll或者select监听相应的socket，当收到通知后，再调用recvfrom函数去读取接收队列的数据。两种情况都能正常的接收到相应的数据包。