## Dual Stack
> [IPv6](../../Computer%20Network/Protocol/network/IPv6.md)

### Goals
- 支持 Pod 之间的 IPv4 或 IPv6 通信
- 支持 Pod 使用 IPv4 或者 IPv6 访问集群外部
- Ingress Controller 支持 IPv4 或 IPv6 的外部访问
- 为 ClusterIP，NodePort，ExternalIP 类型的 Service 提供双栈支持

### Non-Goals
- IPv4 与 IPv6 之间的通信不在考虑范围之内（Ingress Controller 或许可以将流量负载均衡至同一个 EndPoint的 IPv4 或 IPv6 地址）


### Proposal
- Service Cluster IP Range 配置 `--service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>`。如果 Service Spec 里面没有指定 IP family，就从第一个 CIDR 中分配 IP。
- Endpoint 的地址族和 Service 的第一个地址族相同，例如一个 IPv6 的 Service 只会有 IPv6 的 Endpoint。
- EndpointSlices 支持双栈 endpoints。


### Pod 多 IP 的感知
- 扩展 Pod Status


### 环境搭建
使用 kubeadm 搭建

[How to enable IPv6 on Kubernetes (aka dual-stack cluster)](https://medium.com/@elfakharany/how-to-enable-ipv6-on-kubernetes-aka-dual-stack-cluster-ac0fe294e4cf)

CNI 插件 以 calico 为例
