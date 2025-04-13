# 36、Elastic Stack（ELK）

![image-20250316135504748](5day-png/36、ELK.png)

## **ELK** **概述**

生产中可能会遇到如下场景

- 场景一：页面提示超时了，到底是哪个服务响应慢了呢？
- 场景二：系统当前的处理能力如何，TPS、QPS 指标是多少呢？
- 场景三：线上服务部署在多台机器上，你挨个登录机器去查日志，感觉烦不烦？

需要通过各种监控系统解决以下问题

当今监控天下三分。被划分为基于Metrics（指标）监控、基于 Tracing（调用链）监控、基于Logging（日志） 的监控。

![image-20250316144148510](5day-png/image-20250316144148510.png)

![image-20250407142403849](5day-png/image-20250407142403849.png)

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
#生产建议设置为半数以上
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



### **Elasticsearch** **集群扩容和缩容**

#### **集群扩容**

新加入两个节点node4和node5，变为Data节点

在两个新节点安装 ES

```bash
[root@ubuntu2404 ~]#hostnamectl hostname node4
[root@ubuntu2404 ~]#hostnamectl hostname node5

[root@node4 ~]#apt install ./elasticsearch-8.17.3-amd64.deb 
[root@node5 ~]#apt install ./elasticsearch-8.17.3-amd64.deb 

#修改配置文件
[root@node4 ~]#cat /etc/elasticsearch/elasticsearch.yml
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
node.name: node-4			#node-5 修改
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
#cluster.initial_master_nodes: ["node-1", "node-2"]
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
# generated to configure Elasticsearch security features on 17-03-2025 11:44:31
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
#cluster.initial_master_nodes: ["node4"]

# Allow HTTP API connections from anywhere
# Connections are encrypted and require user authentication
http.host: 0.0.0.0

# Allow other nodes to join the cluster from anywhere
# Connections are encrypted and mutually authenticated
#transport.host: 0.0.0.0

#----------------------- END SECURITY AUTO CONFIGURATION -------------------------

#修改service文件
[root@node4 ~]#cat /lib/systemd/system/elasticsearch.service
[Unit]
Description=Elasticsearch
Documentation=https://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]

LimitMEMLOCK=infinity				#增加这一行

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

#同步到node5
[root@node4 ~]#scp /etc/elasticsearch/elasticsearch.yml 10.0.0.205:/etc/elasticsearch/elasticsearch.yml   
[root@node4 ~]#scp /lib/systemd/system/elasticsearch.service 10.0.0.205:/lib/systemd/system/elasticsearch.service

#启动
[root@node4 ~]#systemctl daemon-reload && systemctl enable --now elasticsearch.service
[root@node5 ~]#systemctl daemon-reload && systemctl enable --now elasticsearch.service
```

![image-20250317195747782](5day-png/36elasticsearch扩容.png)



#### **集群缩容**

从集群中删除两个节点node4和node5，在两个节点按一定的顺序逐个停止服务，即可自动退出集群

**注意：停止服务前，要观察索引的情况，按一定顺序关机，即先关闭一台主机，等数据同步完成后，再关闭第二台主机，防止数据丢失**

```
systemctl stop elasticsearch
```

#### **Data 节点变为Coordinating 节点**

将一个节点node4变为Coordinating 节点

在node4执行下面操作

```bash
[root@node4 ~]#vim /etc/elasticsearch/elasticsearch.yml 

#Elasticsearch 7.x 及以前的版本使用：
node.master: false
#存储数据,默认值为true,此值为false则不存储数据而成为一个路由节点,如果将原有的true改为false,需要先执行 清理数据
node.data: false

#Elasticsearch 8.x 版本修改为：

#表示该节点 没有任何角色，即它不会作为 Master、Data、Ingest、ML 等角色使用。
#仅作为协调节点（coordinating-only node） 的 Elasticsearch 实例
node.roles: []

#让这个节点 仅作为数据节点：
node.roles: [ "data" ]

#让这个节点 仅作为主节点：
node.roles: [ "master" ]

#同时是主节点和数据节点：
node.roles: [ "master", "data" ]

[root@node4 ~]#/usr/share/elasticsearch/bin/elasticsearch-node repurpose
------------------------------------------------------------------------

    WARNING: Elasticsearch MUST be stopped before running this tool.

Node has node.data=true -> no clean up necessary

#重启服务
[root@node4 ~]#systemctl restart elasticsearch

```



### **Elasticsearch** **数据冷热分离**

#### **冷热分离原理**

为了保证Elasticsearch的读写性能，官方建议磁盘使用SSD固态硬盘。然而Elasticsearch要解决的是海量数据的存储和检索问题，海量的数据就意味需要大量的存储空间，如果都使用SSD固态硬盘成本将成为一个很大的问题，这也是制约许多企业和个人使用Elasticsearch的因素之一。为了解决这个问题，Elasticsearch冷热分离架构应运而生。

传统的Elasticsearch集群中所有节点均采用相同的配置，然而Elasticsearch并没有对节点的规格一致性做要求，换而言之就是每个节点可以是任意规格，当然这样做会导致集群各节点性能不一致，影响集群稳定性。但是如果有规则的将集群的节点分成不同类型，部分是高性能的节点用于存储热点数据，部分是性能相对差些的大容量节点用于存储冷数据，却可以一方面保证热数据的性能，另一方面保证冷数据的存储，降低存储成本，这也是Elasticsearch冷热分离架构的基本思想

#### **指定节点的冷热属性**

仅仅将不同的节点设置为不同的规格还不够，为了能明确区分出哪些节点是热节点，哪些节点是冷节点，需要为对应节点打标签

Elasticsearch支持给节点打标签，具体方式是在elasticsearch.yml文件中增加

```bash
node.attr.{attribute}: {value}
#其中attribute为用户自定义的任意标签名，value为该节点对应的该标签的值
```

例如对于冷热分离，可以使用如下设置

```bash
#示例
vim elasticsearch.yml
node.attr.temperature: hot //热节点
node.attr.temperature: warm //冷节点

#验证
GET _cat/nodeattrs?v&h=node,attr,value&s=attr:desc

node       attr       value
node1   temperature     hot
node2   temperature     hot
node3   temperature     warm
node4   temperature     hot
node5   temperature     warm
```

**指定索引冷热属性**

指定数据的冷热属性，来设置和调整数据分布。冷热分离方案中数据冷热分布的基本单位是索引，即指定某个索引为热索引，另一个索引为冷索引。通过索引的分布来实现控制数据分布的目的。

Elasticsearch提供了index shard filtering功能(2.x开始)，该功能在索引配置中提供了如下几个配置

用户可以在创建索引，或后续的任意时刻设置这些配置来控制索引在不同标签节点上的分配动作。

```bash
index.routing.allocation.include.{attribute} #表示索引可以分配在包含多个逗号分隔的值中其中一个的节点上。
index.routing.allocation.require.{attribute} #表示索引要分配在包含索引指定多个逗号分隔的值的节点上,并且多个值都要匹配
index.routing.allocation.exclude.{attribute} #表示索引只能分配在不包含指定多个逗号分隔的值值的节点上。
```

**业务方可以根据实际情况决定索引的冷热属性**

```bash
#对于热数据，索引设置如下
PUT hot_warm_test_index/_settings
{
  "index.routing.allocation.require.temperature": "hot"
}

#查看分片分配,发现分片均分配在热节点上
GET _cat/shards/hot_warm_test_index?v&h=index,shard,prirep,node&s=node

#对于冷数据，索引设置
PUT hot_warm_test_index/_settings
{
  "index.routing.allocation.require.temperature": "warm"
}

#查看分片分配，发现分片均分配到冷节点上
GET _cat/shards/hot_warm_test_index?v&h=index,shard,prirep,node&s=node
```

## Beats 收集数据

虽然利用 logstash 就可以收集日志，功能强大，但由于 Logstash 是基于Java实现，需要在采集日志的主机上安装JAVA环境

logstash运行时最少也会需要额外的500M的以上的内存，会消耗比较多的内存和磁盘空间，可以采有基于Go开发的 Beat 工具代替 Logstash 收集日志，部署更为方便，而且只占用10M左右的内存空间及更小的磁盘空间。

```bash
#官方链接
https://www.elastic.co/cn/beats/

#Github 链接
https://github.com/elastic/beats

#下载链接
https://www.elastic.co/cn/downloads/beats
```

### Beats系列

- filebeat

  用于采集日志和其他数据的轻量型采集器

- metricbate

  轻量型指标数据采集器

- packetbeat

  轻量型网络数据采集器

- Heartbat

  用于运行状态监测的轻量型采集器

```powershell
filebeat:收集日志文件数据。最常用的工具
packetbeat:用于收集网络数据。一般用zabbix实现此功能
metricbeat:从OS和服务收集指标数据，比如系统运行状态、CPU 内存利用率等。
winlogbeat: 从Windows平台日志收集工具。
heartbeat: 定时探测服务是否可用。支持ICMP、TCP 和 HTTP，也支持TLS、身份验证和代理
auditbeat:收集审计日志
Functionbeat:使用无服务器基础架构提供云数据。面向云端数据的无服务器采集器，处理云数据
```

注意: Beats 版本要和 Elasticsearch 相同的版本，否则可能会出错

### **利用** **Metricbeat** **监控性能相关指标**

Metricbeat 可以收集指标数据，比如系统运行状态、CPU、内存利用率等。

生产中一般用 zabbix 等专门的监控系统实现此功能

官方配置说明

```
https://www.elastic.co/guide/en/beats/metricbeat/current/configuring-howto-metricbeat.html
```

```bash
#下载地址
https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/m/metricbeat/metricbeat-8.17.3-amd64.deb
```

#### 部署

注意：安装前建议先修改主机名，确保每个主机名称唯一

```bash
[root@ubuntu2404 ~]#hostnamectl hostname web01
[root@ubuntu2404 ~]#hostnamectl hostname web02

[root@web01 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/m/metricbeat/metricbeat-8.17.3-amd64.deb
[root@web02 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/m/metricbeat/metricbeat-8.17.3-amd64.deb

[root@web01 ~]#dpkg -i metricbeat-8.17.3-amd64.deb 
[root@web02 ~]#dpkg -i metricbeat-8.17.3-amd64.deb

#修改配置文件

[root@web02 ~]#vim /etc/metricbeat/metricbeat.yml
# ---------------------------- Elasticsearch Output ----------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["10.0.0.201:9200","10.0.0.202:9200","10.0.0.203:9200"]		#添加es主机，默认为localhost

#同步到 web01 10.0.0.204
[root@web02 ~]#scp /etc/metricbeat/metricbeat.yml 10.0.0.204:/etc/metricbeat/metricbeat.yml

#启动
[root@web01 ~]#systemctl enable --now metricbeat.service
[root@web02 ~]#systemctl enable --now metricbeat.service
```

**通过** **Kibana** **查看收集的性能指标**

Observability --- 基础设施- invertory 库存

### **利用** **Heartbeat** **监控**

heartbeat 用来定时探测服务是否正常运行。支持也支持TLS、身份验证和代理

官方heartbeat配置文档

```
https://www.elastic.co/guide/en/beats/heartbeat/current/configuring-howto-heartbeat.html
```

#### 部署

注意：安装前建议先修改主机名，确保每个主机名称唯一

```bash
[root@ubuntu2404 ~]#hostnamectl hostname web01
[root@ubuntu2404 ~]#hostnamectl hostname web02
```

```bash
[root@web01 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/h/heartbeat-elastic/heartbeat-8.17.3-amd64.deb
[root@web01 ~]#scp heartbeat-8.17.3-amd64.deb 10.0.0.205:
[root@web01 ~]#dpkg -i heartbeat-8.17.3-amd64.deb 
[root@web02 ~]#dpkg -i heartbeat-8.17.3-amd64.deb 

```

修改配置文件

```bash
#官方文档
https://www.elastic.co/guide/en/beats/heartbeat/current/configuration-heartbeat-options.html
```

```bash
# heartbeat.yml
heartbeat.monitors:
- type: icmp
  id: ping-myhost
  name: My Host Ping
  hosts: ["myhost"]
  schedule: '*/5 * * * * * *'
- type: tcp
  id: myhost-tcp-echo
  name: My Host TCP Echo
  hosts: ["myhost:777"]  # default TCP Echo Protocol
  check.send: "Check"
  check.receive: "Check"
  schedule: '@every 5s'
- type: http
  id: service-status
  name: Service Status
  service.name: my-apm-service-name
  hosts: ["http://localhost:80/service/status"]
  check.response.status: [200]
  schedule: '@every 5s'
heartbeat.scheduler:
  limit: 10
```



```bash
#修改配置文件
[root@web01 ~]#vim /etc/heartbeat/heartbeat.yml 

############################# Heartbeat ######################################

# Define a directory from which to load monitor definitions. Definitions take the form
# of individual yaml files.
heartbeat.config.monitors:
  # Directory + glob pattern to search for configuration files
  path: ${path.config}/monitors.d/*.yml
  # If enabled, heartbeat will periodically check the config.monitors path for changes
  reload.enabled: false
  # How often to check for changes
  reload.period: 5s

# Configure monitors inline
heartbeat.monitors:
- type: http
  # Set enabled to true (or delete the following line) to enable this monitor
  enabled: true
  # ID used to uniquely identify this monitor in Elasticsearch even if the config changes
  id: my-monitor
  # Human readable display name for this service in Uptime UI and elsewhere
  name: My Monitor
  # List of URLs to query
  urls: ["http://localhost"]
  # Configure task schedule
  schedule: '@every 1s'
  # Total test connection and data exchange timeout
  #timeout: 16s
  # Name of corresponding APM service, if Elastic APM is in use for the monitored service.
  #service.name: my-apm-service-name

# Experimental: Set this to true to run heartbeat monitors exactly once at startup
#heartbeat.run_once: true

- type: icmp
  id: ping-myhost
  name: My Host Ping
  hosts: ["10.0.0.205"]
  schedule: '*/5 * * * * * *'
- type: tcp
  id: myhost-tcp-echo
  name: My Host TCP Echo
  hosts: ["10.0.0.204:80"]  # default TCP Echo Protocol
  schedule: '@every 1s'
...
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["10.0.0.201:9200","10.0.0.202:9200","10.0.0.203:9200"]	#修改为es地址


```

**通过** **Kibana** **查看收集的性能指标**

Observability---运行时间--监测 Uptime Monitors

### **利用** **Filebeat** **收集日志**

- INPUT：收集日志
- 转化为json模式
- OUTPUT：ES

Filebeat 是用于转发和集中日志数据的轻量级传送程序。作为服务器上的代理安装，Filebeat监视您指定的日志文件或位置，收集日志事件，并将它们转发到Elasticsearch或Logstash进行索引。

Logstash 也可以直接收集日志,但需要安装JDK并且会占用至少 500M 以上的内存

生产一般使用filebeat代替logstash, 基于go开发,部署方便,重要的是只需要10M多内存,比较节约资源.

filebeat 支持从日志文件,Syslog,Redis,Docker,TCP,UDP,标准输入等读取数据,对数据做简单处理，再输出至Elasticsearch,logstash,Redis,Kafka等

Filebeat的工作方式如下：

- 启动Filebeat时，它将启动一个或多个输入源，这些输入将在为日志数据指定的位置中查找。
- 对于Filebeat所找到的每个日志，Filebeat都会启动收集器harvester进程。
- 每个收集器harvester都读取一个日志以获取新内容，并将新日志数据发送到libbeat
- libbeat会汇总事件并将汇总的数据发送到为Filebeat配置的输出。

```bash
#官方说明
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html
https://www.elastic.co/guide/en/beats/filebeat/current/configuring-howto-filebeat.html
```

![image-20250318110327973](5day-png/36filebeat.png)

#### 部署

