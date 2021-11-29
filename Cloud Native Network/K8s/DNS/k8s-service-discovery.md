## 服务发现
### 环境变量
Pod 被创建的时候，kubelet 会在该 Pod 中注入集群内所有 Service 的相关环境变量。

要想一个 Pod 中注入某个 Service 的环境变量，则 Service 必须先于该 Pod 创建。这一点，几乎使得这种方式进行服务发现不可用。

```bash
REDIS_MASTER_SERVICE_HOST=172.16.50.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://172.16.50.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://172.16.50.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=172.16.50.11
```

### DNS
通过 DNS 进行服务发现。

`http-service.default.svc.cluster.local.` 是一个 FQDN [^1]，也称 absolute domain name。（注意最后的 `.`）

以一个 Pod 的 `/etc/resolv.conf` 中内容进行介绍：

```conf
nameserver 2001:db8:42:1::a
search default.svc.cluster.local svc.cluster.local cluster.local mydomain otherdomain
options ndots:5
```

nameserver 地址为 DNS Service 的地址: `kube-dns.kube-system.svc.cluster.local`。客户端会根据 search list 来发起请求，首先是请求 `http-service.default.svc.cluster.local.`，如果不存在则请求 `http-service.svc.cluster.local.`，再次不存在在依次请求 `http-service.cluster.local.`、`http-service.mydomain.`、`http-service.otherdomain.`，都不匹配的情况下将其当作 FQDN 进行请求，即 `http-service.`。

ndots 则是指定匹配 search list 中后缀的个数。值为 5，代表每一个都要匹配，这会加大 DNS Server 的压力。在大量外部域名解析的场景下，建议 Pod 配置 ndots,例如取值为 2。

```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  ...
  dnsConfig:
    nameservers:
      - 1.2.3.4
    searches:
      - ns1.svc.cluster-domain.example
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
```




[^1]: Fully Qualified Domain Name