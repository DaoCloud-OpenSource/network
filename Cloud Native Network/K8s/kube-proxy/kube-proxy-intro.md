## kube-proxy 简介

kube-proxy 是 Kubernetes 的核心组件，作为 Kubernetes 的网络代理在每个节点上运行。它是实现 Kubernetes Service 的通信与负载均衡机制的重要组件。

网络代理反映了每个节点上 Kubernetes API 中定义的服务，并且可以执行简单的 TCP、UDP 和 SCTP 流转发，或者在一组后端进行循环 TCP、UDP 和 SCTP 转发。 

kube-proxy 存在于每个 node 结点，监听 master node API，从 apiserver 获取与 Service、Endpoint 相关的信息，为 Pod 创建代理服务，每发现一个新服务就在 node 上开启一个随机端口代理到 Pod 的连接，在每个节点上实现对 Service 请求的路由和转发，从而实现 K8s 层级的虚拟转发网络。

kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡，定时从 etcd 服务获取到 Service 信息来做相应的策略，维护网络规则和四层负载均衡工作。在 K8s 集群中微服务的负载均衡是由 kube-proxy 实现的，它是 K8s 集群内部的负载均衡器，也是一个分布式代理服务器，在 K8s 的每个节点上都有一个，这一设计体现了它的伸缩性优势。

在 kube-proxy 的作用下，客户端访问 Service 无须关心后端有几个 Pod，中间过程的通信、负载均衡及故障恢复都是透明的。

### 代理模式
1. **User space**: 路由发生在用户空间。
2. **iptables**: 利用 Linux 内核的 Netfilter 规则来实现转发，是目前的默认模式。
3. **IPVS** (IP Virtual Server): 建立在 Netfilter 之上, 在内核级别实现了 Layer-4 负载均衡, 支持多种负载均衡算法, 需要 Linux 内核加载 IPVS 模块。