```bash
[root@web01 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/f/filebeat/filebeat-8.17.3-amd64.deb
[root@web02 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/f/filebeat/filebeat-8.17.3-amd64.deb

[root@web01 ~]#dpkg -i filebeat-8.17.3-amd64.deb 
[root@web02 ~]#dpkg -i filebeat-8.17.3-amd64.deb 

#二进制文件
[root@web01 ~]#ldd /usr/share/filebeat/bin/filebeat
	linux-vdso.so.1 (0x00007fffb818d000)
	libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007cb4bd690000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007cb4b1bfb000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007cb4b1bf6000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007cb4b1800000)
	/lib64/ld-linux-x86-64.so.2 (0x00007cb4bd6aa000)
[root@web01 ~]#which filebeat 
/usr/bin/filebeat
[root@web01 ~]#ll /usr/share/filebeat/bin/filebeat
-rwxr-xr-x 1 root root 195005248 Feb 28 17:01 /usr/share/filebeat/bin/filebeat*
[root@web01 ~]#ll /usr/bin/filebeat 	#脚本，调用/usr/share/filebeat/bin/filebeat
-rwxr-xr-x 1 root root 335 Feb 28 17:01 /usr/bin/filebeat*
```

#### **从标准输入读取再输出至标准输出**

```
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-stdin.html
```

```
https://www.elastic.co/guide/en/beats/filebeat/current/console-output.html
```

```bash
#增加配置文件
[root@web01 ~]#vim /etc/filebeat/stdin.yml

filebeat.inputs:
- type: stdin
output.console:
  pretty: true

#测试
[root@web01 ~]#filebeat test config -c /etc/filebeat/stdin.yml 
Config OK
#路径也可以不用写
[root@web01 ~]#filebeat test config -c stdin.yml 
Config OK

#启动
[root@web01 ~]#filebeat -e -c stdin.yml.
#输入hello，world
hello,world
#输出下面json格式
{
  "@timestamp": "2025-03-18T04:41:53.145Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "log": {
    "offset": 0,
    "file": {
      "path": ""
    }
  },
  "message": "hello,world",
  "input": {
    "type": "stdin"
  },
  "ecs": {
    "version": "8.0.0"
  },
  "host": {
    "name": "web01"
  },
  "agent": {
    "version": "8.17.3",
    "ephemeral_id": "e91696a5-78ea-4dcd-ac66-56b55580e667",
    "id": "3a55b7b5-828a-46f1-8be6-df4871c6537f",
    "name": "web01",
    "type": "filebeat"
  }
}
```

**解析json格式文本**

```
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-stdin.html#filebeat-input-stdin-config-json
```

默认不会解析json格式，`message`

```json
{"name" : "hzookang", "age" : "20", "phone" : "0123456789"}
{
  "@timestamp": "2025-03-18T04:44:39.467Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "log": {
    "file": {
      "path": ""
    },
    "offset": 0
  },
  "message": "{\"name\" : \"hzookang\", \"age\" : \"20\", \"phone\" : \"0123456789\"}",
  "input": {
    "type": "stdin"
  },
  "host": {
    "name": "web01"
  },
  "agent": {
    "id": "3a55b7b5-828a-46f1-8be6-df4871c6537f",
    "name": "web01",
    "type": "filebeat",
    "version": "8.17.3",
    "ephemeral_id": "e91696a5-78ea-4dcd-ac66-56b55580e667"
  },
  "ecs": {
    "version": "8.0.0"
  }
}
```

```bash
#修改配置文件
[root@web01 ~]#vim /etc/filebeat/stdin.yml
filebeat.inputs:
- type: stdin
  json.keys_under_root: true
output.console:
  pretty: true

#再重新启动
[root@web01 ~]#filebeat -c stdin.yml
```

```json
{"name": "hzookang", "age": 20, "phone": "0123456789"}
{
  "@timestamp": "2025-03-18T04:54:01.242Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "log": {
    "file": {
      "path": ""
    },
    "offset": 0
  },
  "name": "hzookang",
  "age": 20,
  "phone": "0123456789",
  "input": {
    "type": "stdin"
  },
  "ecs": {
    "version": "8.0.0"
  },
  "host": {
    "name": "web01"
  },
  "agent": {
    "version": "8.17.3",
    "ephemeral_id": "241d9465-f222-4c26-a7e3-595290b4776b",
    "id": "3a55b7b5-828a-46f1-8be6-df4871c6537f",
    "name": "web01",
    "type": "filebeat"
  }
}
```

#### **从标准输入读取再输出至** **Json** **格式的文件**

```
https://www.elastic.co/guide/en/beats/filebeat/current/file-output.html
```

```yaml
[root@web01 ~]#vim /etc/filebeat/stdin.yml 
filebeat.inputs:
- type: stdin
  json.keys_under_root: true
output.file:
  path: "/tmp/filebeat"
  filename: "filebeat.log"

```

```bash
[root@web01 ~]#filebeat -c stdin.yml
{"name": "hzookang", "age": 20, "phone": "0123456789"}
```

```yaml
[root@web01 ~]#tail -f /tmp/filebeat/filebeat.log-20250318.ndjson 
{"@timestamp":"2025-03-18T05:03:53.405Z","@metadata":{"beat":"filebeat","type":"_doc","version":"8.17.3"},"name":"hzookang","age":20,"phone":"0123456789","input":{"type":"stdin"},"ecs":{"version":"8.0.0"},"host":{"name":"web01"},"agent":{"version":"8.17.3","ephemeral_id":"21bb8d1c-318f-4843-8853-02ad47a4ad6a","id":"3a55b7b5-828a-46f1-8be6-df4871c6537f","name":"web01","type":"filebeat"},"log":{"offset":0,"file":{"path":""}}}

#安装jq工具

[root@web01 ~]#cat /tmp/filebeat/filebeat.log-20250318.ndjson | jq
{
  "@timestamp": "2025-03-18T05:03:53.405Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "name": "hzookang",
  "age": 20,
  "phone": "0123456789",
  "input": {
    "type": "stdin"
  },
  "ecs": {
    "version": "8.0.0"
  },
  "host": {
    "name": "web01"
  },
  "agent": {
    "version": "8.17.3",
    "ephemeral_id": "21bb8d1c-318f-4843-8853-02ad47a4ad6a",
    "id": "3a55b7b5-828a-46f1-8be6-df4871c6537f",
    "name": "web01",
    "type": "filebeat"
  },
  "log": {
    "offset": 0,
    "file": {
      "path": ""
    }
  }
}

```



#### **从文件读取再输出至标准输出**

**filebeat会将每个文件的读取数据的相关信息记录在/var/lib/filebeat/registry/filebeat/log.json文件中,可以实现日志采集的持续性,而不会重复采集**

- 当日志文件大小发生变化时，filebeat会接着上一次记录的位置继续向下读取新的内容
- 当日志文件大小没有变化，但是内容发生变化，filebeat会将文件的全部内容重新读取一遍

```
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html
```

```yaml
[root@web01 ~]#vim /etc/filebeat/file.yml 
filebeat.inputs:
- type: log
  paths:
    - /var/log/test.log

output.console:
  pretty: true
```

```bash
#运行
[root@web01 ~]#filebeat -c file.yml
#新开终端，写入数据
[root@web01 ~]#echo "this is a test log" > /var/log/test.log
```

```json
{
  "@timestamp": "2025-03-18T05:10:40.179Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "ecs": {
    "version": "8.0.0"
  },
  "host": {
    "name": "web01"
  },
  "agent": {
    "type": "filebeat",
    "version": "8.17.3",
    "ephemeral_id": "c51f2cca-6a76-42dc-8b30-eda8ef803440",
    "id": "3a55b7b5-828a-46f1-8be6-df4871c6537f",
    "name": "web01"
  },
  "log": {
    "file": {
      "path": "/var/log/test.log"
    },
    "offset": 0
  },
  "message": "this is a test log",
  "input": {
    "type": "log"
  }
}
```

```bash
[root@web01 ~]#echo '{"name": "hzookang", "age": 20, "phone": "0123456789"}' >> /var/log/test.log 
```

```json
{
  "@timestamp": "2025-03-18T05:13:55.191Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "agent": {
    "type": "filebeat",
    "version": "8.17.3",
    "ephemeral_id": "c51f2cca-6a76-42dc-8b30-eda8ef803440",
    "id": "3a55b7b5-828a-46f1-8be6-df4871c6537f",
    "name": "web01"
  },
  "log": {
    "offset": 19,
    "file": {
      "path": "/var/log/test.log"
    }
  },
  "message": "{name: hzookang, age: 20, phone: 0123456789}",
  "input": {
    "type": "log"
  },
  "ecs": {
    "version": "8.0.0"
  },
  "host": {
    "name": "web01"
  }
}
```

**解析json格式文本**

```
#修改配置
[root@web01 ~]#vim /etc/filebeat/file.yml 
filebeat.inputs:
- type: log
  json.keys_under_root: true
  paths:
    - /var/log/test.log

output.console:
  pretty: true
```

```bash
[root@web01 ~]#filebeat -c file.yml
```

```bash
[root@web01 ~]#echo '{"name": "hzookang", "age": 20, "phone": "0123456789"}' >> /var/log/test.log
```

```json
{
  "@timestamp": "2025-03-18T05:19:14.584Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "input": {
    "type": "log"
  },
  "agent": {
    "type": "filebeat",
    "version": "8.17.3",
    "ephemeral_id": "f7b634d8-9f7e-466e-907b-0a4b3e0c4ee1",
    "id": "3a55b7b5-828a-46f1-8be6-df4871c6537f",
    "name": "web01"
  },
  "ecs": {
    "version": "8.0.0"
  },
  "host": {
    "name": "web01"
  },
  "log": {
    "file": {
      "path": "/var/log/test.log"
    },
    "offset": 109
  },
  "name": "hzookang",
  "age": 20,
  "phone": "0123456789"
}
```



#### 不会读取以前日志文件

```bash
[root@web01 ~]#cat /var/lib/filebeat/registry/filebeat/log.json 
{"k":"filebeat::logs::native::6160502-64512","v":{"timestamp":[2062190796648,1742275459],"ttl":-1,"type":"log","FileStateOS":{"inode":6160502,"device":64512},"identifier_name":"native","id":"native::6160502-64512","prev_id":"","source":"/var/log/test.log","offset":164}}
[root@web01 ~]#ll /var/log/test.log 
-rw-r--r-- 1 root root 164 Mar 18 13:19 /var/log/test.log
[root@web01 ~]#stat /var/log/test.log
  File: /var/log/test.log
  Size: 164       	Blocks: 8          IO Block: 4096   regular file
Device: 252,0	Inode: 6160502     Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2025-03-18 13:19:14.583205653 +0800
Modify: 2025-03-18 13:19:07.849201659 +0800
Change: 2025-03-18 13:19:07.849201659 +0800
 Birth: 2025-03-18 13:10:39.187397573 +0800
```

**filebeat默认以root身份运行**

```bash
#syslog日志普通用户不可访问
[root@web01 ~]#ll /var/log/
-rw-r-----   1 syslog adm             4601963 Mar 18 13:29 syslog		#日志
-rw-rw-r--   1 root   utmp              13056 Mar 18 13:03 wtmp			#root登录信息
```

#### **利用** **Filebeat** **收集系统日志到** **ELasticsearch**

```bash
[root@web01 ~]#cp /etc/filebeat/filebeat.yml{,.bak}
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
#发送到es收集
filebeat.inputs:
- type: log
  json.keys_under_root: true
  enabled: true           # 开启日志
  paths:
    - /var/log/syslog     # 指定收集的日志文件

#-------------------------- Elasticsearch output ------------------------------
output.elasticsearch:
  hosts: ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  #指定ELK集群任意节点的地址和端口,多个地址容错
```

```bash
[root@web01 ~]#filebeat test config -c /etc/filebeat/filebeat.yml
Config OK

#启动服务
[root@web01 ~]#systemctl restart filebeat.service 
```

Filebeat收集的日志在Elasticsearch中默认生成的索引名称为

```bash
#8.X新版
.ds-filebeat-<版本>-<时间>-<ID>
#旧版
filebeat-<版本>-<时间>-<ID>
```

#### **自定义索引名称收集日志到** **ELasticsearch**

```yaml
[root@web01 ~]#cd /etc/filebeat/filebeat.yml

filebeat.inputs:
- type: log
  json.keys_under_root: true
  enabled: true               # 开启日志
  paths:
    - /var/log/syslog     # 指定收集的日志文件
  include_lines: ['sshd', 'failed', 'password']  # 只过滤指定包含关键字的日志
  #include_lines: ['^ERR', '^WARN'] # 只过滤指定包含关键字的日志
  #exclude_lines: ['Debug']         # 排除包含关键字的日志
  #exclude_files: ['.gz$']          # 排除文件名包含关键字的日志文件

output.elasticsearch:
  hosts: ["10.0.0.201:9200"]       # 指定ES集群服务器地址和端口
  index: "kang-%{[agent.version]}-%{+yyyy.MM.dd}" # 自定义索引名称，8.X的索引为 .ds-kang, agent.version 是 filebeat 添加的元数据字段

# 注意: 8.X版生成的索引名为 .ds-kang-%{[agent.version]}-日期-日期-00000n
# 注意：如果自定义索引名称，没有添加下面三行的配置会导致 filebeat 无法启动，提示错误日志如下
# filebeat:Exiting: setup.template.name 和 setup.template.pattern 必须设置，否则会报错
# systemd: filebeat.service: main process exited, code=exited，status=1/FAILURE

setup.ilm.enabled: false  # 关闭索引生命周期 ILM 功能，默认开启时索引名称只能为 filebeat-*，自定义索引名必须修改为 false
setup.template.name: "kang"  # 定义模板名称，要自定义索引名称，必须指定此项，否则无法启动
setup.template.pattern: "kang-*"  # 定义模板的匹配索引名称，要自定义索引名称，必须指定此项，否则无法启动
setup.template.settings:
  index.number_of_shards: 3
  index.number_of_replicas: 1
```

```bash
[root@web01 ~]#systemctl restart filebeat.service 
```

#### **利用tags收集Nginx的Json格式访问日志和错误日志到Elasticsearch不同的索引**

 生产环境中经常需要获取Web访问用户的信息，比如:访问网站的PV、UV、状态码、用户来自哪个地区，访问时间等

可以通过收集的Nginx的J访问日志实现

默认Nginx的每一次访问生成的访问日志是一行文本,ES没办法直接提取有效信息,不利于后续针对特定信息的分析

可以将Nginx访问日志转换为JSON格式解决这一问题

安装nginx配置访问日志使用json格式





```yaml
http {
...
    ##
    # Logging Settings
    ##
    log_format access_json '{"@timestamp":"$time_iso8601",'
        '"host":"$server_addr",'
        '"clientip":"$remote_addr",'
        '"size":$body_bytes_sent,'
        '"responsetime":$request_time,'
        '"upstreamtime":"$upstream_response_time",'
        '"upstreamhost":"$upstream_addr",'
        '"http_host":"$host",'
        '"uri":"$uri",'
        '"domain":"$host",'
        '"xff":"$http_x_forwarded_for",'
        '"referer":"$http_referer",'
        '"tcp_xff":"$proxy_protocol_addr",'
        '"http_user_agent":"$http_user_agent",'
        '"status":"$status"}';

    #access_log /var/log/nginx/access.log;
    access_log /var/log/nginx/access_json.log access_json;

...
}
```

```bash
[root@web01 ~]#systemctl restart nginx.service 
```

```json
[root@web01 ~]#tail -f /var/log/nginx/access_json.log 
{"@timestamp":"2025-03-18T14:44:15+08:00","host":"10.0.0.204","clientip":"10.0.0.205","size":615,"responsetime":"0.000","upstreamtime":"","upstreamhost":"","http_host":"10.0.0.204","uri":"/index.nginx-debian.html","domain":"10.0.0.204","xff":"","referer":"","tcp_xff":"","http_user_agent":"curl/8.5.0","status":"200"}
```

**修改 Filebeat 配置文件**

