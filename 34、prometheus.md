# 34、prometheus

## Prometheus介绍

一个完整的监控系统需要包括如下功能：数据产生、数据采集、数据存储、数据处理、数据展示分析、告警

### 数据采集

- 软件方式：

  agent: 专用的软件的一种应用机制。

  http: 基于http 协议实现数据采集

  ssh: 系统常见的一种应用通信机制，但是并非所有系统都支持。

  SNMP: 简单网络管理协议(Simple Network Management Protocol),是工作在各种网络设备中的一种机制。

- 硬件方式：

  IPMI: 智慧平台管理接口(Intelligent Platform Management Interface)是一种工业标准用于采集硬件设备的各种物理健康状态数据，如温度、电压、风扇工作状态、电源状态等。

注意：由于每个业务场景，需要采集的指标数量是不确定的，有时只是一个业务场景，就需要采集数百个指标，如果按照上述所说的周期性采集的方式来说，数据的采集量是相当大的

### **监控内容和方法**

**资源数据**

- 硬件设备：服务器、路由器、交换机、IO系统等
- 系统资源：OS、网络、容器、VM实例
- 应用软件：Nginx、MySQL、Java应用等

**业务服务**

- 业务状态：服务通信、服务运行、服务下线、性能指标、QPS、DAU(Daily Active User )日活、转化率、业务口(登陆，注册，聊天，留⾔)、产品转化率、充值额度、⽤户投诉等
- 一般故障：访问缓慢、存储空间不足，数据同步延迟，主机宕机、主机不可达
- 严重故障：服务不可用、集群故障

**趋势分析**

- 数据统计：时间序列数据展示历史数据等
- 数据预测：事件什么时候发生、持续时间、发生概率是多大等,比如:电商大促时间

**监控方法**

**Google 的四个黄金指标**

常用于在服务级别帮助衡量终端用户体验、服务中断、业务影响等层面的问题，适用于应用及服务监控

- 延迟(Latency)

  服务请求所需要的时长，例如HTTP请求平均延迟

  应用程序响应时间会受到所有核心系统资源（包括网络、存储、CPU和内存）延迟的影响需要区分失败请求和成功请求

- 流量(Traffic)，也称为吞吐量

  衡量服务的容量需求，例如每秒处理的HTTP请求数QPS或者数据库系统的事务数量TPS

  吞吐量指标包括每秒Web请求、API调用等示例，并且被描述为通常表示为每秒请求数的需求

- 错误(Errors)

  失败的请求（流量)的数量，通常以绝对数量或错误请求占请求总数的百分比表示，请求失败的速率，用于衡量错误发生的情况

  例如：HTTP 500错误数等显式失败，返回错误内容或无效内容等隐式失败，以及由策略原因导致的失败(例如强制要求响应时间超过30毫秒的请求视为错误)

- 饱和度(Saturation)

  衡量资源的使用情况,用于表达应用程序有多"满"

  资源的整体利用率，包括CPU（容量、配额、节流)、内存(容量、分配)、存储（容量、分配和 I/O吞吐量)和网络

  例如：内存、CPU、I/O、磁盘等资源的使用量

### **什么是序列数据**

参考资料：https://db-engines.com/en/ranking/time+series+dbms

时间序列数据(TimeSeries Data) : 按照时间顺序记录系统、设备状态变化的数据被称为时序数据。

时序数据库记录的数据以时间为横座标,纵坐标为数据

时间序列数据库 (Time Series Database , 简称 TSDB) 是一种高性能、低成本、稳定可靠的在线时间序列数据库服务，提供高效读写、高压缩比存储、时序数据插值及聚合计算等服务，广泛应用于物联网（IoT）设备监控系统、企业能源管理系统（EMS）、生产安全监控系统和电力检测系统等行业场景；除此以外，还提供时空场景的查询和分析的能力。

TSDB 具备秒级写入百万级时序数据的性能，提供高压缩比低成本存储、预降采样、插值、多维聚合计算、可视化查询结果等功能，解决由设备采集点数量巨大、数据采集频率高造成的存储成本高、写入和查询分析效率低的问题。

TSDB是一个分布式时间序列数据库，具备多副本高可用能力。同时在高负载大规模数据量的情况下可以方便地进行弹性扩容，方便用户结合业务流量特点进行动态规划与调整。

