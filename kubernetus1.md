# kubernetus

## StatefulSet&CAS

```powershell
容器化应用编排系统：
	声明式API + Controller
	工作负载型控制器:
		Deployment:
			部署，扩容、缩容，发布（滚动更新，回滚），故障处理（故障自愈） 
		Daemonset:系统级应用（实例和节点一一对应）
		Job：作业编排系统，手动触发、一次性任务。（未来的某个时间点执行一次性任务）
		CronJob：作业编排系统，定时触发、周期性任务
		StatefulSet：用于编排有状态应用的基础构架
			operator：
				operatorHub.io
				Operator 本质上是一个运行在 K8S 集群里的控制器（Controller），它用来自动管理特定类型的资源（CR，自定义资源），把以前人工执行的运维动作变成程序自动执行。
					引入自定义的资源类型
						CRD：CRD（自定义资源定义）= 自定义一种新的 Kubernetes 资源类型。 
	HPA:
		指标系统：
			Metrics Server (只可以抓取核心指标，cpu、内存)
			promethues

CAS
	iscsi协议:基于tcp的scsi存储协议
    OpenEBS是CAS存储机制的著名实现之一
	控制平面：管理存储
	数据平面：执行存储
	OpenEBS:
		OpenEBS = 基于 Kubernetes 的容器存储方案（CSP），用 CRD + Operator 来管理存储资源。
		核心定位：
			Kubernetes Native Storage
			动态/静态存储卷管理
			软件定义存储（SDS）
			3.x版本多种存储引擎（LocalPV / cStor / Jiva / Mayastor）
			4.x版本只支持 （LocalPV / Mayastor / LVM）
				LocalPV：单节点单实例，没有跨节点的冗余能力
				Jiva：可以实现跨节点冗余
			本地卷 Local Volumes：节点级卷，仅支持在卷所在的节点本地访问，因此，pod也必须调度至卷所在节点才能使用本地卷
				OpenEBS可基于本地块设备或分区、子目录或者LVM、ZFS、甚至是由文件模拟的设备来创建PV，这些统称为本地卷
				pod和卷必须在同一个节点上,先创建卷再创建pod，根据pod的调度结果临时来置备卷
			复制卷 Replicated Volumes：支持将数据同步复制到多个节点的卷，因而实现节点容错，以及跨可用区进行数据复制 
				OpenEBS可基于Mayastor、cStor 或 Jiva三种引擎之一，为每个分布式复制卷创建一个微服务、Pod通过iSCSI（cStor和Jiva）或NVMeoF（Mayastor）连接至卷
			
			复制卷和本地卷，都只允许单路读写，复制卷采用iSCSI协议，pod认为就是一个块设备
			IsCSt：磁盘级的存储协议，块设备
			
			OpenEBS的hostpath卷支持PV动态置备机制 和 kubernetus 内置的 hostpath 不一样
	注意：默认只支持单路读写 RwadWriteOnce 可以部署 nfs-provisioner 实现多路读写
```

 ![image-20250409105429920](kubernetes/image-20250409105429920.png)

##  OpenEBS

#### 数据引擎（Data Engines）

作用：负责底层存储的具体读写操作，相当于直接和存储设备打交道的部分。

特点：

- 是 OpenEBS 的核心组件。
- Pod 访问存储（PVC）时，实际操作的就是数据引擎。
- 不同的数据引擎支持不同的存储方式。

常见的数据引擎类型：

| 引擎类型      | 说明                                      |
| ------------- | ----------------------------------------- |
| Mayastor      | 高性能、适合 NVMe 存储                    |
| cStor         | 经典引擎，支持副本、快照                  |
| Jiva          | 轻量级引擎                                |
| Local Volumes | 本地存储，支持 LVM、ZFS、hostpath、device |

------

#### 控制平面（Control Plane）

作用：负责管理、编排、监控 OpenEBS 存储的生命周期。

特点：

- 配置和管理存储资源（SC、PVC、PV）
- 部署和管理数据引擎
- 集成 Prometheus 进行监控与告警
- 插件机制（Velero Plugin）做备份与恢复

控制平面包含：

- CSI Drivers（K8S 通用存储接口）
- Storage Operators（存储控制器）
- Prometheus Exporters（监控数据输出）

------

整体架构逻辑

```
Stateful Workloads（有状态应用，例如 MySQL、PostgreSQL、Kafka 等）
        ↓
Kubernetes Storage Control Plane（存储控制器：SC/PVC/PV/CSI）
        ↓
OpenEBS Control Plane（控制平面管理和调度存储资源）
        ↓
OpenEBS Data Engines（数据引擎执行读写操作）
```

------

其他关键点

- Enterprise Framework / Tools：可接入外部工具（Prometheus、Grafana、Velero、EFK）
- Any Platform, Any Storage：OpenEBS 适配各种平台（云、本地、虚拟化、物理机）以及多种存储介质（SSD、HDD、NVMe）

------

#### 总结一句话

> OpenEBS 把存储划分为两大块 —— *控制平面* 负责管理，*数据引擎* 负责读写，配合 Kubernetes 的 SC/PVC/PV，实现了云原生环境下的存储管理。



#### 部署

```powershell
部署OpenEBS的基本流程
1 在各个节点部署iSCSI client（部署复制卷的cSor和Jiva时需要部署）
2 在Kubernetus上部署OpenEBS
3 选择使用的数据引擎
4 为选择的数据引擎准备StorageClas

注意：在应用级本身有数据冗余机制的话，未必分要有冗余机制
	例如：mysql主从复制，一主二从，已经有三份数据，没必要在做副本复制卷，冗余机制
	redis Cluster，如果没有对redis cluster的pod做主从复制，那么需要做多副本，冗余机制
    
   数据卷，不能取代数据备份
```

```
https://openebs.io/docs/3.10.x/user-guides/installation
```

要继续默认安装模式，请使用以下命令安装 OpenEBS。OpenEBS 安装在 `openebs` 命名空间中

```
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
```

上述命令安装了 Jiva 和 Local PV 组件。要安装并启用其他引擎，您需要运行以下命令：

- cStor

  ```
  kubectl apply -f https://openebs.github.io/charts/cstor-operator.yaml
  ```

- Local PV ZFS

  ```
  kubectl apply -f https://openebs.github.io/charts/zfs-operator.yaml
  ```

- Local PV LVM

  ```
  kubectl apply -f https://openebs.github.io/charts/lvm-operator.yaml
  ```

