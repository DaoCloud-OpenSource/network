## iptables 代理模式
在当前版本的 K8s 中，kube-proxy 默认使用的是 iptables 模式，通过各个 node 节点上的 iptables 规则来实现 Service 的负载均衡。

iptables 模式与 userspace 相同，kube-proxy 持续监听 Service 以及 Endpoints 对象的变化；但它并不在本地节点开启反向代理服务，而是把反向代理全部交给 iptables 来实现；即 iptables 直接将对 VIP 的请求转发给后端 Pod，通过 iptables 设置转发策略。

### 工作流程


```
                  +--------+         +---------+
                  | Client | ----->  | CoreDNS |  -----+
                  +--------+         +---------+       |
                                                       |
                        +----------+                   |
                        | iptables | <-----------------+
                        +----------+ 
                              |
                    +---------|---------+
                    |         |         |
                    |         |         |
                 +-----+   +-----+   +-----+                
                 | Pod |   | Pod |   | Pod |                
                 +-----+   +-----+   +-----+                

```









### 优缺点
相比 userspace 模式，克服了请求在用户态-内核态反复传递的问题，性能上有所提升，但使用 iptables NAT 来完成转发，存在不可忽视的性能损耗。

在大规模的集群中，iptables 添加规则会有很大的延迟。因为使用 iptables，每增加一个 Service 都会增加一条 iptables 的 chain。并且 iptables 修改了规则后必须得全部刷新才可以生效。

目前大部分生产环境，不会直接用 kube-proxy 作为服务代理，而是通过自己开发或者通过 Ingress Controller 来集成 HAProxy, Nginx 来代替 kube-proxy。