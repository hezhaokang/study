# 42、Kuberbetes配置管理

![image-20250405142807289](kubernetes/image-20250405142807289.png)

# 配置管理

```powershell
k8s 实现配置中心		针对系统级服务，mysql、Redis、nacos（部署到k8s）
nacos 实现配置中心	针对java业务服务 myapp 

configmaps	存放配置信息
secrets		存放安全要求高的信息（eg：密码）
```

```
[root@master1 ~]#kubectl api-resources 
NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
configmaps                          cm           v1                                true         ConfigMap
secrets                                          v1                                true         Secret
```

在容器化大行其道的当前，在为容器化应用提供配置信息方法一般有下面方法

- 制作镜像时，提前将定义好的配置文件拷贝进镜像之中
- 制作镜像时，内置环境变量，在启动容器时通过环境变量向容器传递配置数据，容器启动后将无法传递新的变量
- 启动容器时，基于存储卷向容器传递配置文件,但需要提前准备外部的存储
- 启动容器时，传递自定义的启动的命令，选项，参数

kubernetes作为分布式容器调度平台，同样会遇到配置变更的问题，如果将资源删除，重新修改配置再重新创建，这种方法太繁琐。

kubernetes提供了对 Pod 容器应用可以实现集中式的配置管理功能的相关资源：

- ConfigMap
- Secret
- downwardAPI
- Projected

通过这些组件来实现向pod中的容器应用中注入配置信息的机制，从而避免了开发者参与

注意：对于运行中容器的配置改变，还需要通过应用程序重载相关配置才能生效

## 配置组件简介

### Configmap

Configmap是Kubernetes集群中非常重要的一种配置管理资源对象。

借助于ConfigMap API可以向pod中的容器中注入配置信息。

ConfigMap不仅可以保存环境变量或命令行参数等属性，也可以用来保存整个配置文件或者JSON格式的文件。

各种配置属性和数据以 k/v或嵌套k/v 样式 存在到Configmap中

注意：所有的配置信息都是以明文的方式来进行保存，实现资源配置的快速获取或者更新。

### Secret

Kubernetes集群中，有一些配置属性信息是非常敏感的，所以这些信息在传递的过程中，是不希望其他人能够看到的

Kubernetes提供了一种加密场景中的配置管理资源对象Secret。

它在进行数据传输之前，会对数据进行编码，在数据获取的时候，会对数据进行解码。从而保证整个数据传输过程的安全。

注意：这些数据通常采用Base64机制保存，所以安全性一般

### DownwardAPI

downwardAPI 为运行在pod中的应用容器提供了一种反向引用。让容器中的应用程序了解所处pod或Node的一些基础外部属性信息。

从严格意义上来说，downwardAPI不是存储卷，它自身就存在。

相较于configmap、secret等资源对象需要创建后才能使用，而downwardAPI引用的是Pod自身的运行环境信息，这些信息在Pod启动的时候就存在。

### Projected

一个 `projected Volumes` 投射卷可以将若干现有的卷源映射到同一个目录之上。



## ConfigMap(cm)

```
https://kubernetes.io/zh-cn/docs/concepts/configuration/configmap/
https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/
```

