## 访问不通
- 按链路思想进行排查

### IPv6 Service 访问不通
- ip6tables 中 OUPTPUT 链 nat 表数据包数目不变
- 排查发现没有 IPv6 的默认路由
- `ip -6 route add default via fdb2:2c26:f4e4::1`


## 其他问题
### Endpoint 一会消失一会出现
- 某个 Service 的后端 endpoint 一会显示有后端，一会显示没有。显示没有后端，意味着后端的 address 被判定为 NotReady。
- kubelet 在准备上报信息时，需要收集容器、镜像等的信息。虽然 kubelet 默认是 10 秒上报一次，但是实际的上报周期约为 20~50 秒。而 kube-controller-manager 判断 node 上报心跳超时的时间为 40 秒。所以会有一定概率超时。一旦超时，kube-controller 会将该 node 上的所有 pod 的 status 从 Ready 置为 False。
- 较为简单的方案是在 kube-controller 上配置这个超时时间 node-monitor-grace-period 长一些。