Installing Jiva[#](https://openebs.io/docs/3.10.x/user-guides/jiva/jiva-install#installing-jiva)

- 要安装最新的 Jiva 版本，请执行：

  ```
  kubectl apply -f https://openebs.github.io/charts/jiva-operator.yaml
  ```

- 接下来，验证 Jiva 运算符和 CSI Pod 是否在集群上运行。要获取 Pod 的状态，请执行以下命令：

  ```
  kubectl get pod -n openebs
  
  
  Sample Output:
  NAME                                           READY   STATUS    RESTARTS   AGE
  jiva-operator-7765cbfffd-vt787                 1/1     Running   0          10s       
  openebs-localpv-provisioner-57b44f4664-klsrw   1/1     Running   0          118s       
  openebs-jiva-csi-controller-0                  4/4     Running   0          6m14s     
  openebs-jiva-csi-node-56t5g                    2/2     Running   0          6m13s     
  openebs-jiva-csi-node-xtyhu                    2/2     Running   0          6m20s     
  openebs-jiva-csi-node-h2unk                    2/2     Running   0          6m20s
  ```

  ```
  https://github.com/iKubernetes/learning-k8s/tree/master/OpenEBS
  ```



#### 范例：localPV部署（本地卷）

```
https://openebs.io/docs/3.10.x/user-guides/installation
```

```bash
[root@master1 tools]#kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
namespace/openebs created
serviceaccount/openebs-maya-operator created
clusterrole.rbac.authorization.k8s.io/openebs-maya-operator created
clusterrolebinding.rbac.authorization.k8s.io/openebs-maya-operator created
customresourcedefinition.apiextensions.k8s.io/blockdevices.openebs.io created
customresourcedefinition.apiextensions.k8s.io/blockdeviceclaims.openebs.io created
configmap/openebs-ndm-config created
daemonset.apps/openebs-ndm created
deployment.apps/openebs-ndm-operator created
deployment.apps/openebs-ndm-cluster-exporter created
service/openebs-ndm-cluster-exporter-service created
daemonset.apps/openebs-ndm-node-exporter created
service/openebs-ndm-node-exporter-service created
deployment.apps/openebs-localpv-provisioner created
storageclass.storage.k8s.io/openebs-hostpath created
storageclass.storage.k8s.io/openebs-device created

[root@master1 tools]#kubectl get namespaces 
NAME              STATUS   AGE
default           Active   13m
kube-flannel      Active   9m52s
kube-node-lease   Active   13m
kube-public       Active   13m
kube-system       Active   13m
openebs           Active   10s
[root@master1 tools]#kubectl get po -n openebs 
NAME                                           READY   STATUS    RESTARTS   AGE
openebs-localpv-provisioner-65dd55b8dc-4gt97   1/1     Running   0          100s
openebs-ndm-24c8d                              1/1     Running   0          101s
openebs-ndm-4dtsl                              1/1     Running   0          101s
openebs-ndm-8hzhd                              1/1     Running   0          101s
openebs-ndm-cluster-exporter-848db89c6-tmd6n   1/1     Running   0          100s
openebs-ndm-node-exporter-gvjpz                1/1     Running   0          100s
openebs-ndm-node-exporter-kqfms                1/1     Running   0          100s
openebs-ndm-node-exporter-x4gdt                1/1     Running   0          100s
openebs-ndm-operator-5849bb84b8-jnh77          1/1     Running   0          100s

[root@master1 tools]#kubectl get sc
NAME               PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device     openebs.io/local   Delete          WaitForFirstConsumer   false                  114s
openebs-hostpath   openebs.io/local   Delete          WaitForFirstConsumer   false                  114s
[root@master1 tools]#kubectl get sc openebs-hostpath -o yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    cas.openebs.io/config: "#hostpath type will create a PV by \n# creating a sub-directory
      under the\n# BASEPATH provided below.\n- name: StorageType\n  value: \"hostpath\"\n#Specify
      the location (directory) where\n# where PV(volume) data will be saved. \n# A
      sub-directory with pv-name will be \n# created. When the volume is deleted,
      \n# the PV sub-directory will be deleted.\n#Default value is /var/openebs/local\n-
      name: BasePath\n  value: \"/var/openebs/local/\"\n"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"cas.openebs.io/config":"#hostpath type will create a PV by \n# creating a sub-directory under the\n# BASEPATH provided below.\n- name: StorageType\n  value: \"hostpath\"\n#Specify the location (directory) where\n# where PV(volume) data will be saved. \n# A sub-directory with pv-name will be \n# created. When the volume is deleted, \n# the PV sub-directory will be deleted.\n#Default value is /var/openebs/local\n- name: BasePath\n  value: \"/var/openebs/local/\"\n","openebs.io/cas-type":"local"},"name":"openebs-hostpath"},"provisioner":"openebs.io/local","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
    openebs.io/cas-type: local
  creationTimestamp: "2025-04-09T05:00:30Z"
  name: openebs-hostpath
  resourceVersion: "1929"
  uid: 39a84534-a3b8-4d4d-b368-7b8981214c31
provisioner: openebs.io/local
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer

#默认在/var/openebs/local 
#修改时修改
	name: BasePath\n  value: \"/var/openebs/local/\"\
```

```yaml
[root@master1 local-pv-hostpath]#cat openebs-local-hostpath-pvc.yaml 
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: openebs-local-hostpath-pvc
spec:
  storageClassName: openebs-hostpath
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G
```

```bash
[root@master1 local-pv-hostpath]#kubectl apply -f openebs-local-hostpath-pvc.yaml
persistentvolumeclaim/openebs-local-hostpath-pvc created
[root@master1 local-pv-hostpath]#kubectl get pvc
NAME                         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS       VOLUMEATTRIBUTESCLASS   AGE
openebs-local-hostpath-pvc   Pending                                      openebs-hostpath   <unset>                 13s
#现在没有消费者调度，只有在首个消费者调度后才会被创建
```

```yaml
[root@master1 local-pv-hostpath]#cat redis-with-openebs-local-hostpath.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: redis-with-openebs-local-hostpath
spec:
  containers:
  - name: redis
    image: redis:7-alpine
    ports:
    - containerPort: 6379
      name: redis
    volumeMounts:
    - mountPath: /data
      name: local-storage
  volumes:
  - name: local-storage
    persistentVolumeClaim:
      claimName: openebs-local-hostpath-pvc
```

```bash
#创建消费者
[root@master1 local-pv-hostpath]#kubectl apply -f redis-with-openebs-local-hostpath.yaml 
pod/redis-with-openebs-local-hostpath created
[root@master1 local-pv-hostpath]#kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
redis-with-openebs-local-hostpath   1/1     Running   0          75s
[root@master1 local-pv-hostpath]#kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       VOLUMEATTRIBUTESCLASS   AGE
openebs-local-hostpath-pvc   Bound    pvc-3eb95ba2-d37b-4bb0-9248-fe4e260717a1   5G         RWO            openebs-hostpath   <unset>                 2m59s
[root@master1 local-pv-hostpath]#kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                STORAGECLASS       VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-3eb95ba2-d37b-4bb0-9248-fe4e260717a1   5G         RWO            Delete           Bound    default/openebs-local-hostpath-pvc   openebs-hostpath   <unset>                          5s
```

```bash
#在宿主机位置
[root@master1 local-pv-hostpath]#kubectl get pod -o wide 
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
redis-with-openebs-local-hostpath   1/1     Running   0          92s   10.244.1.5   node1.kang.com   <none>           <none>
[root@node1 ~]#ls /var/openebs/local
pvc-3eb95ba2-d37b-4bb0-9248-fe4e260717a1
```

```bash
#查看
[root@master1 local-pv-hostpath]#kubectl exec -it redis-with-openebs-local-hostpath -- sh
/data # redis-cli 
127.0.0.1:6379> sed name kang
(error) ERR unknown command 'sed', with args beginning with: 'name' 'kang' 
127.0.0.1:6379> set  name kang
OK
127.0.0.1:6379> GET name
"kang"
127.0.0.1:6379> BGSAVE
Background saving started
127.0.0.1:6379> exit
/data # ls
dump.rdb

[root@node1 ~]#ls /var/openebs/local/pvc-3eb95ba2-d37b-4bb0-9248-fe4e260717a1/
dump.rdb
```

#### 范例：jiva部署（复制卷）

```
https://openebs.io/docs/3.10.x/user-guides/jiva/jiva-install
```

```bash
#所有节点安装iscsi
apt install open-iscsi
```

```bash
[root@master1 local-pv-hostpath]#kubectl apply -f https://openebs.github.io/charts/jiva-operator.yaml
namespace/openebs unchanged
customresourcedefinition.apiextensions.k8s.io/jivavolumepolicies.openebs.io created
customresourcedefinition.apiextensions.k8s.io/jivavolumes.openebs.io created
customresourcedefinition.apiextensions.k8s.io/upgradetasks.openebs.io created
serviceaccount/jiva-operator created
clusterrole.rbac.authorization.k8s.io/jiva-operator created
clusterrolebinding.rbac.authorization.k8s.io/jiva-operator created
deployment.apps/jiva-operator created
csidriver.storage.k8s.io/jiva.csi.openebs.io created
serviceaccount/openebs-jiva-csi-controller-sa created
priorityclass.scheduling.k8s.io/openebs-jiva-csi-controller-critical created
priorityclass.scheduling.k8s.io/openebs-jiva-csi-node-critical created
clusterrole.rbac.authorization.k8s.io/openebs-jiva-csi-role created
clusterrolebinding.rbac.authorization.k8s.io/openebs-jiva-csi-binding created
statefulset.apps/openebs-jiva-csi-controller created
clusterrole.rbac.authorization.k8s.io/openebs-jiva-csi-attacher-role created
clusterrolebinding.rbac.authorization.k8s.io/openebs-jiva-csi-attacher-binding created
serviceaccount/openebs-jiva-csi-node-sa created
clusterrole.rbac.authorization.k8s.io/openebs-jiva-csi-registrar-role created
clusterrolebinding.rbac.authorization.k8s.io/openebs-jiva-csi-registrar-binding created
configmap/openebs-jiva-csi-iscsiadm created
daemonset.apps/openebs-jiva-csi-node created

[root@master1 local-pv-hostpath]#kubectl get pod -n openebs  -o wide 
NAME                                           READY   STATUS    RESTARTS   AGE     IP           NODE             NOMINATED NODE   READINESS GATES
jiva-operator-5488577f47-mj8dc                 1/1     Running   0          4m26s   10.244.1.6   node1.kang.com   <none>           <none>
openebs-jiva-csi-controller-0                  5/5     Running   0          4m25s   10.244.3.5   node3.kang.com   <none>           <none>
openebs-jiva-csi-node-ggkv7                    3/3     Running   0          4m25s   10.0.0.12    node2.kang.com   <none>           <none>
openebs-jiva-csi-node-p826g                    3/3     Running   0          4m25s   10.0.0.11    node1.kang.com   <none>           <none>
openebs-jiva-csi-node-rxgvk                    3/3     Running   0          4m25s   10.0.0.13    node3.kang.com   <none>           <none>
openebs-localpv-provisioner-65dd55b8dc-4gt97   1/1     Running   0          39m     10.244.3.4   node3.kang.com   <none>           <none>
openebs-ndm-24c8d                              1/1     Running   0          39m     10.0.0.13    node3.kang.com   <none>           <none>
openebs-ndm-4dtsl                              1/1     Running   0          39m     10.0.0.12    node2.kang.com   <none>           <none>
openebs-ndm-8hzhd                              1/1     Running   0          39m     10.0.0.11    node1.kang.com   <none>           <none>
openebs-ndm-cluster-exporter-848db89c6-tmd6n   1/1     Running   0          39m     10.244.1.2   node1.kang.com   <none>           <none>
openebs-ndm-node-exporter-gvjpz                1/1     Running   0          39m     10.244.1.3   node1.kang.com   <none>           <none>
openebs-ndm-node-exporter-kqfms                1/1     Running   0          39m     10.244.2.6   node2.kang.com   <none>           <none>
openebs-ndm-node-exporter-x4gdt                1/1     Running   0          39m     10.244.3.3   node3.kang.com   <none>           <none>
openebs-ndm-operator-5849bb84b8-jnh77          1/1     Running   0          39m     10.244.3.2   node3.kang.com   <none>           <none>
```

```yaml
[root@master1 jiva-csi]#cat openebs-jivavolumepolicy-demo.yaml 
apiVersion: openebs.io/v1alpha1
kind: JivaVolumePolicy		#自定义资源类型
metadata:
  name: jivavolumepolicy-demo 
  namespace: openebs
spec:
  replicaSC: openebs-hostpath	#后端使用本地卷的卷存储类
  target:
    # This sets the number of replicas for high-availability
    # replication factor <= no. of (CSI) nodes
    replicationFactor: 2
    # disableMonitor: false
    # auxResources:
    # tolerations:
    # resources:
    # affinity:
    # nodeSelector:
    # priorityClassName:
  # replica:
    # tolerations:
    # resources:
    # affinity:
    # nodeSelector:
    # priorityClassName:
```

```bash
[root@master1 jiva-csi]#kubectl apply -f openebs-jivavolumepolicy-demo.yaml
jivavolumepolicy.openebs.io/jivavolumepolicy-demo created

[root@master1 jiva-csi]#kubectl get jivavolumepolicies.openebs.io -n openebs 
NAME                    AGE
jivavolumepolicy-demo   62s
```

```yaml
#创建基于JivaVolumePolicy策略的存储类
[root@master1 jiva-csi]#cat openebs-jiva-csi-storageclass.yaml 
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: openebs-jiva-csi
provisioner: jiva.csi.openebs.io
allowVolumeExpansion: true
parameters:
  cas-type: "jiva"
  policy: "jivavolumepolicy-demo"	#引用上面的策略
```

```bash
[root@master1 jiva-csi]#kubectl apply -f openebs-jiva-csi-storageclass.yaml
storageclass.storage.k8s.io/openebs-jiva-csi created
#多了一个 jiva 的 storageclasses 
[root@master1 jiva-csi]#kubectl get sc
NAME               PROVISIONER           RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device     openebs.io/local      Delete          WaitForFirstConsumer   false                  51m
openebs-hostpath   openebs.io/local      Delete          WaitForFirstConsumer   false                  51m
openebs-jiva-csi   jiva.csi.openebs.io   Delete          Immediate              true                   77s
```

```yaml
#创建pvc实例
[root@master1 jiva-csi]#cat openebs-jiva-csi-pvc.yaml 
kind: PersistentVolumeClaim      # 资源类型：PVC
apiVersion: v1                   # API版本
metadata:
  name: openebs-jiva-csi-pvc    # PVC名称
spec:
  storageClassName: openebs-jiva-csi  # 指定使用的StorageClass（存储类）
  accessModes:                      # 存储访问模式
    - ReadWriteOnce                 # 只能被单个节点挂载读写（RWO）
  resources:
    requests:
      storage: 5Gi                  # 申请5Gi的存储空间
```

```bash
[root@master1 jiva-csi]#kubectl apply -f openebs-jiva-csi-pvc.yaml
persistentvolumeclaim/openebs-jiva-csi-pvc created

[root@master1 ji va-csi]#kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       VOLUMEATTRIBUTESCLASS   AGE
openebs-jiva-csi-pvc         Bound    pvc-dacc038d-752a-43df-921d-59d1186c712f   5Gi        RWO            openebs-jiva-csi   <unset>                 11s
openebs-local-hostpath-pvc   Bound    pvc-3eb95ba2-d37b-4bb0-9248-fe4e260717a1   5G         RWO            openebs-hostpath   <unset>                 42m

[root@master1 ~]#kubectl get pod -n openebs 
NAME                                                              READY   STATUS    RESTARTS   AGE
jiva-operator-5488577f47-bgl57                                    1/1     Running   0          6m41s
openebs-jiva-csi-controller-0                                     5/5     Running   0          6m41s
openebs-jiva-csi-node-4t6cw                                       3/3     Running   0          6m41s
openebs-jiva-csi-node-dqlk2                                       3/3     Running   0          6m41s
openebs-jiva-csi-node-gg7rv                                       3/3     Running   0          6m41s
openebs-localpv-provisioner-65dd55b8dc-sv8wl                      1/1     Running   0          7m49s
openebs-ndm-2sr7t                                                 1/1     Running   0          7m50s
openebs-ndm-cluster-exporter-848db89c6-rmj2v                      1/1     Running   0          7m49s
openebs-ndm-flnkn                                                 1/1     Running   0          7m50s
openebs-ndm-nk6mk                                                 1/1     Running   0          7m50s
openebs-ndm-node-exporter-g8724                                   1/1     Running   0          7m49s
openebs-ndm-node-exporter-xbdtx                                   1/1     Running   0          7m49s
openebs-ndm-node-exporter-xjpb5                                   1/1     Running   0          7m49s
openebs-ndm-operator-5849bb84b8-v49rc                             1/1     Running   0          7m49s
pvc-e91300c1-7c47-41ac-80be-8255b5ff2e1c-jiva-ctrl-695c7965mvbg   2/2     Running   0          3m20s
pvc-e91300c1-7c47-41ac-80be-8255b5ff2e1c-jiva-rep-0               1/1     Running   0          3m20s
pvc-e91300c1-7c47-41ac-80be-8255b5ff2e1c-jiva-rep-1               1/1     Running   0          3m20s
```

```yaml
#定义redis，消费pvc
[root@master1 jiva-csi]#cat redis-with-openebs-jiva-pvc.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: redis-with-openebs-jiva-pvc
spec:
  containers:
  - name: redis
    image: redis:7-alpine
    ports:
    - containerPort: 6379
      name: redis
    volumeMounts:
    - mountPath: /data
      name: local-storage
  volumes:
  - name: local-storage
    persistentVolumeClaim:
      claimName: openebs-jiva-csi-pvc
```

```bash
[root@master1 local-pv-hostpath]#kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       VOLUMEATTRIBUTESCLASS   AGE
openebs-jiva-csi-pvc         Bound    pvc-e91300c1-7c47-41ac-80be-8255b5ff2e1c   5Gi        RWO            openebs-jiva-csi   <unset>                 24m
openebs-local-hostpath-pvc   Bound    pvc-194fc196-575b-4f2c-8d00-c7ac1e41b70d   5G         RWO            openebs-hostpath   <unset>                 8m15s
[root@master1 local-pv-hostpath]#kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                                 STORAGECLASS       VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-0555b5f2-5a70-4815-8d0e-fa5c97eb78b5   5Gi        RWO            Delete           Bound    openebs/openebs-pvc-e91300c1-7c47-41ac-80be-8255b5ff2e1c-jiva-rep-1   openebs-hostpath   <unset>                          25m
pvc-194fc196-575b-4f2c-8d00-c7ac1e41b70d   5G         RWO            Delete           Bound    default/openebs-local-hostpath-pvc                                    openebs-hostpath   <unset>                          9m21s
pvc-abe20d13-c1bf-4417-8282-112e2b7213ca   5Gi        RWO            Delete           Bound    openebs/openebs-pvc-e91300c1-7c47-41ac-80be-8255b5ff2e1c-jiva-rep-0   openebs-hostpath   <unset>                           25m 
pvc-e91300c1-7c47-41ac-80be-8255b5ff2e1c   5Gi        RWO            Delete           Bound    default/openebs-jiva-csi-pvc                                          openebs-jiva-csi   <unset>                          25m
[root@master1 local-pv-hostpath]#
```



#### 范例:nfs-provisioner，多路读写

不需要部署nfs-server，只需要部署一些OpenEBS组件就可以实现多路读写

在安装 nfs-provisioner 之前，请确保您的 Kubernetes 集群满足以下先决条件：

1. Kubernetes 版本 1.18

2. 本指南假设您有一个可用的后端存储类，可以是默认存储类或任何其他存储类（例如：https://openebs.io/docs/user-guides/localpv-hostpath）。

3. NFS 客户端安装在所有将运行挂载`openebs-rwx`卷的 Pod 的节点上。以下是如何在一些常见的操作系统上准备 NFS 客户端：

   ```
   apt install nfs-common -y
   ```

```
https://github.com/openebs-archive/dynamic-nfs-provisioner/blob/develop/docs/intro.md
```

```bash
[root@master1 local-pv-hostpath]#kubectl apply -f https://openebs.github.io/charts/nfs-operator.yaml
namespace/openebs unchanged
serviceaccount/openebs-maya-operator unchanged
clusterrole.rbac.authorization.k8s.io/openebs-maya-operator configured
clusterrolebinding.rbac.authorization.k8s.io/openebs-maya-operator unchanged
deployment.apps/openebs-nfs-provisioner created
storageclass.storage.k8s.io/openebs-rwx created

#自动创建了一个存储类
[root@master1 local-pv-hostpath]#kubectl get sc
NAME               PROVISIONER           RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-rwx        openebs.io/nfsrwx     Delete          Immediate              false                  118s
```

```yaml
#创建nfs-pvc
[root@master1 nfs-pv]#cat openebs-nfs-pvc.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: "openebs-rwx"
  resources:
    requests:
      storage: 1Gi
```

```bash
[root@master1 nfs-pv]#kubectl apply -f openebs-nfs-pvc.yaml
persistentvolumeclaim/nfs-pvc created
[root@master1 nfs-pv]#kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       VOLUMEATTRIBUTESCLASS   AGE
nfs-pvc                      Bound    pvc-85db1034-fc45-44f1-8870-1e81fc99544b   1Gi        RWX            openebs-rwx        <unset>                 57s

#想象为自己就是一个消费者，自己就可以 bound
```



##  statefulSet

```powershell
StatefulSet
	各副本都有一个唯一名称标识，依赖于一个专用的 Headless Service 实现
	各pod副本存储的状态数据并不相同，因而需要专用且稳定的Volume
		基于podTemplate定义Pod模板
		在podTemplate上使用volumeTemplate为各个副本动态置备PersistentVolume
	基于Pod管理策略（Pod Management Policy），定义创建删除及扩缩容等管理操作期间，施加在Pod副本上的操作方式
		OrderedReady：创建或扩容时前一个实例运行起来且就绪后才创建后一个实例，删除或缩容时，逆序，依次完成相关Pod副本的终止
		Parallel：各Pod副本的创建或删除操作操作不存在顺序方面的要求，可同时进行
		
	Pod副本的专用名称标识
		每个StatefulSet对象强依赖于专用的Headless Service对象
		StatefulSet中各Pod副本分别拥有唯一的名称标识
			前缀格式：“$(statefulset_name)-$(ordinal)”
			后缀格式：“$(service_name).$(namesplace).cluster.local”
		各pod的名称表示可有ClustrDNS直接解析为Pod IP
	卷请求模板（volumeClaimTemplate）
		在闯将Pod副本时绑定至专有PVC
		PVC的名称遵循特定的格式：$(卷模板名称)-$(statefulset_name)-$(ordinal)   .eg：data-redis-0
		从而能够与StatefulSet控制器对象的Pod副本建立关联关系
		支持从静态置备或动态置备的PV中完成绑定
		删除Pod（例如缩容），并不会一并伤处相关PVC
	
	存在问题：
	   有状态、分布式应用在启动、扩容、缩容等运维操作上的步骤存在差异，甚至完全不同，因而StatefulSet只能提供一个基础的编排框架
	   有状态应用所需要的管理操作，需要由用户自行编写代码完成
	   
	更新策略
		rollingUpdate：滚动更新，自动触发
			配置策略
				maxUnavailable：定义单批次允许更新的最大副本数量
				partition <integer>：用于定义更新分区的编号，其序号大于等于该编号的Pod都将被更新，小于该分区号的都不予跟新；默认编号为0，即全部更新 
		onDelete：删除时更新，手动触发
		
	分布式、有状态应用：
		自身具有分布式协同能力：ElasticSearch, ...
			结合StatefulSet编排时，用户几乎不需要自行编写代码
		自身不具有分布式协同能力，需要用户帮助其完成协同功能：MySQL Replication Cluster，Redis Replication Cluster,...
			结合StatefulSet编排时，用户需要自行编写相关的运维代码
```



#### 模板

```yaml
apiVersion: apps/v1    # API 版本（apps 组，v1 版本）
kind: DaemonSet        # 资源类型：DaemonSet（守护进程集）

metadata:              # 元数据
  name: <string>       # DaemonSet 名称（集群内唯一）
  namespace: <string>  # 命名空间

spec:                  # 具体定义内容
  minReadySeconds: <integer>       # Pod 准备就绪的最少秒数，任一容器无 crash 即视为就绪
  serviceName: <string>            # 关联的 Headless Service 名称，需事先存在
  selector:                        # 标签选择器（必须与 template.metadata.labels 匹配）
    matchLabels:                   # 选择的标签
      key: value

  replicas: <integer>              # Pod 副本数量（DaemonSet 一般不写）

  template:                        # Pod 模板定义
    metadata:
      labels:                      # 标签，需与 selector 保持一致
        key: value
    spec:                          # Pod 的具体定义（镜像、端口、环境等）

  volumeClaimTemplates:            # PVC 模板（用于动态申请卷）
  - metadata:
      name: <string>
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

  podManagementPolicy: <string>    # Pod 管理策略
                                   # 可选值：
                                   # OrderedReady（按顺序，前面的 Pod ready 才启动后面的）
                                   # Parallel（并行启动，默认）

  revisionHistoryLimit: <integer>  # 保留历史版本数量，默认 10

  updateStrategy:                  # 滚动更新策略
    type: <string>                 # 更新类型：
                                   # OnDelete（手动删除后更新）
                                   # RollingUpdate（滚动更新）

    rollingUpdate:                 # type 为 RollingUpdate 时才生效
      maxUnavailable: <string>     # 更新时最大不可用 Pod 数量（绝对值或百分比）
      partition: <integer>         # 保留不更新的 Pod 数量（默认 0）
```



#### 范例：演示StatefulSet

```yaml
# demodb，一个教育用 Kubernetes 原生 NoSQL 数据库。
# 特点：分布式 Key-Value 存储，支持永久读写。
# 环境变量：
#   DEMODB_DATADIR 存储路径
#   DEMODB_HOST    服务主机
#   DEMODB_PORT    服务端口
# 默认端口：
#   9907/tcp 客户端访问端口
#   9999/tcp 集群成员通信端口
# 维护者：MageEdu <mage@magedu.com>

---

# 创建 Service，给 StatefulSet 提供稳定的 DNS 解析和集群内部通信
apiVersion: v1
kind: Service
metadata:
  name: demodb          # Service 名称
  namespace: default    # 所在命名空间
  labels:               # 标签（方便管理）
    app: demodb
spec:
  clusterIP: None       # Headless Service（无 ClusterIP）
                        # Pod 会通过域名直接解析 Pod IP
  ports:
  - port: 9907          # 映射端口
  selector:             # 关联 Selector，匹配 Pod
    app: demodb

---

# StatefulSet 资源，适用于有状态服务（如数据库）
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: demodb          # StatefulSet 名称
  namespace: default
spec:
  selector:             # Pod 选择规则
    matchLabels:
      app: demodb
  serviceName: "demodb" # 必须是上面 Service 的 name
  replicas: 2           # 副本数量，2 个 Pod
                        # Pod 名称为 demodb-0、demodb-1
  template:             # Pod 模板
    metadata:
      labels:
        app: demodb
    spec:
      containers:
      - name: demodb-shard    # 容器名称
        image: ikubernetes/demodb:v0.1   # 使用的镜像
        ports:
        - containerPort: 9907  # 容器端口
          name: db             # 端口名称（供探针等引用）

        env:                   # 设置环境变量
        - name: DEMODB_DATADIR
          value: "/demodb/data"

        livenessProbe:         # 存活检查，检测进程是否活着
          initialDelaySeconds: 2  # 容器启动后延迟 2 秒开始检查
          periodSeconds: 10       # 每 10 秒检查一次
          httpGet:                # 通过 HTTP 检查
            path: /status         # 检查路径
            port: db              # 检查端口

        readinessProbe:        # 就绪检查，是否对外提供服务
          initialDelaySeconds: 15  # 延迟 15 秒开始检查
          periodSeconds: 30        # 每 30 秒检查一次
          httpGet:
            path: /status?level=full
            port: db

        volumeMounts:         # 挂载数据卷
        - name: data          # 卷名称
          mountPath: /demodb/data  # 挂载到容器内的路径

  volumeClaimTemplates:       # 动态 PVC 模板（每个 Pod 独立的 PVC）
  - metadata:
      name: data              # PVC 名称（对应 volumeMounts 的 name）
    spec:
      accessModes: [ "ReadWriteOnce" ]  # 访问模式
      storageClassName: "openebs-hostpath"  # 存储类名称
      resources:
        requests:
          storage: 2Gi        # 每个 PVC 请求 2Gi 存储空间
```

```bash
[root@master1 statefulsets]#kubectl apply -f demodb.yaml 
service/demodb created
statefulset.apps/demodb created

[root@master1 statefulsets]#kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
demodb       ClusterIP   None         <none>        9907/TCP   10s
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP    3h45m

[root@master1 ~]#kubectl get pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       VOLUMEATTRIBUTESCLASS   AGE
data-demodb-0                Bound    pvc-8cf076ae-842f-42ad-bf00-b3fd21eac0f4   2Gi        RWO            openebs-hostpath   <unset>                 109s
data-demodb-1                Bound    pvc-22ddbba0-331a-4215-81f1-2e7a68ea08ce   2Gi        RWO            openebs-hostpath   <unset>                 40s
[root@master1 ~]#kubectl get pod -o wide 
NAME                                READY   STATUS               RESTARTS   AGE     IP            NODE             NOMINATED NODE   READINESS GATES
demodb-0                            1/1     Running              0          23m     10.244.1.22   node1.kang.com   <none>           <none>
demodb-1                            1/1     Running              0          22m     10.244.2.27   node2.kang.com   <none>           <none>
```

```bash
[root@master1 statefulsets]#kubectl run client-$RANDOM --restart=Never --rm -it --image=ikubernetes/admin-box:v1.2 --command -- bash
If you don't see a command prompt, try pressing enter.
root@client-28519 ~# host -t A demodb-0.demodb.default.svc.cluster.local
demodb-0.demodb.default.svc.cluster.local has address 10.244.1.22
root@client-28519 ~# host -t A demodb-1.demodb.default.svc.cluster.local
demodb-1.demodb.default.svc.cluster.local has address 10.244.2.27

root@client-28519 ~# host -t PTR 27.2.244.10.in-addr.arpa
27.2.244.10.in-addr.arpa domain name pointer demodb-1.demodb.default.svc.cluster.local.
root@client-28519 ~# host -t PTR 22.1.244.10.in-addr.arpa
22.1.244.10.in-addr.arpa domain name pointer demodb-0.demodb.default.svc.cluster.local.
```



#### 范例：部署 mysql

```yaml
[root@master1 mysql]#ls
01-configmap-mysql.yaml  02-services-mysql.yaml  03-statefulset-mysql.yaml  README.md
[root@master1 mysql]#cat 01-configmap-mysql.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
data:
  primary.cnf: |
    # Apply this config only on the primary.
    [mysql]
    default-character-set=utf8mb4
    [mysqld]
    log-bin
    character-set-server=utf8mb4
    [client]
    default-character-set=utf8mb4

  replica.cnf: |
    # Apply this config only on replicas.
    [mysql]
    default-character-set=utf8mb4
    [mysqld]
    super-read-only    
    character-set-server=utf8mb4
    [client]
    default-character-set=utf8mb4
[root@master1 mysql]#cat 02-services-mysql.yaml 
# Headless service for stable DNS entries of StatefulSet members.
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - name: mysql
    port: 3306
  clusterIP: None
  selector:
    app: mysql
---
# Client service for connecting to any MySQL instance for reads.
# For writes, you must instead connect to the primary: mysql-0.mysql.
apiVersion: v1
kind: Service
metadata:
  name: mysql-read
  labels:
    app: mysql
spec:
  ports:
  - name: mysql
    port: 3306
  selector:
    app: mysql
[root@master1 mysql]#cat 03-statefulset-mysql.yaml 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql
  replicas: 3
  template:
    metadata:
      labels:
        app: mysql
    spec:
      initContainers:
      - name: init-mysql
        image: mysql:5.7
        command:
        - bash
        - "-c"
        - |
          set -ex
          # Generate mysql server-id from pod ordinal index.
          [[ $(cat /proc/sys/kernel/hostname) =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          echo [mysqld] > /mnt/conf.d/server-id.cnf
          # Add an offset to avoid reserved server-id=0 value.
          echo server-id=$((100 + $ordinal)) >> /mnt/conf.d/server-id.cnf
          # Copy appropriate conf.d files from config-map to emptyDir.
          if [[ $ordinal -eq 0 ]]; then
            cp /mnt/config-map/primary.cnf /mnt/conf.d/
          else
            cp /mnt/config-map/replica.cnf /mnt/conf.d/
          fi
        volumeMounts:
        - name: conf
          mountPath: /mnt/conf.d
        - name: config-map
          mountPath: /mnt/config-map
      - name: clone-mysql
        image: ikubernetes/xtrabackup:1.0
        command:
        - bash
        - "-c"
        - |
          set -ex
          # Skip the clone if data already exists.
          [[ -d /var/lib/mysql/mysql ]] && exit 0
          # Skip the clone on primary (ordinal index 0).
          [[ $(cat /proc/sys/kernel/hostname) =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          [[ $ordinal -eq 0 ]] && exit 0
          # Clone data from previous peer.
          ncat --recv-only mysql-$(($ordinal-1)).mysql 3307 | xbstream -x -C /var/lib/mysql
          # Prepare the backup.
          xtrabackup --prepare --target-dir=/var/lib/mysql
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: LANG
          value: "C.UTF-8"
        - name: MYSQL_ALLOW_EMPTY_PASSWORD
          value: "1"
        ports:
        - name: mysql
          containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
        livenessProbe:
          exec:
            command: ["mysqladmin", "ping"]
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          exec:
            # Check we can execute queries over TCP (skip-networking is off).
            command: ["mysql", "-h", "127.0.0.1", "-e", "SELECT 1"]
          initialDelaySeconds: 5
          periodSeconds: 2
          timeoutSeconds: 1
      - name: xtrabackup
        image: ikubernetes/xtrabackup:1.0
        ports:
        - name: xtrabackup
          containerPort: 3307
        command:
        - bash
        - "-c"
        - |
          set -ex
          cd /var/lib/mysql

          # Determine binlog position of cloned data, if any.
          if [[ -f xtrabackup_slave_info && "x$(<xtrabackup_slave_info)" != "x" ]]; then
            # XtraBackup already generated a partial "CHANGE MASTER TO" query
            # because we're cloning from an existing replica. (Need to remove the tailing semicolon!)
            cat xtrabackup_slave_info | sed -E 's/;$//g' > change_master_to.sql.in
            # Ignore xtrabackup_binlog_info in this case (it's useless).
            rm -f xtrabackup_slave_info xtrabackup_binlog_info
          elif [[ -f xtrabackup_binlog_info ]]; then
            # We're cloning directly from primary. Parse binlog position.
            [[ `cat xtrabackup_binlog_info` =~ ^(.*?)[[:space:]]+(.*?)$ ]] || exit 1
            rm -f xtrabackup_binlog_info xtrabackup_slave_info
            echo "CHANGE MASTER TO MASTER_LOG_FILE='${BASH_REMATCH[1]}',\
                  MASTER_LOG_POS=${BASH_REMATCH[2]}" > change_master_to.sql.in
          fi

          # Check if we need to complete a clone by starting replication.
          if [[ -f change_master_to.sql.in ]]; then
            echo "Waiting for mysqld to be ready (accepting connections)"
            until mysql -h 127.0.0.1 -e "SELECT 1"; do sleep 1; done

            echo "Initializing replication from clone position"
            mysql -h 127.0.0.1 \
                  -e "$(<change_master_to.sql.in), \
                          MASTER_HOST='mysql-0.mysql', \
                          MASTER_USER='root', \
                          MASTER_PASSWORD='', \
                          MASTER_CONNECT_RETRY=10; \
                        START SLAVE;" || exit 1
            # In case of container restart, attempt this at-most-once.
            mv change_master_to.sql.in change_master_to.sql.orig
          fi

          # Start a server to send backups when requested by peers.
          exec ncat --listen --keep-open --send-only --max-conns=1 3307 -c \
            "xtrabackup --backup --slave-info --stream=xbstream --host=127.0.0.1 --user=root"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
      volumes:
      - name: conf
        emptyDir: {}
      - name: config-map
        configMap:
          name: mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "nfs-csi"
      resources:
        requests:
          storage: 10Gi
```

##### 部署MySQL主从复制集群

依赖的环境：支持PV动态置备的StorageClass/nfs-csi;

##### 部署过程

```bash
kubectl create namespace mysql
kubectl apply ./ -n mysql
```

##### 访问入口

读请求：mysql-read.mysql.svc.cluster.local

写请求：mysql-0.mysql.mysql.svc.cluster.local

 

##  Operator



```
https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/operator/
```

```
https://operatorhub.io/
```

```powershell
Operstor
	部署operstor自身：多数可能都是使用内置的Deployment或StatefulSet来编排运行
	体统专用的CRD： 
		CRD：自定义资源类型
```

#### 实例：eck-operator

```
https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html
```

##### 部署elastic

```bash
#部署crd
[root@master1 ~]#kubectl create -f https://download.elastic.co/downloads/eck/2.16.1/crds.yaml

[root@master1 ~]#kubectl get crd
NAME                                                   CREATED AT
agents.agent.k8s.elastic.co                            2025-04-09T11:36:12Z
apmservers.apm.k8s.elastic.co                          2025-04-09T11:36:12Z
beats.beat.k8s.elastic.co                              2025-04-09T11:36:12Z
blockdeviceclaims.openebs.io                           2025-04-09T05:00:29Z
blockdevices.openebs.io                                2025-04-09T05:00:29Z
elasticmapsservers.maps.k8s.elastic.co                 2025-04-09T11:36:12Z
elasticsearchautoscalers.autoscaling.k8s.elastic.co    2025-04-09T11:36:12Z
elasticsearches.elasticsearch.k8s.elastic.co           2025-04-09T11:36:13Z
enterprisesearches.enterprisesearch.k8s.elastic.co     2025-04-09T11:36:13Z
jivavolumepolicies.openebs.io                          2025-04-09T05:35:22Z
jivavolumes.openebs.io                                 2025-04-09T05:35:23Z
kibanas.kibana.k8s.elastic.co                          2025-04-09T11:36:13Z
logstashes.logstash.k8s.elastic.co                     2025-04-09T11:36:13Z
stackconfigpolicies.stackconfigpolicy.k8s.elastic.co   2025-04-09T11:36:14Z
upgradetasks.openebs.io                                2025-04-09T05:35:23Z

#部署operator
[root@master1 ~]#kubectl apply -f https://download.elastic.co/downloads/eck/2.16.1/operator.yaml

[root@master1 ~]#kubectl get ns elastic-system 
NAME             STATUS   AGE
elastic-system   Active   3m23s

[root@master1 ~]#kubectl get pod -n elastic-system 
NAME                 READY   STATUS    RESTARTS   AGE
elastic-operator-0   1/1     Running   0          3m
```



```yaml
# 定义 Elasticsearch 资源
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: myes                        # Elasticsearch 集群名称
  namespace: elastic-system        # 运行的命名空间
spec:
  version: 8.14.0                  # 使用的 Elasticsearch 版本

  nodeSets:                        # 定义节点组
  - name: default                  # 节点组名称
    count: 3                       # 节点数量，3个副本（Pod）

    config:                        # Elasticsearch 配置
      node.store.allow_mmap: false # 关闭 mmap，避免内存问题

    volumeClaimTemplates:          # 持久化存储声明（PVC模板）
    - metadata:
        name: elasticsearch-data   # PVC 的名称前缀（自动生成）
      spec:
        accessModes:               # 访问模式
        - ReadWriteOnce            # 单节点读写
        resources:
          requests:
            storage: 5Gi           # 每个Pod申请5Gi存储
        storageClassName: openebs-hostpath # 指定存储类，使用 OpenEBS HostPath 存储

```

```bash
[root@master1 eck-operator]#kubectl apply -f elasticsearch-myes-cluster.yaml

[root@master1 eck-operator]#kubectl get elasticsearch -n elastic-system 
NAME   HEALTH   NODES   VERSION   PHASE   AGE
myes   green    3       8.14.0    Ready   48m

[root@master1 eck-operator]#kubectl get pod -n elastic-system  -o wide 
NAME                 READY   STATUS    RESTARTS        AGE   IP            NODE             NOMINATED NODE   READINESS GATES
elastic-operator-0   1/1     Running   2 (6m46s ago)   20m   10.244.1.35   node1.kang.com   <none>           <none>
myes-es-default-0    0/1     Running   0               11m   10.244.2.37   node2.kang.com   <none>           <none>
myes-es-default-1    0/1     Running   0               11m   10.244.1.40   node1.kang.com   <none>           <none>
myes-es-default-2    0/1     Running   0               11m   10.244.3.17   node3.kang.com   <none>           <none>

#我们还要事先获取到访问ElasticSearch的密码，该密码由部署过程自动生成，并保存在了相关名称空间下的Secrets中，该Secrets对象以集群名称为前缀，以“-es-elastic-user”为后缀。下面的命令将获取到的密码保存在名为PASSWORD的变量中。
[root@master1 eck-operator]#kubectl get secrets -n elastic-system myes-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'
jp3JWp3p55uuMaWa21Q5099y
[root@master1 eck-operator]#PASSWORD=$(kubectl get secrets -n elastic-system myes-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
[root@master1 eck-operator]#echo $PASSWORD
jp3JWp3p55uuMaWa21Q5099y

[root@master1 ~]#kubectl run client-$RANDOM --image ikubernetes/admin-box:v1.2 -it --rm --restart=Never --command -- /bin/bash
If you don't see a command prompt, try pressing enter.
root@client-11378 /# PASSWORD=jp3JWp3p55uuMaWa21Q5099y
root@client-11378 /# echo $PASSWORD
jp3JWp3p55uuMaWa21Q5099y

root@client-11378 /# curl -u "elastic:$PASSWORD" -k https://myes-es-http.elastic-system:9200
{
  "name" : "myes-es-default-0",
  "cluster_name" : "myes",
  "cluster_uuid" : "TUsrslSHQJq7X8h4PV6FTA",
  "version" : {
    "number" : "8.14.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8d96bbe3bf5fed931f3119733895458eab75dca9",
    "build_date" : "2024-06-03T10:05:49.073003402Z",
    "build_snapshot" : false,
    "lucene_version" : "9.10.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}

#查看集群的健康状态
root@client-11378 /# curl -u "elastic:$PASSWORD" -k https://myes-es-http.elastic-system:9200/_cat/health
1744200917 12:15:17 myes green 3 3 0 0 0 0 0 0 - 100.0%

#获取索引
root@client-11378 /# curl -u "elastic:$PASSWORD" -k https://myes-es-http.elastic-system:9200/_cat/indices

#查看节点实例
root@client-11378 /# curl -u "elastic:$PASSWORD" -k https://myes-es-http.elastic-system:9200/_cat/nodes
10.244.1.40 18 71 15 1.13 0.78 1.60 cdfhilmrstw * myes-es-default-1
10.244.3.17  7 70 14 0.39 0.74 1.47 cdfhilmrstw - myes-es-default-2
10.244.2.37 48 71 15 1.61 1.27 1.68 cdfhilmrstw - myes-es-default-0
```

##### **部署Filebeat**

```yaml
# 定义 Filebeat 自定义资源（CRD）
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: filebeat  # Filebeat 资源名称
  namespace: elastic-system  # 所属命名空间
spec:
  type: filebeat  # 指定类型为 filebeat
  version: 8.14.0  # 指定版本
  elasticsearchRef:  # 关联 Elasticsearch
    name: "myes"  # Elasticsearch 名称
  kibanaRef:  # 关联 Kibana
    name: "kibana"  # Kibana 名称
  config:  # Filebeat 配置内容
    filebeat:
      autodiscover:  # 自动发现容器日志
        providers:
        - type: kubernetes  # 从 Kubernetes 自动发现
          node: ${NODE_NAME}  # 获取当前节点名
          hints:  # 自动识别 annotations 提示
            enabled: true
            default_config:  # 默认日志采集配置
              type: container  # 采集容器日志
              paths:
              - /var/log/containers/*${data.kubernetes.container.id}.log
        processors:  # 日志处理器
        - add_kubernetes_metadata:  # 自动添加 k8s 元数据
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"  # 匹配容器日志路径
        - drop_event.when:  # 过滤不需要的 namespace 日志
            or:
            - equals:
                kubernetes.namespace: "kube-system"
            - equals:
                kubernetes.namespace: "logging"
            - equals:
                kubernetes.namespace: "ingress-nginx"
            - equals:
                kubernetes.namespace: "kube-node-lease"
            - equals:
                kubernetes.namespace: "elastic-system"

  daemonSet:  # 以 DaemonSet 方式部署，每个节点一个 Pod
    podTemplate:
      spec:
        serviceAccountName: filebeat  # 绑定的服务账户
        automountServiceAccountToken: true
        terminationGracePeriodSeconds: 30  # 优雅退出时间
        dnsPolicy: ClusterFirstWithHostNet
        hostNetwork: true  # 使用宿主机网络（采集宿主机日志更方便）
        containers:
        - name: filebeat  # 容器名称
          securityContext:
            runAsUser: 0  # 以 root 用户运行
          volumeMounts:  # 挂载宿主机日志路径
          - name: varlogcontainers
            mountPath: /var/log/containers
          - name: varlogpods
            mountPath: /var/log/pods
          - name: varlibdockercontainers
            mountPath: /var/lib/docker/containers
          env:  # 获取节点名环境变量
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
        volumes:  # 定义宿主机日志路径
        - name: varlogcontainers
          hostPath:
            path: /var/log/containers
        - name: varlogpods
          hostPath:
            path: /var/log/pods
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers

---

# 授权相关 RBAC 配置

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole  # 定义集群角色
metadata:
  name: filebeat
rules:
- apiGroups: [""]  # core API 组
  resources:
  - namespaces
  - pods
  - nodes
  verbs:  # 允许的操作
  - get
  - watch
  - list

---

apiVersion: v1
kind: ServiceAccount  # 创建 ServiceAccount
metadata:
  name: filebeat
  namespace: elastic-system

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding  # 绑定 ClusterRole 和 ServiceAccount
metadata:
  name: filebeat
subjects:
- kind: ServiceAccount
  name: filebeat
  namespace: elastic-system
roleRef:
  kind: ClusterRole
  name: filebeat
  apiGroup: rbac.authorization.k8s.io

```

```bash
[root@master1 eck-operator]#kubectl apply -f beats-filebeat.yaml
[root@master1 learning-k8s]#kubectl get pod -n elastic-system 
NAME                           READY   STATUS    RESTARTS      AGE
elastic-operator-0             1/1     Running   2 (97m ago)   111m
filebeat-beat-filebeat-jq5sd   1/1     Running   6 (31m ago)   35m
filebeat-beat-filebeat-k5zmk   1/1     Running   6 (30m ago)   35m
filebeat-beat-filebeat-rs4kl   1/1     Running   6 (30m ago)   35m
kibana-kb-ccd69bdd6-tzjl7      1/1     Running   0             35m
myes-es-default-0              1/1     Running   0             102m
myes-es-default-1              1/1     Running   0             102m
myes-es-default-2              1/1     Running   0             102m
```

##### **部署Kibana**

```yaml
# 定义 Kibana 自定义资源（CRD）
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana   # Kibana 资源名称
  namespace: elastic-system   # 所在命名空间
spec:
  version: 8.14.0   # Kibana 版本
  count: 1          # 副本数，单实例部署
  elasticsearchRef:  # 关联的 Elasticsearch 集群
    name: "myes"     # 关联的 Elasticsearch 名称，需与 Elasticsearch CRD 中 name 保持一致
  http:   # HTTP 服务相关配置
    tls:
      selfSignedCertificate:
        disabled: true  # 禁用自签 TLS 证书，走明文 http
    service:   # Kibana Service 的配置
      spec:
        type: LoadBalancer  # 暴露服务类型为 LoadBalancer，方便外网访问（云环境常用）

---

# 下面是 Ingress 的配置（目前被注释掉了）

# 可以通过 Ingress 暴露 Kibana，适合生产环境走域名 https 访问
# 如果启用，取消注释即可使用

# apiVersion: networking.k8s.io/v1
# kind: Ingress
# metadata:
#   name: kibana  # Ingress 名称
#   namespace: elastic-system
# spec:
#   ingressClassName: nginx  # 使用 nginx ingress controller
#   rules:
#   - host: kibana.magedu.com  # 访问域名
#     http:
#       paths:
#       - backend:
#           service:
#             name: kibana-kb-http  # 自动生成的 Kibana Service 名
#             port:
#               number: 5601        # Kibana 端口
#         path: /                   # 匹配所有路径
#         pathType: Prefix
#
#   # tls:  # 如有证书可开启 https
#   # - hosts:
#   #   - kibana.magedu.com
#   #   secretName: tls-secret-name
```

```
[root@master1 eck-operator]#kubectl apply -f kibana-myes.yaml 
[root@master1 eck-operator]#kubectl get pod -n elastic-system 
NAME                           READY   STATUS    RESTARTS      AGE
elastic-operator-0             1/1     Running   2 (98m ago)   111m
filebeat-beat-filebeat-jq5sd   1/1     Running   6 (31m ago)   35m
filebeat-beat-filebeat-k5zmk   1/1     Running   6 (31m ago)   35m
filebeat-beat-filebeat-rs4kl   1/1     Running   6 (31m ago)   35m
kibana-kb-ccd69bdd6-tzjl7      1/1     Running   0             35m
myes-es-default-0              1/1     Running   0             103m
myes-es-default-1              1/1     Running   0             103m
myes-es-default-2              1/1     Running   0             103m
```

##### **部署MetaLB添加地址池**

```bash
[root@master1 MetalLB]#wget https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml

[root@master1 MetalLB]#ls 
metallb-ipaddresspool.yaml  metallb-l2advertisement.yaml  openelb.yaml  README.md

[root@master1 MetalLB]#cat metallb-ipaddresspool.yaml 
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: localip-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.0.0.100-10.0.0.150
  autoAssign: true
  avoidBuggyIPs: true

[root@master1 MetalLB]#cat metallb-l2advertisement.yaml 
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: localip-pool-l2a
  namespace: metallb-system
spec:
  ipAddressPools:
  - localip-pool 
  interfaces:
  - eth0

[root@master1 MetalLB]#kubectl apply -f .
```

```bash
[root@master1 MetalLB]#kubectl get svc -n elastic-system 
NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
elastic-webhook-server   ClusterIP      10.110.79.132   <none>        443/TCP          123m
kibana-kb-http           LoadBalancer   10.109.7.240    10.0.0.100    5601:30669/TCP   47m
myes-es-default          ClusterIP      None            <none>        9200/TCP         114m
myes-es-http             ClusterIP      10.99.171.95    <none>        9200/TCP         114m
myes-es-internal-http    ClusterIP      10.102.28.206   <none>        9200/TCP         114m
myes-es-transport        ClusterIP      None            <none>        9300/TCP         114m
```



##### **访问测试**

访问 10.0.0.100:5601

账号：elastic

密码：通过`kubectl get secrets -n elastic-system myes-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'`获取



#### 范例：使用minio operator部署minio cluster



#### 范例：使用nacos operator部署nacos cluster





## Helm

### Helm概念

```powershell
Helm
	部署一个应用:
		以MySQL为例：
			configmap，secrets，service，deployment/StatefulSet，PVC，ingress
			
		声明式API：编程任务
		
	Chart：特定包格式的程序包（类似于deb，rpm包）
		要部署的应用的资源配置文件以模板形式定义在Chart中，并内置了一个缺省的值文件为模板字串提供默认值
	Helm：包管理器
	Release：基于一个Chart，可以生成一到多个Release，即部署实例；如果部署在同一名称空间下，这些Release需要具有不同的名字
	
	hub仓库、社区网址（类似于注册中心）
		https://artifacthub.io/
		
	在hub仓库上每个chart有两个版本
		chart version：chart版本
		application version：内部部署程序的版本+
		
		docketHub 也支持放 chart
		
	参数值的设定方法
		1. helm install | upgrade 命令行中重复使用“--set”选项；
		2. 自定义的values.yaml文件（自定义文件中没有定义的参数值，使用默认的值）
	
	获取默认的值文件中各parameters的定义：
		helm show values CHART_NAME
	获取部署某release时使用的自定义parameters：
		helm get values RELEASE_NAME -n NAMESPACE
		eg：	
			查看名称空间下所有的Release
				helm list -n blog
			查看使用命令行定义的values.yaml文件
                helm get values mysql -n blog
     
	查看部署完成后的提示信息:
		helm status RELEASE_NAME -n NAMESPACE
		eg:
			helm status mysql -n blog
	获取部署某Release时，经helm渲染后生成的资源配置清单:
		helm get manifest RELEASE_NAME -n NAMESPACE
		eg：
			helm get manifest mysql -n blog
			
	更新：
		helm upgrade RELEASE_NAME -n NAMESPACE ...
	回滚：
		helm rollback RELEASE_NAME REVISION -n NAMESPACE
			注意：REVISION不填写时默认回滚到上一个版本
		eg:
			helm rollback mysql 1 -n blog
```

**关键概念**

1. Chart
   - 结构：包含Chart.yaml（元数据）、values.yaml（默认配置）、templates/（资源模板）等目录。
   - 子Chart：通过 charts/ 目录或dependencies字段声明依赖的其他Chart。
2. Release
   - 代表Chart 在集群中的具体实例。例如，同一个Chart可多次部署为不同Release（如开发环境dev和生产环境prod）。
3. Repository
   - HTTP 服务器托管Chart包和索引文件（index.yaml）。用户可发布自定义 Chart 到仓库供团队共享。
   - [Artifact Hub](https://artifacthub.io]/)是云原生计算基金会（CNCF）托管的云原生制品中心化仓库，它通过集中索引和分类，帮助用户快速查找、安装和发布各类云原生资源，如 Helm Chart、安全策略、插件等。
4. Values
   - 动态配置参数，允许用户覆盖Chart的默认值。例如，通过 --set image.tag=latest 在安装时指定镜像版本。

**Chart处理流程：**

- **模板渲染**：将 Chart 中的模板文件（如deployment.yaml.tpl）与用户提供的 values.yaml 合并，生成标准的Kubernetes资源配置文件。
- **与 Kubernetes API 交互**：Helm CLI直接通过kubeconfig连接 Kubernetes API Server，提交渲染后的资源文件完成部署（Helm 3移除了服务端组件Tiller，简化了架构并提升安全性）。

Release管理：每次安装或升级生成一个 Release 实例，其状态（如配置、版本号）存储在 Kubernetes 的 Secret 或 ConfigMap 中，便于历史记录查询和回滚。

![image-20250410100753864](kubernetes/image-20250410100753864.png)

常用命令：

- `helm install <release-name> <chart>`: 安装一个 Helm Chart。

- `helm upgrade <release-name> <chart>`: 升级已安装的 Release。

- `helm uninstall <release-name>`: 卸载一个 Helm Release。

- `helm list`: 查看已安装的 Helm Release。

- `helm repo add <repo-name> <repo-url>`: 添加 Helm 仓库。

- `helm search <keyword>`: 搜索 Helm Charts。

- `helm repo update` : 更新仓库

- `helm list -n namespace`:查看名称空间下所有的Release

  



```powershell
常用helm命令
	Repostory管理
		repo命令，支持repository的add、list、remove、update和index等子命令
	Chart管理
		create、package、pull、push、dependency、search、show和verify等操作
	Release管理
		install、upgrade、get、list、history、status、rollback和uninstall等操作
```



### 部署Heml

```
https://helm.sh/zh/
```

```
https://helm.sh/zh/docs/intro/install/
```

```bash
[root@master1 ~]#wget https://get.helm.sh/helm-v3.17.3-linux-amd64.tar.gz
[root@master1 ~]#tar xf helm-v3.17.3-linux-amd64.tar.gz -C /usr/local/
[root@master1 ~]#cd /usr/local/
[root@master1 local]#ls
bin  etc  games  include  lib  linux-amd64  man  sbin  share  src
[root@master1 local]#ln -s linux-amd64/ Helm
[root@master1 local]#
[root@master1 local]#ls Helm
helm  LICENSE  README.md
[root@master1 local]#mv Helm/helm /usr/local/bin/
[root@master1 local]#ls /usr/local/bin/
helm
[root@master1 local]#helm version
version.BuildInfo{Version:"v3.17.3", GitCommit:"e4da49785aa6e6ee2b86efd5dd9e43400318262b", GitTreeState:"clean", GoVersion:"go1.23.7"}
```

### 范例：命令使用

```bash
#命令补全
[root@master1 ~]#echo 'source <(helm completion bash)' >> .bashrc

#添加仓库
[root@master1 ~]#helm repo add elastic https://helm.elastic.co
"elastic" already exists with the same configuration, skipping
[root@master1 ~]#helm repo list
NAME   	URL                    
elastic	https://helm.elastic.co

#更新索引
[root@master1 ~]#helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "elastic" chart repository
Update Complete. ⎈Happy Helming!⎈

#搜索（模糊搜索）
#在hub上搜索
[root@master1 ~]#helm search hub kibana
#在配置的仓库上搜索
[root@master1 ~]#helm search repo kibana
```



### 范例：基于Helm部署mysql主从

```bash
#首先，添加bitnami仓库
helm repo add bitnami https://charts.bitnami.com/bitnami
```

```bash
#示例1：部署单节点模式的MySQL：
helm install mysql  \
        --set auth.rootPassword=MageEdu \
        --set primary.persistence.storageClass=nfs-csi \
        --set auth.database=wpdb \
        --set auth.username=wpuser \
        --set auth.password='magedu.com' \
        bitnami/mysql \
        -n blog

#示例2：部署主从复制模式的MySQL：
helm install mysql \
  --set auth.rootPassword=MageEdu \	 # 设置 root 用户的密码
  --set global.storageClass=openebs-hostpath \	 # 指定MySQL存储使用的存储类，这里选择OpenEBS的hostpath存储类
  --set architecture=replication \	 # 设置架构为 replication，启用主从复制
  --set auth.database=wpdb \	 # 创建一个名为 wpdb 的数据库 
  --set auth.username=wpuser \	# 设置 MySQL 普通用户 wpuser 的用户名
  --set auth.password='magedu.com' \	 # 设置 wpuser 的密码
  --set secondary.replicaCount=2 \	# 设置副本的数量为 2，意味着一个主节点和二个从节点
  --set auth.replicationPassword='replpass' \	# 设置复制用户的密码，用于主从复制
  --create-namespace \	#没有名称空间，自动创建名称空间
  bitnami/mysql \	 # 使用 Bitnami 提供的 MySQL Chart
  -n blog	# 在 blog 命名空间中安装 MySQL


helm install mysql  \
        --set auth.rootPassword=MageEdu \
        --set global.storageClass=openebs-hostpath \
        --set architecture=replication \
        --set auth.database=wpdb \
        --set auth.username=wpuser \
        --set auth.password='magedu.com' \
        --set secondary.replicaCount=1 \
        --set auth.replicationPassword='replpass' \
        bitnami/mysql \
        -n blog --create-namespace
        
helm install mysql  \
        --set auth.rootPassword=MagEdu \
        --set global.storageClass=openebs-hostpath \
        --set architecture=replication \
        --set auth.database=wpdb \
        --set auth.username=wpuser \
        --set auth.password='magedu.com' \
        --set secondary.replicaCount=2 \
        --set auth.replicationPassword='replpass' \
        bitnami/mysql \
        -n blog --create-namespace
```

```bash
[root@master1 ~]#kubectl get pod -n  blog -o wide 
NAME                READY   STATUS    RESTARTS   AGE     IP            NODE             NOMINATED NODE   READINESS GATES
mysql-primary-0     1/1     Running   0          4m22s   10.244.2.59   node2.kang.com   <none>           <none>
mysql-secondary-0   1/1     Running   0          4m22s   10.244.1.55   node1.kang.com   <none>           <none>
mysql-secondary-1   1/1     Running   0          2m25s   10.244.3.31   node3.kang.com   <none>           <none>

[root@master1 ~]#kubectl get svc -n blog 
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
mysql-primary              ClusterIP   10.99.64.209    <none>        3306/TCP   5m13s
mysql-primary-headless     ClusterIP   None            <none>        3306/TCP   5m13s
mysql-secondary            ClusterIP   10.99.127.206   <none>        3306/TCP   5m13s
mysql-secondary-headless   ClusterIP   None            <none>        3306/TCP   5m13s

#列出对应的Release
[root@master1 ~]#helm list -n blog
NAME 	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART       	APP VERSION
mysql	blog     	1       	2025-04-10 11:28:16.860623104 +0800 CST	deployed	mysql-12.3.3	8.4.4      

#查看更新历史
[root@master1 ~]#helm history mysql -n blog
REVISION	UPDATED                 	STATUS  	CHART       	APP VERSION	DESCRIPTION     
1       	Thu Apr 10 11:28:16 2025	deployed	mysql-12.3.3	8.4.4      	Install complete

#回滚，默认指定前一个回滚版本
[root@master1 ~]#helm rollback RELEASE_NAME REVISION
#RELEASE_NAME	Helm release 的名字（比如 mysql）
#REVISION	历史版本号（用 helm history 查看）
```



```bash
#查看使用命令行定义的参数值
[root@master1 ~]#helm list -n blog
NAME 	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART       	APP VERSION
mysql	blog     	1       	2025-04-10 11:28:16.860623104 +0800 CST	deployed	mysql-12.3.3	8.4.4      
[root@master1 ~]#helm get values mysql -n blog
USER-SUPPLIED VALUES:
architecture: replication
auth:
  database: wpdb
  password: magedu.com
  replicationPassword: replpass
  rootPassword: MageEdu
  username: wpuser
global:
  storageClass: openebs-hostpath
secondary:
  replicaCount: 2
```



### 范例：滚动更新，回滚

#### 更新

```bash
helm upgrade mysql  \
        --set auth.rootPassword=hezhaokang01 \
        --set global.storageClass=openebs-hostpath \
        --set architecture=replication \
        --set auth.database=wpdb \
        --set auth.username=wpuser \
        --set auth.password='magedu.com' \
        --set secondary.replicaCount=2 \
        --set auth.replicationPassword='replpass' \
        bitnami/mysql \
        -n blog --create-namespace
```

```bash
[root@master1 ~]#MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace blog mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)
[root@master1 ~]#echo $MYSQL_ROOT_PASSWORD
hezhaokang01
```



#### 回滚

```bash
#查看修订版本
[root@master1 ~]#helm history mysql -n blog
REVISION	UPDATED                 	STATUS    	CHART       	APP VERSION	DESCRIPTION     
1       	Thu Apr 10 11:28:16 2025	superseded	mysql-12.3.3	8.4.4      	Install complete
2       	Thu Apr 10 13:22:59 2025	deployed  	mysql-12.3.3	8.4.4      	Upgrade complete

#回滚，REVISION不填写时默认回滚到上一个版本
[root@master1 ~]#helm rollback mysql 1 -n blog
Rollback was a success! Happy Helming!

[root@master1 ~]#helm history mysql -n blog
REVISION	UPDATED                 	STATUS    	CHART       	APP VERSION	DESCRIPTION     
1       	Thu Apr 10 11:28:16 2025	superseded	mysql-12.3.3	8.4.4      	Install complete
2       	Thu Apr 10 13:22:59 2025	superseded	mysql-12.3.3	8.4.4      	Upgrade complete
3       	Thu Apr 10 13:25:03 2025	deployed  	mysql-12.3.3	8.4.4      	Rollback to 1 
```



### 范例：基于helm部署wordpress

使用已经部署完成的现有MySQL数据库，支持Ingress，且外部的MySQL是主从复制架构。注意修改如下命令中各参数值，以正确适配到自有环境。

```bash
helm install wordpress \
       --set mariadb.enabled=false \ # 禁用默认的 MariaDB 数据库，连接外部 MySQL 数据库
       --set externalDatabase.host=mysql-primary.blog.svc.cluster.local \ # 指定外部 MySQL 数据库的主机名
       --set externalDatabase.user=wpuser \ # 外部数据库的用户名
       --set externalDatabase.password='magedu.com' \ # 外部数据库的密码
       --set externalDatabase.database=wpdb \ # WordPress 使用的数据库名称
       --set externalDatabase.port=3306 \ # 外部数据库的端口
       --set persistence.storageClass=openebs-hostpath \ # 设置存储类为 OpenEBS 的 hostpath
       --set ingress.enabled=true \ # 启用 Ingress 控制器
       --set ingress.ingressClassName=nginx \ # 指定 Ingress 控制器为 nginx
       --set ingress.hostname=blog.kang.com \ # 设置 WordPress 站点的域名
       --set ingress.pathType=Prefix \ # 设置路径类型为 Prefix，支持前缀路径路由
       --set wordpressUsername=admin \ # WordPress 的管理员用户名
       --set wordpressPassword='magedu.com' \ # WordPress 的管理员密码
       bitnami/wordpress \ # 使用 Bitnami 提供的 WordPress Helm Chart
       -n blog --create-namespace # 在 blog 命名空间中安装，并自动创建命名空间
```

```bash
helm install wordpress \
       --set mariadb.enabled=false \
       --set externalDatabase.host=mysql-primary.blog.svc.cluster.local \
       --set externalDatabase.user=wpuser \
       --set externalDatabase.password='magedu.com' \
       --set externalDatabase.database=wpdb \
       --set externalDatabase.port=3306 \
       --set persistence.storageClass=openebs-hostpath \
       --set ingress.enabled=true \
       --set ingress.ingressClassName=nginx \
       --set ingress.hostname=blog.kang.com \
       --set ingress.pathType=Prefix \
       --set wordpressUsername=admin \
       --set wordpressPassword='magedu.com' \
       bitnami/wordpress \
       -n blog --create-namespace
```

**基于dockerhub上的oci仓库部署**

下面的命令示例，将使用外部的MySQL数据库，且其访问路径为：mysql-primary.blog.svc.cluster.local。注意修改如下命令中各参数值，以正确适配到自有环境。

```bash
helm install wordpress \
    --set mariadb.enabled=false \  # 禁用内置的 MariaDB 数据库
    --set externalDatabase.host=mysql-primary.blog.svc.cluster.local \  # 外部 MySQL 数据库地址
    --set externalDatabase.user=wpuser \  # 外部数据库的用户名
    --set externalDatabase.password='magedu.com' \  # 外部数据库的密码
    --set externalDatabase.database=wpdb \  # 使用的数据库名
    --set externalDatabase.port=3306 \  # 数据库端口
    --set persistence.storageClass=openebs-hostpath \  # 存储类
    --set ingress.enabled=true \  # 启用 Ingress 控制器
    --set ingress.ingressClassName=nginx \  # 设置 Ingress 控制器类型为 nginx
    --set ingress.hostname=blog.kang.com \  # 设置 WordPress 站点的域名
    --set ingress.pathType=Prefix \  # 设置路径类型为 Prefix
    --set wordpressUsername=admin \  # 设置 WordPress 管理员用户名
    --set wordpressPassword='magedu.com' \  # 设置 WordPress 管理员密码
    oci://registry-1.docker.io/bitnamicharts/wordpress \  # 从 Docker Hub 的 OCI 路径拉取 Helm Chart
    -n blog --create-namespace  # 在 blog 命名空间中安装并创建命名空间
```

```bash
helm install wordpress \
            --set mariadb.enabled=false \
            --set externalDatabase.host=mysql-primary.blog.svc.cluster.local \
            --set externalDatabase.user=wpuser \
            --set externalDatabase.password='magedu.com' \
            --set externalDatabase.database=wpdb \
            --set externalDatabase.port=3306 \
            --set persistence.storageClass=openebs-hostpath \
            --set ingress.enabled=true \
            --set ingress.ingressClassName=nginx \
            --set ingress.hostname=blog.kang.com \
            --set ingress.pathType=Prefix \
            --set wordpressUsername=admin \
            --set wordpressPassword='magedu.com' \
            oci://registry-1.docker.io/bitnamicharts/wordpress \
            -n blog --create-namespace
```

### 范例：部署wordpress多实例

```bash
helm install wordpress \
            --set mariadb.enabled=false \
            --set externalDatabase.host=mysql-primary.blog.svc.cluster.local \
            --set externalDatabase.user=wpuser \
            --set externalDatabase.password='magedu.com' \
            --set externalDatabase.database=wpdb \
            --set externalDatabase.port=3306 \
            --set persistence.storageClass=openebs-rwx \
            --set ingress.enabled=true \
            --set ingress.ingressClassName=nginx \
            --set ingress.hostname=blog.kang.com \
            --set ingress.pathType=Prefix \
            --set wordpressUsername=admin \
            --set wordpressPassword='magedu.com' \
            --set replicaCount=2 \
            oci://registry-1.docker.io/bitnamicharts/wordpress \
            -n blog --create-namespace
```

```bash
helm install wordpress \
       --set mariadb.enabled=false \
       --set externalDatabase.host=mysql-primary.blog.svc.cluster.local \
       --set externalDatabase.user=wpuser \
       --set externalDatabase.password='magedu.com' \
       --set externalDatabase.database=wpdb \
       --set externalDatabase.port=3306 \
       --set persistence.storageClass=openebs-rwx \
       --set ingress.enabled=true \
       --set ingress.ingressClassName=nginx \
       --set ingress.hostname=blog.kang.com \
       --set ingress.pathType=Prefix \
       --set wordpressUsername=admin \
       --set wordpressPassword='magedu.com' \
       --set replicaCount=2 \
       bitnami/wordpress \
       -n blog --create-namespace
```

```
helm install wordpress \
            --set mariadb.enabled=false \
            --set externalDatabase.host=mysql-primary.blog.svc.cluster.local \
            --set externalDatabase.user=wpuser \
            --set externalDatabase.password='magedu.com' \
            --set externalDatabase.database=wpdb \
            --set externalDatabase.port=3306 \
            --set persistence.storageClass=openebs-rwx \
            --set ingress.enabled=true \
            --set ingress.ingressClassName=nginx \
            --set ingress.hostname=blog.magedu.com \
            --set ingress.pathType=Prefix \
            --set wordpressUsername=admin \
            --set wordpressPassword='magedu.com' \
            --set replicaCount=1 \
            oci://registry-1.docker.io/bitnamicharts/wordpress \
            -n blog --create-namespace    
```



### 范例：部署harbor

需要部署ingress 和 LoadBalancer

```bash
#添加更新仓库
[root@master1 harbor]#helm repo add harbor https://helm.goharbor.io
[root@master1 harbor]#helm repo update
```

#### openEBS存储类部署

基于“openebs-hostpath”存储类进行部署

```bash
[root@master1 harbor]#vim harbor-values-openebs.yaml
```

```yaml
#cat harbor-values-openebs.yaml
expose:
  # 配置暴露服务的方式为 Ingress
  type: ingress
  tls:
    enabled: true  # 启用 TLS 加密
    certSource: auto  # 自动生成证书
    auto:
      commonName: ""  # 可选项，设置证书的 Common Name
  ingress:
    hosts:
      core: registry.magedu.com  # 配置主机名，用户通过此域名访问 Harbor
    className: "nginx"  # 使用 Nginx Ingress Controller 来处理流量
    annotations:
      # 配置 Ingress 的 SSL 重定向和代理请求体大小
      ingress.kubernetes.io/ssl-redirect: "true"  # 自动将 HTTP 重定向到 HTTPS
      ingress.kubernetes.io/proxy-body-size: "0"  # 禁用请求体大小限制
      nginx.ingress.kubernetes.io/ssl-redirect: "true"  # Nginx 特定的 SSL 重定向设置
      nginx.ingress.kubernetes.io/proxy-body-size: "0"  # Nginx 特定的代理请求体大小设置
  clusterIP:
    name: harbor  # 服务名称
    staticClusterIP: ""  # 如果需要静态 IP，可以在这里指定
    ports:
      httpPort: 80  # HTTP 端口
      httpsPort: 443  # HTTPS 端口
  nodePort:
    name: harbor  # 服务名称
    ports:
      http:
        port: 80  # HTTP 端口
        nodePort: 30002  # 为 HTTP 服务暴露的节点端口
      https:
        port: 443  # HTTPS 端口
        nodePort: 30003  # 为 HTTPS 服务暴露的节点端口
  loadBalancer:
    name: harbor  # 负载均衡器的名称
    IP: ""  # 如果需要手动指定 IP，可在这里填写
    ports:
      httpPort: 80  # 负载均衡器的 HTTP 端口
      httpsPort: 443  # 负载均衡器的 HTTPS 端口

# 设置 Harbor 外部访问的 URL 地址
externalURL: https://registry.magedu.com

persistence:
  enabled: true  # 启用持久化存储
  resourcePolicy: "keep"  # 在删除 Harbor 时保留存储卷
  persistentVolumeClaim:
    registry:
      storageClass: "openebs-hostpath"  # 使用 OpenEBS 的 hostpath 存储类
      accessMode: ReadWriteOnce  # 存储访问模式
      size: 5Gi  # 记录存储大小为 5Gi
    jobservice:
      jobLog:
        existingClaim: ""  # 使用现有 PVC（如果有）
        storageClass: "openebs-hostpath"  # 存储类
        subPath: ""  # 子路径，默认为空
        accessMode: ReadWriteOnce  # 存储访问模式
        size: 1Gi  # 存储大小为 1Gi
    database:
      existingClaim: ""  # 使用现有 PVC（如果有）
      storageClass: "openebs-hostpath"  # 存储类
      accessMode: ReadWriteOnce  # 存储访问模式
      size: 1Gi  # 存储大小为 1Gi
    redis:
      existingClaim: ""  # 使用现有 PVC（如果有）
      storageClass: "openebs-hostpath"  # 存储类
      accessMode: ReadWriteOnce  # 存储访问模式
      size: 1Gi  # 存储大小为 1Gi
    trivy:
      existingClaim: ""  # 使用现有 PVC（如果有）
      storageClass: "openebs-hostpath"  # 存储类
      accessMode: ReadWriteOnce  # 存储访问模式
      size: 5Gi  # 存储大小为 5Gi

  # 配置 Harbor 镜像和图表存储
  imageChartStorage:
    disableredirect: false  # 是否禁用重定向
    type: filesystem  # 使用文件系统类型存储
    filesystem:
      rootdirectory: /storage  # 存储根目录为 /storage

# 设置 Harbor 管理员密码
harborAdminPassword: "magedu.com"

# 配置 IP 家族支持
ipFamily:
  ipv6:
    enabled: false  # 禁用 IPv6
  ipv4:
    enabled: true  # 启用 IPv4
```

```bash
[root@master1 harbor]#helm install harbor -f harbor-values-openebs.yaml harbor/harbor -n harbor --create-namespace

#harbor:RELEASE_NAME
#harbor/harbor:仓库名字/CHART_NAME
```

```
[root@master1 harbor]#kubectl get pod -n harbor
NAME                                 READY   STATUS    RESTARTS   AGE
harbor-core-64f95b6996-zsjs7         1/1     Running   0          5m28s
harbor-database-0                    1/1     Running   0          5m28s
harbor-jobservice-68ccd94c64-hpvpt   1/1     Running   0          5m28s
harbor-portal-75769cb6c6-rf7qx       1/1     Running   0          5m28s
harbor-redis-0                       1/1     Running   0          5m28s
harbor-registry-7889fdb7cb-9cbw4     2/2     Running   0          5m28s
harbor-trivy-0                       1/1     Running   0          5m28s

[root@master1 harbor]#kubectl get ingress -n harbor
NAME             CLASS   HOSTS             ADDRESS      PORTS     AGE
harbor-ingress   nginx   harbor.kang.com   10.0.0.102   80, 443   25s
```



#### 依赖于 nfs-csi 存储类

```yaml
expose:
  # 配置暴露服务的方式为 Ingress
  type: ingress
  tls:
    enabled: true  # 启用 TLS 加密
    certSource: auto  # 自动生成证书（由 Ingress Controller 或 cert-manager 提供）
  ingress:
    hosts:
      core: registry.magedu.com  # 配置主机名，用户通过此域名访问 Harbor
    className: "nginx"  # 使用 Nginx Ingress Controller 来处理流量
    annotations:
      # 配置 Ingress 的 SSL 重定向和代理请求体大小限制
      ingress.kubernetes.io/ssl-redirect: "true"  # 自动将 HTTP 请求重定向到 HTTPS
      ingress.kubernetes.io/proxy-body-size: "0"  # 禁用请求体大小限制
      nginx.ingress.kubernetes.io/ssl-redirect: "true"  # Nginx 特定的 SSL 重定向设置
      nginx.ingress.kubernetes.io/proxy-body-size: "0"  # Nginx 特定的代理请求体大小设置

# 配置 IP 家族支持
ipFamily:
  ipv4:
    enabled: true  # 启用 IPv4
  ipv6:
    enabled: false  # 禁用 IPv6

# 设置 Harbor 外部访问的 URL 地址
externalURL: https://registry.magedu.com

# 持久化存储配置部分
persistence:
  enabled: true  # 启用持久化存储
  resourcePolicy: "keep"  # 在删除 Harbor 时保留存储卷
  persistentVolumeClaim:  # 定义 Harbor 各个组件的 PVC（持久卷）
    registry:  # 配置 registry 组件的 PVC
      storageClass: "nfs-csi"  # 使用 NFS 存储类
      accessMode: ReadWriteMany  # 存储访问模式，支持多个 Pod 同时读写
      size: 5Gi  # 存储大小为 5Gi
    chartmuseum:  # 配置 chartmuseum 组件的 PVC
      storageClass: "nfs-csi"  # 使用 NFS 存储类
      accessMode: ReadWriteMany  # 存储访问模式，支持多个 Pod 同时读写
      size: 5Gi  # 存储大小为 5Gi
    jobservice:  # 配置 jobservice 组件的 PVC
      jobLog:
        storageClass: "nfs-csi"  # 使用 NFS 存储类
        accessMode: ReadWriteOnce  # 存储访问模式，仅支持一个 Pod 读写
        size: 1Gi  # 存储大小为 1Gi
      # scanDataExports:
      #   storageClass: "nfs-csi"  # 可选项：扫描数据导出 PVC 配置
      #   accessMode: ReadWriteOnce  # 存储访问模式，仅支持一个 Pod 读写
      #   size: 1Gi  # 存储大小为 1Gi
    database:  # 配置 PostgreSQL 数据库组件的 PVC
      storageClass: "nfs-csi"  # 使用 NFS 存储类
      accessMode: ReadWriteMany  # 存储访问模式，支持多个 Pod 同时读写
      size: 2Gi  # 存储大小为 2Gi
    redis:  # 配置 Redis 缓存组件的 PVC
      storageClass: "nfs-csi"  # 使用 NFS 存储类
      accessMode: ReadWriteMany  # 存储访问模式，支持多个 Pod 同时读写
      size: 2Gi  # 存储大小为 2Gi
    trivy:  # 配置 Trivy 漏洞扫描组件的 PVC
      storageClass: "nfs-csi"  # 使用 NFS 存储类
      accessMode: ReadWriteMany  # 存储访问模式，支持多个 Pod 同时读写
      size: 5Gi  # 存储大小为 5Gi

# 设置 Harbor 管理员密码
harborAdminPassword: "magedu.com"
```

```bash
helm install harbor -f harbor-values.yaml harbor/harbor -n harbor --create-namespace
```

### 范例：部署 metrics-server

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

helm upgrade --install metrics-server metrics-server/metrics-server \
  -n metrics-server --create-namespace \
  --set args="{--kubelet-insecure-tls}"
```

### 范例：部署 metallb

```
https://metallb.io/installation/
```

```powershell
1 添加仓库
helm repo add metallb https://metallb.github.io/metallb

2 安装MetalLb
helm install metallb metallb/metallb --namespace metallb-system --create-namespace

3 配置地址池
创建 IPAddressPool 和 L2Advertisement 资源
[root@master1 metallb]#cat metallb-ipaddresspool.yaml 
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: localip-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.0.0.100-10.0.0.150
  autoAssign: true
  avoidBuggyIPs: true

[root@master1 LoadBalancer-MatalLB]#cat metallb-l2advertisement.yaml 
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: localip-pool-l2a
  namespace: metallb-system
spec:
  ipAddressPools:
  - localip-pool
  interfaces:
  - eth0
```

```
kubectl apply -f . -n metallb-system
```



### 范例：部署ingress-nginx

```
https://kubernetes.github.io/ingress-nginx/deploy/
```

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

```bash
#添加promethus监控
helm upgrade ingress-nginx ingress-nginx \
  --install \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.metrics.enabled=true \
  --set-string controller.podAnnotations."prometheus\.io/scrape"="true" \
  --set-string controller.podAnnotations."prometheus\.io/port"="10254" \
  --set controller.service.type=LoadBalancer
```

```bash
[root@master1 ~]#kubectl get po -n ingress-nginx 
NAME                                       READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-b49d9c7b9-xcgmc   1/1     Running   0          28m
[root@master1 ~]#kubectl get svc -n ingress-nginx 
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.106.137.131   10.0.0.101    80:32160/TCP,443:31941/TCP   28m
ingress-nginx-controller-admission   ClusterIP      10.96.191.111    <none>        443/TCP                      28m
```

### 范例：helm部署OpenEBS

```
helm repo add openebs https://openebs.github.io/openebs
```

部署OpenEBS，并禁用了本地的zfs和lvm存储引擎，以及复制引擎Mayastor。

```bash
helm upgrade openebs --install --namespace openebs openebs/openebs --set engines.replicated.mayastor.enabled=false \
            --set engines.local.zfs.enabled=false --set engines.local.lvm.enabled=false --create-namespace
```

> 提示：OpenEBS 4.x系列，目前的Chart并不支持使用“--set nfs-provisioner.enabled=true”选项来启用nfs provisioner，需要该功能时，建议额外使用下面的命令进行手动部署。
>
> ```bash
> kubectl apply -f https://openebs.github.io/charts/nfs-operator.yaml
> ```

### 范例：部署minio

```
https://min.io/docs/minio/kubernetes/upstream/operations/install-deploy-manage/deploy-minio-tenant-helm.html
```

```bash
#添加仓库
helm repo add minio-operator https://operator.min.io

#安装Operator Pod
helm install --namespace minio-operator --create-namespace operator minio-operator/operator

#下载Helm charts
curl -O https://raw.githubusercontent.com/minio/operator/master/helm-releases/tenant-7.0.1.tgz
#解压缩
tar xf tenant-7.0.1.tgz

#修改 Tenant values
vim tenant/values.yaml
...
  pools:
    ###
    # The number of MinIO Tenant Pods / Servers in this pool.
    # For standalone mode, supply 1. For distributed mode, supply 4 or more.
    # Note that the operator does not support upgrading from standalone to distributed mode.
    - servers: 3	# 3 个 MinIO Pod
      ###
      # Custom name for the pool
      name: pool-0	# 池的名称，可以自定义
      ###
      # The number of volumes attached per MinIO Tenant Pod / Server.
      volumesPerServer: 3		#每个 Pod 3 个 PVC
      ###
      # The capacity per volume requested per MinIO Tenant Pod.
      size: 1Gi		# 每个 PVC 的大小
      storageClassName: openebs-hostpath		#添加此行，设置存储类
...

#部署minio tenant
helm install minio ./tenant --namespace minio-operator
```

```bash
#配置 LoadBalancer 使用外部连接
[root@master1 minio]#cat minio-console-lb.yaml 
apiVersion: v1
kind: Service
metadata:
  name: minio-console-lb
  namespace: minio-operator
spec:
  selector:
    v1.min.io/tenant: myminio
  ports:
    - port: 9443
      targetPort: 9443
      protocol: TCP
      name: https
  type: LoadBalancer
```

### 范例：部署ELK

```bash
#添加仓库
helm repo add elastic https://helm.elastic.co

vim elasticsearch-values.yaml

# Elasticsearch 集群配置示例
replicas: 3                      # Elasticsearch 节点数，生产环境建议至少3个节点
minimumMasterNodes: 2            # 最小主节点数量，避免脑裂问题，通常为 (replicas / 2 + 1)

# 持久化存储配置
volumeClaimTemplate:
  accessModes: 
    - ReadWriteOnce             # 存储访问模式，单节点读写
  resources:
    requests:
      storage: 5Gi             # 持久化存储大小，根据实际数据量调整

# 资源请求和限制
resources:
  requests:
    cpu: "1000m"                # 请求的CPU资源，1核
    memory: "2Gi"               # 请求的内存资源，2GB
  limits:
    cpu: "1000m"                # 允许最大CPU资源，1核
    memory: "2Gi"               # 允许最大内存资源，2GB

# 节点角色分配
nodeSelector: {}                # 可用于节点亲和，指定节点标签
tolerations: []                 # 容忍污点，确保Pod能调度到特定节点
affinity: {}                   # 调度亲和策略，按需配置

# 安全和认证
esConfig:                       # Elasticsearch配置文件，可自定义
  elasticsearch.yml: |
    xpack.security.enabled: false  # 关闭安全认证（测试环境），生产环境建议开启并配置证书等

# 网络设置
service:
  type: ClusterIP                # 服务类型，默认为ClusterIP，也可以设置为LoadBalancer
  ports:
    - name: http
      port: 9200
      targetPort: 9200

# 初始化和存储路径
persistence:
  enabled: true                 # 启用持久化存储
  storageClass: "openebs-hostpath" # 使用的存储类名称
  size: 5Gi                   # 存储容量，需和volumeClaimTemplate中一致

# StatefulSet 使用 volumeClaimTemplate 创建 PVC
volumeClaimTemplate:
  accessModes:
    - ReadWriteOnce
  storageClassName: "openebs-hostpath"  # 这里也需要写上 storageClassName，确保PVC使用指定存储类
  resources:
    requests:
      storage: 5Gi

# Pod 亲和、调度等高级配置
podManagementPolicy: "Parallel" # Pod 管理策略，默认Parallel比OrderedReady效率高

# 关闭集群自动快照等实验功能（如不需要）
# masterService: elasticsearch-master

# 自定义启动参数
extraEnvs:                     # 额外环境变量，可配置Java堆大小等
  - name: ES_JAVA_OPTS
    value: "-Xms1g -Xmx1g"


helm install elasticsearch elastic/elasticsearch --namespace logging -f elasticsearch-values.yaml --create-namespace

1. 观察所有集群成员的出现.
  $ kubectl get pods --namespace=logging -l app=elasticsearch-master -w
2. 检索 elastic 用户的密码.
  $ kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
3. 使用 Helm 测试来测试集群健康状况.
  $ helm --namespace=logging test elasticsearch

```

```bash


helm install kibana elastic/kibana \
  --namespace logging \
  -f kibana-values.yaml \
  --create-namespace
```







### *范例：部署nacos

```bash
#添加仓库
helm repo add bitnami https://charts.bitnami.com/bitnami

#部署MySQL
helm install mysql  \
        --set auth.rootPassword=Nacos \
        --set global.storageClass=openebs-hostpath \
        --set architecture=replication \
        --set auth.database=ncdb \
        --set auth.username=ncuser \
        --set auth.password='nacos.com' \
        --set secondary.replicaCount=2 \
        --set auth.replicationPassword='replpass' \
        bitnami/mysql \
        -n nacos --create-namespace

#初始化 SQL 路径
https://github.com/alibaba/nacos/blob/2.4.0/distribution/conf/mysql-schema.sql

#先把 mysql-schema.sql 复制进 Pod
#先运行一个 mysql-client 容器并保持开启：
kubectl run mysql-client --rm -it --image=bitnami/mysql -n nacos --env MYSQL_PWD=Nacos -- bash

#然后 新开一个终端或窗口，把本地文件拷贝进去：
kubectl cp mysql-schema.sql nacos/mysql-client:/tmp/mysql-schema.sql

#然后在第一个终端里执行：
mysql -h mysql-primary.nacos.svc.cluster.local -uroot -pNacos ncdb < /tmp/mysql-schema.sql

#添加 nacos 仓库
helm repo add ygqygq2 https://ygqygq2.github.io/charts/

#编辑valuse
nacos:
  mysql:
    host: my-nacos-mysql-primary.nacos.svc.cluster.local
    port: 3306
    db: nadb
    user: ncuser
    password: nacos.com

#安装
helm upgrade --install nacos nacos/nacos -f values.yaml -n nacos


helm install nacos . \
  --namespace nacos \
  --create-namespace \
  --set replicaCount=3 \
  --set mysql.enabled=false \
  --set externalMysql.enabled=true \
  --set externalMysql.dbHost=mysql-primary.nacos.svc.cluster.local \
  --set externalMysql.dbPort=3306 \
  --set externalMysql.dbName=ncdb \
  --set externalMysql.dbUser=ncuser \
  --set externalMysql.dbPassword='nacos.com' \
  --set storage.storageClass=openebs-hostpath \
  --set service.type=NodePort \
  --set auth.enabled=true

```



### 自定义 Chart

#### 固定配置的chart

```
https://docs.helm.sh/docs/chart_template_guide/getting_started/
```

```bash
# 创建chart文件结构
root@master01:/data#helm create myapp-chart
Creating myapp-chart

root@master01:/data#tree myapp-chart/
myapp-chart/
├── charts
├── Chart.yaml                        # 必须项，包含了该chart的描述，helm show chart [CHART] 查看到即此文件内容
├── templates                         # 包括了各种资源清单的模板文件
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml                       # 如果templates/目录中包含变量时,可以通过此文件提供变量的默认值
                                      # 这些值可以在用户执行 helm install 或 helm upgrade 时被覆盖
                                      # helm show values  [CHART]  查看到即此文件内容
3 directories, 10 files
```

```bash
# 删除不需要的文件
root@master01:/data# rm -rf myapp-chart/templates/* myapp-chart/values.yaml myapp-chart/charts/
root@master01:/data# tree
.
└── myapp-chart
    ├── charts
    ├── Chart.yaml
    ├── templates
    └── values.yaml

# 生成相关的资源清单文件
root@master01:/data# kubectl create deployment myapp --image registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1 --replicas 3 --dry-run=client -o yaml > myapp-chart/templates/myapp-deployment.yaml
root@master01:/data# kubectl create service nodeport myapp --tcp 80:80 --dry-run=client -o yaml > myapp-chart/templates/myapp-service.yaml
root@master01:/data# tree myapp-chart/
myapp-chart/
├── Chart.yaml
└── templates
    ├── myapp-deployment.yaml
    └── myapp-service.yaml

# 修改清单文件
root@master01:/data# vim myapp-chart/templates/myapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test:v0.1
        name: pod-test

root@master01:/data# vim myapp-chart/templates/myapp-service.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    app: myapp
  name: myapp
spec:
  ports:
  - name: 80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: myapp
  type: NodePort

# 修改配置
root@master01:/data# vim myapp-chart/Chart.yaml
apiVersion: v2
name: myapp-chart
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "0.1.0"

# 检查语法
root@master01:/data# helm lint myapp-chart/
==> Linting myapp-chart/
[INFO] Chart.yaml: icon is recommended
[INFO] values.yaml: file does not exist

1 chart(s) linted, 0 chart(s) failed

# 部署应用
root@master01:/data# helm install myapp ./myapp-chart/ --create-namespace --namespace helmdemo
NAME: myapp
LAST DEPLOYED: Tue May  6 16:50:12 2025
NAMESPACE: helmdemo
STATUS: deployed
REVISION: 1
TEST SUITE: None

root@master01:/data# kubectl get pod -n helmdemo 
NAME                     READY   STATUS    RESTARTS   AGE
myapp-6d67457cf5-4s5mg   1/1     Running   0          32s
myapp-6d67457cf5-6rpc4   1/1     Running   0          32s
myapp-6d67457cf5-86l8z   1/1     Running   0          32s

# 将目录打包至文件
root@master01:/data# helm package ./myapp-chart/
Successfully packaged chart and saved it to: /data/myapp-chart-0.1.0.tgz
root@master01:/data# ll myapp-chart-0.1.0.tgz 
-rw-r--r-- 1 root root 788  5月  6 16:51 myapp-chart-0.1.0.tgz
```

#### 可变配置的 Chart

```bash
root@master01:/data# helm create myweb-chart
Creating myweb-chart
root@master01:/data# tree myweb-chart/
myweb-chart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

# 删除多余的文件
root@master01:/data# rm -rf myweb-chart/templates/*
root@master01:/data# tree myweb-chart/
myweb-chart/
├── charts
├── Chart.yaml
├── templates
└── values.yaml

# 创建资源清单文件
root@master01:/data# kubectl create deployment myweb --image nginx:1.22.0 --replicas=3 --dry-run=client -o yaml > myweb-chart/templates/myweb-deployment.yaml
root@master01:/data# kubectl create service nodeport myweb --tcp 80:80  --dry-run=client -o yaml > myweb-chart/templates/myweb-service.yaml

# 修改清单文件为动态模版文件
root@master01:/data#vim myweb-chart/templates/myweb-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.deployment_name }}
  #namespace: {{ .Values.namespace }} 
  namespace: {{ .Release.Namespace }}
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.pod_label }}
  template:
    metadata:
      labels:
        app: {{ .Values.pod_label }}
    spec:
      containers:
      - image: {{ .Values.image }}:{{ .Values.imageTag }}
        name: {{ .Values.container_name }}
        ports:
        - containerPort: {{ .Values.containerport }}
        
