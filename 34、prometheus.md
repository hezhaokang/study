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

### 脚本安装

```bash
[root@ubuntu2404 ~]#cat install_prometheus.sh 
#!/bin/bash

#支持在线和离线安装,建议离线安装,在线下载很慢

PROMETHEUS_VERSION=2.53.3
#PROMETHEUS_VERSION=3.0.1
#PROMETHEUS_VERSION=2.53.2
#PROMETHEUS_VERSION=2.53.0
#PROMETHEUS_VERSION=2.50.1
#PROMETHEUS_VERSION=2.42.0
#PROMETHEUS_VERSION=2.39.1
#PROMETHEUS_VERSION=2.37.0
#PROMETHEUS_VERSION=2.30.3
#PROMETHEUS_VERSION=2.17.1

PROMETHEUS_FILE="prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz"
#PROMETHEUS_URL="https://mirrors.tuna.tsinghua.edu.cn/github-release/prometheus/prometheus/LatestRelease/${PROMETHEUS_FILE}"
PROMETHEUS_URL="https://github.com/prometheus/prometheus/releases/download/v${PROMETHEUS_VERSION}/${PROMETHEUS_FILE}"

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


install_prometheus () {
    if [ ! -f  ${PROMETHEUS_FILE} ] ;then
        wget ${GITHUB_PROXY}${PROMETHEUS_URL} ||  { color "下载失败!" 1 ; exit ; }
    fi
    [ -d $INSTALL_DIR ] || mkdir -p $INSTALL_DIR
    tar xf ${PROMETHEUS_FILE} -C $INSTALL_DIR
    cd $INSTALL_DIR &&  ln -s prometheus-${PROMETHEUS_VERSION}.linux-amd64 prometheus
    mkdir -p $INSTALL_DIR/prometheus/{bin,conf,data}
    cd $INSTALL_DIR/prometheus && { mv prometheus promtool bin/ ; mv prometheus.yml conf/; }
    id prometheus &>/dev/null || useradd -r -s /sbin/nologin prometheus
    chown -R prometheus:prometheus ${INSTALL_DIR}/prometheus/
    
    cat >>  /etc/profile <<EOF
export PROMETHEUS_HOME=${INSTALL_DIR}/prometheus
export PATH=\${PROMETHEUS_HOME}/bin:\$PATH
EOF

}


prometheus_service () {
    cat > /lib/systemd/system/prometheus.service <<EOF
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Restart=on-failure
User=prometheus
Group=prometheus
WorkingDirectory=${INSTALL_DIR}/prometheus
ExecStart=${INSTALL_DIR}/prometheus/bin/prometheus --config.file=${INSTALL_DIR}/prometheus/conf/prometheus.yml --web.enable-lifecycle
ExecReload=/bin/kill -HUP \$MAINPID
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
    systemctl daemon-reload
    systemctl enable --now prometheus.service
}


start_prometheus() { 
    systemctl is-active prometheus
    if [ $?  -eq 0 ];then  
        echo 
        color "Prometheus 安装完成!" 0
        echo "-------------------------------------------------------------------"
        echo -e "访问链接: \c"
        msg_info "http://$HOST:9090/" 
    else
        color "Prometheus 安装失败!" 1
        exit
    fi 
}

install_prometheus

prometheus_service

start_prometheus
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

#### 二进制部署

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



#### **在prometheus server上脚本安装node-exporter**

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

#### 部署

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

#### 脚本安装

```bash
[root@ubuntu2404 data]#cat install_pushgateway.sh 
#!/bin/bash

#支持在线和离线安装，建议离线

PUSHGATEWAY_VERSION=1.11.0
#PUSHGATEWAY_VERSION=1.10.0
#PUSHGATEWAY_VERSION=1.9.0
#PUSHGATEWAY_VERSION=1.7.0
#PUSHGATEWAY_VERSION=1.5.1
#PUSHGATEWAY_VERSION=1.4.3
PUSHGATEWAY_FILE="pushgateway-${PUSHGATEWAY_VERSION}.linux-amd64.tar.gz"
PUSHGATEWAY_URL=https://github.com/prometheus/pushgateway/releases/download/v${PUSHGATEWAY_VERSION}/${PUSHGATEWAY_FILE}

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


install_pushgateway () {
    if [ ! -f  ${PUSHGATEWAY_FILE} ] ;then
        wget ${GITHUB_PROXY}${PUSHGATEWAY_URL} ||  { color "下载失败!" 1 ; exit ; }
    fi
    [ -d $INSTALL_DIR ] || mkdir -p $INSTALL_DIR
    tar xf ${PUSHGATEWAY_FILE} -C $INSTALL_DIR
    cd $INSTALL_DIR &&  ln -s pushgateway-${PUSHGATEWAY_VERSION}.linux-amd64 pushgateway
    mkdir -p $INSTALL_DIR/pushgateway/bin
    cd $INSTALL_DIR/pushgateway &&  mv pushgateway bin/ 
        id prometheus &>/dev/null || useradd -r -s /sbin/nologin prometheus
        chown -R prometheus.prometheus $INSTALL_DIR/pushgateway/
    ln -s $INSTALL_DIR/pushgateway/bin/pushgateway /usr/local/bin/
}


pushgateway_service () {
    cat > /lib/systemd/system/pushgateway.service <<EOF
[Unit]
Description=Prometheus Pushgateway
After=network.target

[Service]
Type=simple
ExecStart=$INSTALL_DIR/pushgateway/bin/pushgateway
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
User=prometheus
Group=prometheus


[Install]
WantedBy=multi-user.target
EOF
    systemctl daemon-reload
    systemctl enable --now pushgateway.service
}


start_pushgateway() { 
    systemctl is-active pushgateway.service
    if [ $?  -eq 0 ];then  
        echo 
        color "pushgateway 安装完成!" 0
        echo "-------------------------------------------------------------------"
        echo -e "访问链接: \c"
        msg_info "http://$HOST:9091" 
    else
        color "pushgateway 安装失败!" 1
        exit
    fi 
}

install_pushgateway

pushgateway_service

start_pushgateway
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

 

## **PromQL**

### **指标数据**

时间序列数据：

- 按照时间顺序记录系统、设备状态变化的数据，每个数据称为一个样本
- 数据采集以特定的时间周期进行，随着时间将这些样本数据记录下来，将生成一个离散的样本数据序列,该序列也称为向量（Vector）
- 将多个序列放在同一个坐标系内（以时间为横轴，以序列为纵轴），将形成一个由数据点组成的矩阵

Prometheus基于指标名称（metrics name）以及附属的标签集（labelset）唯一定义一条时间序列

- 指标名称代表着监控目标上某类可测量属性的基本特征标识
- 标签则是这个基本特征上再次细分的多个可测量维度

**数据模型**

Prometheus中，每个时间序列都由指标名称（Metric Name）和标签（Label）来唯一标识

Metric Name的表示方式有下面两种

```powershell
<metric name>{<label name>=<label value>, …}
{__name__="metric name",<label name>=<label value>, …} #通常用于Prometheus内部
```

![image-20250303191513767](5day-png/34PromQL数据模型.png)

**指标名称：**

- 通常用于描述系统上要测定的某个特征
- 支持使用字母、数字、下划线和冒号，且必须能匹配RE2规范的正则表达式
- 例如：http_requests_total表示接收到的HTTP请求总数

**标签：**

- 键值型数据，附加在指标名称之上，从而让指标能够支持更多细化的多纬度特征；此为可选项
- 标签名称可使用字母、数字和下划线，且必须能匹配RE2规范的正则表达式
- 注意：以两个下划线 "__" 为前缀的名称为Prometheus系统预留使用

Prometheus的每个数据样本由两部分组成

- key: 包括三部分Metric 名称,Label, Timestamp(毫秒精度的时间戳)
- value: float64格式的数据

![image-20250303191728858](5day-png/34PromQL数据样本.png)

PromQL支持基于定义的指标维度进行过滤，统计和聚合

- 指标名称和标签的特定组合代表着一个时间序列
- 不同的指标名称代表着不同的时间序列
- 指标名称相同，但标签不同的组合分别代表着不同的时间序列
- 更改任何标签值，包括添加或删除标签，都会创建一个新的时间序列
- 应该尽可能地保持标签的稳定性，否则，则很可能创建新的时间序列，更甚者会生成一个动态的数据环境，并使得监控的数据源难以跟踪，从而导致建立在该指标之上的图形、告警及记录规则变得无效

### **表达式形式**

每一个PromQL其实都是一个表达式，这些语句表达式或子表达式的计算结果可以为以下四种类型：

- instant vector 即时向量,瞬时数据
  - 具有相同时间戳的一组样本值的集合
  - 在某一时刻，抓取的所有监控项数据。这些度量指标数据放在同一个key中。
  - 一组时间序列，包含每个时间序列的单个样本，所有时间序列共享相同的时间戳
- range vector 范围向量
  - 指定时间范围内的所有时间戳上的数据指标，即在一个时间段内，抓取的所有监控项数据。
  - 一组时间序列，其中包含每个时间序列随时间变化的一系列数据点
- scalar 标量
  - 一个简单的浮点类型数值
- string 字符串
  - 一个简单字符串类型, 当前并没有使用, a simple string value; currently unused

范例: 利用API查询数据

```bash
#查看现在的时间
[root@ubuntu2404 ~]#date +%s
1741001706
查看现在的数据
[root@ubuntu2404 ~]#curl --data 'query=prometheus_http_requests_total' --data time=1741001706 'http://10.0.0.200:9090/api/v1/query' | jq
```

**基本语法**

数值

对于数值来说，主要记住两种类型的数值：字符串和数字。

字符串

```bash
#字符串可以用单引号，双引号或反引号指定为文字，如果字符串内的特殊符号想要生效，可以使用反引号。
"this is a string"
'these are unescaped: n t'
`these are not unescaped: n ' " t`
```

数字

```bash
#对于数据值的表示，可以使用我们平常时候的书写方法 "[-](digits)[.(digits)]"
2、2.43、-2.43等
```

**表达式使用要点**

表达式的返回值类型是即时向量、范围向量、标量或字符串4种数据类型其中之一，但是，有些使用场景要求表达式返回值必须满足特定的条件，例如

- 需要将返回值绘制成图形时,仅支持即时向量类型的数据
- 对于诸如rate一类的速率函数来说，其要求使用的却又必须是范围向量型的数据
- 由于范围向量选择器的返回的是范围向量型数据，它不能用于在浏览器中表达式图形绘制功能，否则，表达式浏览器会返回"Error executing query: invalid expressiontype "range vector" for range query,must be Scalar or instant Vector"一类的错误
- 范围向量选择几乎总是结合速率类的函数rate一同使用



**数据选择器**

所谓的数据选择器，其实指的是获取实时数据或者历史数据的一种方法

样式

```
metrics_name{筛选label=值,...}[<时间范围>] offset <偏移>
y	年
w	周
d	天
h	时
m	分
s	秒
ms	毫秒
```

```bash
#五分钟之前的值
prometheus_http_requests_total{code="200", handler="/api/v1/query", instance="localhost:9090", job="prometheus"} offset 5m

#现在到五分钟之前的所有数据
prometheus_http_requests_total{code="200", handler="/api/v1/query", instance="localhost:9090", job="prometheus"} [5m]

#五分钟之前的前三分钟的全部数据
prometheus_http_requests_total{code="200", handler="/api/v1/query", instance="localhost:9090", job="prometheus"} [3m] offset 5m  
```

```bash
#15s采集一次数据
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml 
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
```

#### 即时向量选择器由两部分组成

- 指标名称:用于限定特定指标下的时间序列，即负责过滤指标;可选
- 匹配器(Matcher) :或称为标签选择器，用于过滤时间序列上的标签;定义在{}之中;可选
- 定义即时向量选择器时，以上两个部分应该至少给出一个

根据数据的精确度，可以有以下几种使用方法：

- 根据监控项名称获取最新值

  仅给定指标名称，或在标签名称上使用了空值的匹配器

  返回给定的指标下的所有时间序列各自的即时样本

  例如：http_requests_total和http_requests_total{}的功能相同，都是用于返回http_requests_total指标下各时间序列的即时样本

  ```bash
  http_requests_total
  http_requests_total{}
  ```

- 通过 `__name__` 匹配多个监控项的名称

  ```bash
  {__name__="prometheus_http_requests_total"}
  {__name__=~"^prometheus_http_.*"}
  ```

- 通过` __name__ `匹配多个监控项的名称

  ```bash
  {__name__="prometheus_http_requests_total"}
  {__name__=~"^prometheus_http_.*"}
  ```

- 仅给定匹配器

  返回所有符合给定的匹配器的所有时间序列上的即时样本

  注意:这些时间序列可能会有很多的不同的指标名称

  例如：{job=".*", method="get"}

- 指标名称和匹配器的组合

  通过 name{key=value,...}样式获取符合条件的数据值

  返回给定的指定下的，且符合给定的标签过滤器的所有时间序列上的即时样本

  例如： http_requests_total{method="get"}

范例

```bash
{__name__="prometheus_http_requests_total",handler="/-/reload"}
prometheus_http_requests_total{handler="/-/reload"}
```



**匹配器** **Matcher** **使用规则**

**匹配器用于定义标签过滤条件，目前支持如下4种匹配操作符**

```bash
=  #精确匹配
!= #不匹配
=~ #正则匹配,全部匹配，而非包含
!~ #正则不匹配
```

**匹配到空标签值的匹配器时，所有未定义该标签的时间序列同样符合条件**

```
http_requests_total{env=""} 则该指标名称上所有未使用该标签env的时间序列或者env的值为空的都符合条件
比如时间序列http_requests_total {method ="get"}
如果想查询所有 HTTP 请求总数，直接使用 http_requests_total 即可。
想排除 env 为空的数据，可以使用 http_requests_total{env!=""}
```



**正则表达式将执行完全锚定机制，它需要匹配指定的标签的整个值**

```
http_requests_total {method =~"^ge"} 是错误的，应该是http_requests_total {method =~"^ge.*"}
```



**多个条件间可以使用逗号**","**隔开，每个条件内部可以通过多种符号，表示不同含义**

**如果条件中存在多值，可以使用"|"表示或**

```
比如：env=~"staging|testing|development"
```



**向量选择器至少要包含一个指标名称**,**或者条件中至少包含一个非空标签值的选择器**

```bash
#示例：
{__name__=~"http_requests_.*"}
#能够匹配所有以"http_requests_"为前缀的所有指标
```

```bash
#示例
prometheus_http_requests_total{instance="localhost:9090", job="prometheus"}
prometheus_http_requests_total{handler=~".*meta.*"}
node_memory_MemFree_bytes{instance=~"10.0.0.(101|102):9100"} 

