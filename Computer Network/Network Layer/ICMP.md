## ICMP
ICMP（Internet Control Message Protocol）是 TCP/IP 协议簇的一个子协议，用于在 IP 主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。例如，数据报大小超过设备的最大处理能力，设备就会丢弃该数据报并往回发送一个 ICMP 消息。又例如网关发现一个更短的路由路径，就会往回发送一个 ICMP 消息，数据报会被路由到更短的路径。

ICMP 协议是为了辅助 IP 协议，交换各种各样的控制信息而被制造出来的，它工作在网络层，是 IP 的组成部分，必须由每个 IP 模块实现。

ICMP 大致分成两种功能：差错通知和信息查询。


### ICMP 功能

1. MTU 探索：找到通信对方之间不用分片 IP 数据报，就能交流的 MTU 大小的功能。
2. 改变路由：路由器向送信方指示路径改变。源主机根据路由信息来决定传送目标，没有相应的路由信息就发给默认网关的路由器。默认网关接收到数据包，发现将数据包发给局域网内的其它路由器会比较快的时候，将这一信息通过 ICMP 通知发送方。
3. 源点抑制：数据包集中到达某一路由器后，数据包来不及被处理，有可能被丢弃。此时向送信方发送 ICMP 源点抑制报文，用来使送信方减慢发送速度。
4. ping
5. traceroute：通过发送不同 TTL 的数据报，然后凭借 ICMP 超时报文来确定路径信息。


### ICMP 数据包格式
ICMP 信息包含在 IP 数据报的数据部分，IP 数据报的协议字段值为 1。

ICMP 消息包含源数据报完整的首部，以使源主机定位出错的包。

一般格式
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      Type     |      Code     |            Checksum           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Pointer: Identifies the problem in the original IP message   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                          Message Body                         +
|                                                               |
```

Type: 消息类型，8 bits，常见有

1. Type 0 -- Echo reply
2. Type 3 -- Destination unreachable
3. Type 8 -- Echo
4. Type 5 -- Redirect

Code: 编码，8 bits，提供消息类型更详细的信息。

Pointer：指针，32 bits，指出原数据报中问题所在字节位置。

TYPE | CODE | Description | Query | Error
:-----:|:------:|:-------------|:-------:|:------:
0 | 0 | Echo Reply——回显应答（Ping应答） | x | 
3 | 0 | Network Unreachable——网络不可达 |  | x
3 | 1 | Host Unreachable——主机不可达 |  | x
3 | 2 | Protocol Unreachable——协议不可达 |  | x
3 | 3 | Port Unreachable——端口不可达 |  | x
3 | 4 | Fragmentation needed but no frag. bit set——需要进行分片但设置不分片比特 |  | x
3 | 5 | Source routing failed——源站选路失败 |  | x
3 | 6 | Destination network unknown——目的网络未知 |  | x
3 | 7 | Destination host unknown——目的主机未知 |  | x
3 | 8 | Source host isolated (obsolete)——源主机被隔离（作废不用） |  | x
3 | 9 | Destination network administratively prohibited——目的网络被强制禁止 |  | x
3 | 10 | Destination host administratively prohibited——目的主机被强制禁止 |  | x
3 | 11 | Network unreachable for TOS——由于服务类型TOS，网络不可达 |  | x
3 | 12 | Host unreachable for TOS——由于服务类型TOS，主机不可达 |  | x
3 | 13 | Communication administratively prohibited by filtering——由于过滤，通信被强制禁止 |  | x
3 | 14 | Host precedence violation——主机越权 |  | x
3 | 15 | Precedence cutoff in effect——优先中止生效 |  | x
4 | 0 | Source quench——源端被关闭（基本流控制） |  | 
5 | 0 | Redirect for network——对网络重定向 |  | 
5 | 1 | Redirect for host——对主机重定向 |  | 
5 | 2 | Redirect for TOS and network——对服务类型和网络重定向 |  | 
5 | 3 | Redirect for TOS and host——对服务类型和主机重定向 |  | 
8 | 0 | Echo request——回显请求（Ping请求） | x | 
9 | 0 | Router advertisement——路由器通告 |  | 
10 | 0 | Route solicitation——路由器请求 |  | 
11 | 0 | TTL equals 0 during transit——传输期间生存时间为0 |  | x
11 | 1 | TTL equals 0 during reassembly——在数据报组装期间生存时间为0 |  | x
12 | 0 | IP header bad (catchall error)——坏的IP首部（包括各种差错） |  | x
12 | 1 | Required options missing——缺少必需的选项 |  | x
13 | 0 | Timestamp request (obsolete)——时间戳请求（作废不用） | x | 
14 |  | Timestamp reply (obsolete)——时间戳应答（作废不用） | x | 
15 | 0 | Information request (obsolete)——信息请求（作废不用） | x | 
16 | 0 | Information reply (obsolete)——信息应答（作废不用） | x | 
17 | 0 | Address mask request——地址掩码请求 | x | 
18 | 0 | Address mask reply——地址掩码应答 |  | 


### ICMPv6
ICMPv6 除了与 ICMP 相同的功能外，还有基于 ICMPv6 实现的邻居发现，包括：地址解析、重复地址检测、路由发现、Path MTU发现以及重定向等功能。

ICMPv6 的协议号为 58，也就是在 IPv6 报文中的 Next Header 的值为58。