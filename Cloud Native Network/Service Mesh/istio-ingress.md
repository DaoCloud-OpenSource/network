Kubernetes Ingress 针对 L7，可定义路由规则
Istio Ingress 针对 L4-L6，只定义接入点，把所有路由规则交给 VirtualService。虚拟服务可以重用，和其他定义解耦。