## iptables 代理模式
随着 Service 数量的增大，iptables 模式由于线性查找匹配、全量更新等特点，其性能会显著下降。从 K8s 的 1.8 版本开始，kube-proxy 引入了 IPVS 模式。

与 iptables、userspace 一样，kube-proxy 依然监听 Service 以及 Endpoints 对象的变化, 不过它并不创建反向代理, 也不创建大量的 iptables 规则, 而是通过 netlink 创建ipvs规则，并使用 K8s Service 与 Endpoints 信息，对所在节点的 ipvs 规则进行定期同步。netlink 与 iptables 底层都是基于 netfilter 钩子，但是 netlink 由于采用了 hash table 而且直接工作在内核态，在性能上比 iptables 更优。当 service 数量达到一定规模时，hash 查表的速度优势就会显现出来，从而提高 service 的服务性能。


### 工作流程


```
                  +--------+         +---------+
                  | Client | ----->  | CoreDNS |  -----+
                  +--------+         +---------+       |
                                                       |
                        +----------+                   |
                        |   ipvs   | <-----------------+
                        +----------+ 
                              |
                    +---------|---------+
                    |         |         |
                    |         |         |
                 +-----+   +-----+   +-----+                
                 | Pod |   | Pod |   | Pod |                
                 +-----+   +-----+   +-----+                

```