root@master01:/data#vim myweb-chart/templates/myweb-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service_name }}
  namespace: {{ .Release.Namespace }}
spec:
  ports:
  - port: {{ .Values.port }}
    protocol: TCP
    targetPort: {{ .Values.targetport }}
  selector:
    app: {{ .Values.pod_label }}
  type: NodePort
  
# 编辑values.yaml文件
root@master01:/data#vim myweb-chart/values.yaml
#namespace: default
deployment_name: myweb-deployment
replicas: 3
pod_label: myweb-pod-label
image: registry.cn-beijing.aliyuncs.com/wangxiaochun/pod-test
imageTag: v0.1
container_name: myweb-container
service_name: myweb-service
port: 80
targetport: 80
containerport: 80

# 查看Chart.yaml
root@master01:/data#grep -v "#" myweb-chart/Chart.yaml
apiVersion: v2
name: myweb-chart
description: A Helm chart for Kubernetes

type: application

version: 0.1.0

appVersion: "1.16.0"

root@master01:/data# tree myweb-chart/
myweb-chart/
├── charts
├── Chart.yaml
├── templates
│   ├── myweb-deployment.yaml
│   └── myweb-service.yaml
└── values.yaml

#部署
root@master01:/data# helm install myweb ./myweb-chart/ --create-namespace --namespace helmtest
NAME: myweb
LAST DEPLOYED: Tue May  6 17:03:19 2025
NAMESPACE: helmtest
STATUS: deployed
REVISION: 1
TEST SUITE: None

