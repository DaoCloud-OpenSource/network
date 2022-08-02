1. VMware Common Networking Configurations:
  - host-only: 只有 host OS 能连接到，无法访问外网
  - bridge： 独立 ip，和 host OS 不同，可以理解为室内路由器下的一个新 ip
  - NAT: 使用 host OS 的 ip 地址，进行 NAT 转换
2. 本机网络分析：
  - Win 在公司 wi-fi: 172.30.2.28/21, Default Gateway: 172.30.0.1
  - VMware NAT 对应的 adapter VMnet8: 192.168.160.1/24, VMware 网关以及DNS 在 192.168.160.2， K8S master 在 192.168.160.11， node1 在 192.168.160.12， node2 在 192.168.160.13
  - WSL: 本机为 172.18.80.1/20, WSL2 为 172.18.80.183/20
  - WSL、master、node1、node2 之间互相 ping， 都可以通, 本机 ping 它们也可以 ping 通， 前面几个走本机代理， 使用 172.18.80.1：7890
  - ssh 登录虚拟机，显示来源 ip 为 192.168.160.1


## 相关概念
### Service Discovery
- Service discovery is the process of automatically detecting devices and services on a network. Service discovery protocol (SDP) is a networking standard that accomplishes detection of networks by identifying resources.
- 避免了配置的麻烦
- 例如 DHCP
- 
- 三个组成部分： Service Registry, Service Provider, Service Consumer

### Link-local Address
- In computer networking, a link-local address is a network address that is valid only for communications within the network segment or the broadcast domain that the host is connected to. 主机相互通信使用。这类主机通常不需要外部互联网服务，仅有主机间相互通讯的需求。

### round-robin DNS
- responding to DNS requests not only with a single potential IP address, but with a list of potential IP addresses corresponding to several servers that host identical services.
- The order in which IP addresses from the list are returned is the basis for the term round robin.
- With each DNS response, the IP address sequence in the list is permuted. Traditionally, IP clients initially attempt connections with the first address returned from a DNS query, so that on different connection attempts, clients would receive service from different providers, thus distributing the overall load among servers.
- Some resolvers attempt to re-order the list to give priority to numerically "closer" networks. This behavior was standardized during the definition of IPv6, and has been blamed for defeating round-robin load-balancing. Some desktop clients do try alternate addresses after a connection timeout of up to 30 seconds.
- Round-robin DNS is often used to load balance requests among a number of Web servers. For example, a company has one domain name and three identical copies of the same web site residing on three servers with three IP addresses. The DNS server will be set up so that domain name has multiple A records, one for each IP address. When one user accesses the home page it will be sent to the first IP address. The second user who accesses the home page will be sent to the next IP address, and the third user will be sent to the third IP address. In each case, once the IP address is given out, it goes to the end of the list. The fourth user, therefore, will be sent to the first IP address, and so forth.
- 缺点： DNS 体系本身以及客户端两边的缓存可能无法及时识别失效的节点