# 43、Kubernetes流量调度Ingress

# Ingress 原理

```
https://kubernetes.github.io/ingress-nginx/
https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/
https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/
```



## Ingress 工作原理

Kubenetes集群中的通信的不仅有内部的流量，还有外部流量

流量方向

- 将集群内部 pod之类对象之间的网络通信称为东西向流量

- 将集群外部应用和集群内部之间对象 之间的网络通信称为南北向流量

  - 从北到南（入）↓ -- ingress，从集群外到集群内

  - 从南到北（出）↑ -- egress ，从集群内到集群外

可以基于四层的两种常用的方式实现将集群外部的流量引入到集群的内部中来，从而实现外部客户的正常访问。

- service方式：nodePort、LoadBalancer、externalIP 等service对象方式，借助于 namespace、iptables、ipvs 等四层反向代理实现流量的转发
- Host方式：通过node节点的 hostNetwork 与 hostPort 及配套的port映射，以间接的方式实现service的效果

kubernetes网络

![image-20250405162307539](kubernetes/image-20250405162307539.png)

四层方式存在的问题

- 基于service或host等多种方式实现集群内外的网络通信都是基于四层协议来进行调度的，而且应用级别的健康检查功能很难实现。
- 对于外部流量的接入，一般都是基于http(s)协议通信，四层协议无法实现基于应用层的代理,尤其是涉及到各种ssl会话的管理
- 不支持基于FQDN的方式进行访问应用
- 不支持基于URL等机制对HTTP/HTTPS协议进行高级路由、超时/重试、基于流量的灰度等高级流量
- 难以将多个Service流量统一管理,虽然service能够实现将外部流量引入到集群内部，但是其本质上，是通过网络规则方式来进行转发的，将集群内部的服务暴露到外部。这会造成对外过多的地址和端口暴露

![image-20250405162501165](kubernetes/image-20250405162501165.png)

虽然可以自行通过部署七层代理解决上述问题,但配置繁琐以及无法和Kubernetes集群更好的结合联动.

因此Kubernetes 引入了一种新的资源 ingress，可以解决上面提出问题：

- Ingress本质就是七层代理, 所以可以基于http/https的方式，将集群外部的流量统一的引入到集群内部
- 通过一个统一的流量入口，避免将集群内部大量的端口直接暴露给外部

Ingress公开从集群外部到集群内服务 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。

下面是一个将所有流量都发送到同一 Service 的简单 Ingress 示例：



```mermaid
graph LR;
  client([客户端])-. Ingress-管理的 <br> 负载均衡器 .->ingress[Ingress];
  ingress-->|路由规则|service[Service];
  subgraph 集群
  ingress;
  service-->pod1[Pod];
  service-->pod2[Pod];
  end
  classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
  classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
  classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
  class ingress,service,pod1,pod2 k8s;
  class client plain;
  class cluster cluster;
```



Ingress 可为 Service 提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及基于名称的虚拟托 管。 Ingress 控制器 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

Ingress 不会公开任意端口或协议。 将 HTTP 和 HTTPS 以外的服务公开到 Internet 时，通常使用Service.Type=NodePort 或 Service.Type=LoadBalancer 类型的 Service。

Ingress这种利用应用层协议来进行流量的负载均衡效果，它可以实现让用户通过域名来访问相应的service就可以了，无需关心Node IP及Port是什么，避免了信息的泄露。

|        **组件**        |       **类型**        |                         **核心功能**                         |           **部署方式**            |  **工作层级**  |         **配置方式**          |
| :--------------------: | :-------------------: | :----------------------------------------------------------: | :-------------------------------: | :------------: | :---------------------------: |
|    **Ingress API**     | Kubernetes标准API资源 | - 定义抽象路由规则（Host/Path→Service映射） - 声明流量调度和路由规则 | 通过kubectl apply创建YAML资源对象 | 命名空间级资源 |   每个服务独立YAML配置文件    |
| **Ingress Controller** | 七层反向代理服务程序  | - 监听APIServer的Ingress变动 - 生成反向代理配置并动态加载 - 实际转发流量 | 以Pod形式部署在集群中，需独立安装 |   集群级服务   | 自动转化Ingress规则为代理配置 |

因为ingress Controller是以pod的方式部署的,所以需要解决如下问题

- ingress的pod如何引入外部流量

  通过一个专用的service 即可实现

- 如何实现ingress的Pod的流量负载均衡

  关于pod负载均衡的流量，直接通过deployment/daemonset等controller转发给后端pod即可。

- 后端应用的 Pod 很多，如何找到要转发的目标？

  通过k8s的service对所有的pod进行分组管理，再用controller内部的负载均衡配置，找到对应的目标。

  即后端应用的Pod对应的service 只是起到服务发现Pod的功能，而从外部访问应用的Pod的流量转发过程中不需要再经过此service

**Ingress 访问过程**

- 从外部流量调度到kubernetes中Ingress service，有多种实现方案，比如使用节点网络中的EXTERNAL-IP或者NodePort方式
- 从service调度到ingress-controller
- ingress-controller根据ingress Pod 中的定义，比如虚拟主机或者后端的url
- 根据虚拟主机名直接调度到后端的一组应用pod中

![image-20250405163617573](kubernetes/image-20250405163617573.png)

注意：

- 整个流程中涉及到了两处service内容
- service ingress-nginx 是帮助 ingress controller Pod 接入外部流量的
- 后端的服务对应的service 只起到帮助 ingress controller Pod 找到具体的服务的Pod，即只用于服务发现 ，而流量不需要经过后端服务的Service，直接从ingress controller Pod转到至具体的Pod
- 虚线表示service对后端的应用进行分组，实线表示ingress实际的访问流向

## Ingress controller 常见的解决方案

Ingress controller常见的实现方式有：

```
https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
```



- Kubernetes Ingress

  Kubernetes官方发布, Ingress-Nginx 自从k8s-v1.6版本引入，在k8s-v1.19 正式进入了一个稳定版本状态。

  ```
  http://github.com/kubernetes/ingress-nginx
  ```



各种 Ingress Controller 比较

```
https://docs.google.com/spreadsheets/d/1DnsHtdHbxjvHmxvlu7VhzWcWgLAn_Mc5L1WlhLDA__k/edit?spm=a2c6h.12873639.article-detail.4.1eff3d8dO53Ifj#gid=0
```



# Ingress-nginx Controller 安装和配置

```
https://kubernetes.github.io/ingress-nginx/deploy/
```

```bash
[root@master1 ~]#wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/cloud/deploy.yaml
```

```bash
#修改镜像文件地址
[root@master1 ~]#grep image deploy.yaml 
        #image: registry.k8s.io/ingress-nginx/controller:v1.12.1@sha256:d2fbc4ec70d8aa2050dd91a91506e998765e86c96f32cffb56c503c9c34eed5b
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.12.1 
        imagePullPolicy: IfNotPresent
        #image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.5.2@sha256:e8825994b7a2c7497375a9b945f386506ca6a3eda80b89b74ef2db743f66a5ea
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.5.2 
        imagePullPolicy: IfNotPresent
        #image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.5.2@sha256:e8825994b7a2c7497375a9b945f386506ca6a3eda80b89b74ef2db743f66a5ea
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.5.2 
        imagePullPolicy: IfNotPresent

#service使用LoadBalancer，需要先部署LoadBalancer
[root@master1 ~]#grep type deploy.yaml 
  type: LoadBalancer
```

```bash
#2)开放外部访问入口地址
[root@master1 ~]#vim deploy.yaml
......
apiVersion: v1
kind: Service
metadata:
.......
  name: ingress-nginx-controller
  namespace: ingress-nginx
  annotations: #添加如下三行,用于支持prometheus监控,可选
    prometheus.io/scrape: "true"
    prometheus.io/port: "10254"
spec:
  externalTrafficPolicy: Local #默认为Local,只能本机处理流量，即下面externalIPs所在主机和Ingress-Controller Pod运行的主机必须是同一个节点，如果修改为Cluster，所有节点:3XXXX端口都能处理流量实现负载均衡，如果type是LoadBalancer类型并且已配置openelb类似相关服务,则可以配置为Local，避免跨主机的网络转发，从而有更好的性能，
.....
  type: LoadBalancer #默认值LoadBalancer可不做修改，即使没有配置Openelb也可以不做修改，也支持其它类型，比如：NodePort，ClusterIP
  externalIPs: ['10.0.0.99'] #如果type是LoadBalancer类型并且配置openelb则此项可选,增加一个或多个外网访问的入口IP，如果externalTrafficPolicy为Local则此IP必须是正在运行Ingress-Controller Pod的节点上面的可用IP,只有此IP能才处理流量,如果为Cluster则可以是任意集群节点的IP，此项和type无关
  ports:
    - name: http
    port: 80
.....
```

```bash
#3)默认ingress-nginx-controller只有一个Pod副本的,可选
#方法1: 指定2个副本实现高可用（此步可选）
---
apiVersion: apps/v1
kind: Deployment
...
spec:
  replicas: 2

#方法2可以修改为DaemonSet提高性能（此步可选）
apiVersion: apps/v1
#kind: Deployment
kind: DaemonSet #修改此行为 DaemonSet
.....
    spec:
      hostNetwork: true #使用宿主机的网络,直接使用宿主机的IP:80和宿主机的IP:443
      hostPID: true #使用宿主机的Pid
```

```bash
[root@master1 ~]#kubectl apply -f deploy.yaml 
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

```bash
[root@master1 ~]#kubectl get all -n ingress-nginx 
NAME                                            READY   STATUS              RESTARTS   AGE
pod/ingress-nginx-admission-create-v5qxc        0/1     Completed           0          49s
pod/ingress-nginx-admission-patch-l822v         0/1     Completed           2          49s
pod/ingress-nginx-controller-5db8d9c658-fdpzh   0/1     ContainerCreating   0          49s
pod/ingress-nginx-controller-5db8d9c658-gjnxm   0/1     Running             0          49s

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.106.79.228   10.0.0.10     80:30848/TCP,443:32630/TCP   50s
service/ingress-nginx-controller-admission   ClusterIP      10.107.87.231   <none>        443/TCP                      49s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   0/2     2            0           49s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-5db8d9c658   2         2         0       49s