root@master01:/data# kubectl get pod -n helmtest 
NAME                                READY   STATUS    RESTARTS   AGE
myweb-deployment-5c54699c4d-5ghfj   1/1     Running   0          49s
myweb-deployment-5c54699c4d-fp87c   1/1     Running   0          49s
myweb-deployment-5c54699c4d-snxxx   1/1     Running   0          49s

#打包
root@master01:/data# helm package ./myweb-chart/
Successfully packaged chart and saved it to: /data/myweb-chart-0.1.0.tgz
root@master01:/data# ll
-rw-r--r--  1 root root  953  5月  6 17:04 myweb-chart-0.1.0.tgz
```





## 认证体系与Service Account

### Kubernetes的访问控制体系

```powershell
安全管控
	认证、鉴权、准入控制（3A）
		Authentication、Authorization、Admission
		认证：核验请求者身份的合法性
		鉴权：核验请求的操作是否获得许可
		准入控制：检查操作内容是否合规
	
	认证:身份核验过程遵循“或”逻辑，且任何一个插件核验成功后都将不再进行后续的插件验证
		均不成功，则失败，或以“匿名者”身份访问
		建议禁用“匿名者”
	授权:鉴权过程遵循“或”逻辑，且任何一个插件对操作的许可授权后都将不再进行后续的插件验证
		均未许可，则拒绝请求的操作
	准入控制:内容合规性检查过程遵循“与”逻辑，且无论成败，每次的操作请求都要经由所有插件的检验
		将数据写入etcd前，负责检查内容的有效性，因此仅对“写”操作有效
		分两类:validating(校验)和mutating(补全或订正)
	
	集群组件：
		Master(Control Plane):
			API Server: 无状态的http服务，（http协议格式的数据库），把 etcd 的能力封装为一个个数据存储方案（资源类型 kind）
			etcd: 自由方案的k/v存储
			Controller Manager: 控制器
			Scheduler: Pod 调度器
		Worker(Data Plane):
			Kubelet: Kubernetes 节点上的守护进程，它接收来自 API Server 的 Pod 指令，并确保这些 Pod 被正确创建、运行，并处于健康状态。
			Kube Proxy: 监视 API Server 上的 Service 的怎删改查操作，并动态的转成自己所在节点上的规则
			
		CRI: containerd、docker(cri-dockerd)、CRI-O
		CNI: flannel、Calico、Cilium
		CSI: (非必须) OpenEBS
			
		三个网络：
			节点网络，Service Network，Pod Network
				Service:服务发现、负载均衡
					服务发现: label selector(标签选择器)
```

![image-20250413140005501](kubernetes/image-20250413140005501.png)

#### **1. 认证（Authentication, Authn）**

- 核心逻辑：`或`逻辑（短路模式）
  - 用户请求依次通过多个认证插件（如 Token、Client Cert、OIDC 等）。
  - **任意一个插件认证成功**，即通过认证，后续插件跳过。
  - **全部失败**：返回 `401 Unauthorized` 或允许匿名访问（若启用）。
- 关键点：
  - 匿名访问需显式配置（如 `system:anonymous` 用户）。
  - 常见插件：`Bearer Token`、`X509 Client Cert`、`Basic Auth`。

------

#### **2. 授权（Authorization, Authz）**

- 核心逻辑：`或`逻辑（短路模式）
  - 通过认证的用户需被授权（如 RBAC、ABAC、Node 等插件）。
  - **任意一个插件授权通过**，请求被允许，后续插件跳过。
  - **全部拒绝**：返回 `403 Forbidden`。
- 关键点：
  - RBAC 是最常用插件（通过 `Role`/`ClusterRole` 绑定权限）。
  - 超级用户（如 `cluster-admin`）默认拥有所有权限。

------

#### **3. 准入控制（Admission Control）**

- 核心逻辑：`与`逻辑（一票否决制）
  - **仅对写操作**（如 `CREATE/UPDATE/DELETE`）生效。
  - 请求需通过所有准入插件的校验，**任意一个插件拒绝则整体失败**。
- 插件分类：
  - **Validating**：合规性检查（如资源配额、Pod 安全策略）。
  - **Mutating**：自动补全或修改请求（如注入 Sidecar、默认值填充）。
- 关键点：
  - 执行顺序：先 `Mutating` → 后 `Validating`。
  - 常见插件：`ResourceQuota`、`PodSecurity`、`DefaultStorageClass`。

------

#### **4. 完整流程示例**

1. **用户请求** → **认证（Authn）** → 失败则 `401`，成功进入下一步。
2. **授权（Authz）** → 失败则 `403`，成功进入下一步。
3. **写操作?** → 是则触发准入控制，否则直接返回结果。
   - 准入插件全部通过 → 请求生效。
   - 任意插件拒绝 → 返回错误（如 `LimitExceeded`）。

------

#### **常见问题排查思路**

- **`401` 错误**：检查认证插件配置（如 Token 是否过期）。
- **`403` 错误**：检查 RBAC 权限绑定（如 `RoleBinding` 是否匹配）。
- **准入拒绝**：查看 `kubectl get events` 或 API Server 日志（如 `ResourceQuota` 超限）。



### API 身份验证

```powershell
Kubernetes上的用户
	“用户”即服务请求者的身份指代，一般使用身份标识符进行识别
		用户标识：用户名或者ID
		用户组
		
Kubernetes系统的用户大一可分为2类
	Service Account:服务账户，指Pod内进场范文API service时使用的身份信息
		API Service使用ServiceAccount类型的资源对象来保存该类账号
		认证到API Service的认证信息称为Service Account tocken,他们保存于同名的专用类型的Secret对象中
		名称空间级别
	
	User Account: 用户账号，指非Pod类的客户端访问API Service时使用的身份标识，一般指现实中的人
		APIServer没有为这类账户提供保存其信息的资源类型，相关的信息通常保存于外部的文件或认证系统中
		身份核验操作可由API Server进行，也可能是由外部身份认证服务完成
		本身非由Kubernetes管理，因而作用域为整个集群级别
	
	不能识别Service Account，也不能识别User Account的用户，即“匿名用户”
	
身份认证策略：
	X.509客户端证书认证
	持有者令牌（bearer tocken）
		静态令牌文件（Static Token File）
		Bootstrap令牌（引导令牌）
		Service Account令牌
		OIDC(OpenID Connect)令牌
		Webhook令牌
	身份认证代理（Authenticating Proxy）
	匿名请求
	
kubelet 也有 内在的API Service和访问控制
	kubectl exec -it ... ——> API Service ——> kubelet ——> pod
						        客户端			服务端
	
	|-----------|-------------------------------|		
	|Kubelet API| 功能简介			             |
	|-----------|-------------------------------|
	| /pods		| 列出当前kubelet节点上的pod	  |
	|-----------|-------------------------------|		
	| /run	    | 在一个容器内运行指定的命令		  |
	|-----------|-------------------------------|
	| /exec		| 在一个容器内运行指定的命令	      |
	|-----------|-------------------------------|		
	| /config	| 设置Kubelet的配置文件参数	 	 |
	|-----------|-------------------------------|
	| /debug	| 调试设置                       |
	|-----------|-------------------------------|
	
#查看系统默认能支持的认证	
cat /etc/kubernetes/manifests/kube-apiserver.yaml
...
spec:
	x509客户端证书认证
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
	Bootstrap令牌认证
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
	身份代理认证
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
	Service Account认证
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
...
```

```powershell
Service Account:
	在kubernetes上创建一个pod，就会在同一个名称空间下自动创建一个默认的 Service Account 名为default。
	这个default的Service Account授权了接入到API Service，访问自己这个pod自身，这个名字内部的容器，他所属的这个namespace等信息。通过Service Account token认证到API Aervice
	每创建一个pod，都会被自动关联到Service Account上来
	也可以额外创建Service Account
	
User Account:
	X.509客户端证书认证
		API service 要先trust（信任）这个 CA
		任何客户端用该 CA 签发的 Cret （证书），来访问API Service
		那么API service信任了该CA，它就信任了该证书
		在这证书里面的 
			CN 被识别为“用户名"  
			O 被识别为 “group（组）”
		需要先认证到认证中心，认证中心才会签发令牌
		
静态令牌文件（Static Token File）	
	静态令牌文件
		令牌信息保存于文本文件中
		由kube-apiserver在启动时通过--token-auth-file选项加载
		加载完成后的文件变动，仅能通过重启程序进行重载，因此，相关的令牌会长期有效
		客户端在HTIP请求中，通过“Authorization Bearer TOKEN”标头附带令牌令牌以完成认证
```



#### 范例：静态令牌认证

```powershell
静态令牌认证的基础配置
	令牌信息保存于文本文件中
		文件格式为CSV，每行定义一个用户，由“令牌、用户名、用户ID和所属的用户组”四个字段组成，用户组为可选字段
		格式:token,user,uid,"groupl,group2,group3
	由kube-apiserver在启动时通过--token-auth-fle选项加载
	加载完成后的文件变动，仅能通过重启程序进行重载，因此，相关的令牌会长期有效
	客户端在HTTP请求中，通过“Authorization Bearer TOKEN”标头附带令牌令牌以完成认证
	
配置示例
1 生成token，命令:echo"$(openssl rand -hex 3).$(openssl rand -hex 8)"
2 生成static token文件
3 配置kube-apiserver加载该静态令牌文件以启用相应的认证功能
4 测试，命令:
		curl -k-H"Authorization: Bearer TOKEN"-k https://API SERVER:6443/api/v1/namespaces/default/pods/
```

```powershell
1 在宿主机上创建令牌文件
2 在API Service的pod上创建卷，基于host path加载所在节点上定义的令牌文件
3 在API Service容器上挂载卷
4 编辑API service的命令行选项，可以能加载卷上的文件，作为使用令牌文件

注意：加载后是在内存中使用的，在宿主即上修改文件，不会生效，要想生效，重启API Service
```

```bash
#生成随机tocken
[root@master1 ~]#echo "$(openssl rand -hex 3).$(openssl rand -hex 8)"
7411c7.96e1f6bdda487c76
[root@master1 ~]#echo "$(openssl rand -hex 3).$(openssl rand -hex 8)"
06f301.1c5bfd3fff1965cd
[root@master1 ~]#echo "$(openssl rand -hex 3).$(openssl rand -hex 8)"
9a326d.dc62a0f2fa288767

#准备文件
[root@master1 ~]#cd /etc/kubernetes/
[root@master1 kubernetes]#mkdir authfile
[root@master1 kubernetes]#chmod 600 authfile/
[root@master1 kubernetes]#cd authfile/
[root@master1 kubernetes]#vim tokens.csv
7411c7.96e1f6bdda487c76,xiaoming,1001,kubeadmin
06f301.1c5bfd3fff1965cd,xiaohong,2001,kubeusers
9a326d.dc62a0f2fa288767,xiaolan,2002,"kubeadmin,developers"		#多个组要加引号，并且逗号隔开
```

```bash
#注意：不要再manifests文件下编辑
[root@master1 ~]#cp /etc/kubernetes/manifests/kube-apiserver.yaml .
[root@master1 ~]#cp kube-apiserver.yaml /tmp/
[root@master1 ~]#vim /tmp/kube-apiserver.yaml 
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.0.0.20:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.0.0.20
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --token-auth-file=/etc/kubernetes/authfile/tokens.csv	#添加此行
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: registry.aliyuncs.com/google_containers/kube-apiserver:v1.32.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 10.0.0.20
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 10.0.0.20
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 10.0.0.20
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/authfile/tokens.csv	#添加此行和下面两行
      name: auth-tokens
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
  - hostPath:				#添加此行和下面三行
      path: /etc/kubernetes/authfile/tokens.csv
      type: FileOrCreate
    name: auth-tokens
status: {}
```

```bash
[root@master1 ~]#cp /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```



```bash
#测试访问，可以识别用户名字，但是没有权限
[root@master1 ~]#curl -H"Authorization: Bearer 7411c7.96e1f6bdda487c76" -k https://10.0.0.20:6443/api
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.0.20:6443"
    }
  ]
}
[root@master1 ~]#curl -H"Authorization: Bearer 7411c7.96e1f6bdda487c76" -k https://10.0.0.20:6443/api/v1/namespace/default/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "namespace \"default\" is forbidden: User \"xiaoming\" cannot get resource \"namespace/pods\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "name": "default",
    "kind": "namespace"
  },
  "code": 403
}

#worker节点访问
kubectl options 	#查看options

[root@node1 ~]#kubectl get pods --server=https://10.0.0.20:6443 --token="7411c7.96e1f6bdda487c76" --insecure-skip-tls-verify
Error from server (Forbidden): pods is forbidden: User "xiaoming" cannot list resource "pods" in API group "" in the namespace "default"

--insecure-skip-tls-verify			#跳过证书认证
--server=https://10.0.0.20:6443		#指定服务器地址
--token="7411c7.96e1f6bdda487c76"	#指定token

#每个节点上都有ca.crt证书
[root@node1 ~]#ls /etc/kubernetes/pki/
ca.crt
[root@node1 ~]#kubectl get pods --server=https://10.0.0.20:6443 --token="7411c7.96e1f6bdda487c76" --certificate-authority=/etc/kubernetes/pki/ca.crt
Error from server (Forbidden): pods is forbidden: User "xiaoming" cannot list resource "pods" in API group "" in the namespace "default"

--certificate-authority=/etc/kubernetes/pki/ca.crt	#指定证书路径
```



#### 范例：X509数字证书认证

Kubernetes集群中的X509客户端认证依赖于PKI证书体系,有如下三套CA证书系统

![image-20250413172032424](kubernetes/image-20250413172032424.png)



```powershell
3组私有CA
1 etcd-ca
	1 etcd各个节点之间互相作为集群成员节点通信证书（对等证书），实现各个节点之间对等通信
	2 etcd作为 kube-apiservice 服务端，还需要一个服务端证书（servercert）
	3 给 kube-apiservice 签发一个客户端证书（clientcert），实现双向数字证书认证
2 kubernetes-ca
	1 kube-apiservice 作为服务端的服务端证书（servercert）
	2 kube-scheduler、kube-controller-manager、kube-proxy、kubectl 作为客户端，访问kube-apiservice 的客户端证书（clientcert），实现双向数字证书认证
	3 kube-apiservice 作为客户端请求 kubelet 服务端的客户端证书（clientcert）
	4 kubelet 作为服务端的服务端证书（servercert）
3 front-proxy-ca
	kube-aggregator 聚合层
		所有的请求都发给聚合层，如果请求的时系统内置的，就发给 kube-apiservice，如果不是系统内置的就发给extension-apiservice（第三方的apiservice）
	kube-aggregator 和 extension-apiservice（第三方的apiservice）之间通信签发的证书
	
kubeadm部署Kubernetes集群时会自动生成所需要的证书，它们位于/etc/kubernetes/pki自录下
```

```powershell
[root@master1 ~]#ls /etc/kubernetes/pki/ -l
total 60
-rw-r--r-- 1 root root 1298 Apr 12 17:48 apiserver.crt
-rw-r--r-- 1 root root 1123 Apr 12 17:48 apiserver-etcd-client.crt
-rw------- 1 root root 1679 Apr 12 17:48 apiserver-etcd-client.key
-rw------- 1 root root 1675 Apr 12 17:48 apiserver.key
-rw-r--r-- 1 root root 1176 Apr 12 17:48 apiserver-kubelet-client.crt
-rw------- 1 root root 1675 Apr 12 17:48 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1107 Apr 12 17:48 ca.crt
-rw------- 1 root root 1675 Apr 12 17:48 ca.key
drwxr-xr-x 2 root root 4096 Apr 12 17:48 etcd
-rw-r--r-- 1 root root 1123 Apr 12 17:48 front-proxy-ca.crt
-rw------- 1 root root 1679 Apr 12 17:48 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Apr 12 17:48 front-proxy-client.crt
-rw------- 1 root root 1679 Apr 12 17:48 front-proxy-client.key
-rw------- 1 root root 1675 Apr 12 17:48 sa.key
-rw------- 1 root root  451 Apr 12 17:48 sa.pub		#令牌签发中心，实现动态令牌管理机制

[root@master1 ~]#ls /etc/kubernetes/pki/etcd/ -l
total 32
-rw-r--r-- 1 root root 1094 Apr 12 17:48 ca.crt
-rw------- 1 root root 1679 Apr 12 17:48 ca.key
-rw-r--r-- 1 root root 1123 Apr 12 17:48 healthcheck-client.crt
-rw------- 1 root root 1675 Apr 12 17:48 healthcheck-client.key
-rw-r--r-- 1 root root 1220 Apr 12 17:48 peer.crt
-rw------- 1 root root 1675 Apr 12 17:48 peer.key
-rw-r--r-- 1 root root 1220 Apr 12 17:48 server.crt
-rw------- 1 root root 1679 Apr 12 17:48 server.key

[root@node1 ~]#ls /var/lib/kubelet/pki/ -l
total 12
-rw------- 1 root root 1118 Apr 12 17:49 kubelet-client-2025-04-12-17-49-08.pem
lrwxrwxrwx 1 root root   59 Apr 12 17:49 kubelet-client-current.pem -> /var/lib/kubelet/pki/kubelet-client-2025-04-12-17-49-08.pem
-rw-r--r-- 1 root root 2311 Apr 12 17:49 kubelet.crt
-rw------- 1 root root 1679 Apr 12 17:49 kubelet.key


第一组etcd-ca
	etcd的证书和私钥
		/etc/kubernetes/pki/etcd/ca.crt
		/etc/kubernetes/pki/etcd/ca.key
	etcd对等证书和私钥
		/etc/kubernetes/pki/etcd/peer.crt
		/etc/kubernetes/pki/etcd/peer.key
	etcd作为服务端的证书和私钥
		/etc/kubernetes/pki/etcd/server.crt
		/etc/kubernetes/pki/etcd/server.key
	api-service作为客户端的证书和私钥
		/etc/kubernetes/pki/apiserver-etcd-client.crt
		/etc/kubernetes/pki/apiserver-etcd-client.key
第二组kubernetes-ca
	kube-apiservice的证书和私钥
		/etc/kubernetes/pki/ca.crt
		/etc/kubernetes/pki/ca.key
	kube-apiservice作为服务端的证书和私钥
		/etc/kubernetes/pki/apiserver.crt
		/etc/kubernetes/pki/apiserver.key
	kube-apiservice作为客户端访问kubelet的证书和私钥
		/etc/kubernetes/pki/apiserver-kubelet-client.crt
		/etc/kubernetes/pki/apiserver-kubelet-client.key
	kubelet的证书和私钥，各个worker节点的
		/var/lib/kubelet/pki/kubelet.crt
		/var/lib/kubelet/pki/kubelet.key
```

