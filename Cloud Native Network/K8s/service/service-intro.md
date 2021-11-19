## Service 简介
为了支持集群的水平扩展和高可用性，Kubernetes 抽象出了 Service 的概念。

Service 是对提供相同服务的一组 Pod 的抽象。通过 Service 提供的统一入口对外提供服务，每个 Service 都有一个虚拟 IP 地址（Virtual IP）和端口号供客户端访问。它会根据访问策略(如负载均衡策略)来访问这组 Pod。

客户端通过访问虚拟 IP 地址来访问服务，服务则负责将请求转发到后端的 Pod 上。这其实就是一个反向代理，但与普通的反向代理有一些不同：它的 IP 地址是虚拟的，使用这个 VIP 无法从集群外面访问服务；它的部署和启停是由 Kubernetes 统一自动管理的。

在很多情况下，Service 只是一个概念，真正将 Service 的作用落实的是它背后 kube-proxy。