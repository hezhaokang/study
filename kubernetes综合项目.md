# kubernetes综合项目

![image-20250421183700469](kubernetes/image-20250421183700469.png)



![image-20250422130743203](kubernetes/image-20250422130743203.png)



# Mall-swarm

```
https://github.com/macrozheng/mall-swarm
```

![image-20250422131410287](kubernetes/image-20250422131410287.png)

```
https://github.com/iKubernetes/learning-k8s/tree/master/Mall-MicroService
```

**部署mysql主从**

```
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install mysql  \
        --set auth.rootPassword=MagEdu \
        --set global.storageClass=openebs-hostpath \
        --set architecture=replication \
        --set auth.database=nacosdb \
        --set auth.username=nacos \
        --set auth.password='magedu.com' \
        --set secondary.replicaCount=2 \
        --set auth.replicationPassword='replpass' \
        bitnami/mysql \
        -n nacos --create-namespace
```

