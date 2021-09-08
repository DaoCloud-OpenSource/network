## K8s 网络概述
### 容器到容器
- Pod 内部容器共享一个网络命名空间，使用 loopback 通信
- Pod 使用 pause 容器启动网络命名空间，pause 的 NetworkMode 是 bridge，其它容器都共享 pause 容器的网络

### Pod 到 Pod
1. 每一个 Pod 都有唯一的 IP 地址。
2. 同一 node 上的 Pod 都通过 Veth 连接到 docker0，可以直接用对方的 IP 地址通信，而不需要其它发现机制，例如 DNS、Consul、etcd。
3. 不同 node 上的 Pod 也是通过 IP 地址通信，但是需要进行配置。每个 node 的 docker0 地址不能有冲突，通过找到对方 Pod 所在机器 IP，将数据包通过本机网卡发向对方 Pod 所在机器的网卡。数据包到达对方网卡之后，对方知道如何将数据通过 docker0 转发到 Pod。
4. 每个主机的 docker0 都可以被路由到，因此在网络层可以把 node 看作一个路由器
(不同网络方案有所出入)