```powershell
X509数字证书认证到API Server
1 创建证书签署请求
2 有Kubernetes CA签署证书
3 用户使用证书认证到API server
创建有Kubernetes CA签署的证书
	创建证书签署请求
		openssl genrsa -out mason.key 2048
		openssl req -new -key mason.key -out mason.csr -subj "/CN=mason/O=kubeadmin"
	
	签署证书（方式一）	
	创建CertificateSignRequest资源
	清单文件
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: mason
spec:
  request: （base64编码后的csr文件）
  expirationSeconds: 864000  # ten days
  usages:
  - client auth


		kubectl apply -f certificatesignrequest-mason.yaml
		kubectl get csr
NAME     AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
mason   10s   kubernetes.io/kube-apiserver-client   kubernetes-admin   10d                 Pending

		kubectl certificate approve mason
			approve	签署
			deny	拒签
			
	获取证书，保存为证书文件
		kubectl get csr mason -o jsonpath='{.status.certificate}'| base64 -d > mason.crt 
			
			
	签署证书（方式二）直接基于Kubernetes CA签署并生成证书文件
		openssl x509 -req -days 10 -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -in ./mason.csr -out ./mason.crt
	
	将签署好的证书发送到node1上面测试,只需要拷贝mason.key、mason.crt
	kubectl测试
		kubectl get pods --client-certificate=./mason.crt --client-key=./mason.key --server=https://10.0.0.20:6443/ --certificate-authority=/etc/kubernetes/pki/ca.crt
		kubectl get pods --client-certificate=./mason.crt --client-key=./mason.key --server=https://10.0.0.20:6443/ --insecure-skip-tls-verify
	curl测试
		curl --cert ./mason.crt --key ./mason.key https://10.0.0.20:6443/api/v1/default/pods --cacert /etc/kubernetes/pki/ca.crt
		
```



### kubeconfig

```powershell
使用 kubeadm 部署的集群,具有管理员权限的认证请求文件：/etc/kubernetes/admin.conf
```

```powershell
kubectl config SUBCOMMAND [options]
  Available Commands:
  current-context   显示当前上下文
  delete-cluster    从kubeconfig中删除指定的集群
  delete-context    从kubeconfig中删除指定的上下文从
  delete-user       kubeconfig中删除指定的用户
  get-clusters      显示在kubeconfig中定义的集群
  get-contexts      描述一个或多个上下文
  get-users         显示在kubeconfig中定义的用户
  rename-context    从kubeconfig文件中重命名上下文
  set               在 kubeconfig 文件中设置单个值
  set-cluster       在kubeconfig中设置集群条目
  set-context       在kubeconfig中设置上下文条目
  set-credentials   在kubeconfig中设置用户条目
  unset             在 kubeconfig 文件中取消设置单个值
  use-context       在kubeconfig文件中设置当前上下文
  view              显示合并的kubeconfig设置或指定的kubeconfig文件
  
  
加载机制优先级
	1 在命令行中指定 --kubeconfig 选项指定的
	2 使用环境变量指定的 KUBECONFIG 
		环境变量可以指定多个路径，中间用冒号‘:’隔开
		eg:
			KUBECONFIG="FILE1:FILE2:..."
	3 默认文件: $HOME/.kube/config


#添加集群
kubectl config set-cluster mykube01 --server=https://10.0.0.20:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true
#添加账号
kubectl config set-credentials xiaoming --token='7411c7.96e1f6bdda487c76'
kubectl config set-credentials mason --client-certificate=./mason.crt --client-key=./mason.key --embed-certs=true
#创建contexts，关联集群和用户
kubectl config set-context xiaoming@mykube01 --cluster=mykube01 --user=xiaoming
kubectl config set-context mason@mykube01 --cluster=mykube01 --user=mason
#切换上下文
kubectl config use-context xiaoming@mykube01
#删除集群
kubectl config delete-cluster mykube02

#环境变量，从左至右，左边优先级最高
export KUBECONFIG="/root/.kube/config:/root/.kube/kubefile"

```

```powershell
[root@master1 ~]#kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED	CA证书
    server: https://10.0.0.20:6443		集群地址
  name: kubernetes

表示集群的连接信息：
	server: API Server 的地址（这里是 10.0.0.20:6443）
	certificate-authority-data: 集群的 CA 证书，用于验证 API Server 身份（已省略）
	
contexts:
- context:
    cluster: kubernetes		集群
    user: kubernetes-admin	用户
  name: kubernetes-admin@kubernetes
表示使用哪个集群 + 哪个用户进行连接。
可以配置多个上下文，快速切换。

current-context: kubernetes-admin@kubernetes
当前正在使用的上下文。你现在使用的是 kubernetes-admin 用户连接 kubernetes 集群。

kind: Config
preferences: {}
users:
- name: kubernetes-admin	用户名
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
表示客户端使用的身份认证方式，这里是基于证书的：
	client-certificate-data: 用户的证书（Base64 编码，省略了）
	client-key-data: 对应私钥
```

  ![image-20250414165634491](kubernetes/image-20250414165634491.png)



#### 范例：创建config文件

```bash
#添加集群
[root@node1 ~]#kubectl config set-cluster mykube01 --server=https://10.0.0.20:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true

set-cluster mykube01		#设置一个名为 mykube01 的集群配置
--server=https://10.0.0.20:6443		#Kubernetes API Server 的地址
--certificate-authority=/etc/kubernetes/pki/ca.crt	#CA 根证书，用于校验 API Server 的合法性
--embed-certs=true		#把证书的内容直接嵌入到配置文件里（base64 编码），而不是只写路径。方便在其他机器上使用这个 kubeconfig

#查看
[root@node1 ~]#kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.20:6443
  name: mykube01
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null

#查看集群定义
[root@node1 ~]#kubectl config get-clusters 
NAME
mykube01

#这个命令执行完会在你的 ~/.kube/config 中添加配置
[root@node1 ~]#cat .kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: 证书
    server: https://10.0.0.20:6443
  name: mykube01
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null
```

```bash
#添加账号
[root@node1 ~]#kubectl config set-credentials xiaoming --token='7411c7.96e1f6bdda487c76'

#查看token
[root@master1 ~]#cat /etc/kubernetes/authfile/tokens.csv 
7411c7.96e1f6bdda487c76,xiaoming,1001,kubeadmin
06f301.1c5bfd3fff1965cd,xiaohong,2001,kubeusers
9a326d.dc62a0f2fa288767,xiaolan,2002,"kubeadmin,developers"

#查看
[root@node1 ~]#kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.20:6443
  name: mykube01
contexts: null
current-context: ""
kind: Config
preferences: {}
users:
- name: xiaoming
  user:
    token: REDACTED

#查看user
[root@node1 ~]#kubectl config get-users 
NAME
xiaoming
```

```bash
#创建contexts，关联集群和用户
#创建contexts，关联mykube01和xiaoming
[root@node1 ~]#kubectl config set-context xiaoming@mykube01 --cluster=mykube01 --user=xiaoming

[root@node1 ~]#kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.20:6443
  name: mykube01
contexts:
- context:
    cluster: mykube01
    user: xiaoming
  name: xiaoming@mykube01
current-context: ""
kind: Config
preferences: {}
users:
- name: xiaoming
  user:
    token: REDACTED
```

```bash
[root@node1 ~]#kubectl get pod --context='xiaoming@mykube01'
Error from server (Forbidden): pods is forbidden: User "xiaoming" cannot list resource "pods" in API group "" in the namespace "default
```

```bash
#查看现有上下文
kubectl config get-contexts

#切换上下文
kubectl config use-context <上下文名称>

eg:
[root@node1 ~]#kubectl config get-contexts
CURRENT   NAME                CLUSTER    AUTHINFO   NAMESPACE
          xiaoming@mykube01   mykube01   xiaoming   
[root@node1 ~]#kubectl config use-context xiaoming@mykube01

[root@node1 ~]#kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.20:6443
  name: mykube01
contexts:
- context:
    cluster: mykube01
    user: xiaoming
  name: xiaoming@mykube01
current-context: xiaoming@mykube01
kind: Config
preferences: {}
users:
- name: xiaoming
  user:
    token: REDACTED
```

```bash
[root@node1 ~]#kubectl get pod
```

```bash
#添加另一个账号
[root@node1 ~]#cd /etc/kubernetes/pki/user-certs/mason/
[root@node1 mason]#ls
mason.crt  mason.csr  mason.key
[root@node1 mason]#kubectl config set-credentials mason --client-certificate=./mason.crt --client-key=./mason.key --embed-certs=true
User "mason" set.

[root@node1 mason]#kubectl config get-users 
NAME
mason
xiaoming

[root@node1 mason]#kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.20:6443
  name: mykube01
contexts:
- context:
    cluster: mykube01
    user: xiaoming
  name: xiaoming@mykube01
current-context: xiaoming@mykube01
kind: Config
preferences: {}
users:
- name: mason
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: xiaoming
  user:
    token: REDACTED
```

```bash
#关联集群
[root@node1 mason]#kubectl config set-context mason@mykube01 --cluster=mykube01 --user=mason
Context "mason@mykube01" created.
[root@node1 mason]#kubectl config get-contexts 
CURRENT   NAME                CLUSTER    AUTHINFO   NAMESPACE
          mason@mykube01      mykube01   mason      
*         xiaoming@mykube01   mykube01   xiaoming   
```

```bash
[root@node1 mason]#kubectl get pod --context=mason@mykube01 

#修改mason为默认值
[root@node1 mason]#kubectl config use-context mason@mykube01 
Switched to context "mason@mykube01".
[root@node1 mason]#kubectl config get-contexts 
CURRENT   NAME                CLUSTER    AUTHINFO   NAMESPACE
*         mason@mykube01      mykube01   mason      
          xiaoming@mykube01   mykube01   xiaoming  
```



```bash
#环境变量
[root@node1 miller]#kubectl config set-cluster mykube02 --server=https://10.0.0.20:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --kubeconfig=/root/.kube/kubefile
[root@node1 miller]#export KUBECONFIG=/root/.kube/kubefile
[root@node1 miller]#kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.0.0.20:6443
  name: mykube02
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null

```



### RBAC鉴权

```powershell
API Server中的鉴权模块
	Node: 专用的鉴权模块，它基于kubelet将要运行的POd向kubelet进行授权
	ABAC: 通过将属性（包括资源属性、用户属性、对象和环境属性等）组合在一起的策略，将访问权限授予用户
	RBAC: 基于企业个人用户的角色来管理对计算机或网络资源的访问的鉴权方法；
	Webhook: 用于支持同Kubernetes外部的授权机制进行集成
	
	RBAC：
		Role-based access control
		角色
		
		陈述句
			主:Subject
			谓:动作
			宾:Object
		权限
			给Object施加的动作
			
		Role:
			能过对那些Object施加那些动作
			
		Subject ——> Bonding ——> Role
		从而Subject就拥有了这个Role的权限
		
	API:HTTPS协议
		Method: GET/POST/HEAD/PUT/DELETE/OPTIONS/...
		Object: URL PATH 端点
			/api/v1/namespace/default/pods/mypod
			
	ABAC:RBAC的超集
		Attribution 
			管控条件扩展到了请求报文的任何属性，而不仅仅是 Method 和 URL
		策略编程：
			OPA: Open Policy agent (开源的策略代理)
 
RBAC:
	Subject:
		UserAccount
		ServiceAccount
		
	Action:
		HTTP Method
			读操作：get,watch,list
			写操作：create,delete,update/edit/replace,deldtecollection
			
	Object:
		各类型的资源对象：读写
			每一个资源对象都有一个 URL，并不是每一个 URL 都是资源对象
			名称空间级别
			集群级别
		非资源型URL：读
		
	为实现RBAC机制：
		提供四个资源类型
			ROle: 名称空间级别资源类型进行许可授权
			ClusterRole: 名称空间和集群级别资源类型进行许可授权
			RoleBinding: Subject ——> RoleBinding ——> Role
						 Subject ——> RoleBinding ——> ClusterRole
						 	ClusterRole 会被降级使用：
                            	集群级别的不会生效
                            	名称空间级别的只会在 RoleBinding 所在的一个名称空间级
			ClusterRoleBing: Subject ——> ClusterRoleBinding ——> ClusterRole
			
			RoleBinding 可以 Binding Role和ClusterRole
			ClusterRoleBing 不可以 Binding Role 只能 Binding ClusterRole
			
[root@master1 ~]#kubectl get clusterrolebindings.rbac.authorization.k8s.io  kubeadm:cluster-admins -o yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2025-04-12T09:48:22Z"
  name: kubeadm:cluster-admins
  resourceVersion: "238"
  uid: 437a5486-b7b4-416f-a056-2a0d54e3e300
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: kubeadm:cluster-admins
	
kubernetes-admin 用户 ，所属的组为 kubeadm:cluster-admins
	所以kubernetes-admin就有了该集群的管理权限
	
可用的写权限
	verbs:
	- create
	- delete
	- deletecollection
	- update
	- patch
```

| 默认 ClusterRole  |       绑定的 ClusterRoleBinding        |                       描述（核心权限）                       |
| :---------------: | :------------------------------------: | :----------------------------------------------------------: |
| **cluster-admin** |          `system:masters` 组           | **最高权限**： - 通过 `ClusterRoleBinding` 授权时：可操作集群及所有命名空间的任何资源。 - 通过 `RoleBinding` 授权时：可操作绑定命名空间的所有资源（包括 `Namespace` 自身）。 |
|     **admin**     | 无默认绑定（需手动关联 `RoleBinding`） | **命名空间管理员权限**： - 读写命名空间内大多数资源（如 Pod、Deployment）。 - 可创建子 `Role` 和 `RoleBinding`。 - **限制**：不能操作 `ResourceQuota` 和 `Namespace` 本身。 |
|     **edit**      |               无默认绑定               | **编辑权限**： - 读写命名空间内大多数对象（包括 `Secret`）。 - **限制**：不可查看或修改 `Role` 和 `RoleBinding`。 |
|     **view**      |               无默认绑定               | **只读权限**： - 可查看命名空间内大多数对象。 - **限制**：不可访问 `Role`、`RoleBinding` 和 `Secret`。 |

1. **ClusterRole 与 RoleBinding 的关系**

   - `ClusterRole` 是集群级别的角色模板，通过 `ClusterRoleBinding` 或 `RoleBinding` 绑定到用户/组。
   - 区别：
     - `ClusterRoleBinding`：权限作用于整个集群（如 `cluster-admin`）。
     - `RoleBinding`：权限仅作用于单个命名空间（如 `admin`、`edit`、`view`）。

2. **权限递进逻辑**

   ```
   view（只读） → edit（读写，不含RBAC） → admin（读写+RBAC管理） → cluster-admin（全集群控制）
   ```

3. **实际应用场景**

   - **`cluster-admin`**：集群运维人员（需谨慎分配）。
   - **`admin`**：命名空间负责人（如开发团队Leader）。
   - **`edit`**：普通开发者（需修改资源但无需管理权限）。
   - **`view`**：监控或审计人员（仅需查看资源状态）。

```bash
#默认开启的鉴权
[root@master1 ~]#cat /etc/kubernetes/manifests/kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.0.0.20:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.0.0.20
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
...
```



#### 范例:创建用户 binding

```bash
#创建relo
[root@master1 ~]#kubectl create role readers --verb=get,list,watch --resource=pods,services,deployments,statefulsets -n default --dry-run=client -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: readers
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - deployments
  - statefulsets
  verbs:
  - get
  - list
  - watch

[root@master1 ~]#kubectl create role readers --verb=get,list,watch --resource=pods,services,deployments,statefulsets -n default

[root@master1 ~]#kubectl get role
NAME      CREATED AT
readers   2025-04-15T12:36:17Z

```

```bash
#使用relobinding,绑定relo和user
[root@master1 ~]#kubectl create rolebinding xiaoming-as-readers --role=readers --user=xiaoming -n default --dry-run=client -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: xiaoming-as-readers
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: readers
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: xiaoming

[root@master1 ~]#kubectl create rolebinding xiaoming-as-readers --role=readers --user=xiaoming -n default

[root@node1 ~]#kubectl get pods --context='xiaoming@mykube01'
No resources found in default namespace.

```

```bash
#创建ClusterRelo
[root@node1 ~]#kubectl create clusterrole cluster-readers --verb=get,list,watch --resource=persistentvolumes,storageclasses,daemonsets,ingresses --dry-run=client -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: cluster-readers
rules:
- apiGroups:
  - ""
  resources:
  - persistentvolumes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - daemonsets
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - get
  - list
  - watch

[root@master1 ~]#kubectl create clusterrole cluster-readers --verb=get,list,watch --resource=persistentvolumes,storageclasses,daemonsets,ingresses

[root@master1 ~]#kubectl get clusterrole cluster-readers 
NAME              CREATED AT
cluster-readers   2025-04-15T12:56:28Z
```

```bash
#使用relobinding,绑定ClusterRelo和user
[root@master1 ~]#kubectl create clusterrolebinding mason-as-cluster-readers --clusterrole=cluster-readers --user=mason --dry-run=client -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: null
  name: mason-as-cluster-readers
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-readers
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: mason

[root@master1 ~]#kubectl create clusterrolebinding mason-as-cluster-readers --clusterrole=cluster-readers --user=mason
```

```bash
#使用relobinding,绑定ClusterRelo和user
[root@master1 ~]#kubectl create rolebinding jerry-as-cluster-readers --clusterrole=cluster-readers --user=xiaohong

[root@node1 ~]#kubectl get ingress --context="xiaohong@mykube02"
No resources found in default namespace.

```



范例：

```bash
#创建一个用户，授予默认名称空间下的所有权限
[root@master1 ~]#kubectl create rolebinding mason-as-default-ns-admin --clusterrole=admin --user=mason
rolebinding.rbac.authorization.k8s.io/mason-as-default-ns-admin created

```

### ServiceAccount

```powershell
ServiceAccount：API上的标准资源类型，namespace 级的
	权限并不极限与名称空间，权限局限于使用的是 clusterrolebinding 还是 rolebinding ，binding 到那个 role 或 clusterrole 上
	认证到API Server
		serviceAccount Token:
			API Server 就是 Token 的签发者 
			Issuer:API Server
	获得授权：RBAC
		额外的权限，必须有RBAC进行定义
每一个名称空间都会自动生成一个 default 的 ServiceAccouint

ServiceAccount Token的不同实现方式
Kubernetes v1.20-
	系统自动生成专用的Secret对象，并基于secret卷插件关联至相关的Pod;
	Secret中会自动附带Token，且永久有效;
Kubernetes v1.21-y1.23:
	系统自动生成专用的Secret对象，并通过projected卷插件关联至相关的Pod;
	Pod不会使用Secret上的Token，而是由Kubelet向TokenRequest API请求生成，默认有效期为一年，且每小时更新一次;
Kubernetes v1.24+:
	系统不再自动生成专用的Secret对象
	由Kubelet负责向TokenRequestAPI请求生成Token
	默认卷路径
		/var/run/secrets/kubernetes.io/serviceaccount
		
如果使用了ServiceAccount可以将连接harbor仓库的Secrets定义到ServiceAccount
kubectl explain sa.imagePullSecrets
如果harbor仓库密码，只需要改ServiceAccount，所有使用ServiceAccount定义的pod都修改
```

#### 范例：创建ServiceAccount

```bash
[root@master1 ~]#kubectl create namespace jenkins
[root@master1 ~]#kubectl create serviceaccount jenkins-master -n jenkins --dry-run=client -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: jenkins-master
  namespace: jenkins
[root@master1 ~]#kubectl create serviceaccount jenkins-master -n jenkins
[root@master1 ~]#kubectl get sa -n jenkins 
NAME             SECRETS   AGE
default          0         106s
jenkins-master   0         46s

#授权jenkins名称空间下的管理员权限
[root@master1 ~]#kubectl create rolebinding jenkins-master-as-ns-admin --clusterrole=admin --serviceaccount=jenkins:jenkins-master -n jenkins
								   namespace:serviceAccount
rolebinding.rbac.authorization.k8s.io/jenkins-master-as-ns-admin created


[root@master1 ~]#vim myapp.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: jenkins
spec:
  containers:
  - name: mypod
    image: ikubernete/demoapp:v1.0
  serviceAccountName: jenkins-master



#kubectl exec -it -n jenkins mypod -- sh
#cd /run/secrets/kubernetes.io/serviceaccount/

#curl -k -H "Authorization: Bearer $(cat token)" https://kubernetes.default.svc.cluster.local/api/v1/namespaces/jenkins/pods/
```



### 范例：安装部署kuboard

```bash
---
apiVersion: v1
kind: Namespace
metadata:
  name: kuboard

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: kuboard-v3-config
  namespace: kuboard
data:
  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-built-in.html
  # [common]
  KUBOARD_ENDPOINT: 'http://kuboard.magedu.com'
  KUBOARD_AGENT_SERVER_UDP_PORT: '30081'
  KUBOARD_AGENT_SERVER_TCP_PORT: '30081'
  KUBOARD_SERVER_LOGRUS_LEVEL: info  # error / debug / trace
  # KUBOARD_AGENT_KEY 是 Agent 与 Kuboard 通信时的密钥，请修改为一个任意的包含字母、数字的32位字符串，此密钥变更后，需要删除 Kuboard Agent 重新导入。
  KUBOARD_AGENT_KEY: 32b7d6572c6255211b4eec9009e4a816  

  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-gitlab.html
  # [gitlab login]
  # KUBOARD_LOGIN_TYPE: "gitlab"
  # KUBOARD_ROOT_USER: "your-user-name-in-gitlab"
  # GITLAB_BASE_URL: "http://gitlab.mycompany.com"
  # GITLAB_APPLICATION_ID: "7c10882aa46810a0402d17c66103894ac5e43d6130b81c17f7f2d8ae182040b5"
  # GITLAB_CLIENT_SECRET: "77c149bd3a4b6870bffa1a1afaf37cba28a1817f4cf518699065f5a8fe958889"
  
  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-github.html
  # [github login]
  # KUBOARD_LOGIN_TYPE: "github"
  # KUBOARD_ROOT_USER: "your-user-name-in-github"
  # GITHUB_CLIENT_ID: "17577d45e4de7dad88e0"
  # GITHUB_CLIENT_SECRET: "ff738553a8c7e9ad39569c8d02c1d85ec19115a7"

  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-ldap.html
  # [ldap login]
  # KUBOARD_LOGIN_TYPE: "ldap"
  # KUBOARD_ROOT_USER: "your-user-name-in-ldap"
  # LDAP_HOST: "ldap-ip-address:389"
  # LDAP_BIND_DN: "cn=admin,dc=example,dc=org"
  # LDAP_BIND_PASSWORD: "admin"
  # LDAP_BASE_DN: "dc=example,dc=org"
  # LDAP_FILTER: "(objectClass=posixAccount)"
  # LDAP_ID_ATTRIBUTE: "uid"
  # LDAP_USER_NAME_ATTRIBUTE: "uid"
  # LDAP_EMAIL_ATTRIBUTE: "mail"
  # LDAP_DISPLAY_NAME_ATTRIBUTE: "cn"
  # LDAP_GROUP_SEARCH_BASE_DN: "dc=example,dc=org"
  # LDAP_GROUP_SEARCH_FILTER: "(objectClass=posixGroup)"
  # LDAP_USER_MACHER_USER_ATTRIBUTE: "gidNumber"
  # LDAP_USER_MACHER_GROUP_ATTRIBUTE: "gidNumber"
  # LDAP_GROUP_NAME_ATTRIBUTE: "cn"

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kuboard-etcd
  namespace: kuboard
  labels:
    app: kuboard-etcd
spec:
  serviceName: kuboard-etcd
  replicas: 3
  selector:
    matchLabels:
      app: kuboard-etcd
  template:
    metadata:
      name: kuboard-etcd
      labels:
        app: kuboard-etcd
    spec:
      containers:
      - name: kuboard-etcd
        image: eipwork/etcd:v3.4.14
        ports:
        - containerPort: 2379
          name: client
        - containerPort: 2380
          name: peer
        env:
        - name: KUBOARD_ETCD_ENDPOINTS
          value: >-
            kuboard-etcd-0.kuboard-etcd:2379,kuboard-etcd-1.kuboard-etcd:2379,kuboard-etcd-2.kuboard-etcd:2379
        volumeMounts:
        - name: data
          mountPath: /data
        command:
          - /bin/sh
          - -c
          - |
            PEERS="kuboard-etcd-0=http://kuboard-etcd-0.kuboard-etcd:2380,kuboard-etcd-1=http://kuboard-etcd-1.kuboard-etcd:2380,kuboard-etcd-2=http://kuboard-etcd-2.kuboard-etcd:2380"
            exec etcd --name ${HOSTNAME} \
              --listen-peer-urls http://0.0.0.0:2380 \
              --listen-client-urls http://0.0.0.0:2379 \
              --advertise-client-urls http://${HOSTNAME}.kuboard-etcd:2379 \
              --initial-advertise-peer-urls http://${HOSTNAME}:2380 \
              --initial-cluster-token kuboard-etcd-cluster-1 \
              --initial-cluster ${PEERS} \
              --initial-cluster-state new \
              --data-dir /data/kuboard.etcd
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      # 请填写一个有效的 StorageClass name
      storageClassName: openebs-rwx
      accessModes: [ "ReadWriteMany" ]
      resources:
        requests:
          storage: 5Gi


---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kuboard-data-pvc
  namespace: kuboard
spec:
  storageClassName: openebs-hostpath
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
apiVersion: v1
kind: Service
metadata:
  name: kuboard-etcd
  namespace: kuboard
spec:
  type: ClusterIP
  ports:
  - port: 2379
    name: client
  - port: 2380
    name: peer
  selector:
    app: kuboard-etcd

---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: '9'
    k8s.kuboard.cn/ingress: 'false'
    k8s.kuboard.cn/service: NodePort
    k8s.kuboard.cn/workload: kuboard-v3
  labels:
    k8s.kuboard.cn/name: kuboard-v3
  name: kuboard-v3
  namespace: kuboard
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s.kuboard.cn/name: kuboard-v3
  template:
    metadata:
      labels:
        k8s.kuboard.cn/name: kuboard-v3
    spec:
      containers:
        - env:
            - name: KUBOARD_ETCD_ENDPOINTS
              value: >-
                kuboard-etcd-0.kuboard-etcd:2379,kuboard-etcd-1.kuboard-etcd:2379,kuboard-etcd-2.kuboard-etcd:2379
          envFrom:
            - configMapRef:
                name: kuboard-v3-config
          image: 'eipwork/kuboard:v3'
          imagePullPolicy: Always
          name: kuboard
          volumeMounts:
            - mountPath: "/data"
              name: kuboard-data
      volumes:
      - name: kuboard-data
        persistentVolumeClaim:
          claimName: kuboard-data-pvc

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    k8s.kuboard.cn/workload: kuboard-v3
  labels:
    k8s.kuboard.cn/name: kuboard-v3
  name: kuboard-v3
  namespace: kuboard
spec:
  ports:
    - name: webui
      nodePort: 30080
      port: 80
      protocol: TCP
      targetPort: 80
    - name: agentservertcp
      nodePort: 30081
      port: 10081
      protocol: TCP
      targetPort: 10081
    - name: agentserverudp
      nodePort: 30081
      port: 10081
      protocol: UDP
      targetPort: 10081
  selector:
    k8s.kuboard.cn/name: kuboard-v3
  sessionAffinity: None
  type: NodePort
```

**ingress**

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kuboard-v3
  namespace: kuboard
spec:
  ingressClassName: nginx
  rules:
  - host: kuboard.kang.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: kuboard-v3
            port:
              number: 80
        pathType: Prefix
```

```powershell
初始化账号密码：admin/Kuboard123

添加集群
	Token
	KubeConfig
	Kuboard Agent
```

#### Token获取方法

在master上执行下面命令

```
cat << EOF > kuboard-create-token.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: kuboard

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kuboard-admin
  namespace: kuboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kuboard-admin-crb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kuboard-admin
  namespace: kuboard

---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  annotations:
    kubernetes.io/service-account.name: kuboard-admin
  name: kuboard-admin-token
  namespace: kuboard
EOF