NAME                                       STATUS     COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   Complete   1/1           19s        49s
job.batch/ingress-nginx-admission-patch    Complete   1/1           31s        49s
```

```bash
[root@master1 ~]#kubectl get po -o wide -n ingress-nginx 
NAME                                        READY   STATUS      RESTARTS   AGE     IP             NODE             NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-v5qxc        0/1     Completed   0          2m15s   10.244.4.207   node2.kang.org   <none>           <none>
ingress-nginx-admission-patch-l822v         0/1     Completed   2          2m15s   10.244.1.218   node1.kang.org   <none>           <none>
ingress-nginx-controller-5db8d9c658-fdpzh   1/1     Running     0          2m15s   10.244.1.219   node1.kang.org   <none>           <none>
ingress-nginx-controller-5db8d9c658-gjnxm   1/1     Running     0          2m15s   10.244.3.218   node3.kang.org   <none>           <none>
[root@master1 ~]#kubectl get ep -n ingress-nginx 
NAME                                 ENDPOINTS                                                       AGE
ingress-nginx-controller             10.244.1.219:443,10.244.3.218:443,10.244.1.219:80 + 1 more...   2m53s
ingress-nginx-controller-admission   10.244.1.219:8443,10.244.3.218:8443                             2m53s
```

** **

![image-20250405191400622](kubernetes/image-20250405191400622.png)

**微服务**

![image-20250405174130902](kubernetes/image-20250405174130902.png)





# Ingress 实现

### 命令实现

```bash
#创建Ingress的命令：
kubectl create ingress NAME --rule=domain/url=service:port[,tls[=secret]] [options]
#常用选项
--annotation=[] 	#注解信息，格式“annotation=value”
--rule=[] 			#代理规则，格式“host/path=service:port[,tls=secretname]”,注意:rule中外部域名要在所有的名称空间唯一
--class=‘’ 			#此Ingress适配的Ingress Class Controller
-n 名称空间 			#ingress的名称空间，需要和service在同一个名称空间
#基于URI方式代理不同应用的请求时，后端应用的URI若与代理时使用的URI不同，则需要启用URL Rewrite完成URI的重写
#Ingress-Nginx支持使用“annotation nginx.ingress.kubernetes.io/rewrite-target”注解进行
```

**注意：k8s中域名只能解析到IP，不能解析端口**

```
https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
```

### 声明式实现

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress       # Ingress名称
  namespace: default          # 所属命名空间
  annotations:                # 注解
    kubernetes.io/ingress.class: "nginx"   # 指定Ingress Controller类型（常用nginx）

spec:
  ingressClassName: nginx     # 推荐用ingressClassName指定controller
  tls:                        # 配置HTTPS
  - hosts:
    - www.example.com
    secretName: example-tls   # 证书Secret的名称

  rules:                      # Ingress转发规则
  - host: www.example.com     # 访问域名
    http:
      paths:
      - path: /               # 访问路径
        pathType: Prefix      # 匹配模式（Prefix / Exact / ImplementationSpecific）
        backend:              # 后端服务
          service:
            name: example-service
            port:
              number: 80
```

| 字段                        | 说明                                            |
| --------------------------- | ----------------------------------------------- |
| apiVersion                  | 资源版本，一般是`networking.k8s.io/v1`          |
| kind                        | 资源类型，Ingress                               |
| metadata.name               | Ingress名称                                     |
| metadata.namespace          | 命名空间                                        |
| metadata.annotations        | 注解，常用于指定ingress class                   |
| spec.ingressClassName       | 推荐使用，指定Controller                        |
| spec.tls                    | https配置                                       |
| spec.rules                  | 域名和路径的转发规则                            |
| host                        | 需要匹配的域名                                  |
| paths.path                  | 匹配路径，必须以`/`开头                         |
| pathType                    | 路径类型：Exact、Prefix、ImplementationSpecific |
| backend.service.name        | 目标Service名称                                 |
| backend.service.port.number | Service端口号                                   |

```
https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#%E7%A4%BA%E4%BE%8B
```

| 类型   | 路径                            | 请求路径        | 匹配与否？               |
| ------ | ------------------------------- | --------------- | ------------------------ |
| Prefix | `/`                             | （所有路径）    | 是                       |
| Exact  | `/foo`                          | `/foo`          | 是                       |
| Exact  | `/foo`                          | `/bar`          | 否                       |
| Exact  | `/foo`                          | `/foo/`         | 否                       |
| Exact  | `/foo/`                         | `/foo`          | 否                       |
| Prefix | `/foo`                          | `/foo`, `/foo/` | 是                       |
| Prefix | `/foo/`                         | `/foo`, `/foo/` | 是                       |
| Prefix | `/aaa/bb`                       | `/aaa/bbb`      | 否                       |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb`      | 是                       |
| Prefix | `/aaa/bbb/`                     | `/aaa/bbb`      | 是，忽略尾部斜线         |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb/`     | 是，匹配尾部斜线         |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb/ccc`  | 是，匹配子路径           |
| Prefix | `/aaa/bbb`                      | `/aaa/bbbxyz`   | 否，字符串前缀不匹配     |
| Prefix | `/`, `/aaa`                     | `/aaa/ccc`      | 是，匹配 `/aaa` 前缀     |
| Prefix | `/`, `/aaa`, `/aaa/bbb`         | `/aaa/bbb`      | 是，匹配 `/aaa/bbb` 前缀 |
| Prefix | `/`, `/aaa`, `/aaa/bbb`         | `/ccc`          | 是，匹配 `/` 前缀        |
| Prefix | `/aaa`                          | `/ccc`          | 否，使用默认后端         |
| 混合   | `/foo` (Prefix), `/foo` (Exact) | `/foo`          | 是，优选 Exact 类型      |

主机名通配符

主机名可以是精确匹配（例如 “`foo.bar.com`”）或者使用通配符来匹配 （例如 “`*.foo.com`”）。 精确匹配要求 HTTP `host` 头部字段与 `host` 字段值完全匹配。 通配符匹配则要求 HTTP `host` 头部字段与通配符规则中的后缀部分相同。

| 主机        | host 头部         | 匹配与否？                          |
| ----------- | ----------------- | ----------------------------------- |
| `*.foo.com` | `bar.foo.com`     | 基于相同的后缀匹配                  |
| `*.foo.com` | `baz.bar.foo.com` | 不匹配，通配符仅覆盖了一个 DNS 标签 |
| `*.foo.com` | `foo.com`         | 不匹配，通配符仅覆盖了一个 DNS 标签 |





```bash
#准备环境实现两个service应用 pod-test1和pod-test2
#准备后端的应用pod-test v0.1和相应的service
[root@master1 ~]#kubectl create deployment pod-test1 --image=registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1 --replicas=3
[root@master1 ~]#kubectl create service clusterip pod-test1 --tcp=80:80

#准备后端的应用pod-test v0.2和相应的service
[root@master1 ~]#kubectl create deployment pod-test2 --image=registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.2 --replicas=3
[root@master1 ~]#kubectl create service clusterip pod-test2 --tcp=80:80
```

```bash
[root@master1 ~]#kubectl get pod
NAME                         READY   STATUS    RESTARTS   AGE
pod-test1-75894cb755-4f2ww   1/1     Running   0          31s
pod-test1-75894cb755-t4ptw   1/1     Running   0          31s
pod-test1-75894cb755-xbgbt   1/1     Running   0          31s
pod-test2-8cfbcc9c-6tlsf     1/1     Running   0          13s
pod-test2-8cfbcc9c-dpcz5     1/1     Running   0          13s
pod-test2-8cfbcc9c-qkdbp     1/1     Running   0          13s
[root@master1 ~]#kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   14d
pod-test1    ClusterIP   10.111.163.253   <none>        80/TCP    68s
pod-test2    ClusterIP   10.104.71.137    <none>        80/TCP    57s
```

### 单域名

#### 范例: 命令式实现单域名不支持子URL,子URL无法访问返回404

```yaml
#路径精确匹配,对于发往www.wang.org的请求，代理至service/pod-test1，其它的URL则无法代理
[root@master1 ~]#kubectl create ingress demo-ingress --rule="www.kang.org/=pod-test1:80" --class=nginx --dry-run=client -o yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: demo-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test1
            port:
              number: 80
        path: /
        pathType: Exact
status:
  loadBalancer: {}
```

```bash
[root@master1 ~]#kubectl create ingress demo-ingress --rule="www.kang.org/=pod-test1:80" --class=nginx --dry-run=client -o yaml > yaml/ing/demo-ingress.yaml
[root@master1 ~]#kubectl apply -f yaml/ing/demo-ingress.yaml
ingress.networking.k8s.io/demo-ingress created

[root@master1 ~]#kubectl get ingress 
NAME           CLASS   HOSTS          ADDRESS     PORTS   AGE
demo-ingress   nginx   www.kang.org   10.0.0.10   80      47s

```

```bash
#查看ingress-nginx-controller对应的Pod中Nginx配置文件的变化
[root@master1 ~]#kubectl exec -n ingress-nginx ingress-nginx-controller-5db8d9c658-fdpzh -- nginx -T | grep www.kang.org
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
	## start server www.kang.org
		server_name www.kang.org ;
	## end server www.kang.org

```

```bash
#集群外访问
C:\Users\zhaok>curl -H"host: www.kang.org" http://10.0.0.10
kubernetes pod-test v0.1!! ClientIP: 10.244.3.218, ServerName: pod-test1-75894cb755-xbgbt, ServerIP: 10.244.1.220!

#访问子URL失败，原因是只发布了www.wang.org的根目录，其它URL没有发布
C:\Users\zhaok>curl -H"host: www.kang.org" http://10.0.0.10/hostname
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```






#### 实现单域名支持子URL

```bash
[root@master1 ~]#kubectl create ingress demo-ingress --rule="www.kang.org/*=pod-test1:80" --class=nginx --dry-run=client -o yaml > yaml/ing/demo-ingress.yaml
[root@master1 ~]#cat yaml/ing/demo-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: demo-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test1
            port:
              number: 80
        path: /
        pathType: Prefix
status:
  loadBalancer: {}
[root@master1 ~]#kubectl apply -f yaml/ing/demo-ingress.yaml
ingress.networking.k8s.io/demo-ingress created

[root@master1 ~]#kubectl get ingress
NAME           CLASS   HOSTS          ADDRESS     PORTS   AGE
demo-ingress   nginx   www.kang.org   10.0.0.10   80      59s

#访问
C:\Users\zhaok>curl -H"host: www.kang.org" http://10.0.0.10/hostname
ServerName: pod-test1-75894cb755-4f2ww