应用的场景：

- 物联网设备无时无刻不在产生海量的设备状态数据和业务消息数据，这些数据有助于进行设备监控、业务分析预测和故障诊断。
- 传统电力化工以及工业制造行业需要通过实时的监控系统进行设备状态检测，故障发现以及业务趋势分析。
- 系统运维和业务实时监控,通过对大规模应用集群和机房设备的监控，实时关注设备运行状态、资源利用率和业务趋势，实现数据化运营和自动化开发运维。

### **Prometheus** **简介**

Prometheus 本身基于Go语言开发的一套开源的系统监控报警框架和时序列数据库(TSDB)。

Prometheus 的监控功能很完善和全面，性能也足够支撑上万台规模的集群。

网站：https://prometheus.io/

github：https://github.com/prometheus

其特点主要如下：

- 支持多维数据模型：由度量名和键值对组成的时间序列数据
- 内置时间序列数据库TSDB(Time Series Database )
- 支持PromQL(Prometheus Query Language)查询语言，可以完成非常复杂的查询和分析，对图表展示和告警非常有意义
- 支持 HTTP 的 Pull 方式采集时间序列数据
- 支持 PushGateway 采集瞬时任务的数据
- 支持静态配置和服务发现两种方式发现目标
- 多种可视化和仪表盘,支持第三方 Dashboard,比如:Grafana

数据特点

- 监控指标，采用独创的指标格式，我们称之为Prometheus格式，这个格式在监控场景中很常见。
- 数据标签，支持多维度标签，每个独立的标签组合都代表一个独立的时间序列
- 数据处理，Prometheus内部支持多种数据的聚合、切割、切片等功能。
- 数据存储，Prometheus支持双精度浮点型数据存储和字符串

**Prometheus** **不足**

- 不支持集群化
- 被监控集群规模过大后本身性能有一定瓶颈
- 中文支持不好
- 功能不完整，需要结合其它组件实现监控的全部功能

### **Prometheus** **架构**

 Prometheus 同其它TSDB相比有一个非常典型的特性：

它主动从各Target上"拉取（pull)"数据，相当于Zabbix里的被动模式,而非等待被监控端的"推送（push）"

```
zabbix 主动被动模式从agent看
主动：agent主动发数据 （推）
被动：zabbix-server主动拉取数据 （拉） （默认）

prometheus-server 主动拉取数据 （拉）
```

两个方式各有优劣，其中，Pull模型的优势在于：集中控制：有利于将配置集在 Prometheus Server上完成，包括指标及采取速率等,Prometheus的根本目标在于收集在Target上预先完成聚合的聚合型数据，而非一款由事件驱动的存储系统

![image-20250301153251297](/5day-png/34prometheus构架.png)

![image-20250301153339370](/5day-png/34prometheus构架1.png)

采集	存储	展示	告警

**Prometheus Server**: （采集、存储）

- Prometheus 的核心组件，负责数据采集、存储和查询。具体流程如下：
  - **Retrieval (数据采集)**：Prometheus 拉取数据的过程，从监控目标或 Pushgateway 获取数据。
  - **TSDB (时间序列数据库)**：Prometheus 将采集到的时间序列数据存储在本地的硬盘（HDD/SSD）中。
  - **HTTP server (HTTP 服务器)**：Prometheus 提供 Web 界面和 API 供用户查询监控数据。

**Grafana**:（展示）

- 用于将 Prometheus 收集的数据可视化。Grafana 提供丰富的图表、仪表盘和监控面板，帮助用户实时查看和分析监控数据。

**Alertmanager**:（告警）

- 用于处理和管理 Prometheus 的告警。Alertmanager 负责接收来自 Prometheus 的告警信息，并根据配置的规则将告警信息推送到不同的通知渠道，如 **PagerDuty、Email** 等。

 **Service Discovery (服务发现)**:

- Prometheus 通过 **Kubernetes** 或 **file_sd**（文件服务发现）等方式自动发现监控目标，避免手动配置目标列表。

**Pushgateway**:

- 用于处理短期作业或临时任务的监控数据。因为 Prometheus 默认使用拉取（pull）方式来抓取监控数据，但短期任务没有长期运行的指标端点可以供 Prometheus 拉取。Pushgateway 允许这些任务在结束时将数据推送到 Prometheus。

