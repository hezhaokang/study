# 36、Elastic Stack（ELK）

![image-20250316135504748](5day-png/36、ELK.png)

## **ELK** **概述**

生产中可能会遇到如下场景

- 场景一：页面提示超时了，到底是哪个服务响应慢了呢？
- 场景二：系统当前的处理能力如何，TPS、QPS 指标是多少呢？
- 场景三：线上服务部署在多台机器上，你挨个登录机器去查日志，感觉烦不烦？

需要通过各种监控系统解决以下问题

当今监控天下三分。被划分为基于Metrics（指标）监控、基于 Tracing（调用链）监控、基于Logging（日志） 的监控。

![image-20250316144148510](C:/Users/zhaok/AppData/Roaming/Typora/typora-user-images/image-20250316144148510.png)

**什么是** **Elastic Stack**

稍有一定规模的IT架构中，系统和应用的相关日志通常会分散在很多台不同的主机和相应的文件中

如果系统和应用出现异常和问题，相关的开发和运维人员想要排查原因，就要先登录到应用运行所相应的主机，找到上面的相关日志文件再进行查找和分析，所以非常不方便，此外还会涉及到权限和安全问题

而ELK的出现就很好的解决这一问题

ELK 是由一家 Elastic 公司开发的三个开源项目的首字母缩写，即是三个相关的项目组成的系统



**Elasticsearch系是什么?**
Elasticsearch 索引指相互关联的文档集合。Elasticsearch 会以 JSON 文档的形式存储数据。每个文档都会在一组鏈(字段或属性的名称)和它们对应的值(字符串、数字、布尔值、日期、数值组、地理位置或其他类型的数据)之间建立联系。
Elasticsearch 使用的是一种名为倒排索引的数据结构，这一结构的设计可以允许十分快速地进行全文本搜索。倒排索引会列出在所有文档中出现的每个特有词汇，并且可以找到包含每个词汇的全部文档。在索引过程中，Elasticsearch 会存储文档并构建倒排索引，这样用户便可以近实时地对文档数据进行搜索。索引过程是在索引 API 中启动的，通过此 AP|您既可向特定索引中添加 JSON 文档，也可更改特定索引中的 JSON 文档.



**Logstash 的用途是什么?**
Logstash 是 Elastic Stack 的核心产品之一，可用来对数据进行聚合和处理，并将数据发送到 Elasticsearch。Logstash 是一个开源的服务器端数据处理管道，允许您在将数据索引到 Elasticsearch 之前同时从多个来源采集数据，并对数据进行充实和转换。



**Kibana 的用途是什么?**
Kibana 是一款适用于 Elasticsearch 的数据可视化和管理工具，可以提供实时的直方图、线形图、饼状图和地图。Kibana 同时还包括诸如 Canvas 和 Elastic Maps 等高级应用程序;Canvas 允许用户基于自身数据创建定制的动态信息图表，而 Elastic Maps 则可用来对地理空间数据进行可视化。

这三个项目分别是：Elasticsearch、Logstash 和 Kibana。三个项目各有不同的功能

- Elasticsearch 是一个实时的全文搜索,存储库和分析引擎。
- Logstash 是数据处理的管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如Elasticsearch 等存储库中。
- Kibana 则可以让用户在 Elasticsearch 中使用图形和图表对数据进行可视化。

ELK 版本演进: 0.X,1.X,2,X,5.X,6,X,7.X,8.X ....

官网： https://www.elastic.co/

ELK 官方介绍：https://www.elastic.co/cn/what-is/elk-stack

ELK 下载链接：https://www.elastic.co/cn/downloads/

ELK 说明: https://www.elastic.co/guide/cn/index.html

ELK 权威指南: https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

**ELK stack的主要优点：**

- 功能强大：Elasticsearch 是实时全文索引，具有强大的搜索功能
- 配置相对简单：Elasticsearch 全部其于 JSON，Logstash使用模块化配置，Kibana的配置都比较简单。
- 检索性能高效：基于优秀的设计，每次查询可以实时响应，即使百亿级数据的查询也能达到秒级响应。
- 集群线性扩展：Elasticsearch 和 Logstash都可以灵活线性扩展
- 前端操作方便：Kibana提供了比较美观UI前端，操作也比较简单

**官方下载**

```
https://www.elastic.co/cn/downloads/
```

EFK 由ElasticSearch、Fluentd和Kibana三个开源工具组成。

Fluentd是一个实时开源的数据收集器,和logstash功能相似,这三款开源工具的组合为日志数据提供了分布式的实时搜集与分析的监控系统。

Fluentd官网和文档:

```
https://www.fluentd.org/
https://docs.fluentd.org/
```

### **Elasticsearch**

```
https://www.elastic.co/cn/what-is/elasticsearch
```

```
https://db-engines.com/en/ranking
```

为何使用 Elasticsearch？

- **Elasticsearch** **很快。**由于 Elasticsearch 是在 Apache Lucene 基础上构建而成的，所以在全文本搜索方面表现十分出色。Elasticsearch 同时还是一个近实时的搜索平台，这意味着从文档索引操作到文档变为可搜索状态之间的延时很短，一般只有一秒。因此，Elasticsearch 非常适用于对时间有严苛要求的用例，例如安全分析和基础设施监测。
- **Elasticsearch** **具有分布式的本质特征。**Elasticsearch 中存储的文档分布在不同的容器中，这些容器称为分片，可以进行复制以提供数据冗余副本，以防发生硬件故障。Elasticsearch 的分布式特性使得它可以扩展至数百台（甚至数千台）服务器，并处理 PB 量级的数据。
- **Elasticsearch** **包含一系列广泛的功能。**除了速度、可扩展性和弹性等优势以外，Elasticsearch 还有大量强大的内置功能（例如数据汇总和索引生命周期管理），可以方便用户更加高效地存储和搜索数据。
- **Elastic Stack** **简化了数据采集、可视化和报告过程。**通过与 Beats 和 Logstash 进行集成，用户能够在向 Elasticsearch 中索引数据之前轻松地处理数据。同时，Kibana 不仅可针对 Elasticsearch 数据提供实时可视化，同时还提供 UI 以便用户快速访问应用程序性能监测 (APM)、日志和基础设施指标等数据。

Elasticsearch 在速度和可扩展性方面都表现出色，而且还能够索引多种类型的内容，可用于多种场景：

- 应用程序搜索
- 网站搜索
- 企业搜索
- 日志处理和分析
- 基础设施指标和容器监测
- 应用程序性能监测
- 地理空间数据分析和可视化
- 安全分析
- 业务分析

**Near Realtime(NRT)** **几乎实时**

Elasticsearch是一个几乎实时的搜索平台。意思是，从索引一个文档到这个文档可被搜索只需要一点点的延迟，这个时间一般为毫秒级。

**Cluster** **集群**

群集是一个或多个节点（服务器）的集合， 这些节点共同保存整个数据，并在所有节点上提供联合索引和搜索功能。一个集群由一个唯一集群ID确定，并指定一个集群名（默认为“elasticsearch”）。该集群名非常重要，因为节点可以通过这个集群名加入群集，一个节点只能是群集的一部分。

确保在不同的环境中不要使用相同的群集名称，否则可能会导致连接错误的群集节点。

**Node** **节点**

节点是单个服务器实例，它是群集的一部分，可以存储数据，并参与群集的索引和搜索功能。就像一个集群，节点的名称默认为一个随机的通用唯一标识符（UUID），确定在启动时分配给该节点。如果不希望默认，可以定义任何节点名。这个名字对管理很重要，目的是要确定网络服务器对应于ElasticSearch群集节点。

我们可以通过群集名配置节点以连接特定的群集。默认情况下，每个节点设置加入名为“elasticSearch”的集群。这意味着如果启动多个节点在网络上，假设他们能发现彼此都会自动形成和加入一个名为“elasticsearch”的集群。

在单个群集中，您可以拥有尽可能多的节点。此外，如果“elasticsearch”在同一个网络中，没有其他节点正在运行，从单个节点的默认情况下会形成一个新的单节点名为"elasticsearch"的集群。

**Index** **索引** 

索引是具有相似特性的文档集合。例如，可以为客户数据提供索引，为产品目录建立另一个索引，以及为订单数据建立另一个索引。索引由名称（必须全部为小写）标识，该名称用于在对其中的文档执行索引、搜索、更新和删除操作时引用索引。在单个群集中，您可以定义尽可能多的索引。

**注意:索引名不支持大写字母**

**Type** **类型**

在索引中，可以定义一个或多个类型。类型是索引的逻辑类别/分区，其语义完全取决于您。一般来说，类型定义为具有公共字段集的文档。例如，假设你运行一个博客平台，并将所有数据存储在一个索引中。在这个索引中，您可以为用户数据定义一种类型，为博客数据定义另一种类型，以及为注释数据定义另一类型。

