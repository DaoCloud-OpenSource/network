## Overlay Network
延展网络 (Overlay Network)，是指构建在另一个网络 (Underlay 网络) 上的计算机网络。

![](/Pics/2021-09-05-22-32-54.png)

Underlay 网络是专门用来承载用户 IP 流量的基础架构层。Underlay 网络和物理机都是真正存在的实体，它们分别对应着真实存在的网络设备和计算设备，而 Overlay 网络和虚拟机都是依托在下层实体使用软件虚拟出来的层级。

实践中一般会使用虚拟局域网扩展技术(Virtual Extensible LAN，VxLAN)组建 Overlay 网络。VxLAN 使用虚拟隧道端点(Virtual Tunnel End Point, VTEP)设备对服务器发出和收到的数据包进行二次封装和解封。