C:\Users\zhaok>curl -H"host: www.kang.org" http://10.0.0.10
kubernetes pod-test v0.1!! ClientIP: 10.244.3.218, ServerName: pod-test1-75894cb755-t4ptw, ServerIP: 10.244.4.208!
```

### 单域名多URL

```mermaid
graph LR;
  client([客户端])-. Ingress-管理的 <br> 负载均衡器 .->ingress[Ingress, 178.91.123.132];
  ingress-->|/foo|service1[Service service1:4200];
  ingress-->|/bar|service2[Service service2:8080];
  subgraph 集群
  ingress;
  service1-->pod1[Pod];
  service1-->pod2[Pod];
  service2-->pod3[Pod];
  service2-->pod4[Pod];
  end
  classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
  classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
  classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
  class ingress,service1,service2,pod1,pod2,pod3,pod4 k8s;
  class client plain;
  class cluster cluster;
```



#### 单域名多URL不支持子URL

范例: 命令式实现单域名多URL，不支持子URL，如果子URL访问，也全部转发至后端Pod的根路径 /

| 类型        | 作用                       | 常见用途                             |
| ----------- | -------------------------- | ------------------------------------ |
| labels      | 用来查询、选择、分组资源   | Pod选择、Service绑定、Deployment筛选 |
| annotations | 存储额外信息（不用于查询） | 说明文字、ingress配置、监控配置      |



```bash
#路径精确匹配,对于发往www.wang.org/v1和/v2的请求，分别代理至service/pod-test1和service/pod-test2的根

[root@master1 ing]#kubectl create ingress demo-ingress1 --rule='www.kang.org/v1=pod-test1:80' --rule='www.kang.org/v2=pod-test2:80' --class=nginx --annotation nginx.ingress.kubernetes.io/rewrite-target="/" --dry-run=client -o yaml > demo-ingress1.yaml
[root@master1 ing]#cat demo-ingress1.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  creationTimestamp: null
  name: demo-ingress1
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test1
            port:
              number: 80
        path: /v1
        pathType: Exact
      - backend:
          service:
            name: pod-test2
            port:
              number: 80
        path: /v2
        pathType: Exact
status:
  loadBalancer: {}
[root@master1 ing]#kubectl apply -f demo-ingress1.yaml
ingress.networking.k8s.io/demo-ingress1 created
[root@master1 ing]#kubectl get ingress
NAME            CLASS   HOSTS          ADDRESS     PORTS   AGE
demo-ingress1   nginx   www.kang.org   10.0.0.10   80      42s


C:\Users\zhaok>curl www.kang.org/v1
kubernetes pod-test v0.1!! ClientIP: 10.244.3.218, ServerName: pod-test1-75894cb755-t4ptw, ServerIP: 10.244.4.208!

C:\Users\zhaok>curl www.kang.org/v2
kubernetes pod-test v0.2!! ClientIP: 10.244.3.218, ServerName: pod-test2-8cfbcc9c-qkdbp, ServerIP: 10.244.1.221!
```



#### 范例

![image-20250406160357299](kubernetes/image-20250406160357299.png)

Simple fanout: 在同一个FQDN下通过不同的URI完成不同应用间的流量分发

- 基于单个虚拟主机接收多个应用的流量
- 常用于将流量分发至同一个应用下的多个不同子应用，同一个应用内的流量由调度算法分发至该应用的各后端端点
- 不需要为每个应用配置专用的域名

范例：环境准备两个HTTP应用

```bash
#如果前面的资源已删除，重新应用上面小节的资源文件生成deployment和对应的SVC
#访问 www.wang.org/flask的时候，返回flask的结果
#访问 www.wang.org/nginx的时候，返回nginx的结果

#创建Department

# 部署 pod-test
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-test
  template:
    metadata:
      labels:
        app: pod-test
    spec:
      containers:
      - name: pod-test
        image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: deployment-service
spec:
  selector:
    app: pod-test
  ports:
  - name: http
    port: 80
    targetPort: 80

---

# 部署 nginx-test
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-test
  template:
    metadata:
      labels:
        app: nginx-test
    spec:
      containers:
      - name: nginx-test
        image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0
        imagePullPolicy: IfNotPresent
        ports:
        - name: nginx
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-test
  ports:
  - name: nginx
    port: 80
    targetPort: 80
```

```bash
[root@master1 ing]#vim ingress-deployment-svc.yaml
[root@master1 ing]#kubectl apply -f ingress-deployment-svc.yaml
deployment.apps/deployment-test created
service/deployment-service created
deployment.apps/deployment-nginx created
service/nginx-service created
```

##### 单域名多URL不支持子URL

范例: 单域名多URL，不支持子URL

```bash
#ingress清单文件
# Ingress 配置，统一入口
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-mul-url
  annotations:
    nginx.ingress.kubernetes.io/enable-real-ip: "true"
    nginx.ingress.kubernetes.io/rewrite-target: / #默认会转发给后端时会带有URL,添加此行，表示转发时删除后面的URL
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - path: /flask			#指定访问的URL
        pathType: Prefix		#表示以/nginx为开始即可
        backend:
          service:
            name: deployment-service
            port:
              name: http		#匹配对应servcie中的端口name: http
              #number: 80
      - path: /nginx			#指定访问的URL
        pathType: Prefix		#表示以/nginx为开始即可
        backend:
          service:
            name: nginx-service	#指定对应service的名称
            port:
              name: nginx		#匹配对应servcie中的端口name: nginx
              #number: 80
```

```bash
#注意事项：
#默认转给后端服务时会将url也同时转发，而后端服务有可能不存在此URL，所以需要在后端url转发的时候，取消转发关键字。
#方法就是，在annotation中添加一个重写的规则nginx.ingress.kubernetes.io/rewrite-target:/ 即所有的请求把ingress匹配到的url关键字清除掉

#应用资源文件
```

```bash
[root@master1 ing]#vim ingress-http-mul-url.yaml
[root@master1 ing]#kubectl apply -f ingress-http-mul-url.yaml
ingress.networking.k8s.io/ingress-mul-url configured
[root@master1 ing]#kubectl get ingress
NAME              CLASS   HOSTS          ADDRESS     PORTS   AGE
ingress-mul-url   nginx   www.kang.org   10.0.0.10   80      117s

[root@master1 ing]#kubectl describe ingress ingress-mul-url 
Name:             ingress-mul-url
Labels:           <none>
Namespace:        default
Address:          10.0.0.10
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host          Path  Backends
  ----          ----  --------
  www.kang.org  
                /flask   deployment-service:http (10.244.4.225:80,10.244.1.239:80,10.244.3.237:80)
                /nginx   nginx-service:nginx (10.244.3.238:80,10.244.1.240:80,10.244.4.226:80)
Annotations:    nginx.ingress.kubernetes.io/enable-real-ip: true
                nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    14s (x4 over 20m)  nginx-ingress-controller  Scheduled for sync
  Normal  Sync    14s (x4 over 20m)  nginx-ingress-controller  Scheduled for sync


C:\Users\zhaok>curl www.kang.org/nginx
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

C:\Users\zhaok>curl www.kang.org/flask
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: deployment-test-7dff5fbcf-fr277, ServerIP: 10.244.4.225!
```



#### 单域名多URL支持子URL

范例：命令式实现单域名多URL，支持子URL

```yaml
#使用URI的前缀匹配，而非精确匹配，且基于正则表达式模式进行url rewrite
#新版变化：kubernetes-1.32.0 使用指令式命令出错，无法实现

[root@master1 ~]#kubectl create ingress demo-ingress2 --rule='www.kang.org/v1(/|$)(.*)=pod-test1:80' --rule='www.kang.org/v2/(/|$)(.*)=pod-test2:80' --class=nginx --annotation nginx.ingress.kubernetes.io/rewrite-target='/$2' --dry-run=client -o yaml > yaml/ing/demo-ingress.yaml

#修改类型
[root@master1 ~]#cat yaml/ing/demo-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  creationTimestamp: null
  name: demo-ingress2
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test1
            port:
              number: 80
        path: /v1(/|$)(.*)
        #pathType: Exact
        pathType: ImplementationSpecific 		#修改为类型
      - backend:
          service:
            name: pod-test2
            port:
              number: 80
        path: /v2(/|$)(.*)
        #pathType: Exact
        pathType: ImplementationSpecific 		#修改为类型
status:
  loadBalancer: {}
```

```bash
[root@master1 ~]#kubectl apply -f yaml/ing/demo-ingress.yaml
ingress.networking.k8s.io/demo-ingress2 created
[root@master1 ing]#kubectl get ingress
NAME            CLASS   HOSTS          ADDRESS     PORTS   AGE
demo-ingress2   nginx   www.kang.org   10.0.0.10   80      8m14s

C:\Users\zhaok>curl www.kang.org/v1
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test1-75894cb755-xbgbt, ServerIP: 10.244.1.225!

C:\Users\zhaok>curl www.kang.org/v2
kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test2-8cfbcc9c-dpcz5, ServerIP: 10.244.3.223!

C:\Users\zhaok>curl www.kang.org/v1/hostname
ServerName: pod-test1-75894cb755-t4ptw

C:\Users\zhaok>curl www.kang.org/v2/hostname
ServerName: pod-test2-8cfbcc9c-qkdbp
```

 



### 多域名



```mermaid
graph LR;
  client([客户端])-. Ingress-管理的 <br> 负载均衡器 .->ingress[Ingress, 178.91.123.132];
  ingress-->|主机: foo.bar.com|service1[Service service1:80];
  ingress-->|主机: bar.foo.com|service2[Service service2:80];
  subgraph 集群
  ingress;
  service1-->pod1[Pod];
  service1-->pod2[Pod];
  service2-->pod3[Pod];
  service2-->pod4[Pod];
  end
  classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
  classDef k8s fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
  classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
  class ingress,service1,service2,pod1,pod2,pod3,pod4 k8s;
  class client plain;
  class cluster cluster;

```

![image-20250406165129631](kubernetes/image-20250406165129631.png)

范例：命令式实现基于主机头的多虚拟主机

```bash
#可添加的字段
  annotations:
