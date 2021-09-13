### ip
- `ip neigh` 查看 arp table
- `ip route get <ip>`: ask the kernel to report the route it would use to send a packet to the specified address (也是 gateway)
- `ip link`: MAC address
- `ip route list`: display all the IP addresses with their device names that are currently available
- `ip maddr show ens33` 主机网卡上 MAC 地址，使用 macvlan 主机网卡会有多个 MAC 地址