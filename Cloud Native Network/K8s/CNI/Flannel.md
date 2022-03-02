## Flannel
Flannel 是 Kubernetes 生态体系中最简单的三层网络配置，通常是默认配置方法。

Flannel 创建了一个 Overlay 网络来给每个主机分配一个字网。

Flannel 提供了三层 IPv4 的网络。

Flannel.1 是 VxLAN 接口
CNI0 是 桥

Pod 的 namespace 里 `eth0@if12` 对应 root namespace 第 12 个网口

Pod 之间通信， cni0 桥回应 ARP 请求
路由表

Flannel.1 负责 VxLAN 的封解包
流量在离开之前先经过 Flannel.1



### VxLAN
only one vxlan network is created

### Direct Routing
for direct routing, remote gw must be reachable via layer-2

### UDP
for debugging purposes