**Prometheus Targets (监控目标)**:

- Prometheus 需要监控的目标。通常这些目标是一些运行着应用、服务或机器的节点，它们通过 **jobs/exporters** 暴露出相关的监控数据。常见的 exporter 包括 Node Exporter、Kubernetes Exporter 等。

**PromQL**:

- Prometheus 查询语言（PromQL），允许用户编写查询来获取存储的时间序列数据。这些查询可以通过 Prometheus 的 Web UI 或 Grafana 进行可视化。

**Prometheus Web UI 和 API Clients**:

- Prometheus 提供了一个 Web 界面和 API，用户可以通过这些界面查询、查看和分析存储的数据。

总结来说，Prometheus 通过 **Pull** 方式定期从监控目标获取数据，支持动态的服务发现并将数据存储到本地的 TSDB 中。通过 PromQL 可以查询数据并在 Grafana 等工具中进行可视化，告警信息则通过 Alertmanager 发送到不同的通知渠道。



```
pormetheus通过http拉取数据

APP --> exporter proxy(http server + data) --> http data --> prometheus
```



Prometheus 的主要模块包括：

- prometheus 

  时序数据存储、监控指标管理

- 可视化

  Prometheus web UI : 集群状态管理、promQL 

  Grafana:非常全面的可视化套件

- 数据采集

  Exporter: 为当前的客户端暴露出符合 Prometheus 规格的数据指标,Exporter 以守护进程的模式运行井开始采集数据,Exporter 本身也是一个http_server 可以对http请求作出响应返回数据 (K/V形式的metrics)

  Pushgateway : 拉模式下数据的采集工具

- 监控目标

  服务发现 :文件方式、dns方式、console方式、k8s方式

- 告警: 

  alertmanager 

Prometheus 由几个主要的软件组件组成，其职责概述如下

| 组件               | 解释                                                         |
| ------------------ | ------------------------------------------------------------ |
| Prometheus Server  | 彼此独立运行，仅依靠其本地存储来实现其核心功能：抓取时序数据，规则处理和警报等。 |
| Client Library     | 客户端库，为需要监控的服务生成相应的 metrics 并暴露给 Prometheus server。当 Prometheus server 来 pull 时，直接返回实时状态的 metrics。 |
| Push Gateway       | exporter采集类型已经很丰富，但是依然需要很多自定义的监控数据,用pushgateway可以实现自定义的监控数据,任意灵活想做什么都可以做到 exporter的开发需要使用真正的编程语言，不支持shell这种快速脚本,而 pushgateway开发去容易的多 pushgateway主要用于短期的 jobs。由于这类 jobs 存在时间较短，可能在 Prometheus 来 pull 之前就消失了。为此，这次 jobs 可以直接向 Prometheus server 端推送它们的 metrics。这种方式主要用于服务层面的 metrics，对于机器层 面的 metrices，需要使用 node exporter |
| Exporters          | 部署到第三方软件主机上，用于暴露已有的第三方服务的 metrics 给 Prometheus。 |
| Alertmanager       | 从 Prometheus server 端接收到 alerts 后，会进行去除重复数据，分组，并路由到 对应的接受方式，以高效向用户完成告警信息发送。常见的接收方式有：电子邮件， pagerduty，OpsGenie, webhook 等,一些其他的工具。 |
| Data Visualization | Prometheus Web UI （Prometheus Server内建），及Grafana等     |
| Service Discovery  | 动态发现待监控的Target，从而完成监控配置的重要组件，在容器化环境中尤为有 用；该组件目前由Prometheus Server内建支持； |

**工作流程**

- Prometheus server 定期从配置好的 jobs 或者 exporters 中拉 metrics，或者接收来自 Pushgateway 发过来的 metrics，或者从其他的 Prometheus server 中拉 metrics。
- Prometheus server 在本地存储收集到的 metrics，并运行已定义好的 alert.rules，记录新的时间序列或者向 Alertmanager 推送警报，实现一定程度上的完全冗余功能。
- Alertmanager 根据配置文件，对接收到的警报进行去重分组，根据路由配置，向对应主机发出告警。
- 集成Grafana或其他API作为图形界面，用于可视化收集的数据。

**生态组件**

```
https://prometheus.io/docs/instrumenting/exporters/
```



