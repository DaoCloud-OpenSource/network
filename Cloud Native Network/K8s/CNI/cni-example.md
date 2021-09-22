## 实例
下面使用单个 CNI 插件以及链式调用插件分别配置 Docker 容器网络。

> 本实验使用的是笔者自己制作的镜像，监听在 8080 端口，收到 GET 请求，返回请求的 IP 地址、端口以及容器的 IP 地址、端口。

目录结构

```
$ ls *
bridge.json  portmap.conflist

bin:
bandwidth  dhcp      host-device  ipvlan    macvlan  ptp  static  vlan
bridge     firewall  host-local   loopback  portmap  sbr  tuning  vrf
```

cni 相关信息文件存在于 `/var/lib/cni` 目录下

### 单个插件 bridge
- 指定 `--net=none` 创建了容器，容器所在网络命名空间只有 loopback 设备。

```sh
$ docker run -d --rm --name http-test --net=none  registry.cn-hangzhou.aliyuncs.com/ericwvi/http-test
71ad9b005af450ee7d40e0959b6b340f0c929824cd9ae7a2f6f3c1f5a369241d

$ docker inspect -f '{{.State.Pid}}' 71a
23986

$ nsenter -t 23986 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

- 使用 bridge 插件为容器创建网络接口，并连接到主机网桥。

```sh
$ cat > bridge.conf << EOF
{
    "cniVersion": "0.4.0",
    "name": "mybridge",
    "type": "bridge",
    "bridge": "mybridge0",
    "isDefaultGateway": true,
    "forceAddress": false,
    "ipMasq": true,
    "hairpinMode": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.10.0.0/16"
    }
}
EOF

$ contid=71ad9b005af450ee7d40e0959b6b340f0c929824cd9ae7a2f6f3c1f5a369241d # 容器ID
$ pid=23986
$ netnspath=/proc/$pid/ns/net
$ CNI_COMMAND=ADD CNI_CONTAINERID=$contid CNI_NETNS=$netnspath CNI_IFNAME=eth0 CNI_PATH=./bin ./bin/bridge < bridge.json
{
    "cniVersion": "0.4.0",
    "interfaces": [
        {
            "name": "mybridge0",
            "mac": "e2:7e:f2:88:bf:3a"
        },
        {
            "name": "veth495fddad",
            "mac": "5a:1e:2d:ff:b5:2f"
        },
        {
            "name": "eth0",
            "mac": "5a:79:9c:b9:76:b6",
            "sandbox": "/proc/23986/ns/net"
        }
    ],
    "ips": [
        {
            "version": "4",
            "interface": 2,
            "address": "10.10.0.6/16",
            "gateway": "10.10.0.1"
        }
    ],
    "routes": [
        {
            "dst": "0.0.0.0/0",
            "gw": "10.10.0.1"
        }
    ],
    "dns": {}
}

$ nsenter -t 23986 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 5a:79:9c:b9:76:b6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.10.0.6/16 brd 10.10.255.255 scope global eth0
       valid_lft forever preferred_lft forever

$ ip a
4: mybridge0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether e2:7e:f2:88:bf:3a brd ff:ff:ff:ff:ff:ff
    inet 10.10.0.1/16 brd 10.10.255.255 scope global mybridge0
       valid_lft forever preferred_lft forever
    inet6 fe80::e07e:f2ff:fe88:bf3a/64 scope link
       valid_lft forever preferred_lft forever
5: veth495fddad@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master mybridge0 state UP group default
    link/ether 5a:1e:2d:ff:b5:2f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::581e:2dff:feff:b52f/64 scope link
       valid_lft forever preferred_lft forever
```

- 主机 veth 设备 `veth495fddad@if2` 中 2 对应容器命名空间里的 2 号设备。容器命名空间里 `eth0@if5` 中 5 对应主机的 5 号设备。

- 删除设备

```
CNI_COMMAND=DEL CNI_CONTAINERID=$contid CNI_NETNS=$netnspath CNI_IFNAME=eth0 CNI_PATH=./bin ./bin/bridge < bridge.json
```

- 删除网桥 `ip link delete mybridge0 type bridge`


### 链式调用 bridge & portmap
- 为了避免配置文件格式转换的繁琐工作，使用 cnitool 进行链式调用的配置 `go install github.com/containernetworking/cni/cnitool@latest`

```
$ docker run -d --rm --name http-test --net=none  registry.cn-hangzhou.aliyuncs.com/ericwvi/http-test
5057c3f9742ed088b0e5fdc1664417a1b365c653b00e08d5a5027417f6c710a1

$ docker inspect -f '{{.State.Pid}}' 505
9084

$ nsenter -t 9084 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

- 配置文件：设置 bridge 及 端口映射，文件后缀为 `.conflist`。

```
cat > portmap.conflist << EOF
{
  "cniVersion": "0.4.0",
  "name": "portmap",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "mynet0",
      "isDefaultGateway": true, 
      "forceAddress": false, 
      "ipMasq": true, 
      "hairpinMode": true,
      "ipam": {
        "type": "host-local",
        "subnet": "10.20.0.0/16",
        "gateway": "10.20.0.1"
      }
    },
    {
      "type": "portmap",
      "runtimeConfig": {
        "portMappings": [
          {"hostPort": 8080, "containerPort": 8080, "protocol": "tcp"}
        ]
      }
    }
  ]
}
EOF
```

- 配置

```
$ contid=5057c3f9742ed088b0e5fdc1664417a1b365c653b00e08d5a5027417f6c710a1 # 容器ID
$ pid=9084
$ netnspath=/proc/$pid/ns/net
$ CNI_PATH=./bin NETCONFPATH=.  cnitool add portmap $netnspath
{
    "cniVersion": "0.4.0",
    "interfaces": [
        {
            "name": "mynet0",
            "mac": "e2:1c:14:e8:0f:7a"
        },
        {
            "name": "vethd7fca858",
            "mac": "92:6d:68:de:3b:46"
        },
        {
            "name": "eth0",
            "mac": "16:b8:13:d8:2d:63",
            "sandbox": "/proc/9084/ns/net"
        }
    ],
    "ips": [
        {
            "version": "4",
            "interface": 2,
            "address": "10.10.0.3/16",
            "gateway": "10.10.0.1"
        }
    ],
    "routes": [
        {
            "dst": "0.0.0.0/0",
            "gw": "10.10.0.1"
        }
    ],
    "dns": {}
}
```

- 访问验证

```
$ nsenter -t 9084 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 16:b8:13:d8:2d:63 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.10.0.3/16 brd 10.10.255.255 scope global eth0
       valid_lft forever preferred_lft forever

$ nsenter -t 9084 -n ss -nultp
Netid      State       Recv-Q      Send-Q            Local Address:Port             Peer Address:Port      Process
tcp        LISTEN      0           4096                          *:8080                        *:*          users:(("http-test",pid=9084,fd=3))

$ curl 127.0.0.1:8080
Hello, 10.10.0.1:59732! I'm 10.10.0.3:8080, 2021-09-16 08:59:08.05518945 +0000 UTC m=+421.494357499
```

- 删除网络

```
CNI_PATH=./bin NETCONFPATH=.  cnitool del portmap $netnspath
```

- 删除网桥 `ip link delete mynet0 type bridge`
