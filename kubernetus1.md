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

###  OpenEBS

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
mathematica复制编辑Stateful Workloads（有状态应用，例如 MySQL、PostgreSQL、Kafka 等）
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



###  statefulSet

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

 

###  Operator



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
[root@master1 LoadBalancer-MatalLB]#cat metallb-ipaddresspool.yaml 
i

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
[root@master1 ~]#kubectl get po -n ingress-nginx 
NAME                                       READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-b49d9c7b9-xcgmc   1/1     Running   0          28m
[root@master1 ~]#kubectl get svc -n ingress-nginx 
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.106.137.131   10.0.0.101    80:32160/TCP,443:31941/TCP   28m
ingress-nginx-controller-admission   ClusterIP      10.96.191.111    <none>        443/TCP                      28m
```



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
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2FUQ0NBVkVDQVFBd0pERU9NQXdHQTFVRUF3d0ZiV0Z6YjI0eEVqQVFCZ05WQkFvTUNXdDFZbVZoWkcxcApiakNDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFMak0yek45YnBzandqL05nVHp3CjN4dGY4dFRqSVdQcUNHTW0yT0ZjRzBPNmt2aXhSWjhGNFdjckk3WXg3S3VLazFVdEFBd3RIY2o3YmJwZ1FNRGMKVFVKWktzcG5JbEs0L2VOaUlTTzgwUFhYNVpRM2wxblQvVmx3VUhITTFoWWtFZ1g5eWJoN2g4aXdBSDJod1BoZgpsZmxYaEFRa0N5eEdHd2t1NDAwQllMMEErRjZSaE9FUFZReU5CakFkUkVZdERiM2tYTEdOK251REVvSklIQUFKCjBudmQyRFFMNmoxVGF6MXJJY1M0b0doYTdxQVdhOFV2Vnl2cVBBcXFvcjRUSDdlS0lrOWl4YkhKbzZveXUrUzIKdkRWbjJkNVluYVY4OUtQTzBxMlFSeGMxeHVxdEFwVmdpQlNyWVFWdDUyZUNWWUhNYklKb1dhaDRzUDRJdWRncgpQeGNDQXdFQUFhQUFNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUJaQUJ0enZLQXp5a2xmak1TbkNLVVpDem5LCjVaWkhMT3FaRVNvWU4wb2daUml4WUNFRlJMLzhYVlBTMW5TZUQ1TkRNcERWTlF6aU1jRzJSTHBobFVoUU5uYU8KWmkySHJCQkxHQTBoTDEycnpFNTBrSW52VUFWVEFKUUE4b3RGZjlDdnpYWkhWODVWcGoxQWFBWTQ2Tk03RFJpZQpjMlp5OWNjMjJjQjZyQy9KVnVQL2t5WkZKbVhpSEcxakJJdmlMS3k0bEl3bU1PZVFyVTVlcUlBZmt4NERXcjFZCmtHblhnYno5allaVFJ3cGNxeUhid1dZRWNNWGlpRGxnSllwc2NaRlpQSHc3RHZmTHM5L3JNbGhiUDZKTDYyUWwKa2NzR0lrbnFGWldUOXk1eFNPczZmRE1Ga2RjVTduRWNsME5hdmJCTHpSL1ZqV24rTmhXN3ZOclZ2WXJrCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
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



### config

```powershell
使用 kubeadm 部署的集群,具有管理员权限的认证请求文件：/etc/kubernetes/admin.conf
```