## **Prometheus** **部署和监控**

| 软件            | 地址                                                         |
| --------------- | ------------------------------------------------------------ |
| prometheus      | https://github.com/prometheus/prometheus/releases/download/v2.53.3/prometheus-2.53.3.linux-amd64.tar.gz |
| alertmanager    | https://github.com/prometheus/alertmanager/releases/download/v0.28.0/alertmanager-0.28.0.linux-amd64.tar.gz |
| node_exporter   | https://github.com/prometheus/node_exporter/releases/download/v1.9.0/node_exporter-1.9.0.linux-amd64.tar.gz |
| mysqld_exporter | https://github.com/prometheus/mysqld_exporter/releases/download/v0.17.2/mysqld_exporter-0.17.2.linux-amd64.tar.gz |
| grafana         | https://dl.grafana.com/enterprise/release/grafana-enterprise_11.5.2_amd64.deb |
| pushgateway     | https://github.com/prometheus/pushgateway/releases/download/v1.11.0/pushgateway-1.11.0.linux-amd64.tar.gz |

### **Prometheus** **部署和配置**

#### 包安装

```
[root@ubuntu2404 ~]#apt list prometheus
Listing... Done
prometheus/noble-security,noble-updates,noble-updates,noble-security 2.45.3+ds-2ubuntu0.2 amd64
N: There is 1 additional version. Please use the '-a' switch to see it
[root@ubuntu2404 ~]#apt update && apt install prometheus -y 
```

### **二进制安装** **Prometheus**

生产建议安装LTS版本

```bash
[root@ubuntu2404 ~]#tar xf prometheus-2.53.3.linux-amd64.tar.gz -C /usr/local/
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ls
bin  etc  games  include  lib  man  prometheus-2.53.3.linux-amd64  sbin  share  src
[root@ubuntu2404 local]#ln -s prometheus-2.53.3.linux-amd64/ prometheus
[root@ubuntu2404 local]#ls prometheus
console_libraries  consoles  LICENSE  NOTICE  prometheus  prometheus.yml  promtool
[root@ubuntu2404 prometheus]#mkdir conf bin
[root@ubuntu2404 prometheus]#mv prometheus promtool bin       
[root@ubuntu2404 prometheus]#mv prometheus.yml conf/
[root@ubuntu2404 prometheus]#ls
bin  conf  console_libraries  consoles  LICENSE  NOTICE
[root@ubuntu2404 prometheus]#ls bin/
prometheus  promtool
[root@ubuntu2404 prometheus]#ls conf/
prometheus.yml
[root@ubuntu2404 ~]#ln -s /usr/local/prometheus/bin/* /usr/local/bin/
[root@ubuntu2404 ~]#prometheus --config.file="/usr/local/prometheus/conf/prometheus.yml"
```

```bash
#创建启动用户
[root@ubuntu2404 ~]#useradd -r -s /sbin/nologin prometheus
[root@ubuntu2404 ~]#id prometheus 
uid=999(prometheus) gid=989(prometheus) groups=989(prometheus)

#创建server文件
[root@ubuntu2404 ~]#vim /lib/systemd/system/prometheus.service
[root@ubuntu2404 ~]#cat /lib/systemd/system/prometheus.service
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Restart=on-failure
User=prometheus
Group=prometheus
WorkingDirectory=/usr/local/prometheus/
ExecStart=/usr/local/prometheus/bin/prometheus --config.file=/usr/local/prometheus/conf/prometheus.yml
ExecReload=/bin/kill -HUP $MAINPID
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#chown -R prometheus:prometheus /usr/local/prometheus/
[root@ubuntu2404 ~]#systemctl daemon-reload 
[root@ubuntu2404 ~]#systemctl enable --now prometheus.service 
```



### **API** **访问**

注意：{ip:port} 是Prometheus所在的IP和端口

- 健康检查

  GET {ip:port}/-/healthy

  该端点始终返回200，应用于检查Prometheus的运行状况。

- 准备检查

  GET {ip:port}/-/ready

  当Prometheus准备服务流量（即响应查询）时，此端点返回200。

- 加载配置

  PUT {ip:port}/-/reload

  POST {ip:port}/-/reload

- 关闭服务

  PUT {ip:port}/-/quit

  POST {ip:port}/-/quit