#注意：指标 prometheus_http_requests_total 默认情况下，针对的是 localhost:9090 的target，其他无效
```



#### **范围选择器** **Range Vector Selector**

Range Vector Selector 工作方式与瞬时向量选择器一样，区别在于时间范围长一些

返回0个、1个或多个时间序列上在给定时间范围内的各自的一组样本值

主要是在瞬时选择器多了一个[]格式的时间范围后缀

在[]内部可以采用多个单位表示不同的时间范围，比如s(秒)、m(分)、h(时)、d(日)、w(周)、y(年)

必须使用整数时间单位，且能够将多个不同级别的单位进行串联组合，以时间单位由大到小为顺序

例如：1h30m，但不能使用1.5h

```bash
prometheus_http_requests_total{job="prometheus"}[5m]
#属性解析：这表示过去5分钟内的监控数据值，这些数据一般以表格方式展示，而不是列表方式展示
```

注意：

- 范围向量选择器返回的是一定时间范围内的数据样本，虽然不同时间序列的数据抓取时间点相同，但它们的时间戳并不会严格对齐
- 多个Target上的数据抓取需要分散在抓取时间点前后一定的时间范围内，以均衡Prometheus Server的负载
- 因而，Prometheus在趋势上准确，但并非绝对精准

#### **偏移修饰符** **offset**

默认情况下，即时向量选择器和范围向量选择器都以当前时间为基准时间点，而偏移量修改器能够修改该基准

对于某个历史时间段中的数据，需要通过offset时间偏移的方式来进行获取

偏移量修改器的使用方法是紧跟在选择器表达式之后使用"offset"关键字指定

注意：offset与数据选择器是一个整体，不能分割，offset 偏移的是时间基点

```bash
prometheus_http_requests_total offset 5m 
#表示获取以prometheus_http_requests_total为指标名称的所有时间序列在过去5分钟之时的即时样本
prometheus_http_requests_total{code="200"} offset 5m
#如果既有偏移又有范围,先偏移后再取范围,如[5m] offset 3m 表示取当前时间的3分钟前的5m范围的值
http_requests_total[5m] offset 1d 
#表示获取距此刻1天时间之前的5分钟之内的所有样本
http_requests_total{handler="/metrics"}[5m] offset 3m
```



### **指标类型**

Prometheus客户端库提供了四种核心度量标准类型

```
[root@ubuntu2404 ~]#curl -s http://10.0.0.200:9090/metrics |awk '$2=="TYPE"{type[$NF]++}END{for(i in type){print type[i],i}}'
87 gauge
15 histogram
13 summary
102 counter
```

| 类型              | 解析                                                         |
| ----------------- | ------------------------------------------------------------ |
| Counter -计数器   | counter是一个累加的计数器，代表一个从0开始累积单调递增的计数器，其值只能在重 新启动时增加或重置为零。典型的应用如：用户的访问量,请求的总个数，任务的完成数 量或错误的总数量等。不能使用Counter来表示递减值。但可以重置为0，即重新计数 |
| Gauge -计 量器    | Gauge是一种度量标准，只有一个简单的返回值，或者叫瞬时状态，可以代表可以任意 metric的上下波动的数值。通常用于指定时间的测量值，例如，硬盘剩余空间,当前的内 存使用量,一个待处理队列中任务的个数等，还用于可能上升和下降的“计数”，例如, 并发 请求数。 |
| Histogram- 直方图 | Histogram统计数据的分布情况。比如最小值，最大值，中间值，还有中位数，75百分 位,90百分位, 95百分位.98百分位,99百分位,和9.9百分位的值(percenties ,代表着近似的 百分比估算数值  比如: 每天1000万请求,统计http_response_time不同响应时间的分布情况,响应时间在0-0.05s有多少,0.05s到2s有多少,10s以上有多少 每个存储桶都以"_BucketFuncName{...}" 样式来命名.例如 hist_sum、hist_count等 可以基于histogram_quantile()函数对直方图甚至是直方图的聚合来进行各种分析计算。 比如: 统计考试成绩在0-60分之间有多少,60-80之间有多少个,80-100分之间有多少个 示例:prometheus_tsdb_compaction_chunk_range_seconds_bucket histogram中位数由服务端计算完成，对于分位数的计算而言，Histogram则会消耗更多 的资源 |
| Summary- 摘要     | 和Histogram类似，用于表示一段时间内的数据采样结果,典型的应用如：请求持续时 间，响应大小。它直接存储了分位数(将一个随机变量的概率分布范围分为几个等份的数 值点，常用的有中位数即二分位数、四分位数、百分位数等)，而不是通过区间来计算。 类似于直方图，摘要会基于阶段性的采样观察结果进行信息描述。它还提供了观测值的 累计计算、百分位计算等功能，每个摘要的命名与Histogram 类似，例如： summary_sum、summary_count等 示例: go_gc_duration_seconds Sumamry的分位数则是直接在客户端计算完成，对于分位数的计算而言，Summary在 通过PromQL进行查询时有更好的性能表现 |



**Counter和Gauge**

通常情况不会直接使用Counter总数，而是需要借助于rate、topk、incrcase和irate等函数来生成样本数据的变化状况(增长率）

```bash
topk(3, http_requests_total)， #获取该指标下http请求总数排名前3的时间序列
rate(http_requests_total[2h])，#获取2小时内，各时间序列上的http总请求数的增长速率，此为平均值，无法反映近期的精确情况
irate(http_requests_total[2h]) #高灵敏度函数，用于计算指标的瞬时速率，基于样本范围内的最后两个样本进行计算，相较于rate函数来说，irate更适用于精准反映出短期时间范围内的变化速率
```

Gauge用于存储其值可增可减的指标的样本数据，常用于进行求和、取平均值、最小值、最大值等聚合计算

也会经常结合PromQL的predict_linear和delta函数使用

predict_linear(v range-vector, t, scalar)函数可以预测时间序列v在t秒后的值，它通过线性回归的方式来预测样本数据的Gauge变化趋势

delta(v range-vector)函数计算范围向量中每个时间序列元素的第一个值与最后一个值之差，从而展示不同时间点上的样本值的差值

```
delta(cpu_temp_celsius {host="prometheus.wang.org"[2h) #返回该服务器上的CPU温度与2小时之前的差异
```



### **PromQL** **运算**

对于PromQL来说，它的操作符号主要有以下两类：

- 二元运算符
- 聚合运算

#### 二元运算符

是prometheus进行数据可视化或者数据分析操作的时候，应用非常多的一种功能

对于二元运算符来说，它主要包含三类：算术、比较、逻辑

```bash
#算术运算符：
+ (addition)
- (subtraction)
* (multiplication)
/ (division)
% (modulo)
^ (power/exponentiation)

#比较运算符：
== (equal)
!= (not-equal)
> (greater-than)
< (less-than)
>= (greater-or-equal)
<= (less-or-equal)

#逻辑运算符：
and、or、unless 
#目前该运算符仅允许在两个即时向量之间进行操作，不支持标量(标量只有一个数字，没有时序)参与运算

#运算符从高到低的优先级
1 ^ 
2 *, /, %
3 +, -
4 ==, !=, <=, <, >=, >
5 and, unless
6 or

#注意：
#具有相同优先级的运算符满足结合律（左结合)，但幂运算除外，因为它是右结合机制
#可以使用括号()改变运算次序
```

范例：取GC平均值

```
go_gc_duration_seconds_sum{instance="10.0.0.200:9090",job="prometheus"}/go_gc_duration_seconds_count{instance="10.0.0.200:9090",job="prometheus"}
```

范例：

```powershell
#正则表达式
node_memory_MemAvailable_bytes{instance =~ "10.0.0.20.:9100"}
node_memory_MemAvailable_bytes{instance =~ "10.0.0.20[12]:9100"}
node_memory_MemAvailable_bytes{instance =~ "10.0.0.20[1-3]:9100"}
node_memory_MemAvailable_bytes{instance =~ "10.0.0.20.*:9100"}
node_memory_MemAvailable_bytes{instance =~ "10.0.0.20[0-9]:9100"}
node_memory_MemAvailable_bytes{instance =~ "10.0.0.20[^01]:9100"}
node_memory_MemAvailable_bytes{instance !~ "10.0.0.20[12]:9100"}

#单位换算
node_memory_MemFree_bytes / (1024 * 1024)

#可用内存占用率
node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes * 100

#内存使用率
(node_memory_MemTotal_bytes - node_memory_MemFree_bytes) / node_memory_MemTotal_bytes * 100

#磁盘使用率
(node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_free_bytes{mountpoint="/"}) /node_filesystem_size_bytes{mountpoint="/"} * 100

#阈值判断
(node_memory_MemTotal_bytes - node_memory_MemFree_bytes) / node_memory_MemTotal_bytes > 0.95

#内存利用率是否超过80
( 1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes ) * 100 > bool 80

#布尔值,当超过1000为1,否则为0
prometheus_http_requests_total > bool 1000
node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes * 100 > bool 80

#注意：
对于比较运算符来说，条件成立有结果输出，否则没有结果输出
使用bool运算符后，布尔运算不会对时间序列进行过滤，而是直接依次瞬时向量中的各个样本数据与标量的比较结果0或者1。从而形成一条新的时间序列。
```

集合处理

```bash
#并集 or
node_memory_MemTotal_bytes{instance="10.0.0.104:9100"} or node_memory_MemFree_bytes{instance="10.0.0.105:9100" } 

node_memory_MemTotal_bytes{instance=~"10.0.0.(103|104):9100"} 

#交集 and
node_memory_MemTotal_bytes{instance="10.0.0.200:9100"} and node_memory_MemFree_bytes{instance="10.0.0.201:9100" }

#补集 unless：排除
#排除200
node_memory_MemTotal_bytes{job="node_exporter"} unless node_memory_MemTotal_bytes{instance="10.0.0.200:9100", job="node_exporter"}

node_memory_MemTotal_bytes{job="k8s-node"} unless 
node_memory_MemTotal_bytes{instance="10.0.0.104:9100"}
 
#注意：
and、or、unless 主要是针对获取的数据值进行条件选集合用的
and、or、unless 针对的对象是一个完整的表达式
```

向量匹配关健字

```bash
#https://prometheus.io/docs/prometheus/latest/querying/operators/#group-modifiers
ignoring 	#定义匹配检测时要忽略的标签
on  		#定义匹配检测时只使用的标签
```

一对一向量匹配

```bash
#https://prometheus.io/docs/prometheus/latest/querying/operators/#one-to-one-vector-matches
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

多对一和一对多向量匹配

```
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

#### **1. `ignoring()` vs. `on()`**

这两个关键字用于控制**如何匹配时间序列**：

- **`on(<label list>)`**：仅使用 `<label list>` 指定的标签进行匹配，忽略其他标签。
- **`ignoring(<label list>)`**：忽略 `<label list>` 指定的标签，使用**剩余的所有标签**进行匹配。

**示例**

假设有两个指标：

**(1) HTTP 请求总数**

```
plaintext复制编辑http_requests_total{job="api", instance="server1", method="GET"}  100
http_requests_total{job="api", instance="server2", method="POST"} 200
```

**(2) 请求处理时间**

```
plaintext复制编辑request_duration_seconds{job="api", instance="server1"}  0.5
request_duration_seconds{job="api", instance="server2"}  0.7
```

我们希望计算**每个实例的平均请求时间**，即：

```
request_duration_seconds / http_requests_total
```

但是 `http_requests_total` 有 `method` 标签，而 `request_duration_seconds` 没有，因此它们无法直接匹配。

**使用 `on()`**

```
request_duration_seconds on(instance) / http_requests_total
```

**匹配逻辑**：

- 仅使用 `instance` 标签进行匹配，**忽略 `job`、`method`** 等其他标签。
- 这样 `request_duration_seconds{instance="server1"}` 可以匹配 `http_requests_total{instance="server1"}`。

**使用 `ignoring()`**

```
request_duration_seconds ignoring(method) / http_requests_total
```

**匹配逻辑**：

- 忽略 `method` 标签，使用剩下的 `job` 和 `instance` 进行匹配。
- 适用于 **某些标签不同，但可以忽略的情况**。



#### **2.`group_left()` vs. `group_right()`**

如果左右两个向量的标签匹配存在 **一对多（1:N）或多对一（N:1）关系**，需要 `group_left()` 或 `group_right()` 来解决匹配问题。

- **`group_left(<extra_labels>)`**：用于**右侧向量有多个匹配项**，即 **1:N** 匹配。
- **`group_right(<extra_labels>)`**：用于**左侧向量有多个匹配项**，即 **N:1** 匹配。

**示例**

假设：

**(1) HTTP 请求总数**

```
plaintext复制编辑http_requests_total{job="api", instance="server1", method="GET"}  100
http_requests_total{job="api", instance="server1", method="POST"} 200
http_requests_total{job="api", instance="server2", method="GET"}  150
http_requests_total{job="api", instance="server2", method="POST"} 250
```

**(2) 请求错误率**

```
plaintext复制编辑error_rate{job="api", instance="server1"}  0.05
error_rate{job="api", instance="server2"}  0.02
```

我们想计算**按 `method` 统计的错误请求总数**：

```
error_rate * http_requests_total
```

但 `error_rate` **没有 `method` 标签**，而 `http_requests_total` **有 `method` 标签**，导致无法匹配。

**使用 `group_left(method)`**

```
error_rate on(instance) group_left(method) * http_requests_total
```

**匹配逻辑**：

- `error_rate` 只有 `instance`，`http_requests_total` 有 `instance` 和 `method`。
- `group_left(method)` 让 `error_rate` **继承 `method` 标签**，从而正确计算每种方法的错误率。



#### **3. `group_right()` 示例**

假设有：

**(1) 服务 CPU 负载**

```
cpu_usage{job="api", instance="server1"}  80
cpu_usage{job="api", instance="server2"}  70
```

**(2) 实例信息**

```
instance_info{job="api", instance="server1", team="devops"}  1
instance_info{job="api", instance="server2", team="backend"} 1
instance_info{job="api", instance="server2", team="frontend"} 1
```

**问题**：`instance_info` 里 `server2` **有两个 `team` 标签值**，而 `cpu_usage` 只有 `instance`，**1:N 匹配问题**。

**使用 `group_right(team)`**

```
cpu_usage on(instance) group_right(team) * instance_info
```

**匹配逻辑**：

- `cpu_usage` 只有 `instance`，而 `instance_info` 有 `team`。
- `group_right(team)` 让 `cpu_usage` **继承 `team` 标签**，从而生成多个 `team` 维度的 `cpu_usage`。

------

#### **4. 总结**

| 方式                        | 作用                                                   | 适用场景                                |
| --------------------------- | ------------------------------------------------------ | --------------------------------------- |
| `on(labels)`                | 仅按指定的 `labels` 匹配，忽略其他                     | 两个向量标签不同，必须指定关键匹配标签  |
| `ignoring(labels)`          | 忽略指定的 `labels`，使用剩余的进行匹配                | 某些标签不同，但可以忽略                |
| `group_left(extra_labels)`  | 右侧数据多（1:N），让左侧**继承**右侧的 `extra_labels` | 计算 per-label 数据（如 `method` 维度） |
| `group_right(extra_labels)` | 左侧数据多（N:1），让右侧**继承**左侧的 `extra_labels` | 计算 per-group 数据（如 `team` 维度）   |



#### **聚合操作**

常见的11种聚合操作：

```bash
sum、min、max、avg、count、count_values(值计数)stddev(标准差)、stdvar(标准差异)、bottomk(最小取样)、topk(最大取样，即取前几个)、quantile(分布统计)

sum() 		#对样本值求和
avg() 		#对样本值求平均值，这是进行指标数据分析的标准方法
count()		#对分组内的时间序列进行数量统计
min()		#求取样本值中的最小者
max()		#求取样本值中的最大者
topk()		#逆序返回分组内的样本值最大的前k个时间序列及其值
bottomk()	#顺序返回分组内的样本值最小的前k个时间序列及其值
quantile()	#分位数用于评估数据的分布状态，该函数会返回分组内指定的分位数的值，即数值落在小于等于指定的分位区间的比例
count_values()	#对分组内的时间序列的样本值进行数量统计
stddev() 	#对样本值求标准差，以帮助用户了解数据的波动大小(或称之为波动程度) 
stdvar() 	#对样本值求方差，它是求取标准差过程中的中间状态
```

```
聚合操作符(metric表达式) sum、min、max、avg、count等
聚合操作符(描述信息，metric) count_values、bottomk、topk等
```

可以借助于without和by功能获取数据集中的一部分进行分组统计

```
#without 表示显示信息的时候，排除此处指定的标签列表，对以外的标签进行分组统计，即：使用除此标签之外的其它标签进行分组统计
#by表示显示信息的时候，仅显示指定的标签的分组统计，即针对哪些标签分组统计
```

![](5day-png/34prometheus promql运算without.png)



```bash
#显示系统服务版本
#node_os_version
node_os_version{id="ubuntu", id_like="debian", instance="10.0.0.200:9100", job="node_exporter", name="Ubuntu"}
24.04
node_os_version{id="ubuntu", id_like="debian", instance="10.0.0.201:9100", job="node_exporter", name="Ubuntu"}
24.04
node_os_version{id="ubuntu", id_like="debian", instance="10.0.0.202:9100", job="node_exporter", name="Ubuntu"}

#统计Ubuntu系统主机数
#count(node_os_version{id="ubuntu"})

#分别统计不同OS的数量
#按node_os_version返回值value分组统计个数,将不同value加个新标签为os_version
#count_values("os_version",node_os_version)

#内存总量
#sum(node_memory_MemTotal_bytes)

#按instance分组统计内存总量
#sum(node_memory_MemTotal_bytes) by (instance)

#确认所有主机的CPU的总个数
#count(node_cpu_seconds_total{mode='system'})

#确认每个主机的CPU的总个数
#count(node_cpu_seconds_total{mode='system'}) by (instance)

#获取最大的值
#max(prometheus_http_requests_total)

#按handler,instance分组统计
#max(prometheus_http_requests_total) by (handler,instance)

#分组统计计数
count_values("counts",node_filesystem_size_bytes)

#获取前五个最大值
#topk(5, prometheus_http_requests_total)

#获取前5个最小值
#bottomk(5, prometheus_http_requests_total)

#对除了instance和job以外的标签分组求和
#sum(prometheus_http_requests_total) without (instance,job)

#仅对mode标签进行分组求和
#sum(node_cpu_seconds_total) by (mode)

#多个分组统计
#count(prometheus_http_requests_total) by (code,instance)

#查询handler为/metrics占所有handler的百分比
#sum(prometheus_http_requests_total{handler="/metrics",instance="10.0.0.200:9090"})/sum(prometheus_http_requests_total{instance="10.0.0.200:9090"}) * 100 

#查出200响应码占全部的百分比
#sum(prometheus_http_requests_total{code="200",handler="/-/ready",instance="10.0.0.200:9090"})/sum(prometheus_http_requests_total{handler="/-/ready",instance="10.0.0.200:9090"})


#下面语句有问题：
/ 前后两部分并不匹配：左侧的查询表达式 prometheus_http_requests_total{code="200", handler="/-/ready", instance="localhost:9090"} 是一个带有特定标签过滤条件的时间序列。右侧的表达式 sum (prometheus_http_requests_total{handler="/-/ready", instance="localhost:9090"}) by (handler,instance) 是一个汇总操作返回的时间序列矩阵（聚合后的多条时间序列）。
在 PromQL 中，/（除法）操作要求左右两侧的时间序列结构必须兼容，比如必须是相同的标签维度，且数值类型
必须相同。由于左侧是带有过滤条件的单一时间序列，而右侧是按标签聚合的多个时间序列矩阵，这样会导致除法运
算无法匹配。
#prometheus_http_requests_total{code="200", handler="/-/ready", instance="localhost:9090"} / sum (prometheus_http_requests_total{handler="/-/ready", instance="localhost:9090"}) by (handler,instance)
#nginx_http_requests_total{path="/api",method="GET",code="200"}/ sum nginx_http_requests_total{path="/api",method="GET"} by (path,method)

#查出200响应码占全部的百分比
#sum(nginx_http_requests_total{path="/api",method="GET",code="200"})/ sum(nginx_http_requests_total{path="/api",method="GET"})
```

#### **功能函数**

默认prometheus官方提供功能函数有40个,主要有以下几类

计算相关

```powershell
绝对值abs()、导数deriv()、指数exp()、对数ln()、二进制对数log2()、10进制对数log10()、平方根sqrt()
向上取整ceil()、向下取整floor()、四舍五入round()
样本差idelta()、差值delta()、递增值increase()、重置次数resets()
递增率irate()、变化率rate()、平滑值holt_winters()、直方百分位histogram_quantile()
预测值predict_linear()、参数vector()
范围最小值min_over_time()、范围最大值max_over_time()、范围平均值avg_over_time()、范围求和值sum_over_time()、范围计数值count_over_time()、范围分位数quantile_over_time()、范围标准差stddev_over_time()、范围标准方差stdvar_over_time()
```

取样相关

```powershell
获取样本absent()、升序sort()、降序sort_desc()、变化数changes()
即时数据转换为标量scalar()、判断大clamp_max()、判断小clamp_min()
范围采样值absent_over_time()，
```

时间相关

```powershell
day_of_month()、day_of_week()、days_in_month()、hour()、minute()、month()、time()、timestamp()、year()
```

标签相关

```powershell
标签合并label_join()、标签替换labelreplace()
```

实例

```bash
ceil()：向上取整
floor():向下取整
round():四舍五入
#示例 获取 15 分钟平均负载（load average）。
ceil(node_load15 * 10)
floor(node_load15 * 10)
round(node_load15 * 10)

increase(): 增长量,即last值-last前一个值
#示例:最近1分钟内CPU处于空闲状态时间
increase(node_cpu_seconds_total{cpu="0",mode="idle"}[1m])

#实例:CPU利用率
(1-sum(increase(node_cpu_seconds_total{mode="idle"}[1m])) by (instance) / sum(increase(node_cpu_seconds_total[1m])) by (instance)) *100

rate()
#平均变化率,计算在指定时间范围内计数器每秒增加量的平均值,即(last值-first值)/时间差的秒数，常用于counter的数据
#示例:过去一分钟每次磁盘读的变化率
rate(node_disk_read_bytes_total[1m])
#示例:一分钟内网卡传输的字节数(MB)
rate(node_network_transmit_bytes_total{device='eth0'}[1m]) /1024/1024

#判断在过去5分钟内HTTP请求状态码以"5"开头的请求的速率是否大于所有HTTP请求速率的10%。
rate(http_requests_total{status_code=~"5.*"}[5m]) > rate(http_requests_total[5m])*0.1

#每台主机CPU在5分钟内的平均使用率
(1-avg(irate(node_cpu_seconds_total{mode='idle'}[5m])) by (instance)) *100

irate()：查看瞬时变化率,即:(last值-last前一个采样的值)/时间戳差值
#高灵敏度函数，用于计算指标的瞬时速率，常用于counter的数据
#示例:过去一分钟每次磁盘读的瞬时变化率
irate(node_disk_read_bytes_total[1m])

#示例:查看CPU最近5m内最多的增长率
irate(node_cpu_seconds_total{instance='10.0.0.200:9100',mode='idle'}[5m])

#irate和rate都会用于计算某个指标在一定时间间隔内的变化速率。但是它们的计算方法有所不同：irate取的是在指定时间范围内的最近两个数据点来算速率，而rate会取指定时间范围内所有数据点，算出一组速率，然后取平均值作为结果。所以官网文档说：irate适合快速变化的计数器（counter），而rate适合缓慢变化的计数器（counter）。对于快速变化的计数器，如果使用rate，因为使用了平均值，很容易把峰值削平。除非我们把时间间隔设置得足够小，就能够减弱这种效应。
 
 
time(): 获取当前时间值
#示例:计算当前每个主机的运行时间
(time() - node_boot_time_seconds) / 3600

#示例:计算所有主机的总运行时间
sum(time() - node_boot_time_seconds) / 3600

histogram_quantile(): 百分取样值
#示例:计算过去10m内请求持续时间的第90个百分位数
histogram_quantile(0.9 , rate(prometheus_http_request_duration_seconds_bucket[10m]))

#absent 有值返回空,无值返回1,可用于告警判断
absent(node_memory_SwapTotal_byte)
absent(node_memory_SwapTotal_bytes)
```

****

**rate** **和** **irate** **函数**

**`rate()`：计算一段时间的平均速率**

```
rate(http_requests_total[5m])
```

**作用：**

- 计算 `http_requests_total` 在 **最近 5 分钟（[5m]）内的平均请求速率（每秒）**。
- `rate()` 通过 **线性回归（Linear Regression）** 平滑处理短期的波动，适用于 **长时间的趋势分析**。

****

**`irate()`：计算最近两个点的瞬时速率**

```
irate(http_requests_total[5m])
```

**作用：**

- 仅使用 **最近两个数据点** 计算 **瞬时速率**（每秒）。
- 不做平滑处理，能反映最新的突发流量变化，适用于 **短时间告警**。

**`rate()` vs `irate()` 对比**

|                    | `rate()`                             | `irate()`          |
| ------------------ | ------------------------------------ | ------------------ |
| **计算方式**       | 过去时间窗口的所有数据点，做线性回归 | 只看最近两个数据点 |
| **适用于**         | 长时间趋势分析                       | 短期突发检测       |
| **平滑处理**       | 是（减少波动）                       | 否（更敏感）       |
| **典型用途**       | Grafana 监控趋势                     | 短时告警           |
| **计算结果稳定性** | 更稳定                               | 可能波动较大       |



rate函数可以用来求指标的平均变化速率

rate函数=时间区间前后两个点的差 / 时间范围

一般rate函数可以用来求某个时间区间内的请求速率，也就是我们常说的QPS

<img src="5day-png/34promQL-rate.png" alt="image-20250304150010425" style="zoom:50%;" />

但是rate函数只是算出来了某个时间区间内的平均速率，没办法反映突发变化，假设在一分钟的时间区间里，前50秒的请求量都是0到10左右，但是最后10秒的请求量暴增到100以上，这时候算出来的值可能无法很好的反映这个峰值变化。这个问题可以通过irate函数解决

irate函数求出来的就是瞬时变化率

```
irate函数=时间区间内最后两个样本点的差 / 最后两个样本点的时间差
```

<img src="5day-png/34promQL-irate.png" alt="image-20250304150124220" style="zoom:50%;" />

一般情况下，irate函数的图像峰值变化大，rate函数变化较为平缓。





## **定制开发** **Exporter**

Prometheus 监控的指标可以通过下面方式提供

- Prometheus 内置
- instrumentation 程序仪表: 应用内置的指标功能,比如: Zookeeper,Gitlab,Grafana等
- 额外的exporter,使用第三方开发的功能
- Pushgateway 提供
- 通过自行编程实现的功能代码,需要开发能力

Prometheus对于监控功能的核心要素就是metric的监控项是否正常工作

Metric 本质就是对应的服务启动后自动生成的一个基于http协议URL地址，通过该地址可以获取想要的监控项。

实际生产环境中，想要监控好多指标，但是prometheus 可能并没有提供相应的metric监控项条目，比如: 某业务指标、转化率等，所以就需要实现自定义的metric条目。

开发应用服务的时候，就需要根据metric的数据格式，定制标准的/metric接口。

```
https://github.com/prometheus/client_golang
https://github.com/prometheus/client_python
https://github.com/prometheus/client_java
https://github.com/prometheus/client_rust
```

**定制** **Exporter** **案例: Python实现**

**准备** **Python** **开发** **Web** **环境**

```bash
#安装Python包管理器，默认没有安装
[root@ubuntu2404 ~]#apt update && apt install -y python3-pip
#虚拟环境安装
#############################如何不安装虚拟环境下面不需要执行#################################
#安装虚拟环境软件
~# pip3 install pbr virtualenv
~# pip3 install --no-deps stevedore virtualenvwrapper
#创建用户
~# useradd -m -s /bin/bash python
#准备目录
~# mkdir -p /data/venv
~# chown python.python /data/venv
#修改配置文件(可选)
~# su - python
~# vim .bashrc
force_color_prompt=yes #取消此行注释,清加颜色显示
#配置加速
~# mkdir ~/.pip
~# vim .pip/pip.conf 
[global] 
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install] 
trusted-host=pypi.douban.com
#配置虚拟软件
echo 'export WORKON_HOME=/data/venv' >> .bashrc
echo 'export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3' >> .bashrc
echo 'export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv' >> .bashrc
echo 'source /usr/local/bin/virtualenvwrapper.sh' >> .bashrc
source .bashrc
#注意：virtualenv 和 virtualenvwrapper.sh 的路径位置
#创建新的虚拟环境并自动进入
~# mkvirtualenv -p python3 flask_env
#进入已创建的虚拟环境
~# su - python
~# workon flask_env
#虚拟环境中安装相关模块库
~# pip install flask prometheus_client
~# pip list
```



```bash
#安装Python包管理器，默认没有安装
[root@ubuntu2404 ~]#apt update && apt install -y python3-pip
[root@ubuntu2404 ~]#apt install -y python3-flask python3-prometheus-client
```

```python
[root@ubuntu2404 ~]#cat flask_metric.py 
from prometheus_client import start_http_server,Counter, Summary
from flask import Flask, jsonify
from wsgiref.simple_server import make_server
import time

app = Flask(__name__)

# Create a metric to track time spent and requests made
REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')
COUNTER_TIME  = Counter("request_count", "Total request count of the host")

@app.route("/metrics")
@REQUEST_TIME.time()
def requests_count():
    COUNTER_TIME.inc()
    return jsonify({"return": "success OK!"})

if __name__ == "__main__":
    start_http_server(8000)
    httpd = make_server( '0.0.0.0', 8001, app )
    httpd.serve_forever()
```

```bash
#运行flask_metric.py 
[root@ubuntu2404 ~]#python3 flask_metric.py 
```

```bash
#在prometheus server中添加监控
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
....
- job_name: "my_metric"
    static_configs:
      - targets: 
        - "10.0.0.200:8000"
        labels: {app: "flask_web"}
```

```bash
[root@ubuntu2404 ~]#while :; do curl 127.0.0.1:8001/metrics;sleep $[RANDOM%3];done
```

**定制** **Exporter** **案例: Golang实现**

基于Golang实现

```bash
#先安装go环境
[root@ubuntu2404 ~]#apt update && apt -y install golang

#准备代码，利用SDK实现
[root@ubuntu2404 code]#vim main.go 

[root@ubuntu2404 code]#export CGO_ENABLED=0
[root@ubuntu2404 code]#export GOARCH=amd64
[root@ubuntu2404 code]#export GOOS=linux
[root@ubuntu2404 code]#go mod init my_metric
go: creating new go.mod: module my_metric
go: to add module requirements and sums:
        go mod tidy
[root@ubuntu2404 code]#ls
go.mod  main.go
[root@ubuntu2404 code]#cat go.mod 
module my_metric

go 1.22.2
[root@ubuntu2404 code]#go env -w GOPROXY=https://goproxy.cn,direct
[root@ubuntu2404 code]#go get github.com/prometheus/client_golang/prometheus/promhttp
go: downloading github.com/prometheus/client_golang v1.21.0
go: downloading github.com/klauspost/compress v1.17.11
go: downloading github.com/prometheus/client_model v0.6.1
go: downloading github.com/prometheus/common v0.62.0
go: downloading github.com/beorn7/perks v1.0.1
go: downloading github.com/cespare/xxhash/v2 v2.3.0
go: downloading github.com/prometheus/procfs v0.15.1
go: downloading golang.org/x/sys v0.28.0
go: downloading google.golang.org/protobuf v1.36.1
go: downloading github.com/munnerz/goautoneg v0.0.0-20191010083416-a7dc8b61c822
go: added github.com/beorn7/perks v1.0.1
go: added github.com/cespare/xxhash/v2 v2.3.0
go: added github.com/klauspost/compress v1.17.11
go: added github.com/munnerz/goautoneg v0.0.0-20191010083416-a7dc8b61c822
go: added github.com/prometheus/client_golang v1.21.0
go: added github.com/prometheus/client_model v0.6.1
go: added github.com/prometheus/common v0.62.0
go: added github.com/prometheus/procfs v0.15.1
go: added golang.org/x/sys v0.28.0
go: added google.golang.org/protobuf v1.36.1
[root@ubuntu2404 code]#go build -o exporter_demo
[root@ubuntu2404 code]#./exporter_demo
```



## **Prometheus** **标签管理**

标签功能: 用于对数据分组和分类,利用标签可以将数据进行过滤筛选

标签管理的常见场景:

- 删除不必要的指标
- 从指标中删除敏感或不需要的标签
- 添加、编辑或修改指标的标签值或标签格式

标签分类：

- 默认标签: Prometheus 自身内置

  形式: __keyname__

- 应用标签: 应用本身内置

  形式: keyname

- 自定义标签: 用户定义

  形式: keyname

范例: 添加主机节点查看默认标签

```bash
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
....
- job_name: "my_metric"
    static_configs:
      - targets: 
        - "10.0.0.200:8000"
        labels: {app: "flask_web"}	#定义标签

#重启服务  
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

范例: 添加主机标签

```bash
#编辑prometheus.yml配置文件
[root@prometheus ~]#vim /usr/local/prometheus/conf/prometheus.yml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['10.0.0.104:9100', '10.0.0.105:9100', '10.0.0.106:9100']
        labels:
          node: "worker node"
          type: "test"

  - job_name: 'zookeeper'
    static_configs:
      - targets: ['10.0.0.105:7000', '10.0.0.106:7000']
        labels:
          app: "zookeeper"
          type: "dev"

#配置解析：labels: 的编写方法，在此处遵循 yaml的字典格式

#重启服务        
[root@prometheus ~]#systemctl reload prometheus.service
#浏览器访问确认
```

**指标的生命周期**

- 私有标签

  ```
  私有标签以"__*"样式存在，用于获取监控目标的默认元数据属性，比如__address__用于获取目标的地址，__scheme__用户获取目标的请求协议方法，__metrics_path__获取请求的url地址等
  ```

- 普通标签

  ```
  对个监控主机节点上的监控指标进行各种灵活的管理操作，常见的操作有，删除不必要|敏感指标，添加、编辑或者修改指标的标签值或者标签格式。
  ```



![image-20250304160103349](5day-png/34Prometheus对数据的处理流程.png)



- 在每个scrape_interval期间，Prometheus都会检查执行的作业 Job

- 这些作业首先会根据Job上指定的发现配置生成target列表，此即服务发现过程

  服务发现会返回一个Target列表，其中包含一组称为元数据的标签，这些标签都以 "`__meta__" 为前缀__`

  服务发现还会根据目标配置来设置其它标签，这些标签带有 "__" 前缀和后缀，包括如下：

  ```
  "__scheme__"  #协议http或https，默认为http
  "__address__"  #target的地址
  "__metrics_path__" #target指标的URI路径（默认为/metrics）
  #若URI路径中存在任何参数，则它们的前缀会设置为 "__param__"
  ```

  这些目标列表和标签会返回给Prometheus，其中的一些标签也可以配置为被覆盖或替换为其它标签

- 配置标签会在抓取的生命周期中被利用以生成其他标签

  例如：指标上的instance标签的默认值就来自于 `__address__ `标签的值

- 对于发现的各个目标，Prometheus提供了可以重新标记(relabel_config) 目标的机会

  它定义在 Job 配置段的relabel_config配置中，常用于实现如下功能

  - 将来自服务发现的元数据标签中的信息附加到指标的标签上

  - 过滤目标

- 数据抓取、以及指标返回的过程

- 抓取而来的指标在保存之前，还允许用户对指标重新打标并过滤的方式

  它定义在job配置段的metric_relabel_configs配置中，常用于实现如下功能

  - 删除不必要的指标

  - 从指标中删除敏感或不需要的标签

  - 添加、编辑或修改指标的标签值或标签格式

**relabel_configs** **和** **metric_relabel_configs**

relabel_config、metric_relabel_configs这两个配置虽然在作用上类似，但是还是有本质上的区别的，这些区别体现在两个方面：执行顺序和数据处理上。



`relabel_configs` 和 `metric_relabel_configs` 是 Prometheus 配置中用于重新标记（relabeling）的两个关键配置项。它们主要用于在抓取指标之前或之后对标签进行修改、过滤或删除。以下是它们的详细说明：

------

### **1. relabel_configs**

`relabel_configs` 用于在抓取指标之前对目标的标签进行重新标记。它通常用于以下场景：

- 动态生成或修改目标的标签。
- 过滤或选择特定的目标。
- 从标签中提取信息并生成新的标签。

#### **常见用途**

- **服务发现**：在 Kubernetes 或其他服务发现机制中，动态生成目标的标签。
- **标签过滤**：根据标签值选择或排除特定的目标。
- **标签修改**：修改或添加标签，例如将 `__meta_kubernetes_pod_name` 转换为 `pod`。

#### **配置示例**

```yaml
scrape_configs:
  - job_name: 'my_job'
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: 'pod'
      - source_labels: [__meta_kubernetes_namespace]
        target_label: 'namespace'
      - action: keep
        source_labels: [__meta_kubernetes_service_label_app]
        regex: 'my_app'
```

#### **常用动作（action）**

- `replace`：默认动作，用 `source_labels` 的值替换 `target_label`。
- `keep`：保留匹配 `regex` 的目标。
- `drop`：丢弃匹配 `regex` 的目标。
- `labelmap`：将 `source_labels` 的键映射到新的标签键。

------

### **2. metric_relabel_configs**

`metric_relabel_configs` 用于在抓取指标之后对指标的标签进行重新标记。它通常用于以下场景：

- 过滤或删除不需要的指标。
- 修改或添加指标的标签。
- 清理或标准化指标标签。

#### **常见用途**

- **指标过滤**：根据指标名称或标签值过滤掉不需要的指标。
- **标签清理**：删除敏感或不必要的标签。
- **标签标准化**：统一标签的命名或格式。

#### **配置示例**

```yaml
scrape_configs:
  - job_name: 'my_job'
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: 'http_requests_total'
        action: keep
      - source_labels: [instance]
        regex: '10.0.0.1:8080'
        action: drop
      - source_labels: [env]
        target_label: 'environment'
```

#### **常用动作（action）**

- `replace`：用 `source_labels` 的值替换 `target_label`。
- `keep`：保留匹配 `regex` 的指标。
- `drop`：丢弃匹配 `regex` 的指标。
- `labeldrop`：删除匹配 `regex` 的标签。
- `labelkeep`：保留匹配 `regex` 的标签。

------

### **主要区别**

| 特性         | `relabel_configs`                     | `metric_relabel_configs`               |
| :----------- | :------------------------------------ | :------------------------------------- |
| **作用阶段** | 在抓取目标之前                        | 在抓取指标之后                         |
| **主要用途** | 动态生成目标标签、过滤目标            | 过滤指标、修改或删除指标标签           |
| **操作对象** | 目标的元数据（如服务发现生成的标签）  | 抓取到的指标及其标签                   |
| **常见动作** | `replace`, `keep`, `drop`, `labelmap` | `replace`, `keep`, `drop`, `labeldrop` |

------

### **总结**

- **`relabel_configs`**：用于在抓取目标之前对目标的标签进行处理，通常与服务发现结合使用。
- **`metric_relabel_configs`**：用于在抓取指标之后对指标的标签进行处理，通常用于指标过滤和标签清理。



常见配置如下

```yaml
scrape_configs:
  - job_name: 'prometheus'

    # 目标重写（Relabel）
    relabel_configs:
      - source_labels: ['__address__']
        separator: ';'
        regex: '(.*)'
        replacement: '$1'
        target_label: 'instance'
        action: 'replace'

    # 指标重写（Metric Relabel）
    metric_relabel_configs:
      - source_labels: ['http_requests_total']
        separator: ';'
        regex: '(.*)'
        replacement: '$1'
        target_label: 'http_requests'
        action: 'replace'
```



```bash
#配置示例如下：
[root@ubuntu2404 ~]#cat a.yaml
scrape_configs:
  - job_name: 'prometheus'
    relabel_configs|metric_relabel_configs:
    - source_labels: [<labelname> [, ...]]
      separater: '<string> | default = ;'
      regex: '<regex> | default = (.*)'
      replacement: '<string> | default = $1'
      target_label: '<labelname>'
      action: '<relabel_action> | default = replace'

#属性解析：
action 			#对标签或指标进行管理，常见的动作有replace|keep|drop|labelmap|labeldrop等,默认为replace
source_labels 	#指定正则表达式匹配成功的Label进行标签管理,此为列表
target_label 	#在进行标签替换的时候，可以将原来的source_labels替换为指定修改后的目标label
separator       #指定用于联接多个source_labels为一个字符串的分隔符,默认为分号
regex 			#表示source_labels对应Label的名称或者值(和action有关)进行匹配此处指定的正则表达式
replacement 	#替换标签时,将 target_label对应的值进行修改成此处的值

#action说明：
#1）替换标签的值: 
replace   	#此为默认值，首先将source labels中指定的各标签的值使用separator指定的字符进行串连,如果串连后的值和regex匹配，则使用replacement指定正则表达式模式或值对target_label字段进行赋值，如果target_label不存在，也可以用于创建新的标签名为target_label
hashmod  	#将target_label的值设置为一个hash值，该hash是由modules字段指定的hash模块算法对source_labels上各标签的串连值进行hash计算生成

#2）保留或删除指标: 该处的每个指标名称对应一个target或metric
keep   		#如果获取指标的source_labels的各标签的值串连后的值与regex匹配时,则保留该指标,反之则删除该指标
drop   		#如果获取指标的source_labels的各标签的值串连后的值与regex匹配时,则删除该指标,反之则保留该指标，即与keep相反

#3）创建或删除标签
labeldrop  	#如果source labels中指定的标签名称和regex相匹配,则删除此标签,相当于标签黑名单
labelkeep  	#如果source labels中指定的标签名称和regex相匹配,则保留,不匹配则删除此标签,相当于标签
白名单
labelmap   	#一般用于生成新标签，将regex对source labels中指定的标签名称进行匹配，而后将匹配到的标签的值赋值给replacement字段指定的标签；通常用于取出匹配的标签名的一部分生成新标签,旧的标签仍会存在

#4）大小写转换
lowercase 	#将串联的 source_labels 映射为其对应的小写字母
uppercase 	#将串联的 source_labels 映射到其对应的大写字母
```

范例:

```yaml
#示例:删除指标名node_network_receive开头的标签
  - job_name: "my_metric"
    static_configs:
      - targets: 
        - "10.0.0.200:8000"
        labels: {app: "flask_web"}
    metric_relabel_configs:
    - source_labels: [__name__]
      regex: 'node_network_receive.*'
      action: drop
#示例:替换 将__name__的值匹配正则表达式'/.*' 替换为123456 标签名为replace_id
    - source_labels: [__name__]
      regex: '/.*'
      replacement: '123456'
      target_label: replace_id
```

```yaml
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "10.0.0.201:9100"
          - "10.0.0.200:9100"
          - "10.0.0.202:9100"
    relabel_configs:
      - source_labels: 
        - "__scheme__"
        - "__address__"
        - "__metrics_path__"
        regex: "(http|https)(.*)"
        separator: ""                 # 将 source_labels 拼接在一起，不加分隔符
        target_label: "endpoint"      # 生成新的 endpoint 标签
        replacement: "${1}://${2}"    # 替换为 http(s)://<地址><路径>
        action: replace
      - source_labels: [endpoint]     # 将endpoint改名为myendpoint
        target_label: myendpoint
```

```yaml
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
scrape_configs:
  - job_name: "my_metric"
    static_configs:
      - targets: 
        - "10.0.0.200:8000"
        labels: {app: "flask_web"}
    relabel_configs:                   #所有名称为job或app的标签修改其标签名称加后缀_name,但旧的标签还存在
      - regex: "(job|app)"
        replacement: "${1}_name"
        action: labelmap
      - regex: "(job|app)"             #删除名称为job或app的标签
        action: labeldrop
```

范例: 用于在相应的job上，删除发现的各target之上面以"go"为前名称前缀的指标，以前的数据还在，以后的数据不在采集

```yaml
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "10.0.0.201:9100"
          - "10.0.0.200:9100"
          - "10.0.0.202:9100"
    relabel_configs:
      - source_labels:
        - "__scheme__"
        - "__address__"
        - "__metrics_path__"
        regex: "(http|https)(.*)"
        separator: ""                 # 将 source_labels 拼接在一起，不加分隔符
        target_label: "endpoint"      # 生成新的 endpoint 标签
        replacement: "${1}://${2}"    # 替换为 http(s)://<地址><路径>
        action: replace
      - source_labels: [endpoint]
        target_label: myendpoint
    metric_relabel_configs:
      - source_labels: 
      	- "__name__"
        regex: "go.*"
        action: drop
```



## 监控告警

 可以在prometheus.yaml配置文件中通过rule_fies属性进行导入即可，格式如下

```yaml
rule_files:
  - "first_rules.yml"
  - "second_rules.yml"
  - "../rules/*.yml"
  
#注意: 如果用相对路径是指相对于prometheus.yml配置文件的路径
```

### **记录规则说明**

记录规则常用的场景：

- 将预先计算经常需要或计算量大的复杂的PromQL语句指定为一个独立的metric监控项，这样在查询的时候就非常方便，而且查询预计算结果通常比每次需要原始表达式都要快得多，尤其是对于仪表板特别有用，仪表板每次刷新时都需要重复查询相同的表达式。
- 此外在告警规则中也可以引用记录规则
- 记录规则效果与shell中的别名相似

```
#规则文件的语法为：
groups:
 [ - <rule_group> ]
  
#简单的规则文件示   
groups:
  - name: example
    interval: 10s
    limit: <init> | default=0
    rules:
      - record: job:http_inprogress_requests:sum
        expr: sum(http_inprogress_requests) by (job)
        labels:
          [ <labelname>: <labelvalue> ]
```

属性解析

```bash
name 		#规则组名，必须是唯一的
interval 	#定制规则执行的间隔时间,默认值为prometheus.yml配置文件中的global.evaluation_interval
limit:      #限制条件，对于记录规则用于限制其最多可生成序列数量，对于告警规则用于限制最多可生成的告警数量
rules 		#设定规则具体信息
record 		#定制指标的名称
expr 		#执行成功的PromQL
labels 		#为该规则设定标签
```

案例：

```bash
#在Prometheus查询部分指标时需要通过将现有的规则组合成一个复杂的表达式，才能查询到对应的指标结果，比如在查询"自定义的指标请求处理时间"，参考如下
request_processing_seconds_sum{instance="10.0.0.200:8000",job="my_metric"} / request_processing_seconds_count{instance="10.0.0.200:8000",job="my_metric"}
#以上的查询语句写起来非常长，在我们graph绘图的时候，每次输入命令都是非常繁琐,很容易出现问题。
```

```bash
#创建规则记录文件
[root@ubuntu2404 ~]# mkdir /usr/local/prometheus/rules
[root@ubuntu2404 ~]# vim /usr/local/prometheus/rules/prometheus_record_rules.yml
groups:
  - name: myrules
    rules:
    - record: "request_process_per_time"
      expr: request_processing_seconds_sum{job="my_metric"} / request_processing_seconds_count{job="my_metric"}
      labels:
        app: "flask"
        role: "web"
    - record: "request_count_per_minute"
      expr: increase(request_count_total{job="my_metric"}[1m])
      labels:
        app: "flask"
        role: "web"
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
...
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "../rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"
...

[root@ubuntu2404 ~]#promtool check config /usr/local/prometheus/conf/prometheus.yml
Checking /usr/local/prometheus/conf/prometheus.yml
  SUCCESS: 1 rule files found
 SUCCESS: /usr/local/prometheus/conf/prometheus.yml is valid prometheus config file syntax

Checking /usr/local/prometheus/rules/prometheus_record_rules.yml
  SUCCESS: 2 rules found
  
[root@ubuntu2404 ~]#systemctl reload prometheus.service
```



### **告警说明和** **Alertmanager** **部署**

告警能力在Prometheus的架构中被划分成两个独立的部分。

- 通过在Prometheus中定义AlertRule(告警规则)，Prometheus会周期性的对告警规则进行计算，如果满足告警触发条件就会向Alertmanager发送告警信息。
- 然后，Alertmanager管理这些告警，包括进行重复数据删除，分组和路由，以及告警的静默和抑制

![image-20250304182140921](5day-png/34alertmamager告警.png)

**告警特性**

![image-20250304182222207](5day-png/34告警特性.png)



- 去重

  将多个相同的告警,去掉重复的告警,只保留不同的告警

- 分组 Grouping

  分组机制可以将相似的告警信息合并成一个通知。

  在某些情况下，比如由于系统宕机导致大量的告警被同时触发，在这种情况下分组机制可以将这些被触发的告警合并为一个告警通知，避免一次性接受大量的告警通知，而无法对问题进行快速定位。

  告警分组，告警时间，以及告警的接受方式可以通过Alertmanager的配置文件进行配置。

- 抑制 Inhibition

  系统中某个组件或服务故障，那些依赖于该组件或服务的其它组件或服务可能也会因此而触发告警，抑制便是避免类似的级联告警的一种特性，从而让用户能将精力集中于真正的故障所在

  抑制可以避免当某种问题告警产生之后用户接收到大量由此问题导致的一系列的其它告警通知

  抑制的关键作用在于，同时存在的两组告警条件中，其中一组告警如果生效，能使得另一组告警失效

  同样通过Alertmanager的配置文件进行设置

- 静默 Silent

  静默提供了一个简单的机制可以快速根据标签在一定的时间对告警进行静默处理。

  如果接收到的告警符合静默的配置，Alertmanager则不会发送告警通知。

  比如：通常在系统例行维护期间，需要激活告警系统的静默特性

  静默设置可以在Alertmanager的Web页面上进行设置。

- 路由 Route

  将不同的告警定制策略路由发送至不同的目标，比如：不同的接收人或接收媒介

### **Alertmanager** **部署**

```bash
[root@ubuntu2404 ~]#ls
alertmanager-0.28.0.linux-amd64.tar.gz
[root@ubuntu2404 ~]#tar xf alertmanager-0.28.0.linux-amd64.tar.gz -C /usr/local/
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ln -s alertmanager-0.28.0.linux-amd64/ alertmanager
[root@ubuntu2404 local]#cd alertmanager
[root@ubuntu2404 alertmanager]#ls
alertmanager  alertmanager.yml  amtool  LICENSE  NOTICE
[root@ubuntu2404 alertmanager]#mkdir bin
[root@ubuntu2404 alertmanager]#mv alertmanager amtool bin/
[root@ubuntu2404 alertmanager]#ls
alertmanager.yml  bin  LICENSE  NOTICE
[root@ubuntu2404 alertmanager]#mkdir conf
[root@ubuntu2404 alertmanager]#mv alertmanager.yml conf/
[root@ubuntu2404 alertmanager]#chown -R prometheus:prometheus /usr/local/alertmanager/
[root@ubuntu2404 alertmanager]#vim /lib/systemd/system/alertmanager.service
[root@ubuntu2404 alertmanager]#cat /lib/systemd/system/alertmanager.service
[Unit]
Description=alertmanager project
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/alertmanager/bin/alertmanager --config.file=/usr/local/alertmanager/conf/alertmanager.yml --storage.path=/usr/local/alertmanager/data --web.listen-address=0.0.0.0:9093
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target

#属性解析：最好配置 --web.listen-address ，因为默认的配置是:9093,有可能在启动时候报错
[root@ubuntu2404 alertmanager]#systemctl daemon-reload 
[root@ubuntu2404 alertmanager]#systemctl enable --now alertmanager.service 
Created symlink /etc/systemd/system/multi-user.target.wants/alertmanager.service → /usr/lib/systemd/system/alertmanager.service.
[root@ubuntu2404 alertmanager]#systemctl status alertmanager.service 
```

**Ansible** **方式部署**

```
https://github.com/prometheus-community/ansible/tree/main/roles/alertmanager
```

**容器方式部署**

```dockerfile
version: '3.6'

networks:
  monitoring:
    driver: bridge
    ipam:
      config:
        - subnet: 172.31.0.0/24

services:
  alertmanager:
    image: prom/alertmanager:v0.24.0
    volumes:
      - ./alertmanager/:/etc/alertmanager/
      # - alertmanger_data:/alertmanager
    networks:
      - monitoring
    restart: always
    ports:
      - 9093:9093
    command:
      - '--config.file=/etc/alertmanager/config.yml'
      - '--storage.path=/alertmanager'
      - '--log.level=debug'
```

**脚本部署**

```powershell
[root@ubuntu2404 ~]#cat install_alertmanager.sh 
#!/bin/bash
ALERTMANAGER_VERSION=0.28.0
#ALERTMANAGER_VERSION=0.27.0
#ALERTMANAGER_VERSION=0.25.0
#ALERTMANAGER_VERSION=0.24.0
#ALERTMANAGER_VERSION=0.23.0
ALERTMANAGER_FILE="alertmanager-${ALERTMANAGER_VERSION}.linux-amd64.tar.gz"

ALERTMANAGE_URL="https://github.com/prometheus/alertmanager/releases/download/v${ALERTMANAGER_VERSION}/${ALERTMANAGER_FILE}"

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


install_alertmanager () {
    if [ ! -f  ${ALERTMANAGER_FILE} ] ;then
        wget ${GITHUB_PROXY}${ALERTMANAGE_URL}  || { color "下载失败!" 1 ; exit ; }
    fi
    [ -d $INSTALL_DIR ] || mkdir -p $INSTALL_DIR
    tar xf ${ALERTMANAGER_FILE} -C $INSTALL_DIR
    cd $INSTALL_DIR &&  ln -s alertmanager-${ALERTMANAGER_VERSION}.linux-amd64 alertmanager
    mkdir -p $INSTALL_DIR/alertmanager/{bin,conf,data}
    cd $INSTALL_DIR/alertmanager && { mv alertmanager.yml conf/;  mv alertmanager amtool bin/; }
    id prometheus &> /dev/null ||useradd -r -g prometheus -s /sbin/nologin prometheus
    chown -R prometheus.prometheus $INSTALL_DIR/alertmanager/
      
    cat >>  /etc/profile <<EOF
export ALERTMANAGER_HOME=${INSTALL_DIR}/alertmanager
export PATH=\${ALERTMANAGER_HOME}/bin:\$PATH
EOF

}


alertmanager_service () {
    cat > /lib/systemd/system/alertmanager.service <<EOF
[Unit]
Description=Prometheus alertmanager
After=network.target

[Service]
Type=simple
ExecStart=$INSTALL_DIR/alertmanager/bin/alertmanager --config.file=${INSTALL_DIR}/alertmanager/conf/alertmanager.yml --storage.path=${INSTALL_DIR}/alertmanager/data --web.listen-address=0.0.0.0:9093
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target
EOF
    systemctl daemon-reload
    systemctl enable --now alertmanager.service
}


start_alertmanager() { 
    systemctl is-active alertmanager.service
    if [ $?  -eq 0 ];then
        echo
        color "alertmanager 安装完成!" 0
        echo "-------------------------------------------------------------------"
        echo -e "访问链接: \c"
        msg_info "http://$HOST:9093"
    else
        color "alertmanager 安装失败!" 1
        exit
    fi

}

install_alertmanager

alertmanager_service

start_alertmanager
```



### **Prometheus** **集成**

配置修改

```bash
#修改配置文件，加载alertmanager配置属性
#方式1：静态配置
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 10.0.0.200:9093
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "../rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"
...
scrape_configs:
  - job_name: "alertmanager"
    static_configs:
      - targets: 
        - "10.0.0.200:9093"
       
[root@ubuntu2404 ~]#systemctl reload prometheus.service
```

### alertmanager配置文件

```
#配置文件总共定义了五个模块，global、templates、route，receivers，inhibit_rules
```

```yaml
global:
  resolve_timeout: 1m
  smtp_smarthost: 'localhost:25'
  smtp_from: 'ops@example.com'
  smtp_require_tls: false

templates:
  - '/etc/alertmanager/template/*.tmpl'

route:
  receiver: 'admin'  # 指向 receivers 里定义的接收器
  group_by: ['alertname']
  group_wait: 20s
  group_interval: 10m
  repeat_interval: 3h

receivers:
  - name: 'admin'
    email_configs:
      - to: 'admin@example.com'
```

```bash
#说明
global 
#用于定义Alertmanager的全局配置。
#相关配置参数
resolve_timeout 	#定义持续多长时间未接收到告警标记后，就将告警状态标记为resolved
smtp_smarthost  	#指定SMTP服务器地址和端口
smtp_from 			#定义了邮件发件的的地址
smtp_require_tls 	#配置禁用TLS的传输方式

templates
#用于指定告警通知的信息模板，如邮件模板等。由于Alertmanager的信息可以发送到多种接收介质，如邮件、微信等，通常需要能够自定义警报所包含的信息，这个就可以通过模板来实现。

route
#用于定义Alertmanager接收警报的处理方式，根据规则进行匹配并采取相应的操作。路由是一个基于标签匹配规则的树状结构，所有的告警信息都会从配置中的顶级路由(route)进入路由树。从顶级路由开始，根据标签匹配规则进入到不同的子路由，并且根据子路由设置的接收者发送告警。在示例配置中只定义了顶级路由，并且配置的接收者为wang，因此，所有的告警都会发送给到admin的接收者。

#相关参数
group_by        	#用于定义分组规则，使用告警名称做为规则，满足规则的告警将会被合并到一个通知中
group_wait      	#当 Alertmanager 收到一个告警组时，它会在 group_wait 时间内等待是否还有其他属于同一组的告警。如果在这个时间内没有收到其他属于同一组的告警，Alertmanager 将认为该组的告警已经完整，并开始进行通知操作。这样做可以在一定程度上避免频繁发送不完整的告警通知，而是等待一段时间后再一起发送
group_interval 		#用于控制在一段时间内收集这些相同告警规则的实例，并将它们组合成一个告警组。在这个时间间隔内，如果相同的告警规则再次触发，它们将被添加到同一个告警组中。这样做可以避免过于频繁地发送重复的告警通知，从而避免对接收者造成困扰。配置分组等待的时间间隔，在这个时间内收到的告警，会根据前面的规则做合并
repeat_interval 	#参数定义了重复发送告警通知的时间间隔。如果告警状态在 repeat_interval 时间内持续存在（即告警没有被解决），Alertmanager 会定期重复发送相同的告警通知。这样做可以确保接收者持续得到告警的提醒，直到告警状态得到解决为止。

receivers
#用于定义相关接收者的地址信息
#告警的方式支持如下
email_configs  	#配置相关的邮件地址信息
wechat_configs  #指定微信配置
webhook_configs #指定webhook配置,比如:dingtalk
```

alertmanager配置文件语法检查命令

```bash
amtool check-config /usr/local/alertmanager/conf/alertmanager.yml
```

### **Alertmanager** **启用邮件告警**

```yaml
[root@ubuntu2404 ~]#cat /usr/local/alertmanager/conf/alertmanager.yml 
global:
  resolve_timeout: 5m                        
  smtp_smarthost: 'smtp.163.com:25'           #基于全局块指定发件人信息,此处设为25，如果465还需要添加tls相关配置
  smtp_from: 'hzokang@163.com'
  smtp_auth_username: 'hzokang@163.com'
  smtp_auth_password: 'EBgCxViwqvnxxbEu'
  smtp_hello: '163.com'
  smtp_require_tls: false        #启用tls安全,默认true,此处设为false

# 路由配置
route:
  group_by: ['instance', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'email'

# 收信人员
receivers:
- name: 'email'
  email_configs:
  - to: 'hzokang@outlook.com'
    send_resolved: true           #问题解决后也会发送恢复通知  
```

```bash
#语法检查
[root@ubuntu2404 ~]#/usr/local/alertmanager/bin/amtool check-config /usr/local/alertmanager/conf/alertmanager.yml
Checking '/usr/local/alertmanager/conf/alertmanager.yml'  SUCCESS
Found:
 - global config
 - route
 - 0 inhibit rules
 - 1 receivers
 - 0 templates
[root@ubuntu2404 ~]#systemctl reload alertmanager.service 
```

### **告警规则**

```
https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
```

警报规则可以实现基于Prometheus表达式语言定义警报条件，并将有关触发警报的通知发送到外部服务。只要警报表达式在给定的时间点生成一个或多个动作元素，警报就被视为这些元素的标签集处于活动状态。

告警规则中使用的查询语句较为复杂时，可将其保存为记录规则，而后通过查询该记录规则生成的时间序列来参与比较，从而避免实时查询导致的较长时间延迟

警报规则在Prometheus中的基本配置方式与记录规则基本一致。

在Prometheus中一条告警规则主要由以下几部分组成：

- 告警名称：用户需要为告警规则命名，当然对于命名而言，需要能够直接表达出该告警的主要内容
- 告警规则：告警规则实际上主要由PromQL进行定义，其实际意义是当表达式（PromQL）查询结果持续多长时间（During）后出发告警

在Prometheus中，还可以通过Group（告警组）对一组相关的告警进行统一定义。这些定义都是通过YAML文件来统一管理的

告警规则文件示例

```bash
groups:
  - name: example
    rules:
      - alert: HighRequestLatency
        expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
        for: 10m
        labels:
          severity: warning
          project: myproject
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 10 minutes."


#属性解析：
alert       #定制告警的动作名称
expr        #是一个布尔型的条件表达式，一般表示满足此条件时即为需要告警的异常状态
for         #条件表达式被触发后，一直持续满足该条件长达此处时长后才会告警，即发现满足expr表达式后，在告警前的等待时长，默认为0，此时间前为pending状态，之后为firing，此值应该大于抓取间隔时长，避免偶然性的故障
labels      #指定告警规则的标签，若已添加，则每次告警都会覆盖前一次的标签值
labels.severity #自定义的告警级别的标签
annotations     #自定义注释信息，注释信息中的变量需要从模板中或者系统中读取，最终体现在告警通知的信息中
```

**Prometheus** **告警规则**

```
https://samber.github.io/awesome-prometheus-alerts/
```

```yaml
[root@ubuntu2404 ~]#vim /usr/local/prometheus/rules/prometheus_alert_rules.yml 
[root@ubuntu2404 ~]#cat /usr/local/prometheus/rules/prometheus_alert_rules.yml
groups:
  - name: flask_web
    rules:
      - alert: InstanceDown
        expr: up{job="my_metric"} == 0
        # expr: up == 0  # 监控所有 targets
        for: 10s
        labels:
          severity: "1"
        annotations:
          summary: "Instance {{ $labels.instance }} 停止工作"
          description: "{{ $labels.instance }} job {{ $labels.job }} 已经停止 10s 以上"
```

```bash
[root@ubuntu2404 ~]#promtool check config /usr/local/prometheus/conf/prometheus.yml 
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

### **静默** **silence**

![image-20250304195429408](5day-png/34静默silence.png)

![image-20250304195527180](5day-png/34静默silence1.png)

**取消静默silence**

![image-20250304195623348](5day-png/34静默silence2.png)

### **告警模板**

```
https://prometheus.io/docs/alerting/latest/notifications/
https://www.w3school.com.cn/html/html_tables.asp
```

使用流程：

- 分析关键信息
- 定制模板内容
- alertmanager 加载模板文件
- 告警信息使用模板内容属性



```bash
[root@ubuntu2404 alertmanager]#mkdir tmpl
[root@ubuntu2404 alertmanager]#ls
bin  conf  data  LICENSE  NOTICE  tmpl
[root@ubuntu2404 alertmanager]#vim tmpl/email.tml

[root@ubuntu2404 alertmanager]#cat conf/alertmanager.yml
global:
  resolve_timeout: 5m                        
  smtp_smarthost: 'smtp.163.com:25'           #基于全局块指定发件人信息,此处设为25，如果465还需要添加tls相关配置
  smtp_from: 'hzokang@163.com'
  smtp_auth_username: 'hzokang@163.com'
  smtp_auth_password: 'EBgCxViwqvnxxbEu'
  smtp_hello: '163.com'
  smtp_require_tls: false        #启用tls安全,默认true,此处设为false

# 路由配置
route:
  group_by: ['instance', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'email'

# 收信人员
receivers:
- name: 'email'
  email_configs:
  - to: 'hzokang@outlook.com'
    send_resolved: true           #问题解决后也会发送恢复通知  
    headers: { Subject: "[WARN] 报警邮件"}   #添加此行,定制邮件标题
    html: '{{ template "email.html" . }}'    #添加此行,调用模板显示邮件正文

#模板配置文件
templates:            #加下面两行加载模板文件
  - '../tmpl/*.tmpl'  #相对路径是相对于altermanager.yml文件的路径
[root@ubuntu2404 alertmanager]#systemctl reload alertmanager.service 
```

**模板一**

```bash
[root@ubuntu2404 alertmanager]#vim /usr/local/alertmanager/tmpl/email.tml
```

```bash
{{ define "email.html" }}
<table border="1">
        <tr>
                <th>报警项</th>
                <th>实例</th>
                <th>报警阀值</th>
                <th>开始时间</th>
        </tr>
        {{ range $i, $alert := .Alerts }}
                <tr>
                        <td>{{ index $alert.Labels "alertname" }}</td>
                        <td>{{ index $alert.Labels "instance" }}</td>
                        <td>{{ index $alert.Annotations "value" }}</td>
                        <td>{{ $alert.StartsAt }}</td>
                </tr>
        {{ end }}
</table>
{{ end }}


#属性解析
{{ define "test.html" }} 表示定义了一个 test.html 模板文件，通过该名称在配置文件中应用上边模板文件就是使用了大量的jinja2模板语言。
$alert.xxx 其实是从默认的告警信息中提取出来的重要信息
```

**模板二**

```bash
[root@ubuntu2404 alertmanager]#vim /usr/local/alertmanager/tmpl/email_template.tmpl
```

```yaml
{{ define "email.html" }}
{{- if gt (len .Alerts.Firing) 0 -}}
{{ range .Alerts }}
=========start==========<br>
告警程序: prometheus_alert <br>
告警级别: {{ .Labels.severity }} <br>
告警类型: {{ .Labels.alertname }} <br>
告警主机: {{ .Labels.instance }} <br>
告警主题: {{ .Annotations.summary }}  <br>
告警详情: {{ .Annotations.description }} <br>
触发时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }} <br>
=========end==========<br>
{{ end }}{{ end -}}
 
{{- if gt (len .Alerts.Resolved) 0 -}}
{{ range .Alerts }}
=========start==========<br>
告警程序: prometheus_alert <br>
告警级别: {{ .Labels.severity }} <br>
告警类型: {{ .Labels.alertname }} <br>
告警主机: {{ .Labels.instance }} <br>
告警主题: {{ .Annotations.summary }} <br>
告警详情: {{ .Annotations.description }} <br>
触发时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }} <br>
恢复时间: {{ .EndsAt.Format "2006-01-02 15:04:05" }} <br>
=========end==========<br>
{{ end }}{{ end -}}
{{- end }}
```



### **告警路由**

Alertmanager的route配置段支持定义"树"状路由表，入口位置称为根节点，每个子节点可以基于匹配条件定义出一个独立的路由分支

- 所有告警都将进入路由根节点，而后进行子节点遍历
- 若路由上的continue字段的值为false，则遇到第一个匹配的路由分支后即终止；否则，将继续匹配后续的子节点

![image-20250304203752491](5day-png/34告警路由.png)

上图所示：Alertmanager中的第一个Route是根节点，每一个match 都是子节点

范例: 路由示例

```yaml
route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'email' 
```

Alertmanager 的相关配置

```bash
# 默认信息的接收者，这一项是必须选项，否则程序启动不成功
[ receiver: <string> ]
# 分组时使用的标签，默认情况下，所有的告警都组织在一起，而一旦指定分组标签，则Alertmanager将按这些标签进行分组
[ group_by: '[' <labelname>, ... ']' ]

# 在匹配成功的前提下，是否继续进行深层次的告警规则匹配
[ continue: <boolean> | default = false ]

# 基于字符串验证，判断当前告警中是否存在标签labelname并且其值等于label value，满足则进行内部的后续处理。
#新版使用指令matchers替换match和match_re
matchers:
  - alertname = Watchdog
  - severity =~ "warning|critical"
#=、!=、=~ 或 !~ 之一。= 表示相等，!= 表示字符串不相等，=~ 用于正则表达式相等，!~ 用于正则表达式不相等。它们具有与 PromQL 选择器中相同的含义。

#旧版用法
match:
  [ <labelname>: <labelvalue>, ... ]

# 基于正则表达式验证，判断当前告警标签的值是否满足正则表达式的内容，满足则进行内部的后续处理
match_re:
  [ <labelname>: <regex>, ... ]

# 发出一组告警通知的初始等待时长；允许等待一个抑制告警到达或收集属于同一组的更多初始告警，通常是0到数分钟
[ group_wait: <duration> | default = 30s ]

# 发送关于新告警的消息之前，需要等待多久；新告警将被添加到已经发送了初始通知的告警组中；一般在5分钟或以上
[ group_interval: <duration> | default = 5m ]

# 成功发送了告警后再次发送告警信息需要等待的时长，一般至少为3个小时
[ repeat_interval: <duration> | default = 4h ]'

# 子路由配置
routes:
  [ - <route> ... ]
  
#注意：每一个告警都会从配置文件中顶级的route进入路由树，需要注意的是顶级的route必须匹配所有告警, 不能有 match 和 match_re
```

配置实例

```
#1) 在prometheus上定制告警规则
groups:
- name: example
  rules:
    ...
- name: nodes_alerts
  rules:
    - alert: DiskWillFillIn12Hours
      expr: predict_linear(node_filesystem_free_bytes{mountpoint="/"}[1h], 12*3600) < 0
      for: 1m
      labels:
        severity: critical
      annotations:
        description: "Disk on {{ $labels.instance }} will fill in approximately 12 hours."
        summary: "磁盘即将在 12 小时内填满"

- name: prometheus_alerts
  rules:
    - alert: PrometheusConfigReloadFailed
      expr: prometheus_config_last_reload_successful == 0
      for: 3m
      labels:
        severity: warning
      annotations:
        description: "Reloading Prometheus configuration has failed on {{ $labels.instance }}."
        summary: "Prometheus 配置重载失败"

#2) 在alertmanager 定制路由
global:
...
templates:
...
route:
  group_by: ['instance']
  ...
  receiver: email-receiver

  routes:
    - match:
        severity: critical
      receiver: leader-team

    - match_re:
        severity: ^(warning)$
      receiver: ops-team

receivers:
  - name: 'leader-team'
    email_configs:
      - to: '2352136389@qq.com'
      - to: 'kang@qq.com'

  - name: 'ops-team'
    email_configs:
      - to: 'hzokang@outlook.com'
      - to: 'kang@outlook.com'

```



案例：

定制的 metric 要求如下

- 如果是每分钟的QPS超过 500 的时候，将告警发给运维团队
- 如果是服务终止的话，发给管理团队

**定制告警规则**

```bash
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml|grep -Ev "^$|^ *#"
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 10.0.0.200:9093		#指定alertmaanager地址
rule_files:
    - "../rules/*.yml"			#指定规则文件路径
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["10.0.0.200:9090"]
  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "10.0.0.201:9100"
          - "10.0.0.200:9100"
          - "10.0.0.202:9100"
  - job_name: "grafana"
    static_configs:
      - targets: 
        - "10.0.0.200:3000"
  - job_name: "pushgateway"
    static_configs:
      - targets: 
        - "10.0.0.200:9091"
  - job_name: "my_metric"
    static_configs:
      - targets: 
        - "10.0.0.200:8000"
        labels: {app: "flask_web"}
  - job_name: "alertmanager"
    static_configs:
      - targets: 
        - "10.0.0.200:9093"	
```

```bash
#配置告警规则
[root@ubuntu2404 ~]#cat /usr/local/prometheus/rules/prometheus_alert_route.yml
groups:
- name: flask_web
  rules:
  - alert: InstanceDown
    expr: up{job="my_metric"} == 0
    for: 1s
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance  }} 停止工作"
      description: "{{ $labels.instance  }} job {{ $labels.job  }} 已经停止1秒钟以上"
      value: "{{$value}}"
- name: flask_QPS
  rules:
  - alert: InstanceQPSIsHight
    expr: increase(request_count_total{job="my_metric"}[1m]) > 500
    for: 1s
    labels:
      severity: warning
    annotations:
      summary: "Instance {{ $labels.instance  }} QPS 持续过高"
      description: "{{ $labels.instance  }} job {{ $labels.job  }} QPS 持续过高"
      value: "{{$value}}"
```

```bash
#指定flask_web规则的labels为 severity: critical
#指定flask_QPS规则的labels为 severity: warning
 
#重启prometheus服务
systemctl reload prometheus.service
```

**定制路由分组**

```bash
#指定路由分组
[root@ubuntu2404 ~]#vim /usr/local/alertmanager/conf/alertmanager.yml 
# 全局配置
global:
  resolve_timeout: 5m                        
  smtp_smarthost: 'smtp.163.com:25'           #基于全局块指定发件人信息,此处设为25，如果465还需要添加tls相关配置
  smtp_from: 'hzokang@163.com'
  smtp_auth_username: 'hzokang@163.com'
  smtp_auth_password: 'EBgCxViwqvnxxbEu'
  smtp_hello: '163.com'
  smtp_require_tls: false        #启用tls安全,默认true,此处设为false

# 模板配置文件
templates:            #加下面两行加载模板文件
  - '../tmpl/*.tmpl'  #相对路径是相对于altermanager.yml文件的路径

# 路由配置
route:
  group_by: ['instance', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'email'
# 新版：使用指令matchers替换match和match_re,如下示例
  routes:
  - receiver: 'leader-team'
    matchers:
    - severity = "critical"
  - receiver: 'ops-team'
    matchers:
    - severity =~ "^(warning)$"
# 旧版：使用match和match_ce
#  routes:
#    - match:
#        severity: critical
#      receiver: 'leader-team'
#
#    - match_re:
#        severity: warning  # `warning` 不需要 `^(warning)$`
#      receiver: 'ops-team'

# 收信人员
receivers:
- name: 'email'
  email_configs:
  - to: 'hzokang@outlook.com'
    send_resolved: true           #问题解决后也会发送恢复通知  
    headers: { Subject: "[WARN] 报警邮件"}   #添加此行,定制邮件标题
    html: '{{ template "email.html" . }}'    #添加此行,调用模板显示邮件正文

- name: 'leader-team'
  email_configs:
  - to: '2352136389@qq.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[CRITICAL] 应用服务报警邮件"}
    send_resolved: true

- name: 'ops-team'
  email_configs:
  - to: 'hzokang@outlook.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[WARNNING] QPS负载报警邮件"}
    send_resolved: true
```

```bash
#语法检查
[root@ubuntu2404 ~]#/usr/local/alertmanager/bin/amtool check-config /usr/local/alertmanager/conf/alertmanager.yml
Checking '/usr/local/alertmanager/conf/alertmanager.yml'  SUCCESS
Found:
 - global config
 - route
 - 0 inhibit rules
 - 3 receivers
 - 1 templates
  SUCCESS
```

```bash
#定义邮件模板文件
[root@ubuntu2404 ~]#vim /usr/local/alertmanager/tmpl/email_template.tmpl 
{{ define "test.html" }}
{{- if gt (len .Alerts.Firing) 0 -}}
{{ range .Alerts }}
=========start==========<br>
告警程序: prometheus_alert <br>
告警级别: {{ .Labels.severity }} <br>
告警类型: {{ .Labels.alertname }} <br>
告警主机: {{ .Labels.instance }} <br>
告警主题: {{ .Annotations.summary }}  <br>
告警详情: {{ .Annotations.description }} <br>
触发时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }} <br>
=========end==========<br>
{{ end }}{{ end -}}
 
{{- if gt (len .Alerts.Resolved) 0 -}}
{{ range .Alerts }}
=========start==========<br>
告警程序: prometheus_alert <br>
告警级别: {{ .Labels.severity }} <br>
告警类型: {{ .Labels.alertname }} <br>
告警主机: {{ .Labels.instance }} <br>
告警主题: {{ .Annotations.summary }} <br>
告警详情: {{ .Annotations.description }} <br>
触发时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }} <br>
恢复时间: {{ .EndsAt.Format "2006-01-02 15:04:05" }} <br>
=========end==========<br>
{{ end }}{{ end -}}
{{- end }}
```

```bash
#服务生效
[root@ubuntu2404 ~]#systemctl reload alertmanager.service
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

### **告警抑制**

![image-20250304213120854](5day-png/34告警抑制.png)

配置解析

```bash
#源告警信息匹配 -- 报警的来源
source_match:
  [ <labelname>: <labelvalue>, ... ]
source_match_re:
  [ <labelname>: <regex>, ... ]

#目标告警信息匹配 - 触发的其他告警
target_match:
  [ <labelname>: <labelvalue>, ... ]
target_match_re:
  [ <labelname>: <regex>, ... ]
  
#目标告警是否是被触发的 - 要保证业务是同一处来源
[ equal: '[' <labelname>, ... ']' ]
#同时告警目标上的标签与之前的告警标签一样，那么就不再告警 
```

```bash
#例如：
集群中的A主机节点异常导致NodeDown告警被触发，该告警会触发一个severity=critical的告警级别。
由于A主机异常导致该主机上相关的服务，会因为不可用而触发关联告警。
根据抑制规则的定义：
如果有新的告警级别为severity=critical，且告警中标签的node值与NodeDown告警的相同
则说明新的告警是由NodeDown导致的，则启动抑制机制,从而停止向接收器发送通知。
  
inhibit_rules:    		# 抑制规则
- source_match:        	# 源标签警报触发时会抑制含有目标标签的警报，被依赖服务A
   alertname: NodeDown 	# 可以针对某一个特定的告警动作规则
   severity: critical 	# 限定源告警规则的等级
 target_match:        	# 定制要被抑制的告警规则的所处位置，依赖服务B
   severity: normal    	# 触发告警目标标签值的正则匹配规则，可以是正则表达式如: 
".*MySQL.*"
 equal:        			# 因为源告警和触发告警必须处于同一业务环境下，确保他们在同一个业务中
    - instance    		# 源告警和触发告警存在于相同的 instance 时，触发告警才会被抑制。
      					# 格式二 equal: ['alertname','operations', 'instance']
#表达式
  up{node="node01.wang.org",...} == 0
      severity: critical
#触发告警
  ALERT{node="node01.wang.org",...,severity=critical}
```

**告警抑制案例**

对于当前的flask应用的监控来说，上面做了两个监控指标：

- 告警级别为 critical 的 服务异常终止
- 告警级别为 warning 的 QPS访问量突然降低为0，这里以服务状态来模拟

```yaml
#准备告警规则
[root@ubuntu2404 ~]#cat /usr/local/prometheus/rules/prometheus_alert_inhibit.yml 
groups:
- name: flask_web
  rules:
  - alert: InstanceDown
    expr: up{job="my_metric"} == 0
    for: 1s
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance  }} 停止工作"
      description: "{{ $labels.instance  }} job {{ $labels.job  }} 已经停止1分钟以上"
      value: "{{$value}}"