Elasticsearch 版本对 type 概念的演变情况如下：

在 5.X 版本中，一个 index 下可以创建多个 type

在 6.X 版本中，一个 index 下只能存在一个 type

在 7.X 版本中，默认可以支持 type ,但可以禁用

在 8.X 版本中，直接就删除 type,即 index 不再支持 type

**Document** **文档**

文档是可以被索引的信息的基本单位。例如，您可以为单个客户提供一个文档，单个产品提供另一个文档，以及单个订单提供另一个文档。本文件的表示形式为JSON（JavaScript Object Notation）格式，这是一种非常普遍的互联网数据交换格式。

在索引/类型中，您可以存储尽可能多的文档。请注意，尽管文档物理驻留在索引中，文档实际上必须索引或分配到索引中的类型。

**Shards & Replicas** **分片与副本**

索引可以存储大量的数据，这些数据可能超过单个节点的硬件限制。例如，十亿个文件占用磁盘空间1TB的单指标可能不适合对单个节点的磁盘, 或者仅从单个节点的搜索请求服务可能太慢

为了解决这一问题，Elasticsearch提供细分指标分成多个块称为分片的能力。当创建一个索引，可以简单地定义想要的分片数量。每个分片本身是一个全功能的、独立的“指数”，可以托管在集群中的任何节点。

Shards分片的重要性主要体现在以下两个特征：

- 分片允许您水平拆分或缩放内容的大小
- 分片允许你分配和并行操作的碎片（可能在多个节点上）从而提高性能/吞吐量

这个机制中的碎片是分布式的以及其文件汇总到搜索请求是完全由ElasticSearch管理，对用户来说是透明的。

在同一个集群网络或云环境上，故障是任何时候都会出现的，拥有一个故障转移机制以防分片和结点因为某些原因离线或消失是非常有用的，并且被强烈推荐。为此，Elasticsearch允许你创建一个或多个拷贝，索引分片进入所谓的副本或称作复制品的分片，简称Replicas。

**注意：ES的副本指不包括主分片的其它副本,即只包括备份，这与Kafka是不同的**

Replicas的重要性主要体现在以下两个特征：

- 副本为分片或节点失败提供了高可用性。需要注意的是，一个副本的分片不会分配在同一个节点作为原始的或主分片，副本是从主分片那里复制过来的。
- 副本允许用户扩展你的搜索量或吞吐量，因为搜索可以在所有副本上并行执行。

**相关概念在关系型数据库和ElasticSearch中的对应关系**

![image-20250316145833281](5day-png/36关系型数据库和ElasticSearch中的对应关系.png)

| 关系型数据库    | Elacsearch                                                   |
| --------------- | ------------------------------------------------------------ |
| 数据库 Database | 索引 Index，支持全文检索                                     |
| 表 table        | 类型 Type（废弃）                                            |
| 数据行 Row      | 文档 Document，但不需要固定结构，不同文档可以具有不同字段集合 |
| 数据列 Coumn    | 字段 Field                                                   |
| SQL 语言        | DSL(domain specific language) 是Elasticsearch提供的JSON风格的请求语句，用来操作ES实现CRUD |

| 概念           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| 索引库 indices | indices是index的复数，代表许多的索引，                       |
| 类型 type      | 类型是模拟mysql中的table概念，一个索引库下可以有不同类型的索引，比如商品索引，订单索引，其数据格式不同。不过这会导致索引库混乱，因此未来版本中会移除这个概念 |
| 文档 document  | 存入索引库原始的数据。比如每一条商品信息，就是一个文档       |
| 字段 field     | 文档中的属性                                                 |

### **Logstash**

```
https://www.elastic.co/cn/what-is/elasticsearch
```

![image-20250316150336387](5day-png/36Logstash.png)

Logstash 是 Elastic Stack 的核心产品之一，可用来对数据进行聚合和处理，并将数据发送到Elasticsearch。Logstash 是一个基于Java实现的开源的服务器端数据处理管道，允许您在将数据索引到Elasticsearch 之前同时从多个来源采集数据，并对数据进行过滤和转换。

可以通过插件实现日志收集和转发，支持日志过滤，支持普通log、自定义json格式的日志解析。

### Kibana

Kibana 是一款适用于 Elasticsearch 的基于Javascript语言实现的数据可视化和管理工具，可以提供实时的直方图、线形图、饼状图和地图。Kibana 同时还包括诸如 Canvas 和 Elastic Maps 等高级应用程序；Canvas 允许用户基于自身数据创建定制的动态信息图表，而 Elastic Maps 则可用来对地理空间数据进行可视化。

官方文档

```
https://www.elastic.co/cn/kibana
```

主要是通过接口调用elasticsearch的数据，并进行前端数据可视化的展现。

Kibana 与 Elasticsearch 和更广意义上的 Elastic Stack 紧密的集成在一起，这一点使其成为支持以下场景的理想选择：

- 搜索、查看并可视化 Elasticsearch 中所索引的数据，并通过创建柱状图、饼图、表格、直方图和地图对数据进行分析。仪表板视图能将这些可视化元素组织到一起，然后通过浏览器进行分享，以提供对海量数据的实时分析视图，所支持的用例如下：

  ```powershell
  1. 日志处理和分析
  2. 基础设施指标和容器监测
  3. 应用程序性能监测 (APM)
  4. 地理空间数据分析和可视化
  5. 安全分析
  6. 业务分析
  ```

- 借助网络界面来监测和管理 Elastic Stack 实例并确保实例的安全

- 针对基于 Elastic Stack 开发的内置解决方案（面向可观测性、安全和企业搜索应用程序），将其访问权限集中到一起

### **ELK** **应用场景**

运维主要应用场景：

- 将分布在不同主机/容器的日志统一收集,并进行转换，通过集中的Web UI 进行查询和管理
- 通过查看汇总的日志,找到故障的根本原因
- Web 展示和报表功能
- 实现安全和事件等管理

大数据运维主要应用场景：

- 查询聚合, 大屏分析
- 预测告警, 网络指标，业务指标安全指标
- 日志查询，问题排查，基于API可以实现故障恢复和自愈
- 用户行为，性能,业务分析



## **Elasticsearch** **部署和管理**

部署方式

- 包安装
- 二进制安装
- Docker 部署
- Kubernetes 部署
- Ansible 批量部署

ES支持操作系统版本和 Java 版本官方说明

```
https://www.elastic.co/cn/support/matrix
```

**Elasticsearch** **安装前准备**

**关闭防火墙和SELinux**

关闭防所有服务器的防火墙和 SELinux

```bash
#RHEL系列的系统执行下以下配置
[root@es-node1 ~]# systemctl disable firewalld
[root@es-node1 ~]# systemctl disable NetworkManager
[root@es-node1 ~]# sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
[root@es-node1 ~]# reboot
```

**优化资源限制配置**

内核参数 vm.max_map_count 用于限制一个进程可以拥有的VMA(虚拟内存区域)的数量

使用默认系统配置，二进制安装时会提示下面错误，包安装会自动修改此配置

```bash
#查看默认值
[root@ubuntu2204 ~]#sysctl -a |grep vm.max_map_count 
vm.max_map_count = 65530

[root@es-node1 ~]#sysctl -a |grep vm.max_map_count 
vm.max_map_count = 65530

#修改配置
[root@es-node1 ~]#echo "vm.max_map_count = 262144" >> /etc/sysctl.conf

#设置系统最大打开的文件描述符数
[root@es-node1 ~]#echo "fs.file-max = 1000000" >> /etc/sysctl.conf
[root@es-node1 ~]#sysctl -p 
vm.max_map_count = 262144

#Ubuntu22.04默认值已经满足要求
[root@ubuntu2204 ~]#sysctl fs.file-max
fs.file-max = 9223372036854775807
```

**注意:安装7.X以后版本官方建议要安装集成JDK的包,所以无需再专门安装JDK**

```
https://www.elastic.co/cn/support/matrix#matrix_jvm
```

```bash
#包含JDK的版本下载
https://www.elastic.co/downloads/elasticsearch
https://mirrors.tuna.tsinghua.edu.cn/elasticstack/
#不包含JDK的版本下载
https://www.elastic.co/cn/downloads/elasticsearch-no-jdk
```

范例: 查看两种包

```bash
[root@node1 ~]#ll -h elasticsearch-7.14.0-*
-rw-r--r-- 1 root root 329M Aug 15 19:39 elasticsearch-7.14.0-amd64.deb
-rw-r--r-- 1 root root 183M Aug 15 19:38 elasticsearch-7.14.0-no-jdk-amd64.deb
```



### **包安装** **Elasticsearch**