### Prometheus 选项

```bash
[root@ubuntu2404 ~]#prometheus --help
...
--web.enable-lifecycle #可以支持http方式实现reload和shutdown功能，可以被远程关毕服务，有安全风险，不建议开启，默认关闭
--web.read-timeout=5m  #Maximum duration before timing out read of the request, and closing idle connections. 请求连接的最⼤等待时间,可以防⽌太多的空闲连接占⽤资源
--web.max-connections=512 #Maximum number of simultaneous connections. 最⼤链接数
--storage.tsdb.retention=15d #How long to retain samples in the storage.prometheus开始采集监控数据后 会存在内存中和硬盘中对于保留期限的设置,此值太长会导致硬盘和内存都吃不消,但太短要查历史数据就没有了,企业中一般设置15天为宜,默认值为0
--storage.tsdb.path="data/" #Base path for metrics storage. 存储数据路径,建议独立分区,防止把根⽬录塞满,默认data/目录
--query.timeout=2m #Maximum time a query may take before being aborted.此为默认值2m
--query.max-concurrency=20 #Maximum number of queries executed concurrently.此为默认值20
```

![image-20250301174748976](5day-png/34prometheus配置.png)



### docker部署

```bash
#简单启动
#默认容器的配置文件路径/etc/prometheus/prometheus.yml。数据存储路径默认为/prometheus
[root@prometheus ~]#docker run -d --name prometheus -p 9090:9090 prom/prometheus
#定制配置，持久化
[root@ubuntu2204 ~]#docker run -d --name prometheus -p 9090:9090 -v ./prometheus.yml:/etc/prometheus/prometheus.yml -v prometheus_data:/prometheus prom/prometheus:v2.53.0
```

```bash
#Docker compose部署


version: '3.6'
volumes:
  prometheus_data: {}
networks:
  monitoring:
    driver: bridge
services:
  prometheus:
    image: prom/prometheus:v2.40.2
    volumes:
      - ./prometheus/:/etc/prometheus/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - monitoring
    ports:
      - 9090:9090
    restart: always
```



### prometheus配置文件

```bash
[root@ubuntu2404 prometheus]#cat conf/prometheus.yml 
#全局配置块
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

#告警相关
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

#告警规则
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

#采集信息配置
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```



### nood-exporter

```bash
https://github.com/prometheus/node_exporter/releases/download/v1.9.0/node_exporter-1.9.0.linux-amd64.tar.gz

[root@ubuntu2404 ~]#ls
node_exporter-1.9.0.linux-amd64.tar.gz
[root@ubuntu2404 ~]#tar xvf node_exporter-1.9.0.linux-amd64.tar.gz -C /usr/local/
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ls
bin  etc  games  include  lib  man  node_exporter-1.9.0.linux-amd64  sbin  share  src
[root@ubuntu2404 local]#ln -s node_exporter-1.9.0.linux-amd64/ node_exporter
[root@ubuntu2404 local]#cd node_exporter
[root@ubuntu2404 node_exporter]#ls
LICENSE  node_exporter  NOTICE
[root@ubuntu2404 node_exporter]#mkdir bin
[root@ubuntu2404 node_exporter]#mv node_exporter bin/
[root@ubuntu2404 node_exporter]#ln -s /usr/local/node_exporter/bin/node_exporter /usr/local/bin/

#启动
[root@ubuntu2404 ~]#node_exporter 
[root@ubuntu2404 ~]#curl 10.0.0.201:9100/metrics
```

**准备** **service** **文件**

```bash
[root@ubuntu2404 ~]#useradd -s /sbin/nologin -r prometheus
[root@ubuntu2404 ~]#cat /lib/systemd/system/node-exporter.service
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
Type=simple
#no-collector.uname:禁止收集uname信息，会导致grafana无法显示此主机信息
#ExecStart=/usr/local/node_exporter/bin/node_exporter --no-collector.uname --
collector.cgroups
ExecStart=/usr/local/node_exporter/bin/node_exporter --collector.cgroups
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target
[root@ubuntu2404 ~]#systemctl daemon-reload 
[root@ubuntu2404 ~]#systemctl enable --now node-exporter.service 
Created symlink /etc/systemd/system/multi-user.target.wants/node-exporter.service → /usr/lib/systemd/system/node-exporter.service.
[root@ubuntu2404 ~]#systemctl status node-exporter.service 
```



