## CNI
CNI（Container Network Interface） 是一个抽象的接口层，将容器网络配置方案与容器平台方案解耦。它定义了一套接口标准，提供了规范文档以及一些标准实现。采用 CNI 规范来设置容器网络的容器平台不需要关注网络的设置的细节，只需要按 CNI 规范来调用 CNI 接口即可实现网络的设置。

CNI 的接口并不是指 HTTP，gRPC 接口，CNI 接口是指对可执行程序的调用（exec)。这些可执行程序称之为 CNI 插件，以 K8S 为例，K8S 节点默认的 CNI 插件路径为 `/opt/cni/bin`。

容器运行时通过设置环境变量以及从标准输入传入的配置文件来向插件传递参数。CNI 通过 JSON 格式的配置文件来描述网络配置，当需要设置容器网络时，由容器运行时负责执行 CNI 插件，并通过 CNI 插件的标准输入（stdin）来传递配置文件信息，通过标准输出（stdout）接收插件的执行结果。

### 环境变量
- CNI_COMMAND: 定义期望的操作，可以是ADD，DEL，CHECK 或 VERSION。
- CNI_CONTAINERID: 容器 ID，由容器运行时管理的容器唯一标识符。
- CNI_NETNS: 容器网络命名空间的路径。（如 `/run/netns/[nsname]` )。
- CNI_IFNAME: 需要被创建的网络接口名称，例如 eth0。
- CNI_ARGS: 运行时调用时传入的额外参数，格式为分号分隔的key-value对，例如 FOO=BAR;ABC=123
- CNI_PATH: CNI 插件可执行文件的路径，如 `/opt/cni/bin`。