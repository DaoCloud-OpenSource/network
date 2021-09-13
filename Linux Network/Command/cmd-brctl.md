### 网桥
- 新增：`brctl addbr <name>`
- 将网卡连接到网桥：`brctl addif <name> ethx`
- 网卡作为网口在链路层工作，接上网桥不再需要 IP 地址，IP 地址自然失效 `ifconfig ethx 0.0.0.0`
- 网桥配置 IP `ifconfig brxxx xxx.xxx.xxx.xxx`