**在 prometheus server 中增加监控主机**

```bash
[root@ubuntu2404 prometheus]#cat conf/prometheus.yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["10.0.0.201:9100"]
[root@ubuntu2404 prometheus]#systemctl reload prometheus.service 
```



**在prometheus server上安装node-exporter**

```bash
#脚本安装
[root@ubuntu2404 ~]#cat install_node_exporter.sh 
#!/bin/bash

#支持在线和离线安装，建议离线安装

NODE_EXPORTER_VERSION=1.9.0
#NODE_EXPORTER_VERSION=1.8.2
#NODE_EXPORTER_VERSION=1.8.1
#NODE_EXPORTER_VERSION=1.7.0
#NODE_EXPORTER_VERSION=1.5.0
#NODE_EXPORTER_VERSION=1.4.0
#NODE_EXPORTER_VERSION=1.3.1
#NODE_EXPORTER_VERSION=1.2.2

NODE_EXPORTER_FILE="node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz"
NODE_EXPORTER_URL=https://github.com/prometheus/node_exporter/releases/download/v${NODE_EXPORTER_VERSION}/${NODE_EXPORTER_FILE}

GITHUB_PROXY=https://mirror.ghproxy.com/

INSTALL_DIR=/usr/local

HOST=`hostname -I|awk '{print $1}'`


. /etc/os-release

msg_error() {
  echo -e "\033[1;31m$1\033[0m"
}

msg_info() {
  echo -e "\033[1;32m$1\033[0m"
}

msg_warn() {
  echo -e "\033[1;33m$1\033[0m"
}


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


install_node_exporter () {
    if [ ! -f  ${NODE_EXPORTER_FILE} ] ;then
        wget ${GITHUB_PROXY}${NODE_EXPORTER_URL} ||  { color "下载失败!" 1 ; exit ; }
    fi
    [ -d $INSTALL_DIR ] || mkdir -p $INSTALL_DIR
    tar xf ${NODE_EXPORTER_FILE} -C $INSTALL_DIR
    cd $INSTALL_DIR &&  ln -s node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64 node_exporter
    mkdir -p $INSTALL_DIR/node_exporter/bin
    cd $INSTALL_DIR/node_exporter &&  mv node_exporter bin/ 
    id prometheus &> /dev/null || useradd -r -s /sbin/nologin prometheus
    chown -R prometheus:prometheus ${INSTALL_DIR}/node_exporter/
      
    cat >  /etc/profile.d/node_exporter.sh <<EOF
export NODE_EXPORTER_HOME=${INSTALL_DIR}/node_exporter
export PATH=\${NODE_EXPORTER_HOME}/bin:\$PATH
EOF

}


node_exporter_service () {
    cat > /lib/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
Type=simple
ExecStart=$INSTALL_DIR/node_exporter/bin/node_exporter
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target
EOF
    systemctl daemon-reload
    systemctl enable --now node_exporter.service
}


start_node_exporter() { 
    systemctl is-active node_exporter.service
    if [ $?  -eq 0 ];then  
        echo 
        color "node_exporter 安装完成!" 0
        echo "-------------------------------------------------------------------"
        echo -e "访问链接: \c"
        msg_info "http://$HOST:9100/metrics" 
    else
        color "node_exporter 安装失败!" 1
        exit
    fi 
}

install_node_exporter

node_exporter_service

start_node_exporter
```

在prometheus server上监控node-exporter

```bash
[root@ubuntu2404 prometheus]#cat /usr/local/prometheus/conf/prometheus.yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: 
        - "10.0.0.201:9100"
        - "10.0.0.200:9100"
      #  - targets: ["10.0.0.200:9100","10.0.0.201:9100"]
#语法检查
[root@ubuntu2404 prometheus]#promtool check config /usr/local/prometheus/conf/prometheus.yml 
Checking /usr/local/prometheus/conf/prometheus.yml
 SUCCESS: /usr/local/prometheus/conf/prometheus.yml is valid prometheus config file syntax
 
[root@ubuntu2404 prometheus]#systemctl reload prometheus.service 
```

#### **容器化启动** **Node Exporter**

```
docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
```

docker compose 方式

