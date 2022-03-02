## 流量控制
### 主要功能
- 路由、流量转移
- 流量进出
- 网络弹性能力
- 测试相关

### CRD
- 虚拟服务 Virtual Service: 虚拟服务是实现路由功能的重要组件，它将流量路由到给定目标地址。将请求的地址和实际的 workload 解耦。
- 目标规则 Destination Rule: 目标规则定义虚拟服务路由目标地址的真实地址,即子集 subset。
- 网关 Gateway
- 服务入口 Service Entry: 服务入口把外部服务注册到网格中。
- Sidecar




超时：控制故障范围，避免故障扩散
重试：解决网络抖动时通信失败的问题

熔断：一种过载保护的手段，避免服务的级联失败。
关键点：三个状态；失败计数器（阈值）；超时时钟
httpbin
fortio 负载均衡测试服务
FORTIO_POD=$(k get po| grep fortio | awk '{print $1}')
k exec -it $FORTIO_POD -c fortio /usr/bin/fortio -- load -curl http://httpbin:8000/get

并发 2 同时 20 次
/usr/bin/fortio -- load -c 2 -qps 0 -n 20 -loglevel Warning -curl http://httpbin:8000/get

k exec $FORTIO_POD -c istio-proxy -- pilot-agent request GET stats | grep httpbin.default | grep pending