```
https://www.elastic.co/cn/downloads/elasticsearch
https://mirrors.tuna.tsinghua.edu.cn/elasticstack/
https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/8.x/pool/main/e/elasticsearch/
```

```bash
[root@ubuntu2404 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/e/elasticsearch/elasticsearch-8.17.3-amd64.deb
[root@ubuntu2404 ~]#apt install -y ./elasticsearch-8.17.3-amd64.deb
#创建用户和组，服务启动必须用普通用户启动
[root@ubuntu2404 ~]#id elasticsearch
uid=108(elasticsearch) gid=108(elasticsearch) groups=108(elasticsearch)

[root@ubuntu2404 ~]#/usr/share/elasticsearch/jdk/bin/java --version
openjdk 23 2024-09-17
OpenJDK Runtime Environment (build 23+37-2369)
OpenJDK 64-Bit Server VM (build 23+37-2369, mixed mode, sharing)

#启动
[root@ubuntu2404 ~]#systemctl enable --now elasticsearch.service 
[root@ubuntu2404 ~]#systemctl status elasticsearch.service
[root@ubuntu2404 ~]#ss -tnulp | grep java
tcp   LISTEN 0      4096                    *:9200            *:*    users:(("java",pid=1630,fd=537))                      
tcp   LISTEN 0      4096   [::ffff:127.0.0.1]:9300            *:*    users:(("java",pid=1630,fd=535))                      
tcp   LISTEN 0      4096                [::1]:9300         [::]:*    users:(("java",pid=1630,fd=534))  
```

```bash
Authentication and authorization are enabled.
TLS for the transport and HTTP layers is enabled and configured.

The generated password for the elastic built-in superuser is : IoTqAu0pW3Mu1wIzS8oy

[root@ubuntu2404 ~]#curl 10.0.0.200:9000
curl: (7) Failed to connect to 10.0.0.200 port 9000 after 0 ms: Couldn't connect to server
[root@ubuntu2404 ~]#curl https://10.0.0.200:9200 -k
{"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":["Basic realm=\"security\", charset=\"UTF-8\"","Bearer realm=\"security\"","ApiKey"]}}],"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":["Basic realm=\"security\", charset=\"UTF-8\"","Bearer realm=\"security\"","ApiKey"]}},"status":401}
[root@ubuntu2404 ~]#curl https://elastic:IoTqAu0pW3Mu1wIzS8oy@10.0.0.200:9200 -k
{
  "name" : "ubuntu2404.wang.org",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "JT2eKuH0SHGhFvwndNHkCg",
  "version" : {
    "number" : "8.17.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
    "build_date" : "2025-02-28T10:07:26.089129809Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}

#关闭xpack
[root@ubuntu2404 ~]#vim /etc/elasticsearch/elasticsearch.yml 
# Enable security features
xpack.security.enabled: false		#true改为false

xpack.security.enrollment.enabled: true

[root@ubuntu2404 ~]#systemctl restart elasticsearch.service 
[root@ubuntu2404 ~]#curl http://10.0.0.200:9200
{
  "name" : "ubuntu2404.wang.org",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "JT2eKuH0SHGhFvwndNHkCg",
  "version" : {
    "number" : "8.17.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
    "build_date" : "2025-02-28T10:07:26.089129809Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

**查看service文件**

```bash
[root@ubuntu2404 ~]#cat /lib/systemd/system/elasticsearch.service 
[Unit]
Description=Elasticsearch
Documentation=https://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
# the elasticsearch process currently sends the notifications back to systemd
# and for some reason exec does not work (even though it is a child). We should change
# this notify access back to main (the default), see https://github.com/elastic/elasticsearch/issues/86475
NotifyAccess=all
RuntimeDirectory=elasticsearch
PrivateTmp=true
Environment=ES_HOME=/usr/share/elasticsearch
Environment=ES_PATH_CONF=/etc/elasticsearch
Environment=PID_DIR=/var/run/elasticsearch
Environment=ES_SD_NOTIFY=true
EnvironmentFile=-/etc/default/elasticsearch

WorkingDirectory=/usr/share/elasticsearch

User=elasticsearch
Group=elasticsearch

ExecStart=/usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid --quiet

# StandardOutput is configured to redirect to journalctl since
# some error messages may be logged in standard output before
# elasticsearch logging system is initialized. Elasticsearch
# stores its logs in /var/log/elasticsearch and does not use
# journalctl by default. If you also want to enable journalctl
# logging, you can simply remove the "quiet" option from ExecStart.
StandardOutput=journal
StandardError=inherit

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65535

# Specifies the maximum number of processes
LimitNPROC=4096

# Specifies the maximum size of virtual memory
LimitAS=infinity

# Specifies the maximum file size
LimitFSIZE=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=0

# SIGTERM signal is used to stop the Java process
KillSignal=SIGTERM

# Send the signal only to the JVM rather than its control group
KillMode=process

# Java process is never killed
SendSIGKILL=no

# When a JVM receives a SIGTERM signal it exits with code 143
SuccessExitStatus=143

# Allow a slow startup before the systemd notifier module kicks in to extend the timeout
TimeoutStartSec=900

[Install]
WantedBy=multi-user.target

# Built for packages-8.17.3 (packages)
```



### 二进制安装

```bash
#官方文档
https://www.elastic.co/guide/en/elasticsearch/reference/master/targz.html
#下载二进制文件
https://www.elastic.co/cn/downloads/elasticsearch
```

```bash
[root@ubuntu2204 ~]#wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.3-linux-x86_64.tar.gz
[root@ubuntu2204 ~]#ls
elasticsearch-8.17.3-linux-x86_64.tar.gz
[root@ubuntu2204 ~]#tar xf elasticsearch-8.17.3-linux-x86_64.tar.gz -C /usr/local/
[root@ubuntu2204 ~]#cd /usr/local/
[root@ubuntu2204 local]#ln -s elasticsearch-8.17.3/ elasticsearch
[root@ubuntu2204 local]#cd elasticsearch
[root@ubuntu2204 elasticsearch]#ls
bin  config  jdk  lib  LICENSE.txt  logs  modules  NOTICE.txt  plugins  README.asciidoc
```

**编辑服务配置文件**

```bash
[root@ubuntu2204 ~]#vim /usr/local/elasticsearch/config/elasticsearch.yml 
[root@ubuntu2204 ~]#cat /usr/local/elasticsearch/config/elasticsearch.yml | grep -Ev '^#'
cluster.name: es-clusterx
node.name: es-node
path.data: /data/es-data
path.logs: /data/es-logs
network.host: 10.0.0.100
```

**修改** **ELK** **内存配置和内核参数**

```bash
[root@ubuntu2204 ~]#vim /usr/local/elasticsearch/config/jvm.options
-Xms512M
-Xmx512M

[root@ubuntu2204 ~]#echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
[root@ubuntu2204 ~]#sysctl -p
```

**创建用户**

```bash
[root@ubuntu2204 ~]#useradd -r elasticsearch
[root@ubuntu2204 ~]#id elasticsearch 
uid=998(elasticsearch) gid=998(elasticsearch) groups=998(elasticsearch)
```

**目录权限更改**

```bash
#可以不用创建下面目录es-data和es-logs，系统可以自动创建
[root@ubuntu2204 ~]#mkdir -p /data/es-{data,logs}
[root@ubuntu2204 ~]#chown -R elasticsearch:elasticsearch /data/es-{data,logs}
[root@ubuntu2204 ~]#chown -R elasticsearch.elasticsearch /usr/local/elasticsearch/
```

**创建** **service** **文件**

```bash
[root@ubuntu2204 ~]#vim /lib/systemd/system/elasticsearch.service 
[Unit]
Description=Elasticsearch
Documentation=http://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
RuntimeDirectory=elasticsearch
PrivateTmp=true
Environment=PID_DIR=/var/run/elasticsearch
WorkingDirectory=/usr/local/elasticsearch
User=elasticsearch
Group=elasticsearch
ExecStart=/usr/local/elasticsearch/bin/elasticsearch -p
${PID_DIR}/elasticsearch.pid --quiet

# StandardOutput is configured to redirect to journalctl since
# some error messages may be logged in standard output before
# elasticsearch logging system is initialized. Elasticsearch
# stores its logs in /var/log/elasticsearch and does not use
# journalctl by default. If you also want to enable journalctl
# logging, you can simply remove the "quiet" option from ExecStart.

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65535

# Specifies the maximum number of processes
LimitNPROC=4096

# Specifies the maximum size of virtual memory
LimitAS=infinity

# Specifies the maximum file size
LimitFSIZE=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=0

# SIGTERM signal is used to stop the Java process
KillSignal=SIGTERM

# Send the signal only to the JVM rather than its control group
KillMode=process

# Java process is never killed
SendSIGKILL=no

# When a JVM receives a SIGTERM signal it exits with code 143
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target