```bash
version: '3.6'

networks:
  monitoring:
    driver: bridge

services:
  node-exporter:
    image: prom/node-exporter:v1.4.0
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
      - '--path.rootfs=/host'
    ports:
      - "9100:9100"
    networks:
      - monitoring
    restart: always
```



#### **Ansible** **部署** **Node Exporter**

```
https://github.com/prometheus-community/ansible/tree/main/roles/node_exporter
```



### Grafana部署

```bash
https://dl.grafana.com/enterprise/release/grafana-enterprise_11.5.2_amd64.deb
[root@ubuntu2404 ~]#apt install ./grafana-enterprise_11.5.2_amd64.deb
```

登录默认用户名密码：admin/admin

```
模板路径
https://grafana.com/dashboards/
```

模板

导入8919（中文）,1860,11074,13978模板

#### prometheus监控grafana

```bash
[root@ubuntu2404 prometheus]#cat /usr/local/prometheus/conf/prometheus.yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: 
        - "10.0.0.201:9100"
        - "10.0.0.200:9100"
      #  - targets: ["10.0.0.200:9100","10.0.0.201:9100"]
  - job_name: "grafana"
    static_configs:
      - targets: 
        - "10.0.0.200:3000"
```





### pushgateway

```bash
https://github.com/prometheus/pushgateway/releases/download/v1.11.0/pushgateway-1.11.0.linux-amd64.tar.gz

[root@ubuntu2404 ~]#tar xfv pushgateway-1.11.0.linux-amd64.tar.gz -C /usr/local/
pushgateway-1.11.0.linux-amd64/
pushgateway-1.11.0.linux-amd64/LICENSE
pushgateway-1.11.0.linux-amd64/pushgateway
pushgateway-1.11.0.linux-amd64/NOTICE
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ln -s pushgateway-1.11.0.linux-amd64/ pushgateway
[root@ubuntu2404 local]#cd pushgateway
[root@ubuntu2404 pushgateway]#ls
LICENSE  NOTICE  pushgateway
[root@ubuntu2404 pushgateway]#mkdir bin
[root@ubuntu2404 pushgateway]#mv pushgateway bin/
[root@ubuntu2404 pushgateway]#ln -s /usr/local/pushgateway/bin/pushgateway /usr/local/bin/
[root@ubuntu2404 pushgateway]#ll /usr/local/bin/
total 8
drwxr-xr-x  2 root root 4096 Mar  1 19:35 ./
drwxr-xr-x 13 root root 4096 Mar  1 19:34 ../
lrwxrwxrwx  1 root root   36 Mar  1 16:29 prometheus -> /usr/local/prometheus/bin/prometheus*
lrwxrwxrwx  1 root root   34 Mar  1 16:29 promtool -> /usr/local/prometheus/bin/promtool*
lrwxrwxrwx  1 root root   38 Mar  1 19:35 pushgateway -> /usr/local/pushgateway/bin/pushgateway*

#准备server文件
[root@ubuntu2404 pushgateway]#cat /lib/systemd/system/pushgateway.service
[Unit]
Description=Prometheus Pushgateway
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/pushgateway/bin/pushgateway
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 pushgateway]#systemctl daemon-reload 
[root@ubuntu2404 pushgateway]#systemctl enable --now pushgateway.service 
Created symlink /etc/systemd/system/multi-user.target.wants/pushgateway.service → /usr/lib/systemd/system/pushgateway.service.
[root@ubuntu2404 pushgateway]#systemctl status pushgateway.service 
```

```
http://10.0.0.200:9091/metrics
```

#### prometheus监控pushgateway

```bash
[root@ubuntu2404 prometheus]#cat /usr/local/prometheus/conf/prometheus.yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: 
        - "10.0.0.201:9100"
        - "10.0.0.200:9100"
      #  - targets: ["10.0.0.200:9100","10.0.0.201:9100"]
  - job_name: "grafana"
    static_configs:
      - targets: 
        - "10.0.0.200:3000"
  - job_name: "pushgateway"
    static_configs:
      - targets: 
        - "10.0.0.200:9091"
```

```bash
 - job_name: "pushgateway"
   honor_labels: true    #可选项，设置为true，Prometheus将使用Pushgateway上的job和instance标签。如果设置为false那么它将重命名这些值，在它们前面加上exported_前缀，并在服务器上为这些标签附加新值。
   scrape_interval: 10s  #可选项
   static_configs:
     - targets: #可以写在同一行,比如:["pushgateway.wang.org:9091"]
       - "pushgateway.wang.org:9091"
```



