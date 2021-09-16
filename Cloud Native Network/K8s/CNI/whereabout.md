[whereabouts]是一种`IPAM`工具，`IPAM`是IP分配管理工具，kubernetes的CRI调用CNI为pod分配IP地址时，CNI插件需要调用IPAM插件获取IP。whereabouts优势在于可以跨域整个集群的节点，为整个网络环境回收IP和分配IP，而且支持IP4和IP6协议，它更像是为kubernetes开发的。

#### whereabouts安装
它提供使用kubernetes安装方式，非常的简单：

```shell
git clone https://github.com/dougbtv/whereabouts && cd whereabouts
       kubectl apply \
         -f doc/crds/daemonset-install.yaml \
         -f doc/crds/whereabouts.cni.cncf.io_ippools.yaml \
         -f doc/crds/whereabouts.cni.cncf.io_overlappingrangeipreservations.yaml \
         -f doc/crds/ip-reconciler-job.yaml
```

### macvlan和whereabouts的简单使用
`macvlan`是CNI原生的插件，不需要再额外安装，安装CNI后可以在/opt/cni/bin/目录下进行查看：

```shell
[root@k8s-master01 bin]# ls /opt/cni/bin/
bandwidth   dhcp   flannel   host-local   loopback   portmap   sbr   tuning   whereabouts
bridge   firewall   host-device   ipvlan   macvlan   ptp   static   vlan
```

先清除其他的网络插件，然后删除/etc/cni/net.d/的其他网络插件的文件。然后新建文件10-macvlannet.conf，它是一个json格式的文件：

```json
{
    "cniVersion": "0.3.0",
    "name": "whereaboutsexample",
    "type": "macvlan",
    "master": "ens192",
    "mode": "bridge",
    "ipam": {
        "type": "whereabouts",
        "range": "10.6.212.10/28"
    }
}
```

注意：这个配置文件并不是调用CNI插件的配置文件，libcni包会解析这个文件，然后转化为调用CNI插件的配置，有时候创建网络会链式调用多个CNI插件。这个json文件的格式要求极为严格，复制粘贴的时候请格外小心tab和空格。

whereabouts核心参数：

* type：必须设置为whereabouts

* range：使用IP的CIDR的表示法

whereabouts可选参数：

* range_start：从设置的地址开始

* range_end：从设置的地址结束

* exclude：从分配中排出的CIDR列表

设置完成后请重新启动kubelet，然后我们在环境中运行两个Pod在不同的node上，查看IP地址。

[whereabouts]: https://github.com/k8snetworkplumbingwg/whereabouts