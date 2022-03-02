



添加 IPv6 默认路由后，我们从一台主机通过 curl ClusterIP 访问另一台主机上的 Service。

源主机发送数据包，进入 OUTPUT，OUT=enp0s5，如果没有配置默认路由，FIB 中没有路由信息，那么进不了 netfilter。

nat OUTPUT 中大致走了 nat OUTPUT，nat cali-OUTPUT，nat cali-fip-dnat，到底了一直没有做出 nat 动作，然后原路返回到 nat OUTPUT，匹配第二个 nat KUBE-SERVICES，走了 nat KUBE-SVC-5D2QDAMXQVK5QHQI 中第一条 nat KUBE-MARK-MASQ 中 MARK 了一下，然后匹配 nat KUBE-SVC-5D2QDAMXQVK5QHQI 第二条 nat KUBE-SEP-ZS3VF6UHHUUFLHHI，在其中进行 DNAT 操作，目标地址转换为 Service 对应的 Endpoint 的地址。

nat OUTPUT 之后是 filter OUTPUT，没有做出动作。之后是 POSTROUTING ，这里做了 MASQUERADE（如果是 Pod 间通信就不会 MASQUERADE，src IP 就是 源 Pod IP）。然后数据就从网卡发向目标主机。





进入目标主机后，数据报首先走 PREROUTING，全匹配完了没有符合的规则，按照 policy ACCEPT，默认接受。目的地址不是本机，因此走 FORWARD。根据 FIB 信息，OUT=cali6dc96cc5161。然后走了一系列 Calico 相关的链。最后 POSTROUTING 没有额外动作。数据报通过 veth pair 一端 cali6dc96cc5161 发向另一端，即 Pod 里的 eth0。







总结：

本机没有 IPv6 默认路由，本机进程例如 curl 请求无法进入 netfilter。Pod 通信没有这个问题。

Calico IPv6 Pod 间通信直接通过物理网卡。Calico IPv4 Pod 间通信走 tunl 隧道。

因为 IPv4 FIB 中有 10.244.104.0/26 via 10.211.55.13 dev tunl0 proto bird onlink。IPv6 FIB 中有 2001:db8:42:74:d03b:4785:4379:9540/122 via fdb2:2c26:f4e4:0:21c:42ff:fe1d:4460 dev enp0s5 proto bird metric 1024 pref medium。