kubectl apply -f kuboard-create-token.yaml 
echo -e "\033[1;34m将下面这一行红色输出结果填入到 kuboard 界面的 Token 字段：\033[0m"
echo -e "\033[31m$(kubectl -n kuboard get secret $(kubectl -n kuboard get secret kuboard-admin-token | grep kuboard-admin-token | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d)\033[0m"

```

#### KubeConfig获取方法

```bash
[root@master1 Kuboard]#cat ~/.kube/config 
[root@master1 Kuboard]#kubectl config view --raw
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJT2UwcFNMUUFwN1F3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBME1USXdPVFF6TVRSYUZ3MHpOVEEwTVRBd09UUTRNVFJhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUNpTVc0NDlydk5zcXhUdmU4c1poRU5CLzN4M2JwSEVjeTRtT3hOQW9YckFFZWQ0OGRuaFFubjdhcU8KL0RRQnYwa0xNK3JRM243RE9meE1sckVyQmVOaWNLY1NNUUNPT2R5dXUrQ1hpbEcrRXRLeC8yMkIyTkU3emtPWgpRNUt5cENPM0ViUGxkdndhMFJ5Mmk3SVlyZWlkRVUxWHRaRUJVK3ZsTHh0RTU5SEpoUjA0TWhkMjRRYU5qblM4CjlmRDZhcTduOFVrRkZIREZKK3FNZmRmV0JhMkdUWmlVSUhhZ0hGOTU0andHUUhORHBFY0VUOXZjNTFucDVEbWsKZmYvbU1mZUorS1ZUcFpFOW5RcUhUMXExdVFJQTZkZlhtK2t4QjRHdjl1MFA4L2FBcUVXQXFlQXQvNWZTVzc3dwpiSy9SSU40Kzh2MXdpNFJsNjl4Y1ArU09ZTnQzQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRTUJLajdCR01DZDlYelRMMnp2UHNvNjc3YlpEQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQUlvdXhnWXF5NwpBT0pYNG9BWm15b1V3VVRUMUZickxVZEVNZ0Y4ZnVBazdXOVB1TUJQeGZTS2hqd3RRanRTdStnZGNyeXliYnh5CkF1eGNzQjdtWCszMVBFYlUwL0VnRGU5K1pId21BQmU4dWNYMlZxMS9KaDFTL2FBQlBwMFN2Q2lUUTU2MnZsaVoKNnVldjNUM3lWMURxZ1dvbHp3blpZYkV3cGU0T0ZGK0gxL2NNcm9uTU1sekxMaENBczdOUWF0WTZVdnZ1SnJ2KwpvVGtXS2dCTFZiUHRvWnpRZXFBSjJ5eTRtZVhVNjl0eHM1aWYvR2U5YThuQTdTVi9LL1UvTnBqUjFod3pvUHVjCnNoV0xNc2JJSk5DcmRYb09RMk1iTEtYN0RQcnZuenpsWUZPLzdkZnhEQ1lYa1pQNDVMMVdFZDhpRU80azI3V28KVzJZNW1OSExoZUpFCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://10.0.0.20:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURLVENDQWhHZ0F3SUJBZ0lJWjM4aUFPWVBFVkV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBME1USXdPVFF6TVRSYUZ3MHlOakEwTVRJd09UUTRNVFJhTUR3eApIekFkQmdOVkJBb1RGbXQxWW1WaFpHMDZZMngxYzNSbGNpMWhaRzFwYm5NeEdUQVhCZ05WQkFNVEVHdDFZbVZ5CmJtVjBaWE10WVdSdGFXNHdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEYWZNTmIKeXBKeHAwRG5nSTN1cmVOMERSbEo2c00zSGxNdU14RkZPT1dHT2Ztc0lWNXZlOUI5MGF0MEZoYmFOcEJhY1ZKOApxWnB4VFJzR0NQQ3ZTeFdWVll0L0U3Ukdhc0t1RmhURm9UcjZ0TUFNdkpvdmtXa1I0QXZ3amJhN0Y1a1ozMVBBCkZLYlhOR05Fck01VkNFditURExjeGpSTmliVWVJV2tzdHRObjNySVlldUZKL1dXWnI2bUtGQm00eEJ1Wk1FMjEKL3VabjZkNUo0SzNMd05tMEpDeElDeHRQSUFuWEZ1RGJKTWwyQ05qb3V4SWVBMkorTXJGbXVHT0lyREdLWmZZcwpiZjV0V0NCL25zdnpjU1orTVBQdUExVkRCaW5CbTlYMFU0RXBReS9PckUzTjBpcWRCNGhGU0NEbnZicWtLS1psCkxJekIzTHlseGhhczR1V0RBZ01CQUFHalZqQlVNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUsKQmdnckJnRUZCUWNEQWpBTUJnTlZIUk1CQWY4RUFqQUFNQjhHQTFVZEl3UVlNQmFBRkF3RXFQc0VZd0ozMWZOTQp2Yk84K3lqcnZ0dGtNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUJFdlU2c3RCd0huZGV4QXpuN01jRWQzZ29DCnRaQW9WaGFUN0w3a0g2RTg3ZnJmUmV3N3Zsd05pajFTZSs3Nkk1UGZ1NjJXenBxcElyeFZrMnZSMWRScVhBcWgKMVlpMEQ4MGZMNHBzTWdWWGYwWnRWcmdjRjJkOXZrNDRKZUE4a2dTTEFReGhia2hONHFMSlV0d09rd1gyaXM5cAplRHg0UnVPZHlxL1pqSmFLbm80Nm8rYVRBVzFWNjFUa0k1VXBFb2tEVjkvcStGNm5zZVNwSmhjTy9uWlZmOVRkClRNRlhHbFRndC9kRTZwVWMrQU5lWkhzbmV0Y2h0UXFVRU8rMkxwd29BdDF3aUlzUmtJQnNqS2p1WUhnWDZnVk8KL1hFbkRLZ2hEVGhSMkphZkxKdmozL2p6dWhsRzllVG1veWR3RktjSGFGWE1WVERSSkZkYXV5dkZNaEhQCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBMm56RFc4cVNjYWRBNTRDTjdxM2pkQTBaU2VyRE54NVRMak1SUlRqbGhqbjVyQ0ZlCmIzdlFmZEdyZEJZVzJqYVFXbkZTZkttYWNVMGJCZ2p3cjBzVmxWV0xmeE8wUm1yQ3JoWVV4YUU2K3JUQURMeWEKTDVGcEVlQUw4STIydXhlWkdkOVR3QlNtMXpSalJLek9WUWhML2t3eTNNWTBUWW0xSGlGcExMYlRaOTZ5R0hyaApTZjFsbWErcGloUVp1TVFibVRCTnRmN21aK25lU2VDdHk4RFp0Q1FzU0FzYlR5QUoxeGJnMnlUSmRnalk2THNTCkhnTmlmakt4WnJoamlLd3hpbVgyTEczK2JWZ2dmNTdMODNFbWZqRHo3Z05WUXdZcHdadlY5Rk9CS1VNdnpxeE4KemRJcW5RZUlSVWdnNTcyNnBDaW1aU3lNd2R5OHBjWVdyT0xsZ3dJREFRQUJBb0lCQVFDb3FlZWwxSnQ0WVVVWgpjWVFmM056Wm1jTUw3TThHbmNKWXg5TnRQSjd6SDQ0OTh3U1R5MkVIdi9RN2lWSGE0b1JOVFc0QURtM0xTVnF6CkxmT2ZYcmNxc1A4ZStuY1FaUm9raWFjL2FWZStjZ3BQeXNpOEwrU01pQWl2aXJhbGQzSVpKdTNnT2hFUjBMOVIKSkpXanp1UGJTN2QzOXdvcFVVVWdIV3F6dWU4NUhxT1RwaStWKytqSUJSWlZ6V3Z5QlozZ0t0ZWNqNXdSMkJsbQpOa3dYZ3c0cWs3OS9nY29lUlpiZnhsNWdhUWtSdllFcnZlb0twVE9Da05obDB3NzU4Q2F1d1R2bGE5ZHdzWGFGCjZSYm1sSXhrZFhWcFROeFJQN0N5K25NZUdEV2hsM21lQ25KOWdUQnI2amZBSk5Fcy92UzYvLzdxRkhpSm1FWXQKV2xzZjJ5Z0JBb0dCQVBlK0Z3eUw5M2x4S1kvdU51akxzMk8wbkowRFFnbmNGUkZDN2xFWUllU3h2U0lpaGFrQwpxUGQ3dmhmMiszNmdSK2FKNEhyQkpPdFNTb2J2YXFYY3RRNURYdFpIWHRUUTQ1ZnhNcDhxUFZTNDBiMWxkMnVSCjJPR25ENVRiK01xTG9FNGlEOUtLRUlROE1yV1ZXY1FSWlRBQkgrOXhaZWFDUnYyb1NUOStvYklqQW9HQkFPSEYKRERiakM1aVdFMDRaQXVrSS9UVUhxNGpUNVRlemorMUNJWTBMOUVMOHVic3hWY3JldzQzZ2Rob2thRHQ2anRFaApLbmlOMnNBb3p1NXdtMXZOK1QxYVFhL1BlLzJWN3VrYklxMlk1RGdrUDlTOC8zOVJnQzBoU3M1OTBwa3RXMzl3CnJlS3hjTWVyQk9WQmhHQ2lJQVdSSGg2SUtUOUdkbXBHckpDY2tzVWhBb0dCQU5NeUZMb1ljLzd1VG0wcHVWdVoKazdNUzNGUXAyOWxGNmh2T0FCWFh5Y1VKRkdBT0ovMmRpK2QyY09aREljQ2Y2TXVLakhoNVFQenZLU09BNUZ6RApHd0l1d3FGUE5IT2VJL2Q2b2huM3kxTDNQNjRDMnR3eitEemR5eld1bEpndWtaa3FCbTBJVCs0NjEwdmZKeWd6CllCeWRTTms1eFpITlM3R2dEZGw0SFdZYkFvR0FDbUx0VCswY0VIWC9CMTNCTTRWVldNWTBqd1BvaktwM0dad3MKUFBmcTBkWWNtVThJdWwrTE1aQzgvakRrbHEvcHVCZEZnK3hLdndKaG1yaVZmU0M1c2FmZ1U3MUE0QWF3eWdxVQppdFg0MGRoaEUyRnFnNm4xTXA1UWViVnlKZGZmV0xxUFZWbUNiYjBoYVlhZEYzRDk5aU9aOWgrZmZpaTRzK1R5CmRXaXVtK0VDZ1lBZ2QvZW9KMkFKWi9yQ1drc2NNMmxWR1VyOTh1bHJrR1JFMWZncHRCTGRDeVloalJvdThiTjAKZWlmSEhLY3ZHU1ZBMTZqbjRzWXcvRGN5b1I0V0tHaCtlazVxcXlUT3JrazJaaEYySHBSRXE4c08vcHlQSEp6NApwRDJiRWdwWVRWK3BYLzRtU1FUZnRUTHpTLy9sTVRrWVNzL0FHTEhGUk9DWHhCNFdxYTVKOWc9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```





## Kubernetes指标系统

### Kubernetes指标流水线 

```powershell
指标系统
	kubectl top pod
	kubectl top node
	
	dashboard|kuboard
		
		node: cpu、ram用量信息
		pod: cpu、ram
		
	声明式API
		kubectl get ...
		
		资源群组
			metrics.k8s.io
		
		Metrics Server
		
		promethus自动发现机制：
			kubernetes role
				pod
				Service
				Ingress
				Node
				Endpoints
			
		注解设置：
			promethus.io/scrape="true"
			promethus.io/port="PORT"
			promethus.io/metrhics="/metrics"
		
	Promethus Adapater(Promethus 适配器)：可以把promethus格式指标转化为kubernetes格式的指标
    
    核心指标流水线
    	client ——> API Service ——> Metrics Service ——> Kubelet
    	metrics.k8s.io/v1beta1
    
    https://APISERVICE:PORT/apis/metrics.k8s.io/v1beta1/namespace/defauilt/pods/mypod
    
    自定义指标流水线
		client ——> API Service ——> Promethus Adapater ——> (PromQL)promethus Server  ——> exporters/Instrumentation
		custom.metrics.k8s.io/v1beta1
		excernal.k8s.io/v1beta1
		
https://APISERVICE:PORT/apis/custom.metrics.k8s.io/v1beta1/namespace/defauilt/pods/mypod
	
	在 kube-aggregator 上添加反代，代理到 Promethus Adapater，Promethus Adapater 再发送给 promethus Server
	
	
	资源类型
		用于再 API Service 上注册反代的端点：APIService
			
			kube-aggregator 和 Promethus Adapater 要使用加密通信
			方法两种
			1 Promethus Adapater 的证书，是 kubernetes Master CA 签名
			2 使用 --kubelet-insecure-tls 禁用证书验证
			kube-aggregator 会通过 Kubernetes 的内部 DNS 名称（如 prometheus-adapter.monitoring.svc）访问 Adapter。
			默认走的是 HTTPS，所以你要么配置 TLS，要么开启 insecureSkipTLSVerify。
			
			
	监控 kubernetes 集群：
		监控系统：
			Promethus Service
			Exporters/Instrumentation
			Push Gateway
			Alert Manager
			Grafana
			Blackbox Exporter
			Node Exporter
			
			Kube-State-Metrics:
				kubernetes Exporter 
					除了前面的role之外还需要很多监控：
						PVC
						PV 
						Delployment
						StatefulSet
```

![image-20250413172032424](kubernetes/image-20250413172032424.png)

```
┌────────────────────┐
│   Horizontal Pod   │
│    Autoscaler (HPA)│
└────────┬───────────┘
         │ 访问自定义指标API（如：custom.metrics.k8s.io）
         ▼
┌────────────────────┐
│  kube-apiserver    │
│（内置 kube-aggregator）│
└────────┬───────────┘
         │ 匹配 APIService (custom.metrics.k8s.io)
         ▼
┌────────────────────────┐
│   Prometheus Adapter   │  ◄────┐
│(扩展API服务 + 注册APIService)│   │
└────────┬───────────────┘       │
         │ 查询 Prometheus       │
         ▼                      │
  ┌──────────────────────┐     │
  │     Prometheus       │◄────┘
  └──────────────────────┘
```

```powershell
常见的部署方法
部署 promethus 和 Promethus Adapater 
	资源配置清单
	helm
	Promethus Operator:kube-promethus
```



#### 范例：helm 部署 Promethus

需要先部署OpenEBS和ingress，metallb

```bash
#添加仓库
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
#更新仓库
helm repo update
```

```yaml
#root@master01:~/k8s-prom/helm# cat prom-values.yaml 
server:
  name: server

  # 限制部署运行2.x系列的Prometheus，3.x系列的版本同Metrics-APP示例存在兼容性问题；
  # 现实应用中，可以不用顾及该限制；
  image:
    repository: quay.io/prometheus/prometheus
    # if not set appVersion field from Chart.yaml is used
    tag: "v2.55.1"
    pullPolicy: IfNotPresent

  # List of flags to override default parameters, e.g:
  # - --enable-feature=agent
  # - --storage.agent.retention.max-time=30m
  # - --config.file=/etc/config/prometheus.yml
  defaultFlagsOverride: []

  extraFlags:
    - web.enable-lifecycle
    # 开启 Prometheus 的热加载配置接口，比如 /-/reload 接口可以触发配置重载。
    ## web.enable-admin-api flag controls access to the administrative HTTP API which includes functionality such as
    ## deleting time series. This is disabled by default.
    # - web.enable-admin-api
    ##
    ## storage.tsdb.no-lockfile flag controls BD locking
    # - storage.tsdb.no-lockfile
    ##
    ## storage.tsdb.wal-compression flag enables compression of the write-ahead log (WAL)
    # - storage.tsdb.wal-compression

  ## Path to a configuration file on prometheus server container FS
  configPath: /etc/config/prometheus.yml

  global:
    # scrape_interval: 1m
    scrape_interval: 15s
    scrape_timeout: 10s
    # evaluation_interval: 1m
    evaluation_interval: 15s
    #配置全局抓取周期为 15 秒，抓取超时时间为 10 秒，规则评估周期也为 15 秒。
  ## https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote_write
  ##
  remoteWrite: []
  ## https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote_read
  ##
  remoteRead: []

  ingress:
    enabled: true
    ingressClassName: nginx
    annotations: {}
      #kubernetes.io/ingress.class: nginx
      #kubernetes.io/tls-acme: 'true'

    hosts:
      - prometheus.kang.com
      #启用了 ingress，通过 prometheus.kang.com 这个域名来访问 Prometheus UI。
    path: /
    pathType: Prefix

  persistentVolume:
    ## If true, Prometheus server will create/use a Persistent Volume Claim
    ## If false, use emptyDir  
    enabled: true
    accessModes:
      - ReadWriteOnce
    mountPath: /data
    size: 8Gi
    storageClass: "openebs-hostpath"
    #使用了持久化存储，采用的是 openebs-hostpath 存储类，数据保存位置为 /data，容量 8GB。

  emptyDir:
    sizeLimit: ""

  ## 若副本数量多于1个，请启用下面的StatefulSet
  replicaCount: 1

  statefulSet:
    ## 设置为“true”，才可将server的副本数量设置为1个以上
    enabled: false

    podManagementPolicy: OrderedReady

    ## Alertmanager headless service to use for the statefulset
    ##
    headless:
      servicePort: 9090
      ## Enable gRPC port on service to allow auto discovery with thanos-querier
      gRPC:
        enabled: false
        servicePort: 10901
        # nodePort: 10901

    pvcDeleteOnStsDelete: false
    pvcDeleteOnStsScale: false

  service:
    ## If false, no Service will be created for the Prometheus server
    ##
    enabled: true

    externalIPs: []
    servicePort: 9090
    type: ClusterIP


  ## Prometheus data retention period (default if not specified is 15 days)
  ##
  retention: "15d"
  #配置数据保存时间为 15 天。
  ## Prometheus' data retention size. Supported units: B, KB, MB, GB, TB, PB, EB.
  ##
  retentionSize: ""

## Prometheus server ConfigMap entries for rule files (allow prometheus labels interpolation)
ruleFiles: {}

## Prometheus server ConfigMap entries for scrape_config_files
## (allows scrape configs defined in additional files)
##
scrapeConfigFiles: []

## Prometheus server ConfigMap entries
##
serverFiles:
  ## Alerts configuration
  ## Ref: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
  alerting_rules.yml: {}
  # groups:
  #   - name: Instances
  #     rules:
  #       - alert: InstanceDown
  #         expr: up == 0
  #         for: 5m
  #         labels:
  #           severity: page
  #         annotations:
  #           description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes.'
  #           summary: 'Instance {{ $labels.instance }} down'


# adds additional scrape configs to prometheus.yml，其值必须为“字符”型数据，
# 因此，需要在“extraScrapeConfigs:”添加一个“|”
# example adds prometheus-blackbox-exporter scrape config
extraScrapeConfigs: ""
#这个字段可以用于添加额外的 scrape_configs，比如抓取黑盒探针、外部服务等
  # - job_name: 'prometheus-blackbox-exporter'
  #   metrics_path: /probe
  #   params:
  #     module: [http_2xx]
  #   static_configs:
  #     - targets:
  #       - https://example.com
  #   relabel_configs:
  #     - source_labels: [__address__]
  #       target_label: __param_target
  #     - source_labels: [__param_target]
  #       target_label: instance
  #     - target_label: __address__
  #       replacement: prometheus-blackbox-exporter:9115

alertmanager:
  enabled: true
#启用 alertmanager，用于告警通知系统（例如发邮件、Slack、企业微信等）。

  persistence:
    size: 2Gi
    storageClass: "openebs-hostpath"
    accessModes:
      - ReadWriteOnce

kube-state-metrics:
  enabled: true
#启用 kube-state-metrics，用来采集 K8s 对象（如 Deployment、Pod、PVC 等）的状态信息。

prometheus-node-exporter:
  enabled: true
#启用 node-exporter，用于采集节点的系统指标（CPU、内存、磁盘、网络等）。

prometheus-pushgateway:
  enabled: true
#启用 PushGateway，适用于短生命周期任务（如批处理）主动 Push 指标到 Prometheus。

  # Optional service annotations
  serviceAnnotations:
    prometheus.io/probe: pushgateway
```

```bash
#部署 prometheus
helm install prometheus prometheus-community/prometheus --namespace monitoring --values prom-values.yaml --create-namespace
```

**创建pod 抓取pod指标**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: metrics-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: metrics-app
      controller: metrics-app
  template:
    metadata:
      labels:
        app: metrics-app
        controller: metrics-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "80"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - image: ikubernetes/metrics-app
        name: metrics-app
        ports:
        - name: web
          containerPort: 80
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "256Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-app
spec:
  type: NodePort
  ports:
  - name: web
    port: 80
    targetPort: 80
  selector:
    app: metrics-app
    controller: metrics-app
```

```yaml
#解释
template:
  metadata:
    labels:
      app: metrics-app
      controller: metrics-app
    annotations:
      prometheus.io/scrape: "true"
      prometheus.io/port: "80"
      prometheus.io/path: "/metrics"
```

`labels`：用于服务发现和选择器匹配。

`annotations`：给 Prometheus 的 scrape 配置用，意思是：

- `scrape: "true"` → 启用抓取
- `port: "80"` → 抓取的端口
- `path: "/metrics"` → 抓取的路径



```bash
#部署前确定pod是支持指标抓取的
kubectl apply -f metrics-example-app.yaml 
```

#### 部署prometheus Adpater

```yaml
# Url to access prometheus
prometheus:
  # Value is templated
  url: http://prometheus-server.monitoring.svc
  port: 9090
  path: ""

replicas: 1

rules:
  default: true

  # Enabling this option will cause custom metrics to be served at /apis/custom.metrics.k8s.io/v1beta1.
  custom: []
    # - seriesQuery: '{__name__=~"^some_metric_count$"}'
    #   resources:
    #     template: <<.Resource>>
    #   name:
    #     matches: ""
    #     as: "my_custom_metric"
    #   metricsQuery: sum(<<.Series>>{<<.LabelMatchers>>}) by (<<.GroupBy>>)

  # Mounts a configMap with pre-generated rules for use. Overrides the
  # default, custom, external and resource entries
  existing:

  # Enabling this option will cause external metrics to be served at /apis/external.metrics.k8s.io/v1beta1. 
  external:
    # 基于应用上特定的http_requests_total指标生成http_requests_per_second指标的示例
    - seriesQuery: 'http_requests_total{kubernetes_namespace!="",kubernetes_pod_name!=""}'
      resources:
        overrides:
          kubernetes_namespace: {resource: "namespace"}
          kubernetes_pod_name: {resource: "pod"}
      name:
        matches: "^(.*)_total"
        as: "${1}_per_second"
      metricsQuery: rate(<<.Series>>{<<.LabelMatchers>>}[1m])

    # 有时，对于有些Java程序来说，基于内存资源用量进行自动扩缩容并不总是有效，因而可考虑根据JVM的平均使用量作为衡量指标；
    # 下面就是用于生成相关自定义指标的规则示例；
    - seriesQuery: '{__name__=~"jvm_memory_bytes_(used|max)",area="heap"}'
      seriesFilters:
      - is: ^jvm_memory_bytes_(used|max)$
      resources:
        overrides:
          namespace:
            resource: namespace
          service:
            resource : service
          pod:
            resource : pod
      name:
        matches: ^jvm_memory_bytes_(used|max)$
        as: "jvm_used_percent_housing"
      metricsQuery: ((sum((jvm_memory_used_bytes{area="heap", <<.LabelMatchers>>}))by(<<.GroupBy>>)*100/sum((jvm_memory_max_bytes{area="heap", <<.LabelMatchers>>}))by(<<.GroupBy>>)))/1000

  # Enabling this option will cause resource metrics to be served at /apis/metrics.k8s.io/v1beta1
  resource:
    cpu:
      containerQuery: |
        sum by (<<.GroupBy>>) (
          rate(container_cpu_usage_seconds_total{container!="",<<.LabelMatchers>>}[3m])
        )
      nodeQuery: |
        sum  by (<<.GroupBy>>) (
          rate(node_cpu_seconds_total{mode!="idle",mode!="iowait",mode!="steal",<<.LabelMatchers>>}[3m])
        )
      resources:
        overrides:
          node:
            resource: node
          namespace:
            resource: namespace
          pod:
            resource: pod
      containerLabel: container
    memory:
      containerQuery: |
        sum by (<<.GroupBy>>) (
          avg_over_time(container_memory_working_set_bytes{container!="",<<.LabelMatchers>>}[3m])
        )
      nodeQuery: |
        sum by (<<.GroupBy>>) (
          avg_over_time(node_memory_MemTotal_bytes{<<.LabelMatchers>>}[3m])
          -
          avg_over_time(node_memory_MemAvailable_bytes{<<.LabelMatchers>>}[3m])
        )
      resources:
        overrides:
          node:
            resource: node
          namespace:
            resource: namespace
          pod:
            resource: pod
      containerLabel: container
    window: 3m
```

```bash
helm install prometheus-adapter prometheus-community/prometheus-adapter --values prom-adapter-values.yaml --namespace monitoring
```

**测试**

待Prometheus和Prometheus Adapter的相关Pod均就绪后，获取针对现存系统环境由规则生成的自定义指标信息。

```bash
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```

部署示例应用metrcis app，它附带有“http_requests_total”指标。

```bash
kubectl apply -f https://raw.githubusercontent.com/iKubernetes/k8s-prom/master/prometheus-adpater/example-metrics/metrics-example-app.yaml
```

待Metrics App的Pod就绪后，等待Prometheus Server的几个指标抓取周期，即可尝试获取由规则生成自定义指标http_requests_per_second。

```bash
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second | jq .
```

### HPA

####  范例：基于 prometheus Adpater实现自动扩缩容

```bash
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2
metadata:
  name: metrics-app-hpa  # HorizontalPodAutoscaler 的名称
spec:
  # scaleTargetRef：指定要自动扩缩容的目标资源（这里是 Deployment）
  scaleTargetRef:
    apiVersion: apps/v1  # 目标资源的 API 版本
    kind: Deployment     # 目标资源的类型，表示这是一个 Deployment
    name: metrics-app    # 目标资源的名称，表示我们要操作名为 'metrics-app' 的 Deployment
  minReplicas: 2  # 最小副本数，当负载较低时，Pod 数量将不会小于 2
  maxReplicas: 10 # 最大副本数，负载增加时，Pod 数量不会超过 10
  metrics:
  - type: Pods  # 这里使用的是基于 Pod 的自定义指标来调整副本数
    pods:
      metric:
        name: http_requests_per_second  # 自定义指标的名称，表示每秒 HTTP 请求数
      target:
        type: AverageValue  # 指标的目标类型，表示平均值
        averageValue: 5     # 如果每个 Pod 的 http_requests_per_second 指标的平均值超过 5，则扩容
  behavior:
    # 扩缩容时的行为设置
    scaleDown:
      stabilizationWindowSeconds: 120  # 缩容时的稳定时间窗口，表示缩容操作将在 120 秒内稳定执行，避免频繁缩容
```

```bash
kubectl apply -f metrics-app-hpa.yaml

#kubectl get hpa
NAME              REFERENCE                TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
metrics-app-hpa   Deployment/metrics-app   <unknown>/5   2         10        0          13s
```



#### 范例：部署blackbox-exporter

Blackbox Exporter

部署Blackbox Exporter，启用黑盒监控；如下命令指定了Release名称为“prometheus-blackbox-exporter”，该名称作为Service名称，将被下面示例中的配置引用。

```bash
helm install --name-template prometheus-blackbox-exporter  prometheus-community/prometheus-blackbox-exporter \
          -f blackbox-exporter-values.yaml -n monitoring
```

随后，需要在Prometheus的values文件中，启用额外的Scrape Job，以关联Blackbox Exporter。一个示例配置如下，注意该字段的值必须为字符型数据，因此需要在字段后添加“|”。

```yaml
extraScrapeConfigs: |
  - job_name: 'prometheus-blackbox-exporter'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://www.magedu.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: prometheus-blackbox-exporter:9115
```

最后，更新Prometheus的Release，确保配置生效。

blackbox-exporter-values.yaml:

```yaml
config:
  modules:
    http_2xx:
      prober: http
      timeout: 5s
      http:
        valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
        follow_redirects: true
        preferred_ip_protocol: "ip4"


extraConfigmapMounts: []
  # - name: certs-configmap
  #   mountPath: /etc/secrets/ssl/
  #   subPath: certificates.crt # (optional)
  #   configMap: certs-configmap
  #   readOnly: true
  #   defaultMode: 420

service:
  type: ClusterIP
  port: 9115

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: probe.kang.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local


extraArgs: []
  # - --history.limit=1000

replicas: 1

# Extra manifests to deploy as an array
extraManifests: []
  # - apiVersion: v1
  #   kind: ConfigMap
  #   metadata:
  #   labels:
  #     name: prometheus-extra
  #   data:
  #     extra-data: "value"

configReloader:
  enabled: false
  containerPort: 8080
  config:
    logFormat: logfmt
    logLevel: info
    watchInterval: 1m
```



#### 范例：部署grafana

```bash
#添加仓库
helm repo add grafana https://grafana.github.io/helm-charts

#部署
helm install grafana grafana/grafana -n monitoring --create-namespace

#创建ingress
kubectl create ingress grafana \
  --rule=grafana.kang.com/=grafana:80 \
  --class=nginx \
  -n monitoring
```

```bash
#helm部署的grafana的密码
kubectl get secret grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```



### 部署promethus operator

```
https://github.com/prometheus-operator/kube-prometheus
```

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus/

kubectl apply --server-side -f manifests/setup

kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
	
kubectl apply -f manifests/
```

配置ingress查看grafana