#kubernetes.io/ingress.class: "nginx" #指定Ingress Controller的类型，k8s-v1.32.0不再支持
	nginx.ingress.kubernetes.io/use-regex: "true" #指定后面rules定义的path使用的正则表达式
	nginx.ingress.kubernetes.io/proxy-body-size: "100m" #客户端上传文件最大值，默认为1m
	nginx.ingress.kubernetes.io/proxy-connect-timeout: "60" #后端服务器的连接超时的时间,默认值为5s
	nginx.ingress.kubernetes.io/proxy-send-timeout: "120" #后端服务器数据回传超时时间,即规定时间之内后端服务器必须传完所有的数据,默认值为60s
	nginx.ingress.kubernetes.io/proxy-read-timeout: "120" #后端服务器响应的超时时间,默认值为60s
#nginx.ingress.kubernetes.io/app-root: /index.html #指定默认页面文件\
```

```yaml
[root@master1 ing]#kubectl create ingress demo-ingress3 --rule="test1.kang.org/*=pod-test1:80" --rule="test2.kang.org/*=pod-test2:80" --class=nginx --dry-run=client -o yaml > demo-ingress3.yaml
[root@master1 ing]#cat demo-ingress3.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: demo-ingress3
spec:
  ingressClassName: nginx
  rules:
  - host: test1.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test1
            port:
              number: 80
        path: /
        pathType: Prefix
  - host: test2.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test2
            port:
              number: 80
        path: /
        pathType: Prefix
status:
  loadBalancer: {}
[root@master1 ing]#kubectl apply -f demo-ingress3.yaml
ingress.networking.k8s.io/demo-ingress3 created
```

```bash
[root@master1 ing]#kubectl get ingress
NAME            CLASS   HOSTS                           ADDRESS     PORTS   AGE
demo-ingress3   nginx   test1.kang.org,test2.kang.org   10.0.0.10   80      113s

C:\Users\zhaok>curl test1.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test1-75894cb755-4f2ww, ServerIP: 10.244.3.225!

C:\Users\zhaok>curl test1.kang.org/hostname
ServerName: pod-test1-75894cb755-4f2ww

C:\Users\zhaok>curl test2.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test2-8cfbcc9c-6tlsf, ServerIP: 10.244.4.211!

C:\Users\zhaok>curl test2.kang.org/hostname
ServerName: pod-test2-8cfbcc9c-qkdbp
```

### HTTPS

```bash
#基于TLS的Ingress要求事先准备好专用的“kubernetes.io/tls”类型的Secret资源对象

[root@master1 ing]#(umask 077; openssl genrsa -out www.kang.org.key 2048)
[root@master1 ing]#openssl req -new -x509 -key www.kang.org.key -out www.kang.org.crt -subj /C=CN/ST=Beijing/L=Beijing/O=SRE/CN=www.kang.org -days 365
[root@master1 ing]#ls 
www.kang.org.crt  www.kang.org.key

#创建Secret
[root@master1 ing]#kubectl create secret tls tls-kang --cert=./www.kang.org.crt --key=./www.kang.org.key --dry-run=client -o yaml > tls-kang.yaml
[root@master1 ing]#cat tls-kang.yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURqVENDQW5XZ0F3SUJBZ0lVZUdaUWo5bmd4M1l5UWJvako4ZEZUaG00MXBzd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1ZqRUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdNQjBKbGFXcHBibWN4RURBT0JnTlZCQWNNQjBKbAphV3BwYm1jeEREQUtCZ05WQkFvTUExTlNSVEVWTUJNR0ExVUVBd3dNZDNkM0xtdGhibWN1YjNKbk1CNFhEVEkxCk1EUXdOakEyTkRreE9Wb1hEVEkyTURRd05qQTJORGt4T1Zvd1ZqRUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlYKQkFnTUIwSmxhV3BwYm1jeEVEQU9CZ05WQkFjTUIwSmxhV3BwYm1jeEREQUtCZ05WQkFvTUExTlNSVEVWTUJNRwpBMVVFQXd3TWQzZDNMbXRoYm1jdWIzSm5NSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDCkFRRUF0Z3l0aUZrL2t6RHdtWDVxU05lOFpHZExGZFpvU2hSMVZLa0NSd3EzQSt6Z2V3aks0VHBJL3Fmb2xuZWkKUS9ZSlpzSzFaQWtlaGtxMlVVRjVKS0lWZk9oQ3M0MlVoZWxUVXhtVTFnVUJQcFNURUs2cnpVN3RYTnE4L0ZvVApqTFhrMktIaXYxK2NFZ1ZZRlI4TGxPUXorU01VOGV1YWsxNE9oQVpHS241ZjlNbkpDdVpKNk1uemV1VlJ1TU5wClFTSElWd3hjMXFlemdEaVNoU1lYdVMvTFFjMzAyQ1dVTy92ckRTQmN6K3ZSY1ZsbTNGZDlLU2hsN0J1bHJVRXQKbXd6ajkydEw5M0E3VHMyMUZ2NzBjc0IvWjVnbnRndFZ5c0dSRFZ4bXZEUVJTZUg2bEZROStYemtRRHFKTjB2TQpFcGQ0N1YyMktSUEtBYy9Vai9pN0xUSllkUUlEQVFBQm8xTXdVVEFkQmdOVkhRNEVGZ1FVZ2FXcmZpOFpSQTBMCm1MTkJVNG42bmpmMHI2MHdId1lEVlIwakJCZ3dGb0FVZ2FXcmZpOFpSQTBMbUxOQlU0bjZuamYwcjYwd0R3WUQKVlIwVEFRSC9CQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFtemdoazRjT2laMmRRbVFDbnpmWgpJSVNNMnVvQTFxQ1JjVjJWbmFybjMzZVlmY25CeXk0YytFdWJnVXF6VHl4eTExSlZtWkw5bWllMGxrS0JYalp0Ci9TOHpscDRJaElrTHNYekdSdXhpSklXSnVlajJNQzJVNUpXd09MWFVLRUhoUStCaXZncHBMTEt2bXdzTVQ0OGEKTzdHTWdXZEFxeWNWU2hHY1lJMEFTRkpDc2FqRGh5SEIwazAyTkJWbzhXVXNURHRGcXowSElHQWNFL2FRaUk5MAoyM29UeWFJSkJyMzgxS01ZdzVRMUsrbW93VVhTYjBJUnlzSEpvUklhRFlIVmRIREJPYnQxMjNiUnpiM0xVVUZrCnNBb0F0NThZVU9VazR1cmh1L2JaeGlMZlhaMnh6ckNBaUUrN3pGSExPa0NBQzdXT1l6ZStIWm1mTndUM00yQTgKU3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2QUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktZd2dnU2lBZ0VBQW9JQkFRQzJESzJJV1QrVE1QQ1oKZm1wSTE3eGtaMHNWMW1oS0ZIVlVxUUpIQ3JjRDdPQjdDTXJoT2tqK3AraVdkNkpEOWdsbXdyVmtDUjZHU3JaUgpRWGtrb2hWODZFS3pqWlNGNlZOVEdaVFdCUUUrbEpNUXJxdk5UdTFjMnJ6OFdoT010ZVRZb2VLL1g1d1NCVmdWCkh3dVU1RFA1SXhUeDY1cVRYZzZFQmtZcWZsLzB5Y2tLNWtub3lmTjY1Vkc0dzJsQkljaFhERnpXcDdPQU9KS0YKSmhlNUw4dEJ6ZlRZSlpRNysrc05JRnpQNjlGeFdXYmNWMzBwS0dYc0c2V3RRUzJiRE9QM2EwdjNjRHRPemJVVwovdlJ5d0g5bm1DZTJDMVhLd1pFTlhHYThOQkZKNGZxVVZEMzVmT1JBT29rM1M4d1NsM2p0WGJZcEU4b0J6OVNQCitMc3RNbGgxQWdNQkFBRUNnZ0VBSkt6ckxPUG1CZTQvR24wNFJsS0lGcVlrbjFLaGtnTUxIRElzVGttZlUwQWUKZlVycExIVEVqZUVmMzJoY25WcHN6SEgvZnhiRE9vVEV0QnYvbnR5MGZZU0JJNUVyVWNzT3lWZ0cveExsREVsOApVMGtOUFlCZ0cydkoxUUVFblQxcnpDVFkvSHJ6ZzBIbVcrTDBQMXUyQ01RRGpUVk0zOUY4RlNiNmJjY2w1N2VECjlsREFGRjNpd3I2MGt2bWJIRTllTjJPQ0NVYmJpcjRCVlZSK1FxSGJLazJYUWdMZGZYRTNUbVFiNUlZRm1IdkIKaDYzVExpZXpuSWxxcHcvMTByUzJ1ZGhGYUpSVlpFTUU2d1NadEJyUmJvTjNoWGZ6QnR2NTVyNDdHODlpeGdqVgpFbDJHRndkWnRNTE5TdURZY0dSZ2RZbGtScThDMHVRY2dhYU9UZVIwaFFLQmdRREg0OXFpQUhtTjM0SkdGRm1PCmpwZE1BY21zTE9yY0tiWm9GTldvTGFDeGdkaXlDL096WnB6ZkV0OHkySHFpdGgxc0VTcm8vMzV1OGpZMFArY3cKZldqRHFRcFRRVklqcWZYUSsvd0dRNnpYWVMxTW5wd0N3OVhDRjRoa3V0clduWWhHdW1KWmo0QlFlVktQck91bwpnS1dXeW9GMkJBM3N6QjFqVjYxTzFOSDd2d0tCZ1FEcEpzeU9BRFBmVjNUUnlrSWtrcnVieVo5WUxHTHFnS2FUCjdrQ3BIRGpYS2NDSHFGZFpONVcvNFpvMjdRZ1d4b2dWVkZramZaV0lYNmJvZlM1T2JDMStId3lMYWw0ZC9mMEoKczNzTjBGNUJ3aHdzSkJ0QzF5b21nbXRKc0FUdWU4NkF4b1pBaXRmbHUvL1huWFBCUjZySTZYMVhxYVJKUUFodApXTHRyQko1SXl3S0JnRTVNSTB6SDBOVlEwbyt1MHZHelN2SmIxT3BiWWRmQmd4Qzc4SUVDOW1ueGliY2NWV1pWCmNHYzAxamJ0Y3dRZlArRWZ6R1llQmlGaFFQZ3dlOWVUbUNEa0t3WTNsUHdoVEFlRFhOSDdXazIzWSs0YVJ0UVMKYjcycVNDUGt5V205cFM3SFBWVUZqK1oxejNBMTlNMDVMeU9reVJjNE9WZ2pEL1BrMlI4QzRBbXpBb0dBV2t0WgpheUV1OWE4a0lkOW44RXdYZGtVR2ZocHFrSkVVY2pRU1BLd2VZMDNiTDI2RnJacTJWeUtqRytSK2ZudVMxbGZkClViNWdPL2wzWlpDZGIyZTcwcDFwdVZDUGRLUVo3RGpqVGoxRlU4ZytLdVlzM0owQnlQbzNYTWxiajVUSks5dzIKd0NaRmFNWi9WZHkzcU1IOXBXT3YxRlJ4dUtDZzR1QTdiVDNUb3pNQ2dZQWhrazdpU3BTQldLZERyTFMwTHNzeQoxeHphZkc2UUpHalJyYmtkWWcvSDAzMjFkd1hmUklsZU84ckFNMlhuYWZmcE9icmJ5K0Q0a05VMG1lS2tyZkRhCkVNaDZjNnAwdGRHWlJNZlFtcmk1QmEzdisydU8rZjdVTEpVbFVxZzM4QVZxb0xSQ2Zwa252OVdsc1JRWEU0aEYKK2dpQzMzMXJZdEp2czJRVk03QWdVZz09Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
kind: Secret
metadata:
  creationTimestamp: null
  name: tls-kang