```bash
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
```

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access_json.log
    json.keys_under_root: true  # 解析 JSON 日志，使其成为结构化字段
    json.overwrite_keys: true   # 允许 JSON 覆盖默认的 `message` 字段
    tags: ["nginx-access"]      # 指定 tag，分类日志

  - type: log
    enabled: true
    paths:
      - /var/log/nginx/error.log
    tags: ["nginx-error"]

output.elasticsearch:
  hosts: ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]
  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "nginx-access"   # 仅当日志包含 `nginx-access` tag 时写入此索引
    - index: "nginx-error-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "nginx-error"    # 仅当日志包含 `nginx-error` tag 时写入此索引

setup.ilm.enabled: false          # 关闭 ILM，允许自定义索引名称
setup.template.name: "nginx"       # 自定义模板名称，避免冲突
setup.template.pattern: "nginx-*"  # 适配所有 `nginx-*` 命名的索引
setup.template.settings:
  index.number_of_shards: 3
  index.number_of_replicas: 1

```

**利用fields收集Nginx的同一个访问日志中不同响应码的行到Elasticsearch不同的索引**

```bash
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
```

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    include_lines: ['404']  # 仅采集包含 "404" 的日志行
    fields:
      status_code: "404"

  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    include_lines: ['200']  # 仅采集包含 "200" 的日志行
    fields:
      status_code: "200"

  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    include_lines: ['304']  # 仅采集包含 "304" 的日志行
    fields:
      status_code: "304"

# 输出到 Elasticsearch 的配置
output.elasticsearch:
  hosts: ["http://10.0.0.201:9200"]
  indices:
    - index: "myapp-error-404-%{+yyyy.MM.dd}"
      when.equals:
        fields.status_code: "404"

    - index: "myapp-ok-200-%{+yyyy.MM.dd}"
      when.equals:
        fields.status_code: "200"

    - index: "myapp-red-304-%{+yyyy.MM.dd}"
      when.equals:
        fields.status_code: "304"

# 关闭 ILM（索引生命周期管理）
setup.ilm.enabled: false

# 自定义索引模板配置
setup.template:
  enabled: true  # 默认启用
  name: "myapp"
  pattern: "myapp-*"

# 自定义索引模板（格式 2）
setup.template.name: "myapp"    # 定义模板名称，必须指定
setup.template.pattern: "myapp-*"  # 定义模板匹配规则，必须指定
```

#### **利用 Filebeat 收集 Tomat 的 Json 格式的访问日志和错 误日志到 Elasticsearch**

```bash
[root@ubuntu2404 ~]#apt update && apt -y install tomcat10
[root@web01 ~]#vim /etc/tomcat10/server.xml 
        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
####################################删除原来的行添加下面一行###################################
pattern="{&quot;clientip&quot;:&quot;%h&quot;,&quot;ClientUser&quot;:&quot;%l&quot;,&quot;authenticated&quot;:&quot;%u&quot;,&quot;AccessTime&quot;:&quot;%t&quot;,&quot;method&quot;:&quot;%r&quot;,&quot;status&quot;:&quot;%s&quot;,&quot;SendBytes&quot;:&quot;%b&quot;,&quot;Query?string&quot;:&quot;%q&quot;,&quot;partner&quot;:&quot;%{Referer}i&quot;,&quot;AgentVersion&quot;:&quot;%{User-Agent}i&quot;}" />
############################################################################################

```

```bash
#重启服务
[root@web01 ~]#systemctl restart tomcat10.service 
[root@web01 ~]#> /var/log/tomcat10/localhost_access_log.2025-03-18.txt 
```



```bash
#修改filebeat文件
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
```

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/localhost_access_log.* #二进制安装
    - /var/log/tomcat9/localhost_access_log.*        #包安装
    - /var/log/tomcat10/localhost_access_log.*       #包安装    
  json.keys_under_root: true  #默认False会将json数据存储至message，改为true则会独立message外存储
  json.overwrite_keys: false  #设为true,覆盖默认的message字段，使用自定义json格式中的key
  tags: ["tomcat-access"]

- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/catalina.*.log #二进制安装
    - /var/log/tomcat9/catalina.*.log        #包安装
    - /var/log/tomcat10/catalina.*.log       #包安装    
  tags: ["tomcat-error"]

output.elasticsearch:
  hosts: ["10.0.0.201:9200"]        #指定ELK集群服务器地址和端口
  indices:
    - index: "tomcat-access-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "tomcat-access"
    - index: "tomcat-error-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "tomcat-error"

setup.ilm.enabled: false
setup.template.name: "tomcat"
setup.template.pattern: "tomcat-*"
setup.template.settings:
  index.number_of_shards: 3
  index.number_of_replicas: 1
```

```bash
#重启服务
[root@web01 ~]#systemctl restart filebeat.service 
```

#### 利用 Filebeat 收集 Tomat 的多行错误日志到 Elasticsearch

```
https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html
```

```yaml
parsers:
- multiline:
    type: pattern
    pattern: '^\['
    negate: true
    match: after
```

```yaml
multiline.type: pattern
multiline.pattern: '^\['
multiline.negate: true
multiline.match: after
```



| 设置`negate` | 设置`match` | 结果                                               | 例子`pattern: ^b`                                            |
| ------------ | ----------- | -------------------------------------------------- | ------------------------------------------------------------ |
| `false`      | `after`     | 与模式匹配的连续行将附加到前一行不匹配的行。       | ![行 abbcbb 变成“abb”和“cbb”](https://www.elastic.co/guide/en/beats/filebeat/current/images/false-after-multi.png) |
| `false`      | `before`    | 与模式匹配的连续行将添加到下一行不匹配的行前面。   | ![行 bbabbc 变成“bba”和“bbc”](https://www.elastic.co/guide/en/beats/filebeat/current/images/false-before-multi.png) |
| `true`       | `after`     | 与模式不匹配的连续行将附加到前一行匹配的行。       | ![行 bacbde 变成“bac”和“bde”](https://www.elastic.co/guide/en/beats/filebeat/current/images/true-after-multi.png) |
| `true`       | `before`    | 与模式不匹配的连续行将被添加到下一行匹配的行前面。 | ![acbdeb 行变为“acb”和“deb”](https://www.elastic.co/guide/en/beats/filebeat/current/images/true-before-multi.png) |



**修改 Filebeat 配置文件**

```yaml
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/localhost_access_log.* #二进制安装
    #- /var/log/tomcat9/localhost_access_log.*        #包安装
    - /var/log/tomcat10/localhost_access_log.*       #包安装    
  json.keys_under_root: true  #默认False会将json数据存储至message，改为true则会独立message外存储
  json.overwrite_keys: false  #设为true,覆盖默认的message字段，使用自定义json格式中的key
  tags: ["tomcat-access"]

- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/catalina.*.log #二进制安装
    #- /var/log/tomcat9/catalina.*.log        #包安装
    - /var/log/tomcat10/catalina.*.log       #包安装    
  tags: ["tomcat-error"]
  multiline.type: pattern            #此为默认值,可省略
  multiline.pattern: '^[0-3][0-9]-'  #正则表达式匹配以两位,或者为'^\d{2}'
  multiline.negate: true             #negate否定无效
  multiline.match: after
  multiline.max_lines: 5000          #默认只合并500行,指定最大合并5000行

output.elasticsearch:
  hosts: ["10.0.0.201:9200"]        #指定ELK集群服务器地址和端口
  indices:
    - index: "tomcat-access-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "tomcat-access"
    - index: "tomcat-error-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "tomcat-error"

setup.ilm.enabled: false
setup.template.name: "tomcat"
setup.template.pattern: "tomcat-*"
setup.template.settings:
  index.number_of_shards: 3
  index.number_of_replicas: 1

```

```bash
#重启
[root@web01 ~]#systemctl restart filebeat.service
```



#### **利用** **Filebeat** **收集** **Nginx** **日志到** **Redis**

```
https://www.elastic.co/guide/en/beats/filebeat/current/redis-output.html
```

```yaml
output.redis:
  hosts: ["localhost"]
  password: "my_password"
  key: "filebeat"
  db: 0
  timeout: 5
```

```yaml
[root@web01 ~]#cat /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/localhost_access_log.* #二进制安装
    #- /var/log/tomcat9/localhost_access_log.*        #包安装
    - /var/log/tomcat10/localhost_access_log.*       #包安装    
  json.keys_under_root: true  #默认False会将json数据存储至message，改为true则会独立message外存储
  json.overwrite_keys: false  #设为true,覆盖默认的message字段，使用自定义json格式中的key
  tags: ["tomcat-access"]

- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/catalina.*.log #二进制安装
    #- /var/log/tomcat9/catalina.*.log        #包安装
    - /var/log/tomcat10/catalina.*.log       #包安装    
  tags: ["tomcat-error"]
  multiline.type: pattern            #此为默认值,可省略
  multiline.pattern: '^[0-3][0-9]-'  #正则表达式匹配以两位,或者为'^\d{2}'
  multiline.negate: true             #negate否定无效
  multiline.match: after
  multiline.max_lines: 5000          #默认只合并500行,指定最大合并5000行

output.redis:
  hosts: ["10.0.0.205"]
  password: "123456"
  key: "filebeat"
  db: 0
  timeout: 5

```

#### **从标准输入读取再输出至** **Kafka**

```
https://www.elastic.co/guide/en/beats/filebeat/current/kafka-output.html
```

```yaml
output.kafka:
  # initial brokers for reading cluster metadata
  hosts: ["kafka1:9092", "kafka2:9092", "kafka3:9092"]

  # message topic selection + partitioning
  topic: '%{[fields.log_topic]}'
  partition.round_robin:
    reachable_only: false

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
```

**脚本部署kafka集群**

每个kafka上都执行

```bash
[root@ubuntu2404 ~]#cat install_kafka_cluster.sh 
#!/bin/bash
#
#********************************************************************
#Author:            wangxiaochun
#QQ:                29308620
#Date:              2020-06-03
#FileName:          install_kafka.sh  
#URL:               http://www.wangxiaochun.com
#Description:       The test script
#Copyright (C):     2020 All rights reserved
#********************************************************************

#支持在线和离线安装安装

NODE1=10.0.0.211
NODE2=10.0.0.212
NODE3=10.0.0.213

KAFKA_VERSION=3.9.0
#KAFKA_VERSION=3.8.0
#KAFKA_VERSION=3.7.0
#KAFKA_VERSION=3.6.1
#KAFKA_VERSION=3.5.1
#KAFKA_VERSION=3.5.0
#KAFKA_VERSION=3.4.0
#KAFKA_VERSION=3.3.2
#KAFKA_VERSION=3.2.0
#KAFKA_VERSION=-3.0.0

SCALA_VERSION=2.13

KAFKA_URL="https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/${KAFKA_VERSION}/kafka_${SCALA_VERSION}-${KAFKA_VERSION}.tgz"
#KAFKA_URL="https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.8.1/kafka_2.13-2.8.1.tgz"
#KAFKA_URL="https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.7.1/kafka_2.13-2.7.1.tgz"


KAFKA_INSTALL_DIR=/usr/local/kafka


HOST=`hostname -I|awk '{print $1}'`

.  /etc/os-release


color () {
    RES_COL=60
    MOVE_TO_COL="echo -en \\033[${RES_COL}G"
    SETCOLOR_SUCCESS="echo -en \\033[1;32m"
    SETCOLOR_FAILURE="echo -en \\033[1;31m"
    SETCOLOR_WARNING="echo -en \\033[1;33m"
    SETCOLOR_NORMAL="echo -en \E[0m"
    echo -n "$1" && $MOVE_TO_COL
    echo -n "["
    if [ $2 = "success" -o $2 = "0" ] ;then
        ${SETCOLOR_SUCCESS}
        echo -n $"  OK  "    
    elif [ $2 = "failure" -o $2 = "1"  ] ;then 
        ${SETCOLOR_FAILURE}
        echo -n $"FAILED"
    else
        ${SETCOLOR_WARNING}
        echo -n $"WARNING"
    fi
    ${SETCOLOR_NORMAL}
    echo -n "]"
    echo 
}

env () {
    if hostname -I |grep -q $NODE1;then
	    ID=1
	   hostnamectl set-hostname node1
	elif hostname -I |grep -q $NODE2;then
	    ID=2
	   hostnamectl set-hostname node2
	elif hostname -I |grep -q $NODE3;then
	    ID=3
	    hostnamectl set-hostname node3
    else
	    color 'IP地址错误' 1
	    exit
	fi
    cat >> /etc/hosts <<EOF
	
$NODE1   node1
$NODE2   node2
$NODE3   node3

EOF
}

install_jdk() {
    java -version &>/dev/null && { color "JDK 已安装!" 1 ; return;  }
    if command -v yum &>/dev/null ; then
        yum -y install java-1.8.0-openjdk-devel || { color "安装JDK失败!" 1; exit 1; }
    elif command -v apt &>/dev/null ; then
        apt update
        #apt install openjdk-11-jdk -y || { color "安装JDK失败!" 1; exit 1; } 
        #apt install openjdk-8-jdk -y || { color "安装JDK失败!" 1; exit 1; } 
        apt install openjdk-21-jdk -y || { color "安装JDK失败!" 1; exit 1; } 
    else
       color "不支持当前操作系统!" 1
       exit 1
    fi
    java -version && { color "安装 JDK 完成!" 0 ; } || { color "安装JDK失败!" 1; exit 1; } 
}



install_zookeeper() {
	mv ${KAFKA_INSTALL_DIR}/config/zookeeper.properties{,.bak}
    cat > ${KAFKA_INSTALL_DIR}/config/zookeeper.properties <<EOF
tickTime=2000
initLimit=10
syncLimit=5
dataDir=${KAFKA_INSTALL_DIR}/data
clientPort=2181
maxClientCnxns=128
autopurge.snapRetainCount=3
autopurge.purgeInterval=24
server.1=${NODE1}:2888:3888
server.2=${NODE2}:2888:3888
server.3=${NODE3}:2888:3888
EOF
    mkdir -p ${KAFKA_INSTALL_DIR}/data
	
    echo $ID > ${KAFKA_INSTALL_DIR}/data/myid

    cat > ${KAFKA_INSTALL_DIR}/bin/zookeeper-startup.sh <<EOF
#!/bin/bash
nohup ${KAFKA_INSTALL_DIR}/bin/zookeeper-server-start.sh ${KAFKA_INSTALL_DIR}/config/zookeeper.properties   &
EOF
    chmod +x ${KAFKA_INSTALL_DIR}/bin/zookeeper-startup.sh
    cat > /lib/systemd/system/zookeeper.service <<EOF
[Unit]
Description=zookeeper.service
After=network.target

[Service]
Type=forking
ExecStart=${KAFKA_INSTALL_DIR}/bin/zookeeper-startup.sh
ExecStop=${KAFKA_INSTALL_DIR}/bin/zookeeper-server-stop.sh 

[Install]
WantedBy=multi-user.target
EOF
    systemctl daemon-reload
    systemctl enable --now  zookeeper.service
    systemctl is-active zookeeper.service
    if [ $? -eq 0 ] ;then 
        color "zookeeper 安装成功!" 0  
    else 
        color "zookeeper 安装失败!" 1
        exit 1
    fi		

} 
   