ConfigMap 是一种 API 对象，用来将非机密性的数据保存到键值对中。使用时， [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 可以将其用作环境变量、命令行参数或者存储卷中的配置文件。

ConfigMap 将你的环境配置信息和[容器镜像](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-image)解耦，便于应用配置的修改。

**注意: Configmap 属于名称空间级别,只能被同一个名称空间的Pod引用**

![image-20250402191859857](kubernetes/image-20250402191859857.png)

**基本属性**

```bash
#kubectl explain cm
	binaryData 				#二进制数据
	data 					#文本数据，支持变量和文件
	immutable <boolean> 	#设为true，不能被修改只能删除，默认为nil可以随时被修改
	
#注意：基于data的方式传递信息的话，会在pod的容器内部生成一个单独的数据文件
```

**数据配置的格式**

```bash
#单行配置数据格式
属性名称key: 属性值value 	#单行配置内容，一般保存变量，参数等
文件名：单行内容 			#配置文件如果只有一行，也使用此方式，key为文件名，value为文件内容

#多行文件数据格式
文件名称1: | 	#注意：| 是"多行键值"的标识符
  文件内容行1 	#内容大小不能超过1M
  文件内容行2
  ......
文件名称2: | 	#注意：| 是"多行键值"的标识符
  文件内容行1 	#内容大小不能超过1M
  文件内容行2
  ......
```

**查看系统中的configmap格式**

```yaml
[root@master1 ~]#kubectl get cm kube-root-ca.crt -o yaml
apiVersion: v1
data:
  ca.crt: |
    -----BEGIN CERTIFICATE-----
    MIIDBTCCAe2gAwIBAgIIXE31jhiif1owDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
    AxMKa3ViZXJuZXRlczAeFw0yNTAzMjIwNzU3MzRaFw0zNTAzMjAwODAyMzRaMBUx
    EzARBgNVBAMTCmt1YmVybmV0ZXMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
    AoIBAQDUp8jisA5NJd6NrbJf0gTE7RcUrI4Lh54dapQ3rExIMps6Q9HzaRrvj+40
    cBvZlOtrPQCKd8UsErh7VWTFclTrTUCQo+Yiub5QnZL/prp4NWgbP8qcMBPUYL2s
    hEvAAFqkiKIFhxBSkGwuoCHV/FvxshZYrcQXP7ERh+GYW+Dtki/6drRlgTvdnj7s
    QwNl9dE2sHvFhg4Y2l93Qg3enZvHsXK0lasma9sGmgJznjo+7vfY3IqGfPX86M1c
    3C6BQoqUTWQ6vFN5xMd3hCMO1/5WQHnvM1wk8/rgZ7LP3WJJVZk/o2nT+3C7so2F
    oCqEHXHpZnN9WBqtzoHA5+cl6eoVAgMBAAGjWTBXMA4GA1UdDwEB/wQEAwICpDAP
    BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBRFLqKrhIz1ilFgL8VmGRowYJd0/DAV
    BgNVHREEDjAMggprdWJlcm5ldGVzMA0GCSqGSIb3DQEBCwUAA4IBAQBXz4SKNBcb
    57e/MoCIP6m3Oi9KIFItL4nZMWW/nNeQ7YJgk+W5dZJ/au0AXWdnyJ4zDybKUivv
    n671hT/mE4qazh72Bt/BGwrfQKGBflDb7igwA+smSE6SRSpK7PR7efJ9jtzggKUW
    E2P/GRMtvw0aJZD2iPEGK9ks1TDWUIjDCSOFsBAbd0RbQWm0YBMJJWUYgOpy/hGS
    aqF8ua9I0M+M+3SVpu78wlmNABqFZhGQ2nf1NxyZSSxNqV5cprWFe9rPNGjnYnlA
    HHmuWnewop72T3gc3ui1BL1VwDD/sAwtgovcqGMxPnB2xXY00z+1WCsC6VDPwgIv
    IHAM4NNAnuK9
    -----END CERTIFICATE-----
kind: ConfigMap
metadata:
  annotations:
    kubernetes.io/description: Contains a CA bundle that can be used to verify the
      kube-apiserver when using internal endpoints such as the internal service IP
      or kubernetes.default.svc. No other usage is guaranteed across distributions
      of Kubernetes clusters.
  creationTimestamp: "2025-03-22T08:03:33Z"
  name: kube-root-ca.crt
  namespace: default
  resourceVersion: "352"
  uid: c2051fc7-6619-499c-be2a-c716fabf62bd.

[root@master1 ~]#cat /etc/kubernetes/pki/ca.crt 
-----BEGIN CERTIFICATE-----
MIIDBTCCAe2gAwIBAgIIXE31jhiif1owDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yNTAzMjIwNzU3MzRaFw0zNTAzMjAwODAyMzRaMBUx
EzARBgNVBAMTCmt1YmVybmV0ZXMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQDUp8jisA5NJd6NrbJf0gTE7RcUrI4Lh54dapQ3rExIMps6Q9HzaRrvj+40
cBvZlOtrPQCKd8UsErh7VWTFclTrTUCQo+Yiub5QnZL/prp4NWgbP8qcMBPUYL2s
hEvAAFqkiKIFhxBSkGwuoCHV/FvxshZYrcQXP7ERh+GYW+Dtki/6drRlgTvdnj7s
QwNl9dE2sHvFhg4Y2l93Qg3enZvHsXK0lasma9sGmgJznjo+7vfY3IqGfPX86M1c
3C6BQoqUTWQ6vFN5xMd3hCMO1/5WQHnvM1wk8/rgZ7LP3WJJVZk/o2nT+3C7so2F
oCqEHXHpZnN9WBqtzoHA5+cl6eoVAgMBAAGjWTBXMA4GA1UdDwEB/wQEAwICpDAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBRFLqKrhIz1ilFgL8VmGRowYJd0/DAV
BgNVHREEDjAMggprdWJlcm5ldGVzMA0GCSqGSIb3DQEBCwUAA4IBAQBXz4SKNBcb
57e/MoCIP6m3Oi9KIFItL4nZMWW/nNeQ7YJgk+W5dZJ/au0AXWdnyJ4zDybKUivv
n671hT/mE4qazh72Bt/BGwrfQKGBflDb7igwA+smSE6SRSpK7PR7efJ9jtzggKUW
E2P/GRMtvw0aJZD2iPEGK9ks1TDWUIjDCSOFsBAbd0RbQWm0YBMJJWUYgOpy/hGS
aqF8ua9I0M+M+3SVpu78wlmNABqFZhGQ2nf1NxyZSSxNqV5cprWFe9rPNGjnYnlA
HHmuWnewop72T3gc3ui1BL1VwDD/sAwtgovcqGMxPnB2xXY00z+1WCsC6VDPwgIv
IHAM4NNAnuK9
-----END CERTIFICATE-----
```

configmap资源类型的创建，与Kubernetes的其他很多资源对象的创建方式一样，主要涉及到两种方式：

- 命令行工具：配置中有大量简单的键值对时建议使用
- 资源定义文件：配置为大量文本内容时建议使用，此方式需要事先准备在资源清单元文件中加入配置文件内容

虽然 configMap 支持动态更新，但仍可能出现对应的Pod无法自动加载已更新的配置信息而导致无法生效的情况

注意:

- 如果Pod是以subPath 的形式挂载ConfigMap和Secret,那么Pod是不会自动感知到ConfigMap和Secret的更新
- 如果Pod 的变量来自于ConfigMap和Secret中定义的内容,那么ConfigMap和Secret更新后，也不会更新Pod中的变量。

通常为了避免配置更新后没有生效的问题，可以在更新 ConfigMap 之后，手动重创建相关的 Pod 或者Deployment

- 运维方式：可以通过 重启 或 重建 的方式
  - 使用 kubectl rollout restart 命令来重启 Deployment
  - 使用 kubectl delete pod 命令来删除 Pod，从而触发 Pod 的重启
- 应用程序方式：不断探测挂载后的 config 的内容变化， 程序自身支持进行热加载操作

### ConfigMap 创建和更新

#### 命令行创建方式

```bash
#创建命令格式
kubectl create configmap NAME [--from-file=[key=]source] [--from-
literal=key1=value1] [--dry-run=server|client|none] [-n <namespace>] [options]

#参数详解:
--from-literal=key1=value1 			#以设置键值对的方式实现变量配置数据
--from-env-file=/PATH/TO/FILE 		#以环境变量专用文件的方式实现配置数据
--from-file=[key=]/PATH/TO/FILE 	#以配置文件的方式创建配置文件数据，如不指定key，FILE名称为key名
--from-file=/PATH/TO/DIR 			#以配置文件所在目录的方式创建配置文件数据

--dry-run=client -o yaml 			#测试运行并显示cm内容

#查看configmap
kubectl create configmap <cm_name> -n <namespace> [-o yaml] --dry-run=client
kubectl get configmap <cm_name> -n <namespace>
kubectl describe configmap <cm_name> -n <namespace>

#删除configmap
kubectl delete configmap <cm_name> [-n <namespace>]
```

#### 命令行创建方式案例

范例：命令行创建基于key/value形式的变量配置

```bash
[root@master1 ~]#kubectl create configmap cm-test1 --from-literal=key1='value1' --from-literal=key2='value2'
configmap/cm-test1 created
[root@master1 ~]#kubectl get cm
NAME               DATA   AGE
cm-test1           2      17s
[root@master1 ~]#kubectl get cm cm-test1 -o yaml
apiVersion: v1
data:
  key1: value1
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: "2025-04-02T11:55:56Z"
  name: cm-test1
  namespace: default
  resourceVersion: "509883"
  uid: 966009b4-63c3-4acd-bf4f-cec9d809d88a
[root@master1 ~]#kubectl describe cm cm-test1 
Name:         cm-test1
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
key1:
----
value1

key2:
----
value2


BinaryData
====

Events:  <none>

#删除CM
[root@master1 ~]#kubectl delete cm cm-test1
```

范例: 命令行创建基于key/value形式的变量配置

```bash
[root@master1 ~]#kubectl create configmap pod-test-config --from-literal=host='127.0.0.1' --from-literal=port='8888'
configmap/pod-test-config created
[root@master1 ~]#kubectl get cm
NAME               DATA   AGE
cm-test1           2      3m38s
kube-root-ca.crt   1      11d
pod-test-config    2      7s
[root@master1 ~]#kubectl get cm pod-test-config -o yaml
apiVersion: v1
data:
  host: 127.0.0.1
  port: "8888"
kind: ConfigMap
metadata:
  creationTimestamp: "2025-04-02T11:59:27Z"
  name: pod-test-config
  namespace: default
  resourceVersion: "510326"
  uid: c2a941bd-3661-4e2b-9725-ee6c5c1e2aef
```

范例: 命令行创建基于环境变量文件的变量配置

```bash
#如果变量较多，使用上面方式一个个的设定环境变量太繁琐，可以全部添加到环境变量文件中然后基于它来创建CM。
#定制环境变量文件
[root@master1 ~]#cat conf.d/env
key1=value1
ket2=value2
#注意：env文件中所有的配置项以 "属性名=属性值" 格式定制
#将所有环境变量添加到到configmap中
[root@master1 ~]#kubectl create configmap cm-test2 --from-env-file=conf.d/env 
configmap/cm-test2 created
[root@master1 ~]#kubectl get cm cm-test2 -o yaml
apiVersion: v1
data:
  ket2: value2
  key1: value1
kind: ConfigMap
metadata:
  creationTimestamp: "2025-04-02T12:02:58Z"
  name: cm-test2
  namespace: default
  resourceVersion: "510779"
  uid: 9d7d167e-23d9-49e1-b976-e4ecfbee19a2
```

范例：命令行创建基于配置文件的文件形式CM

```bash
#直接将多个配置文件创建为一个ConfigMap
[root@master1 ~]#cat conf.d/app1.conf 
[app1]
config1
[root@master1 ~]#cat conf.d/app2.conf 
[app2]
config2

#文件名自动成为key名
[root@master1 ~]#kubectl create configmap cm-test3 --from-file=./conf.d/app1.conf --from-file=./conf.d/app2.conf 
configmap/cm-test3 created
[root@master1 ~]#kubectl get cm cm-test3 -o yaml
apiVersion: v1
data:
  app1.conf: |
    [app1]
    config1
  app2.conf: |
    [app2]
    config2
kind: ConfigMap
metadata:
  creationTimestamp: "2025-04-02T12:06:41Z"
  name: cm-test3
  namespace: default
  resourceVersion: "511251"
  uid: 37a50544-2b26-4c71-933a-55719c65d0fa
```

范例: 命令行创建基于目录的CM

```bash
[root@master1 ~]#cat conf.d/app1.conf 
[app1]
config1
[root@master1 ~]#cat conf.d/app2.conf 
[app2]
config2
[root@master1 ~]#cat conf.d/app3.conf 
[app3]
config3
[root@master1 ~]#cat conf.d/env 
key1=value1
ket2=value2
[root@master1 ~]#ls conf.d/
app1.conf  app2.conf  app3.conf  env

#直接将一个目录下的所有配置文件创建为一个ConfigMap
[root@master1 ~]#kubectl create configmap cm-test4 --from-file ./conf.d/
configmap/cm-test4 created

#结果显示：多个文件之间，属性名是文件名，属性值是文件内容
[root@master1 ~]#kubectl get cm cm-test4 -o yaml
apiVersion: v1
data:
  app1.conf: |
    [app1]
    config1
  app2.conf: |
    [app2]
    config2
  app3.conf: |
    [app3]
    config3
  env: |
    key1=value1
    ket2=value2
kind: ConfigMap
metadata:
  creationTimestamp: "2025-04-02T12:09:38Z"
  name: cm-test4
  namespace: default
  resourceVersion: "511622"
  uid: bf90d2bf-530f-4634-aaf5-b2e11b5b8ed6
```

#### 资源清单文件创建方式

```yaml
#清单文件格式
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm_name
  namespace: default
data:
  key: value #配置信息如果只有一行，也使用此方式，使用卷挂载时，key即为文件名，value为文件内容
  文件名: | #配置文件有多行使用此方式
  文件内容行1
  文件内容行2
  .....
#注意：CM的清单文件没有spec信息，而是data

#命令式:
kubectl create -f /path/file
#声明式
kubectl apply -f /path/file
```

注意：此方式需要事先将配置文件的内容全部写入清单文件，而且配置文件的格式需要调整才能匹配，所以很不方便，推荐如下方式解决

- 先在命令行执行时指定配置文件的方式创建 CM
- 通过 kubectl get cm <CM_NAME> -o yaml > cm.yaml 方式导出资源清单文件
- 或者 kubectl create configmap NAME --dry-run=client -o yaml > cm.yaml 方式导出资源清单文件
- 最后调整和修改上面生成的 yaml资源清单文件

```bash
[root@master1 conf.d]#cat file.conf 
[class]
go
liunx
java
shell

[root@master1 conf.d]#kubectl create configmap cm-name --from-literal=name='hezhaokang' --from-file=./file.conf --dry-run=client -o yaml > stroge-configmap-test.yaml
[root@master1 conf.d]#cat stroge-configmap-test.yaml
apiVersion: v1
data:
  file.conf: |
    [class]
    go
    liunx
    java
    shell
  name: hezhaokang
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm-name

[root@master1 conf.d]#kubectl apply -f stroge-configmap-test.yaml
configmap/cm-name created
[root@master1 conf.d]#kubectl get cm
NAME               DATA   AGE
cm-name            2      8s
```

范例：使用在线编辑文件

```bash
[root@master1 conf.d]#kubectl edit cm cm-name 
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  file.conf: |
    [class]
    go
    liunx
    java
    shell
    python		#添加此行
  name: hezhaokang
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"file.conf":"[class]\ngo\nliunx\njava\nshell\n","name":"hezhaokang"},"kind":"ConfigMap","metadata":{"annotations":{},"creationTimestamp":null,"name":"cm-name","namespace":"default"}}
  creationTimestamp: "2025-04-02T12:52:29Z"
  name: cm-name
  namespace: default
  resourceVersion: "517688"
  uid: e4c28a20-322a-48d3-a915-7dddfac123dd
[root@master1 conf.d]#kubectl get cm cm-name -o yaml 
apiVersion: v1
data:
  file.conf: |
    [class]
    go
    liunx
    java
    shell
    python
  name: hezhaokang
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"file.conf":"[class]\ngo\nliunx\njava\nshell\n","name":"hezhaokang"},"kind":"ConfigMap","metadata":{"annotations":{},"creationTimestamp":null,"name":"cm-name","namespace":"default"}}
  creationTimestamp: "2025-04-02T12:52:29Z"
  name: cm-name
  namespace: default
  resourceVersion: "517688"
  uid: e4c28a20-322a-48d3-a915-7dddfac123dd
```

范例: 只读的configmap

```bash
[root@master1 conf.d]#vim stroge-configmap-test.yaml 

apiVersion: v1
data:
  file.conf: |
    [class]
    go
    liunx
    java
    shell
  name: hezhaokang
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm-name
immutable: true #只读

[root@master1 conf.d]#kubectl apply -f stroge-configmap-test.yaml
configmap/cm-name configured

[root@master1 conf.d]#kubectl edit cm cm-name 
error: configmaps "cm-name" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-1956243534.yaml"
error: Edit cancelled, no valid changes were saved.
```

范例：生成redis配置文件对应的cm

```bash
[root@master1 ~]#kubectl create cm cm-redis --from-file /etc/redis/redis.conf --dry-run=client -o yaml > yaml/cm/cm-redis.yaml

#查看生成的文件
[root@master1 ~]#head yaml/cm/cm-redis.yaml 
apiVersion: v1
data:
  redis.conf: "# Redis configuration file example.\n#\n# Note that in order to read
    the configuration file, Redis must be\n# started with the file path as first argument:\n#\n#
    ./redis-server /path/to/redis.conf\n\n# Note on units: when memory size is needed,
    it is possible to specify\n# it in the usual form of 1k 5GB 4M and so forth:\n#\n#
    1k => 1000 bytes\n# 1kb => 1024 bytes\n# 1m => 1000000 bytes\n# 1mb => 1024*1024
    bytes\n# 1g => 1000000000 bytes\n# 1gb => 1024*1024*1024 bytes\n#\n# units are
    case insensitive so 1GB 1Gb 1gB are all the same.\n\n##################################
    INCLUDES ###################################\n\n# Include one or more other config

[root@master1 ~]#cat yaml/cm/cm-redis.yaml | wc -l
1070
[root@master1 ~]#cat /etc/redis/redis.conf | wc -l
1881
```

#### 在线更新 configmap

注意：configmap虽然支持在线更新，但是configmap更新后可能不会对已有的Pod的应用生效，可能还需要重建Pod才能生效

```bash
#创建 configmap
[root@master1 ~]#kubectl create cm cm-nginx-conf --from-file=nginx.conf

#修改configmap
#方法1,旧版中如果配置内容如果不大,多行内容可以显示在一行,但此方式不方便修改,但如果过大,此方式只能显示大小,而非内容,所以不能修改,新版无此问题
[root@master1 ~]#kubectl edit cm cm-nginx-conf

#方法2
[root@master1 ~]#kubectl get cm cm-nginx-conf -o yaml > cm-config-conf.yaml
[root@master1 ~]#cp cm-config-conf.yaml cm-config-conf.yaml.bak
[root@master1 ~]#vim cm-config-conf.yaml
[root@master1 ~]#kubectl apply -f cm-config-conf.yaml

#方法3
#修改配置
[root@master1 ~]#vim nginx.conf
[root@master1 ~]#kubectl create cm cm-nginx-conf --from-file=nginx.conf --dry-run=client -oyaml | kubectl apply -f -
```

### ConfigMap 使用

使用ConfigMap主要有两种方式：

- 通过环境变量的方式直接传递pod
- 使用volume的方式挂载入到pod内的文件中

注意：

- ConfigMap必须在Pod之前创建
- 与ConfigMap在同一个namespace内的pod才能使用ConfigMap，即ConfigMap不能跨命名空间调用。
- ConfigMap通常存放的数据不要超过1M
- CM 可以支持实时更新，在原来的pod里面直接看到效果

#### 通过环境变量的方式直接传递 Pod

引用ConfigMap对象上特定的key，以valueFrom赋值给Pod上指定的环境变量

也可以在Pod上使用envFrom一次性导入ConfigMap对象上的所有的key-value,key(可以统一附加特定前缀）即为环境变量,value自动成为相应的变量值

环境变量是容器启动时注入的，容器启动后变量的值不会随CM更改而发生变化,即一次性加载configmap,除非删除容器重建

通过环境变量脚本的方式，预处理这些配置信息

注意：环境变量的名称如果要用横线的话，最好使用下划线 _

环境变量格式

方式1：env 对指定的变量一个一个赋值

```bash
kubectl explain pod.spec.containers.env
	name 		#手工定制环境变量时，设置环境变量的名称，必选字段
	value 		#手工定制环境变量时，直接设置环境变量的属性值，不通过CM获取配置，可选字段
	valueFrom 	#手工定制环境变量时，设置环境变量的属性来源，可以支持从CM,Secret,downwordAPI获取
	
kubectl explain pod.spec.containers.env.valueFrom.configMapKeyRef
	name 		#引用指定的configmap
	key 		#引用指定的configmap中的具体哪个key
	optional 	#如果设置为false，标识该项是必选项，如果设置为true标识这个key是可选的。默认false
	
#此方式实现过程
1）容器中自定义环境变量名
2）根据CM的名称和Key名，找到对应的value
3) 再将value赋值给容器的环境变量
```

方式2：envFrom 使用CM的所有变量实现对变量的批量赋值，此方式生产更为推荐

```bash
kubectl explain pod.spec.containers.envFrom
	configMapRef 	#ConfigMap对象中的所有Key
	secretKeyRef 	#Secret对象中的所有Key
	prefix 			#为ConfigMap中的每个属性都添加前缀标识

#此方实现过程
1）容器中自定义环境变量名，并且和CM的key同名
2）找到指定的CM中所有Key名，将值批量直接赋值给容器中相同名称的变量
```



#### 使用volume的方式挂载入到pod内的文件中

在Pod上将 configMap对象引用为存储卷，而后整体由容器mount至某个目录下，key转为文件名，value即为相应的文件内容

在Pod上定义configMap卷时，仅引用其中的部分key，而后由容器mount至目录下

在容器上仅mount configMap卷上指定的key

容器中挂载的Volume数据可以根据时间戳机制和ConfigMap 同步更新，即configmap变更后会自动加载，但更新时间是不确定的，

而且旧式应用不支持热加载，因此可能不会自动加载更新的配置文件，从而导致配置无法生效

所以一般建议当更新configmap的配置后，可以通过重新Pod使之生效，符合不可变基础设施的理念

推荐滚动升级pod的方式来让ConfigMap内容变化生效

volume 格式

```bash
#指定卷信息，引用CM
kubectl explain pod.spec.volumes
  name: volume_name 	#指定volume名
  configMap：
    name: CM_name 		#指定CM的名称
    optional: true|false # 表示是否必须存在
    items <[]Object>
    
    
#挂载卷到容器内的目录
kubectl explain pod.spec.containers.volumeMounts
  volumeMounts:
  - name: CM_name 		#指定CM的名称
    mountPath; /path 	#容器内的挂载点
    readOnly: true 		#只读
    subPath <string>  	#应从中安装容器卷的卷内的路径。
      Defaults to "" (volume's root).
```

为了实现不同环境使用不 的配置，可以通过下现方式实现

- 在configmap 对应的配置文件中使用环境变量，格式：${VAR_NAME}，本质上configmap的成为了模板文件
- 在deployment等通过env定义变量，并引用configmap实现变量替换

### ConfigMap 实战案例

#### 范例：env 变量

```yaml
# 定义一个 ConfigMap，用于存储 Nginx 相关的环境变量配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-nginx-config  # ConfigMap 的名称，Pod 需要引用它
data:
  port: "10086" # 只能存储字符串类型的值，因此需要加引号
  user: "www"   # Nginx 运行用户

---
# 定义一个 Pod，并使用 ConfigMap 来注入环境变量
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-test  # Pod 名称
spec:
  containers:
    - name: configmap-env-test  # 容器名称
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0 # Nginx 镜像
      env:
        - name: NGINX_HOST
          value: "10.0.0.100" # 直接赋值，手动指定 Nginx 监听地址

        - name: NGINX_PORT
          valueFrom:
            configMapKeyRef:
              name: cm-nginx-config  # 引用的 ConfigMap 名称
              key: port               # 取 ConfigMap 中的 `port` 值
              optional: true          # 允许此键不存在（如果 ConfigMap 中无 `port`，则跳过）

        - name: NGINX_USER
          valueFrom:
            configMapKeyRef:
              name: cm-nginx-config  # 引用的 ConfigMap 名称
              key: user               # 取 ConfigMap 中的 `user` 值
              optional: false         # 此键必须存在，否则 Pod 创建失败
              
#配置解析：这里面我们可以使用两种方式在pod中传递变量       
```

```bash
[root@master1 cm]#kubectl apply -f storage-configmap-simple-env.yaml
configmap/cm-nginx-config created
pod/configmap-env-test created
[root@master1 cm]#kubectl get pod
NAME                 READY   STATUS    RESTARTS   AGE
configmap-env-test   1/1     Running   0          6s
[root@master1 cm]#kubectl get cm cm-nginx-config 
NAME              DATA   AGE
cm-nginx-config   2      22s

[root@master1 cm]#kubectl get pod
NAME                 READY   STATUS    RESTARTS   AGE
configmap-env-test   1/1     Running   0          119s
[root@master1 cm]#kubectl exec configmap-env-test -- env
...
NGINX_HOST=10.0.0.100
NGINX_PORT=10086
NGINX_USER=www

#缺陷：后续修改configmap变量，不重新加载pod，变量不会生效
```

#### 范例：env 变量

```yaml
# 定义 ConfigMap（配置数据）
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-pod-test   # ConfigMap 的名称
  namespace: default  # 作用的命名空间
data:
  host: 0.0.0.0   # Nginx 监听地址
  port: "8888"    # 端口（必须是字符串格式）

---
# 定义 Pod，并从 ConfigMap 获取环境变量
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-demo  # Pod 名称
  namespace: default        # 作用的命名空间
spec:
  containers:
    - name: pod-test  # 容器名称
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1 # 镜像
      env:
        - name: HOST
          valueFrom:
            configMapKeyRef:
              name: cm-pod-test   # 关联的 ConfigMap 名称
              key: host           # 取 ConfigMap 中的 `host` 值
              optional: true      # true 表示 key 不存在时，仍然能启动

        - name: PORT
          valueFrom:
            configMapKeyRef:
              name: cm-pod-test   # 关联的 ConfigMap 名称
              key: port           # 取 ConfigMap 中的 `port` 值
              optional: false     # false 表示 key 必须存在，否则 Pod 失败
```

```bash
[root@master1 cm]#kubectl apply -f storage-configmap-valueFrom-env.yaml
configmap/cm-pod-test created
pod/configmap-env-demo created
[root@master1 cm]#kubectl get pod
NAME                 READY   STATUS    RESTARTS   AGE
configmap-env-demo   1/1     Running   0          5s
[root@master1 cm]#kubectl get cm
NAME               DATA   AGE
cm-pod-test        2      8s
kube-root-ca.crt   1      11d
[root@master1 cm]#kubectl exec configmap-env-demo -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=configmap-env-demo
HOST=0.0.0.0
PORT=8888

[root@master1 cm]#kubectl exec configmap-env-demo -- ss -ntl
State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port  Process  
LISTEN   0        128              0.0.0.0:8888          0.0.0.0:*              

[root@master1 cm]#kubectl get pod -o wide 
NAME                 READY   STATUS    RESTARTS   AGE   IP             NODE             NOMINATED NODE   READINESS GATES
configmap-env-demo   1/1     Running   0          12m   10.244.1.191   node1.kang.org   <none>           <none>
[root@master1 cm]#curl 10.244.1.191
curl: (7) Failed to connect to 10.244.1.191 port 80 after 0 ms: Connection refused
[root@master1 cm]#curl 10.244.1.191:8888
kubernetes pod-test v0.1!! ClientIP: 10.244.0.0, ServerName: configmap-env-demo, ServerIP: 10.244.1.191!
```

#### 范例：env 变量

可以基于configmap给pod中的命令执行，以变量的方式，传递一些参数

```yaml
# 定义 ConfigMap，存储时间参数
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-command-arg  # ConfigMap 名称
data:
  time: "3600"  # 休眠时间，字符串格式

---
# 定义 Pod，使用 ConfigMap 作为环境变量
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-command-arg  # Pod 名称
spec:
  containers:
    - name: pod-cm-command-arg-container  # 容器名称
      image: busybox:1.32.0  # 使用 BusyBox 镜像
      command: [ "/bin/sh", "-c", "sleep ${SPECIAL_TIME}" ]  # 读取环境变量执行 sleep
      env:
        - name: SPECIAL_TIME
          valueFrom:
            configMapKeyRef:
              name: cm-command-arg  # 引用 ConfigMap
              key: time             # 指定 ConfigMap 中的 `time` 键

        - name: NAME  # 直接赋值的环境变量
          value: "hezhaokang"

  restartPolicy: Never  # Pod 运行完毕后不自动重启
```

```bash
[root@master1 cm]#vim storage-configmap-simple-evcargs.yaml
[root@master1 cm]#kubectl apply -f storage-configmap-simple-evcargs.yaml
configmap/cm-command-arg created
pod/pod-cm-command-arg created
[root@master1 cm]#kubectl get pod
NAME                 READY   STATUS    RESTARTS   AGE
pod-cm-command-arg   1/1     Running   0          6s
[root@master1 cm]#kubectl get cm
NAME               DATA   AGE
cm-command-arg     1      12s

[root@master1 cm]#kubectl exec pod-cm-command-arg -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=pod-cm-command-arg
SPECIAL_TIME=3600
NAME=hezhaokang

[root@master1 cm]#kubectl exec pod-cm-command-arg -- ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 3600
```

#### 范例：envFrom 批量导入所有变量

```yaml
# 定义 ConfigMap，存储 Nginx 相关环境变量
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-nginx  # ConfigMap 名称
data:
  NGINX_PORT: "10086"  # 端口号（字符串格式）
  NGINX_USER: "www"    # 运行用户

---
# 定义 Pod，并通过 `envFrom` 从 ConfigMap 读取环境变量
apiVersion: v1
kind: Pod
metadata:
  name: configmap-envfrom-test  # Pod 名称
spec:
  containers:
    - name: configmap-envfrom-test  # 容器名称
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # 使用 Nginx 镜像
      envFrom:
        - configMapRef:
            name: cm-nginx  # 引用 ConfigMap，自动注入所有键值对作为环境变量


#配置解析：envFrom 方式使用cm里面所有的属性信息，而且变量名是默认的
```

```bash
[root@master1 cm]#vim storage-configmap-simple-envfrom.yaml 
[root@master1 cm]#kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
configmap-envfrom-test   1/1     Running   0          27s
[root@master1 cm]#kubectl get cm
NAME               DATA   AGE
cm-nginx           2      86s

[root@master1 cm]#kubectl exec configmap-envfrom-test -- env
...
NGINX_PORT=10086
NGINX_USER=www

#注意：这里的变量名和变量全是ConfigMap定义
```

#### 范例：volume 生成配置文件并更新生效

```yaml
# 定义 ConfigMap，并存储为文件
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-volume  # ConfigMap 名称
data:
  author.txt: hezhaokang  # 作为文件 `author.txt` 的内容
  file.conf: |
    [app]
    config1
    config2

---
# 定义 Pod，并挂载 ConfigMap 为 Volume
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-test  # Pod 名称
spec:
  volumes:
    - name: volume-config  # 指定卷名
      configMap:
        name: cm-volume  # 关联 ConfigMap

  containers:
    - name: nginx  # 容器名称
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # 使用 Nginx 镜像
      volumeMounts:
        - name: volume-config  # 挂载 ConfigMap 作为 Volume
          mountPath: /cmap/    # Pod 内的挂载路径
```

```bash
[root@master1 cm]#kubectl get pod
NAME              READY   STATUS    RESTARTS   AGE
pod-volume-test   1/1     Running   0          5s
[root@master1 cm]#kubectl get cm
NAME               DATA   AGE
cm-volume          2      7s
kube-root-ca.crt   1      12d
[root@master1 cm]#kubectl exec pod-volume-test -- ls /cmap/
author.txt
file.conf
[root@master1 cm]#kubectl exec pod-volume-test -- cat /cmap/author.txt
hezhaokang
[root@master1 cm]#kubectl exec pod-volume-test -- cat /cmap/file.conf
[app]
config1
config2
```

**动态更新**

```bash
[root@master1 cm]#kubectl edit cm cm-volume 

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  author.txt: hzk	#修改此行
  file.conf: |
    [app]
    config1
    config2
    config3		#增加此行
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"author.txt":"wangxiaochun","file.conf":"[app]      # 配置文件内容\nconfig1\nconfig2\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"cm-volume","namespace":"default"}}
  creationTimestamp: "2025-04-03T08:24:49Z"
  name: cm-volume
  namespace: default
  resourceVersion: "585153"
  uid: a2088740-0b82-48af-9c0e-f3b13cefbb8c
  
#等待一会
[root@master1 cm]#kubectl exec pod-volume-test -- cat /cmap/author.txt
hzk
[root@master1 cm]#kubectl exec pod-volume-test -- cat /cmap/file.conf
[app]
config1
config2
config3

#更新机制
[root@master1 cm]#kubectl exec pod-volume-test -- ls -l /cmap/
total 0
lrwxrwxrwx 1 root root 17 Apr  3 08:24 author.txt -> ..data/author.txt
lrwxrwxrwx 1 root root 16 Apr  3 08:24 file.conf -> ..data/file.conf

[root@master1 cm]#kubectl exec pod-volume-test -- ls -la /cmap/
total 12
drwxrwxrwx 3 root root 4096 Apr  3 08:29 .
drwxr-xr-x 1 root root 4096 Apr  3 08:24 ..
drwxr-xr-x 2 root root 4096 Apr  3 08:29 ..2025_04_03_08_29_44.3311047242
lrwxrwxrwx 1 root root   32 Apr  3 08:29 ..data -> ..2025_04_03_08_29_44.3311047242
lrwxrwxrwx 1 root root   17 Apr  3 08:24 author.txt -> ..data/author.txt
lrwxrwxrwx 1 root root   16 Apr  3 08:24 file.conf -> ..data/file.conf

#真正的文件放在这个时间戳里面
[root@master1 cm]#kubectl exec pod-volume-test -- ls  /cmap/..2025_04_03_08_29_44.3311047242
author.txt
file.conf
```

#### 范例：volume 挂载全部内容

**文件挂文件夹**

```bash
#准备配置文件
[root@master1 nginx-conf.d]#ls
default.conf  myserver.conf  myserver-gzip.cfg  myserver-status.cfg
[root@master1 nginx-conf.d]#cat default.conf 
server {
    listen       8080;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
[root@master1 nginx-conf.d]#cat myserver.conf 
server {
    listen 8888;
    server_name www.kang.org;
    include /etc/nginx/conf.d/myserver-*.cfg;
    location / {
        root /usr/share/nginx/html;
    }
}
[root@master1 nginx-conf.d]#cat myserver-gzip.cfg 
gzip on;
gzip_comp_level 5;
gzip_proxied     expired no-cache no-store private auth;
gzip_types text/plain text/css application/xml text/javascript;
[root@master1 nginx-conf.d]#cat myserver-status.cfg 
location /nginx-status {
    stub_status on;
    access_log off;
}
```

```bash
#创建CM
[root@master1 yaml]#kubectl create configmap cm-nginx-conf-files --from-file=nginx-conf.d/default.conf --dry-run=client -o yaml > cm/cm-nginx-conf-files.yaml
[root@master1 yaml]#cat cm/storage-configmap-nginx-file.yaml
apiVersion: v1
data:
  default.conf: |
    server {
        listen       8080;
        server_name  localhost;
        #access_log  /var/log/nginx/host.access.log  main;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm-nginx-conf-files


[root@master1 yaml]#kubectl apply -f cm/cm-nginx-conf-files.yaml
configmap/cm-nginx-conf-files created

#查看
[root@master1 yaml]#kubectl get cm
NAME                  DATA   AGE
cm-nginx-conf-files   1      69s
kube-root-ca.crt      1      12d
```

```yaml
#准备清单文件
# 定义 ConfigMap，存储 Nginx 主页内容
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-nginx-index  # ConfigMap 名称
data:
  index.html: "Nginx Configmap page!"  # 单行 HTML 内容

---
# 定义 Pod，挂载多个 ConfigMap 作为 Volume
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx-conf-configmap  # Pod 名称
spec:
  volumes:
    - name: nginx-conf  # 挂载 Nginx 配置文件的卷
      configMap:
        name: cm-nginx-conf-files  # 关联 ConfigMap
        optional: false  # 该 ConfigMap 必须存在，否则 Pod 启动失败

    - name: nginx-index  # 挂载 Nginx 主页文件的卷
      configMap:
        name: cm-nginx-index  # 关联 ConfigMap
        optional: false  # 该 ConfigMap 必须存在，否则 Pod 启动失败

  containers:
    - name: nginx  # 容器名称
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # Nginx 镜像

      volumeMounts:
        - name: nginx-conf  # 挂载 Nginx 配置文件
          mountPath: /etc/nginx/conf.d/
          readOnly: true  # 只读，防止容器修改配置

        - name: nginx-index  # 挂载 Nginx 主页文件
          mountPath: /usr/share/nginx/html/
          readOnly: true  # 只读，防止容器修改主页
```

```bash
[root@master1 yaml]#kubectl apply -f cm/storage-configmap-nginx-file.yaml
configmap/cm-nginx-index created
pod/pod-nginx-conf-configmap created
[root@master1 yaml]#kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
pod-nginx-conf-configmap   1/1     Running   0          4s
[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- ls -l /etc/nginx/conf.d
total 0
lrwxrwxrwx 1 root root 19 Apr  3 08:53 default.conf -> ..data/default.conf
[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- ls -l /usr/share/nginx/html/
total 0
lrwxrwxrwx 1 root root 17 Apr  3 08:53 index.html -> ..data/index.html

[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- cat /etc/nginx/conf.d/default.conf
server {
    listen       8080;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- cat /usr/share/nginx/html/index.html
Nginx Configmap page!

#多重软连接，支持动态更新，但是应用程序要支持热加载
[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- ls -la /usr/share/nginx/html/
total 12
drwxrwxrwx 3 root root 4096 Apr  3 08:53 .
drwxr-xr-x 3 root root 4096 May 12  2021 ..
drwxr-xr-x 2 root root 4096 Apr  3 08:53 ..2025_04_03_08_53_07.4235756882
lrwxrwxrwx 1 root root   32 Apr  3 08:53 ..data -> ..2025_04_03_08_53_07.4235756882
lrwxrwxrwx 1 root root   17 Apr  3 08:53 index.html -> ..data/index.html
[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- ls -la /etc/nginx/conf.d
total 12
drwxrwxrwx 3 root root 4096 Apr  3 08:53 .
drwxr-xr-x 3 root root 4096 May 12  2021 ..
drwxr-xr-x 2 root root 4096 Apr  3 08:53 ..2025_04_03_08_53_07.3935360462
lrwxrwxrwx 1 root root   32 Apr  3 08:53 ..data -> ..2025_04_03_08_53_07.3935360462
lrwxrwxrwx 1 root root   19 Apr  3 08:53 default.conf -> ..data/default.conf
```

**测试**

```bash
[root@master1 yaml]#kubectl get pod -o wide 
NAME                       READY   STATUS    RESTARTS   AGE     IP             NODE             NOMINATED NODE   READINESS GATES
pod-nginx-conf-configmap   1/1     Running   0          6m17s   10.244.1.195   node1.kang.org   <none>           <none>

[root@master1 yaml]#curl 10.244.1.195
curl: (7) Failed to connect to 10.244.1.195 port 80 after 1 ms: Connection refused
[root@master1 yaml]#curl 10.244.1.195:8080
Nginx Configmap page!
```

**动态更新**

```bash
#方法一
[root@master1 yaml]#kubectl edit cm cm-nginx-conf-files 

        server_name  localhost;
        #access_log  /var/log/nginx/host.access.log  main;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"default.conf":"server {\n    listen       8080;\n    server_name  localhost;\n    #access_log  /var/log/nginx/host.access.log  main;\n    location / {\n        root   /usr/share/nginx/html;\n        index  index.html index.htm;\n    }\n    error_page   500 502 503 504  /50x.html;\n    location = /50x.html {\n        root   /usr/share/nginx/html;\n    }\n}\n"},"kind":"ConfigMap","metadata":{"annotations":{},"creationTimestamp":null,"name":"cm-nginx-conf-files","namespace":"default"}}
  creationTimestamp: "2025-04-03T08:49:04Z"
  name: cm-nginx-conf-files
  namespace: default
  resourceVersion: "589111"
  uid: 1fb4dff7-1271-4bbc-84ba-2c2f9b2a70bc

[root@master1 yaml]#kubectl edit cm cm-nginx-index 

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  index.html: Nginx Configmap page v2.0!
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"index.html":"Nginx Configmap page!"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"cm-nginx-index","namespace":"default"}}
  creationTimestamp: "2025-04-03T08:53:07Z"
  name: cm-nginx-index
  namespace: default
  resourceVersion: "601141"
  uid: 7cfd2777-1cd2-4c52-b768-e0d97d0d39ba


#方法二
#修改cm的配置文件
[root@master1 yaml]#vim cm/cm-nginx-conf-files.yaml 
[root@master1 yaml]#kubectl apply -f cm/cm-nginx-conf-files.yaml
```

```bash
#查看，已经更新了
[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- cat /etc/nginx/conf.d/default.conf
server {
    listen       9999;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- cat /usr/share/nginx/html/index.html
Nginx Configmap page v2.0!
```

```bash
#动态更新了，nginx的配置文件更新没有生效，但是静态页面更新生效了，nginx不支持热加载
[root@master1 yaml]#kubectl get pod -o wide 
NAME                       READY   STATUS    RESTARTS   AGE   IP             NODE             NOMINATED NODE   READINESS GATES
pod-nginx-conf-configmap   1/1     Running   0          74m   10.244.1.195   node1.kang.org   <none>           <none>

[root@master1 yaml]#curl 10.244.1.195:9999
curl: (7) Failed to connect to 10.244.1.195 port 9999 after 0 ms: Connection refused
[root@master1 yaml]#curl 10.244.1.195:8080
Nginx Configmap page v2.0!
```

```bash
#手动更新nginx配置文件
[root@master1 yaml]#kubectl exec pod-nginx-conf-configmap -- nginx -s reload
2025/04/03 10:10:23 [notice] 122#122: signal process started
[root@master1 yaml]#curl 10.244.1.195:9999
Nginx Configmap page v2.0!
```

#### 范例：volume 挂载 CM 中部分文件并支持动态更新

范例：下面方式指定configmap的指定key对应的文件挂载至Pod中，在Pod目录中只能看到前面特定的配置文件，pod中目录原有的配置文件将被隐藏，支持动态更新

**文件挂文件夹**

```bash
#准备配置文件
[root@master1 nginx-conf.d]#ls
default.conf  myserver.conf  myserver-gzip.cfg  myserver-status.cfg
[root@master1 nginx-conf.d]#cat default.conf 
server {
    listen       8080;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
[root@master1 nginx-conf.d]#cat myserver.conf 
server {
    listen 8888;
    server_name www.kang.org;
    include /etc/nginx/conf.d/myserver-*.cfg;
    location / {
        root /usr/share/nginx/html;
    }
}

#子配置文件,注意:文件是以cfg为后缀,不能以conf文件后缀,会导致冲突
[root@master1 nginx-conf.d]#cat myserver-gzip.cfg 
gzip on;
gzip_comp_level 5;
gzip_proxied     expired no-cache no-store private auth;
gzip_types text/plain text/css application/xml text/javascript;
#子配置文件,注意:文件是以cfg为后缀,不能以conf文件后缀,会导致冲突
[root@master1 nginx-conf.d]#cat myserver-status.cfg 
location /nginx-status {
    stub_status on;
    access_log off;
}
```

```bash
#创建cm
[root@master1 yaml]#kubectl create cm cm-nginx-conf-files --from-file=nginx-conf.d/ --dry-run=client -o yaml > cm/cm-nginx-conf-files.yaml
[root@master1 yaml]#cat cm/cm-nginx-conf-files.yaml
apiVersion: v1
data:
  default.conf: |
    server {
        listen       8080;
        server_name  localhost;
        #access_log  /var/log/nginx/host.access.log  main;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
  myserver-gzip.cfg: |
    gzip on;
    gzip_comp_level 5;
    gzip_proxied     expired no-cache no-store private auth;
    gzip_types text/plain text/css application/xml text/javascript;
  myserver-status.cfg: |
    location /nginx-status {
        stub_status on;
        access_log off;
    }
  myserver.conf: |
    server {
        listen 8888;
        server_name www.kang.org;
        include /etc/nginx/conf.d/myserver-*.cfg;
        location / {
            root /usr/share/nginx/html;
        }
    }
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm-nginx-conf-files
[root@master1 yaml]#kubectl apply -f cm/cm-nginx-conf-files.yaml
configmap/cm-nginx-conf-files created

[root@master1 yaml]#kubectl get cm cm-nginx-conf-files 
NAME                  DATA   AGE
cm-nginx-conf-files   4      19s
```

```yaml
#清单文件
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-nginx-index  # ConfigMap 的名称，用于提供网页首页内容
data:
  index.html: "Nginx Sub Configmap Page!\n"  # index.html 页面内容，将挂载到 Nginx 的 html 目录中
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-nginx-conf  # Pod 的名称
spec:
  volumes:
    - name: nginx-conf  # 第一个卷，挂载配置文件
      configMap:
        name: cm-nginx-conf-files  # 引用的 ConfigMap 名称，包含多个配置项
        items:  # 指定挂载 ConfigMap 中的哪些 key，并自定义为容器内的文件名
          - key: myserver.conf  # ConfigMap 中的键（文件名）
            path: myserver2.conf  # 挂载到容器中后的文件名
            mode: 0644  # 文件权限（rw-r--r--）
          - key: myserver-status.cfg  # 第二个配置文件
            path: myserver-status2.cfg
            mode: 0644
          - key: myserver-gzip.cfg  # 第三个配置文件
            path: myserver-gzip2.cfg
            mode: 0644
        optional: false  # 表示这个文件必须存在，否则创建 Pod 会失败
    - name: nginx-index  # 第二个卷，挂载默认网页首页
      configMap:
        name: cm-nginx-index  # 引用上方创建的 cm-nginx-index ConfigMap
        optional: false  # 该 ConfigMap 必须存在，否则创建失败
  containers:
    - image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # 容器使用的 Nginx 镜像
      name: pod-cm-nginx-conf-container  # 容器名称
      volumeMounts:
        - name: nginx-conf  # 挂载 nginx 配置文件卷
          mountPath: /etc/nginx/conf.d/  # 挂载到容器内的 Nginx 配置路径
          readOnly: true  # 挂载为只读
        - name: nginx-index  # 挂载首页文件卷
          mountPath: /usr/share/nginx/html/  # Nginx 默认网页根目录
          readOnly: true  # 挂载为只读

```

```bash
[root@master1 yaml]#vim cm/storage-configmap-nginx-subfile.yaml
[root@master1 yaml]#kubectl apply -f cm/storage-configmap-nginx-subfile.yaml
configmap/cm-nginx-index created
pod/pod-cm-nginx-conf created
[root@master1 yaml]#kubectl get pod
NAME                READY   STATUS    RESTARTS   AGE
pod-cm-nginx-conf   1/1     Running   0          6s
[root@master1 yaml]#kubectl get cm
NAME                  DATA   AGE
cm-nginx-conf-files   4      29s
cm-nginx-index        1      10s
kube-root-ca.crt      1      12d
[root@master1 yaml]#kubectl get cm cm-nginx-index -o yaml
apiVersion: v1
data:
  index.html: |
    Nginx Sub Configmap Page!
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"index.html":"Nginx Sub Configmap Page!\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"cm-nginx-index","namespace":"default"}}
  creationTimestamp: "2025-04-04T06:34:04Z"
  name: cm-nginx-index
  namespace: default
  resourceVersion: "612521"
  uid: 216a6010-56d4-4dd5-96e4-7697ad40bd9d
```

```bash
[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- ls /etc/nginx/conf.d
myserver-gzip2.cfg
myserver-status2.cfg
myserver2.conf

#查看权限是644
[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- ls -l /etc/nginx/conf.d/..data/
total 12
-rw-r--r-- 1 root root 149 Apr  4 06:44 myserver-gzip2.cfg
-rw-r--r-- 1 root root  67 Apr  4 06:44 myserver-status2.cfg
-rw-r--r-- 1 root root 163 Apr  4 06:44 myserver2.conf

[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- cat /usr/share/nginx/html/index.html
Nginx Sub Configmap Page!
[root@master1 yaml]#curl 10.244.1.198:8888
Nginx Sub Configmap Page!
```

```bash
#更新配置
[root@master1 yaml]#kubectl edit cm cm-nginx-index 

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  index.html: |
    Nginx Sub Configmap Page v2.0!		#修改此行
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"index.html":"Nginx Sub Configmap Page!\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"cm-nginx-index","namespace":"default"}}
  creationTimestamp: "2025-04-04T06:34:04Z"
  name: cm-nginx-index
  namespace: default
  resourceVersion: "612521"
  uid: 216a6010-56d4-4dd5-96e4-7697ad40bd9d

#更新完立即生效
[root@master1 yaml]#curl 10.244.1.198:8888
Nginx Sub Configmap Page v2.0!
```

```bash
#更新配置
[root@master1 yaml]#kubectl edit cm cm-nginx-conf-files 
...
  myserver.conf: |
    server {
        listen 9999;
        server_name www.kang.org;
        include /etc/nginx/conf.d/myserver-*.cfg;
        location / {
            root /usr/share/nginx/html;
        }
    }
kind: ConfigMap
...

#时间戳文件
[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- ls -la /etc/nginx/conf.d/
total 12
drwxrwxrwx 3 root root 4096 Apr  4 06:44 .
drwxr-xr-x 3 root root 4096 May 12  2021 ..
drwxr-xr-x 2 root root 4096 Apr  4 06:44 ..2025_04_04_06_44_33.551219154
lrwxrwxrwx 1 root root   31 Apr  4 06:44 ..data -> ..2025_04_04_06_44_33.551219154
lrwxrwxrwx 1 root root   25 Apr  4 06:34 myserver-gzip2.cfg -> ..data/myserver-gzip2.cfg
lrwxrwxrwx 1 root root   27 Apr  4 06:34 myserver-status2.cfg -> ..data/myserver-status2.cfg
lrwxrwxrwx 1 root root   21 Apr  4 06:34 myserver2.conf -> ..data/myserver2.conf


[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- cat /etc/nginx/conf.d/myserver2.conf
server {
    listen 9999;
    server_name www.kang.org;
    include /etc/nginx/conf.d/myserver-*.cfg;
    location / {
        root /usr/share/nginx/html;
    }
}

#不支持热加载
[root@master1 yaml]#curl 10.244.1.198:9999
curl: (7) Failed to connect to 10.244.1.198 port 9999 after 1 ms: Connection refused
[root@master1 yaml]#curl 10.244.1.198:8888
Nginx Sub Configmap Page v2.0!

[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- nginx -s reload
2025/04/04 06:49:20 [notice] 148#148: signal process started

[root@master1 yaml]#curl 10.244.1.198:9999
Nginx Sub Configmap Page v2.0!
```



#### 范例：volume 基于subpath实现挂载CM部分文件并修改配置文件名称但不支持动态更新

下面方式指定configmap的指定key对应的文件挂载至Pod中，在Pod目录中只能看到前面特定的配置文件，pod中目录原有的配置文件不会被隐藏

**文件挂文件**

```bash
#准备配置文件
[root@master1 nginx-conf.d]#ls
default.conf  myserver.conf  myserver-gzip.cfg  myserver-status.cfg
[root@master1 nginx-conf.d]#cat default.conf 
server {
    listen       8080;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
[root@master1 nginx-conf.d]#cat myserver.conf 
server {
    listen 8888;
    server_name www.kang.org;
    include /etc/nginx/conf.d/myserver-*.cfg;
    location / {
        root /usr/share/nginx/html;
    }
}

#子配置文件,注意:文件是以cfg为后缀,不能以conf文件后缀,会导致冲突
[root@master1 nginx-conf.d]#cat myserver-gzip.cfg 
gzip on;
gzip_comp_level 5;
gzip_proxied     expired no-cache no-store private auth;
gzip_types text/plain text/css application/xml text/javascript;
#子配置文件,注意:文件是以cfg为后缀,不能以conf文件后缀,会导致冲突
[root@master1 nginx-conf.d]#cat myserver-status.cfg 
location /nginx-status {
    stub_status on;
    access_log off;
}
```

```bash
#创建cm
[root@master1 yaml]#kubectl create cm cm-nginx-conf-files --from-file=nginx-conf.d/ --dry-run=client -o yaml > cm/cm-nginx-conf-files.yaml
[root@master1 yaml]#cat cm/cm-nginx-conf-files.yaml
apiVersion: v1
data:
  default.conf: |
    server {
        listen       8080;
        server_name  localhost;
        #access_log  /var/log/nginx/host.access.log  main;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
  myserver-gzip.cfg: |
    gzip on;
    gzip_comp_level 5;
    gzip_proxied     expired no-cache no-store private auth;
    gzip_types text/plain text/css application/xml text/javascript;
  myserver-status.cfg: |
    location /nginx-status {
        stub_status on;
        access_log off;
    }
  myserver.conf: |
    server {
        listen 8888;
        server_name www.kang.org;
        include /etc/nginx/conf.d/myserver-*.cfg;
        location / {
            root /usr/share/nginx/html;
        }
    }
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm-nginx-conf-files
[root@master1 yaml]#kubectl apply -f cm/cm-nginx-conf-files.yaml
configmap/cm-nginx-conf-files created

[root@master1 yaml]#kubectl get cm cm-nginx-conf-files 
NAME                  DATA   AGE
cm-nginx-conf-files   4      19s
```

```yaml
#清单文件
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-nginx-index
data:
  index.html: "Nginx Use Sub Configmap Page!\n"  # Nginx 首页内容，将挂载到 /usr/share/nginx/html/

---
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-nginx-conf
spec:
  volumes:
    - name: nginx-conf  # 卷名称，用于挂载 Nginx 配置文件
      configMap:
        name: cm-nginx-conf-files  # ConfigMap 的名称
        optional: false  # 该 ConfigMap 必须存在，否则 Pod 创建失败
    - name: nginx-index  # 卷名称，用于挂载网页首页内容
      configMap:
        name: cm-nginx-index  # 引用之前定义的 cm-nginx-index ConfigMap
        optional: false

  containers:
    - image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # 使用的自定义 Nginx 镜像
      name: pod-cm-nginx-conf-container
      volumeMounts:
        # 挂载 myserver.conf 文件
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d/myserver2.conf  # 修改后的文件名
          subPath: myserver.conf  # 只挂载 ConfigMap 中的 myserver.conf 文件
          readOnly: true
        
        # 挂载 myserver-gzip.cfg 文件
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d/myserver-gzip2.cfg  # 修改后的文件名
          subPath: myserver-gzip.cfg  # 只挂载 ConfigMap 中的 myserver-gzip.cfg 文件
          readOnly: true
        
        # 挂载 index.html 文件
        - name: nginx-index
          mountPath: /usr/share/nginx/html/  # Nginx 默认网页内容目录
          readOnly: true
```

```bash
[root@master1 yaml]#kubectl apply -f cm/storage-configmap-nginx-usesnfile.yaml
configmap/cm-nginx-index created
pod/pod-cm-nginx-conf created
[root@master1 yaml]#kubectl get pod
NAME                READY   STATUS    RESTARTS   AGE
pod-cm-nginx-conf   1/1     Running   0          6s
[root@master1 yaml]#kubectl get cm
NAME                  DATA   AGE
cm-nginx-conf-files   4      10m
cm-nginx-index        1      12s
kube-root-ca.crt      1      12d

[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- ls /etc/nginx/conf.d
default.conf			#这个是默认都有的文件
myserver-gzip2.cfg
myserver2.conf

#这个不是双层软连接，不支持动态更新
[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- ls /etc/nginx/conf.d -la
total 28
drwxr-xr-x 1 root root 4096 Apr  4 07:07 .
drwxr-xr-x 1 root root 4096 May 12  2021 ..
-rw-r--r-- 1 root root 1093 Apr  4 07:07 default.conf
-rw-r--r-- 1 root root  149 Apr  4 07:07 myserver-gzip2.cfg
-rw-r--r-- 1 root root  163 Apr  4 07:07 myserver2.conf

#这个是双层软连接，支持动态更新
[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- ls /usr/share/nginx/html -la
total 12
drwxrwxrwx 3 root root 4096 Apr  4 07:07 .
drwxr-xr-x 3 root root 4096 May 12  2021 ..
drwxr-xr-x 2 root root 4096 Apr  4 07:07 ..2025_04_04_07_07_35.303199839
lrwxrwxrwx 1 root root   31 Apr  4 07:07 ..data -> ..2025_04_04_07_07_35.303199839
lrwxrwxrwx 1 root root   17 Apr  4 07:07 index.html -> ..data/index.html

[root@master1 yaml]#curl 10.244.1.199
Nginx Use Sub Configmap Page!
[root@master1 yaml]#curl 10.244.1.199:8888
Nginx Use Sub Configmap Page!
```

```bash
#动态更新页面文件
[root@master1 yaml]#kubectl edit cm cm-nginx-index 

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  index.html: |
    Nginx Use Sub Configmap Page v2.0!
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"index.html":"Nginx Use Sub Configmap Page!\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"cm-nginx-index","namespace":"default"}}
  creationTimestamp: "2025-04-04T07:07:35Z"
  name: cm-nginx-index
  namespace: default
  resourceVersion: "617963"
  uid: 5ab91ab1-c99c-4400-bbc4-5282431de767

#等待一会
[root@master1 yaml]#curl 10.244.1.199:8888
Nginx Use Sub Configmap Page v2.0!
[root@master1 yaml]#curl 10.244.1.199
Nginx Use Sub Configmap Page v2.0!
```

```bash
#动态更新配置文件
[root@master1 yaml]#kubectl edit cm cm-nginx-conf-files 
...
  myserver.conf: |
    server {
        listen 9999;
        server_name www.kang.org;
        include /etc/nginx/conf.d/myserver-*.cfg;
...

[root@master1 yaml]#kubectl exec pod-cm-nginx-conf -- cat /etc/nginx/conf.d/myserver2.conf
server {
    listen 8888;
    server_name www.kang.org;
    include /etc/nginx/conf.d/myserver-*.cfg;
    location / {
        root /usr/share/nginx/html;
    }
}

#不支持动态更新
```

#### 范例：volume 基于subPath 挂载CM 部分文件并保留原目录中的其它文件

```bash
#查看容器内的文件列表
[root@master1 yaml]#docker run --rm --name nginx registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0 ls /etc/nginx/
conf.d
fastcgi_params
mime.types
modules
nginx.conf
scgi_params
uwsgi_params

#导出配置文件
[root@master1 yaml]#docker run  --rm --name nginx registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0 cat /etc/nginx/nginx.conf > nginx.conf
[root@master1 yaml]#cat nginx.conf 

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

```bash
#创建configmap
[root@master1 yaml]#kubectl create cm cm-nginx-conf --from-file=nginx.conf --dry-run=client -o yaml > cm/cm-nginx-conf.yaml
[root@master1 yaml]#cat cm/cm-nginx-conf.yaml
apiVersion: v1
data:
  nginx.conf: |2

    user  nginx;
    worker_processes  auto;

    error_log  /var/log/nginx/error.log notice;
    pid        /var/run/nginx.pid;


    events {
        worker_connections  1024;
    }


    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        keepalive_timeout  65;

        #gzip  on;

        include /etc/nginx/conf.d/*.conf;
    }
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm-nginx-conf

[root@master1 yaml]#kubectl apply -f cm/cm-nginx-conf.yaml
configmap/cm-nginx-conf created
[root@master1 yaml]#kubectl get cm
NAME               DATA   AGE
cm-nginx-conf      1      6s
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-nginx-conf  # Pod 的名称
spec:
  volumes:
    - name: volume-nginx-conf  # 定义一个卷，挂载 ConfigMap
      configMap:
        name: cm-nginx-conf     # 引用名为 cm-nginx-conf 的 ConfigMap

  containers:
    - image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # 使用自定义 Nginx 镜像
      name: pod-cm-nginx-conf-container  # 容器名称
      command: ["sh", "-c", "sleep 3600"]  # 覆盖默认启动命令，让容器保持运行状态，方便调试
      volumeMounts:
        - name: volume-nginx-conf
          mountPath: /etc/nginx/  # 将 ConfigMap 内容挂载到容器的 /etc/nginx/ 目录下

```

```bash
[root@master1 yaml]#vim cm/storage-configmap-subPath-nginx-1.yaml
[root@master1 yaml]#kubectl apply -f cm/storage-configmap-subPath-nginx-1.yaml
pod/pod-cm-nginx-conf created
[root@master1 yaml]#kubectl get pod
NAME                READY   STATUS    RESTARTS   AGE
pod-cm-nginx-conf   1/1     Running   0          6s

#其他文件全部覆盖，无法看到
[root@master1 yaml]#kubectl exec -it pod-cm-nginx-conf -- ls /etc/nginx
nginx.conf
```

```bash
#删除pod
[root@master1 yaml]#kubectl delete -f cm/storage-configmap-subPath-nginx-1.yaml
```

```yaml
#修改配置文件
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-nginx-conf  # Pod 名称
spec:
  volumes:
    - name: volume-nginx-conf  # 卷名称，用于挂载 ConfigMap 中的配置文件
      configMap:
        name: cm-nginx-conf  # 引用的 ConfigMap 名称
        items:
          - key: nginx.conf  # ConfigMap 中的键（文件名）
            path: etc/nginx/nginx.conf  # 必须是相对路径，决定挂载时的路径结构，供 subPath 使用

  containers:
    - image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # Nginx 镜像
      name: pod-cm-nginx-conf-container  # 容器名称
      command: ["sh", "-c", "sleep 3600"]  # 覆盖默认启动命令，保持容器运行状态
      volumeMounts:
        - name: volume-nginx-conf
          mountPath: /etc/nginx/nginx.conf  # 最终挂载到容器的 nginx.conf 位置
          subPath: etc/nginx/nginx.conf  # 必须和上面 `items.path` 保持一致，表示只挂载这一个文件
```

```bash
[root@master1 yaml]#vim cm/storage-configmap-subPath-nginx-2.yaml 
[root@master1 yaml]#kubectl apply -f cm/storage-configmap-subPath-nginx-2.yaml
pod/pod-cm-nginx-conf created
[root@master1 yaml]#kubectl exec -it pod-cm-nginx-conf -- ls /etc/nginx
conf.d		mime.types  nginx.conf	 uwsgi_params
fastcgi_params	modules     scgi_params
```

#### 范例：利用 configmap volume 实现 Redis 配置

开启 AOF 的 redis 日志

```yaml
# 定义一个 ConfigMap，包含 redis.conf 配置文件内容
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-conf  # ConfigMap 名称
data:
  redis.conf: |     # redis 配置内容，以多行文本形式写入
    appendonly yes
    requirepass 123456

---
# 定义一个 Pod，使用上述 ConfigMap 和数据卷
apiVersion: v1
kind: Pod
metadata:
  name: cm-redis-demo  # Pod 名称
spec:
  volumes:
    - name: redis-data     # 临时存储卷，用于 Redis 持久化数据
      emptyDir: {}         # 使用临时目录
    - name: redis-config   # 存放 Redis 配置的卷，挂载 ConfigMap
      configMap:
        name: redis-conf   # 引用前面的 ConfigMap 名称
        items:
          - key: redis.conf      # 指定 ConfigMap 中的键名
            path: redis.conf     # 在容器内生成的文件名

  containers:
    - name: redis  # 容器名称
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/redis:6.2.5  # Redis 镜像
      command:
        - redis-server         # 启动命令
        - "/conf/redis.conf"   # 使用挂载的配置文件
      ports:
        - containerPort: 6379  # Redis 默认端口

      volumeMounts:
        - name: redis-data
          mountPath: /data     # Redis 默认数据目录
        - name: redis-config
          mountPath: /conf     # 配置文件挂载路径，redis.conf 会出现在 /conf/redis.conf

```

```bash
[root@master1 yaml]#vim cm/storage-configmap-redis.yaml
[root@master1 yaml]#kubectl apply -f cm/storage-configmap-redis.yaml 
configmap/redis-conf created
pod/cm-redis-demo created
[root@master1 yaml]#kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
cm-redis-demo   1/1     Running   0          10s
[root@master1 yaml]#kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      12d
redis-conf         1      13s
[root@master1 yaml]#kubectl exec -it cm-redis-demo -- bash
root@cm-redis-demo:/data# redis-cli 
127.0.0.1:6379> info
NOAUTH Authentication required.
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> info
# Server
redis_version:6.2.5
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:19d2f4c94e0b6820
redis_mode:standalone
os:Linux 5.15.0-136-generic x86_64

127.0.0.1:6379> CONFIG GET appendonly
1) "appendonly"
2) "yes"

#redis也不支持热加载
```

## Secret

Secret volume是通过tmpfs（内存文件系统）实现的，所以这种类型的volume不是永久存储的。

每个Secret的数据不能超过1MB，支持通过资源限额控制每个名称空间的Secret的数量

注意: Secret 属于名称空间级别，只能被同一个名称空间的Pod引用

![image-20250404155535760](kubernetes/image-20250404155535760.png)

|     **类型**      |               **核心功能**                |                        **常见子类型**                        |
| :---------------: | :---------------------------------------: | :----------------------------------------------------------: |
|     `generic`     |   存储通用base64编码数据（密码/公钥等）   | `Opaque` `kubernetes.io/service-account-token` `kubernetes.io/basic-auth` `kubernetes.io/ssh-auth` `bootstrap.kubernetes.io/rbd` |
|       `tls`       |        专用于存储TLS/SSL证书和私钥        |                     `kubernetes.io/tls`                      |
| `docker-registry` | 用于kubelet从私有镜像仓库拉取镜像时的认证 |  `kubernetes.io/dockercfg` `kubernetes.io/dockerconfigjson`  |



|     **大类型**      |           **Builtin Type**            |                           **说明**                           |
| :-----------------: | :-----------------------------------: | :----------------------------------------------------------: |
|     **generic**     |               `Opaque`                | 存储任意用户自定义数据（如配置文件、密钥等），需手动base64编码 |
|     **generic**     | `kubernetes.io/service-account-token` | 自动生成的Service Account令牌，挂载到Pod的`/var/run/secrets/kubernetes.io/serviceaccount` |
|     **generic**     |      `kubernetes.io/basic-auth`       |              存储基本认证（用户名/密码）的凭据               |
|     **generic**     |       `kubernetes.io/ssh-auth`        |                 存储SSH认证所需的私钥或公钥                  |
|     **generic**     |    `bootstrap.kubernetes.io/token`    |       用于集群节点加入时的引导令牌（Bootstrap Token）        |
|       **tls**       |          `kubernetes.io/tls`          | 存储TLS证书（`tls.crt`）和私钥（`tls.key`），专用于HTTPS服务 |
| **docker-registry** |   `kubernetes.io/dockerconfigjson`    | 新版Docker镜像仓库认证配置（支持多仓库，存储`~/.docker/config.json`序列化数据） |
| **docker-registry** |       `kubernetes.io/dockercfg`       | 旧版Docker镜像仓库认证配置（单仓库，存储`~/.dockercfg`序列化数据） |



Secret 创建方式

- 手动创建：用户自行创建的Secret 常用来存储用户私有的一些信息
- 自动创建：集群自动创建的Secret 用来作为集群中各个组件之间通信的身份校验使用

### Secret 命令式创建

![image-20250404161819257](kubernetes/image-20250404161819257.png)

官方说明

```
https://kubernetes.io/zh-cn/docs/tasks/configmap-secret/managing-secret-using-kubectl/
```

格式

```
kubectl create secret [flags] [options] [-n <namespace>]
```

```bash
[root@master ~]# kubectl create secret -h
Create a secret using specified subcommand.
Available Commands:
docker-registry 	#创建一个给 Docker registry 使用的 secret
generic	 			#从本地 file, directory 或者 literal value 创建一个 secret
tls 				#创建一个 TLS secret
Usage:
kubectl create secret [flags] [options]
```

命令格式

```bash
#generic类型
kubectl create secret generic NAME [--type=string] [--from-file=[key=]source] [--from-literal=key1=value1]

--from-literal=key1=value1 #以命令行设置键值对的环境变量方式配置数据
--from-env-file=/PATH/FILE #以环境变量的专用文件的方式配置数据
--from-file=[key=]/PATH/FILE #以配置文件的方式创建配置数据，如不指定key，FILE名称为key名
--from-file=/PATH/DIR #以配置文件所在目录的方式创建配置数据

#该命令中的--type选项进行定义除了后面docker-registry和tls命令之外的其它子类型，有些类型有key的特定要求
#注意: 如果vaule中有特殊字符,比如:$,\,*,=,!等,需要用\进行转义或用单引号''引起来

#tls类型
kubectl create secret tls NAME --cert=/path/file --key=/path/file
#其保存cert文件内容的key名称不能指定自动为tls.crt，而保存private key的key不能指定自动为tls.key

#docker-registry类型
#方式1:基于用户名和密码方式实现
kubectl create secret docker-registry NAME --docker-username=user --docker-password=password --docker-email=email [--docker-server=string] [--from-file=[key=]source]


#方式2:基于dockerconfig文件方式实现
kubectl create secret docker-registry KEYNAME --from-file=.dockerconfigjson=path/to/.docker/config.json
#从已有的json格式的文件加载生成的就是dockerconfigjson类型，命令行直接生成的也是该类型
```

范例:

```bash
#创建generic类型
kubectl create secret generic my-secret-generic --from-file=/path/bar
kubectl create secret generic my-secret-generic --from-file=ssh-privatekey=~/.ssh/id_rsa --from-file=ssh-publickey=~/.ssh/id_rsa.pub
kubectl create secret generic my-secret-generic --from-literal=username=admin --from-literal=password=123456
kubectl create secret generic my-secret-generic --from-env-file=path/to/bar.env

#创建tls类型
kubectl create secret tls my-secret-tls --cert=certs/wang.org.cert --
key=certs/wang.org.key

#创建docker-registry类型
#参考示例：https://Kubernetesmeetup.github.io/docs/tasks/configure-pod-container/pull-image-private-registry/
#基于私有仓库的用户名和密码
kubectl create secret docker-registry my-secret-docker-registry --docker-server=harbor.wang.org --docker-username=admin --docker-password=123456 --docker-email=2352136389@qq.com

#先登录并认证到目标仓库server上，认证凭据自动保存在dockercfg文件中,基于dockerconfig文件实现
kubectl create secret docker-registry dockerharbor-auth --fromfile=.dockerconfigjson=/root/.docker/config.json

kubectl create secret generic dockerharbor-auth --type='kubernetes.io/dockerconfigjson' --from-file=.dockerconfigjson=/root/.docker/config.json
```

### Secret 声明式创建

Secret 数据存放在data或stringData字段,其中data字段中的Key/value必须使用base64编码存放,而stringData使用明文存放

Secret 资源的元数据：除了name, namespace之外，常用的还有labels, annotations

- annotation的名称遵循类似于labels的名称命名格式，但其数据长度不受限制
- 它不能用于被标签选择器作为筛选条件；但常用于为那些仍处于Beta阶段的应用程序提供临时的配置接口
- 管理命令：kubectl annotate TYPE/NAME KEY=VALUE

### Secret 引用

<img src="kubernetes/image-20250404170123995.png" alt="image-20250404170123995" style="zoom: 33%;" />

Secret资源在Pod中引用的方式有三种

- 环境变量

  引用Secret对象上特定的key，以valueFrom赋值给Pod上指定的环境变量

  在Pod上使用envFrom一次性导入Secret对象上的所有key-value，key(也可以统一附加特定前缀）即为环境变量名,value自动成为相应的变量值

  注意：容器很可能会将环境变量打印到日志中,基于安全考虑不建议以环境变量方式引用Secret中的敏感数据

- secret卷

  在Pod上将Secret对象引用为存储卷，而后整体由容器mount至某个目录下,其中key名称转为文件名，value的值转为相应的文件内容

  在Pod上定义Secret卷时，也可以仅引用其中的指定的部分key，而后由容器mount至目录下

- 拉取镜像

  在Pod 上使用 imagePullSecrets 拉取私有仓库镜像使用

  Pod引用Secret的方式：pods.spec.imagePullSecrets

### Generic 案例

generic 主要用于实现用户名和密码的加密保存

注意: generic加密保存要注意换行问题 \n

```bash
#generic类型
kubectl create secret generic NAME [--type=string] [--from-file=[key=]source] [--from-literal=key1=value1]

--from-literal=key1=value1 #以命令行设置键值对的方式配置数据
--from-env-file=/PATH/TO/FILE #以环境变量专用文件的方式配置数据
--from-file=[key=]/PATH/TO/FILE #以配置文件的方式创建配置数据，如不指定key，FILE名称为key名
--from-file=/PATH/TO/DIR #以配置文件所在目录的方式创建配置数据
#该命令中的--type选项进行定义除了后面docker-registry和tls命令之外的其它子类型，有些类型有key的特定要求
#注意: 如果vaule中有特殊字符,比如:$,\,*,=,!等,需要用\进行转义或用单引号''引起来
```

#### 范例：命令式创建 generic

```bash
#命令式创建
[root@master1 secret]#kubectl create secret generic secret-mysql-root --from-literal=username=root --from-literal=password=123456
secret/secret-mysql-root created

#查看确认创建成功
[root@master1 secret]#kubectl describe secrets secret-mysql-root 
Name:         secret-mysql-root
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
username:  4 bytes

[root@master1 secret]#kubectl get secrets 
NAME                TYPE     DATA   AGE
secret-mysql-root   Opaque   2      51s
[root@master1 secret]#kubectl get secrets -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    password: MTIzNDU2
    username: cm9vdA==
  kind: Secret
  metadata:
    creationTimestamp: "2025-04-04T09:09:37Z"
    name: secret-mysql-root
    namespace: default
    resourceVersion: "637813"
    uid: eb48bf18-6c0e-473e-8a33-d94bac0191cd
  type: Opaque
kind: List
metadata:
  resourceVersion: ""

#解码
[root@master1 secret]#echo cm9vdA== | base64 -d
root
[root@master1 secret]#echo MTIzNDU2 | base64 -d
123456
```

#### 范例: Base64 编码数据

```bash
#将数据转换成base64编码,注意:需要加-n选项去掉换行符
[root@master1 secret]#echo -n 'hzk' | base64
aHpr
[root@master1 secret]#echo -n '123456' | base64
MTIzNDU2

#清单文件
apiVersion: v1
kind: Secret
metadata:
  name: secret-data
  namespace: default
type: Opaque
data:		#data表示必须以base64编码存放,stringData表示明文存放数据
  user: aHpr
  password: MTIzNDU2
  
[root@master1 secret]#kubectl apply -f storage-secret-Opaque-data.yaml
secret/secret-data created

[root@master1 secret]#kubectl get secrets 
NAME          TYPE     DATA   AGE
secret-data   Opaque   2      61s

[root@master1 secret]#kubectl describe secrets secret-stringdata 
Name:         secret-stringdata
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
user:      3 bytes
```

```bash
#如果安装etcdctl工具,可以看到secret实际保存在etcd中
[root@master1 secret]#apt install etcd-client

```

#### 范例: stringData 明文数据

```yaml
[root@master1 secret]#cat storage-secret-Opaque-stringData.yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-stringdata
  namespace: default
type: Opaque
stringData: 		#stringData表示明文存放数据,data表示必须以base64编码存放
  user: 'hzk'
  password: '123456'
```

```bash
[root@master1 secret]#vim storage-secret-Opaque-stringData.yaml 
[root@master1 secret]#kubectl apply -f storage-secret-Opaque-stringData.yaml
secret/secret-stringdata created
[root@master1 secret]#kubectl get secrets 
NAME                TYPE     DATA   AGE
secret-data         Opaque   2      9m14s
secret-stringdata   Opaque   2      14s
[root@master1 secret]#kubectl get secrets secret-stringdata 
NAME                TYPE     DATA   AGE
secret-stringdata   Opaque   2      79s
[root@master1 secret]#kubectl get secrets secret-stringdata -o yaml
apiVersion: v1
data:
  password: MTIzNDU2	#此处显示base64编码形式
  user: aHpr			#此处显示base64编码形式

[root@master1 secret]#kubectl describe secrets secret-data 
Name:         secret-data
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
user:      3 bytes
```

#### 范例: Secret 通过环境变量为提供MySQL环境初始化的密码信息

```bash
#生成密码的base64编码
[root@master1 secret]#echo -n root | base64
cm9vdA==
[root@master1 secret]#echo -n 123456 | base64
MTIzNDU2

#清单文件，secret 通过环境为提供MySQL环境初始化的密码信息，但很不安全

[root@master1 secret]#vim storage-secret-mysql-init.yaml

# 创建一个 Secret，用于存储 MySQL 的认证信息（用户名和密码）
apiVersion: v1
kind: Secret
metadata:
  name: secret-mysql
type: kubernetes.io/basic-auth  # 指定 Secret 类型为 basic-auth（也可以用 Opaque）
# type: Opaque  # 也可以使用通用的 Opaque 类型
data:
  username: cm9vdAo=    # base64 编码的 "root"，注意最后的换行符会影响解码
  password: MTIzNDU2    # base64 编码的 "123456"

---

# 定义一个 Pod，使用上面创建的 Secret 来设置 MySQL 的 root 密码
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-mysql-init
spec:
  containers:
  - name: mysql
    image: registry.cn-beijing.aliyuncs.com/wangxiaochun/mysql:8.0.29-oracle
    env:
    - name: MYSQL_ROOT_PASSWORD              # 设置环境变量，用于初始化 MySQL 密码
      valueFrom:
        secretKeyRef:
          name: secret-mysql                # 引用的 Secret 名字
          key: password                     # 从 Secret 中取的键
          optional: false                   # 必须存在该值，否则启动失败
```

```bash
[root@master1 secret]#kubectl apply  -f storage-secret-mysql-init.yaml
secret/secret-mysql created
pod/pod-secret-mysql-init created
[root@master1 secret]#kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
pod-secret-mysql-init   1/1     Running   0          10s
[root@master1 secret]#kubectl get secrets 
NAME           TYPE                       DATA   AGE
secret-mysql   kubernetes.io/basic-auth   2      14s
[root@master1 secret]#kubectl get secrets secret-mysql -o yaml
apiVersion: v1
data:
  password: MTIzNDU2
  username: cm9vdAo=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"password":"MTIzNDU2","username":"cm9vdAo="},"kind":"Secret","metadata":{"annotations":{},"name":"secret-mysql","namespace":"default"},"type":"kubernetes.io/basic-auth"}
  creationTimestamp: "2025-04-04T09:47:33Z"
  name: secret-mysql
  namespace: default
  resourceVersion: "643974"
  uid: 976573fe-2615-4f7d-8a02-0a5a793aef81
type: kubernetes.io/basic-auth
```

```bash
#通过环境变量可以看到密码
[root@master1 secret]#kubectl exec pod-secret-mysql-init -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=pod-secret-mysql-init
MYSQL_ROOT_PASSWORD=123456
...

[root@master1 secret]#kubectl exec -it pod-secret-mysql-init -- bash
bash-4.4# mysql -uroot -p123456
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```

```bash
#环境变量不支持动态更新
[root@master1 secret]#echo -n 654321 | base64
NjU0MzIx

[root@master1 secret]#kubectl edit secrets secret-mysql 

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  password: NjU0MzIx
  username: cm9vdAo=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"password":"MTIzNDU2","username":"cm9vdAo="},"kind":"Secret","metadata":{"annotations":{},"name":"secret-mysql","namespace":"default"},"type":"kubernetes.io/basic-auth"}
  creationTimestamp: "2025-04-04T09:47:33Z"
  name: secret-mysql
  namespace: default
  resourceVersion: "643974"
  uid: 976573fe-2615-4f7d-8a02-0a5a793aef81
type: kubernetes.io/basic-auth

[root@master1 secret]#kubectl exec pod-secret-mysql-init -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=pod-secret-mysql-init
MYSQL_ROOT_PASSWORD=123456
...
```

#### 范例: 通过卷调用 secret

```bash
[root@master1 secret]#echo -n admin | base64
YWRtaW4=
[root@master1 secret]#echo -n password | base64
cGFzc3dvcmQ=

#清单文件
[root@master1 secret]#vim storage-secret-test-pod.yaml

apiVersion: v1
kind: Secret
metadata:
  name: secret-test
type: kubernetes.io/basic-auth
#type: Opaque #也可以用Opaque类型
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
  
#说明：
#type类型指定为 Opaque 或者 kubernetes.io/basic-auth,username和password是加密的信息
#资源定义文件时，与命令行创建secret的--from-literal=username=admin 效果是一样的,但命令行的变量无需加密

# 定义一个 Pod，将 Secret 以卷（volume）的形式挂载到容器中

---
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-volume  # Pod 的名称
spec:
  volumes:
    - name: secret  # 卷的名称
      secret:
        secretName: secret-test  # 引用的 Secret 名称，需事先创建好

  containers:
    - name: secret-test-container  # 容器名称
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0  # 使用 Nginx 镜像
      volumeMounts:
        - name: secret  # 绑定上面定义的卷
          mountPath: /secret/  # 指定挂载路径，Secret 中的 key 将以文件的形式出现在此目录
          readOnly: true  # 挂载为只读，防止容器内程序更改 Secret 文件
```

```bash
[root@master1 secret]#kubectl apply -f storage-secret-test-pod.yaml
secret/secret-test created
pod/pod-secret-volume created
[root@master1 secret]#kubectl get secrets 
NAME           TYPE                       DATA   AGE
secret-mysql   kubernetes.io/basic-auth   2      17m
secret-test    kubernetes.io/basic-auth   2      6s
[root@master1 secret]#kubectl get secrets secret-test 
NAME          TYPE                       DATA   AGE
secret-test   kubernetes.io/basic-auth   2      13s
[root@master1 secret]#kubectl get secrets secret-test  -o yaml 
apiVersion: v1
data:
  password: cGFzc3dvcmQ=
  username: YWRtaW4=
...

[root@master1 secret]#kubectl get pod
NAME                READY   STATUS    RESTARTS   AGE
pod-secret-volume   1/1     Running   0          91s
[root@master1 secret]#kubectl exec pod-secret-volume -- ls /secret/
password
username

[root@master1 secret]#kubectl exec pod-secret-volume -- cat /secret/password
password
[root@master1 secret]#kubectl exec pod-secret-volume -- cat /secret/username
admin

#通过两次软连接实现，支持动态更新
[root@master1 secret]#kubectl exec pod-secret-volume -- ls /secret/ -la
total 4
drwxrwxrwt 3 root root  120 Apr  4 10:05 .
drwxr-xr-x 1 root root 4096 Apr  4 10:05 ..
drwxr-xr-x 2 root root   80 Apr  4 10:05 ..2025_04_04_10_05_02.3814141662
lrwxrwxrwx 1 root root   32 Apr  4 10:05 ..data -> ..2025_04_04_10_05_02.3814141662
lrwxrwxrwx 1 root root   15 Apr  4 10:05 password -> ..data/password
lrwxrwxrwx 1 root root   15 Apr  4 10:05 username -> ..data/username

#在Pod运行的宿主机可以看到secret对应的文件
[root@node3 ~]#cat /var/lib/kubelet/pods/ca85fd7d-bced-4402-8ddc-f865ec81b5a4/volumes/kubernetes.io~secret/secret/username
admin
[root@node3 ~]#cat /var/lib/kubelet/pods/ca85fd7d-bced-4402-8ddc-f865ec81b5a4/volumes/kubernetes.io~secret/secret/password
password
```

#### 范例: 为 Service Account 创建 Secret

```yaml
#k8s-v1.24版本后,创建SA不会自动创建secret,需要手动创建secret
apiVersion: v1
kind: ServiceAccount
metadata:
  name: testsa

---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: testsa-secret
  namespace: default
  annotations:
    kubernetes.io/service-account.name: "testsa"
```

```bash
[root@master1 secret]#vim storage-secret-sa.yaml
[root@master1 secret]#kubectl apply -f storage-secret-sa.yaml
serviceaccount/testsa created
secret/testsa-secret created
[root@master1 secret]#kubectl get secrets 
NAME            TYPE                                  DATA   AGE
secret-test     kubernetes.io/basic-auth              2      31m
testsa-secret   kubernetes.io/service-account-token   3      10s
[root@master1 secret]#kubectl get sa
NAME      SECRETS   AGE
default   0         13d
testsa    0         19s
[root@master1 secret]#kubectl get secrets 
NAME            TYPE                                  DATA   AGE
secret-test     kubernetes.io/basic-auth              2      32m
testsa-secret   kubernetes.io/service-account-token   3      52s
```

```bash
#显示，base64转码的tocken
[root@master1 secret]#kubectl get secrets testsa-secret -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJWEUzMWpoaWlmMW93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBek1qSXdOelUzTXpSYUZ3MHpOVEF6TWpBd09EQXlNelJhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURVcDhqaXNBNU5KZDZOcmJKZjBnVEU3UmNVckk0TGg1NGRhcFEzckV4SU1wczZROUh6YVJydmorNDAKY0J2WmxPdHJQUUNLZDhVc0VyaDdWV1RGY2xUclRVQ1FvK1lpdWI1UW5aTC9wcnA0TldnYlA4cWNNQlBVWUwycwpoRXZBQUZxa2lLSUZoeEJTa0d3dW9DSFYvRnZ4c2haWXJjUVhQN0VSaCtHWVcrRHRraS82ZHJSbGdUdmRuajdzClF3Tmw5ZEUyc0h2RmhnNFkybDkzUWczZW5adkhzWEswbGFzbWE5c0dtZ0p6bmpvKzd2ZlkzSXFHZlBYODZNMWMKM0M2QlFvcVVUV1E2dkZONXhNZDNoQ01PMS81V1FIbnZNMXdrOC9yZ1o3TFAzV0pKVlprL28yblQrM0M3c28yRgpvQ3FFSFhIcFpuTjlXQnF0em9IQTUrY2w2ZW9WQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJSRkxxS3JoSXoxaWxGZ0w4Vm1HUm93WUpkMC9EQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQlh6NFNLTkJjYgo1N2UvTW9DSVA2bTNPaTlLSUZJdEw0blpNV1cvbk5lUTdZSmdrK1c1ZFpKL2F1MEFYV2RueUo0ekR5YktVaXZ2Cm42NzFoVC9tRTRxYXpoNzJCdC9CR3dyZlFLR0JmbERiN2lnd0Erc21TRTZTUlNwSzdQUjdlZko5anR6Z2dLVVcKRTJQL0dSTXR2dzBhSlpEMmlQRUdLOWtzMVREV1VJakRDU09Gc0JBYmQwUmJRV20wWUJNSkpXVVlnT3B5L2hHUwphcUY4dWE5STBNK00rM1NWcHU3OHdsbU5BQnFGWmhHUTJuZjFOeHlaU1N4TnFWNWNwcldGZTlyUE5Ham5ZbmxBCkhIbXVXbmV3b3A3MlQzZ2MzdWkxQkwxVndERC9zQXd0Z292Y3FHTXhQbkIyeFhZMDB6KzFXQ3NDNlZEUHdnSXYKSUhBTTROTkFudUs5Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkluQXlabHBSY2s5R1RFbGpVbVF6YTNrM1J6RkdaRE5zZEhBd2VuWTRhR0ZaWjI5TU4yUnBkelZ3U3pRaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJblJsYzNSellTMXpaV055WlhRaUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNXVZVzFsSWpvaWRHVnpkSE5oSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXpaWEoyYVdObExXRmpZMjkxYm5RdWRXbGtJam9pTURJellqYzBZVE10TnpjME5pMDBNVEk0TFRoa1ptWXROR1V5WlRoaU1EWXlZVFl5SWl3aWMzVmlJam9pYzNsemRHVnRPbk5sY25acFkyVmhZMk52ZFc1ME9tUmxabUYxYkhRNmRHVnpkSE5oSW4wLlQyM1lJYkdfX3FhZWUxTEdxRHpiRDd5VW4tSElMT1lMWjVScUthOTNvUnEydjc5Nk1uQ3RZZW9PeWs1bHV0aGNfa1F4ZFBwcnlwejJYME5ONDlXRmNUX29UaWlXRlpBcGdxUl9xTmhYbWh2d2dMbk9UMU9sV0dPUzhmNEdzbDlLNy0tWkFOUkgwd2twZWhHU0xOUFJQVUk5dWxMRXVGa2hRSUh5clZrOE96TU1ubFdJWUJZUHBpd24weEpZVUJNQmFMWE15SFROWnBXdnQxelhYWWFrWFBWWXdZVXRBcklvaU5OUjh6VkYzNmF3bFlJVmxFOEVmR1VLTVlSa0pLTDNULTUwR18xSjJSMUFzQlFWN2VhUmM0b3FsUkdjcTdoVTBfbFNfNkpDRWdESXBHUG5GdlJaWkVMbUlXcHVHRkdvNC1TMTgzeFNWWWpJRWlVUlN5M0pqdw==
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{"kubernetes.io/service-account.name":"testsa"},"name":"testsa-secret","namespace":"default"},"type":"kubernetes.io/service-account-token"}
    kubernetes.io/service-account.name: testsa
    kubernetes.io/service-account.uid: 023b74a3-7746-4128-8dff-4e2e8b062a62
  creationTimestamp: "2025-04-04T10:36:46Z"
  name: testsa-secret
  namespace: default
  resourceVersion: "651968"
  uid: 7ebe275b-ed51-4460-b220-945c0cb3c22c
type: kubernetes.io/service-account-token

#用describe显示的是base64源码的token
[root@master1 secret]#kubectl describe secrets testsa-secret 
Name:         testsa-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: testsa
              kubernetes.io/service-account.uid: 023b74a3-7746-4128-8dff-4e2e8b062a62

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1107 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InAyZlpRck9GTEljUmQza3k3RzFGZDNsdHAwenY4aGFZZ29MN2RpdzVwSzQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InRlc3RzYS1zZWNyZXQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoidGVzdHNhIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMDIzYjc0YTMtNzc0Ni00MTI4LThkZmYtNGUyZThiMDYyYTYyIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6dGVzdHNhIn0.T23YIbG__qaee1LGqDzbD7yUn-HILOYLZ5RqKa93oRq2v796MnCtYeoOyk5luthc_kQxdPprypz2X0NN49WFcT_oTiiWFZApgqR_qNhXmhvwgLnOT1OlWGOS8f4Gsl9K7--ZANRH0wkpehGSLNPRPUI9ulLEuFkhQIHyrVk8OzMMnlWIYBYPpiwn0xJYUBMBaLXMyHTNZpWvt1zXXYakXPVYwYUtArIoiNNR8zVF36awlYIVlE8EfGUKMYRkJKL3T-50G_1J2R1AsBQV7eaRc4oqlRGcq7hU0_lS_6JCEgDIpGPnFvRZZELmIWpuGFGo4-S183xSVYjIEiURSy3Jjw
```

### TLS 案例

TLS 类型的 Secret 主要用于对https场景的证书和密钥文件来进行加密传输

```bash
#tls类型格式
kubectl create secret tls NAME --cert=/path/file --key=/path/file
#注意：
#保存cert文件内容的key名称不能指定，自动为tls.crt，卷挂载后生成的文件名也为tls.crt
#保存private key文件内容的key不能指定,自动为tls.key，卷挂载后生成的文件名也为tls.key
```

![image-20250404184129008](kubernetes/image-20250404184129008.png)

下面案例实现一个基于https 的nginx的web服务

#### 创建 TLS 证书文件

```bash
#生成私钥
[root@master1 ~]#mkdir nginx-certs
[root@master1 yaml]#openssl genrsa -out nginx-certs/www.kang.org.key 2048

#生成自签名证书
[root@master1 yaml]#openssl req -new -x509 -key nginx-certs/www.kang.org.key -days 3650 -out nginx-certs/www.kang.org.crt -subj /C=CN/ST=Beijing/L=Beijing/O=DevOps/CN=www.kang.org

#查看文件
[root@master1 yaml]#ls nginx-certs/
www.kang.org.crt  www.kang.org.key
```

#### **基于 TLS 证书文件创建对应的 Secret**

```bash
[root@master1 yaml]#kubectl create secret tls secret-nginx-ssl --cert=nginx-certs/www.kang.org.crt --key=nginx-certs/www.kang.org.key 
secret/secret-nginx-ssl created

[root@master1 yaml]#kubectl create secret tls secret-nginx-ssl --cert=nginx-certs/www.kang.org.crt --key=nginx-certs/www.kang.org.key  --dry-run=client -o yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURrekNDQW51Z0F3SUJBZ0lVUTkzelZaclZuUC9LZnJGQVZqeDVSK0pPOENVd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1dURUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdNQjBKbGFXcHBibWN4RURBT0JnTlZCQWNNQjBKbAphV3BwYm1jeER6QU5CZ05WQkFvTUJrUmxkazl3Y3pFVk1CTUdBMVVFQXd3TWQzZDNMbXRoYm1jdWIzSm5NQjRYCkRUSTFNRFF3TkRFd05EVTBPVm9YRFRNMU1EUXdNakV3TkRVME9Wb3dXVEVMTUFrR0ExVUVCaE1DUTA0eEVEQU8KQmdOVkJBZ01CMEpsYVdwcGJtY3hFREFPQmdOVkJBY01CMEpsYVdwcGJtY3hEekFOQmdOVkJBb01Ca1JsZGs5dwpjekVWTUJNR0ExVUVBd3dNZDNkM0xtdGhibWN1YjNKbk1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBCk1JSUJDZ0tDQVFFQTBjc1JpajlrWkswRWl1Ykh0YjU0TExUcXAvU1I0WklYY1NLZkZrYkJvK3M5UDN0Qy9Fc3oKUmRuY2hCVlhya0FKQnJwd0F5RWRpd2dVdEdwbDZDUDlSb2dNLysrWTc1SEhpNEFRVDJUMExGZmd6Ym96bDNzVgpnSnc0b2RMTVBxc3pveW5TaFNkMHZIV1EveDN3NWdtOUVRMGI4TUtoSklqbFR0bXNHNVdvd2lJMGxsMmxMSncrCmtaaWJHZGpRS29sWWhIUzV0SksrY1drVWpDOHRTcWlCUWU5bjJidzB5b29BUVZxTEhmenVuU3dXeHlKN2hqaXIKOHRjdDg1VUVOYTJCdU16cklPWFhuMXo5b2dWbTBxTEp6Z09YdFhRaFpvcm1nRFNJK0NMd1JlRkt3NjlNMXpZTApmTCs1SlFLaHBSbDZ3ZU1vaStkVmJ3MEgzS3VBYUh1T1FRSURBUUFCbzFNd1VUQWRCZ05WSFE0RUZnUVVkQWRGCmxSZHhpMElCeU5EWjZVR2Rob3lrc0I0d0h3WURWUjBqQkJnd0ZvQVVkQWRGbFJkeGkwSUJ5TkRaNlVHZGhveWsKc0I0d0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQWY4UXVVWk1tS1VMYgpZQlE3b0NmcFhCdjRMV2lHUkY3Rk41eGJxTDVXVFp3U0pxdHJZS2JHTVc1dmxXbnBnWE52Qzg4K2hwUWl5WTVxCmhINlE1VkdjWlk2VmdWSUcxdXF4NmFHOXRvVFkxTElZR2E3YkRXZlJYTEtreUFKSnBXVnNZNmxEQnZrbmlRS1YKeCt0NEV3dzBzb1hBUHZ3dm8vUGphZnFHRkNja3Y3UFRPMHdidm81aXM3eVhIaXN1T3VDR2ZSNG81QTk5Wld5cwpjUklCUEk3SytRb2Ixd2hzR04xN3hLZ2NWN0llbnFnZ3ZKMjc5b0V4MmhSNjdoeHdkcmRHOFU4R0M1Tk9zS2tMCmZ0K3l6Qk12RW14a0VBaHU2QnBQMWdRQWF6UXZKSWNmVkhreEVOdkNBUmFYVnNSVHZIY0RlenBUUWMyaVZsOUkKM205akpETWE3dz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRRFJ5eEdLUDJSa3JRU0sKNXNlMXZuZ3N0T3FuOUpIaGtoZHhJcDhXUnNHajZ6MC9lMEw4U3pORjJkeUVGVmV1UUFrR3VuQURJUjJMQ0JTMAphbVhvSS8xR2lBei83NWp2a2NlTGdCQlBaUFFzVitETnVqT1hleFdBbkRpaDBzdytxek9qS2RLRkozUzhkWkQvCkhmRG1DYjBSRFJ2d3dxRWtpT1ZPMmF3YmxhakNJalNXWGFVc25ENlJtSnNaMk5BcWlWaUVkTG0wa3I1eGFSU00KTHkxS3FJRkI3MmZadkRUS2lnQkJXb3NkL082ZExCYkhJbnVHT0t2eTF5M3psUVExcllHNHpPc2c1ZGVmWFAyaQpCV2JTb3NuT0E1ZTFkQ0ZtaXVhQU5JajRJdkJGNFVyRHIwelhOZ3Q4djdrbEFxR2xHWHJCNHlpTDUxVnZEUWZjCnE0Qm9lNDVCQWdNQkFBRUNnZ0VBRGtES0ljYlp4cjg4ZWtWZ2x4Zm9LVDFuWGQ3UFZIQnMrRW8rWGJDVDhsT0cKbHp3RUdvanZaRlQvVnRmdVc0L05MVEJZY05YOG9GZjl0QjR3YXp1YXR3S0NRNWNmSUZPYXAzOGRYSStDeVVwMQpJdlQrRVZJUzdXaHZyb3JZUm4reUkra3cvTXZuOGUvWTJJUkx2QTA0VTZLODNiZ1RZNXlMVzdhTksxaVpIUXMvClVYVlZTK1J0RVM1bmhZcW5OMVJpZXBYQTJSTlNRandLVlFZakx6RjRRZFFzODNxdS95ckJXRkkzd2ZvTUt1LzAKSG9kM25WWWdyUEo5OFM5ZnhWZFRaZHc4VHRLODE4Z1JmNTU2c3hFUGo3Y2VpdHk2dUl2aXJKNWd2eVgweG0vTwpWa1pPTlQ3SEJCTE5CUmNSMzI2bjFZQmZkbnUwNTJLSXlEM2JBT1FXOHdLQmdRRDNqQ29tVE1xVGFKOURPdWMwCkdOZTg4VnZMSmFVRnJkbG9EZXdSM1lSWXFtTENlS09oL3ZjUHdtR201SnROS0dpQ0twQy9PWTY1S3JvR1dEN0oKTmJSb1NJR3NGeVIyNmlhd00xN1QzSmJuT3J0YVJBeURyeUwxUE9PRFJ4ZEhFTXFGYTZuWmszYXpwNllnK2VYYQo3bnl5bjdMOUVZUkxWWWJUcWczQUZQa2F4d0tCZ1FEWTlPUG9JdGlvRzBxb3pKOFBRMDZOc095aW5jQzJTZE5RClV1SWc1ZTVZYVV4RGJTSTBrWFhDZDFtMXVrVDE4bkZidUhNanQ0amIzWVY4dUJobGNycWxLK1Vmd2N2NTUzOU8KVjJVWXg2aytZVll5V095MnpDZm9mcDdTWUFXMjdTRFlmRHhHQXpXUmM2ZTMrRWZ0ejNMd012Y013VDJkdmw1RApjbTI2YXpGR3R3S0JnQTY3TEpieTR1czZ3cCtCbGRpbjBjN1FuaUhNcjdGUzB5azZDdDVWVEZicWJNNGQxNzhBCktUdXEyWnpRNWQwRmxwS0liZXl6UkNjSXYrRTR4eU82S200NlFsMDNVMWd3UHZVbFdYTktJVURKWWZUUkw5MTIKUXFKTmFVeWFvbXY4TjlFUXd4N0xnN2pWTlRHcExXQ0Y2SzNJaXlJOE5KODNDN3IwMTdTaWl1QnBBb0dBQWMzZgppMUdtM1pjRWd1ellzY2JNZzI4aU9LUDVNMHJmMy80Z2RVZVVPTlgvVlZ3MStadndEOEx3VXd1T2dVRVg0anhoCmNMMlkvTU1WMWhpdDAwSWwvVE9sUHlOY2IrS1Y4Mi9Vbm1VaXFpc2pEVjRoOGM4MjBiOGowUG9OY2h6OHRmQXEKakNhQXJvbjRyaFg1ZXlmTGUrWGdtaEdOVXA5L3B5M3ZiYXB3aVhjQ2dZRUF6V0Nua01wTUNJR0ZOMGN1RnZ5VwpLWnQwS3dFYnNkWDVRVW5aTGZ6Zm9DYSttNVZDT1JLbmVSZDd6bXV1UXhoUnhRMU9XazJScW9wbVNjcDdVNGhLCjR6MWVOajVIQktzRU93d2xzQWZRblpncnhVeDVwaVBjUFU5d0ZjMWVBMWxXcmVISkRUSlFoRGl6RjFrZGlJUEEKU1R5RzVSRVVQVkQyTHpONzNVUTJneE09Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
kind: Secret
metadata:
  creationTimestamp: null
  name: secret-nginx-ssl
type: kubernetes.io/tls
```

```bash
#查看内容
[root@master1 yaml]#kubectl get secrets 
NAME               TYPE                DATA   AGE
secret-nginx-ssl   kubernetes.io/tls   2      3m15s
```

#### 准备 Nginx 配置文件

```bash
[root@master1 yaml]#ls nginx-ssl-conf.d/
myserver.conf  myserver-gzip.cfg  myserver-status.cfg

[root@master1 yaml]#ls nginx-ssl-conf.d/
myserver.conf  myserver-gzip.cfg  myserver-status.cfg
[root@master1 yaml]#cat nginx-ssl-conf.d/myserver.conf 
server {
    listen 80;
    server_name www.kang.org; 
    return 301 https://$host$request_uri; 
}


server {
    listen 443 ssl;
    server_name www.wang.org;

    ssl_certificate /etc/nginx/certs/tls.crt; 
    ssl_certificate_key /etc/nginx/certs/tls.key;

    ssl_session_timeout 5m;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 

    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
    ssl_prefer_server_ciphers on;

    include /etc/nginx/conf.d/myserver-*.cfg;

    location / {
        root /usr/share/nginx/html;
    }
}

[root@master1 yaml]#cat nginx-ssl-conf.d/myserver-gzip.cfg
gzip on;
gzip_comp_level 5;
gzip_proxied     expired no-cache no-store private auth;
gzip_types text/plain text/css application/xml text/javascript;
[root@master1 yaml]#cat nginx-ssl-conf.d/myserver-status.cfg
location /status {
    stub_status on;
    access_log off;
}
```

#### 创建配置文件对应的 Configmap

```bash
[root@master1 yaml]#kubectl create configmap cm-nginx-ssl-conf --from-file=nginx-ssl-conf.d/ --dry-run=client -o yaml > cm/cm-nginx-ssl-conf.yaml
[root@master1 yaml]#cat cm/cm-nginx-ssl-conf.yaml
apiVersion: v1
data:
  myserver-gzip.cfg: |
    gzip on;
    gzip_comp_level 5;
    gzip_proxied     expired no-cache no-store private auth;
    gzip_types text/plain text/css application/xml text/javascript;
  myserver-status.cfg: |
    location /status {
        stub_status on;
        access_log off;
    }
  myserver.conf: "server {\n    listen 80;\n    server_name www.kang.org; \n    return
    301 https://$host$request_uri; \n}\n\n\nserver {\n    listen 443 ssl;\n    server_name
    www.wang.org;\n\n    ssl_certificate /etc/nginx/certs/tls.crt; \n    ssl_certificate_key
    /etc/nginx/certs/tls.key;\n\n    ssl_session_timeout 5m;\n\n    ssl_protocols
    TLSv1 TLSv1.1 TLSv1.2; \n\n    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    \n    ssl_prefer_server_ciphers on;\n\n    include /etc/nginx/conf.d/myserver-*.cfg;\n\n
    \   location / {\n        root /usr/share/nginx/html;\n    }\n}\n\n"
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: cm-nginx-ssl-conf
[root@master1 yaml]#kubectl apply -f cm/cm-nginx-ssl-conf.yaml
configmap/cm-nginx-ssl-conf created

[root@master1 yaml]#kubectl get cm
NAME                DATA   AGE
cm-nginx-ssl-conf   3      84s
kube-root-ca.crt    1      13d
```

#### 创建引用Secret和configmap资源配置文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx-ssl
  namespace: default
spec:
  volumes:
    - name: nginx-certs  # 用于挂载包含 SSL 证书的 Secret
      secret:
        secretName: secret-nginx-ssl  # 指定包含证书和私钥的 Secret 名称

    - name: nginx-confs  # 用于挂载 Nginx 配置的 ConfigMap
      configMap:
        name: cm-nginx-ssl-conf      # 指定包含配置的 ConfigMap 名称
        optional: false              # 配置必须存在，否则 Pod 无法启动

  containers:
    - name: nginx-ssl-server
      image: registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0
      volumeMounts:
        - name: nginx-certs
          mountPath: /etc/nginx/certs/  # 证书文件和私钥文件会自动以 tls.crt 和 tls.key 命名
          readOnly: true                # 只读挂载，保证安全性

        - name: nginx-confs
          mountPath: /etc/nginx/conf.d/  # Nginx 会读取这个目录下的 .conf 配置文件
          readOnly: true
```

```bash
[root@master1 yaml]#vim secret/storage-secret-nginx-ssl.yaml
[root@master1 yaml]#kubectl apply -f secret/storage-secret-nginx-ssl.yaml
pod/pod-nginx-ssl created
[root@master1 yaml]#kubectl get pod -o wide 
NAME            READY   STATUS    RESTARTS   AGE   IP             NODE             NOMINATED NODE   READINESS GATES
pod-nginx-ssl   1/1     Running   0          45s   10.244.1.206   node1.kang.org   <none>           <none>
```

####  验证结果

```bash
[root@master1 yaml]#curl 10.244.1.206 -I
HTTP/1.1 301 Moved Permanently
Server: nginx/1.20.0
Date: Fri, 04 Apr 2025 11:09:49 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive
Location: https://10.244.1.206/

[root@master1 yaml]#curl 10.244.1.206 -Lk
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

```bash
#修改DNS或hosts文件，添加域名解析
[root@master1 yaml]#curl -H"host: www.kang.org" 10.244.1.206 -Lk
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

#查看通过自己颁发的证书
[root@master1 yaml]#curl -H"host: www.kang.org" 10.244.1.206 -Lkv
...
*  subject: C=CN; ST=Beijing; L=Beijing; O=DevOps; CN=www.kang.org
*  issuer: C=CN; ST=Beijing; L=Beijing; O=DevOps; CN=www.kang.org
```

#### 更新证书

以上颁发的证书是有一定的有效期限的，在过期前需要提前更新证书

在线更新证书流程

- 生成新的证书和私钥文件
- 将生成的新的证书文件和私钥文件的全部内容分别先进行base64编码并删除空行
- 利用编码后的内容分别替换以下命令kubectl edit secret secret-nginx-ssl 中原有的证书信息即可

证书的有效期一般为1年,到期前需要提前更新证书

```bash
[root@master1 yaml]#curl -H"host: www.kang.org" 10.244.1.206 -Lkv
*  subject: C=CN; ST=Beijing; L=Beijing; O=DevOps; CN=www.kang.org
*  start date: Apr  4 10:45:49 2025 GMT
*  expire date: Apr  2 10:45:49 2035 GMT
*  issuer: C=CN; ST=Beijing; L=Beijing; O=DevOps; CN=www.kang.org
```

范例: 证书更新

```bash
#查看容器内是双重软连接，支持动态更新
[root@master1 nginx-certs]#kubectl exec pod-nginx-ssl -- ls -la /etc/nginx/certs
total 4
drwxrwxrwt 3 root root  120 Apr  4 11:36 .
drwxr-xr-x 1 root root 4096 Apr  4 11:08 ..
drwxr-xr-x 2 root root   80 Apr  4 11:36 ..2025_04_04_11_36_07.659583444
lrwxrwxrwx 1 root root   31 Apr  4 11:36 ..data -> ..2025_04_04_11_36_07.659583444
lrwxrwxrwx 1 root root   14 Apr  4 11:08 tls.crt -> ..data/tls.crt
lrwxrwxrwx 1 root root   14 Apr  4 11:08 tls.key -> ..data/tls.key
```

```bash
[root@master1 nginx-certs]#(umask 077; openssl genrsa -out kang.key 2048)
[root@master1 nginx-certs]#openssl req -new -x509 -key kang.key -out kang.crt -subj /C=CN/ST=Beijing/L=Beijing/O=SRE/CN=www.kang.org -days 36500
[root@master1 nginx-certs]#ll
total 24
drwxr-xr-x  2 root root 4096 Apr  4 19:32 ./
drwxr-xr-x 13 root root 4096 Apr  4 17:08 ../
-rw-r--r--  1 root root 1294 Apr  4 19:32 kang.crt
-rw-------  1 root root 1704 Apr  4 19:32 kang.key

#方法1:
#在线修改证书配置,需要提前先将新证书文件用base64编码并删除换行符
[root@master1 nginx-certs]#cat kang.crt | base64 | tr -d '\n'
[root@master1 nginx-certs]#cat kang.key | base64 | tr -d '\n'
#上面生成的内容替换下面命令的内容,立即生效
[root@master1 ~]#kubectl edit secrets secret-nginx-ssl

#方法2：
#删除旧证书配置
[root@master1 nginx-certs]#kubectl delete secrets secret-nginx-ssl 
secret "secret-nginx-ssl" deleted

#创建新证书配置
[root@master1 nginx-certs]#kubectl create secret tls secret-nginx-ssl --cert=./kang.crt --key=./kang.key 
secret/secret-nginx-ssl created

#可以直接
kubectl delete secrets secret-nginx-ssl;kubectl create secret tls secret-nginx-ssl --cert=./kang.crt --key=./kang.key 

#集群外验证证书是否生效
#nginx不支持热加载，需要手动加载
[root@master1 nginx-certs]#kubectl exec pod-nginx-ssl -- nginx -s reload

#查看已经更新了100年
[root@master1 nginx-certs]#curl -H"host: www.kang.org" 10.244.1.206 -Lkv
*  subject: C=CN; ST=Beijing; L=Beijing; O=SRE; CN=www.kang.org
*  start date: Apr  4 11:32:58 2025 GMT
*  expire date: Mar 11 11:32:58 2125 GMT
*  issuer: C=CN; ST=Beijing; L=Beijing; O=SRE; CN=www.kang.org
```

### Docker-registry 案例

有时Docker的仓库是私有的,如果想下载镜像,就需要在下载镜像的主机上登录仓库并认证后才能实现

但是在Kubernetes集群使用私有仓库镜像就比较繁琐,因为运行Pod 的宿主机并不固定,如果在每个worker节点登录验证显然是不方便的

```bash
#默认走https，添加以下几行
[root@master1 ~]#cat /etc/docker/daemon.json 
{
"registry-mirrors": ["https://docker.m.daocloud.io", "https://docker.1panel.live"],
"insecure-registries":["harbor.kang.org"],
 "exec-opts": ["native.cgroupdriver=systemd"] 
}
```

```bash
#修改名字
[root@master1 ~]#docker tag registry.cn-beijing.aliyuncs.com/wangxiaochun/nginx:1.20.0 harbor.kang.org/example/nginx:1.20.0
[root@master1 ~]#docker push harbor.kang.org/example/nginx:1.20.0 
The push refers to repository [harbor.kang.org/example/nginx]
272bc57d3405: Preparing 
f7141923aaa3: Preparing 
9b63e6289fbe: Preparing 
a2f4f809e04e: Preparing 
1839f9962bd8: Preparing 
02c055ef67f5: Waiting 
unauthorized: unauthorized to access repository: example/nginx, action: push: unauthorized to access repository: example/nginx, action: push
[root@master1 ~]#docker login harbor.kang.org
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

#上传镜像
[root@master1 ~]#docker push harbor.kang.org/example/nginx:1.20.0 
The push refers to repository [harbor.kang.org/example/nginx]
272bc57d3405: Pushed 
f7141923aaa3: Pushed 
9b63e6289fbe: Pushed 
a2f4f809e04e: Pushed 
1839f9962bd8: Pushed 
02c055ef67f5: Pushed 
1.20.0: digest: sha256:598057a5c482d2fb42092fd6f4ba35ea4cc86c41f5db8bb68d1ab92c4c40db98 size: 1570
```

#### 通过命令创建

```bash
kubectl create secret docker-registry KEYNAME --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=EMAIL
```

#### 通过docker认证文件创建

```bash
#先登录并认证到目标仓库server上，认证凭据自动保存在~/.docker/config.json文件中
docker login --username=DOCKER_USER DOCKER_REGISTRY_SERVER

#基于config.json文件生成secret
kubectl create secret docker-registry KEYNAME --from-file=.dockerconfigjson=path/to/.docker/config.json

kubectl create secret generic KEYNAME --type='kubernetes.io/dockerconfigjson' --from-file=.dockerconfigjson=/root/.docker/config.json

#示例：
kubectl create secret docker-registry my-secret --from-file=.dockerconfigjson=path/to/.docker/config.json

kubectl create secret generic dockerharbor-auth --type='kubernetes.io/dockerconfigjson' --from-file=.dockerconfigjson=/root/.docker/config.json
```

```json
[root@master1 ~]#cat .docker/config.json 
{
	"auths": {
		"harbor.kang.org": {
			"auth": "YWRtaW46MTIzNDU2"
		}
	}
}
```



#### 通过命令创建私有仓库登录认证信息

```bash
#修改所有集群节点的Docker配置支持私有仓库
[root@master1 ~]#cat /etc/docker/daemon.json 
{
"registry-mirrors": ["https://docker.m.daocloud.io", "https://docker.1panel.live"],
"insecure-registries":["harbor.kang.org"],
 "exec-opts": ["native.cgroupdriver=systemd"] 
}

#上传镜像
[root@master1 ~]#docker push harbor.kang.org/example/nginx:1.20.0 
The push refers to repository [harbor.kang.org/example/nginx]
272bc57d3405: Pushed 
f7141923aaa3: Pushed 
9b63e6289fbe: Pushed 
a2f4f809e04e: Pushed 
1839f9962bd8: Pushed 
02c055ef67f5: Pushed 
1.20.0: digest: sha256:598057a5c482d2fb42092fd6f4ba35ea4cc86c41f5db8bb68d1ab92c4c40db98 size: 1570
```

```bash
#方法1：命令行创建Secret yaml
[root@master1 ~]#kubectl create secret docker-registry harbor-docker-registry-secret --docker-server=harbor.kang.org --docker-username=admin --docker-password=123456 --dry-run=client -o yaml > yaml/secret/harbor-docker-registry-secret.yaml
[root@master1 ~]#cat yaml/secret/harbor-docker-registry-secret.yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJoYXJib3Iua2FuZy5vcmciOnsidXNlcm5hbWUiOiJhZG1pbiIsInBhc3N3b3JkIjoiMTIzNDU2IiwiYXV0aCI6IllXUnRhVzQ2TVRJek5EVTIifX19
kind: Secret
metadata:
  creationTimestamp: null
  name: harbor-docker-registry-secret
type: kubernetes.io/dockerconfigjson

[root@master1 ~]#kubectl apply -f yaml/secret/harbor-docker-registry-secret.yaml
secret/harbor-docker-registry-secret created
```

```bash
#方法2：登录后引用登录文件创建
[root@master1 ~]#kubectl create secret docker-registry harbor-docker-registry-secret --from-file=.dockerconfigjson=/root/.docker/config.json
```

```bash
[root@master1 ~]#kubectl get secrets 
NAME                            TYPE                             DATA   AGE
harbor-docker-registry-secret   kubernetes.io/dockerconfigjson   1      100s
secret-nginx-ssl                kubernetes.io/tls                2      18h
```

```yaml
#准备清单文件

# 资源类型：Deployment（用于管理Pod副本）
apiVersion: apps/v1
kind: Deployment
metadata:
  # Deployment的名称
  name: deployment-harbor-docker-registry-secret
spec:
  # 副本数量，表示期望运行的Pod数量
  replicas: 3
  selector:
    matchLabels:
      # 选择标签，匹配Pod
      app: pod-test-harbor-docker-registry-secret
  template:
    metadata:
      labels:
        # 给Pod打标签，方便管理和Service识别
        app: pod-test-harbor-docker-registry-secret
    spec:
      # 定义容器信息
      containers:
      - name: pod-test-harbor-docker-registry-secret  # 容器名称
        image: harbor.kang.org/example/nginx:1.20.0   # 镜像地址（来自Harbor私有仓库）
        ports:
        - containerPort: 80                          # 容器对外暴露端口
      # 镜像拉取时使用的Harbor认证Secret
      imagePullSecrets:
      - name: harbor-docker-registry-secret          # 指定docker-registry类型的Secret名称
```

```bash
[root@master1 yaml]#vim secret/storage-secret-docker-registry-harbor.yaml
[root@master1 yaml]#kubectl apply -f secret/storage-secret-docker-registry-harbor.yaml
deployment.apps/deployment-harbor-docker-registry-secret created
[root@master1 yaml]#kubectl get pod
.NAME                                                       READY   STATUS    RESTARTS      AGE
deployment-harbor-docker-registry-secret-554c75c4d-6xv94   1/1     Running   0             5s
deployment-harbor-docker-registry-secret-554c75c4d-8q6nf   1/1     Running   0             5s
deployment-harbor-docker-registry-secret-554c75c4d-fjftv   1/1     Running   0             5s
pod-nginx-ssl                                              1/1     Running   1 (16h ago)   18h
[root@master1 yaml]#kubectl get deployments.apps 
NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment-harbor-docker-registry-secret   3/3     3            3           10s
```

 



### downwardAPI

downwardAPI 不是一种独立的API资源类型，只是一种引用Pod自身的运行环境信息

![image-20250405135133574](kubernetes/image-20250405135133574.png)

![image-20250405135145620](kubernetes/image-20250405135145620.png)

```
https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#downwardapi
```



#### 可通过 `fieldRef` 获得的信息

对于某些 Pod 级别的字段，你可以将它们**作为环境变量或使用** `downwardAPI` 卷提供给容器。 通过这两种机制可用的字段有：

- `metadata.name`

  Pod 的名称

- `metadata.namespace`

  Pod 的[命名空间](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/namespaces/)

- `metadata.uid`

  Pod 的唯一 ID

- `metadata.annotations['<KEY>']`

  Pod 的[注解](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/annotations/) `<KEY>` 的值（例如：`metadata.annotations['myannotation']`）

- `metadata.labels['<KEY>']`

  Pod 的[标签](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/) `<KEY>` 的值（例如：`metadata.labels['mylabel']`）

以下信息可以**通过环境变量**获得，但**不能作为 `downwardAPI` 卷 `fieldRef`** 获得：

- `spec.serviceAccountName`

  Pod 的[服务账号](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-service-account/)名称

- `spec.nodeName`

  Pod 运行时所处的[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)名称

- `status.hostIP`

  Pod 所在节点的主 IP 地址

- `status.hostIPs`

  这组 IP 地址是 `status.hostIP` 的双协议栈版本，第一个 IP 始终与 `status.hostIP` 相同。

- `status.podIP`

  Pod 的主 IP 地址（通常是其 IPv4 地址）

- `status.podIPs`

  这组 IP 地址是 `status.podIP` 的双协议栈版本, 第一个 IP 始终与 `status.podIP` 相同。

以下信息可以通过 `downwardAPI` 卷 `fieldRef` 获得，但**不能作为环境变量**获得：

- `metadata.labels`

  Pod 的所有标签，格式为 `标签键名="转义后的标签值"`，每行一个标签

- `metadata.annotations`

  Pod 的全部注解，格式为 `注解键名="转义后的注解值"`，每行一个注解

#### 可通过 `resourceFieldRef` 获得的信息

- `resource: limits.cpu`

  容器的 CPU 限制值

- `resource: requests.cpu`

  容器的 CPU 请求值

- `resource: limits.memory`

  容器的内存限制值

- `resource: requests.memory`

  容器的内存请求值

- `resource: limits.hugepages-*`

  容器的巨页限制值

- `resource: requests.hugepages-*`

  容器的巨页请求值

- `resource: limits.ephemeral-storage`

  容器的临时存储的限制值

- `resource: requests.ephemeral-storage`

  容器的临时存储的请求值

#### 资源限制的后备信息

如果没有为容器指定 CPU 和内存限制时尝试使用 Downward API 暴露该信息，那么 kubelet 默认会根据 [节点可分配资源](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable) 计算并暴露 CPU 和内存的最大可分配值。

- 
- 环境变量：用于单个变量，可以将 Pod 信息和容器信息直接注入容器内部
- Volume挂载：将 Pod 信息生成为文件，再挂载到容器内部中

类似于ConfigMap或Secret资源，容器能够在环境变量中valueFrom字段中基于两个字段来引用其所属Pod对象的元数据信息

- fieldRef：引用常规性的元数据
- resourceFieldRef：引用和资源限制和资源请求相关的元数据

容器规范中可在环境变量配置中的valueFrom通过内嵌字段fieldRef引用的信息包括如下信息：

```bash
metadata.name：Pod对象的名称
metadata.namespace：Pod对象隶属的名称空间
metadata.uid：Pod对象的UID
metadata.labels['<KEY>']：Pod对象标签中的指定键的值，例如metadata.labels['mylabel']
metadata.annotations['<KEY>']：Pod对象注解信息中的指定键的值
```

容器上的计算资源需求和资源限制相关的信息，以及临时存储资源需求和资源限制相关的信息可通过容器规范中的resourceFieldRef字段引用，相关字段包括requests.cpu、limits.cpu、requests.memory和limits.memory等

```powershell
status.podIP：Pod对象的IP地址
spec.serviceAccountName：Pod对象使用的ServiceAccount资源名称
spec.nodeName：节点名称
status.hostIP：节点IP地址
```

容器上的当前容器的资源请求及资源限额的定义，可以通过resourceFieldRef字段引用，因此它们包括6项

```
requests.cpu
requests.memory
requests.ephemeral-storage
limits.cpu
limits.memory
limits.ephemeral-storage
```



#### downwardAPI 案例

##### 范例: 获取基本的变量信息通过变量方式引用

```yaml
# 资源类型：Pod
apiVersion: v1
kind: Pod
metadata:
  # Pod名称
  name: downwardapi-env-test
  labels:
    # 自定义标签，方便管理和引用
    app: downwardapi-env
spec:
  containers:
  - name: downwardapi-env-test  # 容器名称
    image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1  # 容器镜像
    resources:
      # 资源请求，保证最低资源
      requests:
        memory: "32Mi"
        cpu: "250m"
      # 资源限制，防止资源超标
      limits:
        memory: "64Mi"
        cpu: "500m"
    env:
    # ↓↓↓ 使用 Downward API 获取 Pod 元数据 ↓↓↓
    - name: THIS_POD_NAME  # 获取Pod名称
      valueFrom:
        fieldRef:
          fieldPath: metadata.name

    - name: THIS_POD_NAMESPACE  # 获取Pod所在命名空间
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace

    - name: THIS_APP_LABEL  # 获取Pod的标签app的值
      valueFrom:
        fieldRef:
          fieldPath: metadata.labels['app']

    # ↓↓↓ 获取Pod的资源信息 ↓↓↓
    - name: THIS_CPU_LIMIT  # 获取容器cpu限制值
      valueFrom:
        resourceFieldRef:
          resource: limits.cpu

    - name: THIS_MEM_REQUEST  # 获取容器memory请求值，单位Mi
      valueFrom:
        resourceFieldRef:
          resource: requests.memory
          divisor: 1Mi  # 转换单位为Mi（默认是字节）

    # ↓↓↓ 普通变量引用其他环境变量（注意这个是在容器内部 shell 中才能解析）↓↓↓
    - name: VAR_REF
      value: $(THIS_POD_NAMESPACE).kang.org  # 容器内 shell 或代码中可以识别

```

```bash
[root@master1 yaml]#vim secret/storage-downwardapi-env-test.yaml
[root@master1 yaml]#kubectl apply -f secret/storage-downwardapi-env-test.yaml
pod/downwardapi-env-test created
[root@master1 yaml]#kubectl get pod
NAME                                                       READY   STATUS    RESTARTS      AGE
downwardapi-env-test                                       1/1     Running   0             8s

[root@master1 yaml]#kubectl exec downwardapi-env-test  -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=downwardapi-env-test
THIS_POD_NAME=downwardapi-env-test
THIS_POD_NAMESPACE=default
THIS_APP_LABEL=downwardapi-env
THIS_CPU_LIMIT=1
THIS_MEM_REQUEST=32
VAR_REF=default.kang.org
```

##### 范例: 存储卷方式使用

```yaml
# 资源类型：Pod
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-volume-test  # Pod名称
  labels:
    zone: Beijing               # 区域标签
    rack: zhongguancun          # 机架标签
    app: redis-master           # 应用标签
  annotations:
    region: Asia-China          # 区域注解
spec:
  # 定义Volume，使用downwardAPI挂载Pod相关信息
  volumes:
  - name:  
    downwardAPI:
      defaultMode: 0420         # 文件权限（十进制表示为272，相当于rw-r-----
      items:
      # 挂载Namespace信息
      - fieldRef:
          fieldPath: metadata.namespace
        path: "pod_namespace"

      # 挂载Labels信息（所有标签）
      - fieldRef:
          fieldPath: metadata.labels
        path: "pod_labels"

      # 挂载Annotations信息（所有注解）
      - fieldRef:
          fieldPath: metadata.annotations
        path: "pod_annotations"

      # 挂载容器的CPU limits限制
      - resourceFieldRef:
          containerName: downwardapi-volume-test
          resource: limits.cpu
        path: "cpu_limit"

      # 挂载容器的Memory requests请求值，单位Mi
      - resourceFieldRef:
          containerName: downwardapi-volume-test
          resource: requests.memory
          divisor: "1Mi"
        path: "mem_request"

  containers:
  - name: downwardapi-volume-test    # 容器名称
    image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1  # 镜像
    resources:
      requests:
        memory: "32Mi"  # 容器最小内存请求
        cpu: "250m"     # 容器最小CPU请求
      limits:
        memory: "64Mi"  # 容器最大内存限制
        cpu: "500m"     # 容器最大CPU限制
    volumeMounts:
    - name: podinfo
      mountPath: /etc/podinfo/  # 将 Volume 挂载到容器内路径
      readOnly: false           # 只读或可写（这里是可写）
#属性解析；defaultMode 表示挂载文件后，文件的权限限制,items 定制自己的一些名称
```

```bash
[root@master1 yaml]#vim secret/storage-downwardapi-volume-test.yaml
[root@master1 yaml]#kubectl apply -f secret/storage-downwardapi-volume-test.yaml
pod/downwardapi-volume-test created
[root@master1 yaml]#kubectl exec -it downwardapi-volume-test -- sh
[root@downwardapi-volume-test /]# ls /etc/podinfo/
cpu_limit        mem_request      pod_annotations  pod_labels       pod_namespace
[root@downwardapi-volume-test /]# cat /etc/podinfo/pod_labels 
app="redis-master"
rack="zhongguancun"
zone="Beijing"
[root@downwardapi-volume-test /]# cat /etc/podinfo/cpu_limit 
1
```

### Projected

将多个卷挂载到容器的同一个目录内

![image-20250405145634574](kubernetes/image-20250405145634574.png)

Projected Volume仅支持对如下四种类型的卷（数据源）进行投射操作

- Secret：投射Secret对象
- ConfigMap：投射ConfgMap对象
- DownwardAPI：投射Pod元数据
- ServiceAccountToken：投射ServiceAccountToken

#### 范例：默认每一个Pod都有Projected卷

```bash
[root@master1 yaml]#kubectl run myapp --image registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1
pod/myapp created
[root@master1 yaml]#kubectl get pod myapp -o yaml

apiVersion: v1
kind: Pod
metadata:
...
spec:
...
  volumes:
  - name: kube-api-access-j7hsv
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
...
status:
...
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-j7hsv
      readOnly: true
      recursiveReadOnly: Disabled
...
```

```bash
[root@master1 yaml]#kubectl exec -it myapp -- sh
[root@myapp /]# ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt     namespace  token
```

范例：Example configuration with a secret, a downwardAPI, and a configMap

```yaml
# 定义 Secret，保存敏感数据（用户名）
apiVersion: v1
kind: Secret
metadata:
  name: mysecret       # Secret名称
  namespace: default   # 命名空间
type: Opaque           # 自定义类型
data:
  username: d2FuZ3hpYW9jaHVu   # base64编码后的用户名 wangxiaochun

---

# 定义 ConfigMap，保存普通配置信息
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap    # ConfigMap名称
  namespace: default
data:
  myconfig: Hello,Myconfig   # 配置信息内容

---

# 定义 Pod，使用 Projected Volume 将 Secret、ConfigMap、DownwardAPI 整合到一个Volume里
apiVersion: v1
kind: Pod
metadata:
  name: projected-volume-demo
spec:
  containers:
  - name: container-test    # 容器名称
    image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1  # 镜像
    volumeMounts:
    - name: all-in-one              # 挂载Volume名称
      mountPath: "/projected-volume"  # 容器内路径
      readOnly: true                # 只读挂载

  volumes:
  - name: all-in-one
    projected:                    # 定义投射（整合）多个资源
      sources:
      # 第一部分：来自Secret
      - secret:
          name: mysecret
          items:
          - key: username
            path: my-group/my-username   # 映射到 my-group/my-username 文件

      # 第二部分：来自DownwardAPI（Pod自身信息）
      - downwardAPI:
          items:
          - path: "labels"              # 文件名 labels
            fieldRef:
              fieldPath: metadata.labels # Pod的labels信息
          - path: "cpu_limit"           # 文件名 cpu_limit
            resourceFieldRef:
              containerName: container-test
              resource: limits.cpu       # 容器的CPU limits

      # 第三部分：来自ConfigMap
      - configMap:
          name: myconfigmap
          items:
          - key: myconfig
            path: my-group/my-config     # 映射到 my-group/my-config 文件
```

```bash
[root@master1 yaml]#vim secret/storage-projected-demo.yaml
[root@master1 yaml]#kubectl apply -f secret/storage-projected-demo.yaml
secret/mysecret created
configmap/myconfigmap created
pod/projected-volume-demo created
[root@master1 yaml]#kubectl exec -it projected-volume-demo -- sh
[root@projected-volume-demo /]# ls -l /projected-volume/
total 0
lrwxrwxrwx    1 root     root            16 Apr  5 07:04 cpu_limit -> ..data/cpu_limit
lrwxrwxrwx    1 root     root            13 Apr  5 07:04 labels -> ..data/labels
lrwxrwxrwx    1 root     root            15 Apr  5 07:04 my-group -> ..data/my-group
[root@projected-volume-demo /]# ls -l /projected-volume/my-group/
total 8
-rw-r--r--    1 root     root            14 Apr  5 07:04 my-config
-rw-r--r--    1 root     root            12 Apr  5 07:04 my-username
```



# 综合案例: 使用持久卷和CM等部署WordPress 和 MySQL

![image-20250405153222032](kubernetes/image-20250405153222032.png)

在kubernetes部署 wordpress，要满足以下要求：

基本需求

- 独立部署两个wordpress Pod实例，它们使用NFS StorageClass 存储卷存储用户上传的图片或文件等数据；以ConfigMap和Secret提供必要的配置
- 部署一个MySQL数据库；以ConfigMap和Secret提供必要的配置
- service 暴露wordpress 的服务

扩展需求

- 部署一个独立的ingress或者nginx Pod实例，为wordpress提供反向代理
- 同时提供https和http虚拟主机，其中发往http的请求都重定向给https；以ConfigMap和Secret提供必要的配置

- 动态的蓝绿发布和滚动发布
  - 对于wordpress 来说，没有本质的区别
  - nginx的更新，依赖configmap和secret的内容

## MySQL的相关资源

```bash
#提前准备名为sc-nfs的storage Class 和 Metalib 的 LoadBalance
[root@master1 wordpress]#kubectl get sc
NAME               PROVISIONER                                   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
sc-nfs (default)   k8s-sigs.io/nfs-subdir-external-provisioner   Delete          Immediate           false                  2d23h

[root@ubuntu2204 ~]#exportfs -v
/data/sc-nfs  	<world>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)

[root@master1 wordpress]#kubectl get all -n metallb-system 
NAME                             READY   STATUS    RESTARTS        AGE
pod/controller-bb5f47665-xkvgt   1/1     Running   7 (18h ago)     5d4h
pod/speaker-7smt9                1/1     Running   14 (148m ago)   5d4h
pod/speaker-dsvw5                1/1     Running   13 (148m ago)   5d4h
pod/speaker-rvxvf                1/1     Running   14 (148m ago)   5d4h
pod/speaker-wzsv2                1/1     Running   14 (148m ago)   5d4h

NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/metallb-webhook-service   ClusterIP   10.98.57.161   <none>        443/TCP   5d4h

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   4         4         4       4            4           kubernetes.io/os=linux   5d4h

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           5d4h

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-bb5f47665   1         1         1       5d4h
```

```yaml
# 创建一个独立的wordpress命名空间
apiVersion: v1
kind: Namespace
metadata:
  name: wordpress
---
# 创建Secret，存放MySQL的密码，类型使用basic-auth（也可用Opaque）
apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
  namespace: wordpress
type: kubernetes.io/basic-auth
data:
  password: MTIzNDU2   # base64编码的123456
---
# 创建MySQL的Service，ClusterIP类型为None，做成Headless服务，方便Pod直接通信
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  namespace: wordpress
  labels:
    app: wordpress
spec:
  ports:
  - port: 3306  # 对外暴露端口
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None   # Headless Service（无集群IP）
---
# 创建PVC，用于持久化MySQL的数据，防止Pod重建数据丢失
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: wordpress
  labels:
    app: wordpress
spec:
  storageClassName: sc-nfs  # 指定StorageClass名称
  accessModes:
  - ReadWriteOnce  # 允许单节点读写
  resources:
    requests:
      storage: 20Gi  # 申请20G存储空间
---
# 创建MySQL Deployment，部署MySQL容器
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  namespace: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate  # 重建模式，适用于数据库这类Stateful应用
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - name: mysql
        image: registry.cn-beijing.aliyuncs.com/wangxiaochun/mysql:8.0.29-oracle  # 自定义镜像
        env:
        # 设置环境变量，MySQL账号密码配置
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306  # MySQL对外端口
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql  # MySQL数据挂载路径
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim  # 挂载前面申请的PVC

```

```bash
[root@master1 wordpress]#vim storage-wordpress-mysql.yaml
[root@master1 wordpress]#kubectl apply -f storage-wordpress-mysql.yaml
namespace/wordpress created
secret/mysql-pass created
service/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/wordpress-mysql created
```

## wordpress 相关资源

```yaml
# WordPress 对外暴露的 Service，类型为 LoadBalancer
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wordpress
  labels:
    app: wordpress
spec:
  ports:
  - port: 80  # 暴露80端口，外部访问WordPress
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer  # 云厂商可用，内部测试可改为 ClusterIP
  # type: ClusterIP # 如果使用Ingress，可以用这个
---
# WordPress 的持久化存储 PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  namespace: wordpress
  labels:
    app: wordpress
spec:
  storageClassName: sc-nfs  # 使用前面配置好的存储类
  accessModes:
  - ReadWriteOnce  # 单节点读写
  resources:
    requests:
      storage: 20Gi  # 申请20Gi存储空间
---
# 部署WordPress的 Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wordpress
  labels:
    app: wordpress
spec:
  replicas: 2  # 启动2个副本Pod，实现高可用
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate  # 更新方式：重建
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - name: wordpress
        image: registry.cn-beijing.aliyuncs.com/wangxiaochun/wordpress:php8.2-apache  # WordPress镜像
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql  # 数据库服务名
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password  # 从Secret中取密码
        - name: WORDPRESS_DB_USER
          value: wordpress  # 数据库用户名
        ports:
        - containerPort: 80  # WordPress对外端口
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html  # WordPress程序挂载目录
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim  # 挂载PVC

```

```bash
[root@master1 wordpress]#vim storage-wordpress-wordpress.yaml 
[root@master1 wordpress]#kubectl apply -f storage-wordpress-wordpress.yaml
service/wordpress created
persistentvolumeclaim/wp-pv-claim created
deployment.apps/wordpress created
```

## 创建 Ingress

```yaml
# 定义 Ingress 资源，实现域名访问
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-test  # Ingress资源名称
  namespace: wordpress  # 所属命名空间
  # 注解部分
  # 老版本(<1.19)使用这个指定Ingress控制器
  # annotations:
  #   kubernetes.io/ingress.class: "nginx"
spec:
  # 新版推荐的写法，指定Ingress控制器类型
  ingressClassName: nginx  # 对应部署好的nginx ingress controller

  # 定义域名规则
  rules:
  - host: www.kang.org  # 外部访问域名
    http:
      paths:
      - path: /         # 匹配所有路径
        pathType: Prefix  # 前缀匹配
        backend:
          service:        # 指定转发到的后端服务
            name: wordpress  # wordpress服务名称
            port:
              number: 80  # wordpress服务暴露的端口
```

```bash
[root@master1 wordpress]#vim ingress-wordpress.yaml 
[root@master1 wordpress]#kubectl apply -f ingress-wordpress.yaml 
ingress.networking.k8s.io/ingress-test configured

[root@master1 wordpress]#kubectl get ingress -n wordpress 
NAME           CLASS   HOSTS          ADDRESS     PORTS   AGE
ingress-test   nginx   www.kang.org   10.0.0.10   80      11s
```

## 测试访问

```bash
[root@master1 wordpress]#kubectl get all -n wordpress 
NAME                                   READY   STATUS    RESTARTS   AGE
pod/wordpress-7674c49955-9hmjp         1/1     Running   0          3m40s
pod/wordpress-7674c49955-j9bj9         1/1     Running   0          3m40s
pod/wordpress-mysql-7fdbb5fb4c-zbb86   1/1     Running   0          7m21s

NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/wordpress         LoadBalancer   10.98.247.111   10.0.0.10     80:32062/TCP   3m40s
service/wordpress-mysql   ClusterIP      None            <none>        3306/TCP       7m21s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress         2/2     2            2           3m40s
deployment.apps/wordpress-mysql   1/1     1            1           7m21s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-7674c49955         2         2         2       3m40s
replicaset.apps/wordpress-mysql-7fdbb5fb4c   1         1         1       7m21s

[root@master1 wordpress]#kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-0433605b-5b72-4b5d-9b90-ed78879ffff9   20Gi       RWO            Delete           Bound    wordpress/mysql-pv-claim   sc-nfs         <unset>                          13m
pvc-7ac288b9-753e-4a90-9ca4-4dc130348c91   20Gi       RWO            Delete           Bound    wordpress/wp-pv-claim      sc-nfs         <unset>                          9m59s
[root@master1 wordpress]#kubectl get pvc -n wordpress 
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mysql-pv-claim   Bound    pvc-0433605b-5b72-4b5d-9b90-ed78879ffff9   20Gi       RWO            sc-nfs         <unset>                 13m
wp-pv-claim      Bound    pvc-7ac288b9-753e-4a90-9ca4-4dc130348c91   20Gi       RWO            sc-nfs         <unset>                 10m

[root@ubuntu2204 ~]#ls /data/sc-nfs
wordpress-mysql-pv-claim-pvc-0433605b-5b72-4b5d-9b90-ed78879ffff9  wordpress-wp-pv-claim-pvc-7ac288b9-753e-4a90-9ca4-4dc130348c91

```