- name: flask_QPS
  rules:
  - alert: InstanceQPSIsHight
    expr: increase(request_count_total{job="my_metric"}[1m]) > 500
    for: 1s
    labels:
      severity: warning
    annotations:
      summary: "Instance {{ $labels.instance  }} QPS 持续过高"
      description: "{{ $labels.instance  }} job {{ $labels.job  }} QPS 持续过高"
      value: "{{$value}}"
  - alert: InstanceQPSIsLow                  #判断是否QPS访问为0
    expr: up{job="my_metric"} == 0
    for: 1s
    labels:
      severity: warning
    annotations:
      summary: "Instance {{ $labels.instance  }} QPS 异常为零"
      description: "{{ $labels.instance  }} job {{ $labels.job  }} QPS 异常为0"
      value: "{{$value}}"
```

```bash
#定制告警对象
[root@ubuntu2404 tmpl]#cat /usr/local/alertmanager/conf/alertmanager.yml
# 全局配置
global:
  resolve_timeout: 5m                        
  smtp_smarthost: 'smtp.163.com:25'           #基于全局块指定发件人信息,此处设为25，如果465还需要添加tls相关配置
  smtp_from: ' hzokang@163.com'
  smtp_auth_username: 'hzokang@163.com'
  smtp_auth_password: 'EBgCxViwqvnxxbEu'
  smtp_hello: '163.com'
  smtp_require_tls: false        #启用tls安全,默认true,此处设为false

