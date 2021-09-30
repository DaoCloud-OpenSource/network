# Linux 网络概述
## 接收数据
网卡需要有驱动才能工作，驱动是加载到内核中的模块，负责衔接网卡和内核的网络模块，驱动在加载的时候将自己注册进网络模块，当相应的网卡收到数据包时，网络模块会调用相应的驱动程序处理数据。

数据包（packet）进入内存，并被内核的网络模块处理的流程（从物理层经过数据链路层进入网络层）大致如下：

1. 数据包从外面的网络进入物理网卡 (NIC, Network Interface Card)。如果目的地址不是该网卡，且该网卡没有开启混杂模式 (promiscuous)，该包会被网卡丢弃。
2. 网卡将数据包通过 DMA (Direct Memory Access) 的方式写入到指定的内存地址，该地址由网卡驱动分配并初始化。
3. 网卡通过硬中断（IRQ, Interrupt ReQuest）通知 CPU 有数据要处理
4. CPU 根据中断表，调用已经注册的中断函数，这个中断函数会调用驱动程序（NIC Driver）中相应的函数。
5. 驱动先禁用网卡的中断，表示驱动程序已经知道内存中有数据了，告诉网卡下次再收到数据包直接写内存就可以了，不用再通知 CPU，这样可以提高效率，避免 CPU 不停的被中断。
6. 启用软中断。由于硬中断过程不能被中断，如果它执行时间过长，会导致 CPU 没法响应其它硬件的中断，于是内核引入软中断，将硬中断处理函数中耗时的部分移到软中断处理函数中慢慢处理。
7. 软中断会触发内核网络模块中的软中断处理函数。网络模块调用网卡驱动里的 poll 函数来一个接一个地处理数据包，内存中数据包的格式只有驱动知道。
8. 驱动程序将内存中的数据包转换成内核网络模块能识别的 skb(socket buffer) 格式。
9. 网卡驱动将数据包转为 skb 之后，如果有 AF_PACKET 类型的 socket，拷贝一份数据给它。（tcpdump 抓包就是抓的这里的包）
10. 调用协议栈相应的函数，将数据包交给协议栈处理。（netfilter 相关钩子函数在这）
11. 待内存中的所有数据包被处理完成后（即 poll 函数执行完成），启用网卡的硬中断，这样下次网卡再收到数据的时候就会通知 CPU。


## 处理数据
// TODO 配置 macvlan 情况下进行抓包分析，虚拟网卡是不是还是使用的 root ns；eth0 是不是抓包可以看到走向虚拟网卡的数据包（我猜看不到，因为抓包点位于网卡之后）
// `ip_rcv` 函数是 IP 模块的入口函数，在该函数里面，第一件事就是将垃圾数据包（目的mac地址不是当前网卡，但由于网卡设置了混杂模式而被接收进来）直接丢掉

- `ip_rcv` 函数是 IP 模块的入口函数，它会将垃圾数据包（目的 MAC 地址不是当前网卡，但由于网卡设置了混杂模式而被接收进来）丢掉，然后调用注册在 NF_INET_PRE_ROUTING 上的函数。
- NF_INET_PRE_ROUTING 是 netfilter 放在协议栈中的钩子，可以通过 iptables 来注入一些数据包处理函数，用来修改或者丢弃数据包，如果数据包没被丢弃，将继续往下走
- routing: 进行路由，如果目的 IP 不是本地 IP，且没有开启 ip forward 功能，那么数据包将被丢弃，如果开启了 ip forward 功能，那将进入 `ip_forward` 函数。如果目的 IP 是本地 IP,则会调用 `ip_local_deliver`。
- `ip_forward` 会先调用 netfilter 注册的 NF_INET_FORWARD 相关函数，如果数据包没有被丢弃，那么将继续往后调用 `dst_output_sk` 函数。
- `dst_output_sk` 会调用 IP 层的相应函数将该数据包发送出去，同下文的[发送数据](#发送数据)。
- `ip_local_deliver` 会先调用 NF_INET_LOCAL_IN 相关的函数，如果通过，数据包将会进入上一层协议栈，传输层。
- 数据到达传输层，通过 socket 与应用层进行交互。


## 发送数据



## 参考
1. [Illustrated Guide to Monitoring and Tuning the Linux Networking Stack: Receiving Data | Packagecloud Blog](https://packagecloud.io/blog/illustrated-guide-monitoring-tuning-linux-networking-stack-receiving-data/)
2. [Linux网络 - 数据包的接收过程](https://segmentfault.com/a/1190000008836467)
3. [Monitoring and Tuning the Linux Networking Stack: Sending Data | Packagecloud Blog](https://packagecloud.io/blog/monitoring-tuning-linux-networking-stack-sending-data/)
4. [Queueing in the Linux Network Stack](https://www.coverfire.com/articles/queueing-in-the-linux-network-stack/)
5. [Linux网络 - 数据包的发送过程](https://segmentfault.com/a/1190000008926093)

