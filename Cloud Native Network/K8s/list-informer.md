Etcd存储集群的数据信息，apiserver作为统一入口，任何对数据的操作都必须经过 apiserver。客户端(kubelet/scheduler/controller-manager)通过 list-watch 监听 apiserver 中资源(pod/rs/rc等等)的 create, update 和 delete 事件，并针对事件类型调用相应的事件处理函数。

那么list-watch 具体是什么呢，顾名思义，list-watch有两部分组成，分别是list和 watch。list 非常好理解，就是调用资源的list API罗列资源，基于HTTP短链接实现；watch则是调用资源的watch API监听资源变更事件，基于HTTP 长链接实现，也是本文重点分析的对象。以 pod 资源为例，它的 list 和watch API 分别为： [List API](https://v1-10.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#list-all-namespaces-63)，返回值为 [PodList](https://v1-10.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#podlist-v1-core)，即一组 pod`。

Watch API，往往带上 watch=true，表示采用 HTTP 长连接持续监听 pod 相关事件，每当有事件来临，返回一个 WatchEvent。

HTTP 分块传输编码允许服务器为动态生成的内容维持 HTTP 持久链接。通常，持久链接需要服务器在开始发送消息体前发送Content-Length消息头字段，但是对于动态生成的内容来说，在内容创建完之前是不可知的。使用分块传输编码，数据分解成一系列数据块，并以一个或多个块发送，这样服务器可以发送数据而不需要预先知道发送内容的总大小。

当客户端调用 watch API 时，apiserver 在response 的 HTTP Header 中设置 Transfer-Encoding的值为chunked，表示采用分块传输编码，客户端收到该信息后，便和服务端该链接，并等待下一个数据块，即资源的事件信息。

[理解 K8S 的设计精髓之 List-Watch机制和Informer模块](https://www.jianshu.com/p/234d27d5c1c1?utm_campaign=maleskine&utm_content=note&utm_medium=reader_share&utm_source=weixin)