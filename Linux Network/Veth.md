### Veth
- 所有网络设备只能属于一个命名空间。
- 不同的网络空间之间相互隔离，彼此无法通信，在协议栈内部看不到对方。不同命名空间中的网络相互通信，并与外部网络进行通信，可以使用 Veth 设备对。Veth 设备对就像一条管道，连接着不同网络命名空间的协议栈。
- 新生成的网络命名空间默认只有回环设备（lo），且为停止状态，其它设备需要一一手动建立，这些 docker 都帮我们配置好了。
- 一些网络设备支持在命名空间之间移动。 
- 如果 `ethtool -k enp0s5 |grep netns-local` 值为 on 则说明不可以转移。

### 实例

- 创建命名空间 `ip netns add <name>`
- 命名空间运行命令 `ip netns exec <name> <command>`
- 创建 Veth 设备对: `ip link add veth0 type veth peer name veth1`
- 将 Veth 转移到命名空间：`ip link set veth1 netns netns1`
- 分配 ip：`ip netns exec netns1 ip addr add 10.1.1.1/24 dev veth1`, `ip addr add 10.1.1.2/24 dev veth0`
- 启动 Veth：`ip netns exec netns1 ip link set dev veth1 up`, `ip link set dev veth0 up`
- 两个命名空间可以相互通信：`ip netns exec netns1 ping 10.1.1.2`, `ping 10.1.1.1`
- 查看对端：`ip netns exec netns1 ethtool -S veth1`
```sh
NIC statistics:
    peer_ifindex: 5
```
- 另一端接口设备序列号为 5，`ip netns exec netns2 ip link | grep 5` 得到 veth0


- 运行一个 busybox 容器， Pid 为 1503
- 查看设备序列号，可以看到 4 号设备与 5 号设备为 Veth 设备对。
- Veth 设备对一个在容器的网络命名空间，一个在主机命名空间。位于主机的 Veth 设备作为网口在链路层工作，接上网桥不再需要 IP 地址。因此主机的 Veth 设备没有显示 IP 地址。
```
主机
$ ip a
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:38:b7:ae:7f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:38ff:feb7:ae7f/64 scope link
       valid_lft forever preferred_lft forever
5: veth30314cf@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 2e:a1:2d:73:79:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::2ca1:2dff:fe73:7903/64 scope link

$ ethtool -S veth30314cf
NIC statistics:
     peer_ifindex: 4
     rx_queue_0_xdp_packets: 0
     rx_queue_0_xdp_bytes: 0
     rx_queue_0_xdp_drops: 0

$ nsenter -t 1503 -n ip link
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

容器内
$ ip a
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```