type: kubernetes.io/tls
[root@master1 ing]#kubectl apply -f tls-kang.yaml
secret/tls-kang created
[root@master1 ing]#kubectl get secrets 
NAME               TYPE                DATA   AGE
tls-kang           kubernetes.io/tls   2      29s
```

```yaml
#ingress清单文件
[root@master1 ing]#kubectl create ingress tls-demo-ingress --rule="www.kang.org/*=pod-test1:80,tls=tls-kang" --class=nginx --dry-run=client -o yaml > tls-demo-ingress.yaml
[root@master1 ing]#cat tls-demo-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: tls-demo-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test1
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - www.kang.org
    secretName: tls-kang
status:
  loadBalancer: {}
```

```bash
[root@master1 ing]#kubectl apply -f tls-demo-ingress.yaml
ingress.networking.k8s.io/tls-demo-ingress configured
[root@master1 ing]#kubectl get ingress
NAME               CLASS   HOSTS          ADDRESS     PORTS     AGE
tls-demo-ingress   nginx   www.kang.org   10.0.0.10   80, 443   116s
```

```bash
#注意：启用tls后，该域名下的所有URI默认为强制将http请求利用308跳转至https，若不希望使用该跳转功能，可以使用如下注解选项
--annotation nginx.ingress.kubernetes.io/ssl-redirect=false
eg:
kubectl create ingress tls-demo-ingress2 --rule="www.kang.org/*=pod-test1:80,tls=tls-kang" --annotation nginx.ingress.kubernetes.io/ssl-redirect=false --class=nginx --dry-run=client -o yaml > tls-demo-ingress2.yaml

[root@master1 ing]#cat tls-demo-ingress2.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: null
  name: tls-demo-ingress2
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - backend:
          service:
            name: pod-test1
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - www.kang.org
    secretName: tls-kang
status:
  loadBalancer: {}

```

### 证书更新

HTTPS 的证书的有效期一般为1年,到期前需要提前更新证书

```bash
[root@master1 ~]#openssl s_client www.kang.org:443
...
Certificate chain
 0 s:C = CN, ST = Beijing, L = Beijing, O = SRE, CN = www.kang.org
   i:C = CN, ST = Beijing, L = Beijing, O = SRE, CN = www.kang.org
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Apr  6 06:49:19 2025 GMT; NotAfter: Apr  6 06:49:19 2026 GMT
...
```

范例: 证书更新

```bash
#重新颁发证书
[root@master1 ing]#(umask 077; openssl genrsa -out kang.key 2048)
[root@master1 ing]#openssl req -new -x509 -key kang.key -out kang.crt -subj /C=CN/ST=Beijing/L=Beijing/O=SRE/CN=www.kang.org -days 3650
[root@master1 ing]#ls
kang.key
kang.crt

#方法1:
#在线修改证书配置,需要提前先将新证书文件用base64编码并删除换行符
[root@master1 ing]#cat kang.crt | base64 | tr -d '\n'
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURqVENDQW5XZ0F3SUJBZ0lVSDJlanBTT2NHQWZHWGp1QzNxMU04QzJiNmNFd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1ZqRUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdNQjBKbGFXcHBibWN4RURBT0JnTlZCQWNNQjBKbAphV3BwYm1jeEREQUtCZ05WQkFvTUExTlNSVEVWTUJNR0ExVUVBd3dNZDNkM0xuZGhibWN1YjNKbk1CNFhEVEkxCk1EUXdOakEzTVRFMU1Wb1hEVE0xTURRd05EQTNNVEUxTVZvd1ZqRUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlYKQkFnTUIwSmxhV3BwYm1jeEVEQU9CZ05WQkFjTUIwSmxhV3BwYm1jeEREQUtCZ05WQkFvTUExTlNSVEVWTUJNRwpBMVVFQXd3TWQzZDNMbmRoYm1jdWIzSm5NSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDCkFRRUF6QlFZc2pWa2MxcXJkRGVzRlJXSWpUdUU3TE1CaGxEUlljdDFwNXo3cm92M21aMGFCcU53M0M4YkZieVcKV3orcHYvL2ZkeHhia0dyVk5hUU1DTFphWkhnNVdqMEZoWnExazM1S01VYklISFFJcmR4ZFp3SGw2QXdGOHFhOQpqeE9oWnUwMGVyWFc5NnpSaFA2OS96bEdHaEt1ODJ4a2hoTXRhR3dOU3kxaWpzNmY5dmpGZlZ0MzhySHhZWi9GCk9YVjNRMHMzcndpc1hHM1h5cHkxMlgrbFhuSHZrbWo0VloxenhlZm9CWUU5bDhJYWRiWDhENTJKZ1RmcUY5bEoKc0tORGJESHVEYmpjRURBK2kwK2ZUeVVoYThscjZuNlc1a281YXNEU0R5emNLcHVhTGM3OXlxWktZblVYSVRJSwpVU25rSlI5Z2xoT3dQUFovWkZScHB0YkZrd0lEQVFBQm8xTXdVVEFkQmdOVkhRNEVGZ1FVNkw4R0YzUE5wWkQvCm5jQzBleUN4K3Q4VDJLSXdId1lEVlIwakJCZ3dGb0FVNkw4R0YzUE5wWkQvbmNDMGV5Q3grdDhUMktJd0R3WUQKVlIwVEFRSC9CQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUF1Z2xLRWJjTUxhK0dwKzMwVXFIcQpPTnZOdHFoUWluNHRDWStpZHJFdGsreDk3emMvYVZpcGIxRTh1R3lydDdMZmt5aGlXdDRPVjlkRGJoNnpNMndmCmlxOU53QUh2V1krd2NKaDd3WHRiOFpFb3ZCd2tMOUwyWGVVOTJHMnlPNU85QnlpSXZ1ZmR0UGozTDR3cVhMWHYKdW0zcnRtdTJDZmFyVytFQ3lyV2NIRFYrRk13bWE4WVdFNG1qOUpEOCtuT01PbVVNZGV6cjhyS3dDb3lsbGVKSQpjU3h0RUFRYnp2bjJwVjUxdkcvR1EyNE0xbmVnRnNLcFZzM0Y5QnVjTHlpWHFFczV2elk3U2FzV0d5NElockQvClhGUFQ5cW5WaVZZSWZHbjVyTXdxME4yK2xNUW8zaDV3bis5QVFDZG1kWXlFZzhRUlZ4WG5Gb0ZzQ0lZWkpTdTIKWFE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
[root@master1 ing]#cat kang.key | base64 | tr -d '\n'
.LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRRE1GQml5TldSeldxdDAKTjZ3VkZZaU5PNFRzc3dHR1VORmh5M1dublB1dWkvZVpuUm9HbzNEY0x4c1Z2SlpiUDZtLy85OTNIRnVRYXRVMQpwQXdJdGxwa2VEbGFQUVdGbXJXVGZrb3hSc2djZEFpdDNGMW5BZVhvREFYeXByMlBFNkZtN1RSNnRkYjNyTkdFCi9yMy9PVVlhRXE3emJHU0dFeTFvYkExTExXS096cC8yK01WOVczZnlzZkZobjhVNWRYZERTemV2Q0t4Y2JkZksKbkxYWmY2VmVjZStTYVBoVm5YUEY1K2dGZ1QyWHdocDF0ZndQblltQk4rb1gyVW13bzBOc01lNE51TndRTUQ2TApUNTlQSlNGcnlXdnFmcGJtU2pscXdOSVBMTndxbTVvdHp2M0twa3BpZFJjaE1ncFJLZVFsSDJDV0U3QTg5bjlrClZHbW0xc1dUQWdNQkFBRUNnZ0VBRFhyeVdxc3VOSFJ6eU54NGtIZjRxaU9aVnkxSDE0QjFKRG1WVHZvT29ldjYKOGZtdm8ycWVaTnp6dEE3ZCs2OEwzblBmWmZYSWtwU0wrdHdZTTJjZ04yeTlsUmxjeVFOOEpQNzlsWjJ3VURqVgo3ZTRWZVYxbHUxczM5VnhRUHlnenpDRzFDcm9DTmlBRFF2NjB2Q3dqVmt1UHBPS3ZEVEF0R3Eyd2Z4dkRXOTFwCi9RSkdEMm03NGhBRFpwUFg0MXF6RnVuYXBTcjg4Y3VPUFAwcTB2cHFKNU1lVWlWTm93NXhHaFh3SU5EaVdDWDkKUC94bkR6M3o5RTJmcWxRZ0pMRGRYWnQwbWVRSmxWZnNGT2ZvbVhuczhHU1Y1L2EyZkRCTWdQOXBia1lsaFRNOApicGZPQ0pVcWlJSm9Ha2JadU1Kc1A4WmJoaGVlTmJkR0lWak1yZFhBOFFLQmdRRGZEVitnN0hxNkZPbU5kMG94CnJHUWw2bmtFRDFzMERsRkE5V2dBY0ZUYnh5enVxNGFJVTFCYmExQ3poK21mekY1UXB1anhwemxUcHdRc1dKdjcKTG5aNFR5NnNiME03Q252M0hiclF1MDN3NzAzRG9aWVFkMG04SEVxZDlLNGtMMGdOUkZUaUgrblpNWkFYSVpEbQpLOEw4cDJnVUltUnB0cW0xQ0lZVUdZb0d5d0tCZ1FEcU9UMHVvempsWVpwcUI5UW92VmpZblMrTFlWNnhyZzlYCk9yZnhUcG5XVGtlTmMrTVowNVE1K0ptT3prL3ZHc293TzZIQUdFTG50KzJia2JlU2QzaUxHYWNOQnN2L3JUVHYKRnNVbWZYVERMeG9mMDA4b3BrNlJOOElyQTk3eCtVVmhLVlIxNHJua2QyTmxqY3VaMlpTNzlJc3FYT2l0d2FUYQphQ3RwcmdrYldRS0JnRkc1U0t4aGVLWHRPNVZGZTcydmVCSWNVcktXSGNoNDVuUnlkZVNUUThReXVkZklxTVlTCklJTWVVcmMwdVBoL25KN1lTTWtoaC9PS1loRU5kamtWMFBwTHFNSXRFZWp6OG0rY1NQSnBiaWsweEwxRTJ3SzEKY2lLbUN3RE13WEVva2JHcXM1OFdLa3dWeU1GdzN2STJYbjdMcDJxWkgwbE1UK213Y2MremU4cjVBb0dCQU9Mdgp0cEhiWDZNUDRGYmZZdlplRHhsQm92RSsxRXIvUktuVm9LZFM2c2F6YmF4bUhpSEYyQlliQWwxVkZDbXRJVDErCmFZZi9lY09kZDhDRmZRR1NjOThDYWRLN1JTaGluNmtQdk5GQU9NTnNmYmxjT1pRL0hseWhVNlh0eUROV3U1d2EKT1ZWRzZhbXM2MGRZRFc3eFc2U0ltZFhmMHltWE42ek54VkpOWjFGeEFvR0FmaU1RWXBESEdZOUlJeUIvSVF2QQpqWFN6T1VKeUdWd3RQZ3ZpRFl4dTltZ0ZFWUVQcEZEVC8yM1hrZGQ4NGg3RytNcHFETEtpT3E3bHdtWUNMT2tRCjB6bnpIbzlLRjBKZUNMRGROVGNQZ3RhU1hvZGoxSG5KTjZyOTYyakdIZGRCQmhyRXc5TkkvL3c2ZW8rNnZkU1AKOVhqR0R4UUFaNjlMZEN1MG1qS29wVkU9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K