#### **Docker** **安装**

```
docker pull prom/pushgateway
docker run -d -p 9091:9091 prom/pushgateway
```



#### ansible安装

```
https://github.com/prometheus-community/ansible/tree/main/roles/pushgateway
```



#### **配置客户端发送数据给** **Pushgateway**

Examples:

- Push a single sample into the group identified by `{job="some_job"}`:

  ```
    echo "some_metric 3.14" | curl --data-binary @- http://pushgateway.example.org:9091/metrics/job/some_job
  ```

  Since no type information has been provided, `some_metric` will be of type `untyped`.

- Push something more complex into the group identified by `{job="some_job",instance="some_instance"}`:

  ```
    cat <<EOF | curl --data-binary @- http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance
    # TYPE some_metric counter
    some_metric{label="val1"} 42
    # TYPE another_metric gauge
    # HELP another_metric Just an example.
    another_metric 2398.283
    EOF
  ```

  Note how type information and help strings are provided. Those lines are optional, but strongly encouraged for anything more complex.

- Delete all metrics in the group identified by `{job="some_job",instance="some_instance"}`:

  ```
    curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance
  ```

- Delete all metrics in the group identified by `{job="some_job"}` (note that this does not include metrics in the `{job="some_job",instance="some_instance"}` group from the previous example, even if those metrics have the same job label):

  ```
    curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job
  ```

- Delete all metrics in all groups (requires to enable the admin API via the command line flag `--web.enable-admin-api`):

  ```
    curl -X PUT http://pushgateway.example.org:9091/api/v1/admin/wipe
  ```

```bash
POST	insert
PUT		update
GET		select
DELECT	delect
```

```bash
[root@ubuntu2404 ~]#echo 'age 18' | curl --data-binary @- http://10.0.0.200:9091/metrics/job/pusggateway/instance/`hostname -I`

[root@ubuntu2404 ~]#echo "login_number `ss -nt|grep -c ESTAB`" | curl --data-binary @- http://10.0.0.200:9091/metrics/job/pusggateway/instance/`hostname -I`
```

![image-20250301195617896](5day-png/34pushgateway.png)

![image-20250301195723115](5day-png/34pushgateway1.png)

#### **数据格式**



Prometheus 对收集的数据有一定的格式要求，本质上就是将收集的数据转化为对应的文本格式，并提供响应给Prometheus Server的 http 请求。

Exporter 收集的数据转化的文本内容以行 (\n) 为单位，空行将被忽略, 文本内容最后一行为空行

文本内容，如果以 # 开头通常表示注释。

以 # HELP 开头表示 metric 帮助说明。

以 # TYPE 开头表示定义 metric 类型，包含 counter, gauge, histogram, summary, 和 untyped 类型。

其他表示一般注释，供阅读使用，将被 Prometheus 忽略。

```bash
[root@ubuntu2404 ~]#cat pushgateway_metric.sh 
#!/bin/bash
#

METRIC_NAME=mem_free
METRIC_VALUE_CMD="free -b  | awk 'NR==2{print \$4}'"
#METRIC_VALUE_CMD="free -b  | awk 'NR==2'| tr  -s ' ' | cut -d' ' -f4"
METRIC_TYPE=gauge
METRIC_HELP="free memory"

PUSHGATEWAY_HOST=10.0.0.200:9091
EXPORTED_JOB=pushgateway_test
INSTANCE=`hostname -I|awk '{print $1}'`
SLEEP_TIME=1


CURL_URL="curl --data-binary @- http://${PUSHGATEWAY_HOST}/metrics/job/${EXPORTED_JOB}/instance/${INSTANCE}"

push_metric()  {
    while true ;do
        VALUE=`eval "$METRIC_VALUE_CMD"`
        echo $VALUE
        cat  <<EOF |  $CURL_URL
# HELP ${METRIC_NAME} ${METRIC_HELP}
# TYPE ${METRIC_NAME} ${METRIC_TYPE}
${METRIC_NAME} ${VALUE}
EOF
        sleep $SLEEP_TIME
    done
}

push_metric
```

