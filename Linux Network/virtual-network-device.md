## 虚拟网络设备
如今云时代的网络，到处都是虚拟机和容器，它们背后的网络管理都离不开虚拟网络设备。

Linux 下的网络设备 net_dev 并不一定都对应实际的硬件设备，只要注册一个 struct net_device{} 结构体（netdevice.h）到内核中，那么这个网络设备就存在了。该结构体很庞大，其中包含设备的协议地址（对于 IP 即 IP 地址），这样它就能被网络层识别，并参与路由系统，最有名的当数 loopback 设备。