```yaml
apiVersion: v1
items:
- apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    creationTimestamp: "2025-04-22T06:08:25Z"
    generation: 2
    name: grafana
    namespace: monitoring
    resourceVersion: "1147712"
    uid: 6c9c64fd-5340-42b2-8b08-f6b1c855920d
  spec:
    ingressClassName: nginx
    rules:
    - host: grafana.kang.com
      http:
        paths:
        - backend:
            service:
              name: grafana
              port:
                number: 3000
          path: /
          pathType: Prefix
  status:
    loadBalancer:
      ingress:
      - ip: 10.0.0.100
kind: List
metadata:
  resourceVersion: ""
```



## 调度器和调度流程-kube-scheduler

```powershell
kube-scheduler 调度过程

节点预选
	根据预选策略（Predicates）过滤掉所有不符合运行该 pod 的节点，只保留可以运行该 pod 的节点
节点优选
	根据优选策略（Priorities）对可以运行该 pod 的节点进行打分，并根据得分的节点进行逆序排序
选定
	选定，并把该 pod 绑定到得分最高的 node 上
	

可调度节点
	首先要满足Pod的资源需求
	而后要满足同其它Pod间的特殊关系限制
	再次要满足同Node之间的限制条件
	最后确保整个集群资源得到合理利用
```

![image-20250420125544441](kubernetes/image-20250420125544441.png)

#### 预选算法Predicates

```powershell
Predicates的功能，大体相当于节点过滤器(Filter)
	它基于调度策略，从集群的所有节点中，过滤出符合条件的节点
	执行具体过滤操作的是一组预选插件plugin)
经典调度器的预选算法分类
	存储约束条件
		名称中带有Disk或Volume的算法
	Pod间的特殊关系限制
		MatchInterPodAffinity
	Pod同Node的限制约束
		名称中带有Node的算法
		General Predicates
	Pod的散置性要求
		CheckServiceAffinity
		EvenPodsSpread
```

![image-20250420130707439](kubernetes/image-20250420130707439.png)

```powershell
几个重要的Predicates说明
	PodFitsHostPorts
		检查Pod的各Containers中声明的Ports是否已经被节点上现有的Pod所占用
	MatchNodeSelector
		检查Pod的spec.affinity.nodeAffinity和spec.nodeSelector的定义是否同节点的标签相匹配
	PodFitsResources
		检查Pod的资源需求是否能被节点上的可用资源量所满足
	PodToleratesNodeTaints
		检查Pod是否能够容忍节点土的污点
	MaxCSIVolumeCount
		检查Pod依赖的由某CSI插件提供的PVC，是否超出了节点的单机上限
	MatchInterPodAffinity
		检查Pod间的亲和和反亲和定义是否得到满足
	EvenPodsSpread
		为一组Pod设定在指定TopologyKey上的散置要求，即打散一组Pod至不同的拓扑位置
```

#### 优选算法Priorities

```powershell
Priorities的功能，大体相当于计分器
	执行具体打分操作的是一组优选算法
	每个节点的总得分，则由各算法为其评分的各项分值之和
经典优选算法的分类
	节点资源分配倾向
		BalancedResourceAllocation
		LeastRequestedPriority/MostRequestedPriority
		Resourcel imitsPriority
		RequestedToCapacityRatioPriority
	Pod散置
		SelectorSpreadPriority、EvenPodsSpreadPriority、ServiceSpreadingPriority
	Node亲和与反亲和
		NodeAffinityPriority、NodePreferAvoidPodsPriority
		TaintTolerationPriority
		ImageLocalityPriority
	Pod间的亲和与反亲和
		InterPodAffinityPriority
```

![image-20250420153718286](C:/Users/zhaok/AppData/Roaming/Typora/typora-user-images/image-20250420153718286.png)



#### 调度框架

**框架工作流程**

调度框架上定义了一些扩展点，调度器插件完成注册后可以一个或多个扩展点上被调用

每次调度一个pod的尝试都可划分为两个阶段：**调度周期**和**绑定周期**

![image-20250420161158722](kubernetes/image-20250420161158722.png)



```bash
#默认的两个priorityclasses
root@master01:/etc/kubernetes/manifests# kubectl get priorityclasses.scheduling.k8s.io 
NAME                      VALUE        GLOBAL-DEFAULT   AGE     PREEMPTIONPOLICY
system-cluster-critical   2000000000   false            2d22h   PreemptLowerPriority
system-node-critical      2000001000   false            2d22h   PreemptLowerPriority

```



## 资源约束和 Pod Qos

```powershell
资源需求和资源限制
	资源需求(requests)
		定义需要系统预留给该容器使用的资源最小可用值
		容器运行时可能用不到这些额度的资源，但用到时必须确保有相应数量的资源可用
		资源需求的定义会影响调度器的决策
	资源限制(limits)
		定义该容器可以申请使用的资源最大可用值，超出该额度的资源使用请求将被拒绝
		该限制需要大于等于requests的值，但系统在其某项资源紧张时，会从容器那里回收其使用的超出其requests值的那部分
		
requests和limits定义在容器级别，主要围绕cpu、memory、hugepages和ephemeral-storage四种资源
	spec.containers.resources.limits.
		cpu、memory、hugepages-<size>和ephemeral-storage
	spec.containers.resources.requests.
		cpu、memory、hugepages-<size>和ephemeral-storage
		
Extended resources
	所有那些不属于kubernetes.io域的资源，即为扩展资源，例如"nvidia.com/gpu"
	又存在节点级别和集群级别两个不同级别的扩展资源
```

```powershell
 
```

![image-20250420163052563](kubernetes/image-20250420163052563.png)



## Node Affnity——Node亲和调度

亲和调度：

- Pod对节点亲和：判定标准取决于自身属性

  - nodeName

  - nodeSelector

  - `pods.spec.affinity.nodeAffinity`

    软亲和
    	`preferredDuringSchedulingIgnoredDuringExecution		<[]PreferredSchedulingTerm>`
    		`preference`:条件
    		`weight`:权重
    硬亲和
    	`requiredDuringSchedulingIgnoredDuringExecution		<NodeSelector>`



- Pod间的亲和：判定标准取决于节点上已存在的pod的属性

  - `pods.spec.affinity.podAffinity`

    软亲和
    	`preferredDuringSchedulingIgnoredDuringExecution	<[]WeightedPodAffinityTerm>`
    		`podAffinityTerm	<PodAffinityTerm> -required- `：条件-pod的筛选条件
    		`weight`：权重-得分
    硬亲和
    	`requiredDuringSchedulingIgnoredDuringExecution	<[]PodAffinityTerm>`

    ​		`labelSelector	<LabelSelector>`

    ​		`topologyKey	<string> -required-`

    - labelSelector:
        matchLabels:
          app: myapp
        topologyKey: kubernetes.io/hostname

    字段说明
    labelSelector	匹配目标 Pod 的标签（也就是你想靠近的 Pod）
    topologyKey	表示匹配的粒度，比如按节点 (kubernetes.io/hostname)、机架、区域等

    | 类型                                              | 字段 | 是否必须满足 | 场景                 |
    | ------------------------------------------------- | ---- | ------------ | -------------------- |
    | `requiredDuringSchedulingIgnoredDuringExecution`  | 强制 | 必须满足     | 精确调度需求         |
    | `preferredDuringSchedulingIgnoredDuringExecution` | 优先 | 尽量满足     | 柔性调度，实用性更强 |

    ​		**pod拓补分散约束:**

    ​			定义跨拓补域分配工作负载的策略

    ​			常用于在不显著增加成本的情况下增强抵御紫铜故障的能力

    ​				可基本策略确保在可用区中仍存在可用pod

    ​		**拓扑域**

​							在kubernetes中，拓扑域是指按节点标签定义和分组的节点集合

​							例如，区域、可用区、机架、主机名都可以作为将节点划分成不同拓扑域的依据

​								Kubernetes使用Topology Key(拓扑键)来定义将节点划分至不同拓扑域的标准

![image-20250421164957180](kubernetes/image-20250421164957180.png)

Pod的分散约束：

`pods.spec.topologySpreadConstraints`

散置偏差（Skew）

Pod打散调度至多个拓扑域后，拥有Pod数量做多的拓扑域，与拥有Pod数量最少的拓扑域之间的差值

![image-20250421172744276](kubernetes/image-20250421172744276.png)

Pod分散约束中允许的最大偏差及妥协策略

- Pod分散约束策略基于允许的最大偏差(maxSkew)来定义Pod在拓扑域中的分散逻辑
- 无法满足最大偏差约束时，whenUnsatishable则用于定义妥协策略
  - DoNotSchedule (默认):不予调度，Pod将处于Pending状态，实现的是硬限制
  - ScheduleAnyway:根据每个节点的skew值打分排序后进行调度，因而实现的是软限制

`kubectl explain pods.spec.topologySpreadConstraints.`：跨节点进行冗余

​	`whenUnsatisfiable	<string> -required-`

​			`DoNotSchedule`：条件不符合，一定不能调度过去

​			`ScheduleAnyway`：无论如何都要调度过去

​		`maxSkew	<integer> -required-`：允许的最大偏差多大

​		`minDomains	<integer>`：允许的最小偏差

​		`labelSelector	<LabelSelector>`：对那一组pod进行调度



- Pod间的反亲和
  	`pods.spec.affinity.podAntiAffinity`

  kubectl explain pods.spec.affinity.nodeAffinity
  	

Pod对节点亲和

- 基于Pod和Node关系的调度策略，称为Node亲和调度

何时需要用到Afhnity调度?

- Pod的运行依赖于特殊硬件，例如SSD或GPU，但这些设备仅部分节点具备具有高计算量要求的Pod，也可能需要限制运行在特定的节点



#### 硬亲和和软亲和

```powershell
Pod与Node的亲和关系存在两种约束强度
	硬亲和:必须满足的亲和约束，约束评估仅发生在调度期间
    	requiredDuringSchedulinglgnoredDuringExecution
    		必须调度至满足条件的节点
    软亲和:有倾向性的亲和约束，不同的约束条件存在不同的权重，约束评估同样仅发生在调度期间
    	preferredDuringSchedulingIgnoredDuringExecution
    		优先调度至更为满足条件(权重更高)的节点
```

```yaml
#范例：
apiVersion: v1
kind: Pod
metadata:
  name: affinity-example
spec:
  affinity:
    nodeAffinity:
      # 软亲和性：优先调度到 SSD 节点
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          preference:
            matchExpressions:
              - key: "disktype"
                operator: "In"
                values: ["ssd"]
      # 硬亲和性：必须调度到 AMD64 架构节点
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "kubernetes.io/arch"
                operator: "In"
                values: ["amd64"]
  containers:
    - name: nginx
      image: nginx
```

```powershell
节点上的预定义标签
	kubernetes.io/hostname:节点名称
	kubernetes.io/os:操作系统，v1.14版本起启用
	kubernetes.io/arch:平台架构类型，v1.14版本起启用
匹配节点表达式(matchExpressions)支持操作符
	In:指定的label的值存在于给定列表中
	NotIn:指定的label的值未存在于给定列表中
	Gt:指定的label的值大于给定值
	Lt:指定的label的值小于给定值
	Exists:指定label存在于节点上
	DoesNotExist:指定label末存在于节点上
```

<img src="kubernetes/image-20250421163414382.png" alt="image-20250421163414382" style="zoom:50%;" />

**节点软亲和**

![image-20250421163532061](kubernetes/image-20250421163532061.png)



## Taints 与 Tolerations 污点和容忍度

Taints：节点上的属性

Tolerations：pod属性

软限制：不能容忍这个污点，但是没有其他节点可调度，也可以调度过来

硬限制：不能容忍node上的污点就一定不能调度过来

**Node Taints 和 Tolerations调度**

**Node Taints**

- 基于Pod和Node关系的调度策略
  - Node Afinity，用于定义Pod对Node的倾向性，即基于Node的特定属性(标签或字段)来吸引特定的Pod
  - 而Node Taints(污点)则产生的是反向作用力，它用于让Node来排斥特定的Pod，仅那些能够容忍Node Taints的Pod才能运行于该节点上
  - Pod上定义的用于容忍Node Taints的属性，称为Pod容忍度(Toleration)
  - Taint和Toleration相互配合，可以用来阻止调度器将Pod分配到不适用的节点上
- Node Taints
  - 节点属性，定义在spec.taints字段上
  - 也可由“kubect taint”命令进行添加或移除等管理操作
    - `kubectl taint nodes NAME KEY_1=VAL_1:TAINT_EFFECT_1 .. KEY_N=VAL_N:TAINT_EFFECT_N [options]`
  - 污点效用:指示该污点不能得到容忍时，将如何影响调度
    - PreferNoSchedule:不能容忍该污点的Pod，要尽量避免调度至该节点
    - NoSchedule:不能容忍则不许调度至该节点，但仅在调度决策执行期间产生影响，不影响已经运行于该节点的Pod
    - NoExecute:不能容忍则不许调度至该节点，且将会对已经运行于该节点的Pod产生影响
      - 不能容忍该污点的Pod将立即被驱逐
      - 若Pod能够容忍该污点，日Pod的容忍度上未定义tolerationSeconds，则Pod可以一直运行于该节点
      - 若Pod能够容忍该污点，月Pod的容忍度上定义了tolerationSeconds，则Pod运行指定的时长后即会被驱逐

**Tolerations**

- 管理Pod容忍度
  - Pod属性，定义在字段上 `spec.tolerations`字段上，列表值
  - 每个列表项出key、operator、value、toleratinSeconds和effect几个字段组成
    - key:容忍度键，即容忍的污点键
    - operator:操作符，仅支持“Equal”和“Exists”两个
    - value:键值
    - effect:容忍的污点效用，可用值与Node Taint相同
    - tolerationSeconds:NoExecute效用下，当前Pod可容忍某污点时，在驱逐前容许运行的时长
- 如何评估容忍度是否匹配污点?
  - 一个容忍度“匹配”到一个污点，是指二者之间具有同样的key和effect，并且满足如下两个条件之一
    - operator是Exists
    - operator是Equal，且二者的value相同
  - 特殊情形
    - 若某个容忍度的key为空，且operator为Exists，表示其可以容忍任意污点
    - 若effect为空，则可以匹配所有键名相同的污点

**容忍度与污点的匹配机制**

- 节点上可定义多个Taints，Pod上也可定义多个Toleratons，针对一个节点，调度器调度决策过程如下
  - 遍历该节点的所有污点，并过滤掉容忍度可匹配到的污点
  - 余下未被过滤掉的污点，则由污点的cffect来判定其满足状态
    - 若存在至少一个效用为NoSchedule的污点，则Pod不会被调度至该节点
    - 若不存在效用为NoSchedule的污点，但至少存在一个效用为PrefcrNoSchedule的污点，则调度器会尝试避免将Pod调度至该节点
    - 若存在至少一个效用为NoFxecute的污点，则Pod不会被调度至该节点;若Pod已经运行于该节点，则Pod还将被驱逐
- 何时需要使用基于Taint和Toleration的调度?
  - 存在需要保留某些节点，避免调度器自行调度Pod至这些节点，或者将Pod从某些节点驱逐
  - 保留给某些用户，或某特定应用的专用节点，例如专用于Ingress Controller的节点
  - 配置了特殊硬件，只期望将用到这些硬件的Pod调度至这些具有特殊硬件的节点

**基于污点的驱逐**

- 节点状态相关的某种条件(Node Conditons)为真时，节点控制器会自动给节点添加一个污点
  - node.kubernetes.io/not-ready:节点未就绪，即节点状况Ready的值为"False'
  - node.kubernetes.io/unreachable:节点控制器访问不到节点，即节点状况Ready的值为"Unknown"
  - node.kubernetes.io/memory-pressure:甘点内存资源紧张node.kubernetes.io/disk-pressure:节点磁盘空间资源紧张
  - node.kubernetes.io/pid-pressure:节点PID资源紧张node.kubernetes.io/network-unavailable:古点双终不可
  - node.kubernetes.io/unschedulable:节点不可用于调度
  - node.cloudprovider.kubernetes.io/uninitialized: cloud provider尚未进行初始化
- 控制平面基于Node Controller自动创建 与节点状况对应的、效果为NoSchedule的污点
  - 调度器在进行调度时会检查节点上的污点，而非检查节点状况
  - 新建Pod时，可以通过添加相应的容忍度来忽略节点状况。



#### 范例：Master节点上的污点

```yaml
spec:
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
```

有些Pod可以自动添加运行至Master例如：

#### 1. **MetalLB speaker Pod 的 Tolerations**

这些 tolerations 允许 speaker Pod 调度到如下节点上：

```yaml
tolerations:
  # 容忍 master 节点的 NoSchedule 污点，允许部署到 master 节点上
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
    operator: Exists

  # 容忍 control-plane 节点的 NoSchedule 污点，允许部署到 control-plane 节点上
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
    operator: Exists

  # 容忍节点状态为 NotReady 的情况，不会被立即驱逐
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists

  # 容忍节点不可达的情况（例如断网），不会被立即驱逐
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists

  # 容忍节点磁盘压力过大，不影响调度
  - effect: NoSchedule
    key: node.kubernetes.io/disk-pressure
    operator: Exists

  # 容忍节点内存压力过大，不影响调度
  - effect: NoSchedule
    key: node.kubernetes.io/memory-pressure
    operator: Exists
```

> ✅ 作用：允许部署在 master/control-plane 节点，并容忍节点短暂故障和资源压力。

------

#### 🔷 2. **Cilium Agent Pod 的 Tolerations**

```yaml
tolerations:
  # 容忍 NotReady 节点（暂时失联）
  - operator: Exists
    effect: NoExecute
    key: node.kubernetes.io/not-ready

  # 容忍 Unreachable 节点（彻底失联）
  - operator: Exists
    effect: NoExecute
    key: node.kubernetes.io/unreachable

  # 容忍磁盘压力，允许调度
  - operator: Exists
    effect: NoSchedule
    key: node.kubernetes.io/disk-pressure

  # 容忍内存压力，允许调度
  - operator: Exists
    effect: NoSchedule
    key: node.kubernetes.io/memory-pressure

  # 容忍 PID 压力（进程数量限制）
  - operator: Exists
    effect: NoSchedule
    key: node.kubernetes.io/pid-pressure
```

> ✅ 作用：容忍多种系统压力和节点不可达，确保网络组件稳定运行。

------

#### 🔷 3. **CoreDNS Pod 的 Tolerations**

```yaml
tolerations:
  # 容忍只调度 CriticalAddonsOnly 的节点（表示是核心组件）
  - key: CriticalAddonsOnly
    operator: Exists

  # 容忍 control-plane 污点，允许部署在控制节点上
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
    operator: Exists

  # 节点 not-ready 时最多容忍 300 秒，再被驱逐
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300

  # 节点 unreachable 时最多容忍 300 秒，再被驱逐
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
```

> ✅ 作用：

- `CriticalAddonsOnly`：表示这个 Pod 属于关键组件。
- 容忍 control-plane 节点、不可达状态 300 秒。

#### 使用方法

要将 Kubernetes 控制节点设置为不可调度，可以使用以下命令：

```bash
kubectl taint nodes <node-name> node-role.kubernetes.io/master:NoSchedule
```

其中 `<node-name>` 是你要设置为不可调度的节点名称，`node-role.kubernetes.io/master` 是节点的标签，`NoSchedule` 是污点类型，表示该节点不能调度任何新的 pod。

如果你要让该节点重新变得可调度，可以使用：

```bash
kubectl taint nodes <node-name> node-role.kubernetes.io/master:NoSchedule-
```

这样会移除节点上的污点，允许新的 pod 被调度到该节点。





## 网络插件CNI

```powershell
三个网络：
	节点网络：管理员自行管理
		每个节点都拥有该地址范围内的一个地址，配置在节点的某个接口上
	Service网络：也称为集群网络，由集群负责管理，虚拟网络
		10.96.0.0/12
		ClusterIP
			存在于iptables/nftables/ipvs规则
     pod网络：CNI接口，委托第三方插件实现
     	kubenet ——> flannel (coreos)
     	CNI ——> 
     		Calico
     		Cilium
     		...
将Pod接入网络之前要完成任务
	1、创建专用的虚拟网络:
		依托内核功能来实现
		Kubernetes要求:所有的Pod要位于同一个网络平面，Pod间可以直接通信
			隧道网络，overlay
			底层网络：underlay
				MACVLAN，IPVLAN
	2、给Pod提供一个网络接口
		veth pair:虚拟以太网网卡对
		SR-IOV
	3、将Pod通过其网络接口接入到虚拟网络
		veth pair
			一段注入到Pod内作为网络接口使用
			另一端驻留于节点：
				创建一个虚拟网桥:驻留于节点这一端关联至该网桥即可，二层连通
				无需虚拟网桥，而是直接驻留于节点内核上，三层连通
	4、给Pod的网络接口分配IP地址

简单来说，目前的CNI规范主要由NetPlugin(网络插件)和IPAM两个插件API组成
	网络插件也称Main插件，负责创建/删除网络以及向网络添加/删除容器
		它专注于连通容器与容器之间以及容器与宿主机之间的通信，同容器相关网络设备通常都由该类插件所创建，例如bridge、ipvlan、macvlan、loopback、ptp、veth以及vlan等虚拟设备
	IPAM的全称“IP Address Management"，该类插件负责创建/删除地址池以及分配/回收容器的IP地址
		目前，该类型插件的实现主要有host-local和dhcp两个，前一个基于预置的地址范围进行地址分配，而后一个通过dhcp协议获取地址;
		
	虚拟网络
    	隧道模型：Overlay
    		vxlan ——> flannel , calico , cilium
    		ipip ——> calico
    		隧道开销：vxlan和ipip的开销不同
		承载模型：Underlay
			IPVLAN/MACVLAN: 二层解决方案
				MACVLAN:
					把pod的MAC地址通过MACVLAN虚拟到内核的网卡上
					需要打开混杂模式，有安全风险
				IPVLAN
					所有后端的pod共享物理网卡的MAC地址，IP不同，需要用到IPVLAN来区分到那个后端Pod
				使用VLAN的化，需要配置交换机VLAN， 
			Native Routing: 三层解决方案
				flannel:查询库生成路由表
				calico:BGP协议学习生成
				cilium:BGP协议学习生成,但cilium自身并不支持BGP,需要额外再部署一个网络插件（kube-router）lai'ti
				把内核作为路由器
				节点间不能跨路由器

网络插件：
	Pod网络:连通Pod，包括跨节点的Pod
	链路加密:VPN
		IPSec
		WireGuard
	网络策略（NetworkPolicy）
		控制特定的Pod的（跨租户）通信
		反向：
			Egress
			Ingress

Pod网络地址：
	Flannel：10.244.0.0/16
		16bits: 10.244.0.0/24 ——> 10.244.255.0/24：256个，最多256个节点
		每个节点容纳的Pod数量：
			8bit：pod地址：
				10.244.0.0 ——> 10.244.0.255
					0000 0000：网络地址
					1111 1111：广播地址
					
					256-2：254个可用地址;默认110个Pod
	Calico： 192.168.0.0/16
		10bits: 2^10 = 1024,最多可以容纳1024个节点
		6bits: 2^6 -2 = 62,每个节点最多容纳62个Pod
```

### flannel

```powershell
使用“虚拟网桥和veth设备”的方式为Pod创建虚拟网络接口，通过可配置的“后端”定义Pod间的通信网络，支持基于VXLAN和UDP的Overlay网络，以及基于三层路由的Underlay网络
	虚拟网桥cni0
	隧道接口通常为flannel.1
在IP地址分配方面，它将预留的一个专用网络(默认为10.244.0.0/16)切分成多个子网后作为每个节点的podCIDR，而后由节点以IPAM插件host-local进行地址分配，并将子网分配信息保存于etcd之中

flanneld
	Flannel在每个主机上运行一个名为fanneld的二进制代理程序
	该程序负责从预留的网络中按照指定或默认的掩码长度为当前节点申请分配一个子网，并将网络配置、已分配的子网和辅助数据(例如主机的公网IP等)存储于Kubernetes API或etcd之中
Flannel使用称为后端(backend)的容器网络机制转发跨节点的Pod报文，它目前支持的主流backend如下
	vxlan
		使用Linux内核中的vxlan模块封装隧道报文，以叠加网络模型支持跨节点的Pod间互联互通
		额外支持直接路由(Direct Routing)模式，该模式下位于同二层网络内的节点之上的Pod间通信可通过路由模式直接发送，而跨网络的节点之上的Pod间通信仍要使用VXLAN隧道协议转发
		vxlan后端模式中，fanneld监听于8472/UDP发送封装的数据包;
	host-gw
		类似于VXLAN中的直接路由模式，但不支持跨网络的节点，因此这种方式强制要求各节点本身必须在同一个二层网络中，不太适用于较大的网络规模
		有着较好的转发性能，且易于设定，推荐对报文转发性能要求较高的场景使用
		
VXLAN 隧道模式
	对节点网络几乎没有限制
	存在开销
	
host-gw 路由模式
	要求所有网路必须位于同一网络平面
	没有网络开销，性能好
	
混合网络
	同一网络平面上的使用host-gw，跨网络通信的，使用VXLAN
	
#查看使用的网络模式
root@master01:~# kubectl get cm -n kube-flannel 
NAME               DATA   AGE
kube-flannel-cfg   2      17d
root@master01:~# kubectl get cm -n kube-flannel kube-flannel-cfg -o yaml 

#查看node使用的网段
root@master01:~# kubectl get nodes node1.kang.com -o yaml 
...
spec:
  podCIDR: 10.244.1.0/24
  podCIDRs:
  - 10.244.1.0/24
...

root@master01:~# kubectl get nodes node1.kang.com -o jsonpath={.spec.podCIDR}
10.244.1.0/24
```

![image-20250502135254219](kubernetes/image-20250502135254219.png)

![image-20250505143950342](kubernetes/image-20250505143950342.png)

![image-20250505144424297](kubernetes/image-20250505144424297.png)

###  Calico

```powershell
ProjectCalico
	三层的虚拟网络方案
	它把每个节点都当作虚拟路由器(vRouter)，把个节点上的Pod都当作是“节点路由器”后的一个终端设备并为其分配一个IP地址
	各节点路由器通过BGP(Border Gateway Protocol)协议学习生成路由规则从而实现不同节点上Pod间的互联互通

Calico在每一个计算节点利用Linux内核实现了一高效的vRouter(虚拟路由器)进行报文转发，而每个vRouter通过BGP协议负责把自身所属的节点上行的Pod资源的IP地址信息基于节点的agent程序网络内传播(Felix)直接由vRouter生成路由规则问整个Calico

网络模式
	纯隧道：
		ipip
		vxlan
	纯三层：
		bgp
	混合模式
		bgp + ipip
		bgp + vxlan
	
也支持eBGP机制（cilium）
```

```powershell
Calico的工作机制
	Calico把Kuberetes集群环境中的每个节点上的Pod所组成的网络视为一个自治系统，各节点也就是各自治系统的边界网关，它们彼此间通过BGP协议交换路由信息生成路由规则
	考虑到并非所有网络都能支持BGP，以及BGP路由模型要求所有节点必须要位于同一个二层网络，Calico还支持基于IPIP和VXLAN的叠加网络模型
	类似于Panpe!在VXLAN后端中启用DirectRoutine时的网络模型，Caico也支持混合使用路由和叠加网络模型BGP路由模型用于二层网络的高性能通信，IPI或VXLAN用于跨子网的节点间(Cross-Subnet)报文转发
```

![image-20250516140803957](kubernetes/image-20250516140803957.png)

![image-20250512144236942](kubernetes/image-20250512144236942.png)

```powershell
概括来说，Calico主要由Felix、Orchestrator Plugin、etcd、BIRD和BGP Router Reflector等组件组成
	Felix:Calico Agent，运行于每个节点，主要负责维护虚拟接口设备和路由信息	Orchestrator Plugin:编排系统(例如Kubernetes、OpenStack等)用于将Calico整合进行系统中的插件，例如Kubernetes的CNI
	etcd:持久存储Calico数据的存储管理系统
	BIRD:负责分发路由信息的BGP客户端
	BGPRoute Refector:BGP路由反射器，可选组件，用于较大规模的网络场景
```

![image-20250512144918501](kubernetes/image-20250512144918501.png)



```powershell
calico-node
	运行于集群中的每个节点，负责路由编程(Felix)和路由分发(BIRD)
	Felix:负责生成路由规则和iptables规则，前者用于完成Pod报文路由，后者用于支撑NetworkPolicy
	BIRD:读取并分发由同一节点上的Felix生成的路由规则，支持多种分发拓扑
calico-kube-controller
	负责监视Kubernetes对象中(包括NetworkPolicy、Pod、Namespace、ServiceAccount和Node等)会影响到路由的相关变更
	将变更带来的影响生成Calico配置，并保存于Calico Datastorer
```

![image-20250516140940624](kubernetes/image-20250516140940624.png)

```powershell
Typha
	各calico-node实例同Calico Datastore通信的中间层，由其负责将Calico Datastore中生成的更改信息分发给名calico-node，以减轻50个节点以上规模集群中的Calico Datastore的负载
	具有缓存功能，且能够通过删除重复事件，降低系统负载
Calico Datastore
	通用术语，是指存储的Calico配置、路由、策略及其它信息，它们通常表现为Calico CRD资源对象
	支持的CRD包括BGPConfiguration、BGPFilter、BGPPeer、BlockAffinity、CalicoNodeStatus、ClusterInformation、FelixConfiguration、GlobalNetworkPolicy、GlobalNetworkSet、HostEndpoint、IPAMBlock、IPAMConfig.IPAMHandle、IPPool、IPReservation、NetworkPolicy、NetworkSet和KubeControllersConfiguration等
	几个常用的CRD功能说明
		BGPConfiguration:全局BGP配置，用于设定AS(自治系统)编号、node mesh，以及用于通告ClusterIP的设置
		FelixConfiguration:Felix相关的低级别配置，包括iptables、MIU和路由协议等
		GlobalNetworkPolicy:全局网络策略，生效于整个集群级别;
		GlobalNetworkSet:全局网络集，是指可由GobalNetworkPolicy引用的外部网络IP列表或CIDR列表;
		IPPoo!:IP地址池及相关选项，包括要使用的路由协议(IPIP、VXLAN或Native);一个集群支持使用多个Pool;
```

