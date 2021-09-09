## SDN
Software-defined networking (SDN) 技术是一种网络管理方法，它支持动态的、编程高效的网络配置，以改进网络性能和监控，使其相比传统网络管理，更像云计算。

SDN 旨在解决传统网络去中心化的复杂的静态体系结构。当前网络需要更多的灵活性，排查故障需要变得更加容易。SDN 试图通过将网络数据包的转发过程（转发平面 forwarding plane）与路由过程（控制平面 control plane）分离，将网络智能集中在一个网络组件中。

控制平面由一个或多个控制器组成，这些控制器被视为 SDN 网络的大脑，所有的网络智能都包含其中。

然而，网络智能集中在一个组件在安全性、可扩展性和弹性方面也有其自身的缺点。软件的多层次和处理过程会增加性能开销，会使得网络更加复杂。封包、解包需要大量的算力。物理网络也无法根据虚拟网络中的改变自动调整。



### 参考
1. [Software-defined networking - Wikipedia](https://en.wikipedia.org/wiki/Software-defined_networking)
2. [浅谈SDN](https://zhuanlan.zhihu.com/p/144676870)