install_kafka(){
    if [ ! -f kafka_${SCALA_VERSION}-${KAFKA_VERSION}.tgz ];then
        wget -P /usr/local/src/  --no-check-certificate $KAFKA_URL  || { color  "下载失败!" 1 ;exit ; }
    fi
    tar xf /usr/local/src/${KAFKA_URL##*/}  -C /usr/local/
    ln -s /usr/local/kafka_${SCALA_VERSION}-${KAFKA_VERSION}  ${KAFKA_INSTALL_DIR}
    install_zookeeper
	
    echo PATH=${KAFKA_INSTALL_DIR}/bin:'$PATH' >> /etc/profile
	
    mv ${KAFKA_INSTALL_DIR}/config/server.properties{,.bak}
    cat > ${KAFKA_INSTALL_DIR}/config/server.properties <<EOF
broker.id=$ID
listeners=PLAINTEXT://${HOST}:9092
log.dirs=${KAFKA_INSTALL_DIR}/data
num.partitions=1
log.retention.hours=168
zookeeper.connect=${NODE1}:2181,${NODE2}:2181,${NODE3}:2181
zookeeper.connection.timeout.ms=6000
delete.topic.enable=true
EOF
    	
    cat > ${KAFKA_INSTALL_DIR}/bin/kafka-startup.sh <<EOF
#!/bin/bash
nohup ${KAFKA_INSTALL_DIR}/bin/kafka-server-start.sh  ${KAFKA_INSTALL_DIR}/config/server.properties &
EOF
    chmod +x ${KAFKA_INSTALL_DIR}/bin/kafka-startup.sh
	
    cat > /lib/systemd/system/kafka.service <<EOF
[Unit]                                                                          
Description=Apache kafka
After=network.target

[Service]
Type=forking
ExecStart=${KAFKA_INSTALL_DIR}/bin/kafka-startup.sh
ExecStop=/bin/kill  -TERM \${MAINPID}
Restart=always
RestartSec=20

[Install]
WantedBy=multi-user.target

EOF
    systemctl daemon-reload
    systemctl enable --now kafka.service
    #kafka-server-start.sh -daemon ${KAFKA_INSTALL_DIR}/config/server.properties 
    systemctl is-active kafka.service
    if [ $? -eq 0 ] ;then 
        color "kafka 安装成功!" 0  
    else 
        color "kafka 安装失败!" 1
        exit 1
    fi    
}


env

install_jdk

install_kafka

```

**修改配置**

```yaml
output.kafka:
  hosts: ["10.0.0.201:9092", "10.0.0.202:9092", "10.0.0.203:9092"]
  topic: filebeat-log        #指定kafka的topic
  partition.round_robin:
    reachable_only: true     #true表示只发布到可用的分区，不可用分区就不发布，false 时表示所有分区，如果一个节点down，会block
  required_acks: 1           #如果为0，不启用确认机制，错误消息可能会丢失，1等待写入主分区（默认），-1等待写入副本分区
  compression: gzip
  max_message_bytes: 1000000  #每条消息最大长度，以字节为单位，如果超过将丢弃
```

```yaml
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/localhost_access_log.* #二进制安装
    #- /var/log/tomcat9/localhost_access_log.*        #包安装
    - /var/log/tomcat10/localhost_access_log.*       #包安装    
  json.keys_under_root: true  #默认False会将json数据存储至message，改为true则会独立message外存储
  json.overwrite_keys: false  #设为true,覆盖默认的message字段，使用自定义json格式中的key
  tags: ["tomcat-access"]

- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/catalina.*.log #二进制安装
    #- /var/log/tomcat9/catalina.*.log        #包安装
    - /var/log/tomcat10/catalina.*.log       #包安装    
  tags: ["tomcat-error"]
  multiline.type: pattern            #此为默认值,可省略
  multiline.pattern: '^[0-3][0-9]-'  #正则表达式匹配以两位,或者为'^\d{2}'
  multiline.negate: true             #negate否定无效
  multiline.match: after
  multiline.max_lines: 5000          #默认只合并500行,指定最大合并5000行

output.kafka:
  hosts: ["10.0.0.211:9092", "10.0.0.212:9092", "10.0.0.213:9092"]
  topic: tomcat-log        #指定kafka的topic
  partition.round_robin:
    reachable_only: true     #true表示只发布到可用的分区，不可用分区就不发布，false 时表示所有分区，如果一个节点down，会block
  required_acks: 1           #如果为0，不启用确认机制，错误消息可能会丢失，1等待写入主分区（默认），-1等待写入副本分区
  compression: gzip
  max_message_bytes: 1000000  #每条消息最大长度，以字节为单位，如果超过将丢弃
```

```bash
#重启服务
[root@web01 ~]#systemctl restart filebeat.service 
```

#### **从标准输入读取再输出至** **Logstash**

```yaml
filebeat.inputs:
- type: stdin
  enabled: true

output.logstash:
  hosts: ["10.0.0.104:5044", "10.0.0.105:5044"]
  index: filebeat
  loadbalance: true    #默认为false,只随机输出至一个可用的logstash,设为true,则输出至全部logstash
  worker: 1            #线程数量
  compression_level: 3 #压缩比
```

## **Logstash** **过滤**

### Logstash介绍

![image-20250318173435759](5day-png/36logstash过滤.png)

Logstash 是免费且开放的服务器端数据处理管道，能够从多个来源采集数据，转换数据，然后将数据发

送到您最喜欢的一个或多个“存储库”中

Logstash 是整个ELK中拥有最丰富插件的一个组件,而且支持可以水平伸缩

Logstash 基于 Java 和 Ruby 语言开发

Logstash 官网:

```
https://www.elastic.co/cn/logstash/
```

Logstash 官方说明

```
https://www.elastic.co/guide/en/logstash/current/index.html
https://www.elastic.co/guide/en/logstash/7.6/index.html
```

![image-20250318173719435](5day-png/36logstash过滤2.png)

![image-20250318173729221](5day-png/36logstash过滤1.png)



- 输入 Input：用于日志收集,常见插件: Stdin、File、Kafka、Redis、Filebeat、Http
- 过滤 Filter：日志过滤和转换,常用插件: grok、date、geoip、mutate、useragent
- 输出 Output：将过滤转换过的日志输出, 常见插件: File,Stdout,Elasticsearch,MySQL,Redis,Kafka

**Logstash** **和** **Filebeat** **比较**

- Logstash 功能更丰富,可以支持直接将非Josn 格式的日志统一转换为Json格式,且支持多目标输出,和filebeat相比有更为强大的过滤转换功能
- Logstash 资源消耗更多,不适合在每个需要收集日志的主机上安装

### 部署

**注意: 8.X版本之后的logstash包已经内置了JDK无需安装**

**新版Logstash要求JAVA要求11和17**

```
[root@ubuntu2404 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/l/logstash/logstash-8.17.3-amd64.deb
```

```bash
[root@ubuntu2404 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/l/logstash/logstash-8.17.3-amd64.deb
[root@ubuntu2404 ~]#dpkg -i logstash-8.17.3-amd64.deb 

[root@ubuntu2404 ~]#ll /usr/share/logstash/bin/logstash
-rwxr-xr-x 1 root root 2149 Feb 26 21:39 /usr/share/logstash/bin/logstash*
```

```bash
#常用选项
-e 指定配置内容
-f 指定配置文件，支持绝对路径，如果用相对路径，是相对于/usr/share/logstash/的路径
-t 语法检查
-r 修改配置文件后自动加载生效,注意:有时候修改配置还需要重新启动生效
```

```bash
[root@ubuntu2404 ~]#ln -s /usr/share/logstash/bin/logstash /usr/local/bin/
```

### **Logstash** **使用**

#### **Logstash** **输入** **Input** **插件**

```
https://www.elastic.co/guide/en/logstash/current/input-plugins.html
```

```
input { stdin { type => stdin } } output { stdout { codec => rubydebug } }
input { stdin {} } output { stdout {} }
```

```bash
[root@ubuntu2404 ~]#logstash -e 'input { stdin {} } output { stdout {} }'
#输入
hello,world
#输出
{
       "message" => "hello,world",
      "@version" => "1",
         "event" => {
        "original" => "hello,world"
    },
    "@timestamp" => 2025-03-18T10:00:51.917264629Z,
          "host" => {
        "hostname" => "ubuntu2404.wang.org"
    }
}

{"name" : "hzookang", "age" : "20", "phone" : "0123456789"}
{
       "message" => "{\"name\" : \"hzookang\", \"age\" : \"20\", \"phone\" : \"0123456789\"}",
      "@version" => "1",
         "event" => {
        "original" => "{\"name\" : \"hzookang\", \"age\" : \"20\", \"phone\" : \"0123456789\"}"
    },
    "@timestamp" => 2025-03-18T10:01:50.470120231Z,
          "host" => {
        "hostname" => "ubuntu2404.wang.org"
    }
}

```

```bash
#识别json格式
input { stdin { codec => json } } output { stdout {} }
```

```json
[root@ubuntu2404 ~]#logstash -e 'input { stdin { codec => json } } output { stdout {} }'
{"name" : "hzookang", "age" : "20", "phone" : "0123456789"}
{
           "age" => "20",
         "phone" => "0123456789",
          "host" => {
        "hostname" => "ubuntu2404.wang.org"
    },
          "name" => "hzookang",
    "@timestamp" => 2025-03-18T10:11:16.688465601Z,
         "event" => {
        "original" => "{\"name\" : \"hzookang\", \"age\" : \"20\", \"phone\" : \"0123456789\"}\n"
    },
      "@version" => "1"
}

```

#### **从文件输入**

Logstash 会记录每个文件的读取位置,下次自动从此位置继续向后读取

**每个文件的读取位置记录在下面对应的文件中**



```bash
#主配置文件
[root@ubuntu2404 ~]#ls /etc/logstash/logstash.yml 
/etc/logstash/logstash.yml

#业务配置文件
[root@ubuntu2404 ~]#ls /etc/logstash/conf.d/
```

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/file_to_stdout.conf
[root@ubuntu2404 ~]#cat /etc/logstash/conf.d/file_to_stdout.conf
input {
  file {
    path => "/tmp/kang.*"
    type => "kanglog"              # 添加自定义的type字段,可以用于条件判断,和filebeat中tag功能相似
    exclude => "*.txt"             # 排除不采集数据的文件，使用通配符glob匹配语法
    start_position => "beginning"  # 第一次从头开始读取文件,可以取值为:beginning和end,默认为end,即只从最后尾部读取日志
    stat_interval => "3"           # 定时检查文件是否更新，默认1s
    codec => json                  # 如果文件是Json格式,需要指定此项才能解析,如果不是Json格式而添加此行也不会影响结果
  }
  file {
    path => "/var/log/syslog"
    type => "syslog"
    # start_position => "beginning"
    stat_interval => "3"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}

```

```bash
[root@ubuntu2404 ~]#echo hello > /tmp/kang.log
```

```yaml
{
       "message" => "hello",
      "@version" => "1",
          "host" => {
        "name" => "ubuntu2404.wang.org"
    },
          "type" => "kanglog",
           "log" => {
        "file" => {
            "path" => "/tmp/kang.log"
        }
    },
          "tags" => [
        [0] "_jsonparsefailure"
    ],
    "@timestamp" => 2025-03-18T11:19:00.478814439Z
}
```

```bash
[root@ubuntu2404 ~]#echo '{"name" : "hzookang", "age" : "20", "phone" : "0123456789"}' >> /tmp/kang.log
```

```yaml
{
      "@version" => "1",
          "host" => {
        "name" => "ubuntu2404.wang.org"
    },
         "phone" => "0123456789",
          "name" => "hzookang",
           "age" => "20",
           "log" => {
        "file" => {
            "path" => "/tmp/kang.log"
        }
    },
         "event" => {
        "original" => "{\"name\" : \"hzookang\", \"age\" : \"20\", \"phone\" : \"0123456789\"}"
    },
          "type" => "kanglog",
    "@timestamp" => 2025-03-18T11:21:00.580335178Z
}

```

#### **从** **Http** **请求输入数据**

```
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-http.html
```

```yaml
{
    "@version" => "1",
    "headers" => {
           "request_path" => "/twitter/tweet/1",
            "http_accept" => "*/*",
           "http_version" => "HTTP/1.1",
         "request_method" => "PUT",
              "http_host" => "localhost:8080",
        "http_user_agent" => "curl/7.64.1",
         "content_length" => "5",
           "content_type" => "application/x-www-form-urlencoded"
    },
    "@timestamp" => 2021-05-28T19:27:28.609Z,
    "host" => "127.0.0.1",
    "message" => "hello"
}
```

```yaml
{
    "@version" => "1",
    "user_agent" => {
        "original" => "curl/7.64.1"
    },
    "http" => {
        "method" => "PUT",
        "request" => {
            "mime_type" => "application/x-www-form-urlencoded",
            "body" => {
                "bytes" => "5"
            }
        },
        "version" => "HTTP/1.1"
    },
    "url" => {
          "port" => "8080",
        "domain" => "snmp1",
          "path" => "/twitter/tweet/1"
    },
    "@timestamp" => 2021-05-28T23:32:38.222Z,
    "host" => {
        "ip" => "127.0.0.1"
    },
    "message" => "hello",
}
```

```bash
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_to_stdout.conf
input {
  http {
    port => 6666
    codec => json
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_to_stdout.conf -r
```

```yaml
[root@web02 ~]#curl http://10.0.0.210:6666
ok
[root@web02 ~curl -XPOST -d'test log message' http://10.0.0.210:6666
ok
```

```yaml
{
       "message" => "test log message",
      "@version" => "1",
          "tags" => [
        [0] "_jsonparsefailure"
    ],
    "@timestamp" => 2025-03-18T11:31:41.406122642Z,
          "http" => {
         "method" => "POST",
        "version" => "HTTP/1.1",
        "request" => {
                 "body" => {
                "bytes" => "16"
            },
            "mime_type" => "application/x-www-form-urlencoded"
        }
    },
           "url" => {
          "path" => "/",
          "port" => 6666,
        "domain" => "10.0.0.210"
    },
    "user_agent" => {
        "original" => "curl/8.5.0"
    },
          "host" => {
        "ip" => "10.0.0.205"
    }
}
```

```bash
[root@web02 ~]#curl -XPOST -d'{"name" : "hzookang", "age" : "20", "phone" : "0123456789"}' http://10.0.0.210:6666
ok
```

```yaml
{
      "@version" => "1",
          "host" => {
        "ip" => "10.0.0.205"
    },
    "@timestamp" => 2025-03-18T11:33:43.409857299Z,
         "phone" => "0123456789",
           "age" => "20",
          "name" => "hzookang",
          "http" => {
         "method" => "POST",
        "version" => "HTTP/1.1",
        "request" => {
                 "body" => {
                "bytes" => "59"
            },
            "mime_type" => "application/x-www-form-urlencoded"
        }
    },
           "url" => {
          "path" => "/",
          "port" => 6666,
        "domain" => "10.0.0.210"
    },
    "user_agent" => {
        "original" => "curl/8.5.0"
    },
         "event" => {
        "original" => "{\"name\" : \"hzookang\", \"age\" : \"20\", \"phone\" : \"0123456789\"}"
    }
}
```

#### **从** **Filebeat** **输入数据**

```yaml
filebeat.inputs:
- type: log
  enabled: true                        # 开启日志
  paths:
    - /var/log/nginx/access_json.log      # 指定收集的日志文件
  json.keys_under_root: true
  tags: ["nginx-access"]

- type: log
  enabled: true             # 开启日志
  paths:
    - /var/log/syslog            # 指定收集的日志文件
  fields:
    logtype: "syslog"      # 添加自定义字段logtype

# output.console:
#  pretty: true
#  enable: true

output.logstash:
  hosts: ["10.0.0.104:5044"]  # 指定Logstash服务器的地址和端口
```

```bash
#修改filebeat配置文件
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/localhost_access_log.* #二进制安装
    #- /var/log/tomcat9/localhost_access_log.*        #包安装
    - /var/log/tomcat10/localhost_access_log.*       #包安装    
  json.keys_under_root: true  #默认False会将json数据存储至message，改为true则会独立message外存储
  json.overwrite_keys: false  #设为true,覆盖默认的message字段，使用自定义json格式中的key
  tags: ["tomcat-access"]

- type: log
  enabled: true
  paths:
    #- /usr/local/tomcat/logs/catalina.*.log #二进制安装
    #- /var/log/tomcat9/catalina.*.log        #包安装
    - /var/log/tomcat10/catalina.*.log       #包安装    
  tags: ["tomcat-error"]
  multiline.type: pattern            #此为默认值,可省略
  multiline.pattern: '^[0-3][0-9]-'  #正则表达式匹配以两位,或者为'^\d{2}'
  multiline.negate: true             #negate否定无效
  multiline.match: after
  multiline.max_lines: 5000          #默认只合并500行,指定最大合并5000行

output.logstash:
  hosts: ["10.0.0.210:5044"]  # 指定Logstash服务器的地址和端口

```

```bash
#修改logstash配置文件
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/filebeat_to_stdout.conf
input {
  beats {
    port => 5044
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/filebeat_to_stdout.conf -r
```

#### **从** **Redis** **输入数据**

支持由多个 Logstash 从 Redis 读取日志,提高性能

Logstash 从 Redis 收集完数据后,将删除对应的列表Key

```
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-redis.html
https://www.elastic.co/guide/en/logstash/7.6/plugins-inputs-redis.html
```

```bash
[root@web02 ~]#redis-cli -a 123456
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> keys *
1) "filebeat"
```

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/redis_to_stdout.conf

input {
  redis {
    host => '10.0.0.205'
    port => "6379"
    password => "123456"
    db => "0"
    data_type => 'list'
    key => "filebeat"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/redis_to_stdout.conf -r
```

```bash
#此时从redis取走数据，redis中没有数据
127.0.0.1:6379> keys *
(empty array)
```

#### **从** **Kafka** **输入数据**

官方链接

```
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-kafka.html
https://www.elastic.co/guide/en/logstash/7.6/plugins-inputs-kafka.html
```

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/kakfa_to_stdout.conf

input {
  kafka {
    bootstrap_servers => "10.0.0.211:9092,10.0.0.212:9092,10.0.0.213:9092"
    group_id => "logstash" # 多个logstash的group_id如果不同，将实现消息共享（发布者/订阅者模式），如果相同（建议使用），则消息独占（生产者/消费者模式）
    # topics => ["nginx-accesslog","nginx-errorlog"]
    topics => ["tomcat-log"]
    codec => "json"
    auto_offset_reset => "earliest"	#如果有已提交的 offset，就从最早的消息开始消费（即 从头开始消费），不定义，则从现在开始消费
    consumer_threads => 8
  }
}

output {
  stdout {
    codec => rubydebug
  }
}

```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/kakfa_to_stdout.conf -r
```

#### **从** **syslog** **输入数据**

```yaml
input {
  syslog {
    host => "0.0.0.0"
    port => "514"        # 指定监听的UDP/TCP端口，注意普通用户无法监听此端口
    type => "syslog"
    codec => "json"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
#测试访问，json无法解析
```

#### **从**TCP/UDP **输入数据**

```
input {
  tcp {
    port => "9527"
    host => "0.0.0.0"
    type => "tcplog"      # 可用于判断输出
    codec => json         # 如果是json格式,必须加此行,如果不是Json格式,却添加此行也不会影响结果
    mode => "server"      # 默认值,可省略
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```
#TCP访问
[root@web01 ~]#nc 10.0.0.106 9527
this is a test log
{"name" : "wangxiaochun", "age" : "18", "phone" : "0123456789"}

{
  "@timestamp": "2024-10-29T03:37:42.556704832Z",
  "type": "tcplog",
  "tags": [
    "_jsonparsefailure"
  ],
  "message": "this is a test log",
  "@version": "1"
}

{
  "age": "18",
  "phone": "0123456789",
  "@timestamp": "2024-10-29T03:37:53.312579979Z",
  "name": "wangxiaochun",
  "type": "tcplog",
  "@version": "1"
}
```

### **Logstash** **过滤** **Filter** **插件**

数据从源传输到存储库的过程中，Logstash 过滤器能够解析各个事件，识别已命名的字段以构建结构，并将它们转换成通用格式，以便进行更强大的分析和实现商业价值。

Logstash 能够动态地转换和解析数据，不受格式或复杂度的影响

常见的 Filter 插件:

- 利用 Grok 从非结构化数据中转化为结构数据JSON格式
- 利用 GEOIP 根据 IP 地址找出对应的地理位置坐标
- 利用 Date 可以将日志中的指定的日期字符串对应的源字段生成新的目标字段。
- 利用 useragent 从请求中分析操作系统、设备类型
- 利用 Mutate 从请求中整理字段

官方链接

```
https://www.elastic.co/guide/en/logstash/current/filter-plugins.html
https://www.elastic.co/guide/en/logstash/7.6/filter-plugins.html
```

#### **Grok** **插件**

```
https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
```

参考网站

```
https://www.elastic.co/cn/blog/do-you-grok-grok
https://www.alibabacloud.com/help/zh/sls/user-guide/grok-patterns
https://github.com/elastic/logstash/blob/v1.4.2/patterns/grok-patterns
http://grokdebug.herokuapp.com/
http://grokdebug.herokuapp.com/discover?#
https://grokconstructor.appspot.com/do/automatic
https://tongyi.aliyun.com/
https://yiyan.baidu.com/
https://www.doubao.com/chat/
```

**nginx日志**

```
127.0.0.1 - - [18/Mar/2025:10:55:28 +0800] "GET / HTTP/1.1" 200 615 "-" "Elastic-Heartbeat/8.17.3 (linux; amd64; 3747d0eb6c26247477dd62ca51535cff8d6338b7; 2025-02-28 08:56:24 +0000 UTC)"
```

**Grok**

```
%{IPORHOST:client_ip} - - \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status} %{NUMBER:bytes} "%{DATA:referrer}" "%{DATA:user_agent}"
```

**日志格式**

```
58.250.250.21 - - [14/Jul/2020:15:07:27 +0800] "GET /wp-content/plugins/akismet/_inc/form.js?ver=4.1.3 HTTP/1.1" 200 330 "http://www.wangxiaochun.com/?p=117" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "-"
```

**Grok格式**

```
%{IPORHOST:client_ip} - - \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status} %{NUMBER:bytes} "%{URI:referrer}" "%{DATA:user_agent}" "%{DATA:extra}"
```

##### **使用** **grok pattern** **将** **Nginx** **日志格式化为** **json** **格式**

```
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_stdout.conf
input {
  http {
    port => 6666
  }
}

filter {
  # 将nginx日志格式化为json格式
  grok {
    match => {
      "message" => "%{COMBINEDAPACHELOG}"  # 将message字段转化为指定的Json格式
    }
  }
}

output {
  stdout {
    codec => rubydebug
  }
}

```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_grok_stdout.conf -r
```

```
[INFO ] 2025-03-18 21:09:23.957 [[main]-pipeline-manager] javapipeline - Pipeline started {"pipeline.id"=>"main"}
```

```json
{
       "message" => "[INFO ] 2025-03-18 21:09:23.957 [[main]-pipeline-manager] javapipeline - Pipeline started {\"pipeline.id\"=>\"main\"}",
      "@version" => "1",
           "url" => {
          "port" => 6666,
        "domain" => "10.0.0.210",
          "path" => "/"
    },
    "user_agent" => {
        "original" => "insomnia/11.0.0"
    },
    "@timestamp" => 2025-03-18T13:12:19.431165517Z,
          "tags" => [
        [0] "_jsonparsefailure",
        [1] "_grokparsefailure"
    ],
          "host" => {
        "ip" => "10.0.0.1"
    },
          "http" => {
        "request" => {
                 "body" => {
                "bytes" => "113"
            },
            "mime_type" => "application/json"
        },
        "version" => "HTTP/1.1",
         "method" => "POST"
    }
}

```

##### 直接将nginx的访问日志转化为Json格式

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/nginx_grok_stdout.conf

input {
  file {
    path => "/var/log/nginx/access.log"
    type => "nginx-accesslog"
    start_position => "beginning"
    stat_interval => "3"
  }
}

filter {
  # 将nginx日志格式化为json格式
  grok {
    match => {
      "message" => '"%{IPORHOST:client_ip} - - \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status} %{NUMBER:bytes} "%{DATA:referrer}" "%{DATA:user_agent}"'  # 将message字段转化为指定的Json格式
    }
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/nginx_grok_stdout.conf -r
```

```json
{
        "@version" => "1",
         "message" => "10.0.0.210 - - [18/Mar/2025:21:18:32 +0800] \"GET / HTTP/1.1\" 200 615 \"-\" \"curl/8.5.0\"",
       "client_ip" => "10.0.0.210",
           "event" => {
        "original" => "10.0.0.210 - - [18/Mar/2025:21:18:32 +0800] \"GET / HTTP/1.1\" 200 615 \"-\" \"curl/8.5.0\""
    },
      "@timestamp" => 2025-03-18T13:19:56.062995216Z,
          "status" => "200",
        "referrer" => "-",
      "user_agent" => "curl/8.5.0",
         "request" => "/",
    "http_version" => "1.1",
             "log" => {
        "file" => {
            "path" => "/var/log/nginx/access.log"
        }
    },
            "type" => "nginx-accesslog",
       "timestamp" => "18/Mar/2025:21:18:32 +0800",
            "host" => {
        "name" => "ubuntu2404.wang.org"
    },
           "bytes" => "615",
          "method" => "GET"
}

```

#### **GeoIP** **插件**

geoip 根据 ip 地址提供的对应地域信息，比如:经纬度,国家,城市名等,以方便进行地理数据分析

```bash
#查询数据库路径
[root@ubuntu2404 ~]#find / -name GeoLite2-City.mmdb
/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/logstash-filter-geoip-7.3.1-java/vendor/GeoLite2-City.mmdb
```

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_geoip_stdout.conf

input {
  # beats {
  #   port => 5044
  # }
  http {
    port => 6666
    codec => "json"
  }
}

filter {
  # 将nginx日志格式化为json格式
  grok {
    match => {
      "message" => "%{COMBINEDAPACHELOG}"
    }
  }

  # 以上面提取clientip字段为源,获取地域信息
  geoip {
    # database => "/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/logstash-filter-geoip-7.3.0-java/vendor/GeoLite2-City.mmdb" # 8.15.X版本必须指定数据库文件路径，7.X可以不指定
    database => "/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/logstash-filter-geoip-7.3.1-java/vendor/GeoLite2-City.mmdb"
    # source => "clientip" # 7.X版本指定源IP的所在字段
    source => "[source][address]" # 8.X版本变化,结合grok插件
    target => "geoip"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_geoip_stdout.conf -r
```

```
49.7.99.142 - - [17/Mar/2025:16:21:57 +0800] "GET /wp-content/uploads/2023/01/24506AEA-0556-4BBD-800F-A773D637FBB8.jpg HTTP/1.1" 200 275868 "http://www.wangxiaochun.com:8888/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0" "-"
```

```json
{
       "message" => "49.7.99.142 - - [17/Mar/2025:16:21:57 +0800] \"GET /wp-content/uploads/2023/01/24506AEA-0556-4BBD-800F-A773D637FBB8.jpg HTTP/1.1\" 200 275868 \"http://www.wangxiaochun.com:8888/\" \"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0\" \"-\"",
           "url" => {
          "domain" => "10.0.0.210",
            "path" => "/",
        "original" => "/wp-content/uploads/2023/01/24506AEA-0556-4BBD-800F-A773D637FBB8.jpg",
            "port" => 6666
    },
      "@version" => "1",
          "host" => {
        "ip" => "10.0.0.1"
    },
          "http" => {
         "request" => {
             "referrer" => "http://www.wangxiaochun.com:8888/",
            "mime_type" => "application/json",
                 "body" => {
                "bytes" => "307"
            },
               "method" => "GET"
        },
          "method" => "POST",
         "version" => [
            [0] "HTTP/1.1",
            [1] "1.1"
        ],
        "response" => {
            "status_code" => 200,
                   "body" => {
                "bytes" => 275868
            }
        }
    },
    "@timestamp" => 2025-03-18T13:31:11.915557237Z,
     "timestamp" => "17/Mar/2025:16:21:57 +0800",
        "source" => {
        "address" => "49.7.99.142"
    },
          "tags" => [
        [0] "_jsonparsefailure"
    ],
         "geoip" => {
        "geo" => {
             "region_iso_code" => "CN-BJ",
            "country_iso_code" => "CN",
                 "region_name" => "Beijing",
                   "city_name" => "Beijing",
                "country_name" => "China",
                    "timezone" => "A sia/Shanghai",
                    "location" => {
                "lon" => 116.3889,
                "lat" => 39.9288
            },
              "continent_code" => "AS"
        },
         "ip" => "49.7.99.142"
    },
    "user_agent" => {
        "original" => [
            [0] "insomnia/11.0.0",
            [1] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0"
        ]
    }
}

```



#### **Date** **插件**

Date插件可以将日志中的指定的日期字符串对应的源字段生成新的目标字段。

然后替换@timestamp 字段(此字段默认为当前写入logstash的时间而非日志本身的时间)或指定的其他字段

```
match    #类型为数组，用于指定需要使用的源字段名和对应的时间格式
target   #类型为字符串，用于指定生成的目标字段名，默认是 @timestamp
timezone #类型为字符串，用于指定时区域
```

官方说明

```
https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html
```

时区格式参考

```
http://joda-time.sourceforge.net/timezones.html
```

##### 利用源字段timestamp生成新的字段名access_time

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_date_stdout.conf

input {
  http {
    port => 6666
  }
}

filter {
  # 将nginx日志格式化为json格式
  grok {
    match => {
      "message" => "%{COMBINEDAPACHELOG}"
    }
  }

  # 解析源字段timestamp的date日期格式: 14/Jul/2020:15:07:27 +0800
  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
    target => "access_time"        # 将时间写入新生成的access_time字段，源字段仍保留
    # target => "@timestamp"       # 将时间覆盖原有的@timestamp字段
    timezone => "Asia/Shanghai"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_grok_date_stdout.conf -r
```

```
49.7.99.142 - - [17/Mar/2025:16:21:57 +0800] "GET /wp-content/uploads/2023/01/24506AEA-0556-4BBD-800F-A773D637FBB8.jpg HTTP/1.1" 200 275868 "http://www.wangxiaochun.com:8888/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0" "-"
```

```json
{
        "message" => "49.7.99.142 - - [17/Mar/2025:16:21:57 +0800] \"GET /wp-content/uploads/2023/01/24506AEA-0556-4BBD-800F-A773D637FBB8.jpg HTTP/1.1\" 200 275868 \"http://www.wangxiaochun.com:8888/\" \"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0\" \"-\"",
     "user_agent" => {
        "original" => [
            [0] "insomnia/11.0.0",
            [1] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36 Edg/134.0.0.0"
        ]
    },
         "source" => {
        "address" => "49.7.99.142"
    },
      "timestamp" => "17/Mar/2025:16:21:57 +0800",
    "access_time" => 2025-03-17T08:21:57.000Z,
            "url" => {
          "domain" => "10.0.0.210",
            "port" => 6666,
            "path" => "/",
        "original" => "/wp-content/uploads/2023/01/24506AEA-0556-4BBD-800F-A773D637FBB8.jpg"
    },
           "http" => {
          "method" => "POST",
        "response" => {
            "status_code" => 200,
                   "body" => {
                "bytes" => 275868
            }
        },
         "version" => [
            [0] "HTTP/1.1",
            [1] "1.1"
        ],
         "request" => {
               "method" => "GET",
                 "body" => {
                "bytes" => "307"
            },
            "mime_type" => "application/json",
             "referrer" => "http://www.wangxiaochun.com:8888/"
        }
    },
     "@timestamp" => 2025-03-18T13:39:21.235373390Z,
           "tags" => [
        [0] "_jsonparsefailure"
    ],
       "@version" => "1",
           "host" => {
        "ip" => "10.0.0.1"
    }
}

```

#### **Useragent** **插件**

```
111.13.118.64 - - [16/Mar/2025:17:13:14 +0800] "GET /?paged=2 HTTP/1.1" 301 5 "-" "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1" "-"
```

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_useragent_stdout.conf
input {
  http {
    port => 6666
  }
}

filter {
  # 将nginx日志格式化为json格式
  grok {
    match => {
      "message" => "%{COMBINEDAPACHELOG}"
    }
  }

  # 解析date日期如: 10/Dec/2020:10:40:10 +0800
  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
    target => "@timestamp"
    # target => "access_time"
    timezone => "Asia/Shanghai"
  }

  # 提取agent字段，进行解析
  useragent {
    # source => "agent"       # 7.X指定从哪个字段获取数据
    source => "message"        # 8.X指定从哪个字段获取数据
    # source => "[user_agent][original]" # 8.X指定从哪个字段获取数据,直接分析日志文件，curl或insomnia此方式不成功
    # source => "[user_agent][original][1]" # 8.X指定从哪个字段获取数据，如果利用insomnia工具发送http请求，需要加[1]
    target => "useragent" # 指定生成新的字典类型的字段的名称，包括os，device等内容
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_grok_useragent_stdout.conf-r
```

```
111.13.118.64 - - [16/Mar/2025:17:13:14 +0800] "GET /?paged=2 HTTP/1.1" 301 5 "-" "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1" "-"
```

```json
{
       "message" => "111.13.118.64 - - [16/Mar/2025:17:13:14 +0800] \"GET /?paged=2 HTTP/1.1\" 301 5 \"-\" \"Mozilla/5.0 (iPhone; CPU iPhone OS 17_0_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1\" \"-\"\n",
    "@timestamp" => 2025-03-16T09:13:14.000Z,
          "host" => {
        "ip" => "10.0.0.1"
    },
           "url" => {
            "port" => 6666,
            "path" => "/",
        "original" => "/?paged=2",
          "domain" => "10.0.0.210"
    },
          "http" => {
        "response" => {
            "status_code" => 301,
                   "body" => {
                "bytes" => 5
            }
        },
          "method" => "POST",
         "version" => [
            [0] "HTTP/1.1",
            [1] "1.1"
        ],
         "request" => {
               "method" => "GET",
                 "body" => {
                "bytes" => "226"
            },
            "mime_type" => "application/json"
        }
    },
     "timestamp" => "16/Mar/2025:17:13:14 +0800",
      "@version" => "1",
        "source" => {
        "address" => "111.13.118.64"
    },
          "tags" => [
        [0] "_jsonparsefailure"
    ],
    "user_agent" => {
        "original" => [
            [0] "insomnia/11.0.0",
            [1] "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1"
        ]
    },
     "useragent" => {
           "name" => "Mobile Safari",
         "device" => {
            "name" => "iPhone"
        },
        "version" => "17.0",
             "os" => {
               "name" => "iOS",
               "full" => "iOS 17.0.3",
            "version" => "17.0.3"
        }
    }
}

```

#### **Mutate** **插件**

官方链接

```
https://www.elastic.co/guide/en/logstash/master/plugins-filters-mutate.html
https://www.elastic.co/guide/en/logstash/7.6/plugins-filters-mutate.html
```

Mutate 插件主要是对字段进行、类型转换、删除、替换、更新等操作,可以使用以下函数

```
remove_field    #删除字段
split           #字符串切割,相当于awk取列
add_field       #添加字段
convert         #类型转换，支持的数据类型：integer，integer_eu，float，float_eu，
string，boolean
gsub            #字符串替换
rename          #字符串改名
lowercase       #转换字符串为小写
```

##### **remove_field** **删除字段**

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_mutate_remove_field_stdout.conf

input {
  http {
    port => 6666
  }
}

filter {
  # 将nginx日志格式化为json格式
  grok {
    match => {
      "message" => "%{COMBINEDAPACHELOG}"
    }
  }

  # 解析date日期如: 10/Dec/2020:10:40:10 +0800
  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
    target => "@timestamp"
    # target => "access_time"
    timezone => "Asia/Shanghai"
  }

  # mutate 删除指定字段的操作
  mutate {
    # remove_field => ["headers","message", "agent"] #7.X
    remove_field => ["timestamp", "message", "http"] #8.X
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_grok_mutate_remove_field_stdout.conf -r
```

```
111.13.118.64 - - [16/Mar/2025:17:13:14 +0800] "GET /?paged=2 HTTP/1.1" 301 5 "-" "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1" "-"
```

```json
{
    "@timestamp" => 2025-03-16T09:13:14.000Z,
    "user_agent" => {
        "original" => [
            [0] "insomnia/11.0.0",
            [1] "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1"
        ]
    },
      "@version" => "1",
        "source" => {
        "address" => "111.13.118.64"
    },
           "url" => {
        "original" => "/?paged=2",
            "port" => 6666,
          "domain" => "10.0.0.210",
            "path" => "/"
    },
          "host" => {
        "ip" => "10.0.0.1"
    },
          "tags" => [
        [0] "_jsonparsefailure"
    ]
}
```

##### **Split** **切割**

mutate 中的 split 字符串切割，指定字符做为分隔符，切割的结果用于生成新的列表元素

示例: 1000|提交订单|2020-01-08 09:10:21

范例: split 切割字符串取列

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_mutate_split_stdout.conf

input {
  http {
    port => 6666
  }
}

filter {
  # mutate 切割操作
  mutate {
    # 字段分隔符
    split => { "message" => "|" } # 将message字段按 | 分割成名称message列表中多个列表元素
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```
1000|提交订单|2020-01-08 09:10:21
```

```json
{
       "message" => [
        [0] "1000",
        [1] "提交订单",
        [2] "2020-01-08 09:10:21"
    ],
    "user_agent" => {
        "original" => "insomnia/11.0.0"
    },
      "@version" => "1",
          "host" => {
        "ip" => "10.0.0.1"
    },
           "url" => {
          "port" => 6666,
        "domain" => "10.0.0.210",
          "path" => "/"
    },
          "http" => {
         "method" => "POST",
        "version" => "HTTP/1.1",
        "request" => {
                 "body" => {
                "bytes" => "37"
            },
            "mime_type" => "application/json"
        }
    },
    "@timestamp" => 2025-03-18T14:05:55.127250904Z,
          "tags" => [
        [0] "_jsonparsefailure"
    ]
}

```



##### **add_field** **添加字段**

```
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_mutate_add_field_stdout.conf

input {
  http {
    port => 6666
  }
}

filter {
  # mutate 切割操作
  mutate {
    # 字段分隔符
    split => { "message" => "|" }

    # 添加字段,将message的列表的第0个元素添加字段名user_id
    add_field => {
      "user_id" => "%{[message][0]}"
      "action" => "%{[message][1]}"
      "time" => "%{[message][2]}"
    }

    # 添加字段做索引名
    # add_field => {"[@metadata][target_index]" => "app-%{+YYY.MM.dd}"}

    # 删除无用字段
    remove_field => ["headers", "message"]
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_grok_mutate_add_field_stdout.conf -r
```

```
1000|提交订单|2020-01-08 09:10:21
```

```json
{
          "http" => {
         "method" => "POST",
        "request" => {
                 "body" => {
                "bytes" => "37"
            },
            "mime_type" => "application/json"
        },
        "version" => "HTTP/1.1"
    },
          "time" => "2020-01-08 09:10:21",
    "user_agent" => {
        "original" => "insomnia/11.0.0"
    },
          "host" => {
        "ip" => "10.0.0.1"
    },
           "url" => {
          "port" => 6666,
          "path" => "/",
        "domain" => "10.0.0.210"
    },
          "tags" => [
        [0] "_jsonparsefailure"
    ],
    "@timestamp" => 2025-03-18T14:08:48.496501374Z,
      "@version" => "1",
        "action" => "提交订单",
       "user_id" => "1000"
}
```

##### **convert** **转换**

mutate 中的 convert 可以实现数据类型的转换。 支持转换integer、float、string等类型

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_mutate_convert_stdout.conf

input {
  http {
    port => 6666
  }
}

filter {
  # mutate 切割操作
  mutate {
    # 字段分隔符
    split => { "message" => "|" }

    # 添加字段
    add_field => {
      "user_id" => "%{[message][0]}"
      "action" => "%{[message][1]}"
      "time" => "%{[message][2]}"
    }

    # 删除无用字段
    remove_field => ["headers", "message"]

    # 对新添加字段进行格式转换
    convert => {
      "user_id" => "integer"
      "action" => "string"
      "time" => "string"
    }

    # convert => ["excute_time","float] #此格式也可以支持
    # convert => ["time","string" ]
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_grok_mutate_convert_stdout.conf -r
```

```
1000|提交订单|2020-01-08 09:10:21
```

```json
{
          "host" => {
        "ip" => "10.0.0.1"
    },
    "@timestamp" => 2025-03-18T14:12:06.708958863Z,
          "tags" => [
        [0] "_jsonparsefailure"
    ],
    "user_agent" => {
        "original" => "insomnia/11.0.0"
    },
        "action" => "提交订单",
          "time" => "2020-01-08 09:10:21",
          "http" => {
        "request" => {
            "mime_type" => "application/json",
                 "body" => {
                "bytes" => "37"
            }
        },
        "version" => "HTTP/1.1",
         "method" => "POST"
    },
       "user_id" => "1000",
           "url" => {
          "path" => "/",
        "domain" => "10.0.0.210",
          "port" => 6666
    },
      "@version" => "1"
}

```

##### **gsub** **替换**

```yaml
filter {
  mutate {
    gsub => ["message", "\n", " "] # 将message字段中的换行替换为空格
  }
}
```

##### **条件判断**

Filter 语句块中支持 if 条件判断功能

根据您提供的图片信息和配置内容，以下是整理后的 Filebeat 和 Logstash 配置文件，仅对格式进行调整，未进行优化：

**Filebeat 配置文件 (`/etc/filebeat/filebeat.yml`)**

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  tags: ["access"]

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  tags: ["error"]

- type: log
  enabled: true
  paths:
    - /var/log/syslog
  tags: ["system"]

output.logstash:
  hosts: ["10.0.0.104:5044", "10.0.0.105:5044"]
  # loadbalance: true       # 负载均衡
  # worker: 2 # number of hosts * workers # 开启多进程
```

**Logstash 配置文件 (`/etc/logstash/conf.d/filebeat_logstash_es.conf`)**

```yaml
input {
  beats {
    port => 5044
  }
}

filter {
  if "access" in [tags][0] {
    mutate {
      add_field => { "target_index" => "access-%{+YYYY.MM.dd}" }
    }
  }
  else if "error" in [tags][0] {
    mutate {
      add_field => { "target_index" => "error-%{+YYYY.MM.dd}" }
    }
  }
  else if "system" in [tags][0] {
    mutate {
      add_field => { "target_index" => "system-%{+YYYY.MM.dd}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["10.0.0.101:9200", "10.0.0.102:9200", "10.0.0.103:9200"]  # 一般写 data 地址
    index => "%{[target_index]}"   # 使用字段 target_index 值作为索引名
    template_overwrite => true      # 覆盖索引模板
  }
}
```



### **Logstash** **输出** **Output** **插件**

官方链接

```
https://www.elastic.co/guide/en/logstash/current/output-plugins.html
https://www.elastic.co/guide/en/logstash/8.17/output-plugins.html
https://www.elastic.co/guide/en/logstash/7.6/output-plugins.html
```

#### **Stdout** **插件**

stdout 插件将数据输出到屏幕终端，主要用于调试

```yaml
output {
  stdout {
    codec => rubydebug
  }
}
```

#### **File** **插件**

输出到文件，可以将分散在多个文件的数据统一存放到一个文件

示例: 将所有 web 机器的日志收集到一个文件中，从而方便统一管理

```yaml
input {
  stdin {}
}

output {
  stdout {
    codec => rubydebug
  }
  file {
    path => "/var/log/test.log"
  }
}
```

**Elasticsearch** **插件**

官方说明

```
https://www.elastic.co/guide/en/logstash/7.6/plugins-outputs-elasticsearch.html
```

索引的时间格式说明

```
https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html
```

当日志量较小时，可以按月或周生成索引，当日志量比较大时，会按天生成索引，以方便后续按天删除

```yaml
output {
  elasticsearch {
    hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]   # 一般写 ES 中 data 节点地址
    index => "app-%{+YYYY.MM.dd}"                # 指定索引名称,建议加时间,按天建立索引
    # index => "%{[@metadata][target_index]}"    # 使用字段[@metadata][target_index]值作为索引名
    template_overwrite => true                   # 覆盖索引模板,此项可选,默认值为 false
  }
}
```

**注意:索引名必须为小写**

支持多路输出

```yaml
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/http_grok_mutate_convert_stdout.conf
[root@ubuntu2404 ~]#cat /etc/logstash/conf.d/http_grok_mutate_convert_stdout.conf
input {
  http {
    port => 6666
  }
}

filter {
  # mutate 切割操作
  mutate {
    # 字段分隔符
    split => { "message" => "|" }

    # 添加字段
    add_field => {
      "user_id" => "%{[message][0]}"
      "action" => "%{[message][1]}"
      "time" => "%{[message][2]}"
    }

    # 删除无用字段
    remove_field => ["headers", "message"]

    # 对新添加字段进行格式转换
    convert => {
      "user_id" => "integer"
      "action" => "string"
      "time" => "string"
    }

    # convert => ["excute_time","float] #此格式也可以支持
    # convert => ["time","string" ]
  }
}

output {
  stdout {
    codec => rubydebug
  }
  elasticsearch {
    hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]   # 一般写 ES 中 data 节点地址
    index => "app-%{+YYYY.MM.dd}"                # 指定索引名称,建议加时间,按天建立索引
    # index => "%{[@metadata][target_index]}"    # 使用字段[@metadata][target_index]值作为索引名
    template_overwrite => true                   # 覆盖索引模板,此项可选,默认值为 false
  }
}

```

```bash
[root@ubuntu2404 ~]#logstash -f /etc/logstash/conf.d/http_grok_mutate_convert_stdout.conf -r
```

```
1000|提交订单|2020-01-08 09:10:21
```

```json
{
          "tags" => [
        [0] "_jsonparsefailure"
    ],
          "http" => {
         "method" => "POST",
        "request" => {
                 "body" => {
                "bytes" => "37"
            },
            "mime_type" => "application/json"
        },
        "version" => "HTTP/1.1"
    },
       "user_id" => "1000",
        "action" => "提交订单",
          "time" => "2020-01-08 09:10:21",
           "url" => {
        "domain" => "10.0.0.210",
          "path" => "/",
          "port" => 6666
    },
    "user_agent" => {
        "original" => "insomnia/11.0.0"
    },
    "@timestamp" => 2025-03-18T14:25:06.106074983Z,
          "host" => {
        "ip" => "10.0.0.1"
    },
      "@version" => "1"
}

```



#### **Redis** **插件**

Logstash 支持将日志转发至 Redis

```
https://www.elastic.co/guide/en/logstash/7.6/plugins-outputs-redis.html
```

```yaml
input {
  file {
    path => "/var/log/nginx/access.log"
    type => 'nginx-accesslog'
    start_position => "beginning"
    stat_interval => "3"
    codec => json
  }
  stdin {}
}

output {
  if [type] == 'nginx-accesslog' {
    redis {
      host => 'Redis_IP'	#redisIP地址
      port => "6379"
      password => "123456"
      db => "0"
      data_type => 'list'
      key => "nginx-accesslog"
    }
  }
  stdout {
    codec => rubydebug
  }
}
```

#### **Kafka** **插件**

Logstash 支持将日志转发至 Kafka

```
https://www.elastic.co/guide/en/logstash/7.6/plugins-outputs-kafka.html
```

```yaml
output {
  # stdout {}
  if [type] == 'nginx-accesslog' {
    kafka {
      bootstrap_servers => '10.0.0.211:9092,10.0.0.212:9092,10.0.0.213:9092'
      topic_id => 'nginx-accesslog'
      codec => 'json'    # 如果是Json格式,需要标识的字段
    }
  }
  if [type] == 'nginx-errorlog' {
    kafka {
      bootstrap_servers => '10.0.0.211:9092,10.0.0.212:9092,10.0.0.213:9092'
      topic_id => 'nginx-errorlog'
      codec => 'json'  # 为了保留logstash添加的字段,比如:type字段,也需要指定json格式,否则会丢失logstash添加的字段
    }
  }
}
```

### 案例

#### **收集单个系统日志并输出至** **Elasticsearch**

```yaml
input {
  file {
    path => "/var/log/dpkg.log"
    start_position => "beginning"
    stat_interval => "3"
  }
}

output {
  elasticsearch {
    hosts => ["10.0.0.101:9200"]
    index => "linux-%{+YYYY.MM.dd}"
  }
}
```

#### **通过** **Logstash** **收集多个日志文件并输出至** **Elasticsearch**

```yaml
input {
  file {
    path => "/var/log/syslog"            # 日志路径
    start_position => "beginning"        # 第一次收集日志的位置
    stat_interval => "3"                 # 日志收集的间隔时间
    type => "syslog"                     # 事件的唯一类型，用于区分不同日志，分别存储到不同索引
  }

  file {
    path => "/var/log/tomcat9/localhost_access_log.*.txt"  # Tomcat 访问日志路径
    start_position => "beginning"        # 第一次收集日志的位置
    stat_interval => "3"                 # 日志收集的间隔时间
    type => "tomcat-access"              # 事件的唯一类型
  }
}

output {
  if [type] == "syslog" {
    elasticsearch {
      hosts => ["10.0.0.101:9200"]
      index => "syslog-%{+YYYY.MM.dd}"   # 按日期动态生成 syslog 索引
    }
  }

  if [type] == "tomcat-access" {
    elasticsearch {
      hosts => ["10.0.0.101:9200"]
      index => "example-tomcat-%{+YYYY.MM.dd}"  # 按日期动态生成 Tomcat 索引
      # 注意：索引名不要以 "tomcat" 开头，否则在 Elasticsearch 8.10.X 中可能无法获取数据
    }
  }
}
```

#### **配置将不同类型日志写入不同索引**

```yaml
input {
  file {
    path => "/var/log/syslog"            # 系统日志路径
    start_position => "beginning"        # 从文件开头开始读取
    stat_interval => "3"                 # 检查文件状态的时间间隔（秒）
    type => "syslog"                     # 日志类型标识
  }

  file {
    path => "/apps/tomcat/logs/localhost_access_log.*.txt"  # Tomcat 访问日志路径
    start_position => "beginning"        # 从文件开头开始读取
    stat_interval => "3"                 # 检查文件状态的时间间隔（秒）
    type => "tomcat-access"               # 日志类型标识
  }
}

output {
  if [type] == "syslog" {                # 处理 syslog 类型的日志
    elasticsearch {
      hosts => ["10.0.0.101:9200"]       # Elasticsearch 主机地址
      index => "syslog-0.104-%{+YYYY.MM.dd}"  # 动态生成 syslog 索引
    }
    file {                               # 同时将日志写入文件
      path => "/tmp/syslog-0.104.log"    # 文件路径
    }
  }

  if [type] == "tomcat-access" {         # 处理 Tomcat 访问日志
    elasticsearch {
      hosts => ["10.0.0.101:9200"]       # Elasticsearch 主机地址
      index => "tomcat-accesslog-0.104-%{+YYYY.MM.dd}"  # 动态生成 Tomcat 索引
    }
    file {                               # 同时将日志写入文件
      path => "/tmp/tomcat-accesslog-0.104.log"  # 文件路径
    }
  }
}
```



## kibana

### 部署

注意: Kibana的版本要和 Elasticsearch 相同的版本，否则可能会出错

```
https://github.com/elastic/kibana
```

```bash
#下载地址
https://www.elastic.co/downloads/kibana
https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/k/kibana/
```

```bash
[root@ubuntu2404 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/elasticstack/8.x/apt/pool/main/k/kibana/kibana-8.17.3-amd64.deb
[root@ubuntu2404 ~]#apt install ./kibana-8.17.3-amd64.deb 

#修改配置文件
[root@ubuntu2404 ~]#vim /etc/kibana/kibana.yml 
...
# =================== System: Elasticsearch ===================
# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://10.0.0.201:9200","http://10.0.0.202:9200","http://10.0.0.203:9200"]	#修改为elasticsearch主机地址
...
i18n.locale: "zh-CN"		#修改为中文
...
server.host: "0.0.0.0"

[root@ubuntu2404 ~]#systemctl enable --now kibana.service 
[root@ubuntu2404 ~]#systemctl status kibana.service 
[root@ubuntu2404 ~]#ss -tnulp | grep node
tcp   LISTEN 0      511          0.0.0.0:5601       0.0.0.0:*    users:(("node",pid=3339,fd=31))  
```

查看索引位置

management——>stack management——>索引管理

数据视图

```bash
#以前版本叫索引模式
#将多个不同的相同业务的索引合并在一起
nginx-accesslog 数据视图
nginx-accesslog-*

nginx-accesslog-2025-03-17
nginx-accesslog-2025-03-18
nginx-accesslog-2025-03-19

#时间 @timestamp

json
{
	key:v1
	key:v2
	time:xxxxx
}
```

### 设置访问kibana密码

**nginx方式**

```
https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html
```

```bash
#生成用户和密码
[root@ubuntu2404 ~]#apt -y install nginx apache2-utils
[root@ubuntu2404 ~]#htpasswd -bc /etc/nginx/conf.d/kibana.users admin 123456
Adding password for user admin
[root@ubuntu2404 ~]#cat /etc/nginx/conf.d/kibana.users
admin:$apr1$hjwt01bG$siKwDgFt7t5Ymh6vmQN0B1
[root@ubuntu2404 ~]#ps aux | grep nginx
root        1874  0.0  0.0  11156  1716 ?        Ss   22:07   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    1875  0.0  0.2  12880  4532 ?        S    22:07   0:00 nginx: worker process
www-data    1876  0.0  0.2  12880  4404 ?        S    22:07   0:00 nginx: worker process
root        2015  0.0  0.1   9640  2304 pts/0    S+   22:15   0:00 grep --color=auto nginx
[root@ubuntu2404 ~]#chown www-data: /etc/nginx/conf.d/kibana.users
[root@ubuntu2404 ~]#chmod 600 /etc/nginx/conf.d/kibana.users
```

```bash
[root@ubuntu2404 ~]#vim /etc/nginx/conf.d/kibana.conf
server {
    listen 80;                      # 监听 80 端口
    server_name kibana.kang.org;    # 服务器名称

    auth_basic "kibana website";    # 启用基本认证，提示信息为 "kibana website"
    auth_basic_user_file conf.d/kibana.users;  # 指定认证用户文件路径

    location / {
        proxy_pass http://127.0.0.1:5601;  # 将请求代理到本地的 Kibana 服务（端口 5601）
    }
}

[root@ubuntu2404 ~]#systemctl restart nginx.service 

访问 http://kibana.kang.org
```

```bash
#直接输入kibana地址也可以访问
#修改为只能本主机访问
[root@ubuntu2404 ~]#vim /etc/kibana/kibana.yml 
#server.host: "0.0.0.0"
server.host: "localhost"
```



## **ELK** **综合实战案例**

| ip         | 服务           | 主机名   |
| ---------- | -------------- | -------- |
| 10.0.0.201 | es-node1       | node1    |
| 10.0.0.202 | es-node2       | node2    |
| 10.0.0.203 | es-node3       | node3    |
| 10.0.0.205 | redis          | web02    |
| 10.0.0.204 | filebeat       | web01    |
| 10.0.0.210 | logstash-web01 | logstash |
| 10.0.0.211 | kafka-node1    | node1    |
| 10.0.0.212 | kafka-node2    | node2    |
| 10.0.0.213 | kafka-node3    | node3    |
| 10.0.0.200 | kibana         | kibana   |

### **收集应用特定格式的日志输出至Elasticsearch并利用Kibana展示**

**filebeat**

```bash
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
```

```yaml
filebeat.inputs:
  - type: log              # 输入类型为日志文件
    enabled: true          # 启用该输入配置
    paths:                 # 指定日志文件路径
      - /var/log/mall_app.log

output.logstash:           # 配置 Logstash 输出
  hosts: ["logstash.wang.org:6666"]  # Logstash 服务器地址和端口
```

```bash
[root@web01 ~]#vim /etc/filebeat/filebeat.yml
```

**logstash**

```bash
[root@ubuntu2404 ~]#vim /etc/logstash/conf.d/filebeat_to_es.conf
```

```bash
input {
  beats {
    port => 6666  # 监听 fileBeats 输入，端口为 6666
  }
}

filter {
  mutate {
    split => { "message" => "|" }  # 使用 "|" 分隔符切割 message 字段
    add_field => {                  # 添加新字段
      "user_id" => "%{[message][1]}"  # 提取用户 ID
      "action" => "%{[message][2]}"   # 提取用户行为
      "time" => "%{[message][3]}"     # 提取时间
      # "[@metadata][target_index]" => "mall-app-%{+YYYY.MM.dd}"  # 动态生成索引名（注释掉）
    }
    remove_field => ["message"]      # 删除原始 message 字段
    convert => {                     # 转换字段类型
      "user_id" => "integer"         # 将 user_id 转换为整数
      "action" => "string"           # 将 action 转换为字符串
      "time" => "string"             # 将 time 转换为字符串
    }
  }

  date {
    match => ["time", "yyyy-MM-dd HH:mm:ss"]  # 解析时间字段
    target => "@timestamp"                   # 覆盖默认的 @timestamp 字段
    timezone => "Asia/Shanghai"              # 设置时区为上海
  }
}

output {
  stdout {                          # 输出到控制台（用于调试）
    codec => rubydebug              # 使用 rubydebug 格式输出
  }

  elasticsearch {                   # 输出到 Elasticsearch
    hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
    index => "mall-app-%{+YYYY.MM.dd}"  # 动态生成索引名
    # index => "[@metadata][target_index]"  # 使用元数据中的索引名（注释掉）
    template_overwrite => true      # 允许覆盖索引模板
  }
}
```

```
[INFO] 2018-06-28 08:08:12 [www.mall.com] - DAU|9136|加入收藏|2018-06-28 01:05:02
[INFO] 2018-06-28 08:08:14 [www.mall.com] - DAU|5035|搜索|2018-06-28 01:07:01
[INFO] 2018-06-28 08:08:15 [www.mall.com] - DAU|669|使用优惠券|2018-06-28 08:05:13
[INFO] 2018-06-28 08:08:19 [www.mall.com] - DAU|2564|搜索|2018-06-28 08:07:08
[INFO] 2018-06-28 08:08:20 [www.mall.com] - DAU|5294|加入收藏|2018-06-28 02:03:15
```



### **Filebeat收集Nginx日志利用Redis缓存发送至Elasticsearch**

```yaml
#修改nginx的日志为json格式
[root@web01 ~]#vim /etc/nginx/nginx.conf 
	##
	# Logging Settings
	##
    log_format access_json '{"@timestamp":"$time_iso8601",'
        '"host":"$server_addr",'
        '"clientip":"$remote_addr",'
        '"size":$body_bytes_sent,'
        '"responsetime":$request_time,'
        '"upstreamtime":"$upstream_response_time",'
        '"upstreamhost":"$upstream_addr",'
        '"http_host":"$host",'
        '"uri":"$uri",'
        '"domain":"$host",'
        '"xff":"$http_x_forwarded_for",'
        '"referer":"$http_referer",'
        '"tcp_xff":"$proxy_protocol_addr",'
        '"http_user_agent":"$http_user_agent",'
        '"status":"$status"}';
	#access_log /var/log/nginx/access.log;
    access_log /var/log/nginx/access_json.log access_json;
```

```bash
[root@web01 ~]#systemctl restart nginx
```

```bash
#filebeat收集nginx日志
[root@web01 ~]#vim /etc/filebeat/filebeat.yml

filebeat.inputs:
  # 配置 Nginx 访问日志输入
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access_json.log
    json.keys_under_root: true  # 默认 False 会将 JSON 数据存储至 message，改为 true 则会独立存储
    json.overwrite_keys: true   # 设为 true，覆盖默认的 message 字段，使用自定义 JSON 格式中的 key
    tags: ["nginx-access"]      # 添加标签，便于后续分类处理

  # 配置 Nginx 错误日志输入
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/error.log
    tags: ["nginx-error"]       # 添加标签，便于后续分类处理

  # 配置系统日志输入
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
    tags: ["syslog"]            # 添加标签，便于后续分类处理

# 配置 Redis 输出
output.redis:
  hosts: ["10.0.0.205:6379"]   # Redis 服务器地址
  password: "123456"           # Redis 认证密码
  db: "0"                      # Redis 数据库编号
  key: "nginx"              # 所有日志都存放在 key 名称为 filebeat 的列表中，使用 `llen filebeat` 可查看日志记录数

  # 也可以用下面的列表方式实现不同日志存放在不同的 key
  # keys:
  #   - key: "nginx_access"
  #     when.contains:
  #       tags: "access"
  #   - key: "nginx_error"
  #     when.contains:
  #       tags: "error"
```

```bash
[root@web01 ~]#systemctl restart filebeat.service
```

```bash
[root@logstash ~]#vim /etc/logstash/conf.d/redis_filebeat_to_es.conf

#logsdeat取走数据，传给es
input {
  redis {
    host => "10.0.0.205"       # Redis 服务器地址
    port => "6379"            # Redis 端口
    password => "123456"      # Redis 认证密码
    db => "0"                 # Redis 数据库编号
    key => "nginx"         # 从 Redis 的 key 名称为 filebeat 的列表中读取数据
    data_type => "list"       # 数据类型为列表
  }
}

filter {
  # 处理带有 nginx-access 标签的日志
  if "nginx-access" in [tags] {
    # 使用 GeoIP 插件解析客户端 IP 地址
    geoip {
      source => "clientip"    # 日志必须是 JSON 格式，且包含 clientip 字段
      target => "geoip"       # 将解析结果存储到 geoip 字段中
      database => "/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/logstash-filter-geoip-7.3.1-java/vendor/GeoLite2-City.mmdb"  # 8.15.X 版本必须指定数据库文件路径
      add_field => ["[geoip][coordinates]", "%{[geoip][geo][location][lon]}"]  # 添加经度字段
      add_field => ["[geoip][coordinates]", "%{[geoip][geo][location][lat]}"]  # 添加纬度字段
    }
    # 将经纬度字段转换为浮点数（8.X 版本必须做，7.X 版本可不做）
    mutate {
      convert => ["[geoip][coordinates]", "float"]
    }
  }

  # 将 upstreamtime 字段转换为浮点数
  mutate {
    convert => ["upstreamtime", "float"]
  }
}

output {
  # 处理带有 syslog 标签的日志
  if "syslog" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "syslog-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
    }
  }

  # 处理带有 nginx-access 标签的日志
  if "nginx-access" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "nginxaccess-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
      template_overwrite => true  # 允许覆盖模板
    }
    # 将日志输出到控制台（调试用）
    stdout {
      codec => "rubydebug"
    }
  }

  # 处理带有 nginx-error 标签的日志
  if "nginx-error" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "nginxerrorlog-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
      template_overwrite => true  # 允许覆盖模板
    }
  }
}
```

```bash
[root@logstash ~]#systemctl restart logstash.service 
```



### 世界地图

analytics——》maps

**官方例子**

GET /kibana_sample_data_logs

```json
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            },
```

GET /nginxaccess-2022.03.03

```json
{
  "nginxaccess-2022.03.03": {
    "aliases": {},
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "@version": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "agent": {
          "properties": {
            "ephemeral_id": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "id": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "type": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "version": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "clientip": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "domain": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "ecs": {
          "properties": {
            "version": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "event": {
          "properties": {
            "original": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "geoip": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            },
            "geo": {
              "properties": {
                "city_name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "continent_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "country_iso_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "country_name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "location": {
                  "properties": {
                    "lat": {
                      "type": "float"
                    },
                    "lon": {
                      "type": "float"
                    }
                  }
                },
                "postal_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "region_iso_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "region_name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "timezone": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            },
            "ip": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "mmdb": {
              "properties": {
                "dma_code": {
                  "type": "long"
                }
              }
            }
          }
        },
        "host": {
          "properties": {
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "http_host": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "http_user_agent": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "input": {
          "properties": {
            "type": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "log": {
          "properties": {
            "file": {
              "properties": {
                "path": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            },
            "offset": {
              "type": "long"
            }
          }
        },
        "referer": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "responsetime": {
          "type": "long"
        },
        "size": {
          "type": "long"
        },
        "status": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "tags": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "tcp_xff": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "upstreamhost": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "upstreamtime": {
          "type": "float"
        },
        "uri": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "xff": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    },
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "1",
        "provided_name": "nginxaccess-2022.03.03",
        "creation_date": "1742444211290",
        "number_of_replicas": "1",
        "uuid": "TMmu0rXqRoWKUAzzM2GSWw",
        "version": {
          "created": "8521000"
        }
      }
    }
  }
}
```

```
PUT /_template/template_nginx_accesslog   # 模板名称可以自定义
{
  "index_patterns": [
    "nginxaccess-*"                      # 8.X 数据视图对应的索引模式，匹配索引名称的模式
    // "logstash-kafka-nginx-accesslog-*" # 可选的索引模式
  ],
  "order": 0,                            # 模板优先级，数值越小优先级越高
  "aliases": {},                          # 索引别名，可以为空
  "mappings": {                           # 定义字段映射
    "properties": {
      "geoip": {
        "properties": {
          "coordinates": {
            "type": "geo_point"           # 将 coordinates 字段类型设置为 geo_point
          }
          // 其他字段映射可以继续添加
        }
      }
      // 其他字段映射可以继续添加
    }
  },
  "settings": {                           # 定义索引设置
    // 可以根据需要添加索引设置，例如分片数、副本数等
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}
```

```
PUT /_template/template_nginx_accesslog
{
  "index_patterns": [
    "nginxaccess-*"
  ],
  "order": 0,
  "aliases": {},
   "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "@version": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "agent": {
          "properties": {
            "ephemeral_id": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "id": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "type": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "version": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "clientip": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "domain": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "ecs": {
          "properties": {
            "version": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "event": {
          "properties": {
            "original": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "geoip": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            },
            "geo": {
              "properties": {
                "city_name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "continent_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "country_iso_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "country_name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "location": {
                  "properties": {
                    "lat": {
                      "type": "float"
                    },
                    "lon": {
                      "type": "float"
                    }
                  }
                },
                "postal_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "region_iso_code": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "region_name": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                },
                "timezone": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            },
            "ip": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "mmdb": {
              "properties": {
                "dma_code": {
                  "type": "long"
                }
              }
            }
          }
        },
        "host": {
          "properties": {
            "name": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "http_host": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "http_user_agent": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "input": {
          "properties": {
            "type": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        },
        "log": {
          "properties": {
            "file": {
              "properties": {
                "path": {
                  "type": "text",
                  "fields": {
                    "keyword": {
                      "type": "keyword",
                      "ignore_above": 256
                    }
                  }
                }
              }
            },
            "offset": {
              "type": "long"
            }
          }
        },
        "referer": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "responsetime": {
          "type": "long"
        },
        "size": {
          "type": "long"
        },
        "status": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "tags": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "tcp_xff": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "upstreamhost": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "upstreamtime": {
          "type": "float"
        },
        "uri": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "xff": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
}
```

```
#复制上面的mappings开始的行到settings行之前结束,并最后再加一个 }
```

```
#删除旧的数据索引
```





### **Filebeat收集Nginx日志并写入Kafka缓存发送至Elasticsearch**

```yaml
#filebeat收集日志到kafka    
[root@web01 ~]#vim /etc/filebeat/filebeat.yml

filebeat.inputs:
  # 配置 Nginx 访问日志输入
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access_json.log
    json.keys_under_root: true  # 默认 False 会将 JSON 数据存储至 message，改为 true 则会独立存储
    json.overwrite_keys: true   # 设为 true，覆盖默认的 message 字段，使用自定义 JSON 格式中的 key
    tags: ["nginx-access"]      # 添加标签，便于后续分类处理

  # 配置 Nginx 错误日志输入
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/error.log
    tags: ["nginx-error"]       # 添加标签，便于后续分类处理

  # 配置系统日志输入
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
    tags: ["syslog"]            # 添加标签，便于后续分类处理

# 配置 Kafka 输出
output.kafka:
  hosts: ["10.0.0.211:9092", "10.0.0.212:9092", "10.0.0.213:9092"]  # Kafka 集群地址
  topic: filebeat-log              # 指定 Kafka 的 topic
  partition.round_robin:
    reachable_only: true           # true 表示只发布到可用的分区，false 表示所有分区
  required_acks: 1                 # 1 表示等待写入主分区（默认），0 表示不确认，-1 表示等待写入副本分区
  compression: gzip                # 使用 gzip 压缩消息
  max_message_bytes: 1000000       # 每条消息最大长度，以字节为单位，如果超过将丢弃

```

```bash
[root@web01 ~]#systemctl restart filebeat.service 
```

```bash
#logstach收集kafka的数据，发送到es
[root@logstash ~]#vim /etc/logstash/conf.d/kafka_filebeat_to_es.conf

input {
  kafka {
    bootstrap_servers => "10.0.0.201:9092,10.0.0.202:9092,10.0.0.203:9092"  # Kafka 集群地址
    topics => "filebeat-log"                                                # 指定 Kafka topic
    codec => "json"                                                         # 使用 JSON 编解码器解析消息
    # group_id => "logstash"                                                # 消费者组的名称（可选）
    # consumer_threads => "3"                                               # 消费者线程数，建议设置为与 Kafka 分区数相同（可选）
    # topics_pattern => "nginx-.*"                                          # 通过正则表达式匹配 topic（可选）
  }
}

filter {
  # 处理带有 nginx-access 标签的日志
  if "nginx-access" in [tags] {
    # 使用 GeoIP 插件解析客户端 IP 地址
    geoip {
      source => "clientip"    # 日志必须是 JSON 格式，且包含 clientip 字段
      target => "geoip"       # 将解析结果存储到 geoip 字段中
      database => "/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/logstash-filter-geoip-7.3.1-java/vendor/GeoLite2-City.mmdb"  # 8.15.X 版本必须指定数据库文件路径
      add_field => ["[geoip][coordinates]", "%{[geoip][geo][location][lon]}"]  # 添加经度字段
      add_field => ["[geoip][coordinates]", "%{[geoip][geo][location][lat]}"]  # 添加纬度字段
    }
    # 将经纬度字段转换为浮点数（8.X 版本必须做，7.X 版本可不做）
    mutate {
      convert => ["[geoip][coordinates]", "float"]
    }
  }

  # 将 upstreamtime 字段转换为浮点数
  mutate {
    convert => ["upstreamtime", "float"]
  }
}

output {
  # 处理带有 syslog 标签的日志
  if "syslog" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "syslog-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
    }
  }

  # 处理带有 nginx-access 标签的日志
  if "nginx-access" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "nginxaccess-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
      template_overwrite => true  # 允许覆盖模板
    }
    # 将日志输出到控制台（调试用）
    stdout {
      codec => "rubydebug"
    }
  }

  # 处理带有 nginx-error 标签的日志
  if "nginx-error" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "nginxerrorlog-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
      template_overwrite => true  # 允许覆盖模板
    }
  }
}

```

```bash
[root@logstash ~]#systemctl restart logstash.service
```



### **Logstash收集日志写入MySQL数据库**

![image-20250320130959609](C:/Users/zhaok/AppData/Roaming/Typora/typora-user-images/image-20250320130959609.png)

ES中的日志后续会被删除,但有些重要数据,比如状态码、客户端IP、客户端浏览器版本等，后期可以会按月或年做数据统计等,因此需要持久保存

可以将重要数据写入数据库达到持久保存目的

```bash
[root@es-node1 ~]#apt update && apt -y install mysql-server
[root@es-node1 ~]#ss -ntl|grep 3306
LISTEN   0         80                  127.0.0.1:3306             0.0.0.0:*   
[root@es-node1 ~]#sed -i '/127.0.0.1/s/^/#/' /etc/mysql/mysql.conf.d/mysqld.cnf 
[root@es-node1 ~]#vim /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address        = 0.0.0.0
mysqlx-bind-address = 0.0.0.0
[root@es-node1 ~]#systemctl restart mysql.service 
[root@es-node1 ~]#ss -ntl|grep 3306
LISTEN   0         80                  0.0.0.0:3306             0.0.0.0:*  
```

**创建库和表并授权用户登录**

```mysql
mysql> create database elk ;                                      #MySQL8.0以上默认就支持
mysql> create database elk character set utf8 collate utf8_bin;   #MySQL5.7以下需要显示指定
mysql> create user elk@"10.0.0.%" identified by '123456';
mysql> grant all privileges on elk.* to elk@"10.0.0.%";
mysql> flush privileges;

#创建表,字段对应需要保存的数据字段
mysql> use elk;
mysql> create table elklog (clientip varchar(39),responsetime float(10,3),uri varchar(256),status char(3),time timestamp default current_timestamp);
mysql> desc elklog;
+--------------+--------------+------+-----+-------------------+-------------------+
| Field        | Type         | Null | Key | Default           | Extra             |
+--------------+--------------+------+-----+-------------------+-------------------+
| clientip     | varchar(39)  | YES  |     | NULL              |                   |
| responsetime | float(10,3)  | YES  |     | NULL              |                   |
| uri          | varchar(256) | YES  |     | NULL              |                   |
| status       | char(3)      | YES  |     | NULL              |                   |
| time         | timestamp    | YES  |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
+--------------+--------------+------+-----+-------------------+-------------------+

```

**安装logstash-output-jdbc 插件**

```bash
#查看没有 logstash-output-jdbc 插件
[root@logstash ~]#/usr/share/logstash/bin/logstash-plugin list

#插件安装
[root@logstash ~]#/usr/share/logstash/bin/logstash-plugin install logstash-output-jdbc

#离线安装
#如果无法在线安装,可以先从已经安装的主机导出插件,再导入
#导出插件
[root@ubuntu2204 ~]#/usr/share/logstash/bin/logstash-plugin prepare-offline-pack logstash-output-jdbc
Using bundled JDK: /usr/share/logstash/jdk

#离线导入插件
[root@logstash ~]#/usr/share/logstash/bin/logstash-plugin install file:///root/logstash-offline-plugins-8.17.3.zip
Using bundled JDK: /usr/share/logstash/jdk
Installing file: /root/logstash-offline-plugins-8.17.3.zip
Resolving dependencies................................
Install successful

#删除插件
[root@logstash1 ~]#/usr/share/logstash/bin/logstash-plugin remove logstash-output-jdbc
```

**Logstash** **配置** **mysql-connector-java** **包**

官方下载地址：

```
https://dev.mysql.com/downloads/connector/
```

```
https://downloads.mysql.com/archives/c-j/
```

```bash
[root@logstash ~]#ls
logstash-8.17.3-amd64.deb
[root@logstash ~]#dpkg -i ./mysql-connector-j_8.0.33-1ubuntu23.04_all.deb 
[root@logstash ~]#ll /usr/share/java/mysql-connector-java-8.0.33.jar
lrwxrwxrwx 1 root root 28 Apr  4  2023 /usr/share/java/mysql-connector-java-8.0.33.jar -> mysql-connector-j-8.0.33.jar
[root@logstash ~]#ll /usr/share/java/mysql-connector-j-8.0.33.jar
-rw-r--r-- 1 root root 2481541 Apr  4  2023 /usr/share/java/mysql-connector-j-8.0.33.jar

#复制jar文件到logstash指定的目录下
[root@logstash ~]#mkdir -p /usr/share/logstash/vendor/jar/jdbc
[root@logstash ~]#cp /usr/share/java/mysql-connector-j-8.0.33.jar /usr/share/logstash/vendor/jar/jdbc
```

**修改logstash配置文件**

```
   jdbc {
    connection_string => "jdbc:mysql://10.0.0.205/elk?user=elk&password=123456&useUnicode=true&characterEncoding=UTF8"  # MySQL 连接字符串
    statement => [
      "INSERT INTO elklog (clientip, responsetime, uri, status) VALUES (?, ?, ?, ?)",  # SQL 插入语句
      "clientip",  # 第一个占位符对应的字段
      "responsetime",  # 第二个占位符对应的字段
      "uri",  # 第三个占位符对应的字段
      "status"  # 第四个占位符对应的字段
    ]
  }
```

```bash
[root@logstash ~]#vim /etc/logstash/conf.d/kafka_filebeat_to_es.conf 

input {
  kafka {
    bootstrap_servers => "10.0.0.211:9092,10.0.0.212:9092,10.0.0.213:9092"  # Kafka 集群地址
    topics => "filebeat-log"                                                # 指定 Kafka topic
    codec => "json"                                                         # 使用 JSON 编解码器解析消息
    # group_id => "logstash"                                                # 消费者组的名称（可选）
    # consumer_threads => "3"                                               # 消费者线程数，建议设置为与 Kafka 分区数相同（可选）
    # topics_pattern => "nginx-.*"                                          # 通过正则表达式匹配 topic（可选）
  }
}

filter {
  # 处理带有 nginx-access 标签的日志
  if "nginx-access" in [tags] {
    # 使用 GeoIP 插件解析客户端 IP 地址
    geoip {
      source => "clientip"    # 日志必须是 JSON 格式，且包含 clientip 字段
      target => "geoip"       # 将解析结果存储到 geoip 字段中
      database => "/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/logstash-filter-geoip-7.3.1-java/vendor/GeoLite2-City.mmdb"  # 8.15.X 版本必须指定数据库文件路径
      add_field => ["[geoip][coordinates]", "%{[geoip][geo][location][lon]}"]  # 添加经度字段
      add_field => ["[geoip][coordinates]", "%{[geoip][geo][location][lat]}"]  # 添加纬度字段
    }
    # 将经纬度字段转换为浮点数（8.X 版本必须做，7.X 版本可不做）
    mutate {
      convert => ["[geoip][coordinates]", "float"]
    }
  }

  # 将 upstreamtime 字段转换为浮点数
  mutate {
    convert => ["upstreamtime", "float"]
  }
}

output {
  # 处理带有 syslog 标签的日志
  if "syslog" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "syslog-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
    }
  }

  # 处理带有 nginx-access 标签的日志
  if "nginx-access" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "nginxaccess-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
      template_overwrite => true  # 允许覆盖模板
    }

    jdbc {
      connection_string => "jdbc:mysql://10.0.0.205/elk?user=elk&password=123456&useUnicode=true&characterEncoding=UTF8"  # MySQL 连接字符串
      statement => [
        "INSERT INTO elklog (clientip, responsetime, uri, status) VALUES (?, ?, ?, ?)",  # SQL 插入语句
        "clientip",  # 第一个占位符对应的字段
        "responsetime",  # 第二个占位符对应的字段
        "uri",  # 第三个占位符对应的字段
        "status"  # 第四个占位符对应的字段
      ]
    }
  }

  # 处理带有 nginx-error 标签的日志
  if "nginx-error" in [tags] {
    elasticsearch {
      hosts => ["10.0.0.201:9200", "10.0.0.202:9200", "10.0.0.203:9200"]  # Elasticsearch 集群地址
      index => "nginxerrorlog-%{+YYYY.MM.dd}"  # 索引名称，按日期分割
      template_overwrite => true  # 允许覆盖模板
    }
  }
}
```

```bash
[root@logstash ~]#systemctl restart logstash.service
```

```sql
#查询访问次数最多的前三个
mysql> select clientip,count(*) as count from elklog group by clientip order by count(*) desc limit 3;
+----------------+-------+
| clientip       | count |
+----------------+-------+
| 223.70.179.195 | 10053 |
| 114.115.207.72 |  6087 |
| 150.158.91.120 |  2122 |
+----------------+-------+
3 rows in set (0.20 sec)

```