#上面生成的内容替换下面命令的内容,立即生效
[root@master1 ~]#kubectl edit secrets tls-kang


#方法2
#删除旧证书配置
[root@master1 ~]#kubectl delete secrets tls-kang
#创建新证书配置
[root@master1 ~]#kubectl create secret tls tls-kang --cert=./kang.crt --key=./kang.key

#合并成一条命令
[root@master1 ing]#kubectl delete secrets tls-kang;kubectl create secret tls tls-kang --cert=./kang.crt --key=./kang.key
secret "tls-kang" deleted
secret/tls-kang created

#集群外验证证书是否生效
[root@master1 ing]#openssl s_client www.kang.org:443
...
Certificate chain
 0 s:C = CN, ST = Beijing, L = Beijing, O = SRE, CN = www.kang.org
   i:C = CN, ST = Beijing, L = Beijing, O = SRE, CN = www.kang.org
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Apr  6 07:16:13 2025 GMT; NotAfter: Apr  4 07:16:13 2035 GMT
...
```



### 获取真实客户端IP

范例：获取真实客户端IP，需要通过loadBalancer的SVC实现才支持，ExternalIP不支持

```bash
[root@master1 ing]#kubectl create deployment myapp --image registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0
deployment.apps/myapp created

[root@master1 ing]#kubectl create svc clusterip myapp --tcp 80
service/myapp created
```



```yaml
[root@master1 ing]#vim ingress-http-real-ip.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/enable-real-ip: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: www.kang.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              number: 80
```

```bash
[root@master1 ing]#kubectl apply -f ingress-http-real-ip.yaml
ingress.networking.k8s.io/ingress-myapp created

[root@master1 ing]#kubectl get ingress
NAME            CLASS   HOSTS          ADDRESS     PORTS   AGE
ingress-myapp   nginx   www.kang.org   10.0.0.10   80      23s

C:\Users\zhaok>curl www.kang.org

[root@master1 ing]#kubectl exec -it myapp-5d88d79789-mt72z -- nginx -T | grep -A3 log_format
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