# 模板配置文件
templates:            #加下面两行加载模板文件
  - '../tmpl/*.tmpl'  #相对路径是相对于altermanager.yml文件的路径

# 路由配置
route:
  group_by: ['instance', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'email'
# 新版：使用指令matchers替换match和match_re,如下示例
  routes:
  - receiver: 'leader-team'
    matchers:
    - severity = "critical"
  - receiver: 'ops-team'
    matchers:
    - severity =~ "^(warning)$"
# 旧版：使用match和match_ce
#  routes:
#    - match:
#        severity: critical
#      receiver: 'leader-team'
#
#    - match_re:
#        severity: warning  # `warning` 不需要 `^(warning)$`
#      receiver: 'ops-team'



# 收信人员
receivers:
- name: 'email'
  email_configs:
  - to: 'hzokang@outlook.com'
    send_resolved: true           #问题解决后也会发送恢复通知  
    headers: { Subject: "[WARN] 报警邮件"}   #添加此行,定制邮件标题
    html: '{{ template "email.html" . }}'    #添加此行,调用模板显示邮件正文

- name: 'leader-team'
  email_configs:
  - to: '2352136389@qq.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[CRITICAL] 应用服务报警邮件"}
    send_resolved: true

- name: 'ops-team'
  email_configs:
  - to: 'hzokang@outlook.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[WARNNING] QPS负载报警邮件"}
    send_resolved: true

# 抑制措施
inhibit_rules: 
- source_match: 
    severity: critical     #被依赖的告警服务
  target_match:             
    severity: warning      #依赖的告警服务
  equal:
    - instance
```

```bash
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
[root@ubuntu2404 ~]#systemctl reload alertmanager.service 
```

 





### 微信告警

就目前来说，使用微信告警的方式主要有以下三种：

- 用个人号发送告警 - 一般需要借助于专属的开发工具来实现
- 用公众号发送告警 - 借助于软件本身的机制来实现。
- 用企业号发送告警 - 借助于软件本身的机制来实现。此为推荐方式



实现步骤

- 企业微信后台配置
  - 创建企业微信、自定义告警部门
  - 创建企业应用认证信息

- Alertmanager配置告警通知人和告警模板

- Prometheus配置告警规则



1 企业信息里面的企业ID

2 创建部门有部门ID

3 创建微信应用，里面有 应用的 AgentId值 和 应用的 Secret的值

```
查看到企业ID的值为 ww644a0d95807e476b
注意: 创建的部门的部门ID为2

应用的 AgentId值1000004
应用的 Secret的值qYgLlipdHtZidsd8qAZaTKKkGkzIyWxuQSeQOk9Si0M
```



```bash
#配置文件
[root@ubuntu2404 ~]#vim /usr/local/alertmanager/conf/alertmanager.yml
# 全局配置
global:
  resolve_timeout: 5m                        
  smtp_smarthost: 'smtp.163.com:25'           #基于全局块指定发件人信息,此处设为25，如果465还需要添加tls相关配置
  smtp_from: ' hzokang@163.com'
  smtp_auth_username: 'hzokang@163.com'
  smtp_auth_password: 'EBgCxViwqvnxxbEu'
  smtp_hello: '163.com'
  smtp_require_tls: false        #启用tls安全,默认true,此处设为false
  #必须项
  wechat_api_corp_id: 'ww644a0d95807e476b'
  #此处的微信信息可省略,下面wechat_configs 也提供了相关信息
  wechat_api_url: 'https://qyapi.weixin.qq.com/cgi-bin/'
  wechat_api_secret: 'qYgLlipdHtZidsd8qAZaTKKkGkzIyWxuQSeQOk9Si0M'

# 模板配置文件
templates:            #加下面两行加载模板文件
  - '../tmpl/*.tmpl'  #相对路径是相对于altermanager.yml文件的路径

# 路由配置
route:
  group_by: ['instance', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'email'
# 新版：使用指令matchers替换match和match_re,如下示例
  routes:
  - receiver: 'wechat'
    matchers:
    - severity = "critical"
  - receiver: 'ops-team'
    matchers:
    - severity =~ "^(warning)$"
# 旧版：使用match和match_ce
#  routes:
#    - match:
#        severity: critical
#      receiver: 'leader-team'
#
#    - match_re:
#        severity: warning  # `warning` 不需要 `^(warning)$`
#      receiver: 'ops-team'



# 收信人员
receivers:
- name: 'email'
  email_configs:
  - to: 'hzokang@outlook.com'
    send_resolved: true           #问题解决后也会发送恢复通知  
    headers: { Subject: "[WARN] 报警邮件"}   #添加此行,定制邮件标题
    html: '{{ template "email.html" . }}'    #添加此行,调用模板显示邮件正文

- name: 'leader-team'
  email_configs:
  - to: '2352136389@qq.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[CRITICAL] 应用服务报警邮件"}
    send_resolved: true

- name: 'ops-team'
  email_configs:
  - to: 'hzokang@outlook.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[WARNNING] QPS负载报警邮件"}
    send_resolved: true

- name: 'wechat'
  wechat_configs:
  - to_party: '2'
    #to_user: '@all' #支持给企业内所有人发送
    agent_id: '1000004'
    #api_secret: 'qYgLlipdHtZidsd8qAZaTKKkGkzIyWxuQSeQOk9Si0M'
    send_resolved: true
    message: '{{ template "wechat.default.message" . }}'

# 抑制措施
inhibit_rules: 
- source_match: 
    severity: critical     #被依赖的告警服务
  target_match:             
    severity: warning      #依赖的告警服务
  equal:
    - instance
```

```bash
#模板文件
[root@ubuntu2404 tmpl]#vim wechat.tmpl 
{{ define "wechat.default.message" }}
{{- if gt (len .Alerts.Firing) 0 -}}
{{- range $index, $alert := .Alerts -}}
{{- if eq $index 0 }}
========= 监控报警 =========
告警状态：{{   .Status }}
告警级别：{{ .Labels.severity }}
告警类型：{{ $alert.Labels.alertname }}
故障主机: {{ $alert.Labels.instance }}
告警主题: {{ $alert.Annotations.summary }}
告警详情: {{ $alert.Annotations.message }}{{ $alert.Annotations.description}};
触发阀值：{{ .Annotations.value }}
故障时间: {{ ($alert.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
========= = end =  =========
{{- end }}
{{- end }}
{{- end }}
{{- if gt (len .Alerts.Resolved) 0 -}}
{{- range $index, $alert := .Alerts -}}
{{- if eq $index 0 }}
========= 异常恢复 =========
告警类型：{{ .Labels.alertname }}
告警状态：{{   .Status }}
告警主题: {{ $alert.Annotations.summary }}
告警详情: {{ $alert.Annotations.message }}{{ $alert.Annotations.description}};
故障时间: {{ ($alert.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
恢复时间: {{ ($alert.EndsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
{{- if gt (len $alert.Labels.instance) 0 }}
实例信息: {{ $alert.Labels.instance }}
{{- end }}
========= = end =  =========
{{- end }}
{{- end }}
{{- end }}
{{- end }}
```

```
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
[root@ubuntu2404 ~]#systemctl reload alertmanager.service 
```



### 钉钉告警

Prometheus 不直接支持钉钉告警, 但对于prometheus来说，它所支持的告警机制非常多，尤其还支持通过webhook，从而可以实现市面上大部分的实时动态的告警平台。

```
https://github.com/timonwong/prometheus-webhook-dingtalk
```

**工作原理**

![image-20250305184521946](5day-png/34dingtalk工作原理.png)

 添加机器人

```bash
#加签
SEC9bb07d564e34faab27db1f51356f8665ea00efef89e1b6ff694cbd9c78e58f9e

#关键字
PromAlert

#url路径
https://oapi.dingtalk.com/robot/send?access_token=283c518db0da5e50e87f6440a083c0dc8dd745b8ab71f8ea9c7645f07d65c5ac 
```

```
https://github.com/timonwong/prometheus-webhook-dingtalk/
```

```
https://github.com/timonwong/prometheus-webhook-dingtalk/releases/download/v2.1.0/prometheus-webhook-dingtalk-2.1.0.linux-amd64.tar.gz
```

**prometheus-webhook-dingtalk软件部署**

二进制安装

```bash
[root@ubuntu2404 ~]#ls
prometheus-webhook-dingtalk-2.1.0.linux-amd64.tar.gz
[root@ubuntu2404 ~]#tar xf prometheus-webhook-dingtalk-2.1.0.linux-amd64.tar.gz -C /usr/local/
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ln -s prometheus-webhook-dingtalk-2.1.0.linux-amd64/ dingtalk
[root@ubuntu2404 dingtalk]#ls
config.example.yml  contrib  LICENSE  prometheus-webhook-dingtalk
[root@ubuntu2404 dingtalk]#mkdir bin conf
[root@ubuntu2404 dingtalk]#mv config.example.yml conf/config.yml
[root@ubuntu2404 dingtalk]#mv prometheus-webhook-dingtalk bin/

#创建service文件
[root@ubuntu2404 dingtalk]#vim /lib/systemd/system/dingtalk.service
[Unit]
Description=alertmanager project
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/dingtalk/bin/prometheus-webhook-dingtalk --config.file=/usr/local/dingtalk/conf/config.yml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 dingtalk]#chown -R prometheus: /usr/local/dingtalk/

#修改配置文件
[root@ubuntu2404 ~]#cat /usr/local/dingtalk/conf/config.example.yml
## Request timeout
# timeout: 5s

## Uncomment following line in order to write template from scratch (be careful!)
#no_builtin_template: true

## Customizable templates path
#templates:
#  - contrib/templates/legacy/template.tmpl

## You can also override default template using `default_message`
## The following example to use the 'legacy' template from v0.3.0
#default_message:
#  title: '{{ template "legacy.title" . }}'
#  text: '{{ template "legacy.content" . }}'

## Targets, previously was known as "profiles"
targets:
  webhook1:
    url: https://oapi.dingtalk.com/robot/send?access_token=283c518db0da5e50e87f6440a083c0dc8dd745b8ab71f8ea9c7645f07d65c5ac 
    # secret for signature
    secret: SEC9bb07d564e34faab27db1f51356f8665ea00efef89e1b6ff694cbd9c78e58f9e
    
    
[root@ubuntu2404 ~]#systemctl enable --now dingtalk.service 
[root@ubuntu2404 ~]#systemctl status dingtalk.service 
```

```bash
[root@ubuntu2404 ~]#cat /usr/local/alertmanager/conf/alertmanager.yml 
# 全局配置
global:
  resolve_timeout: 5m                        
  smtp_smarthost: 'smtp.163.com:25'           #基于全局块指定发件人信息,此处设为25，如果465还需要添加tls相关配置
  smtp_from: ' hzokang@163.com'
  smtp_auth_username: 'hzokang@163.com'
  smtp_auth_password: 'EBgCxViwqvnxxbEu'
  smtp_hello: '163.com'
  smtp_require_tls: false        #启用tls安全,默认true,此处设为false
  #必须项
  wechat_api_corp_id: 'ww644a0d95807e476b'
  #此处的微信信息可省略,下面wechat_configs 也提供了相关信息
  wechat_api_url: 'https://qyapi.weixin.qq.com/cgi-bin/'
  wechat_api_secret: 'qYgLlipdHtZidsd8qAZaTKKkGkzIyWxuQSeQOk9Si0M'

# 模板配置文件
templates:            #加下面两行加载模板文件
  - '../tmpl/*.tmpl'  #相对路径是相对于altermanager.yml文件的路径

# 路由配置
route:
  group_by: ['instance', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'email'
# 新版：使用指令matchers替换match和match_re,如下示例
  routes:
  - receiver: 'dingtalk'
    matchers:
    - severity = "critical"
  - receiver: 'ops-team'
    matchers:
    - severity =~ "^(warning)$"
# 旧版：使用match和match_ce
#  routes:
#    - match:
#        severity: critical
#      receiver: 'leader-team'
#
#    - match_re:
#        severity: warning  # `warning` 不需要 `^(warning)$`
#      receiver: 'ops-team'



# 收信人员
receivers:
- name: 'email'
  email_configs:
  - to: 'hzokang@outlook.com'
    send_resolved: true           #问题解决后也会发送恢复通知  
    headers: { Subject: "[WARN] 报警邮件"}   #添加此行,定制邮件标题
    html: '{{ template "email.html" . }}'    #添加此行,调用模板显示邮件正文

- name: 'leader-team'
  email_configs:
  - to: '2352136389@qq.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[CRITICAL] 应用服务报警邮件"}
    send_resolved: true

- name: 'ops-team'
  email_configs:
  - to: 'hzokang@outlook.com'
    html: '{{ template "test.html" . }}'
    headers: { Subject: "[WARNNING] QPS负载报警邮件"}
    send_resolved: true

- name: 'wechat'
  wechat_configs:
  - to_party: '2'
    #to_user: '@all' #支持给企业内所有人发送
    agent_id: '1000004'
    #api_secret: 'qYgLlipdHtZidsd8qAZaTKKkGkzIyWxuQSeQOk9Si0M'
    send_resolved: true
    message: '{{ template "wechat.default.message" . }}'

- name: 'dingtalk'
  webhook_configs:
  - url: 'http://10.0.0.200:8060/dingtalk/webhook1/send'
    send_resolved: true

# 抑制措施
inhibit_rules: 
- source_match: 
    severity: critical     #被依赖的告警服务
  target_match:             
    severity: warning      #依赖的告警服务
  equal:
    - instance
```

模板文件 需要带上关键字PromAlert

```bash
[root@ubuntu2404 ~]#vim /usr/local/dingtalk/contrib/templates/dingtalk.tmpl
{{ define "__subject" }}
[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] 
{{ .GroupLabels.SortedPairs.Values | join " " }} 
{{ if gt (len .CommonLabels) (len .GroupLabels) }}
({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }})
{{ end }}
{{ end }}

{{ define "__alertmanagerURL" }}
{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}
{{ end }}

{{ define "__text_alert_list" }}
{{ range . }}
**Labels**
{{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}

**Annotations**
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}

**Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
{{ end }}
{{ end }}

{{ define "___text_alert_list" }}
{{ range . }}
---
**告警主题:** {{ .Labels.alertname | upper }}
**告警级别:** {{ .Labels.severity | upper }}
**触发时间:** {{ dateInZone "2006-01-02 15:04:05" (.StartsAt) "Asia/Shanghai" }}

**事件信息:**
{{ range .Annotations.SortedPairs }} {{ .Value | markdown | html }}
{{ end }}

**事件标签:**
{{ range .Labels.SortedPairs }}
{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}
> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
{{ end }}
{{ end }}
{{ end }}

{{ define "___text_alertresovle_list" }}
{{ range . }}
---
**告警主题:** {{ .Labels.alertname | upper }}
**告警级别:** {{ .Labels.severity | upper }}
**触发时间:** {{ dateInZone "2006-01-02 15:04:05" (.StartsAt) "Asia/Shanghai" }}
**结束时间:** {{ dateInZone "2006-01-02 15:04:05" (.EndsAt) "Asia/Shanghai" }}

**事件信息:**
{{ range .Annotations.SortedPairs }} {{ .Value | markdown | html }}
{{ end }}

**事件标签:**
{{ range .Labels.SortedPairs }}
{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}
> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
{{ end }}
{{ end }}
{{ end }}

{{ define "_default.title" }}
{{ template "__subject" . }}
{{ end }}

{{ define "_default.content" }}
[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}]
**[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**

{{ if gt (len .Alerts.Firing) 0 -}}
![警报](http://www.wangxiaochun.com:8888/testdir/images/ERROR.png)
**======== PromAlert 告警触发 ========**
{{ template "___text_alert_list" .Alerts.Firing }}
{{- end }}

{{ if gt (len .Alerts.Resolved) 0 -}}
![恢复](http://www.wangxiaochun.com:8888/testdir/images/OK.png)
**======== PromAlert 告警恢复 ========**
{{ template "___text_alertresovle_list" .Alerts.Resolved }}
{{- end }}
{{- end }}

{{ define "legacy.title" }}
{{ template "__subject" . }}
{{ end }}

{{ define "legacy.content" }}
[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}]
**[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**

{{ template "__text_alert_list" .Alerts.Firing }}
{{- end }}

{{ define "_ding.link.title" }}
{{ template "_default.title" . }}
{{ end }}

{{ define "_ding.link.content" }}
{{ template "_default.content" . }}
{{ end }}

```

 修改配置文件

```bash
[root@ubuntu2404 ~]#cat /usr/local/dingtalk/conf/config.yml
## Request timeout
# timeout: 5s

## Uncomment following line in order to write template from scratch (be careful!)
#no_builtin_template: true

## Customizable templates path
#templates:
#  - contrib/templates/legacy/template.tmpl

templates:
  - '/usr/local/dingtalk/contrib/templates/dingtalk.tmpl'
default_message:
  title: '{{ template "_ding.link.title" . }}'
  text: '{{ template "_ding.link.content" . }}'

## You can also override default template using `default_message`
## The following example to use the 'legacy' template from v0.3.0
#default_message:
#  title: '{{ template "legacy.title" . }}'
#  text: '{{ template "legacy.content" . }}'

## Targets, previously was known as "profiles"
targets:
  webhook1:
    url: https://oapi.dingtalk.com/robot/send?access_token=283c518db0da5e50e87f6440a083c0dc8dd745b8ab71f8ea9c7645f07d65c5ac 
    # secret for signature
    secret: SEC9bb07d564e34faab27db1f51356f8665ea00efef89e1b6ff694cbd9c78e58f9e
```

```bash
systemctl reload prometheus.service 
systemctl reload dingtalk.service 
systemctl reload alertmanager.service
```





## **Alertmanager** **高可用**

<img src="5day-png/34alertmanager高可用.png" alt="image-20250305200805898" style="zoom:50%;" />

**Gossip** **谣言协议实现**

Alertmanager引入了Gossip机制。Gossip机制为多个Alertmanager之间提供了信息传递的机制。确保及时、在多个Alertmanager分别接收到相同告警信息的情况下，也只有一个告警通知被发送给Receiver。

![image-20250305200936665](5day-png/34alertmanager高可用1.png)

Gossip是分布式系统中被广泛使用的协议，用于实现分布式节点之间的信息交换和状态同步。Gossip协议同步状态类似于流言或者病毒的传播，如下所示：

![image-20250305201001872](5day-png/34alertmanager高可用2.png)

Gossip有两种实现方式分别为Push-based和Pull-based。

在Push-based当集群中某一节点A完成一个工作后，随机的挑选其它节点B并向其发送相应的消息，节点B接收到消息后在重复完成相同的工作，直到传播到集群中的所有节点。

而Pull-based的实现中节点A会随机的向节点B发起询问是否有新的状态需要同步，如果有则返回。

搭建本地集群环境

为了能够让Alertmanager节点之间进行通讯，需要在Alertmanager启动时设置相应的参数。其中主要的参数包括：

```bash
--web.listen-address string      #当前实例Web监听地址和端口,默认9093
--cluster.listen-address string  #当前实例集群服务监听地址,默认9094,集群必选
--cluster.peer value             #后续集群实例在初始化时需要关联集群中的已有实例的服务地址,集群的后续节点必选
```



范例：在不同主机用Alertmanager实现

第一台主机定义Alertmanager实例A，其中Alertmanager的服务运行在9093端口，集群服务地址运行在9094端口。

```bash
[root@ubuntu2404 ~]#/usr/local/alertmanager/bin/alertmanager --config.file=/usr/local/alertmanager/conf/alertmanager.yml --storage.path=/usr/local/alertmanager/data --web.listen-address=0.0.0.0:9093 --cluster.listen-address=0.0.0.0:9094
```

第二台主机定义Alertmanager实例B，其中Alertmanager的服务运行在9093端口，集群服务运行在9094端口。

为了将A1，A2组成集群。 A2启动时需要定义--cluster.peer参数并且指向A1实例的集群服务地址:9094

```
[root@ubuntu2404 ~]#/usr/local/alertmanager/bin/alertmanager --config.file=/usr/local/alertmanager/conf/alertmanager.yml --storage.path=/usr/local/alertmanager/data --web.listen-address=0.0.0.0:9093 --cluster.listen-address=0.0.0.0:9094 --cluster.peer=10.0.0.200:9094
```

创建配置文件

```bash
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
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
          - 10.0.0.200:9093
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "../rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "alertmanager"
    static_configs:
      - targets: 
        - "10.0.0.200:9093"
        - "10.0.0.202:9093"
```



## 服务发现

Prometheus Server的数据抓取工作于Pull模型，因而，它必需要事先知道各Target的位置，然后才能从相应的Exporter或Instrumentation中抓取数据。

```bash
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["10.0.0.200:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "10.0.0.201:9100"
          - "10.0.0.200:9100"
          - "10.0.0.202:9100"
```

| 方法                | 解析                                                         |
| ------------------- | ------------------------------------------------------------ |
| 静态服务发现        | 在Prometheus配置文件中通过static_config项,手动添加监控的主机实现 |
| 基于文件的服务发现  | 将各target记录到文件中，prometheus启动后，周期性刷新这个文件，从而获取最新的target |
| 基于DNS的服务发 现  | 针对一组DNS域名进行定期查询，以发现待监控的目标，并持续监视相关资源的变动 |
| 基于Consul服务发现  | 基于 Consul 服务实现动态自动发现                             |
| 基于HTTP的服务发 现 | 基于 HTTP 的服务发现提供了一种更通用的方式来配置静态目标，并用作插入自定义服务 发现机制的接口。 它从包含零个或多个 列表的 HTTP 端点获取目标。 目标必须回复 HTTP 200 响应。 HTTP header Content-Type 必须是 application/json，body 必须是有效的 JSON。 |
| 基于API 的服务发现  | 支持将Kubernetes API Server中Node、Service、Endpoint、Pod和Ingress等资源类型下相应的各资源对象视作target，并持续监视相关资源的变动。 |

**服务发现原理**

![image-20250306092444598](5day-png/34服务发现原理.png)

Prometheus服务发现机制大致涉及到三个部分：

- 配置处理模块解析的prometheus.yml配置中scrape_configs部分，将配置的job生成一个个Discover服务，不同的服务发现协议都会有各自的Discoverer实现方式，它们根据实现逻辑去发现target，并将其放入到targets列表中
- DiscoveryManager组件内部有一个定时周期触发任务，每5秒检查target列表，如果有变更则将target列表中target信息放入到syncCh消息池中
- scrape组件会监听syncCh消息池，这样需要监控的target信息就传递给scrape组件，然后reload将target纳入监控开始抓取监控指标

### **文件服务发现**

文件发现原理

- Target的文件可由手动创建或利用工具生成，例如Ansible或Saltstack等配置管理系统，也可能是由脚本基于CMDB定期查询生成
- 文件可使用YAML和JSON格式，它含有定义的Target列表，以及可选的标签信息,YAML 适合于运维场景, JSON 更适合于开发场景
- Prometheus Server定期从文件中加载Target信息，根据文件内容发现相应的Target

配置过程和格式

```bash
#准备主机节点列表文件,可以支持yaml格式和json格式
#注意：此文件不建议就地编写生成，可能出现加载一部分的情况
cat targets/prometheus*.yaml
- targets:
  - master1:9100
  labels:
    app: prometheus

#修改prometheus配置文件自动加载实现自动发现
cat prometheus.yml
......
  - job_name: 'file_sd_prometheus'
	scrape_interval: 10s                #指定抓取数据的时间间隔,默认继取全局的配置15s
	file_sd_configs:
	- files: 						#指定要加载的文件列表
	  - targets/prometheus*.yaml 	#要加载的yml或json文件路径，支持glob通配符,相对路径是相对于prometheus.yml配置文件路径
	    refresh_interval: 2m 			#每隔2分钟重新加载一次文件中定义的Targets，默认为5m
      
#注意：文件格式都是yaml或json格式
```

文件发现会生成meta_label

```
__meta_filepath="/usr/local/prometheus/conf/targets/prometheues-server.yml"
```

**案例yaml格式**

```bash
#创建目标目录
[root@ubuntu2404 ~]#mkdir /usr/local/prometheus/conf/targets -p
[root@ubuntu2404 ~]#cd /usr/local/prometheus/conf/targets/
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/targets/prometheus-server.yml 
- targets:
  - 10.0.0.200:9090
  labels:
    app: prometheus-server
    job: prometheus-server
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/targets/prometheus-node.yml 
- targets:
  - 10.0.0.200:9100
  labels:
    app: prometheus-node
    job: prometheus-node
- targets:
  - 10.0.0.201:9100
  - 10.0.0.202:9100
  labels:
    app: node-exporter
    job: node
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/targets/prometheus-flask.yml 
- targets:
  - 10.0.0.200:8000
  labels:
    app: flask-web
    job: prometheus-flask
    
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
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
          - 10.0.0.200:9093
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "../rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
  - job_name: "file_sd_prometheus"
    scrape_interval: 10s		 #指定抓取数据的时间间隔
    file_sd_configs:
    - files:
      - targets/prometheus-server.yml
      refresh_interval: 10s		#指定重读文件的时间间隔,默认值5m

  - job_name: "file_sd_node_exporter"
    scrape_interval: 10s
    file_sd_configs:
    - files:
      - targets/prometheus-node.yml
      refresh_interval: 10s

  - job_name: "file_sd_flask_web"
    scrape_interval: 10s
    file_sd_configs:
    - files:
      - targets/prometheus-flask.yml
      refresh_interval: 10s
```

**案例JSON格式**

```bash
#yaml转json
[root@ubuntu2404 ~]#apt update && apt install reserialize -y
[root@ubuntu2404 targets]#ls
prometheus-flask.yml  prometheus-node.yml  prometheus-server.yml
[root@ubuntu2404 targets]#yaml2json prometheus-flask.yml | jq > prometheus-flask.json
[root@ubuntu2404 targets]#ls
prometheus-flask.json  prometheus-flask.yml  prometheus-node.yml  prometheus-server.yml
[root@ubuntu2404 targets]#cat prometheus-flask.json 
[
  {
    "targets": [
      "10.0.0.200:8000"
    ],
    "labels": {
      "app": "flask-web",
      "job": "prometheus-flask"
    }
  }
]

[root@ubuntu2404 conf]#cat /usr/local/prometheus/conf/prometheus.yml
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
          - 10.0.0.200:9093
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "../rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
  - job_name: "file_sd_flask_web"
    scrape_interval: 10s
    file_sd_configs:
    - files:
      - targets/prometheus-flask.json
      refresh_interval: 10s
```

### DNS 服务发现

<img src="5day-png/34DNS服务发现.png" alt="image-20250306151704582" style="zoom:50%;" />

```
DNS RR 类型 5个字段
$TTL 1D
名字   IN TYPE Value
www  2D   IN  A  1.2.3.4

A 域名--> IPv4
AAAA 域名--> IPv6
PTR IPv4 -->  域名
CNAME 别名  域名 --》 别名的域名
SOA 当前区域zone 的meta 数据，
NS  名称服务器 DNS服务器信息
MX  邮件服务
SRV 服务发现 TCP，UDP
TXT https  
```

**DNS安装脚本**

```bash
[root@ubuntu2404 ~]#cat install_dns.sh 
#!/bin/bash

#在HOST_LIST输入FQDN和IP的对应关系
HOST_LIST="
www 10.0.0.200"

DOMAIN=kang.org

LOCALHOST=`hostname -I | awk '{print $1}'`

. /etc/os-release


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


install_dns () {
    if [ $ID = 'centos' -o $ID = 'rocky' ];then
        yum install -y  bind bind-utils
    elif [ $ID = 'ubuntu' ];then
        apt update
        apt install -y bind9 bind9-utils bind9-host bind9-dnsutils
    else
        color "不支持此操作系统，退出!" 1
        exit
    fi
    
}

config_dns () {
    if [ $ID = 'centos' -o $ID = 'rocky' ];then
        sed -i -e '/listen-on/s/127.0.0.1/localhost/' -e '/allow-query/s/localhost/any/' -e 's/dnssec-enable yes/dnssec-enable no/' -e 's/dnssec-validation yes/dnssec-validation no/'  /etc/named.conf
        cat >>  /etc/named.rfc1912.zones <<EOF
zone "$DOMAIN" IN {
    type master;
    file  "$DOMAIN.zone";
};
EOF
        cat > /var/named/$DOMAIN.zone <<EOF
\$TTL 1D
@       IN SOA  master admin (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS       master
master  A        ${LOCALHOST}         
EOF
        echo "$HOST_LIST" | while read line; do 
            awk '{print $1,"\tA\t",$2}' 
        done >> /etc/bind/$DOMAIN.zone

        chmod 640 /var/named/$DOMAIN.zone
        chgrp named /var/named/$DOMAIN.zone
    elif [ $ID = 'ubuntu' ];then
        sed -i 's/dnssec-validation auto/dnssec-validation no/' /etc/bind/named.conf.options
        cat >>  /etc/bind/named.conf.default-zones <<EOF
zone "$DOMAIN" IN {
    type master;
    file  "/etc/bind/$DOMAIN.zone";
};
EOF
        cat > /etc/bind/$DOMAIN.zone <<EOF
\$TTL 1D
@       IN SOA  master admin (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS       master
master  A        ${LOCALHOST}         
EOF
        echo "$HOST_LIST" | while read line; do
            awk '{print $1,"\tA\t",$2}' 
        done >> /etc/bind/$DOMAIN.zone

        chgrp bind  /etc/bind/$DOMAIN.zone
    else
        color "不支持此操作系统，退出!" 1
        exit
    fi
}

start_service () {
    systemctl enable named
    systemctl restart named
    systemctl is-active named.service
    if [ $? -eq 0 ] ;then 
        color "DNS 服务安装成功!" 0  
    else
        color "DNS 服务安装失败!" 1
    exit 1
    fi   
}

install_dns

config_dns

start_service
```

SRV 记录

- SRV记录是服务器资源记录的缩写，记录服务器提供的服务，SRV记录的作用是说明一个服务器能够提供什么样的服务。
- 在RFC2052中才对SRV记录进行了定义，很多老版本的DNS服务器并不支持SRV记录。
- SRV 资源记录允许为单个域名使用多个服务器，轻松地将服务从一个主机移动到另一个主机，并将某些主机指定为服务的主服务器，将其他主机指定为备份
- 客户端要求特定域名的特定服务/协议，并获取任何可用服务器的名称

```
RFC2782中对于SRV的定义格式是：
_<Service>._<Proto>.<Name> TTL Class SRV Priority Weight Port Target

#格式解析：
Service 	所需服务的符号名称，在Assigned Numbers或本地定义。
   			服务标识符前面加下划线(_)，以避免与常规出现的DNS标签发生冲突。
Proto 		所需协议的符号名称,该名称不区分大小写。_TCP和_UDP目前是该字段最常用的值
 			前面加下划线_，以防止与自然界中出现的DNS标签发生冲突。
Name   		此RR所指的域名。在这个域名下SRV RR是唯一的。
Class 		定制DNS记录类，它主要有以下三种情况：
			对于涉及Internet的主机名、IP地址等DNS记录，记录的CLASS设置为IN。
 			其他两类用的比较少，比如CS(CSNET类)、CH(CHAOS类)、HS(Hesiod)等。
 			每个类都是一个独立的名称空间，其中DNS区域的委派可能不同。
Port 		服务在目标主机上的端口。范围是0-65535。 这是网络字节顺序中的16位无符号整数。
Target 		目标主机的域名。域名必须有一个或多个地址记录，域名绝不能是别名。
 			敦促（但不强求）实现在附加数据部分中返回地址记录。
 			值为"." 表示该域名明确无法提供该服务。
```

```
https://prometheus.io/blog/2015/06/01/advanced-service-discovery/#discovery-with-dns-srv-records
```

```bash
#官方配置实例

#定制job对象
job {
  name: "api-server"
  sd_name: "telemetry.eu-west.api.srv.example.org"
  metrics_path: "/metrics"
}

#定制解析记录
scrape_configs:
- job_name: 'myjob'

  dns_sd_configs:
  - names:
    - 'telemetry.eu-west.api.srv.example.org'
    - 'telemetry.us-west.api.srv.example.org'
    - 'telemetry.eu-west.auth.srv.example.org'
    - 'telemetry.us-east.auth.srv.example.org'

  relabel_configs:
  - source_labels: ['__meta_dns_name']
    regex:         'telemetry\.(.+?)\..+?\.srv\.example\.org'
    target_label:  'zone'
    replacement:   '$1'
  - source_labels: ['__meta_dns_name']
    regex:         'telemetry\..+?\.(.+?)\.srv\.example\.org'
    target_label:  'job'
    replacement:   '$1'
```

**DNS服务发现案例**

```bash
[root@ubuntu2404 ~]#cat /etc/bind/kang.org.zone
$TTL 1D
@       IN SOA  master admin (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS       master
master  A       10.0.0.200         
node1   A       10.0.0.200
node2   A       10.0.0.201
node3   A       10.0.0.202
flask   A       10.0.0.200


_prometheus._tcp.kang.org. 1H IN SRV 10 10 9100 node1.kang.org.
_prometheus._tcp.kang.org. 1H IN SRV 10 10 9100 node2.kang.org.
_prometheus._tcp.kang.org. 1H IN SRV 10 10 9100 node3.kang.org.
```

```bash
[root@ubuntu2404 ~]#cat /etc/bind/kang.org.zone
$TTL 1D
@       IN SOA  master admin (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS       master
master  A       10.0.0.200         
node1   A       10.0.0.200
node2   A       10.0.0.201
node3   A       10.0.0.202
flask   A       10.0.0.200


_prometheus._tcp 1H IN SRV 10 10 9100 node1
_prometheus._tcp 1H IN SRV 10 10 9100 node2
_prometheus._tcp 1H IN SRV 10 10 9100 node3
```

```bash
[root@ubuntu2404 ~]#dig srv _prometheus._tcp.kang.org

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> srv _prometheus._tcp.kang.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2650
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;_prometheus._tcp.kang.org.     IN      SRV

;; ANSWER SECTION:
_prometheus._tcp.kang.org. 3600 IN      SRV     10 10 9100 node3.kang.org.
_prometheus._tcp.kang.org. 3600 IN      SRV     10 10 9100 node1.kang.org.
_prometheus._tcp.kang.org. 3600 IN      SRV     10 10 9100 node2.kang.org.

;; ADDITIONAL SECTION:
node1.kang.org.         86400   IN      A       10.0.0.200
node2.kang.org.         86400   IN      A       10.0.0.201
node3.kang.org.         86400   IN      A       10.0.0.202

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Thu Mar 06 15:35:51 CST 2025
;; MSG SIZE  rcvd: 222
```

```yaml
#添加下面行
- job_name: 'dns_sd_flask'  # 实现单个主机定制的信息解析，也支持DNS或/etc/hosts文件实现解析
  dns_sd_configs:
    - names: ['flask.kang.org']
      type: A  # 指定记录类型，默认SRV
      port: 8000  # 不是SRV时，需要指定Port号
      refresh_interval: 10s

- job_name: 'dns_sd_node_exporter'  # 实现批量主机解析
  dns_sd_configs:
    - names: ['_prometheus._tcp.kang.org']  # SRV记录必须通过DNS的实现
      refresh_interval: 10s  # 指定DNS资源记录的刷新间隔, 默认30s
```

```bash
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
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
          - 10.0.0.200:9093
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "../rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
- job_name: 'dns_sd_flask'  # 实现单个主机定制的信息解析，也支持DNS或/etc/hosts文件实现解析
  dns_sd_configs:
    - names: ['flask.kang.org']
      type: A  # 指定记录类型，默认SRV
      port: 8000  # 不是SRV时，需要指定Port号
      refresh_interval: 10s

- job_name: 'dns_sd_node_exporter'  # 实现批量主机解析
  dns_sd_configs:
    - names: ['_prometheus._tcp.kang.org']  # SRV记录必须通过DNS的实现
      refresh_interval: 10s  # 指定DNS资源记录的刷新间隔, 默认30s
```

```bash
#修改完DNS服务器配置文件，需要清理下DNS缓存
[root@ubuntu2404 ~]#systemctl restart systemd-resolved.service 
```

```bash
[root@ubuntu2404 ~]#cat /usr/local/prometheus/conf/prometheus.yml
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
          - 10.0.0.200:9093
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
    - "../rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
- job_name: 'dns_sd_flask'  # 实现单个主机定制的信息解析，也支持DNS或/etc/hosts文件实现解析
  dns_sd_configs:
    - names: ['flask.kang.org']
      type: A  # 指定记录类型，默认SRV
      port: 8000  # 不是SRV时，需要指定Port号
      refresh_interval: 10s

- job_name: 'dns_sd_node_exporter'  # 实现批量主机解析
  dns_sd_configs:
    - names: ['_prometheus._tcp.kang.org']  # SRV记录必须通过DNS的实现
      refresh_interval: 10s  # 指定DNS资源记录的刷新间隔, 默认30s
  relabel_configs:
  - source_labels: ['__meta_dns_name']
    regex:         '(.+?)\.kang\.org'
    target_label:  'server'
    replacement:   '$1'
  - source_labels: ['__meta_dns_name']
    regex:         '(.+?)\.kang\.org'
    target_label:  'server'
    replacement:   '$1'
```



### **Consul** **服务发现**

服务间的通信成为了迈向微服务大门的第一道难关：

- ServiceA 如何知道 ServiceB 在哪里
- ServiceB 可能会有多个副本提供服务，其中有些可能会挂掉，如何避免访问到"不健康的"的 ServiceB
- 如何控制只有 ServiceA 可以访问到 ServiceB



![image-20250306161522419](5day-png/34consul服务发现工作原理.png)

Consul 是HashiCorp 公司开发的一种服务网格解决方案，使团队能够管理服务之间以及跨本地和多云环境和运行时的安全网络连接。

Consul 提供服务发现、服务网格、流量管理和网络基础设施设备的自动更新

Consul是一个用来实现分布式系统的服务发现与配置的开源工具

Consul采用golang开发

Consul具有高可用和横向扩展特性。

Consul的一致性协议采用更流行的Raft 算法（Paxos的简单版本），用来保证服务的高可用

Consul 使用 GOSSIP 协议（P2P的分布式协议去中心化结构下，通过将信息部分传递，达到全集群的状态信息传播,和 Raft 目标是强一致性不同，Gossip 达到的是最终一致性）管理成员和广播消息, 并且支持 ACL 访问控制

Consul 自带一个Web UI管理系统， 可以通过参数启动并在浏览器中直接查看信息。

Consul 提供了一个控制平面，使您能够注册、查询和保护跨网络部署的服务。控制平面是网络基础结构的一部分，它维护一个中央注册表来跟踪服务及其各自的 IP 地址。它是一个分布式系统，在节点集群上运行，例如物理服务器、云实例、虚拟机或容器。

官网：

```
https://www.consul.io/
```

Consul 关键特性：

- service discovery：consul通过DNS或者HTTP接口实现服务注册和服务发现
- health checking：健康检测使consul可以快速的告警在集群中的操作。和服务发现的集成，可以防止服务转发到故障的服务上面。
- key/value storage：一个用来存储动态配置的系统。提供简单的HTTP接口，可以在任何地方操作
- multi-datacenter：无需复杂的配置，即可支持任意数量的区域

**Prometheus 基于的Consul服务发现过程：**

<img src="5day-png/34Prometheus 基于的Consul服务发现过程.png" alt="image-20250306160925517" style="zoom:50%;" />

- 安装并启动 Consul 服务
- 在Prometheus的配置中关联 Consul服务发现
- 新增服务节点向Consul进行注册
- Prometheus 自动添加新增的服务节点的Target

#### 安装

```bash
#官方仓库安装
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install consul
```

```bash
#二进制安装
https://developer.hashicorp.com/consul/install?product_intent=consul
[root@ubuntu2404 ~]#ls 
consul_1.20.4_linux_amd64.zip 
[root@ubuntu2404 ~]#unzip consul_1.20.4_linux_amd64.zip -d /usr/local/bin/
Archive:  consul_1.20.4_linux_amd64.zip
  inflating: /usr/local/bin/LICENSE.txt  
  inflating: /usr/local/bin/consul   
[root@ubuntu2404 ~]#ls /usr/local/bin/
consul  LICENSE.txt  prometheus  promtool  pushgateway
[root@ubuntu2404 ~]#ldd /usr/local/bin/consul 
        not a dynamic executable
#Bash可能缓存了/usr/bin/consul，导致它一直尝试在 /usr/bin/ 下找 consul。清除缓存：
[root@ubuntu2404 ~]#hash -r
[root@ubuntu2404 ~]#consul version
Consul v1.20.4
Revision 9e308779
Build Date 2025-02-20T12:49:28Z
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
#实现consul命令自动补全
[root@ubuntu2404 ~]#consul -autocomplete-install

#创建用户
[root@ubuntu2404 ~]#useradd -s /sbin/nologin consul
useradd: user 'consul' already exists
[root@ubuntu2404 ~]#id consul 
uid=996(consul) gid=988(consul) groups=988(consul)

#创建目录
[root@ubuntu2404 ~]#mkdir -p /data/consul /etc/consul.d
[root@ubuntu2404 ~]#chown -R consul:consul /data/consul /etc/consul.d

#以server模式启动服务cosnul agent
[root@ubuntu2404 ~]#/usr/local/bin/consul agent -server -ui -bootstrap-expect=1 -data-dir=/data/consul -node=consul -client=0.0.0.0 -config-dir=/etc/consul.d

-server  			#定义agent运行在server模式
-bootstrap-expect 	#在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定server数目的时候才会引导整个集群，该标记不能和bootstrap共用
-bind：		#该地址用来在集群内部的通讯，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0
-node：		#节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名
-ui    		#提供web ui的http功能
-rejoin 	#使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中。
-config-dir #配置文件目录，里面所有以.json结尾的文件都会被加载
-client  	#consul服务侦听地址，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1:8500，要对外提供服务改成0.0.0.0
```

```bash
#创建service文件
[root@ubuntu2404 ~]#vim /lib/systemd/system/consul.service 
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent -server -bind=10.0.0.200 -ui -bootstrap-expect=1 -data-dir=/data/consul -node=consul -client=0.0.0.0 -config-dir=/etc/consul.d
#ExecReload=/bin/kill --signal HUP \$MAINPID
KillMode=process
KillSignal=SIGTERM
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target


[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl enable --now consul.service 
```

#### 脚本安装

```bash
[root@ubuntu2404 ~]#cat install_consul.sh 
#!/bin/bash
#
#支持在线和离线下载安装

CONSUL_VERSION=1.20.4
#CONSUL_VERSION=1.20.1
#CONSUL_VERSION=1.19.2
#CONSUL_VERSION=1.19.0
#CONSUL_VERSION=1.18.0
#CONSUL_VERSION=1.15.0
#CONSUL_VERSION=1.13.3
#CONSUL_VERSION=1.10.2

CONSUL_FILE=consul_${CONSUL_VERSION}_linux_amd64.zip

CONSUL_URL=https://releases.hashicorp.com/consul/${CONSUL_VERSION}/${CONSUL_FILE}

CONSUL_DATA=/data/consul

LOCAL_IP=`hostname -I|awk '{print $1}'`


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

install_consul () {

    if [ ! -f  ${CONSUL_FILE} ] ;then
        wget  ${CONSUL_URL}  ||  { color "下载失败!" 1 ; exit ; }
    fi

    unzip ${CONSUL_FILE} -d /usr/local/bin/

    useradd -s /sbin/nologin consul

    mkdir -p ${CONSUL_DATA} /etc/consul.d

    chown -R consul:consul ${CONSUL_DATA} /etc/consul.d

}

service_consul () {

cat <<EOF > /lib/systemd/system/consul.service
[Unit]
Description="HashiCorp Consul - A service mesh solution"
Documentation=https://www.consul.io/
Requires=network-online.target
After=network-online.target

[Service]
Type=simple
User=consul
Group=consul
ExecStart=/usr/local/bin/consul agent -server -ui -bootstrap-expect=1 -data-dir=${CONSUL_DATA} -node=consul -bind=${LOCAL_IP} -client=0.0.0.0 -config-dir=/etc/consul.d
ExecReload=/bin/kill --signal HUP \$MAINPID
KillMode=process
KillSignal=SIGTERM
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

    systemctl daemon-reload
    systemctl enable --now consul.service

}


start_consul() { 

    systemctl is-active consul

    if [ $?  -eq 0 ];then  
        echo 
        color "Consul 安装完成!" 0
        echo "-------------------------------------------------------------------"
        echo -e "访问链接: \c"
        msg_info "http://${LOCAL_IP}:8500/" 
    else
        color "Consul 安装失败!" 1
        exit
    fi 
}



install_consul

service_consul

start_consul

```

Docker compose 部署 consul

```bash
version: '3.6'

volumes:
  consul_data: {}

networks:
  monitoring:
    driver: bridge

services:
  consul:
    image: consul:1.14
    volumes:
      - ./consul_configs:/consul/config
      - consul_data:/consul/data/
    networks:
      - monitoring
    ports:
      - 8500:8500
    command: 
      - "consul"
      - "agent"
      - "-dev"
      - "-bootstrap"
      - "-config-dir"
      - "/consul/config"
      - "-data-dir"
      - "/consul/data"
      - "-ui"
      - "-log-level"
      - "INFO"
      - "-bind"
      - "127.0.0.1"
      - "-client"
      - "0.0.0.0"
```

```json
#准备配置文件
{
  "services": [
    {
      "id": "node_exporter-prometheus",
      "name": "prometheus-server",
      "address": "prometheus.kang.org",
      "port": 9100,
      "tags": ["nodes"],
      "checks": [
        {
          "http": "http://prometheus.kang.org:9100/metrics",
          "interval": "5s"
        }
      ]
    },
    {
      "id": "node_exporter-node01",
      "name": "server01.kang.org",
      "address": "server01.kang.org",
      "port": 9100,
      "tags": ["nodes"],
      "checks": [
        {
          "http": "http://server01.kang.org:9100/metrics",
          "interval": "5s"
        }
      ]
    },
    {
      "id": "node_exporter-node02",
      "name": "server02.kang.org",
      "address": "server02.kang.org",
      "port": 9100,
      "tags": ["nodes"],
      "checks": [
        {
          "http": "http://server02.kang.org:9100/metrics",
          "interval": "5s"
        }
      ]
    },
    {
      "id": "node_exporter-node03",
      "name": "server03.kang.org",
      "address": "server03.kang.org",
      "port": 9100,
      "tags": ["nodes"],
      "checks": [
        {
          "http": "http://server03.kang.org:9100/metrics",
          "interval": "5s"
        }
      ]
    }
  ]
}

```

#### **Consul** **自动注册和删除服务**

```bash
#列出数据中心
curl http://10.0.0.200:8500/v1/catalog/datacenters

#列出节点
curl http://10.0.0.200:8500/v1/catalog/nodes

#列出服务
curl http://10.0.0.200:8500/v1/catalog/services

#指定节点状态
curl http://10.0.0.200:8500/v1/health/node/node2

#列出服务节点
curl http://10.0.0.200:8500/v1/catalog/service/<service_id>   

#提交Json格式的数据进行注册服务
[root@ubuntu2404 ~]#curl -X PUT -d '{"id": "myservice-id","name": "myservice","address": "10.0.0.201","port": 9100,"tags": ["service"],"checks": [{"http": "http://10.0.0.201:9100/","interval": "5s"}]}' http://10.0.0.200:8500/v1/agent/service/register

[root@ubuntu2404 ~]#curl http://10.0.0.200:8500/v1/catalog/services
{"consul":[],"myservice":["service"]}

#也可以将注册信息保存在json格式的文件中，再执行下面命令注册
[root@ubuntu2404 ~]#vim nodes.json 
{
  "id": "myservice-id",
  "name": "myservice2",
  "address": "10.0.0.201",
  "port": 9100,
  "tags": [
    "service"
  ],
  "checks": [
    {
      "http": "http://10.0.0.201:9100/",
      "interval": "5s"
    }
  ]
}
[root@ubuntu2404 ~]#curl -X PUT --data @nodes.json http://10.0.0.200:8500/v1/agent/service2/register

#删除服务，注意：集群模式下需要在service_id所有在主机节点上执行才能删除该service
[root@ubuntu2404 ~]#curl -X PUT http://10.0.0.200:8500/v1/agent/service/deregister/myservice-id
```

```json
[root@ubuntu2404 ~]#cat service.json 
{
  "services": [
    {
      "id": "myservice1-id",
      "name": "myservice1",
      "address": "10.0.0.200",
      "port": 9100,
      "tags": ["node_exporter"],
      "checks": [
        {
          "http": "http://10.0.0.200:9100/metrics",
          "interval": "5s"
        }
      ]
    },
    {
      "id": "myservice2-id",
      "name": "myservice2",
      "address": "10.0.0.201",
      "port": 9100,
      "tags": ["node_exporter"],
      "checks": [
        {
          "http": "http://10.0.0.201:9100/metrics",
          "interval": "5s"
        }
      ]
    }
  ]
}
```

```bash
#注册服务
[root@ubuntu2404 ~]#consul services register service.json
Node name "ubuntu2404.wang.org" will not be discoverable via DNS due to invalid characters. Valid characters include all alpha-numerics and dashes.
Registered service: myservice1
Registered service: myservice2
#注销服务
[root@ubuntu2404 ~]#consul services deregister -id myservice2-id
Deregistered service: myservice2-id
```





#### **配置** **Prometheus** **使用** **Consul** **服务发现**

 默认情况下，当Prometheus加载Target实例完成后，这些Target时候都会包含一些默认的标签：

```bash
__address__ 		#当前Target实例的访问地址<host>:<port>
__scheme__ 			#采集目标服务访问地址的HTTP Scheme，HTTP或者HTTPS
__metrics_path__ 	#采集目标服务访问地址的访问路径
__param_<name> 		#采集任务目标服务的中包含的请求参数
```

通过Consul动态发现的服务实例还会包含以下Metadata标签信息：

```bash
__meta_consul_address         #consul地址
__meta_consul_dc              #consul中服务所在的数据中心
__meta_consulmetadata         #服务的metadata
__meta_consul_node            #服务所在consul节点的信息
__meta_consul_service_address #服务访问地址
__meta_consul_service_id      #服务ID
__meta_consul_service_port    #服务端口
__meta_consul_service         #服务名称
__meta_consul_tags            #服务包含的标签信息
```

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml

  - job_name: 'consul'
    honor_labels: true  # 如果标签冲突，覆盖 Prometheus 添加的标签，保留原标签
    consul_sd_configs:
      - server: '10.0.0.200:8500'
        services: []  # 指定需要发现的 service 名称，默认为所有 service
        # tags:       # 可以过滤具有指定 tag 的 service
        # - "service"
        # refresh_interval: 2m  # 刷新时间间隔，默认 30s

    relabel_configs:
      - source_labels: ['__meta_consul_service']  # 生成新的标签名
        target_label: 'consul_service'
      - source_labels: ['__meta_consul_dc']  # 生成新的标签名
        target_label: 'datacenter'
      - source_labels: ['__meta_consul_tags']  # 生成新的标签名
        target_label: 'app'
      - source_labels: ['__meta_consul_service']  # 删除 consul 内置 service（不提供 metrics）
        regex: "consul"
        action: drop
 
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```



#### 集群部署

![image-20250306173545263](5day-png/34consul集群部署.png)

```bash
-server    	#使用server 模式运行consul 服务,consul支持以server或client的模式运行, server是服务发现模块的核心, client主要用于转发请求
-bootstrap 	#首次部署使用初始化模式
-bostrap-expect 2 #集群至少两台服务器，才能选举集群leader,默认值为3
-bind 		#该地址用来在集群内部的通讯，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0,有多个IP需要手动指定,否则可能会出错
-client 	#设置客户端访问的监听地址,此地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1
-data-dir 	#指定数据保存路径
-ui 		#运行 web 控制台,监听8500/tcp端口
-node 		#此节点的名称,群集中必须唯一
-datacenter=dc1 	#数据中心名称，默认是dc1
-retry-join 		#指定要加入的集群中已有consul节点的地址，失败会重试, 可多次指定不同的地址,代替旧版本中的-join选项
```

范例:

```bash
#node1:
consul agent -server -bind=10.0.0.200 -client=0.0.0.0 -data-dir=/data/consul -node=node1 -ui -bootstrap

#node2:
consul agent -server -bind=10.0.0.201 -client=0.0.0.0 -data-dir=/data/consul -node=node2 -retry-join=10.0.0.200 -ui -bootstrap-expect 2

#node3:
consul agent -server -bind=10.0.0.202 -client=0.0.0.0 -data-dir=/data/consul -node=node3 -retry-join=10.0.0.200 -ui -bootstrap-expect 2
```

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml

  - job_name: 'consul'
    honor_labels: true  # 如果标签冲突，覆盖 Prometheus 添加的标签，保留原标签
    consul_sd_configs:
      - server: '10.0.0.200:8500'
        services: []  # 指定需要发现的 service 名称，默认为所有 service
        # tags:       # 可以过滤具有指定 tag 的 service
        # - "service"
        # refresh_interval: 2m  # 刷新时间间隔，默认 30s
      - server: '10.0.0.201:8500'  # 添加其它节点，实现冗余
      - server: '10.0.0.202:8500'  # 添加其它节点，实现冗余

    relabel_configs:
      - source_labels: ['__meta_consul_service']  # 生成新的标签名
        target_label: 'consul_service'
      - source_labels: ['__meta_consul_dc']  # 生成新的标签名
        target_label: 'datacenter'
      - source_labels: ['__meta_consul_tags']  # 生成新的标签名
        target_label: 'app'
      - source_labels: ['__meta_consul_service']  # 删除 consul 内置 service（不提供 metrics）
        regex: "consul"
        action: drop
 
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

## 各种 Exporter

```
https://prometheus.io/docs/instrumenting/exporters/
```



### Node Exporter 监控服务

```
[root@ubuntu2404 ~]#/usr/local/node_exporter/bin/node_exporter --help

--collector.systemd               #显示当前系统中所有的服务状态信息
--collector.systemd.unit-include  #仅仅显示符合条件的systemd服务条目
--collector.systemd.unit-exclude  #显示排除列表范围之外的服务条目

#注意：
上面三条仅显示已安装的服务条目，没有安装的服务条目是不会被显示的。
而且后面两个属性是依赖于第一条属性的
这些信息会被显示在 node_systemd_unit_state对应的metrics中
```

**修改** **node_exporter** **的配置文件**

```bash
[root@ubuntu2404 ~]#cat /lib/systemd/system/node_exporter.service
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/node_exporter/bin/node_exporter --collector.systemd --collector.systemd.unit-include=".*(ssh|mysql|node_exporter|nginx).*"
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target

#属性解析：
如果没有提前安装的服务，是不会被查看到的
服务名称的正则符号必须解析正确，否则无法匹配要现实的服务名称
[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl restart node_exporter.service 
```

**修改** **Prometheus** **配置**

```bash
#修改prometheus的配置文件，让它自动过滤文件中的节点信息
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
  - job_name: "node_exporter"
    static_configs:
      - targets:
        - "10.0.0.201:9100"
        - "10.0.0.200:9100"
...

[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```



### MySQL 监控

```bash
https://github.com/prometheus/mysqld_exporter
```

**案例：二进制安装**

```bash
#安装mysql数据库
[root@ubuntu2404 ~]#apt update && apt install mysql-server -y

#更新mysql配置，如果MySQL和MySQL exporter 不在同一个主机，需要修改如下配置
sed -i 's#127.0.0.1#0.0.0.0#' /etc/mysql/mysql.conf.d/mysqld.cnf
systemctl restart mysql

#为mysqld_exporter配置获取数据库信息的用户并授权
mysql> CREATE USER 'exporter'@'10.0.0.200' IDENTIFIED BY '123456';
mysql> GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'10.0.0.200';
mysql> flush privileges;


#二进制安装mysqld_exporter
[root@ubuntu2404 ~]#ls
mysqld_exporter-0.17.2.linux-amd64.tar.gz
[root@ubuntu2404 ~]#tar xfv mysqld_exporter-0.17.2.linux-amd64.tar.gz -C /usr/local/
mysqld_exporter-0.17.2.freebsd-amd64/
mysqld_exporter-0.17.2.freebsd-amd64/LICENSE
mysqld_exporter-0.17.2.freebsd-amd64/mysqld_exporter
mysqld_exporter-0.17.2.freebsd-amd64/NOTICE
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ln -s mysqld_exporter-0.17.2.linux-amd64/ mysqld_exporter
[root@ubuntu2404 local]#cd mysqld_exporter
[root@ubuntu2404 mysqld_exporter]#ls
LICENSE  mysqld_exporter  NOTICE
[root@ubuntu2404 mysqld_exporter]#mkdir bin
[root@ubuntu2404 mysqld_exporter]#mv mysqld_exporter bin/
#用service启动
id prometheus &> /dev/null || useradd -r -s /sbin/nologin prometheus
#在mysqld_exporter的服务目录下，创建 .my.cnf 隐藏文件，为mysqld_exporter配置获取数据库信息的基本属性
[root@ubuntu2404 ~]#cat /usr/local/mysqld_exporter/.my.cnf
[client]
host=10.0.0.200
port=3306
user=exporter
password=123456

#创建service文件

[root@ubuntu2404 ~]#vim /lib/systemd/system/mysqld_exporter.service
[Unit]
Description=mysqld exporter project
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/mysqld_exporter/bin/mysqld_exporter --config.my-cnf="/usr/local/mysqld_exporter/.my.cnf"
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target


systemctl daemon-reload
systemctl enable --now mysqld_exporter.service
systemctl status mysqld_exporter.service

[root@ubuntu2404 mysqld_exporter]#ss -tnulp | grep 9104
tcp   LISTEN 0      4096                                 *:9104             *:*    users:(("mysqld_exporter",pid=19619,fd=3))   
```

**修改** **Prometheus** **配置**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
scrape_configs:
...
  - job_name: "mysql_exporter"
    static_configs:
      - targets: 
        - "10.0.0.200:9104"
        
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

**Grafana** **图形展示**

17320 中文版

**docker**

**1. 目录结构**

```
[root@ubuntu2204 mysql]# tree
.
├── docker-compose.yml  # docker-compose 配置文件
└── mysql
    └── docker.cnf  # MySQL 自定义配置
```

------

**2. MySQL 配置**

**2.1 自定义 MySQL 配置**

```ini
[mysqld]
skip-host-cache
skip-name-resolve
```

- 关闭主机缓存 (`skip-host-cache`)
- 关闭 DNS 解析 (`skip-name-resolve`)，加快访问速度

------

**3. `docker-compose.yml` 配置**

```yaml
version: '3.6'
volumes:
  mysqld_data: {}

networks:
  monitoring:
    driver: bridge
    ipam:
      config:
        - subnet: 172.31.0.0/24

services:
  mysqld:
    image: mysql:8.0  # 也可以使用 mysql:5.7
    volumes:
      - ./mysql:/etc/mysql/conf.d  # 挂载 MySQL 配置目录
      - mysqld_data:/var/lib/mysql  # 持久化 MySQL 数据
    environment:
      - MYSQL_ALLOW_EMPTY_PASSWORD=true  # 允许空密码
    networks:
      - monitoring
    ports:
      - 3306:3306  # 映射 MySQL 端口

  mysqld-exporter:
    image: prom/mysqld-exporter:v0.14.0
    command:
      - --collect.info_schema.innodb_metrics
      - --collect.info_schema.innodb_tablespaces
      - --collect.perf_schema.eventsstatementssum
      - --collect.perf_schema.memory_events
      - --collect.global_status
      - --collect.engine_innodb_status
      - --collect.binlog_size
    environment:
      - DATA_SOURCE_NAME=exporter:exporter@(mysqld:3306)/  # 连接 MySQL
    ports:
      - 9104:9104  # mysqld-exporter 监听端口
    networks:
      - monitoring
    depends_on:
      - mysqld  # 依赖 MySQL 启动
```

------

**4. 启动 MySQL 及 Exporter**

```bash
docker-compose up -d
```

- `-d` 让容器在后台运行

------

**5. 验证容器状态**

```bash
docker-compose ps
```

------

**6. 检查 Exporter 日志**

```bash
docker-compose logs mysqld-exporter
```

------

**7. 创建 Exporter 用户**

```bash
docker-compose exec mysqld /bin/sh
```

进入容器后：

```sql
mysql
CREATE USER 'exporter'@'172.31.%.%' IDENTIFIED BY 'exporter';
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'172.31.%.%';
GRANT SELECT ON performance_schema.* TO 'exporter'@'172.31.%.%';
\q
exit
```

------

**8. 测试 Exporter 指标输出**

```bash
curl http://localhost:9104/metrics
```

如果输出 Prometheus 格式的监控数据，说明 `mysqld-exporter` 运行正常。

------

**9. 修改 Prometheus 监控配置**

编辑 `prometheus.yml`：

```yaml
- job_name: "mysqld-exporter"
  static_configs:
    - targets: ["mysqld-exporter服务器地址:9104"]
```

> **注意**：`mysqld-exporter服务器地址` 需改为实际 IP 或主机名。

------

**10. 重新加载 Prometheus**

```bash
systemctl reload prometheus
```



### Java 应用监控

```
https://github.com/prometheus/jmx_exporter

https://github.com/prometheus/jmx_exporter/blob/main/examples/tomcat.yml
```

```bash
#安装tomcat
[root@ubuntu2404 local]#apt update && apt install tomcat10 -y
```

```bash
#二进制安装jmx
[root@ubuntu2404 ~]#ls
tomcat.yml		
jmx_prometheus_javaagent-1.1.0.jar

[root@ubuntu2404 ~]#mv jmx_prometheus_javaagent-1.1.0.jar /usr/share/tomcat10/lib/
[root@ubuntu2404 ~]#mv tomcat.yml /usr/share/tomcat10/etc/
[root@ubuntu2404 ~]#vim /usr/share/tomcat10/bin/catalina.sh
#添加启动代码
....
# -----------------------------------------------------------------------------

JAVA_OPTS="-javaagent:/usr/share/tomcat10/lib/jmx_prometheus_javaagent-1.1.0.jar=9527:/usr/share/tomcat10/etc/tomcat.yml"

# OS specific support.  $var _must_ be set to either true or false.
...
```

```bash
[root@ubuntu2404 ~]#systemctl restart tomcat10.service 
[root@ubuntu2404 ~]#ps aux | grep jmx
tomcat     22029 14.1  4.8 3629536 192432 ?      Ssl  19:58   0:04 /usr/lib/jvm/default-java/bin/java -Djava.util.logging.config.file=/var/lib/tomcat10/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -javaagent:/usr/share/tomcat10/lib/jmx_prometheus_javaagent-1.1.0.jar=9527:/usr/share/tomcat10/etc/tomcat.yml -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED -classpath /usr/share/tomcat10/bin/bootstrap.jar:/usr/share/tomcat10/bin/tomcat-juli.jar -Dcatalina.base=/var/lib/tomcat10 -Dcatalina.home=/usr/share/tomcat10 -Djava.io.tmpdir=/tmp org.apache.catalina.startup.Bootstrap start
```

**修改** **Prometheus** **配置**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
scrape_configs:
...
  - job_name: "jmx_exporter"
    static_configs:
      - targets: 
        - "10.0.0.200:9527"
        
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

**Grafana** **图形展示**

```
https://grafana.com/grafana/dashboards/9544-nifi-jmx-exporter/
```

**docker compose**

**目录结构**

```
tomcat-metrics/
├── docker-compose.yml   # Docker Compose 配置
└── tomcat/
    ├── context.xml      # Tomcat Manager 应用的 Context 配置
    ├── Dockerfile       # 构建 Tomcat 监控镜像
    ├── sources.list     # Debian 适用于 Tomcat 容器的源
    └── tomcat-users.xml # Tomcat 用户权限配置
```

------

**1. `docker-compose.yml`**

```yaml
version: '3.6'

volumes:
  tomcat_webapps: {}

networks:
  monitoring:
    driver: bridge
    ipam:
      config:
        - subnet: 172.31.0.0/24

services:
  tomcat:
    build:
      context: tomcat
      dockerfile: Dockerfile
    hostname: tomcat.wang.org
    expose:
      - 8080
    ports:
      - 8080:8080
    volumes:
      - tomcat_webapps:/usr/local/tomcat/webapps
      - ./tomcat/tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.xml
    networks:
      - monitoring
    environment:
      TZ: Asia/Shanghai
```

- 通过 `Dockerfile` 构建 `tomcat` 服务。
- 监听 `8080` 端口，挂载 `tomcat-users.xml` 。
- 使用 `monitoring` 网络，子网 `172.31.0.0/24`。

------

**2. `tomcat/context.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context antiResourceLocking="false" privileged="true">
    <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                     sameSiteCookies="strict"/>
    <Manager sessionAttributeValueClassNameFilter="java\.lang\.
        (?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.
        CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```

- **`privileged="true"`**：允许 Tomcat 运行管理 Web 应用（Manager）。
- **`sameSiteCookies="strict"`**：增强 Cookie 保护。
- **`Manager` 过滤配置**：防止反序列化攻击。

------

**3. `tomcat/Dockerfile`**

```dockerfile
FROM tomcat:9.0-jdk17-openjdk-slim

# 替换 Debian 源
ADD ./sources.list /etc/apt/sources.list

ENV TOMCAT_SIMPLECLIENT_VERSION=0.12.0
ENV TOMCAT_EXPORTER_VERSION=0.0.15

RUN apt-get update && apt-get install -y curl && \
    curl -L -o /usr/local/tomcat/lib/simpleclient-${TOMCAT_SIMPLECLIENT_VERSION}.jar \
         https://search.maven.org/remotecontent?filepath=io/prometheus/simpleclient/${TOMCAT_SIMPLECLIENT_VERSION}/simpleclient-${TOMCAT_SIMPLECLIENT_VERSION}.jar && \
    curl -L -o /usr/local/tomcat/lib/simpleclient_common-${TOMCAT_SIMPLECLIENT_VERSION}.jar \
         https://search.maven.org/remotecontent?filepath=io/prometheus/simpleclient_common/${TOMCAT_SIMPLECLIENT_VERSION}/simpleclient_common-${TOMCAT_SIMPLECLIENT_VERSION}.jar && \
    curl -L -o /usr/local/tomcat/lib/simpleclient_hotspot-${TOMCAT_SIMPLECLIENT_VERSION}.jar \
         https://search.maven.org/remotecontent?filepath=io/prometheus/simpleclient_hotspot/${TOMCAT_SIMPLECLIENT_VERSION}/simpleclient_hotspot-${TOMCAT_SIMPLECLIENT_VERSION}.jar && \
    curl -L -o /usr/local/tomcat/lib/simpleclient_servlet-${TOMCAT_SIMPLECLIENT_VERSION}.jar \
         https://search.maven.org/remotecontent?filepath=io/prometheus/simpleclient_servlet/${TOMCAT_SIMPLECLIENT_VERSION}/simpleclient_servlet-${TOMCAT_SIMPLECLIENT_VERSION}.jar && \
    curl -L -o /usr/local/tomcat/lib/simpleclient_servlet_common-${TOMCAT_SIMPLECLIENT_VERSION}.jar \
         https://search.maven.org/remotecontent?filepath=io/prometheus/simpleclient_servlet_common/${TOMCAT_SIMPLECLIENT_VERSION}/simpleclient_servlet_common-${TOMCAT_SIMPLECLIENT_VERSION}.jar && \
    curl -L -o /usr/local/tomcat/lib/tomcat_exporter_client-${TOMCAT_EXPORTER_VERSION}.jar \
         https://search.maven.org/remotecontent?filepath=nl/nlighten/tomcat_exporter_client/${TOMCAT_EXPORTER_VERSION}/tomcat_exporter_client-${TOMCAT_EXPORTER_VERSION}.jar && \
    curl -L -o /usr/local/tomcat/webapps/metrics.war \
         https://search.maven.org/remotecontent?filepath=nl/nlighten/tomcat_exporter_servlet/${TOMCAT_EXPORTER_VERSION}/tomcat_exporter_servlet-${TOMCAT_EXPORTER_VERSION}.war

RUN mv /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps/

ADD ./context.xml /usr/local/tomcat/webapps/manager/META-INF/context.xml
```

- 安装 `Prometheus` 相关 Jar 包，使 `Tomcat` 可被 `Prometheus` 监控。
- 部署 `metrics.war`，提供 `/metrics` 监控接口。
- 添加 `context.xml` 配置。

------

**4. `tomcat/sources.list`**

```bash
deb https://repo.huaweicloud.com/debian/ bullseye main non-free contrib
deb-src https://repo.huaweicloud.com/debian/ bullseye main non-free contrib
deb https://repo.huaweicloud.com/debian-security/ bullseye-security main
deb-src https://repo.huaweicloud.com/debian-security/ bullseye-security main
deb https://repo.huaweicloud.com/debian/ bullseye-updates main non-free contrib
deb-src https://repo.huaweicloud.com/debian/ bullseye-updates main non-free contrib
deb https://repo.huaweicloud.com/debian/ bullseye-backports main non-free contrib
deb-src https://repo.huaweicloud.com/debian/ bullseye-backports main non-free contrib
```

- 使用 **华为云镜像** 加速 `Debian` 软件包下载。

------

**5. `tomcat/tomcat-users.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <user username="tomcat" password="123456" roles="manager-gui,manager-script"/>
</tomcat-users>
```

- `manager-gui` 角色允许访问 **Tomcat Web 界面**。
- `manager-script` 角色允许访问 **Tomcat API**。

------

**启动容器**

```
cd tomcat-metrics
docker-compose up -d
```

- `-d`：后台运行。

------

**访问服务**

1. **访问 Tomcat**

```
http://localhost:8080
```

2. **访问 Tomcat Manager**

```
http://localhost:8080/manager/html
```

- **用户名**：`tomcat`
- **密码**：`123456`

3. **访问 Prometheus Metrics**

```
http://localhost:8080/metrics
```

- 这里是 **Prometheus 采集 `Tomcat` 指标的接口**。



### **Redis** **监控**

```
https://github.com/oliver006/redis_exporter
```

```bash
#redis安装
[root@ubuntu2404 ~]#apt update && apt install redis -y
[root@ubuntu2404 ~]#vim /etc/redis/redis.conf
bind 0.0.0.0
requirepass 123456

[root@ubuntu2404 ~]#systemctl restart redis
```

```bash
[root@ubuntu2404 ~]#ls 
redis_exporter-v1.67.0.linux-amd64.tar.gz
[root@ubuntu2404 ~]#tar xvf redis_exporter-v1.67.0.linux-amd64.tar.gz -C /usr/local/
redis_exporter-v1.67.0.linux-amd64/
redis_exporter-v1.67.0.linux-amd64/LICENSE
redis_exporter-v1.67.0.linux-amd64/README.md
redis_exporter-v1.67.0.linux-amd64/redis_exporter
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ln -s redis_exporter-v1.67.0.linux-amd64/ redis_exporter
[root@ubuntu2404 local]#cd redis_exporter/
[root@ubuntu2404 redis_exporter]#ls
LICENSE  README.md  redis_exporter
[root@ubuntu2404 redis_exporter]#mkdir bin
[root@ubuntu2404 redis_exporter]#mv redis_exporter bin/
#用service启动
id prometheus &> /dev/null || useradd -r -s /sbin/nologin prometheus
#创建service文件
vim /lib/systemd/system/redis_exporter.service
[Unit]
Description=redis exporter project
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/redis_exporter/bin/redis_exporter -redis.addr redis://localhost:6379 -redis.password 123456
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target


[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl enable --now redis_exporter.service 
Created symlink /etc/systemd/system/multi-user.target.wants/redis_exporter.service → /usr/lib/systemd/system/redis_exporter.service.
[root@ubuntu2404 ~]#systemctl status redis_exporter.service 
[root@ubuntu2404 ~]#ss -tnulp | grep 9121
tcp   LISTEN 0      4096                                 *:9121             *:*    users:(("redis_exporter",pid=23208,fd=3))
```



**修改 prometheus 配置**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
  - job_name: "redis_exporter"
    static_configs:
      - targets: 
        - "10.0.0.200:9121"
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

**grafana展示**

763

14615





### **Nginx** **监控**

Nginx 默认自身没有提供 Json 格式的指标数据,可以通过下两种方式实现 Prometheus 监控

方法1：

通过容器方式 nginx/nginx-prometheus-exporter容器配合nginx的stub状态页实现nginx的监控

方法2：Prometheus metric library for Nginx

```bash
https://prometheus.io/docs/instrumenting/exporters/
https://github.com/knyar/nginx-lua-prometheus
```

方法3

需要先编译安装一个模块nginx-vts,将状态页转换为Json格式

再利用nginx-vts-exporter采集数据到Prometheus



#### **nginx-prometheus-exporter** **容器实现**

```
https://hub.docker.com/r/nginx/nginx-prometheus-exporter
```

基于 docker 实现

```
https://hub.docker.com/r/nginx/nginx-prometheus-exporter
```

```bash
[root@ubuntu2404 ~]#apt update && apt install -y docker.io nginx
[root@ubuntu2404 ~]#vim /etc/docker/daemon.json
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1panel.live"
    ]
}

[root@ubuntu2404 ~]#vim /etc/nginx/conf.d/status.conf
server {
    listen 8888;
    location /basic_status {
        stub_status;
    }
}
[root@ubuntu2404 ~]#nginx -s reload
[root@ubuntu2404 ~]#docker pull nginx/nginx-prometheus-exporter:1.4
[root@ubuntu2404 ~]#curl 127.0.0.1:8888/basic_status
Active connections: 1 
server accepts handled requests
 1 1 1 
Reading: 0 Writing: 1 Waiting: 0 


[root@ubuntu2404 ~]#docker run -p 9113:9113 --name nginx-prometheus-exporter --restart always -d nginx/nginx-prometheus-exporter:1.4 --nginx.scrape-uri=http://localhost:8888/basic_status
```

**修改prometheus配置**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
  - job_name: "nginx_exporter"
    static_configs:
      - targets: 
        - "10.0.0.200:9113"
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

#### **nginx-vts-exporter** **实现**

**编译安装** **Nginx** **添加模块**

```
https://github.com/vozlt/nginx-module-vts
```

```bash
#编译安装nginx
[root@ubuntu2404 ~]#cat install_nginx.sh 
#!/bin/bash

NGINX_VERSION=1.22.1
#NGINX_VERSION=1.22.0
#NGINX_VERSION=1.20.2
#NGINX_VERSION=1.18.0
NGINX_FILE=nginx-${NGINX_VERSION}.tar.gz
NGINX_URL=http://nginx.org/download/
NGINX_INSTALL_DIR=/apps/nginx
SRC_DIR=/usr/local/src
CPUS=`lscpu |awk '/^CPU\(s\)/{print $2}'`

. /etc/os-release


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


check () {
    [ -e ${NGINX_INSTALL_DIR} ] && { color "nginx 已安装,请卸载后再安装" 1; exit; }
    cd  ${SRC_DIR}
    if [  -e ${NGINX_FILE}${TAR} ];then
        color "相关文件已准备好" 0
    else
        color '开始下载 nginx 源码包' 0
        wget ${NGINX_URL}${NGINX_FILE}${TAR} 
        [ $? -ne 0 ] && { color "下载 ${NGINX_FILE}${TAR}文件失败" 1; exit; } 
    fi
} 

install () {
    color "开始安装 nginx" 0
    if id nginx  &> /dev/null;then
        color "nginx 用户已存在" 1 
    else
        useradd -s /sbin/nologin -r  nginx
        color "创建 nginx 用户" 0 
    fi
    color "开始安装 nginx 依赖包" 0
    if [ $ID == "centos" ] ;then
            if [[ $VERSION_ID =~ ^7 ]];then
            yum -y  install  gcc  make pcre-devel openssl-devel zlib-devel perl-ExtUtils-Embed
                elif [[ $VERSION_ID =~ ^8 ]];then
            yum -y  install make gcc-c++ libtool pcre pcre-devel zlib zlib-devel openssl openssl-devel perl-ExtUtils-Embed 
                else 
            color '不支持此系统!'  1
            exit
        fi
     elif [ $ID == "rocky"  ];then
            yum -y  install gcc make gcc-c++ libtool pcre pcre-devel zlib zlib-devel openssl openssl-devel perl-ExtUtils-Embed 
     else
        apt update
        apt -y install gcc make  libpcre3 libpcre3-dev openssl libssl-dev zlib1g-dev
     fi
     [ $? -ne 0 ] && { color "安装依赖包失败" 1; exit; } 
     cd $SRC_DIR
     tar xf ${NGINX_FILE}
     NGINX_DIR=`echo ${NGINX_FILE}| sed -nr 's/^(.*[0-9]).*/\1/p'`
     cd ${NGINX_DIR}
     ./configure --prefix=${NGINX_INSTALL_DIR} --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module 
     make -j $CPUS && make install 
     [ $? -eq 0 ] && color "nginx 编译安装成功" 0 ||  { color "nginx 编译安装失败,退出!" 1 ;exit; }
         chown -R nginx.nginx ${NGINX_INSTALL_DIR}
     ln -s ${NGINX_INSTALL_DIR}/sbin/nginx /usr/local/sbin/nginx
     echo "PATH=${NGINX_INSTALL_DIR}/sbin:${PATH}" > /etc/profile.d/nginx.sh
     cat > /lib/systemd/system/nginx.service <<EOF
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=${NGINX_INSTALL_DIR}/logs/nginx.pid
ExecStartPre=/bin/rm -f ${NGINX_INSTALL_DIR}/logs/nginx.pid
ExecStartPre=${NGINX_INSTALL_DIR}/sbin/nginx -t
ExecStart=${NGINX_INSTALL_DIR}/sbin/nginx
ExecReload=/bin/kill -s HUP \$MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
     systemctl daemon-reload
     systemctl enable --now nginx &> /dev/null 
     systemctl is-active nginx &> /dev/null ||  { color "nginx 启动失败,退出!" 1 ; exit; }
     color "nginx 安装完成" 0
}

check

install
```

```bash
[root@ubuntu2404 ~]#cd /usr/local/src/nginx-1.22.1/
[root@ubuntu2404 nginx-1.22.1]#ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
[root@ubuntu2404 nginx-1.22.1]#nginx -V
nginx version: nginx/1.22.1
built by gcc 13.3.0 (Ubuntu 13.3.0-6ubuntu2~24.04) 
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module
[root@ubuntu2404 nginx-1.22.1]#./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=/root/nginx-module-vts-master
[root@ubuntu2404 nginx-1.22.1]#make && make install
```

```bash
[root@ubuntu2404 ~]#vim /apps/nginx/conf/nginx.conf
http {
	...
    vhost_traffic_status_zone;
	...
    server {
		...
        location /status {
            vhost_traffic_status_display;
            vhost_traffic_status_display_format html;
        }
        ...
[root@ubuntu2404 ~]#systemctl restart nginx.service 
[root@ubuntu2404 ~]#systemctl status nginx.service 
```

```bash
#浏览器访问如下
http://10.0.0.201/status

#访问下面可以看到Json格式
http://10.0.0.201/status/format/json
```



**安装** **nginx-vts-exporter**

```bash
https://github.com/sysulq/nginx-vts-exporter/releases
[root@ubuntu2404 ~]#tar xvf nginx-vtx-exporter_0.10.8_linux_amd64.tar.gz -C /usr/local/bin
```

```bash
#创建service文件
vim /lib/systemd/system/nginx_vts_exporter.service
[Unit]
Description=nginx_vts_exporter project
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/nginx-vtx-exporter -nginx.scrape_uri=http://localhost/status/format/json
Restart=on-failure
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target


[root@ubuntu2404 ~]#systemctl daemon-reload;systemctl enable --now nginx_vts_exporter
```

**修改prometheus配置文件**



```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
  - job_name: "nginx_vts_exporter"
    static_configs:
      - targets: 
        - "10.0.0.200:9913"
        
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

**grafana**

nginx-vts-exporter：2949

nginx-prometheus-exporter：14900 11199 11280



### **Consul** **监控**

```
 https://github.com/prometheus/consul_exporter
```

```bash
[root@ubuntu2404 ~]#ls 
consul_exporter-0.12.1.linux-amd64.tar.gz
[root@ubuntu2404 ~]#tar xf consul_exporter-0.12.1.linux-amd64.tar.gz -C /usr/local/
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ln -s consul_exporter-0.12.1.linux-amd64/ consul_exporter
[root@ubuntu2404 local]#cd consul_exporter
[root@ubuntu2404 consul_exporter]#ls
consul_exporter  LICENSE  NOTICE
[root@ubuntu2404 consul_exporter]#mkdir bin
[root@ubuntu2404 consul_exporter]#mv consul_exporter bin/
```

```bash
#用consul用户启动，创建consul用户
useradd -r consul
#编写service文件
vim /lib/systemd/system/consul_exporter.service
[Unit]
Description=consul_exporter
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
User=consul
EnvironmentFile=-/etc/default/consul_exporter
ExecStart=/usr/local/consul_exporter/bin/consul_exporter \
          --consul.server="http://localhost:8500" \
          --web.listen-address=":9107" \
          --web.telemetry-path="/metrics" \
          --log.level=info \
          $ARGS
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
Restart=always

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl enable --now consul_exporter.service 
[root@ubuntu2404 ~]#systemctl status consul_exporter.service
```

```
[root@ubuntu2404 ~]#ss -tnulp | grep 9107
tcp   LISTEN 0      4096                                         *:9107             *:*    users:(("consul_exporter",pid=2381,fd=3))   
```

**修改prometheus配置文件监控consul_exporter**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
  - job_name: "consul_exporter"
    static_configs:
      - targets:
        - "10.0.0.200:9107"
        
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

**Grafana** **展示**

12049



### **黑盒监控** **blackbox_exporter**

![image-20250308122635972](5day-png/34黑盒监控 blackbox_exporter.png)

黑盒监视也称远端探测，监测应用程序的外部，可以查询应用程序的外部特征

比如：是否开放相应的端口,并返回正确的数据或响应代码，执行icmp或者echo检查并确认收到响应

prometheus探测工具是通过运行一个blackbox exporter来探测远程目标，并公开在本地端点上

blackbox_exporter允许通过HTTP、HTTPS、DNS、TCP和ICMP等协议来探测端点状态

blackbox_exporter中，定义一系列执行特定检查的模块，例:检查正在运行的web服务器，或者DNS解析记录

blackbox_exporter运行时，它会在URL上公开这些模块和API

blackbox_exporter是一个二进制Go应用程序，默认监听端口9115

```
https://github.com/prometheus/blackbox_exporter
https://github.com/prometheus/blackbox_exporter/blob/master/blackbox.yml
https://github.com/prometheus/blackbox_exporter/blob/master/example.yml
```

```
https://prometheus.io/download/#blackbox_exporter
```

#### 二进制安装

```bash
[root@ubuntu2404 ~]#ls
blackbox_exporter-0.26.0.linux-amd64.tar.gz
[root@ubuntu2404 ~]#tar xf blackbox_exporter-0.26.0.linux-amd64.tar.gz -C /usr/local/
[root@ubuntu2404 ~]#cd /usr/local/
[root@ubuntu2404 local]#ln -s blackbox_exporter-0.26.0.linux-amd64/ blackbox_exporter
[root@ubuntu2404 local]#cd blackbox_exporter
[root@ubuntu2404 blackbox_exporter]#ls
blackbox_exporter  blackbox.yml  LICENSE  NOTICE
[root@ubuntu2404 blackbox_exporter]#mkdir bin conf
[root@ubuntu2404 blackbox_exporter]#mv blackbox_exporter bin/
[root@ubuntu2404 blackbox_exporter]#mv blackbox.yml conf/
[root@ubuntu2404 blackbox_exporter]#ls
bin  conf  LICENSE  NOTICE
```

```yaml
#配置文件
[root@ubuntu2404 blackbox_exporter]#cat conf/blackbox.yml 
modules:
  http_2xx:			#名字
    prober: http	#协议
    http:
      preferred_ip_protocol: "ip4"
  http_post_2xx:
    prober: http
    http:
      method: POST	#支持GET、POST、默认GET
  tcp_connect:
    prober: tcp
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false		#启用远程证书探测
  grpc:
    prober: grpc
    grpc:
      tls: true
      preferred_ip_protocol: "ip4"		#探测的ip协议版本
  grpc_plain:
    prober: grpc
    grpc:
      tls: false
      service: "service1"
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
      - send: "SSH-2.0-blackbox-ssh-check"
  ssh_banner_extract:
    prober: tcp
    timeout: 5s
    tcp:
      query_response:
      - expect: "^SSH-2.0-([^ -]+)(?: (.*))?$"
        labels:
        - name: ssh_version
          value: "${1}"
        - name: ssh_comments
          value: "${2}"
  irc_banner:
    prober: tcp
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
  icmp_ttl5:
    prober: icmp
    timeout: 5s
    icmp:
      ttl: 5
```

```bash
#创建service文件
[root@ubuntu2404 ~]#vim /lib/systemd/system/blackbox_exporter.service
[Unit]
Description=Prometheus Black Exporter
After=network.target
[Service]
Type=simple
#新版blackbox_exporter-0.26.0如果以普通用户启动，会导致探测失败
#User=prometheus
#Group=prometheus
ExecStart=/usr/local/blackbox_exporter/bin/blackbox_exporter --
config.file=/usr/local/blackbox_exporter/conf/blackbox.yml --web.listen-address=:9115
Restart=on-failure
LimitNOFILE=100000
[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl enable --now blackbox_exporter.service
[root@ubuntu2404 ~]#systemctl status blackbox_exporter.service

[root@ubuntu2404 ~]#ss -tnulp | grep 9115
tcp   LISTEN 0      4096                                         *:9115             *:*    users:(("blackbox_export",pid=1818,fd=3))  
```



#### ansible部署

```
https://github.com/prometheus-community/ansible/tree/main/roles/blackbox_exporter
```



#### docker容器启动

```bash
docker run --rm -d -p 9115:9115 -v pwd:/config prom/blackbox-exporter:master --config.file=/config/blackbox.yml
```



#### 网络连通性探测

**Prometheus** **配置定义监控规则**

```yaml
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
  # ping 检测
  - job_name: 'ping_status_blackbox_exporter'
    scrape_interval: 1m
    scrape_timeout: 10s
    metrics_path: /probe
    params:
      module: [icmp]  # 使用 ICMP (ping) 探测方式
    static_configs:
      - targets:		#探测的目标主机地址
          - '10.0.0.201'
          - 'www.google.com'
        labels:
          instance: 'ping_status'
          group: 'icmp'
    relabel_configs:
      - source_labels: [__address__]	#修改目标URL地址的标签[__address__]为__param_target,用于发送给blackbox使用
        target_label: __param_target  # 目标 URL 传递给 blackbox_exporter
      - target_label: __address__		#添加新标签.用于指定black_exporter服务器地址,此为必须项
        replacement: 10.0.0.200:9115  # 指定 blackbox_exporter 服务器地址
      - source_labels: [__param_target]	#Grafana 使用此标签进行显示，此值是固定的
        target_label: ipaddr  # Grafana 展示字段

[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

```bash
#访问：10.0.0.200:9115
Recent Probes
Module	Target	Result	Debug
icmp	www.google.com	Failure	Logs
icmp	10.0.0.201	Success	Logs
icmp	www.google.com	Failure	Logs
icmp	10.0.0.201	Success	Logs
```

```bash
#在201上禁止ping
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/icmp_echo_ignore_all 
0
[root@ubuntu2404 ~]#echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
```

```
Recent Probes
Module	Target	Result	Debug
icmp	10.0.0.201	Failure	Logs
icmp	www.google.com	Failure	Logs
```



#### TCP端口连通性探测

**Prometheus** **配置定义监控规则**

```yaml
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
# 端口监控
  - job_name: 'port_status_blackbox_exporter'
    scrape_interval: 1m
    scrape_timeout: 10s
    metrics_path: /probe
    params:
      module: [tcp_connect]  # 使用 TCP 连接探测
    static_configs:
      - targets:
          - '10.0.0.201:80'  # 目标地址，带端口
        labels:
          instance: 'port_status'
          group: 'port'
    relabel_configs:
      - source_labels: [__address__]    #修改目标URL地址的标签[__address__]为__param_target,用于发送给blackbox使用
        target_label: __param_target    # 目标 URL 传递给 blackbox_exporter
      - target_label: __address__               #添加新标签.用于指定black_exporter服务器地址,此为必须项
        replacement: 10.0.0.200:9115  # 指定 blackbox_exporter 服务器地址
      - source_labels: [__param_target] #Grafana 使用此标签进行显示，此值是固定的
        target_label: ipaddr_port  # Grafana 展示的字段

[root@ubuntu2404 ~]#promtool check config /usr/local/prometheus/conf/prometheus.yml
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```



#### **Http/Https** **网站监控**

**Prometheus** **配置定义监控规则**

```yaml
# 网站监控
  - job_name: 'http_status_blackbox_exporter'
    scrape_interval: 1m
    scrape_timeout: 10s
    metrics_path: /probe
    params:
      module: [http_2xx]  # 使用 HTTP 2xx 状态码探测
    static_configs:		#如果有https网站,还会自动额外显示TLS版本和证书的有效期,但有些网站可能不会显示证书信息,比如taobao
      - targets:
          - 'https://www.baidu.com'  # HTTPS 站点，支持 TLS 版本和证书有效期检测
          - 'http://www.163.com'  # HTTP 站点
        labels:		#添加标签
          instance: http_status
          group: web
    relabel_configs:
      - source_labels: [__address__]	#修改目标URL地址的标签[__address__]为__param_target,用于发送给blackbox使用
        target_label: __param_target  # 目标 URL 传递给 blackbox_exporter
      - target_label: __address__		#添加新标签,用于指定black_exporter服务器地址,此为必须项
        replacement: black-exporter.wang.org:9115  # 指定 blackbox_exporter 服务器地址
      - source_labels: [__param_target]		#Grafana 使用此url标签，此值是固定的，否则有些模板可能无法显示
        target_label: url  # Grafana 展示字段
      # - target_label: region  # 可选标签，例如标识不同区域的监控
      #   replacement: "remote"

```

```bash
[root@ubuntu2404 ~]#promtool check config /usr/local/prometheus/conf/prometheus.yml
[root@ubuntu2404 ~]#systemctl reload prometheus.service
```



#### **Grafana** **展示**

9965

13587



## **Prometheus** **实现容器监控** -cAdvisor

### 安装部署

```
#包下载
http://github.com/google/cadvisor

#容器下载,需用要google登录
gcr.io/cadvisor/cadvisor
```

#### 二进制方式部署

```bash
[root@ubuntu2404 ~]#ls
cadvisor-v0.51.0-linux-amd64
[root@ubuntu2404 ~]#install cadvisor-v0.51.0-linux-amd64 /usr/local/bin/cadvisor
[root@ubuntu2404 ~]#ldd /usr/local/bin/cadvisor
        not a dynamic executable
        
#编辑service文件
[root@ubuntu2404 ~]#vim /lib/systemd/system/cadvisor.service
[Unit]
Description=Prometheus cAdvisor Exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/cadvisor
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
#User=prometheus #不能以普通用户身份启动，权限不足
#Group=prometheus
Restart=on-failure
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl enable --now cadvisor.service 
```

```bash
[root@ubuntu2404 ~]#ss -tnulp | grep cad
tcp   LISTEN 0      4096                                         *:8080             *:*    users:(("cadvisor",pid=8741,fd=7))    
```

```
10.0.0.200:8080
```

**prometheus配置**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
...
  - job_name: "cadvisor_exporter"
    static_configs:
      - targets: 
        - "10.0.0.200:8080"
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```



#### docker方式部署cAdvisor

```bash
[root@ubuntu2404 ~]#docker pull \
    gcr.io/cadvisor/cadvisor:v0.49.2
```

```bash
VERSION=v0.49.2
[root@ubuntu2404 ~]#docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:v0.49.2
a72bdd52e20f963d907e2ecb6b2f7217ba30c876796223accee98d01597a19cd
```

#### **Ansible** **方式安装** **cAdvisor**

```
https://github.com/prometheus-community/ansible/tree/main/roles/cadvisor
```

### **常见指标**

```bash
container_tasks_state  #gauge类型，容器特定状态的任务数,根据不同的pod_name和state有600+的不同label.
container_memory_failures_total  #counter类型，内存分配失败的累积计数,根据不同的pod_name和
state有600+的不同label.
container_network_receive_errors_total  #counter类型，容器网络接收时遇到的累计错误数。
container_network_transmit_bytes_total  #counter类型，容器发送传输的累计字节数。
container_network_transmit_packets_dropped_total  #counter类型，容器传输时丢弃的累计包数
container_network_transmit_packets_total  #counter类型，传输数据包的累计计数
container_network_transmit_errors_total  #counter类型，传输时遇到的累积错误数
container_network_receive_bytes_total  #counter类型，收到的累计字节数
container_network_receive_packets_dropped_total  #counter类型，接收时丢弃的累计数据包数
container_network_receive_packets_total  #counter类型，收到的累计数据包数
container_spec_cpu_period  #gauge类型，容器的CPU period。
container_spec_memory_swap_limit_bytes  #容器swap内存交换限制字节
container_memory_failcnt  #counter类型，内存使用次数达到限制
container_spec_memory_reservation_limit_bytes  #容器规格内存预留限制字节
container_spec_cpu_shares  #gauge类型，
container_spec_memory_limit_bytes  #容器规格内存限制字节
container_memory_max_usage_bytes  #gauge类型，以字节为单位记录的最大内存使用量
container_cpu_load_average_10s  #gauge类型，最近10秒钟内的容器CPU平均负载值。
container_memory_rss  #gauge类型，容器RSS的大小（以字节为单位）
container_start_time_seconds  #gauge类型，从Unix纪元开始的容器开始时间（以秒为单位）。
container_memory_mapped_file  #gauge类型，内存映射文件的大小（以字节为单位）
container_cpu_user_seconds_total  #conter类型，累计CPU user 时间（以秒为单位）
container_memory_cache  #gauge类型，内存的cache字节数。
container_memory_working_set_bytes  #gague类型，当前工作集（以字节为单位）
container_cpu_system_seconds_total  #conter类型，累计CPU system时间（以秒为单位）
container_memory_swap  #gauge类型，容器交换使用量（以字节为单位）
container_memory_usage_bytes  #gauge类型，当前内存使用情况（以字节为单位），包括所有内存，无论何时访问
container_last_seen  #gauge类型，上一次export看到此容器的时间
container_fs_writes_total  #counter类型，累计写入次数
container_fs_reads_total   #counter类型，类型读取次数
container_cpu_usage_seconds_total  #counter类型，累计消耗CPU的总时间
container_fs_reads_bytes_total  #容器读取的总字节数
container_fs_writes_bytes_total  #容器写入的总字节数
container_fs_sector_reads_total  #counter类型，扇区已完成读取的累计计数
container_fs_inodes_free  #gauge类型，可用的Inode数量
container_fs_io_current  #gauge类型，当前正在进行的I/O数
container_fs_io_time_weighted_seconds_total  #counter类型，累积加权I/O时间（以秒为单位）
container_fs_usage_bytes  #gauge类型，此容器在文件系统上使用的字节数
container_fs_limit_bytes  #gauge类型，此容器文件系统上可以使用的字节数
container_fs_inodes_total  #gauge类型，inode数
container_fs_sector_writes_total  #counter类型，扇区写入累计计数
container_fs_io_time_seconds_total  #counter类型，I/O花费的秒数累计
container_fs_writes_merged_total  #counter类型，合并的累计写入数
container_fs_reads_merged_total  #counter类型，合并的累计读取数
container_fs_write_seconds_total  #counter类型，写花费的秒数累计
container_fs_read_seconds_total   #counter类型，读花费的秒数累计
container_cpu_cfs_periods_total  #counter类型，执行周期间隔时间数
container_cpu_cfs_throttled_periods_total  #counter类型，节流周期间隔数
container_cpu_cfs_throttled_seconds_total  #counter类型，容器被节流的总时间
container_spec_cpu_quota  #gauge类型，容器的CPU配额
machine_memory_bytes  #gauge类型，机器上安装的内存量
scrape_samples_post_metric_relabeling
cadvisor_version_info
scrape_duration_seconds
machine_cpu_cores  #gauge类型，机器上的CPU核心数
container_scrape_error  #gauge类型，如果获取容器指标时出错，则为1，否则为0
scrape_samples_scraped
```

**常见指标**

```
https://www.bookstack.cn/read/prometheus-book/exporter-use-prometheus-monitor-container.md
```

| 指标名称                               | 类型    | 含义                                         |
| :------------------------------------- | :------ | :------------------------------------------- |
| container_cpu_load_average_10s         | gauge   | 过去10秒容器CPU的平均负载                    |
| container_cpu_usage_seconds_total      | counter | 容器在每个CPU内核上的累积占用时间 (单位：秒) |
| container_cpu_system_seconds_total     | counter | System CPU累积占用时间（单位：秒）           |
| container_cpu_user_seconds_total       | counter | User CPU累积占用时间（单位：秒）             |
| container_fs_usage_bytes               | gauge   | 容器中文件系统的使用量(单位：字节)           |
| container_fs_limit_bytes               | gauge   | 容器可以使用的文件系统总量(单位：字节)       |
| container_fs_reads_bytes_total         | counter | 容器累积读取数据的总量(单位：字节)           |
| container_fs_writes_bytes_total        | counter | 容器累积写入数据的总量(单位：字节)           |
| container_memory_max_usage_bytes       | gauge   | 容器的最大内存使用量（单位：字节）           |
| container_memory_usage_bytes           | gauge   | 容器当前的内存使用量（单位：字节             |
| container_spec_memory_limit_bytes      | gauge   | 容器的内存使用量限制                         |
| machine_memory_bytes                   | gauge   | 当前主机的内存总量                           |
| container_network_receive_bytes_total  | counter | 容器网络累积接收数据总量（单位：字节）       |
| container_network_transmit_bytes_total | counter | 容器网络累积传输数据总量（单位：字节）       |

容器的CPU使用率：

```
sum(irate(container_cpu_usage_seconds_total{image!=""}[1m])) without (cpu)
#使用 name="<container_name>"表达 式选择指定名称的容器
```

每个容器的cpu使用率

```
sum(rate(container_cpu_usage_seconds_total{name=~".+"}[1m])) by (name) * 100
```

所用容器system cpu的累计使用时间（1min钟内）

```
sum(rate(container_cpu_system_seconds_total[1m]))
```

每个容器system cpu的使用时间（1min钟内）

```
sum(irate(container_cpu_system_seconds_total{image!=""}[1m])) without (cpu)
```

总容器的cpu使用率

```
sum(sum(rate(container_cpu_usage_seconds_total{name=~".+"}[1m])) by (name) * 100)
```

查询容器内存使用量（单位：字节）:

```
container_memory_usage_bytes{image!=""}
```

查询容器网络接收量速率（单位：字节/秒）：

```
sum(rate(container_network_receive_bytes_total{image!=""}[1m])) without (interface)
```

容器网络接收的字节数（1分钟内），根据名称查询name=~".+"

```
sum(rate(container_network_receive_bytes_total{name=~".+"}[1m])) by (name)
```

查询容器网络传输量速率（单位：字节/秒）：

```
sum(rate(container_network_transmit_bytes_total{image!=""}[1m])) without (interface)
```

查询容器文件系统读取速率（单位：字节/秒）：

```
sum(rate(container_fs_reads_bytes_total{image!=""}[1m])) without (device)
```

查询容器文件系统写入速率（单位：字节/秒）：

```
sum(rate(container_fs_writes_bytes_total{image!=""}[1m])) without (device)
```

### grafana展示

193

14282



## **Prometheus Federation** **联邦**

```
https://prometheus.io/docs/prometheus/latest/federation/
```

在生产环境中，一个Prometheus服务节点所能接管的主机数量有限。只使用一个prometheus节点，随着监控数据的持续增长，将会导致压力越来越大

可以采用prometheus的集群联邦模式，即在原有 Prometheus的Master 节点基础上,再部署多个prometheus的Slave 从节点，分别负责不同的监控数据采集，而Master节点只负责汇总数据与 Grafana 数据展示

联邦模式允许 Prometheus 服务器从另一个 Prometheus 服务器抓取特定数据。

联邦有不同的用例。 通常，它用于实现可扩展的Prometheus监控设置或将相关指标从一个服务的Prometheus拉到另一个服务。

联邦模式有分层联邦和跨服务联邦两种模式，分层联邦较为常用，且配置简单。

**分层联邦**

分层联合允许Prometheus扩展到具有数十个数据中心和数百万个节点的环境。 在此用例中，联合拓扑类似于树，较高级别的Prometheus服务器从较大数量的从属服务器收集聚合时间序列数据。

例如：设置可能包含许多高度详细收集数据的每个数据中心Prometheus服务器（实例级深入分析），以及一组仅收集和存储聚合数据的全局Prometheus服务器（作业级向下钻取） ）来自那些本地服务器。 这提供了聚合全局视图和详细的本地视图。

**跨服务联邦**

在跨服务联合中，一个服务的 Prometheus 服务器配置为从另一个服务的Prometheus服务器中提取所选数据，以便对单个服务器中的两个数据集启用警报和查询。

例如：运行多个服务的集群调度程序可能会暴露有关在集群上运行的服务实例的资源使用情况信息（如内存和CPU使用情况）。 另一方面，在该集群上运行的服务仅公开特定于应用程序的服务指标。 通常，这两组指标都是由单独的Prometheus服务器抓取的。 使用联合，包含服务级别度量标准的Prometheus服务器可以从群集Prometheus中提取有关其特定服务的群集资源使用情况度量标准，以便可以在该服务器中使用这两组度量标准。



针对每个集群，需要采集的主要指标类别包括：

- OS指标，例如节点资源（CPU, 内存，磁盘等）水位以及网络吞吐
- 元集群以及用户集群K8s master指标，例如kube-apiserver, kube-controller-manager, kubescheduler等指标
- K8s组件（kubernetes-state-metrics，cadvisor）采集的关于K8s集群状态
- etcd指标，例如etcd写磁盘时间，DB size，Peer之间吞吐量等等。

![image-20250308145505880](5day-png/34prometheus联邦.png)

| 编号 | 地址       | 角色                  |
| ---- | ---------- | --------------------- |
| 1    | 10.0.0.200 | Prometheus Master     |
| 2    | 10.0.0.201 | Prometheus Federation |
| 3    | 10.0.0.202 | Prometheus Federation |
| 4    | 10.0.0.101 | Node Exporter         |
| 5    | 10.0.0.102 | Node Exporter         |

**部署** **Prometheus** **主节点和联邦节点**

所有联邦节点和Prometheus的主节点安装方法是一样的

```
通过上面脚本安装prometheus
```

**部署** **Node Exporter** **节点**

在所有被监控的节点上安装 Node Exporter,安装方式一样

```
通过上面脚本安装node Exporter
```

**配置** **Prometheus** **联邦节点监控** **Node Exporter**

```bash
201
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml 
  - job_name: "node_exporter"
    static_configs:
      - targets: ["10.0.0.101:9100"]
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
202
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml 
  - job_name: "node_exporter"
    static_configs:
      - targets: ["10.0.0.102:9100"]
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

**配置** **Prometheus** **主节点管理** **Prometheus** **联邦节点**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
```

```yaml
  - job_name: 'federate-201'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'   #指定采集端点的路径,默认为/federate

    params:
      'match[]':
        - '{job="prometheus"}'      #指定只采集指定联邦节点的Job名称对应的数据,默认不指定不会采集任何联邦节点的采集的数据
        - '{job="node_exporter"}'   #指定采集联邦节点的job名称,和联邦节点配置的job_name必须匹配，如果不匹配则不会采集
        - '{__name__=~"job:.*"}'    #指定采集联邦节点的job名称,和联邦节点配置的job_name必须匹配，如果不匹配则不会采集
    static_configs:
      - targets:
        - '10.0.0.201:9090' #指定联邦节点prometheus节点地址,如果在k8s集群内,需要指定k8s的SVC的NodePort的地址信息
  - job_name: 'federate-202'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'   #指定采集端点的路径,默认为/federate

    params:
      'match[]':
        - '{job="prometheus"}'      #指定只采集指定联邦节点的Job名称对应的数据,默认不指定不会采集任何联邦节点的采集的数据
        - '{job="node_exporter"}'   #指定采集联邦节点的job名称,和联邦节点配置的job_name必须匹配，如果不匹配则不会采集
        - '{__name__=~"job:.*"}'    #指定采集联邦节点的job名称,和联邦节点配置的job_name必须匹配，如果不匹配则不会采集
        - '{__name__=~"node.*"}'    #指定从联邦节点中采集job_name以node开头的数据
    static_configs:
      - targets:
        - '10.0.0.202:9090' #指定联邦节点prometheus节点地址,如果在k8s集群内,需要指定k8s的SVC的NodePort的地址信息
                                                                                                                 
```

```bash
[root@ubuntu2404 ~]#promtool check config /usr/local/prometheus/conf/prometheus.yml 
[root@ubuntu2404 ~]#systemctl reload prometheus.service 
```

**Grafana展示Prometheus联邦**

8919



**联邦解决性能问题，不解决高可用**





## **Prometheus** **存储**

#### **Prometheus TSDB数据存储机制**

![image-20250308163004954](5day-png/34TSDB数据存储机制.png)

- 最新的数据是保存在内存中的，并同时写入至预写日志（WAL Write Ahead Log）
- 以每2小时为一个时间窗口，将内存中的数据存储为一个单独的 Block
- Block会被压缩及合并历史Block块，压缩合并后Block数量会减少
- Block的大小并不固定，但最小会保存2个小时的数据
- 后续生成的新数据保存在内存和WAL中，重复以上过程



<img src="5day-png/34TSDB数据存储机制1.png" alt="image-20250308163149763" style="zoom:50%;" />



**Prometheus** **数据目录结构**

<img src="5day-png/34TSDB数据目录结构.png" alt="image-20250308163230047" style="zoom:50%;" />

PTSDB本地存储使用自定义的文件结构。

Prometheus 默认将数据存储在安装目录下的./data/目录

在./data/目录下类似如01JND606HBBW2GVJC68REVY89E形式的目录为2小时块Block目录

每个Block块都有独立的目录,这些目录叫做2小时块。

每个2小时块目录包含一个chunks子目录（包含那个时间窗口里的所有时间序列）、元数据文件meta.json、索引文件index、tombstones

- chunks 目录

  用于保存时序数据文件

  chunks目录中的样例被分组到一个或多个段文件中，各Chunk文件以数字编号,比如:000001

  每个chunk文件的默认最大上限为512MB，如果达到上限则截断并创建为另一 个Chunk

- index 文件

  索引文件，它是Prometheus TSDB实现高效查询的基础

  可以通过Metrics Name和Labels查找时间序列数据在chunk文件中的位置

  索引文件指标名称和标签索引到chunks目录中的时间序列上

- tombstones 文件

  用于对数据进行软删除，不会立即删除块段（chunk segment）中的数据，即“ 标记删除”，以降低删除操作的开销

  删除的记录并保存于墓碑 tombstones文件中，而读取时间序列上的数据时，会基于tombstones进行过滤已经删除的部分

- meta.json 文件

  block的元数据信息，这些元数据信息是block的合并、删除等操作的基础依赖

```bash
[root@ubuntu2404 ~]#ll /usr/local/prometheus/data/
total 80
drwxr-xr-x 15 prometheus prometheus  4096 Mar  8 15:16 ./
drwxr-xr-x  8 prometheus prometheus  4096 Mar  4 17:48 ../
drwxr-xr-x  3 prometheus prometheus  4096 Mar  3 13:00 01JND606HBBW2GVJC68REVY89E/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  3 19:00 01JNDTKCACAXXWCWEGJE5EGZV7/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  4 13:00 01JNFRD03ZA8G3478WC1FENCVS/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  6 09:12 01JNMG6AXE5V3BBV4GV84XTBSG/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  6 12:12 01JNMTFY8MN7F1DH8Y6CX7BAMH/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  6 19:00 01JNNHSH2ZV96856X8HVHFN4NX/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  7 19:14 01JNR51CSZZKASRPMQ8ANPZMPS/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  8 12:16 01JNSZF8N0HSK1AR010ZKHKKWW/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  8 12:16 01JNSZF9FRXGC9EQH45696BXJM/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  8 12:16 01JNSZF9VSEN4HREAQKJXFY264/
drwxr-xr-x  3 prometheus prometheus  4096 Mar  8 15:16 01JNT9RSZDP20701G98R7PJF0A/
drwxr-xr-x  2 prometheus prometheus  4096 Mar  8 15:17 chunks_head/
-rw-r--r--  1 prometheus prometheus     0 Mar  8 13:44 lock			#锁文件
-rw-r--r--  1 prometheus prometheus 20001 Mar  8 16:29 queries.active	#查询相关数据
drwxr-xr-x  3 prometheus prometheus  4096 Mar  8 15:16 wal/			#写前日志
[root@ubuntu2404 ~]#ll /usr/local/prometheus/data/01JND606HBBW2GVJC68REVY89E/
total 612
drwxr-xr-x  3 prometheus prometheus   4096 Mar  3 13:00 ./
drwxr-xr-x 15 prometheus prometheus   4096 Mar  8 15:16 ../
drwxr-xr-x  2 prometheus prometheus   4096 Mar  3 13:00 chunks/		#块存放目录
-rw-r--r--  1 prometheus prometheus 602216 Mar  3 13:00 index		#索引文件
-rw-r--r--  1 prometheus prometheus    588 Mar  3 13:00 meta.json	#元数据，包括样本数，数据时间，合并等
-rw-r--r--  1 prometheus prometheus      9 Mar  3 13:00 tombstones	#标记删除的信息
```

**WAL Write-Ahead Logging（写前日志）**

![image-20250308164137446](5day-png/34TSDB数据存储写前日志.png)

Head块是数据库位于内存中的部分，Block 是磁盘上不可更改的持久块,而预写日志(WAL) 用于辅助完成持久写入

传入的样本(k/v)首先会进入Head，并在内存中停留一段时间默认2小时，然后即会被刷写到磁盘并映射回内存中(M-map) 

内存存储的只是引用而已。使用内存映射，我们可以在需要的时候使用该引用将 chunk 动态加载到内存

当这些内存映射的块Chunks 或内存中的Chunks 块老化到一定程度时，它会将作为持久块刷入到磁盘

随着它们的老化进程，将合并更多的块，最在超过保留期限后被删除

WAL是数据库中发生的事件的顺序日志，在写入/修改/删除数据库中的数据之前，首先将事件记录附加)到WAL中，然后在数据库中执行必要的操作

WAL用于帮助TSDB先写日志，再写磁盘上的Block

使用WAL技术可以方便地保证原子性、重试等

WAL被分割为默认为128MB大小的文件段，它们都位于WAL目录下

WAL日志的数量及截断的位置则保存于checkpoint文件中，该文件的内容要同步写入磁盘，以确保其可靠性

**压缩机制**

<img src="5day-png/34TSDB数据存储压缩机制.png" alt="image-20250308164448714" style="zoom:50%;" />

Prometheus将最近的数据保存在内存中，这样查询最近的数据会变得非常快，然后通过一个compactor定时将数据打包到磁盘。

数据在内存中最少保留2个小时(storage.tsdb.min-block-duration)。

之所以设置2小时这个值，应该是Gorilla那篇论文中上图观察得出的结论

即压缩率在2小时时候达到最高，如果保留的时间更短，就无法最大化的压缩

最新的数据是保存在内存中的，并同时写入至磁盘上的预写日志（WAL），每一次动作都会记录到预写日志中。

预写日志文件被保存在wal目录中，以128MB的段形式存在

这些预写日志文件包含尚未压缩的原始数据，因此它们比常规的块文件大得多

prometheus至少保留3个预写日志文件

高流量的服务器可能会保留多于3个的预写日志文件，以至少保留2个小时的原始数据。

如果prometheus 崩溃时，就会重放这些日志，因此就能恢复数据

2小时块最终在后台会被压缩成更长久的块



#### **Prometheus** **配置本地存储相关选项**

```bash
--storage.tsdb.path           #数据存储位置，默认是安装和录下的data子目录。
--storage.tsdb.retention.time #保留时间，默认是15天，过15天之后就删除。该配置会覆盖--storage.tsdb.retention的值。
--storage.tsdb.retention.size #要保留的块Block的最大字节数(不包括WAL大小)。比如:100GB,支持单位: B,KB, MB, GB, TB, PB, EB如果达到指定大小,则最旧的数据会首先被删除。默认为0或禁用，注意：此值不是指定每个段文件chunk的大小
#注意：上面两个参数值retention.time和retention.size只要有一个满足条件，就会删除旧的数据
--storage.tsdb.wal-compression #开启预写日志的压缩,默认开启
```

**磁盘空间公式**

```bash
needed_disk_space = retention_time_seconds * ingested_samples_per_second * bytes_per_sample
#retention_time_seconds：保留时间
#ingested_samples_per_second：样本数
#bytes_per_sample：每个样本大小
```



#### Prometheus 远程存储 VicetoriaMetrics

Prometheus的远程存储可以解决以上问题，远程存储有多种方案，比如：VictoriaMetrics，Thanos（灭霸） 和 influxdb，也可以通过adapter适配器间接的存储在Elasticsearch或PostgreSQL中

VictoriaMetrics(VM) 是基于Golang实现的一个支持高可用、经济高效且可扩展的监控解决方案和时间序列数据库，可用于 Prometheus 监控数据做长期远程存储。Thanos 方案也可以用来解决 Prometheus 的高可用和远程存储的问题

相对于 Thanos，VictoriaMetrics 主要是一个可水平扩容的本地全量持久化存储方案，性能比thanos性能要更好

而 Thanos不是本地全量的，它很多历史数据是存放在对象存储当中的，如果要查询历史数据都要从对象存储当中去拉取，这肯定比本地获取数据要慢，VictoriaMetrics要比Thanos性能要好

VictoriaMetrics具有以下突出特点：

```
它可以用作Prometheus的长期储存。
它可以用作Grafana中Prometheus的直接替代品，因为它支持Prometheus查询 API。
它可以用作Grafana中Graphite的直接替代品，因为它支持Graphite API。
它实现了基于PromQL的查询语言MetricsQL，它在PromQL之上提供了改进的功能。
它提供全局查询视图。多个Prometheus实例或任何其他数据源可能会将数据摄取到VictoriaMetrics中。稍后可以通过单个查询查询此数据。
它为数据引入和数据查询提供了高性能以及良好的垂直和水平可扩展性。它的性能比InfluxDB和TimescaleDB高出20倍。
在处理数百万个独特的时间序列（又称高基数）时，它使用的RAM比InfluxDB少10倍，比Prometheus、Thanos 或 Cortex 少7倍。
它针对具有高流失率的时间序列进行了优化。
它提供了高数据压缩，因此与TimescaleDB相比，可以将多达70倍的数据点塞入有限的存储空间，与
Prometheus，Thanos或Cortex相比，所需的存储空间最多可减少7倍。
它针对具有高延迟IO和低IOPS（AWS，Google Cloud，Microsoft Azure等中的HDD和网络存储）的存储进行了优化。
单节点VictoriaMetrics可以替代使用Thanos，M3DB，Cortex，InfluxDB或TimescaleDB等竞争解决方案构建的中等规模的集群。
由于存储架构，它可以在不干净关闭（即OOM，硬件重置或kill -9）时保护存储免受数据损坏。
它支持指标重新标记。
它可以通过序列限制器处理高基数问题和高流失率问题。
它非常适合处理来自APM、Kubernetes、物联网传感器、联网汽车、工业遥测、财务数据和各种企业工作负载的量时间序列数据。
它具有开源群集版本。
它具有易于设置和操作的特点
它可以将数据存储在基于 NFS 的存储上，例如Amazon EFS和Google Filestore。
```

官网

```
https://victoriametrics.com/
https://github.com/VictoriaMetrics/VictoriaMetrics
```

官方文档

```
https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html
```

VictoriaMetrics 分为单节点和集群两个方案

- 官方建议如果采集数据点(data points)低于 100w/s时推荐单节点，但不支持告警。
- 集群支持数据水平拆分

常见部署方式

- 基于二进制单机安装
- 基于集群安装
- 基于 Docker 运行
- 基于 Docker Compose 安装

##### **VictoriaMetrics** **单机二进制布署**

```
https://github.com/VictoriaMetrics/VictoriaMetrics/releases
```

```bash
[root@ubuntu2404 ~]#ls
victoria-metrics-linux-amd64-v1.112.0.tar.gz
[root@ubuntu2404 ~]#tar xf victoria-metrics-linux-amd64-v1.112.0.tar.gz -C /usr/local/bin/

#准备用户和数据目录
[root@ubuntu2404 ~]#useradd -r -s /sbin/nologin victoriametrics
[root@ubuntu2404 ~]#mkdir -p /data/victoriametrics
[root@ubuntu2404 ~]#chown -R victoriametrics:victoriametrics /data/ictoriametrics
```

```bash
#创建service文件
[root@ubuntu2404 ~]#vim /lib/systemd/system/victoriametrics.service
[Unit]
Description=VictoriaMetrics
Documentation=https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html
After=network.target

[Service]
Restart=on-failure
User=victoriametrics
Group=victoriametrics
ExecStart=/usr/local/bin/victoria-metrics-prod -httpListenAddr=0.0.0.0:8428 -storageDataPath=/data/victoriametrics -retentionPeriod=12
#ExecReload=/bin/kill -HUP \$MAINPID
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl enable --now victoriametrics.service

```

```bash
[root@ubuntu2404 ~]#ss -tnulp | grep vi
tcp   LISTEN 0      4096                                   0.0.0.0:8428       0.0.0.0:*    users:(("victoria-metric",pid=19679,fd=6))    
[root@ubuntu2404 ~]#ls /data/victoriametrics/
data  flock.lock  indexdb  metadata  snapshots  tmp
```

**修改prometheus使用victoriametrics实现远程存储**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
#加下面两行，和global平级
remote_write:
  - url: http://10.0.0.200:8428/api/v1/write
   
[root@ubuntu2404 ~]#systemctl reload prometheus.service
```

**基于** **Docker** **部署**

```bash
docker run -it --rm -v /path/to/victoria-metrics-data:/victoria-metrics-data -p 8428:8428 victoriametrics/victoria-metrics
```

#### **VictoriaMetrics** **集群部署**

![image-20250308180613787](5day-png/34victoriaMetrics集群部署.png)

集群版的victoriametrics有下面主要服务组成：

![image-20250308181649841](5day-png/34victoriaMetrics集群部署1.png)

- vmstorage

  数据存储节点，负责存储时序数据,默认使用3个端口

  API Server的端口: 8482/tcp，由选项 -httpListenAddr 指定

  从vminsert接收并存入数据的端口:8400/ tcp，由选项-vminsertAddr指定

  从vmselect接收查询并返回数据的端口:8401 / tcp，由选项-vmselectAddr指定

  vmstorage节点间不进行任何交互，都是独立的个体, 使用上层的vminsert产生副本和分片

  有状态，一个vmstorage节点故障，会丢失约1/N的历史数据(N为vmstorage节点的数量)

  数据存储于选项-storageDataPath指定的目录中，默认为./vmstorage-data/

  vmstorage的节点数量,需要真实反映到vminsert和vmselect之上

  支持单节点和集群模式:集群模式支持多租户，其API端点也有所不同

- vminsert

  数据插入节点，负责接收用户插入请求，基于metric名称和label使用一致性Hash算法向不同的

  vmstorage写入时序数据

  负责接收来自客户端的数据写入请求，并转发到选定的vmstorage节点，监听一个端口

  接收数据存入请求的端口:8480/ tcp，由选项-httpListenAddr指定

  它是Prometheus remote_write协议的一个实现，可接收和解析通过该协议远程写入的数据若接入的是

  VM存储集群时

  其调用端点的URL格式为

  ```bash
  http: //<vminsert>:8480 /insert/<accountID>/<suffix>
  #<accountID>是租户标识
  #<suffix>中，值/prometheus和/prometheus/api/v1/write的作用相同，专用于接收prometheus写入请求
  ```

- vmselect

  数据查询节点，负责接收用户查询请求，向vmstorage查询时序数据

  负责接收来自客户端的数据查询请求，转发到各vmstorage节点完成查询，并将结果合并后返回给客户

  端监听一个端口

  基于metric名称和label使用一致性Hash算法向不同的vmstorage查询时序数据

  接收数据查询请求的端口:8481/tcp，可由选项-httpListenAddr指定

  它是prometheus remote_read协议的一个实现，可接收和解析通过该协议传入的远程查询请求

  专用于prometheus的URL格式为

  ```
  http://<vmselect>:8481/select/<accountID>/prometheus
  #<accountID>是租户标识
  ```

- vmagent

  可以用来替换 Prometheus，实现数据指标抓取，支持多种后端存储，会占用本地磁盘缓存

  相比于 Prometheus 抓取指标来说具有更多的灵活性，支持pull和push指标

  默认端口 8429

- vmalert

  报警相关组件，如果不需要告警功能可以不使用，默认端口为 8880

以上组件中 vminsert 以及 vmselect（几乎）都是无状态的，所以扩展很简单，只有 vmstorage 是有状态的。

在部署时可以按照需求，不同的微服务部署不同的副本，以应对业务需求：

- 若数据量比较大，部署较多的vmstorage副本
- 若查询请求比较多，部署较多的vmselect副本
- 若插入请求比较多，部署较多的vminsert副本



**VictoriaMetrics集群二进制部署**

以下实现部署一个三节点的 VictoriaMetrics 集群，且三节点同时都提供vmstorage，vminsert和vmselect角色

![image-20250308182048488](5day-png/34victoriaMetrics集群部署2.png)

```bash
#所有都做以下操作
[root@ubuntu2404 ~]#ls
victoria-metrics-linux-amd64-v1.112.0-cluster.tar.gz
[root@ubuntu2404 ~]#tar xf victoria-metrics-linux-amd64-v1.112.0-cluster.tar.gz -C /usr/local/bin/
```

```bash
[root@ubuntu2404 ~]#vim /lib/systemd/system/vmselect.service 
[Unit]
Description=VictoriaMetrics Cluster Vmselect
Documentation=https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html
After=network.target
[Service]
Restart=on-failure
ExecStart=/usr/local/bin/vmselect-prod \
    -httpListenAddr=:8481 \
    -storageNode=10.0.0.201:8401,10.0.0.202:8401,10.0.0.203:8401
#ExecReload=/bin/kill -HUP \$MAINPID
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
[root@ubuntu2404 ~]#vim /lib/systemd/system/vmstorage.service 
[Unit]
Description=VictoriaMetrics Cluster Vmstorage
Documentation=https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html
After=network.target

[Service]
Restart=on-failure
ExecStart=/usr/local/bin/vmstorage-prod \
    -httpListenAddr=:8482 \
    -vminsertAddr=:8400 \
    -vmselectAddr=:8401 \
    -storageDataPath=/data/vmstorage \
    -loggerTimezone=Asia/Shanghai
#ExecReload=/bin/kill -HUP $MAINPID
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#vim /lib/systemd/system/vminsert.service 
[Unit]
Description=VictoriaMetrics Cluster Vminsert
Documentation=https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html
After=network.target
[Service]
Restart=on-failure
ExecStart=/usr/local/bin/vminsert-prod -httpListenAddr :8480 -storageNode=10.0.0.201:8400,10.0.0.202:8400,10.0.0.203:8400
#ExecReload=/bin/kill -HUP \$MAINPID
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload 
systemctl enable --now vmselect.service 
systemctl enable --now vminsert.service 
systemctl enable --now vmstorage.service 
```

**配置** **Prometheus** **使用** **victoriametrics** **实现集群存储**

```bash
[root@ubuntu2404 ~]#vim /usr/local/prometheus/conf/prometheus.yml
remote_write:
  - url: http://10.0.0.201:8480/insert/0/prometheus
  - url: http://10.0.0.202:8480/insert/0/prometheus
  - url: http://10.0.0.203:8480/insert/0/prometheus
remote_read:
  - url: http://10.0.0.201:8481/select/0/prometheus
  - url: http://10.0.0.202:8481/select/0/prometheus
  - url: http://10.0.0.203:8481/select/0/prometheus

#说明
10.0.0.20[1-3]:8480为 三个集群节点的IP和vminsert组件端口
0表示租户ID,即逻辑名称空间,用于区分不同的prometheus集群
```

也可以使用负载均衡的VIP和端口,以Haproxy为例,如下配置

```bash
vim /etc/haproxy/haproxy.cfg
......
listen victoriametrics-vmselect-8481
   bind 10.0.0.200:8481
   mode tcp
   server vmselect1 10.0.0.201:8481 check inter 1s fall 3 rise 5
   server vmselect2 10.0.0.202:8481 check inter 1s fall 3 rise 5
   server vmselect3 10.0.0.203:8481 check inter 1s fall 3 rise 5
......
#URL填写下面VIP地址和端口
http://10.0.0.200:8481/select/0/prometheus
```



#### grafana展示

1860,8919,11074,13824

 
