## Whereabouts
Whereabouts 是一种 IPAM 插件。Whereabouts 可以跨越整个集群的节点，为整个集群网络分配 IP 和回收 IP，而且支持 IPv4 和 IPv6 协议，类似 DHCP 服务器。它将 IP 分配信息记录在 etcd 中或者作为 K8s Custom Resource 的后端。

Whereabouts 跟踪 Pod 的生命周期，在 Pod 被销毁之后会回收分配出去的 IP，进行下一次分配。Whereabouts 总是分配可用 IP 范围内最低的地址。

Whereabouts 也支持排除指定 IP 段，不进行分配。例如，从 `192.168.2.0/24` 中排除 `192.168.2.0/28`，那么第一个被分配的将是 `192.168.2.16`。

Whereabouts 不会分配网络位全 0 的地址和广播地址。

### 配置

- Whereabouts 以 daemonset 的形式存在于节点上。定义了两种 CRD `IPPool` 和 `OverlappingRangeIPReservation`。并通过 CronJob 来维护 IP 分配信息。如果某个节点宕机，也会回收运行在该节点上 Pods 的 IP。

```shell
git clone https://github.com/k8snetworkplumbingwg/whereabouts && cd whereabouts

       kubectl apply \
         -f doc/crds/daemonset-install.yaml \
         -f doc/crds/whereabouts.cni.cncf.io_ippools.yaml \
         -f doc/crds/whereabouts.cni.cncf.io_overlappingrangeipreservations.yaml \
         -f doc/crds/ip-reconciler-job.yaml
```

- 配置示例。示例中移除了四个 IP 地址： 192.168.2.229, 192.168.2.230, 192.168.2.231, 192.168.2.236。

```
{
      "cniVersion": "0.4.0",
      "name": "whereaboutsexample",
      "type": "macvlan",
      "master": "eth0",
      "mode": "bridge",
      "ipam": {
        "type": "whereabouts",
        "range": "192.168.2.225/28",
        "exclude": [
           "192.168.2.229/30",
           "192.168.2.236/32"
        ]
      }
}
```

- range 也支持类似 `192.168.2.225-192.168.2.230/28` 形式的配置。
- 路由、网关、DNS 配置同 static IPAM plugin。

Whereabouts 配置可选参数：
- range_start: 指定地址范围内第一个进行分配的 IP
- range_end: 指定地址范围内最后一个进行分配的 IP
- exclude: 不进行 IP 分配的地址范围
- 更多配置选项：[Extended configuration](https://github.com/k8snetworkplumbingwg/whereabouts/blob/master/doc/extended-configuration.md)