[root@master1 ing]#kubectl logs -f myapp-5d88d79789-mt72z 
10.244.3.224 - - [06/Apr/2025:07:59:47 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/8.10.1" "10.0.0.1"
10.244.3.224 - - [06/Apr/2025:07:59:47 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/8.10.1" "10.0.0.1"
```



### Kubernetes dashboard 实现 Https

```
https://github.com/kubernetes/dashboard
```

```
#环境准备，提前部署metrics-server
```

```bash
[root@master1 ~]#wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

#修改镜像路径
[root@master1 tools]#grep image recommended.yaml 
          #image: kubernetesui/dashboard:v2.0.0
          image: registry.cn-beijing.aliyuncs.com/wangxiaochun/dashboard:v2.0.0
          imagePullPolicy: Always
          #image: kubernetesui/metrics-scraper:v1.0.4
          image: registry.cn-beijing.aliyuncs.com/wangxiaochun/metrics-scraper:v1.0.4 

#修改service为LoadBalancer
---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: LoadBalancer		#增加此行
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
```

```bash
[root@master1 tools]#kubectl apply -f recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
Warning: spec.template.metadata.annotations[seccomp.security.alpha.kubernetes.io/pod]: non-functional in v1.27+; use the "seccompProfile" field instead
deployment.apps/dashboard-metrics-scraper created

[root@master1 tools]#kubectl get -n kubernetes-dashboard all
3NAME                                            READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-6b89d6c48-xtww7   1/1     Running   0          46s
pod/kubernetes-dashboard-584f694cb8-bn4lg       1/1     Running   0          46s

NAME                                TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
service/dashboard-metrics-scraper   ClusterIP      10.103.189.234   <none>        8000/TCP        46s
service/kubernetes-dashboard        LoadBalancer   10.111.43.83     10.0.0.11     443:31976/TCP   46s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           46s
deployment.apps/kubernetes-dashboard        1/1     1            1           46s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-6b89d6c48   1         1         1       46s
replicaset.apps/kubernetes-dashboard-584f694cb8       1         1         1       46s
```

```bash
[root@master1 tools]#kubectl get svc -n kubernetes-dashboard 
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP      10.103.189.234   <none>        8000/TCP        118s
kubernetes-dashboard        LoadBalancer   10.111.43.83     10.0.0.11     443:31976/TCP   118s
```



```bash
#生成登录需要的token
[root@master1 ~]#kubectl create serviceaccount dashboard-admin -n kube-system
[root@master1 ~]#kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

[root@master1 tools]#vim security-dashboard-admin-secret.yaml

apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: dashboard-admin-secret
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "dashboard-admin"

[root@master1 tools]#kubectl apply -f security-dashboard-admin-secret.yaml
secret/dashboard-admin-secret created

#查看token
[root@master1 tools]#kubectl describe secrets -n kube-system dashboard-admin-secret 
Name:         dashboard-admin-secret
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: 991896bf-788c-47a3-a15c-9124aebd9266

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1107 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InAyZlpRck9GTEljUmQza3k3RzFGZDNsdHAwenY4aGFZZ29MN2RpdzVwSzQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tc2VjcmV0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6Ijk5MTg5NmJmLTc4OGMtNDdhMy1hMTVjLTkxMjRhZWJkOTI2NiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpkYXNoYm9hcmQtYWRtaW4ifQ.JVfDK_-1i0rfWg7UXSzcvUg-qxwHXxRPqmgFumBj5tFQAFdqCcSNgZRQ0BwV3Y6xJEKtRg39H5sXtLpW4CB_-Z2S6plEOST0WGC9zj_fstm7g2-JF7HZRvytrwZ7Emc7eCae1539ltzKn2uFqjdxRBgyw0xAwXJ6XDVaE8B9-GDMJnNh4eI9nSh-kuMRdNIof4CAF1UVvHmDtrwePVlWAtpIOu1JnAxQU8gmCy8ZN5hktYDZ3xNyZ0tn6sWgUw15t0qcDePwafmXSKiKTIuv5ohWMRT_3v2g52j9spRsPgAwxl7N_PNUghseEQhto6dZp08Ye-HdEDq7eFpSHXS0kA

```



```bash
#第一种
client ——》 http://www.kang.org ——》 https:www.kang.org ——》 ingress ——》 http ——》 service ——》 pod

#第二种（相当于四层）
client ——》 https:www.kang.org ——》 ingress ——》 http ——》 service ——》 pod
```

```bash
#实现第二种
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-dashboard
  namespace: kubernetes-dashboard
  annotations:
    ingress.kubernetes.io/ssl-passthrough: "true"                     # 四层代理，TLS透传
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"             # 后端走 HTTPS
    nginx.ingress.kubernetes.io/rewrite-target: /$2                   # 保留 /dashboard 后的路径
spec:
  ingressClassName: nginx                                             # 指定 ingress class
  rules:
  - host: www.wang.org
    http:
      paths:
      - path: /dashboard(/|$)(.*)
        pathType: ImplementationSpecific                              # k8s v1.32 推荐写法
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
```

```bash
[root@master1 ing]#vim ingress-http-dashboard.yaml
[root@master1 ing]#kubectl apply -f ingress-http-dashboard.yaml
ingress.networking.k8s.io/ingress-dashboard created
[root@master1 ing]#kubectl get ingress -n kubernetes-dashboard 
NAME                CLASS   HOSTS          ADDRESS     PORTS   AGE
ingress-dashboard   nginx   www.wang.org   10.0.0.10   80      2m55s

[root@master1 ing]#kubectl get svc -n kubernetes-dashboard 
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP      10.103.189.234   <none>        8000/TCP        137m
kubernetes-dashboard        LoadBalancer   10.111.43.83     10.0.0.11     443:31976/TCP   137m
```



# Ingress Nginx 实现蓝绿 BlueGreen 和灰度Canary 发布

## Ingress Nginx的流量发布机制：

![image-20250406195037506](kubernetes/image-20250406195037506.png)

- 蓝绿：

  production: 100%, canary: 0%

  production: 0%, canary: 100% --> Canary变成后面的Production

- 金丝雀 Canary：

  - 流量比例化切分: 逐渐调整

  - 流量识别，将特定的流量分发给Canary：

    - By-Header：基于特定的标头识别

      Header 值默认：只有Always 或 Nerver 两种值

      Header 值自定义

      Header 值可以基于正则表达式Pattern进行匹配

    - By-Cookie: 基于Cookie识别

**基于Ingress Nginx的Canary规则**

Ingress Nginx 的 Annotations支持的Canary规则， Annotations 和 Label 相似也是保存资源对象上的元数据，但不能被标签选择器选择，且没有Label的名称最长63个字符的限制

- nginx.ingress.kubernetes.io/canary-weight：

  基于服务权重进行流量切分，适用于蓝绿或灰度发布，权重范围0 - 100按百分比将请求路由到Canary Ingress中指定的服务

  权重为 0 意味着该金丝雀规则不会向Canary入口的服务发送任何请求

  权重为100意味着所有请求都将被发送到 Canary 入口

- nginx.ingress.kubernetes.io/canary-by-cookie：

  基于 cookie 的流量切分，适用于灰度发布与 A/B 测试

  cookie 的值设置为 always 时，它将被路由到Canary入口

  cookie 的值设置为 never 时，请求不会被发送到Canary入口

  对于任何其他值，将忽略 cookie 并将请求与其他金丝雀规则进行优先级的比较，默认转发给旧版本

- nginx.ingress.kubernetes.io/canary-by-header：

  基于该Annotation中指定Request Header进行流量切分，适用于灰度发布以及A/B测试

  在请求报文中，若存在该Header且其值为always时，请求将会被发送到Canary版本，注意：always是大小敏感的

  若存在该Header且其值为never，请求将不会被发送至Canary版本

  若存在该Header且其值为任何其它值，将忽略该Annotation指定的Header，并按优先级将请求与其他金丝雀规则进行比较，默认转发给旧版本

  若不存在该Header时，请求将不会被发送至Canary版本

- nginx.ingress.kubernetes.io/canary-by-header-value：

  基于该Annotation中指定的Request Header的值进行流量切分，Header名称则由前一个Annotation（nginx.ingress.kubernetes.io/canary-by-header）进行指定

  请求报文中存在指定的Header，且其值与该Annotation的值匹配时，它将被路由到Canary版本

  对于任何其它值，将忽略该Annotation，默认转发给旧版本

- nginx.ingress.kubernetes.io/canary-by-header-pattern

  同canary-by-header-value的功能类似，但该Annotation基于正则表达式匹配Request Header的值

  若该Annotation与canary-by-header-value同时存在，则该Annotation会被忽略，默认转发给旧版本

**规则的应用次序**

- Canary规则会按特定的次序进行评估
- 优先级从低到高顺序： canary-weight-> canary-by-cookie -> canary-by-header

## 实战案例

### 范例：初始环境准备新旧两个版本应用和Ingress

```yaml
#准备新旧版本对应的各自独立的两套deployment和service

[root@master1 ing]#vim deploy-pod-test-v1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-test-v1
  labels:
    app: pod-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-test
      version: v0.1
  strategy: {}
  template:
    metadata:
      labels:
        app: pod-test
        version: v0.1
    spec:
      containers:
      - name: pod-test
        image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: pod-test-v1
  labels:
    app: pod-test
spec:
  selector:
    app: pod-test
    version: v0.1
  ports:
  - name: http-80
    protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
  

[root@master1 ing]#vim deploy-pod-test-v2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-test-v2
  labels:
    app: pod-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-test
      version: v0.2
  strategy: {}
  template:
    metadata:
      labels:
        app: pod-test
        version: v0.2
    spec:
      containers:
      - name: pod-test
        image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: pod-test-v2
  labels:
    app: pod-test
spec:
  selector:
    app: pod-test
    version: v0.2
  ports:
  - name: http-80
    protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

```bash
[root@master1 ing]#kubectl apply -f deploy-pod-test-v1.yaml -f deploy-pod-test-v2.yaml 
deployment.apps/pod-test-v1 unchanged
service/pod-test-v1 unchanged
deployment.apps/pod-test-v2 unchanged
service/pod-test-v2 unchanged

[root@master1 ing]#kubectl get pod
NAME                           READY   STATUS    RESTARTS   AGE
pod-test-v1-6448fc64d6-jgmfd   1/1     Running   0          107s
pod-test-v2-5d7f797cd8-97fdh   1/1     Running   0          3m40s
[root@master1 ing]#kubectl get deployments.apps 
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
pod-test-v1   1/1     1            1           112s
pod-test-v2   1/1     1            1           3m45s

[root@master1 ing]#kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP   3h58m
pod-test-v1   ClusterIP   10.104.86.183   <none>        80/TCP    2m40s
pod-test-v2   ClusterIP   10.108.28.83    <none>        80/TCP    3m37s
```

```yaml
#创建Ingress对应旧版本的应用

apiVersion: networking.k8s.io/v1  # 使用的API版本
kind: Ingress                      # 资源类型，Ingress资源
metadata:
  name: pod-test                   # Ingress的名称
  # annotations:                   # 注解（暂时没有配置，可以根据需要添加）
  #   kubernetes.io/ingress.class: nginx  # 老版本写法，新版推荐用spec.ingressClassName
spec:
  ingressClassName: nginx         # 指定Ingress Controller类型，nginx
  rules:                          # 规则配置
  - host: www.kang.org            # 指定访问的域名
    http:
      paths:                      # 路由规则
      - path: /                   # 匹配以 / 开头的路径（根路径）
        pathType: Prefix          # 匹配方式，Prefix 表示只要是以 / 开头就行
        backend:                  # 转发到的后端Service配置
          service:
            name: pod-test-v1     # Service的名字，对应后端Service
            port:
              number: 80          # Service暴露的端口
```

```bash
[root@master1 ing]#vim ingress-pod-test.yaml
[root@master1 ing]#kubectl apply -f ingress-pod-test.yaml
ingress.networking.k8s.io/pod-test created
[root@master1 ing]#kubectl get ingress
NAME       CLASS   HOSTS          ADDRESS     PORTS   AGE
pod-test   nginx   www.kang.org   10.0.0.10   80      100s
[root@master1 ing]#curl www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
```

```bash
#持续运行
[root@master1 ~]#while true;do curl www.kang.org;sleep 0.5;done
```



### 范例：蓝绿发布

```bash
#修改上前的Ingress清单文件,对应使用新版本的应用

apiVersion: networking.k8s.io/v1  # 使用的API版本
kind: Ingress                      # 资源类型，Ingress资源
metadata:
  name: pod-test                   # Ingress的名称
  # annotations:                   # 注解（暂时没有配置，可以根据需要添加）
  #   kubernetes.io/ingress.class: nginx  # 老版本写法，新版推荐用spec.ingressClassName
spec:
  ingressClassName: nginx         # 指定Ingress Controller类型，nginx
  rules:                          # 规则配置
  - host: www.kang.org            # 指定访问的域名
    http:
      paths:                      # 路由规则
      - path: /                   # 匹配以 / 开头的路径（根路径）
        pathType: Prefix          # 匹配方式，Prefix 表示只要是以 / 开头就行
        backend:                  # 转发到的后端Service配置
          service:
            #name: pod-test-v1     # Service的名字，对应后端Service
            name: pod-test-v2	   # 修改此行
            
            port:
              number: 80          # Service暴露的端口
```

```bash
[root@master1 ing]#vim ingress-pod-test.yaml
[root@master1 ing]#kubectl apply -f ingress-pod-test.yaml
ingress.networking.k8s.io/pod-test configured


#集群外客户端访问，观察全部都切换至新版本
kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!
kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244
```

```bash
#回滚旧版本
#修改上前的Ingress清单文件,对应使用旧版本的应用

apiVersion: networking.k8s.io/v1  # 使用的API版本
kind: Ingress                      # 资源类型，Ingress资源
metadata:
  name: pod-test                   # Ingress的名称
  # annotations:                   # 注解（暂时没有配置，可以根据需要添加）
  #   kubernetes.io/ingress.class: nginx  # 老版本写法，新版推荐用spec.ingressClassName
spec:
  ingressClassName: nginx         # 指定Ingress Controller类型，nginx
  rules:                          # 规则配置
  - host: www.kang.org            # 指定访问的域名
    http:
      paths:                      # 路由规则
      - path: /                   # 匹配以 / 开头的路径（根路径）
        pathType: Prefix          # 匹配方式，Prefix 表示只要是以 / 开头就行
        backend:                  # 转发到的后端Service配置
          service:
            #name: pod-test-v2     # Service的名字，对应后端Service
            name: pod-test-v1	   # 修改此行
            
            port:
              number: 80          # Service暴露的端口
```

```bash
[root@master1 ing]#vim ingress-pod-test.yaml
[root@master1 ing]#kubectl apply -f ingress-pod-test.yaml
ingress.networking.k8s.io/pod-test configured


kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
```

### 范例：基于权重的金丝雀发布

```bash
#旧版应用
[root@master1 ~]#kubectl apply -f ingress-pod-test.yaml

#新版的清单文件
apiVersion: networking.k8s.io/v1  # API版本
kind: Ingress                      # 类型：Ingress资源
metadata:
  name: pod-test-canary-by-weight  # Ingress的名字
  annotations:                     # 注解，用来控制金丝雀发布策略
    nginx.ingress.kubernetes.io/canary: "true"            # 开启canary（金丝雀发布）
    nginx.ingress.kubernetes.io/canary-weight: "10"       # 金丝雀版本权重，占总流量的10%
spec:
  ingressClassName: nginx  # 指定Ingress Controller类型（nginx）
  rules:
  - host: www.kang.org     # 访问的域名
    http:
      paths:
      - path: /            # 匹配 / 路径
        pathType: Prefix   # 前缀匹配
        backend:           # 转发后端
          service:
            name: pod-test-v2  # 金丝雀版本使用的Service（新版）
            port:
              number: 80
```

```bash
[root@master1 ing]#vim canary-by-wgight.yaml
[root@master1 ing]#kubectl apply -f canary-by-wgight.yaml
ingress.networking.k8s.io/pod-test-canary-by-weight created


kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
```

```yaml
#调整weight权重为90

apiVersion: networking.k8s.io/v1  # API版本
kind: Ingress                      # 类型：Ingress资源
metadata:
  name: pod-test-canary-by-weight  # Ingress的名字
  annotations:                     # 注解，用来控制金丝雀发布策略
    nginx.ingress.kubernetes.io/canary: "true"            # 开启canary（金丝雀发布）
    nginx.ingress.kubernetes.io/canary-weight: "90"       # 金丝雀版本权重，占总流量的10%
spec:
  ingressClassName: nginx  # 指定Ingress Controller类型（nginx）
  rules:
  - host: www.kang.org     # 访问的域名
    http:
      paths:
      - path: /            # 匹配 / 路径
        pathType: Prefix   # 前缀匹配
        backend:           # 转发后端
          service:
            name: pod-test-v2  # 金丝雀版本使用的Service（新版）
            port:
              number: 80
```

```bash
[root@master1 ing]#vim canary-by-wgight.yaml
[root@master1 ing]#kubectl apply -f canary-by-wgight.yaml
ingress.networking.k8s.io/pod-test-canary-by-weight unchanged
```

### 范例：基于 Cookie 实现金丝雀发布

```bash
#旧版应用
[root@master1 ~]#kubectl apply -f ingress-pod-test.yaml
```

```yaml
#清单文件
apiVersion: networking.k8s.io/v1  # API版本
kind: Ingress                      # 类型：Ingress资源
metadata:
  name: pod-test-canary-by-cookie  # Ingress的名字
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"           # 开启金丝雀发布
    nginx.ingress.kubernetes.io/canary-by-cookie: "vip_user" 
    # 说明：
    # 只有当客户端请求的cookie中存在 vip_user=always 时
    # 才会将流量引导到新版 pod-test-v2
spec:
  ingressClassName: nginx  # 指定Ingress Controller类型（nginx）
  rules:
  - host: www.kang.org     # 访问的域名
    http:
      paths:
      - path: /            # 匹配 / 路径
        pathType: Prefix   # 前缀匹配
        backend:           # 转发到 pod-test-v2 (新版)
          service:
            name: pod-test-v2
            port:
              number: 80
```

```bash
[root@master1 ing]#vim canary-by-cookie.yaml
[root@master1 ing]#kubectl apply -f canary-by-cookie.yaml
ingress.networking.k8s.io/pod-test-canary-by-cookie created

[root@master1 ~]#curl -b "vip_user=always" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.1.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!
[root@master1 ~]#curl www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!

```

### 范例：基于请求 Header 固定值的金丝雀发布

```bash
#旧版应用
[root@master1 ~]#kubectl apply -f ingress-pod-test.yaml
```

```yaml
#清单文件
apiVersion: networking.k8s.io/v1     # 版本
kind: Ingress                        # 类型：Ingress资源
metadata:
  name: pod-test-canary-by-header    # Ingress资源名称
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"     # 开启金丝雀发布
    nginx.ingress.kubernetes.io/canary-by-header: "X-Canary" 
    # 金丝雀条件：
    # 请求头中存在 X-Canary=always 的才会命中这个规则
spec:
  ingressClassName: nginx            # 指定Ingress Controller
  rules:
  - host: www.wang.org               # 绑定域名
    http:
      paths:
      - path: /                      # 匹配路径 /
        pathType: Prefix             # 前缀匹配
        backend:                     # 后端新版服务
          service:
            name: pod-test-v2        # 新版Service
            port:
              number: 80
```

```bash
[root@master1 ing]#vim canary-by-header.yaml
[root@master1 ing]#kubectl apply -f canary-by-header.yaml
ingress.networking.k8s.io/pod-test-canary-by-header created

[root@master1 ~]#curl -H"X-Canary: always" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!
[root@master1 ~]#curl -H"X-Canary: never" www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
```

### 范例：基于请求 Header 精确匹配指定值的金丝雀发布

```bash
#旧版应用
[root@master1 ~]#kubectl apply -f ingress-pod-test.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pod-test-canary-by-header-value  # Ingress资源名称
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"                # 开启金丝雀发布
    nginx.ingress.kubernetes.io/canary-by-header: "IsVIP"     # 判断Header字段 IsVIP（大小写不敏感）
    nginx.ingress.kubernetes.io/canary-by-header-value: "true" # 且 Header值= true（大小写敏感）才会走新版
spec:
  ingressClassName: nginx             # 指定Ingress控制器
  rules:
  - host: www.kang.org                # 绑定域名
    http:
      paths:
      - path: /                       # 匹配路径
        pathType: Prefix              # 前缀匹配
        backend:                      # 后端服务
          service:
            name: pod-test-v2         # 新版服务
            port:
              number: 80
```

```bash
[root@master1 ing]#vim canary-by-header-value.yaml 
[root@master1 ing]#kubectl apply -f canary-by-header-value.yaml
ingress.networking.k8s.io/pod-test-canary-by-header-value created

#集群外客户端访问，注意：IsVIP字段名大小写不敏感,true值大小写敏感
#当IsVIP: true，则访问v0.2新版
[root@master1 ~]#curl -H"IsVIP:true" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.1.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!
#当IsVIP: false，则访问v0.1旧版
[root@master1 ~]#curl -H"IsVIP:false" www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
#当IsVIP: 其它值，则访问v0.1旧版
[root@master1 ~]#curl -H"IsVIP:hzk" www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
#当没有首部字段IsVIP，则访问v0.1旧版
[root@master1 ~]#curl www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
```

### 范例：基于请求 Header 正则表达式模式匹配的指定值的金丝雀发布

```bash
#旧版应用
[root@master1 ~]#kubectl apply -f ingress-pod-test.yaml
```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pod-test-canary-by-header-pattern  # Ingress资源名称
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"  # 开启金丝雀发布
    nginx.ingress.kubernetes.io/canary-by-header: "Username"  # 请求头必须包含 Username 字段（大小写不敏感）
    nginx.ingress.kubernetes.io/canary-by-header-pattern: "^(vip|VIP)_.*" 
    # Username 字段的值必须匹配正则：以vip_或VIP_开头，后面任意字符
spec:
  ingressClassName: nginx  # 指定Ingress控制器
  rules:
  - host: www.wang.org  # 绑定域名
    http:
      paths:
      - path: /         # 匹配路径
        pathType: Prefix  # 前缀匹配
        backend:
          service:
            name: pod-test-v2  # 新版本服务
            port:
              number: 80
```

```bash
[root@master1 ing]#vim canary-by-header-pattern.yaml
[root@master1 ing]#kubectl apply -f canary-by-header-pattern.yaml
ingress.networking.k8s.io/pod-test-canary-by-header-pattern created
```

```bash
#集群外客户端访问
#符合正则表达式访问，则访问v0.2新版,字段名Username大小
[root@master1 ing]#curl -H"Username: VIP_kang" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!
[root@master1 ing]#curl -H"Username: VIP_hzk" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.1.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!
[root@master1 ing]#curl -H"Username: VIP_test" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.1.224, ServerName: pod-test-v2-5d7f797cd8-97fdh, ServerIP: 10.244.4.230!

#不符合正则表达式访问，则访问v0.1旧版
[root@master1 ing]#curl -H"Username: kang" www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
[root@master1 ing]#curl -H"Username: test" www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
[root@master1 ing]#curl -H"Username: hzk" www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!

#当没有首部字段Username，则访问v0.1旧版
[root@master1 ing]#curl www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.3.224, ServerName: pod-test-v1-6448fc64d6-jgmfd, ServerIP: 10.244.1.245!
```

### 范例：基于cookie和Header 同时存在的金丝雀发布比较优先级

```bash
#旧版应用
[root@master1 ~]#kubectl apply -f ingress-pod-test.yaml
```

```bash
[root@master1 ing]#cat canary-by-cookie.yaml
apiVersion: networking.k8s.io/v1  # API版本
kind: Ingress                      # 类型：Ingress资源
metadata:
  name: pod-test-canary-by-cookie  # Ingress的名字
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"           # 开启金丝雀发布
    nginx.ingress.kubernetes.io/canary-by-cookie: "vip_user" 
    # 说明：
    # 只有当客户端请求的cookie中存在 vip_user=always 时
    # 才会将流量引导到新版 pod-test-v2
spec:
  ingressClassName: nginx  # 指定Ingress Controller类型（nginx）
  rules:
  - host: www.kang.org     # 访问的域名
    http:
      paths:
      - path: /            # 匹配 / 路径
        pathType: Prefix   # 前缀匹配
        backend:           # 转发到 pod-test-v2 (新版)
          service:
            name: pod-test-v2
            port:
              number: 80
[root@master1 ing]#cat canary-by-header-pattern.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pod-test-canary-by-header-pattern  # Ingress资源名称
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"  # 开启金丝雀发布
    nginx.ingress.kubernetes.io/canary-by-header: "Username"  # 请求头必须包含 Username 字段（大小写不敏感）
    nginx.ingress.kubernetes.io/canary-by-header-pattern: "^(vip|VIP)_.*" 
    # Username 字段的值必须匹配正则：以vip_或VIP_开头，后面任意字符
spec:
  ingressClassName: nginx  # 指定Ingress控制器
  rules:
  - host: www.kang.org  # 绑定域名
    http:
      paths:
      - path: /         # 匹配路径
        pathType: Prefix  # 前缀匹配
        backend:
          service:
            name: pod-test-v2  # 新版本服务
            port:
              number: 80

#同时应用两个ingress的清单文件
[root@master1 ing]#kubectl apply -f canary-by-header-pattern.yaml -f canary-by-cookie.yaml
ingress.networking.k8s.io/pod-test-canary-by-header-pattern unchanged
ingress.networking.k8s.io/pod-test-canary-by-cookie created
```

```bash
#基于cookie访问，优先级低于header，所以不生效，访问旧版本
[root@master1 ing]#curl -H"cookie: vip_user=always" www.kang.org
kubernetes pod-test v0.1!! ClientIP: 10.244.1.224, ServerName: pod-test-v1-6448fc64d6-6dhxh, ServerIP: 10.244.1.246!

#基于header访问，优先级高于cookie，所以优先生效
[root@master1 ing]#curl -H"username: VIP_" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.1.224, ServerName: pod-test-v2-5d7f797cd8-xtdnr, ServerIP: 10.244.3.245!

#删除header的ingress
[root@master1 ing]#kubectl delete ingress pod-test-canary-by-header-pattern
ingress.networking.k8s.io "pod-test-canary-by-header-pattern" deleted

#再基于cookie访问，可以访问新版本
[root@master1 ing]#curl -H"cookie: vip_user=always" www.kang.org
kubernetes pod-test v0.2!! ClientIP: 10.244.3.224, ServerName: pod-test-v2-5d7f797cd8-xtdnr, ServerIP: 10.244.3.245!
```