```powershell
Calico数据存储模式：Kubernetes
在几乎所有情况下，都建议使用Kubernetes数据存储而非独立的ectd
	数据通过CRD存储于kube-apiserver
	对calico资源的访问可由Kubernetes RBAC控制
	但成百上千个Felix实例同时与kube-apiserver交互，会带来负责影响，因而必须要使用typha中间层
```

![image-20250516141314670](kubernetes/image-20250516141314670.png)

```powershell
Calico数据存储模式：etcd
将Calico数据存储于独立的etcd中
	可减轻kube-apiserver的压力，但也会引入不必要的复杂性和安全风险
	切不可直接使用Kubernetes etcd作为Calico的数据存储
```

![image-20250516141514062](kubernetes/image-20250516141514062.png)



```powershell
Calico下的Pod网络接口
Calico网络插件如何为Pod配置网络接口
	为每个Pod创建一组veth pair，一端注入Pod网络名称空间，另一端驻留在宿主机上
		Pod内的一端，通常名称格式为“etho@ifN”，其的N是驻留在宿主机上的另一端的ip link编号
		驻留宿主机的一端，名称格式为“caliXXXXXXXXXXX@ifN，其中的11位X是经由函数计算生成，而N则是注入到Pod网络名称空间中的对端的ip link编号
	Pod网络名称空间中，会生成独特的默认路由，将网关指向169.254.1.1
```

![image-20250516152857736](kubernetes/image-20250516152857736.png)

```powershell
	宿主机为每个cali接口都开启了ARP Proxy功能，从而让宿主机扮演网关设备，并以自己的MAC地址代为应答对端Pod中发来的所有ARP请求
		ARP Proxy的关键配置： /proc/sys/net/ipv4/conf/DEV/proxy_arp
	同一节点上的Pod间通信依赖于为每个Pod单独配置的路由规则
	不同节点上的Pod间通信，则由Calico的路由模式决定
```

![image-20250516152915338](kubernetes/image-20250516152915338.png)

![image-20250516142518162](kubernetes/image-20250516142518162.png)

```powershell
Calico支持多种路由模式
	Native：原生路由五隧道封装
	IP-in-IP：IPIP隧道模式，开销较小的隧道协议
	VXLAN：VXLAN隧道模式
```

![image-20250521152949800](kubernetes/image-20250521152949800.png)

![image-20250521153019644](kubernetes/image-20250521153019644.png)

![image-20250521195039847](kubernetes/image-20250521195039847.png)

![image-20250521195615899](kubernetes/image-20250521195615899.png)

![image-20250521195833917](kubernetes/image-20250521195833917.png)











## Velero备份

### 部署Velero

```bash
# 下载velero（Velero客户端）
[root@master-01 src]#wget https://github.com/vmware-tanzu/velero/releases/download/v1.16.0/velero-v1.16.0-linux-amd64.tar.gz -O /usr/local/src/velero-v1.16.0-linux-amd64.tar.gz

[root@master-01 src]#tar xf velero-v1.16.0-linux-amd64.tar.gz 
[root@master-01 src]#mv velero-v1.16.0-linux-amd64/velero /usr/local/bin

# 测试
[root@master-01 src]#velero version
Client:
	Version: v1.16.0
	Git commit: 8f31599fe4af5453dee032beaf8a16bd75de91a5
```

### 配置Velero认证环境

```bash
# 工作目录：
[root@master-01 src]#mkdir -p /data/velero
[root@master-01 src]#cd /data/velero/

# 访问minio的认证文件：
[root@master-01 velero]#vim velero-auth.txt
[default]
aws_access_key_id = admin
aws_secret_access_key = admin123

# 准备user-csr文件
[root@master-01 velero]#vim awsuser-csr.json
{
  "CN": "awsuser",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}

# 准备证书签发环境
[root@master-01 velero]#apt install golang-cfssl -y
[root@master-01 velero]#wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssl_1.6.1_linux_amd64
[root@master-01 velero]#wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssljson_1.6.1_linux_amd64
[root@master-01 velero]#wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssl-certinfo_1.6.1_linux_amd64
[root@master-01 velero]#mv cfssl-certinfo_1.6.1_linux_amd64 cfssl-certinfo
[root@master-01 velero]#mv cfssl_1.6.1_linux_amd64 cfssl
[root@master-01 velero]#mv cfssljson_1.6.1_linux_amd64 cfssljson
[root@master-01 velero]#cp cfssl-certinfo cfssljson cfssl /usr/local/bin/
[root@master-01 velero]#chmod a+x /usr/local/bin/cfssl*

# 执行证书签发
# kubernetes版本 >= 1.24
[root@haproxy1 ssl]#scp /etc/kubeasz/clusters/k8s-cluster1/ssl/ca-config.json master1:/data/valero

[root@master-01 velero]#cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem -ca-key=/etc/kubernetes/ssl/ca-key.pem -config=./ca-config.json -profile=kubernetes ./awsuser-csr.json |cfssljson -bare awsuser
2025/04/22 19:57:35 [INFO] generate received request
2025/04/22 19:57:35 [INFO] received CSR
2025/04/22 19:57:35 [INFO] generating key: rsa-2048
2025/04/22 19:57:36 [INFO] encoded CSR
2025/04/22 19:57:36 [INFO] signed certificate with serial number 486780908032241703022092215985611021871723081777
2025/04/22 19:57:36 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

[root@master-01 velero]#ll
总计 40264
drwxr-xr-x 2 root root     4096  4月 22 19:57 ./
drwxr-xr-x 3 root root     4096  4月 22 19:55 ../
-rw-r--r-- 1 root root      997  4月 22 19:57 awsuser.csr
-rw-r--r-- 1 root root      220  4月 22 18:05 awsuser-csr.json
-rw------- 1 root root     1675  4月 22 19:57 awsuser-key.pem     # 私钥
-rw-r--r-- 1 root root     1391  4月 22 19:57 awsuser.pem         # 证书
-rw-r--r-- 1 root root      459  4月 22 19:55 ca-config.json
-rw-r--r-- 1 root root 16659824 12月  7  2021 cfssl
-rw-r--r-- 1 root root 13502544 12月  7  2021 cfssl-certinfo
-rw-r--r-- 1 root root 11029744 12月  7  2021 cfssljson
-rw-r--r-- 1 root root       69  4月 22 17:58 velero-auth.txt

# 分发证书到api-server证书路径
[root@master-01 velero]#cp awsuser-key.pem /etc/kubernetes/ssl/
[root@master-01 velero]#cp awsuser.pem /etc/kubernetes/ssl/

# 设定变量（方便修改）
KUBECONFIG_FILE=./awsuser.kubeconfig
APISERVER=https://10.0.0.30:6443
CA_FILE=/etc/kubernetes/ssl/ca.pem
CERT_FILE=/etc/kubernetes/ssl/awsuser.pem
KEY_FILE=/etc/kubernetes/ssl/awsuser-key.pem
USER_NAME=awsuser
NAMESPACE=velero-system

# 设置 cluster
kubectl config set-cluster cluster1 \
  --server=${APISERVER} \
  --certificate-authority=${CA_FILE} \
  --embed-certs=true \
  --kubeconfig=${KUBECONFIG_FILE}

# 设置 user
kubectl config set-credentials ${USER_NAME} \
  --client-certificate=${CERT_FILE} \
  --client-key=${KEY_FILE} \
  --embed-certs=true \
  --kubeconfig=${KUBECONFIG_FILE}

# 设置 context
kubectl config set-context context-${USER_NAME} \
  --cluster=cluster1 \
  --user=${USER_NAME} \
  --namespace=${NAMESPACE} \
  --kubeconfig=${KUBECONFIG_FILE}

# 使用 context
kubectl config use-context context-${USER_NAME} --kubeconfig=${KUBECONFIG_FILE}

# k8s集群中创建awsuser账户
[root@master-01 velero]#kubectl create clusterrolebinding awsuser --clusterrole=admin --user=awsuser
clusterrolebinding.rbac.authorization.k8s.io/awsuser created

# 创建名称空间
[root@master-01 velero]#kubectl create ns velero-system
namespace/velero-system created

# 创建sa
[root@master-01 velero]#kubectl create sa --namespace velero-system awsuser
serviceaccount/awsuser created

# 测试创建的kube-config文件,awsuser.kubeconfig权限是否ok
[root@master velero]#kubectl --kubeconfig=./awsuser.kubeconfig get nodes
NAME        STATUS                     ROLES    AGE   VERSION
master      Ready,SchedulingDisabled   master   45h   v1.31.2
worker-01   Ready                      node     45h   v1.31.2
worker-02   Ready                      node     45h   v1.31.2
worker-03   Ready                      node     45h   v1.31.2


# 执行安装
[root@master velero]#velero install \
  --kubeconfig ./awsuser.kubeconfig \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.5.5 \
  --bucket velero \
  --secret-file ./velero-auth.txt \
  --use-volume-snapshots=false \
  --namespace velero-system \
  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://10.0.0.30:9000

# 卸载velero
# velero uninstall --kubeconfig ./awsuser.kubeconfig --namespace velero-system


# 查看
[root@master velero]#kubectl get pod -n velero-system 
NAME                      READY   STATUS    RESTARTS   AGE
velero-76d4fcf7dc-w9ccj   1/1     Running   0          7s

# 测试
[root@master velero]#velero --namespace velero-system backup-location get
NAME      PROVIDER   BUCKET/PREFIX   PHASE       LAST VALIDATED                  ACCESS MODE   DEFAULT
default   aws        velero          Available   2025-04-28 16:54:46 +0800 CST   ReadWrite     true
```

### 备份

```bash
# 生成当前时间戳（格式：年月日时分秒），用于唯一标识备份
DATE=$(date +%Y%m%d%H%M%S)

# 执行Velero备份命令
velero \
  --kubeconfig=./awsuser.kubeconfig \      # 指定Kubernetes配置文件路径（用于多集群环境）
  --namespace=velero-system \              # 指定Velero控制器所在的命名空间
  backup create default-backup-${DATE} \   # 创建备份，名称包含时间戳避免重复
  --include-cluster-resources=true \      # 备份集群级资源（如StorageClass、Nodes等）
  --include-namespaces='*'                # 备份所有命名空间（*需加引号防止shell扩展）
```

### 查看

```bash
velero --kubeconfig=./awsuser.kubeconfig \
  --namespace=velero-system \
  backup describe default-backup-20250428172211 \
  --details
```

### 恢复

#### 1. 基本恢复命令（恢复到原命名空间）

```bash
velero restore create RESTORE_NAME \
  --from-backup BACKUP_NAME \
  --namespace velero-system \
  --kubeconfig=./awsuser.kubeconfig
```

#### 2. 常用恢复选项

##### 恢复到原命名空间（默认行为）

```bash
velero restore create restore-$(date +%Y%m%d%H%M%S) \
  --from-backup default-backup-20250428172211 \
  --namespace velero-system \
  --kubeconfig=./awsuser.kubeconfig
```

##### 恢复到新命名空间（命名空间映射）

```bash
velero restore create restore-$(date +%Y%m%d%H%M%S) \
  --from-backup default-backup-20250428172211 \
  --namespace-mappings original-ns:new-ns \
  --namespace velero-system \
  --kubeconfig=./awsuser.kubeconfig
```

示例（将 `gitlab-system` 恢复到 `gitlab-restored`）：

```bash
velero restore create gitlab-restore-$(date +%Y%m%d%H%M%S) \
  --from-backup default-backup-20250428172211 \
  --namespace-mappings gitlab-system:gitlab-restored \
  --namespace velero-system \
  --kubeconfig=./awsuser.kubeconfig
```

##### 仅恢复特定资源类型

```bash
velero restore create selective-restore \
  --from-backup default-backup-20250428172211 \
  --include-resources deployments,services \
  --namespace velero-system \
  --kubeconfig=./awsuser.kubeconfig
```

#### 3. 监控恢复进度

```bash
# 查看所有恢复任务
velero restore get --namespace velero-system --kubeconfig=./awsuser.kubeconfig

# 查看特定恢复详情
velero restore describe RESTORE_NAME --namespace velero-system --kubeconfig=./awsuser.kubeconfig

# 查看恢复日志
velero restore logs RESTORE_NAME --namespace velero-system --kubeconfig=./awsuser.kubeconfig
```

#### 4. 恢复完成后的验证

```bash
# 检查恢复的资源
kubectl get all -n RESTORED_NAMESPACE

# 检查 PersistentVolumeClaims
kubectl get pvc -n RESTORED_NAMESPACE

# 检查 Pod 状态
kubectl get pods -n RESTORED_NAMESPACE
```

#### 5. 高级恢复选项

##### 排除某些资源

```bash
velero restore create filtered-restore \
  --from-backup default-backup-20250428172211 \
  --exclude-resources events,secrets \
  --namespace velero-system \
  --kubeconfig=./awsuser.kubeconfig
```

##### 恢复时修改标签

```bash
velero restore create labeled-restore \
  --from-backup default-backup-20250428172211 \
  --selector app=my-app \
  --namespace velero-system \
  --kubeconfig=./awsuser.kubeconfig
```



## ETCD

### 备份

```bash
mkdir /data/backup
DATE=`date +%Y-%m-%d_%H-%M-%S`
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /backup/$DATE.db
```

### 验证

```bash
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /backup/2025-05-06_16-15-01.db
```

### 恢复

#### **标准操作顺序（3个master）**

| 节点           | 操作                                                         |
| -------------- | ------------------------------------------------------------ |
| master01       | `systemctl stop kubelet`                                     |
| master02       | `systemctl stop kubelet`                                     |
| master03       | `systemctl stop kubelet`                                     |
| **全部停止后** | 在 3 个节点都执行 etcdctl snapshot restore（一样的快照文件） |


```bash
cp -a /var/lib/etcd /var/lib/etcd.bak
rm -rf /var/lib/etcd

ETCDCTL_API=3 etcdctl snapshot restore /backup/2025-05-06_16-15-01.db \
  --data-dir=/var/lib/etcd \
  --initial-cluster=default=https://127.0.0.1:2380 \
  --initial-advertise-peer-urls=https://127.0.0.1:2380 \
  --skip-hash-check
```

| 节点           | 操作                                                         |
| -------------- | ------------------------------------------------------------ |
| master01       | `systemctl start kubelet`                                    |
| master02       | `systemctl start kubelet`                                    |
| master03       | `systemctl start kubelet`                                    |

------

#### **特别重要**

> **恢复时，3个节点的数据目录 /var/lib/etcd 都必须用同一个快照恢复出来！**
>  不能只恢复1个节点，其他两个节点不恢复
>  否则 etcd 集群 raft 会数据不一致，启动直接崩溃

------

#### **总结**

✅ **3个 master 节点 kubelet 都要停止**
 ✅ **3个节点 etcd 都要恢复同一个快照**
 ✅ **都恢复完，再同时启动 kubelet**



## 集群管理

### 集群重置

#### **主节点**

```bash
kubeadm reset -f
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
sudo ipvsadm --clear
sudo systemctl restart containerd  # 或者 docker，取决于你的容器运行时
sudo rm -rf ~/.kube
sudo rm -rf /etc/cni/net.d /var/lib/cni/ /var/lib/kubelet /var/lib/etcd /etc/kubernetes
```

#### **node节点**

```bash
sudo kubeadm reset -f
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
sudo ipvsadm --clear
sudo systemctl restart containerd  # 或 docker
sudo rm -rf /etc/cni/net.d /var/lib/cni/ /var/lib/kubelet /etc/kubernetes
```



### 添加节点

#### **添加node**

使用 `kubeadm` 搭建的 Kubernetes 集群中，要添加新的 node 节点（工作节点），可以按照以下步骤进行，分为 **主节点操作** 和 **工作节点操作** 两部分。

------

✅ 一、前提条件

- 已有 kubeadm 搭建好的 Kubernetes 集群（主节点正常运行）
- 新增节点为 Linux 系统，已安装：
  - kubeadm、kubelet、kubectl（kubelet 最重要）
  - Docker 或 containerd
- 新节点的主机名唯一，能通过网络访问主节点
- 关闭防火墙、swap，并启用 `ip_forward` 等基础设置（如下）

📌 系统参数建议

```
# 关闭swap
swapoff -a && sed -i '/swap/d' /etc/fstab

# 修改内核参数
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --system
```

------

🚩 二、主节点操作（获取 `join` 命令）

✅ 1. 在主节点生成 join 命令（带 token）

```
kubeadm token create --print-join-command
```

输出示例：

```
kubeadm join 192.168.0.100:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxx
```

> ⚠️ 默认 token 有效期是 24 小时，超时需重新生成。

------

🖥️ 三、Node 节点操作（执行 join 命令）

✅ 1. 将 `join` 命令复制到新节点执行

```
bash复制编辑kubeadm join 192.168.0.100:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxx
```

执行成功后，node 会加入到集群。

------

📡 四、验证节点是否加入成功（在主节点）

```
kubectl get nodes
```

输出类似：

```
NAME         STATUS   ROLES    AGE   VERSION
master01     Ready    control-plane   5d    v1.28.0
worker01     Ready    <none>          1m    v1.28.0
```

------

🧩 五、常见问题排查

| 问题                                            | 原因                 | 解决方法                                              |
| ----------------------------------------------- | -------------------- | ----------------------------------------------------- |
| `x509: certificate signed by unknown authority` | 主节点证书 hash 不对 | 确保使用 `--discovery-token-ca-cert-hash`             |
| `connection refused`                            | 网络不通、防火墙     | 关闭防火墙，主节点监听地址应为 `0.0.0.0`              |
| `swap not disabled`                             | 未关闭 swap          | `swapoff -a` 并永久关闭                               |
| `token expired`                                 | token 过期           | 重新生成：`kubeadm token create --print-join-command` |

------

✅ 六、附加知识：join 命令补充

可以使用 `--cri-socket` 指定 containerd 或 Docker：

```
--cri-socket /run/containerd/containerd.sock
```

#### 添加master节点

在使用 `kubeadm` 搭建的 Kubernetes 集群中，**添加一个新的 master 节点（控制平面节点）** 是实现高可用集群的关键步骤。与添加普通 node 节点不同，添加 master 节点涉及证书复制、etcd 通信等额外操作。

------

✅ 一、集群添加 master 节点的前提条件

| 条件                                                         | 说明 |
| ------------------------------------------------------------ | ---- |
| 已有一个工作正常的 kubeadm 安装的 master 节点                |      |
| 所有 master 节点之间网络互通                                 |      |
| 使用外部或自建的高可用组件（如 VIP + keepalived 或负载均衡器） |      |
| 新节点已安装 kubeadm、kubelet、kubelet，并关闭 swap、防火墙  |      |

------

🧱 二、添加 master 的核心思路

主节点之间是对等的，需要：

1. **复制证书和 kubeadm 配置**
2. **使用 `kubeadm join` 加 `--control-plane` 加入控制面**
3. **配置 HA（如 VIP / LB）后使用统一的 API Server 地址**

------

🔧 三、操作步骤详解

🌟 1. 主节点生成加入命令（含 control-plane）

在已有主节点执行：

```
kubeadm token create --print-join-command --ttl 0
```

**但要添加 master，需加 `--control-plane` 和证书 hash 文件路径**

------

🛠 2. 拷贝证书到新 master 节点

从已有 master 节点拷贝下列目录到新节点（假设为 `192.168.0.11`）：

```
scp -r /etc/kubernetes/pki root@192.168.0.11:/etc/kubernetes/
scp /etc/kubernetes/admin.conf root@192.168.0.11:/etc/kubernetes/
```

> ⚠️ `pki/etcd` 目录一般不需要复制，除非你使用了自建 etcd 集群；如果是 `kubeadm` 管理的 `etcd`，默认支持复制。

------

🧪 3. 在新 master 上执行 join 命令

```
kubeadm join <API_SERVER>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <证书密钥>
```

- `<API_SERVER>`：可以是主节点 IP，也可以是 VIP 地址（推荐）
- `<token>` 和 `<hash>` 来自上一步 `--print-join-command`
- `<certificate-key>` 是密钥，用于复制控制平面所需证书（下面介绍如何获取）

------

🔐 4. 获取 certificate-key（在主节点执行）

```
kubeadm init phase upload-certs --upload-certs
```

输出类似：

```
Certificate key: 89c3519cbf7b2a1d8a2cfdd214d7fca3e62153b0b6c4010e5ebbb345b4d4c574
```

此 key 有效期为 2 小时，用于新 master 安装控制面时加密解密 pki 证书。

------

🛰️ 四、验证新 master 节点是否添加成功

在任一 master 节点执行：

```
kubectl get nodes
```

应该能看到新加入的 master 节点：

```
NAME         STATUS   ROLES           AGE   VERSION
master-1     Ready    control-plane   5d    v1.28.0
master-2     Ready    control-plane   2m    v1.28.0
```

------

🔁 五、可选：为新 master 配置 kubectl

```
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

------

💡 六、高可用建议

| HA 方案          | 描述                                                |
| ---------------- | --------------------------------------------------- |
| VIP + keepalived | 多个 master，使用 VIP 漂移方式实现主节点高可用      |
| LB + DNS         | 使用 Nginx、HAProxy 或 F5 做负载均衡                |
| etcd 高可用      | 推荐使用外部 etcd 集群或使用 kubeadm 多节点模式部署 |

------

📘 七、总结一句话流程

> **先在主节点生成 token 和 certificate key，然后拷贝必要证书，最后在新 master 上执行带 `--control-plane` 的 join 命令。**



### 删除节点

在 Kubernetes 中删除一个节点（Node）需要分为两部分操作：**从集群中删除节点记录** 和 **从实际机器上停用组件（如 kubelet）**。

------

✅ 一、删除节点的标准操作流程

🧩 步骤 1：在主节点上驱逐节点上的 Pod

防止业务受影响：

```
kubectl drain <节点名> --delete-emptydir-data --force --ignore-daemonsets
```

- `--delete-emptydir-data`：删除使用了 `emptyDir` 卷的 Pod
- `--force`：强制驱逐非管理的 Pod（如静态 Pod）
- `--ignore-daemonsets`：忽略 DaemonSet 的 Pod（因为它们不会自动迁移）

示例：

```
kubectl drain node-1 --delete-emptydir-data --force --ignore-daemonsets
```

------

🧹 步骤 2：将节点从集群中删除

```
kubectl delete node <节点名>
```

示例：

```
kubectl delete node node-1
```

> 此时该节点从 `kubectl get nodes` 中消失，集群视角中已删除。

------

⚙️ 二、在目标节点上做清理操作

✅ 停止并禁用 kubelet

```
systemctl stop kubelet
systemctl disable kubelet
```

✅ 可选：卸载 kubeadm/kubelet/kubectl

```
apt remove kubeadm kubelet kubectl   # Ubuntu/Debian
yum remove kubeadm kubelet kubectl   # RHEL/CentOS/Rocky
```

也可选择保留组件，但关闭服务即可。

------

🧊 三、节点删除后还能恢复吗？

- 如果是误删，只要原节点没有重装系统或变动太大，可以通过重新 `kubeadm join` 加回集群；
- 否则，需要重新初始化节点并加入集群。

------

🧠 四、面试回答模板

> 在 Kubernetes 中删除节点，我会先使用 `kubectl drain` 安全地驱逐 Pod，确保业务迁移后再执行 `kubectl delete node` 从集群中移除节点记录。然后登录该节点，关闭 kubelet 服务，并清理相关组件。整个过程兼顾安全与稳定，避免影响生产环境。



### kubeasz集群节点伸缩管理

集群管理主要是添加master、添加node、删除master与删除node等节点管理及监控

```bash
# 当前集群状态
[root@master-01 ~]#kubectl get nodes
NAME        STATUS                     ROLES    AGE    VERSION
master-01   Ready,SchedulingDisabled   master   128m   v1.30.1
master-02   Ready,SchedulingDisabled   master   128m   v1.30.1
worker-01   Ready                      node     120m   v1.30.1
worker-02   Ready                      node     120m   v1.30.1

[root@haproxy1 kubeasz]#./ezctl --help
Usage: ezctl COMMAND [args]
-------------------------------------------------------------------------------------
Cluster setups:
    list		             to list all of the managed clusters
    checkout    <cluster>            to switch default kubeconfig of the cluster
    new         <cluster>            to start a new k8s deploy with name 'cluster'
    setup       <cluster>  <step>    to setup a cluster, also supporting a step-by-step way
    start       <cluster>            to start all of the k8s services stopped by 'ezctl stop'
    stop        <cluster>            to stop all of the k8s services temporarily
    upgrade     <cluster>            to upgrade the k8s cluster
    destroy     <cluster>            to destroy the k8s cluster
    backup      <cluster>            to backup the cluster state (etcd snapshot)
    restore     <cluster>            to restore the cluster state from backups
    start-aio		             to quickly setup an all-in-one cluster with default settings

Cluster ops:
    add-etcd    <cluster>  <ip>      to add a etcd-node to the etcd cluster
    add-master  <cluster>  <ip>      to add a master node to the k8s cluster
    add-node    <cluster>  <ip>      to add a work node to the k8s cluster
    del-etcd    <cluster>  <ip>      to delete a etcd-node from the etcd cluster
    del-master  <cluster>  <ip>      to delete a master node from the k8s cluster
    del-node    <cluster>  <ip>      to delete a work node from the k8s cluster

Extra operation:
    kca-renew   <cluster>            to force renew CA certs and all the other certs (with caution)
    kcfg-adm    <cluster>  <args>    to manage client kubeconfig of the k8s cluster

Use "ezctl help <command>" for more information about a given command.

```



#### 添加Node节点

```bash
# 1. 打通新加入的Node节点和集群内其他节点的ssh

# 2. 在集群部署服务器，即kubeasz所在服务器，比如新加入node的ip是10.0.0.213
[root@haproxy1 kubeasz]#./ezctl add-node k8s-cluster1 10.0.0.213

# 查看
[root@master-01 ~]#kubectl get node
NAME             STATUS                     ROLES    AGE    VERSION
k8s-10-0-0-213   Ready                      node     54s    v1.30.1
master-01        Ready,SchedulingDisabled   master   144m   v1.30.1
master-02        Ready,SchedulingDisabled   master   144m   v1.30.1
worker-01        Ready                      node     137m   v1.30.1
worker-02        Ready                      node     137m   v1.30.1
```



#### 添加master节点

```bash
# 1. 打通新加入的master节点和集群内其他节点的ssh

# 2. 在集群部署服务器，即kubeasz所在服务器，比如新加入master的ip是10.0.0.203
[root@haproxy1 kubeasz]#./ezctl add-master k8s-cluster1 10.0.0.203

# 查看
[root@master-01 ~]#kubectl get node
NAME             STATUS                     ROLES    AGE     VERSION
k8s-10-0-0-203   Ready,SchedulingDisabled   master   2m36s   v1.30.1
k8s-10-0-0-213   Ready                      node     19m     v1.30.1
master-01        Ready,SchedulingDisabled   master   163m    v1.30.1
master-02        Ready,SchedulingDisabled   master   163m    v1.30.1
worker-01        Ready                      node     155m    v1.30.1
worker-02        Ready                      node     155m    v1.30.1
```



#### 删除node节点

```bash
# 本质上是忽略daemonset,强制drain驱逐node上的pod，再踢出node节点
# --delete-local-data --ignore-daemonsets --force
# --delete-emptydir-data --ignore-daemonsets --force

# 注意！！！，该操作不建议在业务高峰期执行

# 执行删除指定节点
[root@haproxy1 kubeasz]#./ezctl del-node k8s-cluster1 10.0.0.213

# 查看
[root@master-01 ~]#kubectl get node
NAME             STATUS                     ROLES    AGE    VERSION
k8s-10-0-0-203   Ready,SchedulingDisabled   master   10m    v1.30.1
master-01        Ready,SchedulingDisabled   master   170m   v1.30.1
master-02        Ready,SchedulingDisabled   master   170m   v1.30.1
worker-01        Ready                      node     163m   v1.30.1
worker-02        Ready                      node     163m   v1.30.1

# 删除后，重启被删除的node节点，以清理缓存信息
# 但是！！！，此时可能会出现一个问题，就是删除的节点，无法直接再加入集群，原因是hosts文件内的该主机名没有被删除，删除后重新添加就可以了
[root@haproxy1 kubeasz]#vim clusters/k8s-cluster1/hosts
[kube_node]
10.0.0.211 k8s_nodename='worker-01'
10.0.0.212 k8s_nodename='worker-02'
# ？？？ 原10.0.0.213，如果这里没有仍然后痕迹，可能会导致无法加入集群

# 将10.0.0.213再次加入集群
[root@haproxy1 kubeasz]#./ezctl add-node k8s-cluster1 10.0.0.213

# 查看
[root@master-01 ~]#kubectl get node
NAME             STATUS                     ROLES    AGE     VERSION
k8s-10-0-0-203   Ready,SchedulingDisabled   master   36m     v1.30.1
k8s-10-0-0-213   Ready                      node     17m     v1.30.1
master-01        Ready,SchedulingDisabled   master   3h17m   v1.30.1
master-02        Ready,SchedulingDisabled   master   3h17m   v1.30.1
worker-01        Ready                      node     3h10m   v1.30.1
worker-02        Ready                      node     3h10m   v1.30.1
```

