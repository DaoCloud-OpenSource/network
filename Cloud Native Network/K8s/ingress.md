## Ingress
Service 表现形式为 ClusterIP:Port，工作在 Layer 4，应用层的转发机制无法实现。Ingress 除了可以实现应用层的转发，还可以节省宝贵的静态 IP 资源。Ingress 相当于提供配置规则，真正起到作用的是 Ingress Controller。

Ingress Controller 是一个运行在集群中的 Pod，它会解析 Ingress 对象提供的规则，将请求重定向到内部其他服务。Ingress Controller 本身也通过 Service 暴露到集群之外，最常见的是通过 LoadBalancer 来实现。

Ingress 的配置相比于直接配置反向代理更加容易、更加智能、更容易管理。

Ingress 配置中，只能重定向到同一命名空间的服务。

### 配置
示例

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80

```

#### Path Type
路由规则的 path 必须设置相应的 `pathType`。`pathType` 有三种：`ImplementationSpecific`, `Exact`, `Prefix`。

如果没有与请求相匹配的规则，则会将请求转发给 default backend。default backend 是 Ingress controller 的配置选项，并不包含在 Ingress 当中。

如果有多个匹配的规则，则长度更长的规则有更高的优先级，规则相同长度的情况下，exact path 比 prefix path 有更高的优先级。


#### Hostname wildcards
规则中 host 字段可以是精确的匹配，也可以是一个 wildcard，例如 `*.foo.com`，它可以匹配 `bar.foo.com`，但不可以匹配 `foo.com` 和 `baz.bar.foo.com`。

wildcard 只可以匹配一个 DNS label。

#### Ingress Class
在 1.18 之前，Ingress class 由注解指定。

```yaml
kind: Ingress
metadata:
  name: ingress
  annotations:
      kubernetes.io/ingress.class: nginx
...
```

在 1.18 中引入了 `IngressClass` 资源和 `ingressClassName` 字段。

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb
spec:
  controller: example.com/ingress-controller
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb
```

`parameters` 字段是可选的，用来存放和特定 ingress controller 相关的配置。


### TLS
可以通过创建一个 Secret 来保存私钥和证书。

Ingress 只支持一个 443 的 TLS 端口，并且 TLS Termination 发生在 Ingress controller。

TLS 对于默认规则不会生效。

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```




如果你定义了多个 Ingress yaml 配置，那么这些配置会被一个单一的Ingress 控制器合并成一个 Nginx 配置。也就是说所有的人都在使用同一个 LoadBalancer IP。




有时候我们需要对 Ingress Nginx 进行一些微调配置，我们可以通过 Ingress 资源对象中的 annotations 注解来实现，比如我们可以配置各种平时直接在 Nginx 中的配置选项。

```yaml
kind: Ingress
metadata:
  name: ingress
  annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/proxy-connect-timeout: '30'
      nginx.ingress.kubernetes.io/proxy-send-timeout: '500'
      nginx.ingress.kubernetes.io/proxy-read-timeout: '500'
      nginx.ingress.kubernetes.io/send-timeout: "500"
      nginx.ingress.kubernetes.io/enable-cors: "true"
      nginx.ingress.kubernetes.io/cors-allow-methods: "*"
      nginx.ingress.kubernetes.io/cors-allow-origin: "*"
...
```

此外也可以做更细粒度的规则配置，如下所示：

```yaml
nginx.ingress.kubernetes.io/configuration-snippet: |
  if ($host = 'www.qikqiak.com' ) {
    rewrite ^ https://qikqiak.com$request_uri permanent;
  }
```

这些注释都将被转换成 Nginx 配置，你可以通过手动连接(kubectl exec)到 nginx pod 中检查这些配置。




查看 ingress-nginx 日志
要排查问题，通过查看 Ingress 控制器的日志非常有帮助。



使用 Curl 测试
如果我们想测试 Ingress 重定向规则，最好使用 curl -v [yourhost.com](http://yourhost.com) 来代替浏览器，可以避免缓存等带来的问题。



SSL/HTTPS
可能我们想让网站使用安全的 HTTPS 服务，Kubernetes Ingress 也提供了简单的 TLS 校验，这意味着它会处理所有的 SSL 通信、解密/校验 SSL 请求，然后将这些解密后的请求发送到内部服务去。
如果你的多个内部服务使用相同（可能是通配符）的 SSL 证书，这样我们就只需要在 Ingress 上配置一次，而不需要在内部服务上去配置，Ingress 可以使用配置的 TLS Kubernetes Secret 来配置 SSL 证书。

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
    - sslexample.foo.com
    secretName: testsecret-tls
  rules:
    - host: sslexample.foo.com
      http:
        paths:
        - path: /
          backend:
            serviceName: service1
            servicePort: 80
```

不过需要注意的是如果你在不同的命名空间有多个 Ingress 资源，那么你的 TLS secret 也需要在你使用的 Ingress 资源的所有命名空间中可用。




Example: [Manifest File](./ingress.yaml)

```yaml
---
# Config
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-test
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-ingress-controller-conf
  labels:
    app: nginx-ingress-lb
  namespace: ingress
data:
  enable-vts-status: 'true'
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx
  namespace: ingress
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: nginx-role
  namespace: ingress
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - update
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - extensions
    resources:
      - ingresses/status
    verbs:
      - update
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: nginx-role
  namespace: ingress
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-role
subjects:
  - kind: ServiceAccount
    name: nginx
    namespace: ingress
---
# backend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: default-backend
  namespace: ingress
spec:
  replicas: 1
  selector: 
    matchLabels:
      app: default-backend
  template:
    metadata:
      labels:
        app: default-backend
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: default-backend
        image: gcr.io/google_containers/defaultbackend:1.0
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
          timeoutSeconds: 5
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 10m
            memory: 20Mi
          requests:
            cpu: 10m
            memory: 20Mi
---
apiVersion: v1
kind: Service
metadata:
  name: default-backend
  namespace: ingress
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: default-backend
---
# Ingress Controller
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress-lb
  revisionHistoryLimit: 3
  template:
    metadata:
      labels:
        app: nginx-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      serviceAccount: nginx
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.9.0
          imagePullPolicy: Always
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
          livenessProbe:
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            timeoutSeconds: 5
          args:
            - /nginx-ingress-controller
            - --default-backend-service=$(POD_NAMESPACE)/default-backend
            - --configmap=\$(POD_NAMESPACE)/nginx-ingress-controller-conf
            - --v=2
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - containerPort: 80
            - containerPort: 18080
---
# a Service that exposes the ingress controller to the outside world
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: ingress
spec:
  type: NodePort
  ports:
    - port: 80
      name: http
      nodePort: 32000      
    - port: 18080
      name: http-mgmt
  selector:
    app: nginx-ingress-lb

---
# standard Deployments and a Service
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
      app: hollowapp
  name: hollowapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hollowapp
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: hollowapp
    spec:
      containers:
      - name: hollowapp
        image: theithollow/hollowapp-blog:allin1-v2
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
        env:
        - name: SECRET_KEY
          value: "my-secret-key"
---
apiVersion: v1
kind: Service
metadata:
  name: hollowapp
  labels:
    app: hollowapp
spec:
  type: ClusterIP
  ports:
  - port: 5000
    protocol: TCP
    targetPort: 5000
  selector:
    app: hollowapp
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress #ingress resource
metadata:
  name: hollowapp
  labels:
    app: hollowapp
spec:
  rules:
  - host: hollowapp.hollow.local #only match connections to hollowapp.hollow.local. 
    http:
      paths:
      - path: / #root path
        backend:
          serviceName: hollowapp
          servicePort: 5000

```