# Built for packages-7.6.2 (packages)
```

**启动**

```bash
[root@ubuntu2204 ~]#systemctl daemon-reload && systemctl enable --now elasticsearch.service 
[root@ubuntu2204 ~]#ss -tnulp | grep java
tcp   LISTEN 0      4096   [::ffff:10.0.0.100]:9200            *:*    users:(("java",pid=1749,fd=535))         
tcp   LISTEN 0      4096   [::ffff:10.0.0.100]:9300            *:*    users:(("java",pid=1749,fd=533))     
```

**验证**

```bash
[root@ubuntu2204 ~]#curl 10.0.0.100:9200
curl: (52) Empty reply from server
```

```bash
#修改配置
[root@ubuntu2204 ~]#cat /usr/local/elasticsearch/config/elasticsearch.yml | grep -Ev '^#'
cluster.name: es-clusterx
node.name: es-node
path.data: /data/es-data
path.logs: /data/es-logs
network.host: 10.0.0.100


xpack.security.enabled: false

xpack.security.enrollment.enabled: true

xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
cluster.initial_master_nodes: ["es-node"]
```

```bash
[root@ubuntu2204 ~]#curl 10.0.0.100:9200
{
  "name" : "es-node",
  "cluster_name" : "es-clusterx",
  "cluster_uuid" : "Fw_NJuaZTSCa1-Hz2_L1SQ",
  "version" : {
    "number" : "8.17.3",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
    "build_date" : "2025-02-28T10:07:26.089129809Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### docker部署

```bash
docker run --name es-single-node -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.6.2
```

```
[root@ubuntu2204 ~]#curl 10.0.0.101:9200
{
  "name" : "ef602d6557b4",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "SXTA1kQZRJGXQ4qW4SZMFw",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



### 配置文件

```bash
[root@es-node1 ~]# grep "^[a-Z]" /etc/elasticsearch/elasticsearch.yml 
#ELK集群名称，单节点无需配置，同一个集群内每个节点的此项必须相同,新加集群的节点此项和其它节点相同即可加入集群,而无需再验证
cluster.name: ELK-Cluster 

#当前节点在集群内的节点名称，同一集群中每个节点要确保此名称唯一
node.name: es-node1 

#ES 数据保存目录，包安装默认路径：/var/lib/elasticsearch/，生产建议修改
path.data: /data/es-data 

#ES 日志保存目录，包安装默认路径：/var/log/elasticsearch/，生产建议修改
path.logs: /data/es-logs

#服务启动的时候立即分配(锁定)足够的内存，防止数据写入swap,提高启动速度,但是true会导致启动失败，需要优化
bootstrap.memory_lock: true

#指定该节点用于集群的监听IP，默认监听在127.0.0.1:9300，集群模式必须修改此行，单机默认即可
network.host: 0.0.0.0 

#指定HTTP的9200/TCP的监听地址，默认127.0.0.1
http.host: 0.0.0.0

#监听端口
http.port: 9200

#发现集群的node节点列表，可以添加部分或全部节点IP
#在新增节点到已有集群时,此处需指定至少一个已经在集群中的节点地址，给予可用性，一般添加所有节点地址
discovery.seed_hosts: ["10.0.0.101","10.0.0.102","10.0.0.103"]

#集群初始化时指定希望哪些节点可以被选举为 master,只在初始化时使用,新加节点到已有集群时此项可不配置
cluster.initial_master_nodes: ["10.0.0.101","10.0.0.102","10.0.0.103"]

#一个集群中的 N 个节点启动后,才允许进行数据恢复处理，默认是1,一般设为为所有节点的一半以上,防止出现脑裂现象
#当集群无法启动时,可以将之修改为1,或者将下面行注释掉,实现快速恢复启动
gateway.recover_after_nodes: 2

#设置是否可以通过正则表达式或者_all匹配索引库进行删除或者关闭索引库，默认true表示必须需要明确指定索引库名称，不能使用正则表达式和_all，生产环境建议设置为 true，防止误删索引库。
action.destructive_requires_name: true

#如果不参与主节点选举设为false,默认值为true
#新版：8.X https://www.elastic.co/guide/en/elasticsearch/reference/master/modulesnode.html
node.roles: [ master ]
node.roles: [ data, master, voting_only ]
#旧版7,X
node.master: false

#存储数据,默认值为true,此值为false则不存储数据而成为一个路由节点
#如果将原有的true改为false,需要先执行/usr/share/elasticsearch/bin/elasticsearch-node repurpose 清理数据
node.data: true

#7.x以后版本下面指令已废弃，在2.x 5.x 6.x 版本中用于配置节点发现列表
discovery.zen.ping.unicast.hosts: ["10.0.0.101", "10.0.0.102","10.0.0.103"]

#8.X版后默认即开启Xpack功能，可以修改为false禁用
xpack.security.enabled: true

#开启跨域访问支持，默认为false
http.cors.enabled: true

# 跨域访问允许的域名地址，(允许所有域名)以上使用正则
http.cors.allow-origin: "*"
```



```bash
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/lib/elasticsearch
#
# Path to log files:
#
path.logs: /var/log/elasticsearch
#

#数据目录默认存放位置
[root@ubuntu2404 ~]#ls /var/lib/elasticsearch/
indices  node.lock  nodes  snapshot_cache  _state
```

```bash
[root@ubuntu2404 ~]#vim /etc/elasticsearch/elasticsearch.yml 
bootstrap.memory_lock: true
```

#### **开启 bootstrap.memory_lock 优化**

`bootstrap.memory_lock: true` 是 **Elasticsearch** 配置文件 (`elasticsearch.yml`) 中的一个选项，作用是**防止 JVM 内存被操作系统交换到磁盘**（即**防止内存分页**），提高性能和稳定性。

**开启**` bootstrap.memory_lock: true `**可以优化性能，但会导致无法启动的错误解决方法**

注意：开启 bootstrap.memory_lock: true 需要足够的内存，建议4G以上，否则内存不足，启动会很失败或很慢

**作用**

- 启用后，Elasticsearch 会尝试**锁定 JVM 的堆内存**，避免被操作系统交换到磁盘，从而减少 GC（垃圾回收）时的 I/O 磁盘负担，提高搜索性能。
- 特别适用于 **大数据量、高性能需求的 Elasticsearch 集群**，防止因交换（swap）导致的性能下降。

在 Linux 上，`bootstrap.memory_lock: true` 需要 `mlockall` 机制，否则可能启动失败。

在 `elasticsearch.service`（如果使用 systemd）中添加：

```bash
LimitMEMLOCK=infinity
```

**检查是否生效** 运行以下命令：

```bash
[root@ubuntu2404 ~]#curl -X GET "localhost:9200/_nodes?filter_path=**.mlockall&pretty"
{
  "nodes" : {
    "JOyy_-3lQhiCmUIamlnjhg" : {
      "process" : {
        "mlockall" : false
      }
    }
  }
}
```

#### **内存优化**

```
https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-size-settings
https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-heap-size
```

**设置 JVM 堆大小**

默认情况下，Elasticsearch 会根据节点的 [角色](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-roles)和总内存自动设置 JVM 堆大小。建议在大多数生产环境中使用默认大小。

要覆盖默认堆大小，请设置最小和最大堆大小设置，`Xms`以及`Xmx`。最小值和最大值必须相同。

堆大小应基于可用的 RAM：

- `Xms`将设置`Xmx`为不超过总内存的 50%。Elasticsearch 需要内存用于 JVM 堆以外的用途。例如，Elasticsearch 使用堆外缓冲区实现高效的网络通信，并依靠操作系统的文件系统缓存实现高效的文件访问。JVM 本身也需要一些内存。Elasticsearch 使用的内存超过`Xmx`设置中配置的限制是正常的。

- 将`Xms`和`Xmx`设置为不超过压缩普通对象指针 (oops) 的阈值。确切的阈值各不相同，但在大多数系统上 26GB 是安全的，在某些系统上可能高达 30GB。要验证您是否低于阈值，请检查 Elasticsearch 日志中是否有这样的条目：

  ```txt
  heap size [1.9gb], compressed ordinary object pointers [true]
  ```

Elasticsearch 可用的堆越多，它可用于内部缓存的内存就越多。这样操作系统用于文件系统缓存的内存就更少了。较大的堆也会导致更长的垃圾收集暂停时间。

要配置堆大小，请将`Xms`和`Xmx`JVM 参数添加到具有扩展名的自定义 JVM 选项文件中`.options`，并将其存储在`jvm.options.d/`目录中。例如，要将最大堆大小设置为 2GB，请将`Xms`和都设置`Xmx`为`2g`：

推荐使用宿主机物理内存的一半，ES的heap内存最大不超过30G,26G是比较安全的

```bash
#修改jvm堆内存大小，一般为宿主机的一半，不超过26G
[root@ubuntu2404 ~]#vim /etc/elasticsearch/jvm.options
-Xms2g		设置 JVM 启动时的初始堆内存大小为 2GB。
-Xmx2g		设置 JVM 堆内存的最大值为 2GB，即 JVM 运行时最多能使用 2GB 的堆内存。
```

```powershell
堆大小应基于可用 RAM：
将 Xms 和 Xmx 设置为不超过总内存的 50%。 Elasticsearch 需要内存用于 JVM 堆以外的用途。 例如，Elasticsearch 使用堆外缓冲区来实现高效的网络通信，并依靠操作系统的文件系统缓存来高效地访问文件。 JVM 本身也需要一些内存。 Elasticsearch 使用比 Xmx 设置配置的限制更多的内存是正常的。在容器（例如 Docker）中运行时，总内存定义为容器可见的内存量，而不是主机上的总系统内存。将 Xms 和 Xmx 设置为不超过压缩普通对象指针 (oops) 的阈值。 确切的阈值会有所不同，但在大多数系统上 26GB 是安全的，在某些系统上可能高达 30GB。
```

```powershell
关于OOPS的说明
Java 堆中的托管指针指向在 8 字节地址边界上对齐的对象。 压缩 oop 将托管指针（在 JVM 软件中的许多但不是所有地方）表示为相对于 64 位 Java 堆基地址的 32 位对象偏移量。因为它们是对象偏移量而不是字节偏移量，所以它们可用于处理多达 40 亿个对象（不是字节），或高达约32 GB 的堆大小。
要使用它们，必须将它们缩放 8 倍并添加到 Java 堆基地址以找到它们所引用的对象。 使用压缩 oop 的对象大小与 ILP32 模式中的对象大小相当。
```

```powershell
关于 Heap 内存大小
虽然JVM可以处理大量的堆内存，但是将堆内存设置为过大的值可能导致以下问题：

堆内存分配的效率低。Java语言本身就是一种高级语言，这意味着需要更多的堆内存来存储对象。但是，当堆内存过大时，分配对象所需的时间也会相应增加，这可能会导致应用程序出现性能问题。

操作系统内存管理的限制。操作系统必须以页为单位进行内存管理。如果Java堆内存过大，则需要更多的页来管理堆内存。这可能会导致操作系统出现性能问题。

垃圾回收(Garbage Collection, GC)：JVM内存的一部分被用于存储对象，这些对象随着时间的推移可能不再需要。这些不再需要的对象被视为“垃圾”，需要由垃圾收集器清除，以释放内存空间。然而，执行GC会暂停所有的应用线程，这被称为 "Stop-the-World"（暂停世界）。这种暂停可能会影响应用的性能和响应时间。一般来说，如果堆内存非常庞大，GC需要检查和清理的对象数量会变得非常庞大，这会导致GC操作的时间变得非常漫长。

对象指针的大小：在某些JVM实现（例如Oracle的HotSpot），在堆（Heap）大小超过32GB之后，对象指针的表示将从32位压缩oops（Ordinary Object Pointers普通对象指针）转变为64位非压缩指针，这导致了内存使用的增加。如果内存设置接近或略超过32GB，实际上可能会因为此原因造成更多的内存消耗。因此，通常在32GB以下时，我们会使用32位压缩指针，而超过这个阈值时，除非有明确的需要，否则通常会选择保持在30GB左右以避免转为64位指针。

因此，建议将Java堆内存设置为合适的大小，以便在GC操作的同时与应用程序的性能之间进行平衡。通常情况下，堆内存应该设置为操作系统的物理内存的一半或三分之一。虽然这个数字可能会因系统配置和工作负载而有所变化，但是在32G的机器上，32G的堆空间已经超出了大部分Java应用程序的需求，因此更大的堆内存并不是必要的。

当然，根据具体的应用场景和需求，以及你使用的具体的JVM版本和垃圾收集器类型，这个30GB的规则并非绝对。比如ZGC和Shenandoah这类的低延迟垃圾回收器就可以处理大于30GB的堆内存，同时还能保持低停顿时间。
```

#### 修改service文件，做优化配置

```bash
[root@es-node1 ~]# vim /usr/lib/systemd/system/elasticsearch.service #修改内存限制
LimitNOFILE=1000000  #修改最大打开的文件数，默认值为65535
LimitNPROC=65535   #修改打开最大的进程数，默认值为4096
LimitMEMLOCK=infinity #无限制使用内存，以前旧版需要修改，否则无法启动服务，8.X当前版本无需修改
```



### 集群部署

| IP         | hostname |
| ---------- | -------- |
| 10.0.0.201 | node1    |
| 10.0.0.202 | node2    |
| 10.0.0.203 | node3    |



```bash
#修改主机名
[root@ubuntu2404 ~]#hostnamectl hostname node1
[root@ubuntu2404 ~]#hostnamectl hostname node2
[root@ubuntu2404 ~]#hostnamectl hostname node3
```

```bash
[root@node1 ~]#apt install ./elasticsearch-8.17.3-amd64.deb 
[root@node2 ~]#apt install ./elasticsearch-8.17.3-amd64.deb 
[root@node3 ~]#apt install ./elasticsearch-8.17.3-amd64.deb 
```

```bash
#修改配置文件
[root@node1 ~]#vim /etc/elasticsearch/elasticsearch.yml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node-1		#每个节点不一样
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/lib/elasticsearch
#
# Path to log files:
#
path.logs: /var/log/elasticsearch
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
#
network.host: 0.0.0.0
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.seed_hosts: ["10.0.0.201", "10.0.0.202", "10.0.0.203"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["10.0.0.201", "10.0.0.202", "10.0.0.203"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Allow wildcard deletion of indices:
#
#action.destructive_requires_name: false

#----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
#
# The following settings, TLS certificates, and keys have been automatically      
# generated to configure Elasticsearch security features on 16-03-2025 09:11:30
#
# --------------------------------------------------------------------------------

# Enable security features
xpack.security.enabled: false

xpack.security.enrollment.enabled: true

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
# Create a new cluster with the current node only
# Additional nodes can still join the cluster later
#cluster.initial_master_nodes: ["node1"]

# Allow HTTP API connections from anywhere
# Connections are encrypted and require user authentication
http.host: 0.0.0.0

# Allow other nodes to join the cluster from anywhere
# Connections are encrypted and mutually authenticated
#transport.host: 0.0.0.0

#----------------------- END SECURITY AUTO CONFIGURATION -------------------------
```

```bash
#同步到其他节点上
[root@node1 ~]#scp /etc/elasticsearch/elasticsearch.yml 10.0.0.202:/etc/elasticsearch/elasticsearch.yml
[root@node1 ~]#scp /etc/elasticsearch/elasticsearch.yml 10.0.0.203:/etc/elasticsearch/elasticsearch.yml
#在其他节点上修改
node.name: node-1		#每个节点不一样
```

```bash
#修改sercice文件
[root@node1 ~]#vim /lib/systemd/system/elasticsearch.service
[Unit]
Description=Elasticsearch
Documentation=https://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
# the elasticsearch process currently sends the notifications back to systemd
# and for some reason exec does not work (even though it is a child). We should change
# this notify access back to main (the default), see https://github.com/elastic/elasticsearch/issues/86475
NotifyAccess=all
RuntimeDirectory=elasticsearch
PrivateTmp=true
Environment=ES_HOME=/usr/share/elasticsearch
Environment=ES_PATH_CONF=/etc/elasticsearch
Environment=PID_DIR=/var/run/elasticsearch
Environment=ES_SD_NOTIFY=true
EnvironmentFile=-/etc/default/elasticsearch

WorkingDirectory=/usr/share/elasticsearch

User=elasticsearch
Group=elasticsearch

ExecStart=/usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid --quiet

# StandardOutput is configured to redirect to journalctl since
# some error messages may be logged in standard output before
# elasticsearch logging system is initialized. Elasticsearch
# stores its logs in /var/log/elasticsearch and does not use
# journalctl by default. If you also want to enable journalctl
# logging, you can simply remove the "quiet" option from ExecStart.
StandardOutput=journal
StandardError=inherit

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65535

# Specifies the maximum number of processes
LimitNPROC=4096

# Specifies the maximum size of virtual memory
LimitAS=infinity

# Specifies the maximum file size
LimitFSIZE=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=0

# SIGTERM signal is used to stop the Java process
KillSignal=SIGTERM

# Send the signal only to the JVM rather than its control group
KillMode=process

# Java process is never killed
SendSIGKILL=no

# When a JVM receives a SIGTERM signal it exits with code 143
SuccessExitStatus=143

# Allow a slow startup before the systemd notifier module kicks in to extend the timeout
TimeoutStartSec=900
LimitMEMLOCK=infinity			#添加这一行

[Install]
WantedBy=multi-user.target

# Built for packages-8.17.3 (packages)
[root@node1 ~]#scp /lib/systemd/system/elasticsearch.service 10.0.0.202:/lib/systemd/system/elasticsearch.service
[root@node1 ~]#scp /lib/systemd/system/elasticsearch.service 10.0.0.203:/lib/systemd/system/elasticsearch.service
```



```bash
#修改堆内存大小
[root@node1 ~]#vim /etc/elasticsearch/jvm.options
-Xms1g
-Xmx1g
#同步到其他节点
[root@node1 ~]#scp /etc/elasticsearch/jvm.options 10.0.0.202:/etc/elasticsearch/jvm.options
[root@node1 ~]#scp /etc/elasticsearch/jvm.options 10.0.0.203:/etc/elasticsearch/jvm.options
```

```bash
#启动
[root@node1 ~]#systemctl daemon-reload && systemctl enable --now elasticsearch.service
[root@node2 ~]#systemctl daemon-reload && systemctl enable --now elasticsearch.service 
[root@node3 ~]#systemctl daemon-reload && systemctl enable --now elasticsearch.service 
```

```bash
[root@node1 ~]#curl 10.0.0.201:9200
{
  "name" : "node-1",
  "cluster_name" : "my-application",
  "cluster_uuid" : "y5KEvNeHQWmlBU0gBAoi8A",
  "version" : {
    "number" : "8.17.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
    "build_date" : "2025-02-28T10:07:26.089129809Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
[root@node1 ~]#curl 10.0.0.202:9200
{
  "name" : "node-2",
  "cluster_name" : "my-application",
  "cluster_uuid" : "y5KEvNeHQWmlBU0gBAoi8A",
  "version" : {
    "number" : "8.17.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
    "build_date" : "2025-02-28T10:07:26.089129809Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
[root@node1 ~]#curl 10.0.0.203:9200
{
  "name" : "node-3",
  "cluster_name" : "my-application",
  "cluster_uuid" : "y5KEvNeHQWmlBU0gBAoi8A",
  "version" : {
    "number" : "8.17.3",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
    "build_date" : "2025-02-28T10:07:26.089129809Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### **Elasticsearch** **访问**

```
[root@node1 ~]#curl http://127.0.0.1:9200/_cat
=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
/_cat/component_templates/_cat/ml/anomaly_detectors
/_cat/ml/anomaly_detectors/{job_id}
/_cat/ml/datafeeds
/_cat/ml/datafeeds/{datafeed_id}
/_cat/ml/trained_models
/_cat/ml/trained_models/{model_id}
/_cat/ml/data_frame/analytics
/_cat/ml/data_frame/analytics/{id}
/_cat/transforms
/_cat/transforms/{transform_id}
```

**常用 `/_cat` API 命令**

你可以使用 `curl` 访问这些端点，例如：

1. **查看集群健康状态**

   ```
   curl -XGET http://127.0.0.1:9200/_cat/health?v
   ```

2. **查看所有索引**

   ```
   curl -XGET http://127.0.0.1:9200/_cat/indices?v
   ```

3. **查看节点信息**

   ```
   curl -XGET http://127.0.0.1:9200/_cat/nodes?v
   ```

4. **查看主节点**

   ```
   curl -XGET http://127.0.0.1:9200/_cat/master?v
   ```

5. **查看分片信息**

   ```
   curl -XGET http://127.0.0.1:9200/_cat/shards?v
   ```

6. **查看任务**

   ```
   curl -XGET http://127.0.0.1:9200/_cat/tasks?v
   ```

7. **查看所有别名**

   ```
   curl -XGET http://127.0.0.1:9200/_cat/aliases?v
   ```



```bash
[root@node1 ~]#curl -XGET http://127.0.0.1:9200/_cat/health?v
epoch      timestamp cluster        status node.total node.data shards pri relo init unassign unassign.pri pending_tasks max_task_wait_time active_shards_percent
1742118058 09:40:58  my-application green           3         3      0   0    0    0        0            0             0                  -                100.0%
[root@node1 ~]#curl -XGET http://127.0.0.1:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
10.0.0.203           42          75   0    0.00    0.04     0.12 cdfhilmrstw -      node-3
10.0.0.202           55          75   0    0.99    0.63     0.38 cdfhilmrstw *      node-2
10.0.0.201           42          72   0    1.00    0.60     0.30 cdfhilmrstw -      node-1
```

```bash
#创建索引
[root@node1 ~]#curl -XPUT '127.0.0.1:9200/index1'
{"acknowledged":true,"shards_acknowledged":true,"index":"index1"}
#查看索引
[root@node1 ~]#curl -XGET http://127.0.0.1:9200/_cat/indices?v
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
green  open   index1 tUzZayxvTnaHRM4gNhKr7w   1   1          0            0       454b           227b         227b
[root@node1 ~]#curl '127.0.0.1:9200/index1?pretty'
{
  "index1" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "index1",
        "creation_date" : "1742118254160",
        "number_of_replicas" : "1",
        "uuid" : "tUzZayxvTnaHRM4gNhKr7w",
        "version" : {
          "created" : "8521000"
        }
      }
    }
  }
}

#创建3个分片和2个副本的索引
[root@node1 ~]#curl -X PUT '127.0.0.1:9200/index2' -H 'Content-Type: application/json' -d '
{
  "settings": {
    "index": {
      "number_of_shards": 3,
      "number_of_replicas": 2
    }
  }
}'
{"acknowledged":true,"shards_acknowledged":true,"index":"index2"}

[root@node1 ~]#ls /var/lib/elasticsearch/indices/bwfInNoUR9CGQna3Wsj-8g/
0  1  2  _state

#查看索引
[root@node1 ~]#curl -XGET http://127.0.0.1:9200/_cat/indices?v
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
green  open   index2 582P5TyYQsuJyp_BNx2VVA   3   2          0            0      1.9kb           681b         681b
green  open   index1 tUzZayxvTnaHRM4gNhKr7w   1   1          0            0       498b           249b         249b

#调整副本数为1,但不能调整分片数
[root@node1 ~]#curl -X PUT '127.0.0.1:9200/index2/_settings' -H 'Content-Type: application/json' -d '
{
  "settings": { 
    "number_of_replicas": 1
  }
}'
{"acknowledged":true}

#查看
[root@node1 ~]#curl -XGET http://127.0.0.1:9200/_cat/indices?v
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
green  open   index2 582P5TyYQsuJyp_BNx2VVA   3   1          0            0      1.3kb           703b         703b
green  open   index1 tUzZayxvTnaHRM4gNhKr7w   1   1          0            0       498b           249b         249b

#索引存放路径
[root@node1 ~]#ls /var/lib/elasticsearch/indices/
582P5TyYQsuJyp_BNx2VVA  tUzZayxvTnaHRM4gNhKr7w

#删除索引
[root@node1 ~]#curl -XDELETE http://127.0.0.1:9200/index2
{"acknowledged":true}
```

```bash
[root@node1 ~]#curl -X PUT '127.0.0.1:9200/index3' -H 'Content-Type: application/json' -d '
{
  "settings": {
    "index": {
      "number_of_shards": 3,
      "number_of_replicas": 1
    }
  }
}'
{"acknowledged":true,"shards_acknowledged":true,"index":"index3"}

[root@node1 ~]#curl -XGET http://127.0.0.1:9200/_cat/indices?v
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
green  open   index3 TUWCRD4lQIipZWCsyKWTwA   3   1          0            0       908b           454b         454b
green  open   index2 bwfInNoUR9CGQna3Wsj-8g   3   2          0            0      1.9kb           681b         681b
green  open   index1 tUzZayxvTnaHRM4gNhKr7w   1   1          0            0       498b           249b         249b

[root@node1 ~]#ls /var/lib/elasticsearch/indices/TUWCRD4lQIipZWCsyKWTwA/
1  2  _state

[root@node2 ~]#ls /var/lib/elasticsearch/indices/TUWCRD4lQIipZWCsyKWTwA/
0  2  _state

[root@node3 ~]#ls /var/lib/elasticsearch/indices/TUWCRD4lQIipZWCsyKWTwA/
0  1  _state

```



### **生产故障**

```bash
#故障说明:负载不高，但是有的索引会查询不出来,查看到下面的日志信息
```

```bash
#原因说明:在Elasticsearch中，search.max_open_scroll_context 参数用于限制集群中可以同时打开的 scroll 上下文的数量。这个参数的默认值在不同版本的 Elasticsearch 中可能会有所不同，但根据搜索结果显示，在某些版本的 Elasticsearch 中，默认值可能是 500 或 2000 。这个默认值是为了防止过多的 scroll 上下文同时打开导致资源耗尽。如果需要处理大量的数据，可能需要根据实际情况调整这个参数的值。
```

```bash
#查询默认值
[root@node1 ~]#curl -X GET localhost:9200/_cluster/settings
{"persistent":{},"transient":{}}

#集群只需要在一个节点修改此值
[root@node1 ~]#curl -X PUT "localhost:9200/_cluster/settings" \
-H "Content-Type: application/json" \
-d '{
  "persistent": {
    "search.max_open_scroll_context": 8000
  }, 
  "transient": {
    "search.max_open_scroll_context": 8000
  }
}'
{"acknowledged":true,"persistent":{"search":{"max_open_scroll_context":"8000"}},"transient":{"search":{"max_open_scroll_context":"8000"}}}

#查询
[root@node1 ~]#curl -X GET localhost:9200/_cluster/settings
{"persistent":{"search":{"max_open_scroll_context":"8000"}},"transient":{"search":{"max_open_scroll_context":"8000"}}}

[root@node2 ~]#curl -X GET localhost:9200/_cluster/settings
{"persistent":{"search":{"max_open_scroll_context":"8000"}},"transient":{"search":{"max_open_scroll_context":"8000"}}}

[root@node3 ~]#curl -X GET localhost:9200/_cluster/settings
{"persistent":{"search":{"max_open_scroll_context":"8000"}},"transient":{"search":{"max_open_scroll_context":"8000"}}}
```



### **Python** **脚本：集群健康性检查**

```python
[root@node1 ~]#cat es-cluster-monitor.py 
#!/usr/bin/python3
# coding:utf-8

import json
import subprocess

# 运行 curl 命令获取 Elasticsearch 集群健康状态
cmd = ["curl", "-sXGET", "http://127.0.0.1:9200/_cluster/health?pretty=true"]
try:
    result = subprocess.run(cmd, capture_output=True, text=True, check=True)
    data = result.stdout.strip()

    # 解析 JSON 数据
    es_dict = json.loads(data) if data else {}
    status = es_dict.get("status", "unknown")

    # 判断集群状态
    if status == "green":
        print("OK")
    else:
        print("Not OK")

except subprocess.CalledProcessError as e:
    print(f"Error: Command execution failed: {e}")
except json.JSONDecodeError:
    print("Error: Failed to parse JSON response")
```

```bash
[root@node1 ~]#apt install -y python3-pip
[root@node1 ~]#python3 es-cluster-monitor.py 
OK
```

### **Elasticsearch** **插件**

ES集群状态:

- green 绿色状态:表示集群各节点运行正常，而且没有丢失任何数据，各主分片和副本分片都运行正常
- yellow 黄色状态:表示由于某个节点宕机或者其他情况引起的，node节点无法连接、所有主分片都正常分配,有副本分片丢失，但是还没有丢失任何数据
- red 红色状态:表示由于某个节点宕机或者其他情况引起的主分片丢失及数据丢失,但仍可读取数据和存储

监控下面两个条件都满足才是正常的状态

- 集群状态为 green
- 所有节点都启动

#### **Head** **插件**

![image-20250316181757755](5day-png/36Head插件.png)

#### **Cerebro** **插件**

```
https://github.com/lmenezes/cerebro
```

```bash
[root@ubuntu2404 ~]#ls
cerebro_0.9.4_all.deb
[root@ubuntu2404 ~]#apt install ./cerebro_0.9.4_all.deb 
[root@ubuntu2404 ~]#apt update && apt install openjdk-11-jdk -y
[root@ubuntu2404 ~]#systemctl enable --now cerebro.service
[root@ubuntu2404 ~]#systemctl status cerebro.service
```

```
10.0.0.200:9000

http://10.0.0.201:9200
```

**Docker** **运行**

```
https://hub.docker.com/r/lmenezes/cerebro
```

```bash
docker run -d -p 9000:9000 lmenezes/cerebro:0.8.3
```

红色 red 是数据丢了

黄的 yellow 是副本丢了

绿色 green 健康



### **ES** **节点分类和规划**

#### **节点分类**

Elasticsearch 集群的每个节点的角色有所不同,但都会保存集群状态Cluster State的相关的数据信息

- 节点信息：每个节点名称和地址
- 索引信息：所有索引的名称，配置等

Elasticsearch 节点可以承担以下角色：

- Master 节点

  ES集群中只有一个 Master 节点，用于控制和管理整个集群的操作

  Master 节点负责增删索引,增删节点,分片shard的重新分配

  Master 主要维护Cluster State，包括节点名称,节点连接地址,索引名称和配置信息等

  Master 接受集群状态的变化并推送给所有其它节点,集群中各节点都有一份完整的集群状态信息，都由master node负责维护

  Master 节点不需要涉及到文档级别的变更和搜索等操作

  协调创建索引请求或查询请求，将请求分发到相关的node上。

  当Cluster State有新数据产生后， Master 会将数据同步给其他 Node 节点

  Master节点通过超过一半的节点投票选举产生的

  可以设置node.master: true 指定为是否参与Master节点选举, 默认true 

- Data 节点

  存储数据的节点即为 data 节点

  当创建索引后，索引的数据会存储至某个数据节点

  Data 节点消耗内存和磁盘IO的性能比较大

  配置node.data: true, 就是Data节点，默认为 true,即默认所有节点都是 Data 节点类型

- Ingest 节点

  Ingest 节点是 Elasticsearch 5.0 新增的节点类型和功能。

  如果集群中有大量数据预处理需求（如日志解析、字段提取等），可以引入专门的 Ingest 节点。功能类似logstash

  将 Ingest 节点与 Data 节点分离，避免数据预处理影响数据存储和查询性能。

  负责数据预处理（如管道处理、数据转换等）

  Ingest 节点的基础原理是：节点接收到数据之后，根据请求参数中指定的管道流 id，找到对应的已

  注册管道流，对数据进行处理，然后将处理过后的数据，按照 Elasticsearch 标准的 indexing 流程

  继续运行。

  Ingest 节点开启方式为：在 elasticsearch.yml 中定义：node.ingest: true

- Coordinating 节点(协调)

  当集群规模较大时，建议引入专门的 Coordinating 节点。

  Coordinating 节点不存储数据，仅负责接收客户端请求并分发到其他节点。

  这可以减轻 Data 节点的负载，提高查询性能。

  处理请求的节点即为 coordinating 节点，该节点类型为所有节点的默认角色，不能取消

  coordinating 节点主要将请求路由到正确的节点处理。比如创建索引的请求会由 coordinating 路由到 master 节点处理

  当配置 node.master:false、node.data:false node.ingest: false 则只充当 Coordinating 节点

  Coordinating 节点在 Cerebro 等插件中数据页面不会显示出来

- Machine Learning 节点

  负责机器学习任务（如异常检测等）。

  如果使用 Elasticsearch 的机器学习功能，可以配置专门的 Machine Learning 节点。

  这些节点需要较高的 CPU 和内存资源。

  Machine Learning节点开启方式为：在 elasticsearch.yml 中定义node.ml true (需要 enable x-pack)

- Master-eligible 初始化时有资格选举Master的节点

  集群初始化时有权利参于选举Master角色的节点

  只在集群第一次初始化时进行设置有效，后续配置无效

  由 cluster.initial_master_nodes 配置节点地址

**节点参数配置**

一个节点在默认情况下会同时扮演： master eligible ，data node 和 ingest node

| 节点类型          | 配置参数    | 默认值                       |
| ----------------- | ----------- | ---------------------------- |
| master eligible   | node.master | true                         |
| data              | node.data   | true                         |
| ingest            | node.ingest | true                         |
| coodrinating only | 无          | 设置上面三个参数全部为 false |
| machine loearning | node.ml     | true (需要 enable x-pack)    |

**节点规划**

单一职责的节点: 一个节点只承担一个角色

![image-20250316204859166](5day-png/36elasticsearch.png)

- Dedicated master nodes：负责集群状态（cluster state）的管理

  从高可用 & 避免脑裂的角色出发,一般在生产环境中配置 3 台,一个集群只有 1 台活跃的主节点

  使用低配置的 CPU ,RAM 和磁盘

- Dedicated data nodes: 负责数据存储及处理客户端请求

  使用高配置的 CPU,RAM 和磁盘

- Dedicated ingest nodes: 负责数据处理

  使用高配置的 CPU ; 中等配置的 RAM; 低配置的磁盘

- Dedicate Coordinating Only Node (Client Node)

  配置：将 Master ，Data ，Ingest 都配置成 Flase

  Medium / High CUP; Medium / High RAM;Low Disk

  生产环境中，建议为一些大的集群配置 Coordinating Only Nodes,扮演 Load Balancers。 降低Master 和 Data Nodes 的负载

  负载搜索结果的 Gather / Reduce有时候无法预知客户端会发生怎样的请求大量占用内存的结合操作，一个深度聚合可能引发 OOM

**节点架构**

- 基本部署：

  增减节点，水平扩展

  当磁盘容量无法满足需求时，可以增加数据节点；磁盘读写压力大时，增加数据节点

  ![image-20250316205045840](5day-png/36elasticsearch节点构架1.png)

- 水平扩展：Coordinating Only Node

  当系统中有大量的复杂查询及聚合时候，增加 Coordinating 节点，增加查询的性能

  ![image-20250316205118338](5day-png/36elasticsearch节点构架2.png)

- 读写分离

  ![image-20250316205147811](5day-png/36elasticsearch节点构架3.png)

### **ES** **集群选举**

ES集群的选举是由master-eligble（有资格充当的master节点）发起

当该节点发现当前节点不是master，并且该节点通过ZenDiscovery模块ping其他节点，如果发现超过mininum_master_nodes个节点无法连接master时，就会发起新的选举

选举时,优先选举ClusterStateVersion最大的Node节点，如果ClusterStateVersion相同，则选举Node ID最小的Node

ClusterStateVersion是集群的状态版本号，每一次集群选举ClusterStateVersion都会更新，因此最大的ClusterStateVersion是与原有集群数据最接近或者是相同的，这样就尽可能的避免数据丢失。

Node的ID是在第一次服务启动时随机生成的，直接选用最小ID的Node，主要是为了选举的稳定性，尽量少出现选举不出来的问题。

每个集群中只有一个Master节点

**每个集群中损坏的节点不能超过集群一半以上,否则集群将无法提供服务**

### **ES** **集群分片** **Shard** **和副本** **Replication**

**分片** **Shard**

ES 中存储的数据可能会很大,有时会达到PB级别，单节点的容量和性能可以无法满足

基于容量和性能等原因,可以将一个索引数据分割成多个小的分片

再将每个分片分布至不同的节点,从而实现数据的分布存储,实现性能和容量的水平扩展

在读取时,可以实现多节点的并行读取,提升性能

除此之外,如果一个分片的主机宕机,也不影响其它节点分片的读取

横向扩展即增加服务器，当有新的Node节点加入到集群中时，集群会动态的重新进行均匀分配和负载

例如原来有两个Node节点，每个节点上有3个分片，即共6个分片,如果再添加一个node节点到集群中，集群会动态的将此6个分片分配到这三个节点上，最终每个节点上有2个分片。

7,X 默认每个索引只有一个分片

**副本** **Replication**

将一个索引分成多个数据分片,仍然存在数据的单点问题,可以对每一个分片进行复制生成副本,即备份,实现数据的高可用

ES的分片分为主分片（primary shard）和副本分片（复制replica shard），而且通常分布在不同节点

主分片实现数据读写,副本分片只支持读

在索引中的每个分片只有一个主分片,而对应的副本分片可以有多个,一个副本本质上就是一个主分片的备份

每个分片的主分片在创建索引时自动指定且后续不能人为更改

ES 7.X 默认每个索引只有一个主分片和一个副本分片

**默认分片配置**

默认情况下，elasticsearch将分片相关的配置从配置文件中的属性移除了，可以借助于一个默认的模板接口将索引的分片属性更改成我们想要的分片效果。

```bash 
#配置5分片和1副本
#curl -X PUT 'http://127.0.0.1:9200/_template/template_http_request_record' \
     -H 'Content-Type: application/json' \
     -d '{
           "index_patterns": ["*"],
           "settings": {
             "number_of_shards": 5,
             "number_of_replicas": 1
           }
         }'

#属性解析：
接口地址：_template/template_http_request_record
索引类型：index_patterns
分片数量：number_of_shards
副本数量：number_of_replicas
```

### **数据同步机制**

Elasticsearch主要依赖 Zen Discovery 协议来管理集群中节点的加入和离开，以及选举主节点（master node）。

Zen Discovery是Elasticsearch自带的一个协议，不依赖于任何外部服务。

然而，Elasticsearch对于一致性的处理与传统的一致性协议（如Raft或Paxos）有所不同。它采取了一种“最终一致性”（eventual consistency）的模型。

每个索引在Elasticsearch中被分成多个分片（shard），每个分片都有一个主分片和零个或多个副本分片。

主分片负责处理所有的写操作，并将写操作复制到其副本分片。当主分片失败时，一个副本分片会被提升为新的主分片。

Elasticsearch为了提高写操作的性能，允许在主分片写入数据后立即确认写操作，而不需要等待数据被所有副本分片确认写入。这就意味着，在某些情况下，主分片可能会确认写操作成功，而实际上副本分片还没有完全写入数据。这就可能导致数据在短时间内在主分片和副本分片之间不一致。然而，一旦所有副本分片都确认写入了数据，那么系统就会达到“最终一致性”。

为了保证搜索的准确性，Elasticsearch还引入了一个"refresh"机制，每隔一定时间（默认为1秒）将最新的数据加载到内存中，使其可以被搜索到。这个过程是在主分片和所有副本分片上独立进行的，所以可能存在在短时间内搜索结果在不同分片之间有些许不一致的情况，但随着时间的推移，所有分片上的数据都会达到一致状态。

综上所述，Elasticsearch通过Zen Discovery协议管理节点和选举，通过主分片和副本分片的机制保证数据的最终一致性，并通过"refresh"机制保证数据的搜索准确性。

### **ES** **集群故障转移**

故障转移指的是，当集群中有节点发生故障时，ES集群会进行自动修复

假设由3个节点的ES集群组成,有一个索引index_wang，三个主分片，三个副本分片，如果其中一个节点宕机

ES集群的故障转移流程如下

- 重新选举

  假设当前Master节点 node3 节点宕机,同时也导致 node3 的原有的P0和R2分片丢失

  node1 和 node2 发现 Master节点 node3 无法响应

  过一段时间后会重新发起 master 选举

  比如这次选择 node1 为 新 master 节点；此时集群状态变为 Red 状态

  其实无论选举出的新Master节点是哪个节点，都不影响后续的分片的重新分布结果

- 主分片调整

  新的Master节点 node1 发现在原来在node3上的主分片 P0 丢失

  将 node2 上的 R0 提升为主分片

  此时所有的主分片都正常分配，但0和2分片没有副本分片

  集群状态变为 Yellow状态

- 副本分片调整

  node1 将 P0 和 P2 主分片重新生成新的副本分片 R0、R1，此时集群状态变为 Green

- 后续修复好node3节点后，Master 不会重新选举，但会自动将各个分片重新均匀分配

  保证主分片尽可能分布在每个节点上

  副本分片也尽可能分布不同的节点上

  重新分配的过程需要一段时间才能完成

### **ES** **文档路由**

ES文档是分布式存储，当在ES集群访问或存储一个文档时，由下面的算法决定此文档到底存放在哪个主分片中,再结合集群状态找到存放此主分片的节点主机

```bash
shard = hash(routing) % number_of_primary_shards
hash                     #哈希算法可以保证将数据均匀分散在分片中
routing                  #用于指定用于hash计算的一个可变参数，默认是文档id，也可以自定义
number_of_primary_shards #主分片数
#注意：该算法与主分片数相关，一旦确定后便不能更改主分片，因为主分片数的变化会导致所有分片需要重新分配
```

![image-20250316210024988](5day-png/36elasticsearch分片路由.png)

可以发送请求到集群中的任一节点。每个节点都知道集群中任一文档位置， 每个节点都有能力接收请求,再接将请求转发到真正存储数据的节点上

**ES** **文档创建删除流程**

![image-20250316210315505](5day-png/35es文档创建删除流程.png)

- 客户端向集群中某个节点 Node1 发送新建索引文档或者删除索引文档请求
- Node1节点使用文档的 _id 通过上面的算法确定文档属于分片 0 
- 因为分片 0 的主分片目前被分配在 Node3 上,请求会被转发到 Node3
- Node3 在主分片上面执行创建或删除请求
- Node3 执行如果成功，它将请求并行转发到 Node1 和 Node2 的副本分片上
- Node3 将向协调节点Node1 报告成功
- 协调节点Node1 客户端报告成功。

**ES** **文档读取流程**

可以从主分片或者从其它任意副本分片读取文档 ，读取流程如下图所示 ：

![image-20250316210422095](5day-png/35ES 文档读取流程.png)

- 客户端向集群中某个节点 Node1 发送读取请求
- 节点使用文档的 _id 来确定文档属于分片 0 。分片 0 的主副本分片存在于所有的三个节点上
- 在处理读取请求时，协调节点在每次请求的时候都会通过轮询所有的主副本分片来达到负载均衡，此次它将请求转发到 Node2 
- Node2 将文档返回给 Node1 ，然后将文档返回给客户端

