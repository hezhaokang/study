# 25、keepalived

**集群类型 Cluster**

- LB：Load Balance 负载均衡 

  LVS/HAProxy/nginx（http/upstream, stream/upstream） 

- HA：High Availability 高可用集群 

  MySQL、Redis、Zookeeper、Kafka 有状态的服务 

  KeepAlived 通用的高可用集群,更适合无状态的服务 

  SPoF: Single Point of Failure，解决单点故障 

- HPC：High Performance Computing 高性能集群

**系统故障** 

- 硬件故障：设计缺陷、wear out（损耗）、自然灾害…… 

- 软件故障：设计缺陷 bug 

- 人为故障：故意或无意

**高可用实现**

提升系统高用性的解决方案：降低MTTR- Mean Time To Repair(平均故障时间) 

解决方案：建立多节点的冗余机制 

- active/passive 主/备 

- active/active 双主 

- active --> HEARTBEAT --> passive 
- active <--> HEARTBEAT <--> active

**HA Service** 

资源：组成一个高可用服务的“组件”，比如：vip，service process，shared storage 

- passive 

- node的数量 资源切换

**Shared Storage**

- NAS(Network Attached Storage)：网络附加存储，基于网络的共享文件系统。 

- SAN(Storage Area Network)：存储区域网络，基于网络的块级别的共享 

- 分布式存储: Ceph、GlusterFS、HDFS、GFS、DFS、moosefs，MinIO ( OSS 对象存储)等

**HA Cluster 实现方案**

1. AIS：Applicaiton Interface Specification 应用程序接口规范

   - RHCS：Red Hat Cluster Suite 红帽集群套件

   - heartbeat：基于心跳监测实现服务高可用

   - pacemaker+corosync：资源管理与故障转移

2. VRRP：Virtual Router Redundancy Protocol

- 虚拟路由冗余协议,解决静态网关单点风险 

  - 物理层:路由器、三层交换机 

  - 软件层:keepalived



## VRRP

<img src="D:\桌面\mage.md\笔记\5day-png\25VRRP.png" alt="image-20250111104750586" style="zoom: 33%;" />

### 相关术语

- 虚拟路由器：Virtual Router  

- 虚拟路由器标识：VRID(0-255)，唯一标识虚拟路由器 

- VIP：Virtual IP  

- VMAC：Virutal MAC (00-00-5e-00-01-VRID) 

- 物理路由器： 

  - master：主设备 

  - backup：备用设备 

  - priority：优先级

### VRRP 相关技术

通告：心跳，优先级等；周期性 

工作方式：抢占式，非抢占式 

安全认证： 

- 无认证 

- 简单字符认证：预共享密钥 

- MD5  

工作模式： 

- 主/备：单个虚拟路由器 

- 主/主：主/备（虚拟路由器1），备/主（虚拟路由器2）



## Keepalived

### 架构

- 用户空间核心组件： 

  - vrrp stack：VIP消息通告 

  - checkers：监测 

  - Real Server system call：实现 vrrp 协议状态转换时调用脚本的功能 

  - SMTP：邮件组件 

  - IPVS wrapper：生成 IPVS 规则 

  - Netlink Reflector：网络接口 

  - WatchDog：监控进程 

- 控制组件：提供keepalived.conf 的解析器，完成Keepalived配置 

- IO复用器：针对网络目的而优化的自己的线程抽象 

- 内存管理组件：为某些通用的内存管理功能（例如分配，重新分配，发布等）提供访问权限

```powershell
Keepalived <-- Parent process monitoring children
\_ Keepalived <-- VRRP child
\_ Keepalived <-- Healthchecking child
```



## 安装

### 包安装

```powershell
#yum:
yum makecache;yum -y install keepalived

#apt
apt update;apt install keepalived -y
```

```powershell
#apt安装#默认没有配置文件无法启动（/etc/keepalived/keepalived.conf这个文件没有）
#利用范例生成配置文件
cp /usr/share/doc/keepalived/samples/keepalived.conf.sample /etc/keepalived/keepalived.conf

[root@ubuntu2404 ~]#ps auxf | grep keepalived
root        1687  0.0  0.4  27924  9600 ?        Ss   10:31   0:00 /usr/sbin/keepalived --dont-fork
root        1688  0.0  0.2  27924  4652 ?        S    10:31   0:00  \_ /usr/sbin/keepalived --dont-fork
root        1689  0.0  0.1  27924  3760 ?        S    10:31   0:00  \_ /usr/sbin/keepalived --dont-fork
```



### 编译安装

```powershell
#ubuntu安装相关包
apt update && apt -y install make gcc ipvsadm build-essential pkg-config automake autoconf libipset-dev libnl-3-dev libnl-genl-3-dev libssl-dev libxtables-dev libip4tc-dev libip6tc-dev libmagic-dev libsnmp-dev libglib2.0-dev libpcre2-dev libnftnl-dev libmnl-dev libsystemd-dev

#Ubuntu18.04安装相关包
apt update;apt -y install gcc curl openssl libssl-dev libpopt-dev daemon build-essential

#红帽系统安装相关包
yum install gcc curl openssl-devel libnl3-devel net-snmp-devel


#下载软件解压软件
wget https://keepalived.org/software/keepalived-2.0.20.tar.gz
tar xfv keepalived-2.0.20.tar.gz -C /usr/local/src

#编译软件
cd /usr/local/src/keepalived-2.0.20
./configure --prefix=/usr/local/keepalived
make && make install

/usr/local/keepalived/sbin/keepalived -v

#源码目录会自动生成unit文件
cp /usr/local/src/keepalived-2.0.20/keepalived/keepalived.service /lib/systemd/system/

#和包安装一样默认没有配置文件无法启动（/etc/keepalived/keepalived.conf这个文件没有）
#创建配置文件，默认配置路径可以是/usr/local/keepalived/etc/keepalived/keepalived.conf（优先高）或者/etc/keepalived/keepalived.conf（优先级低）
#利用范例生成配置文件
mkdir /etc/keepalived/
cp /usr/local/keepalived/etc/keepalived/samples/keepalived.conf.sample /etc/keepalived/keepalived.conf
systemctl restart keepalived.service 

rocky9启动目录
#编译安装默认没有配置文件无法启动（/etc/keepalived/keepalived.conf这个文件没有）
#利用范例生成配置文件
#创建配置文件，默认配置路径可以是/usr/local/keepalived/etc/keepalived/keepalived.conf（优先高）或者/etc/keepalived/keepalived.conf（优先级低）
mkdir /etc/keepalived/
cp /usr/local/keepalived/etc/keepalived/samples/keepalived.conf.sample /etc/keepalived/keepalived.conf
systemctl restart keepalived/
```



### 脚本编译安装

```
#!/bin/bash
#

#本脚本支持在线或离线编译安装

KEEPALIVED_VERSION=2.3.2
#KEEPALIVED_VERSION=2.2.8
#KEEPALIVED_VERSION=2.2.7
#KEEPALIVED_VERSION=2.2.2
#KEEPALIVED_VERSION=2.0.20
KEEPALIVED_FILE=keepalived-${KEEPALIVED_VERSION}.tar.gz

KEEPALIVED_INSTALL_DIR=/usr/local/keepalived
SRC_DIR=/usr/local/src
KEEPALIVED_URL=https://keepalived.org/software/

CPUS=`grep -c processor  /proc/cpuinfo`

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


download_file (){
    if [ $ID = 'centos' -o $ID = 'rocky' ];then
        rpm -q wget &> /dev/null || yum -y install wget 
    elif [ $ID = 'ubuntu' ];then
        dpkg -l |grep wget || { apt update;  apt install -y wget; } 
    else
        color "不支持此操作系统，退出!" 1
        exit
    fi
    if [ ! -e ${KEEPALIVED_FILE} ];then
        wget --no-check-certificate  ${KEEPALIVED_URL}${KEEPALIVED_FILE} 
        [ $? -ne 0 ] && { color "KEEPALIVED源码包下载失败" 1 ; exit; }
    fi
}

install_keepalived () {
    if [ $ID = 'centos' -o $ID = 'rocky' ];then
        yum -y install make gcc ipvsadm autoconf automake openssl-devel libnl3-devel iptables-devel net-snmp-devel glib2-devel pcre2-devel  libmnl-devel systemd-devel &> /dev/null
    elif [ $ID = 'ubuntu' ];then
        apt update 
        apt -y install make gcc ipvsadm build-essential pkg-config automake autoconf libipset-dev libnl-3-dev libnl-genl-3-dev libssl-dev libxtables-dev libip4tc-dev libip6tc-dev libipset-dev libmagic-dev libsnmp-dev libglib2.0-dev libpcre2-dev libnftnl-dev libmnl-dev libsystemd-dev
    else
        color "不支持此操作系统，退出!" 1
    fi
    tar xf ${KEEPALIVED_FILE} -C ${SRC_DIR}
    cd ${SRC_DIR}/keepalived-${KEEPALIVED_VERSION}
    ./configure --prefix=${KEEPALIVED_INSTALL_DIR} --disable-fwmark
    make -j $CPUS && make install
    if [ $? -eq 0 ];then
        color "KEEPALIVED编译安装成功" 0
    else
        color "KEEPALIVED编译安装失败,退出!" 1
        exit
    fi
    [ -d /etc/keepalived ] || mkdir -p /etc/keepalived
    cp ${KEEPALIVED_INSTALL_DIR}/etc/keepalived/keepalived.conf.sample  /etc/keepalived/keepalived.conf
}

start_keepalived () {
    cp ./keepalived/keepalived.service /lib/systemd/system/
    systemctl daemon-reload
    systemctl enable --now keepalived &> /dev/null 
    systemctl is-active keepalived
    if [ $? -eq 0 ] ;then
        color "Keepalived 服务安装成功!" 0  
    else
        color "Keepalived 服务安装失败!" 1
        exit 1
    fi
}

download_file

install_keepalived

start_keepalived

```



## keepalived配置说明

**配置文件**

```powershell
/etc/keepalived/keepalived.conf
```

**配置文件组成** 

- GLOBAL CONFIGURATION 

  Global definitions：定义邮件配置，route_id，vrrp配置，多播地址等 

- VRRP CONFIGURATION 

  VRRP instance(s)：定义每个vrrp虚拟路由器 

- LVS CONFIGURATION 

  Virtual server group(s) 

  Virtual server(s)：LVS集群的VS和RS

**帮助**

```powershell
#包安装
man keepalived.conf
#编译安装
man /apps/keepalived/share/man/man5/keepalived.conf.5
man /usr/local/keepalived/share/man/man5/keepalived.conf.5
```



```powershell
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen							# eepalived 发生故障切换时邮件发送的目标邮箱，可按行区分写多个
   	 2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc		# 送邮箱的地址
   smtp_server 192.168.200.1			# 件服务器地址
   smtp_connect_timeout 30				# 件服务器连接timeout
   router_id LVS_DEVEL					# 个keepalived主机唯一标识，建议使用当前主机名，如果多节点重名可能会影响切换脚本执行
   vrrp_skip_check_adv_addr				# 认会对所有通告报文都检查，会比较消耗性能，启用此配置后，如果收到的通告报文和上一个报文是同一个路由器，则跳过检查
   vrrp_strict							# 格遵守VRRP协议,启用此项后以下状况将无法启动服务或工作异常:1.无VIP地址 2.配置了单播邻居 3.在VRRP版本2中有IPv6地址，开启动此项并且没有配置  
   										# rrp_iptables时会自动开启iptables(旧内核)或者nft(新内核)的防火墙规则，默认导致VIP无法访问,建议不加此项配置
   vrrp_garp_interval 0      			# ratuitous 免费 ARP messages 报文发送延迟，0表示不延迟
   vrrp_gna_interval 0       			# nsolicited NA messages （不请自来）消息发送延迟
   vrrp_mcast_group4 224.0.0.18 	# 定组播IP地址范围：224.0.0.0到239.255.255.255,默认值：224.0.0.18，如果配置了单播，此项失效
   vrrp_iptables   						# 项和vrrp_strict同时开启时，则不会添加防火墙规则,如果无配置vrrp_strict项,则无需启用此项配置，注意：新版加此项仍有iptables(旧内核)或者nft(新内核)规则
}

vrrp_instance <STRING> {		# String>为vrrp的实例名,一般为业务名称
    state MASTER|BACKUP			# 前节点在此虚拟路由器上的初始状态，状态为MASTER或者BACKUP，当priority相同时，先启动的节点优先获取VIP
	interface eth0				# 定为当前VRRP虚拟路由器使用的物理接口，如：eth0,bond0,br0,可以和VIP不在一个网卡，实现心跳功能
    virtual_router_id 50		# 个虚拟路由器唯一标识，范围：0-255，每个虚拟路由器此值必须唯一，否则服务无法启动，同属一个虚拟路由器的多个keepalived节点必须相同,务必要确认在同一网络中此值必须唯一
    nopreempt					# 不抢占优先级更高的主服务器
    priority 100				# 前物理节点在此虚拟路由器的优先级，范围：1-254，每个keepalived主机节点此值不同，如果多节点此值相同，则先来后到原理获取VIP
    advert_int 1				# rrp通告的时间间隔，默认1s，注意：集群内多节点此值必须相同
    authentication { 			# 证机制
        auth_type AH|PASS   	# H为IPSEC认证(不推荐),PASS为简单密码(建议使用)
        auth_pass <PASSWORD> 	# 共享密钥，仅前8位有效，同一个虚拟路由器的多个keepalived节点必须一样
    }

    virtual_ipaddress {			# 拟IP,生产环境可能指定几十上百个VIP地址
        192.168.200.11			# 定VIP，不指定网卡，默认为eth0,注意：不指定/prefix,默认为/32
        192.168.200.12/24 dev eth1					#定VIP的网卡，建议和interface指令指定的网卡不在一个网卡
        192.168.200.13/24 dev eth2 label eth2:1 	#指定VIP的网卡label 
    }
}

	track_interface { 			# 置监控网络接口，一旦出现故障，则转为FAULT状态实现地址转移
	 	eth0
 		eth1
 		…
	} 

virtual_server 10.10.10.2 1358 {
    delay_loop 6				# 健康检查的周期为 6 秒
    lb_algo rr					# 负载均衡算法为轮询（Round Robin）
    lb_kind NAT					# 负载均衡模式为 NAT
    persistence_timeout 50		# 会话保持时间为 50 秒
    protocol TCP				# 使用 TCP 协议

    sorry_server 192.168.200.200 1358	# 当所有后端都不可用时的备用服务器

    real_server 192.168.200.2 1358 {	
        weight 1				# 权重设置为 1
        HTTP_GET {	
        	url {
                path /
                status_code 200     # 检查返回的 HTTP 状态码是否为 200
            }
            connect_timeout 5s      # 连接超时时间为 5 秒
            nb_get_retry 3          # 最大重试次数为 3
            delay_before_retry 3s   # 每次重试间隔 3 秒
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1                    # 添加第二个后端服务器
        HTTP_GET {
            url {
                path /
                status_code 200
            }
            connect_timeout 5s
            nb_get_retry 3
            delay_before_retry 3s
        }
    }
}
```

## keepalived日志

默认 keepalived的日志记录在LOG_DAEMON中，记录在/var/log/syslog或messages, 也支持自定义日 志配置

```powershell
包安装
rocky
#修改文件路径
vim /etc/sysconfig/keepalived
...
local6.* /var/log/keepalived.log

ubuntu
#修改文件路径



编译安装
grep ExecStart /lib/systemd/system/keepalived.service
ExecStart=/usr/local/keepalived/sbin/keepalived $KEEPALIVED_OPTIONS
vim /usr/local/keepalived/etc/sysconfig/keepalived
...
KEEPALIVED_OPTIONS="-D -S 6"

vim /etc/rsyslog.conf
#添加这一行
local6.* /var/log/keepalived.log

```



## Keepalived 独立子配置文件

利用include 指令可以实现包含子配置文件

格式

```powershell
include /path/file
```

```powershell
# mkdir /etc/keepalived/conf.d
# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
include /etc/keepalived/conf.d/*.conf

# vim /etc/keepalived/conf.d/cluster1.conf
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 80
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        192.168.200.11
        192.168.200.12
        192.168.200.13
    }
}
```

## Keepalived 实现 VRRP

<img src="D:\桌面\mage.md\笔记\5day-png\25keepalivedVRRP结构.png" alt="image-20250111152419838" style="zoom:33%;" />

增加仅主机网卡eth1，让VRRP协议走eth1网卡

```powershell
master 配置
#vim /etc/keepalived/keepalived.conf
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka1.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf

vim /etc/keepalived/conf.d/cluster1.conf
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
}
```

```powershell
backup 配置
vim /etc/keepalived/keepalived.conf
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka2.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf

vim /etc/keepalived/conf.d/cluster1.conf
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
}
```

### 抢占和非抢占模式

默认为抢占模式 preempt，即当高优先级的主机恢复在线后，会抢占低先级的主机的master角色，造成网络抖动，建议设置为非抢占模式 nopreempt ，即高优先级主机恢复后，并不会抢占低优先级主机的 master 角色

注意: 非抢占模式下,如果原主机down机, VIP迁移至的新主机, 后续新主机也发生down（（keepalived  服务down））时,VIP还会迁移回修复好的原主机 但如果新主机的服务down掉（keepalived服务正常），原主机也不会接管VIP，仍会由新主机拥有VIP

即非抢占式模式，只是适合当主节点宕机，切换到从节点的一次性的高可用性，后续即使当原主节点修 复好，仍无法再次起到高用功能

**注意：要关闭 VIP抢占，必须将各 Keepalived 服务器 state 配置为 BACKUP**

```powershell
#ha1 配置
vim /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP				#都为BACKUP
    interface eth1
    virtual_router_id 66
    priority 100				#优先级高·
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    nopreempt					#添加此行，设为nopreempt
}



#ha2 配置
vim /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP				#都为BACKUP
    interface eth1
    virtual_router_id 66
    priority 80					#优先级低
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    #nopreempt					#注意：如果ka2主机也是非抢占式，会导致ka1即使优先级降低于ka2，VIP也不会切换至ka2	
}

```

### 抢占延迟模式 preempt_delay

抢占延迟模式，即优先级高的主机恢复后，不会立即抢回VIP，而是延迟一段时间（默认300s）再抢回VIP

但是如果低优先级的主机down机，则立即抢占VIP地址，而不再延迟

```powershell
preempt_delay      		#指定抢占延迟时间为#s，默认延迟300s，在优先级高的节点配置
```

**注意：需要各keepalived服务器state为BACKUP,并且不要启用 vrrp_strict**

```powershell
#ka1 主机配置
vim /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP					#都为BACKUP
    interface eth1
    virtual_router_id 66
    priority 100					#优先级高
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    preempt_delay 60				#抢占延迟模式，默认延迟300s，这里设置60s·
}

#ka2 主机配置
vim /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP					#都为BACKUP
    interface eth1
    virtual_router_id 66
    priority 80						#优先级低
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
}
```



### VIP单播配置

默认keepalived主机之间利用多播相互通告消息，会造成网络拥塞，可以设置为单播，减少网络流量 

另外：有些公有云不支持多播，可以利用单播实现

单播优先与多播，即同时配置，单播生效

**注意：启用 vrrp_strict 时，不能启用单播**

```powershell
#在所有节点vrrp_instance语句块中设置对方主机的IP，建议设置为专用于对应心跳线网络的地址，而非使用业务网络
unicast_src_ip <IPADDR>  	#指定发送单播的源IP
unicast_peer {
   <IPADDR>     			#指定接收单播的对方目标主机IP
   ......
}
```

```powershell
#ka1 配置
vim /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    preempt_delay 60
    unicast_src_ip 192.168.10.201		#本机IP
    unicast_peer{
        192.168.10.202					#指向对方主机IP,如果有多个keepalived,再加其它节点的IP
    }
}

#ka2 配置
vim /etc/keepalived/conf.d/cluster1.conf  
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202			#本机IP
    unicast_peer{
        192.168.10.201					#指向对方主机IP,如果有多个keepalived,再加其它节点的IP
    }
}
```

单播优先于多播

```powershell
ka1:
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka1.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18				#多播地址，单播优先于多播
}
include /etc/keepalived/conf.d/*.conf

[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    preempt_delay 60
    unicast_src_ip 192.168.10.201			#单播地址
    unicast_peer{
        192.168.10.202
    }
}
```

### Keepalived 通知脚本配置

当keepalived的状态变化时，可以自动触发脚本的执行，比如：发邮件通知用户 

默认以用户keepalived_script身份执行脚本，如果此用户不存在，以root执行脚本 

可以用下面指令指定脚本执行用户的身份

```powershell
global_defs {
	......
	script_user <USER>
	......
}
```

#### 通知脚本类型

- 当前节点成为主节点时触发的脚本

  ```
  notify_master <STRING>|<QUOTED-STRING>
  ```

- 当前节点转为备节点时触发的脚本

  ```
  notify_backup <STRING>|<QUOTED-STRING>
  ```

- 当前节点转为“失败”状态时触发的脚本

  ```
  notify_fault <STRING>|<QUOTED-STRING>
  ```

- 通用格式的通知触发机制，一个脚本可完成以上三种状态的转换时的通知

  ```
  notify <STRING>|<QUOTED-STRING>
  ```

- 当停止VRRP时触发的脚本

  ```
  notify_stop <STRING>|<QUOTED-STRING>
  ```

  

#### 脚本的调用方法

在 vrrp_instance VI_1 语句块的末尾加下面行，并给脚本加执行权限

```powershell
notify_master "/etc/keepalived/notify.sh master"
notify_backup "/etc/keepalived/notify.sh backup"
notify_fault "/etc/keepalived/notify.sh fault"
```

#### 脚本

```powershell
#!/bin/bash
#

contact='zhaokang2004@outlook.com'
email_send='2352136389@qq.com'
email_passwd='yhdoxkdnuylmdhge'
email_smtp_server='smtp.qq.com'

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


install_sendemail () {
    if [[ $ID =~ rhel|centos|rocky ]];then
        rpm -q sendemail &> /dev/null ||  yum install -y sendemail
    elif [ $ID = 'ubuntu' ];then
        dpkg -l |grep -q sendemail  || { apt update; apt install -y libio-socket-ssl-perl libnet-ssleay-perl sendemail ; } 
    else
        color "不支持此操作系统，退出!" 1
        exit
    fi
}

send_email () {
    local email_receive="$1"
    local email_subject="$2"
    local email_message="$3"
    sendemail -f $email_send -t $email_receive -u $email_subject -m $email_message -s $email_smtp_server -o message-charset=utf-8 -o tls=yes -xu $email_send -xp $email_passwd
    [ $? -eq 0 ] && color "邮件发送成功!" 0 || color "邮件发送失败!" 1 
}

notify() {
    if [[ $1 =~ ^(master|backup|fault)$ ]];then
        mailsubject="$(hostname) to be $1, vip floating"
        mailbody="$(date +'%F %T'): vrrp transition, $(hostname) changed to be $1"
        send_email "$contact" "$mailsubject" "$mailbody"
   else
        echo "Usage: $(basename $0) {master|backup|fault}"
        exit 1
   fi
}

install_sendemail 
notify $1
```

#### 实战案例2：实现 Keepalived 状态切换的通知脚本

1. 邮件配置rocky

```powershell
qq邮箱配置
[root@centos8 ~]# vim /etc/mail.rc
#在最后面添加下面行
set from=2352136389@qq.com
set smtp=smtp.qq.com
set smtp-auth-user=2352136389@qq.com
set smtp-auth-password=yhdoxkdnuylmdhge
set smtp-auth=login
set ssl-verify=ignore

163邮箱配置
[root@centos8 ~]#vi /etc/mail.rc
set from=xxx@163.com 			#之前设置好的邮箱地址
set smtp=smtp.163.com 			#邮件服务器
set smtp-auth-user=xxx@163.com 	#之前设置好的邮箱地址
set smtp-auth-password=QXFIOQXEJNSVSDM 	#授权码
set smtp-auth=login 			#默认login即可
```





## 脑裂现象

主备节点同时拥有同一个VIP，此时为脑裂现象 

注意：脑裂现象原因 

- 心跳线故障： 注意:在虚拟机环境中测试可以通过修改网卡的工作模式实现模拟，断开网卡方式无 法模拟 

- 防火墙错误配置：在从节点服务器执行iptables -A INPUT -s 主服务心跳网卡IP -j DROP 进行模拟 

- Keepalived 配置错误：多播或单播地址不同，interface错误，virtual_router_id不一致，密码不一 致

```powershell
arping -I eth1 -c1 10.0.0.10
```



## Master/Master 的 Keepalived 双主架构

master/slave的单主架构，同一时间只有一个Keepalived对外提供服务，此主机繁忙，而另一台主机却 很空闲，利用率低下，可以使用master/master的双主架构，解决此问题。

**Master/Master 的双主架构：**

即将两个或以上VIP分别运行在不同的keepalived服务器，以实现服务器并行提供web访问的目的，提高服务器资源利用率

```powershell
ha1 主机配置
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka1.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.con

[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_2 {
    state BACKUP
    interface eth1
    virtual_router_id 88
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.20 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}

ka2 主机配置
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka2.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf

[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}

[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 88
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.20 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
}
```

## 三个节点的三主三从架构实现

```powershell
#ka1:192.168.10.201\10.0.0.201 10.0.0.10 10.0.0.30
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka1.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state BACKUP
    interface eth1
    virtual_router_id 88
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 12345678
    }
    virtual_ipaddress {
      10.0.0.30 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.203
    }
}

#ka2:192.168.10.202\10.0.0.202 10.0.0.20 10.0.0.10
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka2.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 77
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
      10.0.0.20 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.203
    }
}

#ka3:192.168.10.203\10.0.0.203 10.0.0.30 10.0.0.20
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka3.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 77
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
      10.0.0.20 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.203
    unicast_peer{
        192.168.10.202
    }
}
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 88
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 12345678
    }
    virtual_ipaddress {
      10.0.0.30 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.203
    unicast_peer{
        192.168.30.201
    }
}
```



## 同步组

LVS NAT 模型VIP和DIP需要同步，需要同步组

```powershell
vrrp_sync_group VG_1 {
 group {
  	VI_1  # name of vrrp_instance (below)
  	VI_2  # One for each moveable IP
   }
 }
vrrp_instance VI_1 {
	eth0
	vip
 }
vrrp_instance VI_2 {
	eth1
	dip
 }
```

## 实现 IPVS 的高可用性

### 虚拟服务器配置结构

每一个虚拟服务器即一个IPVS集群 

可以通过下面语法实现

```
virtual_server IP port {
	...
	real_server {
	...
	}
	real_server {
	...
	}
	...
}
```

### Virtual Server （虚拟服务器）的定义格式

```powershell
virtual_server IP port			#定义虚拟主机IP地址及其端口
virtual_server fwmark int 		#ipvs的防火墙打标，实现基于防火墙的负载均衡集群
virtual_server group string 	#使用虚拟服务器组
```

### 虚拟服务器组

将多个虚拟服务器定义成一个组，统一对外服务，如：http和https定义成一个虚拟服务器组

```powershell
virtual_server_group <STRING> {
           # Virtual IP Address and Port
           <IPADDR> <PORT>
           <IPADDR> <PORT>
           ...
           # <IPADDR RANGE> has the form
           # XXX.YYY.ZZZ.WWW-VVV eg 192.168.200.1-10
           # range includes both .1 and .10 address
           <IPADDR RANGE> <PORT># VIP range VPORT
           <IPADDR RANGE> <PORT>
           ...
           # Firewall Mark (fwmark)
           fwmark <INTEGER>
           fwmark <INTEGER>
           ...
}
```

### 虚拟服务器配置

```powershell
virtual_server IP port {    			#VIP和PORT
 	delay_loop <INT> 					#检查后端服务器的时间间隔
 	lb_algo rr|wrr|lc|wlc|lblc|sh|dh 	#定义调度方法
 	lb_kind NAT|DR|TUN 					#集群的类型,注意要大写
 	persistence_timeout <INT> 			#持久连接时长
 	protocol TCP|UDP|SCTP 				#指定服务协议,一般为TCP
 	sorry_server <IPADDR> <PORT> 		#所有RS故障时，备用服务器地址
 	real_server <IPADDR> <PORT> {		#RS的IP和PORT
 		weight <INT>   					#RS权重
 		notify_up <STRING>|<QUOTED-STRING>  		#RS上线通知脚本
 		notify_down <STRING>|<QUOTED-STRING> 		#RS下线通知脚本
 		HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK { ... } 		#定义当前主机健康状态检测方法
	}
}

#注意:括号必须分行写,两个括号写在同一行,如: }} 会出错
```

### 应用层监测

应用层检测：HTTP_GET|SSL_GET

```powershell
HTTP_GET|SSL_GET {
	url {
		path <URL_PATH> 		#定义要监控的URL
		status_code <INT> 		#判断上述检测机制为健康状态的响应码，一般为 200
	}
	connect_timeout <INTEGER> 	#客户端请求的超时时长, 相当于haproxy的timeout server
	nb_get_retry <INT> 			#重试次数
	delay_before_retry <INT> 	#重试之前的延迟时长
	connect_ip <IP ADDRESS> 	#向当前RS哪个IP地址发起健康状态检测请求
	connect_port <PORT> 		#向当前RS的哪个PORT发起健康状态检测请求
	bindto <IP ADDRESS> 		#向当前RS发出健康状态检测请求时使用的源地址
	bind_port <PORT> 			#向当前RS发出健康状态检测请求时使用的源端口
}
```

范例：

```powershell
virtual_server 10.0.0.10 80 {
		delay_loop 3
		lb_algo wrr
		lb_kind DR
		protocol TCP
		sorry_server 127.0.0.1 80
		real_server 10.0.0.201 80 {
			weight 1
			HTTP_GET {
				url {
					path /monitor.html
					status_code 200
				}
				connect_timeout 1
				nb_get_retry 3
				delay_before_retry 1
			}
		}
		real_server 10.0.0.202 80 {
			weight 1
			HTTP_GET {
				url {
					path /
					status_code 200
				}
				connect_timeout 1
				nb_get_retry 3
				delay_before_retry 1
			}
		}
}
```

```powershell
#在后端服务器可以观察到健康检测日志
tail /var/log/nginx/access.log
```

### TCP监测

传输层检测：TCP_CHECK

```
TCP_CHECK {
	connect_ip <IP ADDRESS> 		#向当前RS的哪个IP地址发起健康状态检测请求
	connect_port <PORT> 			#向当前RS的哪个PORT发起健康状态检测请求
	bindto <IP ADDRESS> 			#发出健康状态检测请求时使用的源地址
	bind_port <PORT> 				#发出健康状态检测请求时使用的源端口
	connect_timeout <INTEGER> 		#客户端请求的超时时长, 等于haproxy的timeout server   
}
```

范例：

```powershell
virtual_server 10.0.0.10 80 {
	delay_loop 6
	lb_algo wrr
	lb_kind DR
	#persistence_timeout 120   #会话保持时间
	protocol TCP
	sorry_server 127.0.0.1 80
	real_server 10.0.0.201 80 {
		weight 1
		TCP_CHECK {
			connect_timeout 5
			nb_get_retry 3
			delay_before_retry 3
			connect_port 80
		}
	}
	real_server 10.0.0.2 80 {
		weight 1
		TCP_CHECK {
			connect_timeout 5
			nb_get_retry 3
			delay_before_retry 3
			connect_port 80
		} 
	} 
}
```



## 实现单主的 LVS-DR 模式



<img src="D:\桌面\mage.md\笔记\5day-png\25单主的 LVS-DR 模式.png" alt="image-20250112123315791" style="zoom:50%;" />

```powershell
#10.0.0.201/10.0.0.201
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka1.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo wrr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 2
            TCP_CHECK {
                connect_timeout 5
                nb_get_retry 3
                delay_before_retry 3
                connect_port 80
            }
        }
}

#10.0.0.202/192.168.202
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka2.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo wrr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 2
            TCP_CHECK {
                connect_timeout 5
                nb_get_retry 3
                delay_before_retry 3
                connect_port 80
            }
        }
}


#10.0.0.100、10.0.0.101
安装nginx服务，作为后端服务器
[root@ubuntu2204 ~]#cat lvs_dr_rs.sh 
#!/bin/bash
#Author:wangxiaochun
#Date:2017-08-13
vip=10.0.0.10
mask='255.255.255.255'
dev=lo:1
#rpm -q httpd &> /dev/null || yum -y install httpd &>/dev/null
#service httpd start &> /dev/null && echo "The httpd Server is Ready!"
#echo "`hostname -I`" > /var/www/html/index.html

case $1 in
start)
    echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ifconfig $dev $vip netmask $mask #broadcast $vip up
    #route add -host $vip dev $dev
    echo "The RS Server is Ready!"
    ;;
stop)
    ifconfig $dev down
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
    echo "The RS Server is Canceled!"
    ;;
*) 
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;
esac

[root@ubuntu2204 ~]#bash lvs_dr_rs.sh start
```



## 实现双主的 LVS-DR 模式



<img src="D:\桌面\mage.md\笔记\5day-png\25双主的 LVS-DR 模式.png" alt="image-20250112123547628" style="zoom:50%;" />

两台keepalived+LVS，后端四台nginx服务器

```powershell
#10.0.0.201
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     2352136389@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ka1.example.com
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_mcast_group4 224.0.0.18
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state BACKUP
    interface eth1
    virtual_router_id 88
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
      10.0.0.20 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}
virtual_server 10.0.0.20 80{
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.102 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.103 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}

#10.0.0.202
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}

[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 88
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
      10.0.0.20 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
}
virtual_server 10.0.0.20 80 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.102 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.103 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}
```



## 基于VRRP Script实现其它应用的高可用性

keepalived利用 VRRP Script 技术，可以调用外部的辅助脚本进行资源监控，并根据监控的结果实现优 先动态调整，从而实现其它应用的高可用性功能

### 参考配置文件：

```powershell
[root@ubuntu2404 conf.d]#cat /usr/local/src/keepalived-2.0.20/doc/samples/keepalived.conf.vrrp.localcheck 
! Configuration File for keepalived

vrrp_script chk_sshd {
       script "killall -0 sshd"        # cheaper than pidof
       interval 2                      # check every 2 seconds			#每2秒检查一次
       weight -4                       # default prio: -4 if KO			#默认优先级：如果 KO，则为 -4
       fall 2                          # require 2 failures for KO		#需要失败2次才能KO
       rise 2                          # require 2 successes for OK		#需要 2 次成功才能确定
}

vrrp_script chk_haproxy {
       script "killall -0 haproxy"     # cheaper than pidof
       interval 2                      # check every 2 seconds			#每2秒检查一次
}

vrrp_script chk_http_port {
       script "</dev/tcp/127.0.0.1/80" # connects and exits				#连接并退出
       interval 1                      # check every second				#每秒检查一次
       weight -2                       # default prio: -2 if connect fails	#默认优先级：-2（如果连接失败）
}

vrrp_script chk_https_port {
       script "</dev/tcp/127.0.0.1/443"
       interval 1
       weight -2
}

vrrp_script chk_smtp_port {
       script "</dev/tcp/127.0.0.1/25"
       interval 1
       weight -2
}

vrrp_instance VI_1 {
    interface eth0
    state MASTER
    virtual_router_id 51
    priority 100
    virtual_ipaddress {
        192.168.200.18/25
    }
    track_interface {
       eth1 weight 2   # prio = +2 if UP
       eth2 weight -2  # prio = -2 if DOWN
       eth3            # no weight, fault if down
    }
    track_script {
       chk_sshd                # use default weight from the script
       chk_haproxy weight 2    # +2 if process is present
       chk_http_port
       chk_https_port
       chk_smtp_port
    }
}

vrrp_instance VI_2 {
    interface eth1
    state MASTER
    virtual_router_id 52
    priority 100
    virtual_ipaddress {
        192.168.201.18/26
    }
    track_interface {
       eth0 weight 2   # prio = +2 if UP
       eth2 weight -2  # prio = -2 if DOWN
       eth3            # no weight, fault if down
    }
    track_script {
       chk_haproxy weight 2
       chk_http_port
       chk_https_port
       chk_smtp_port
    }
}

vrrp_instance VI_3 {
    interface eth0
    virtual_router_id 53
    priority 100
    virtual_ipaddress {
        192.168.200.19/27
    }
}

vrrp_instance VI_4 {
    interface eth1
    virtual_router_id 54
    priority 100
    virtual_ipaddress {
        192.168.201.19/28
    }
}

vrrp_instance VI_5 {
    state MASTER
    interface eth0
    virtual_router_id 55
    priority 100
    virtual_ipaddress {
        192.168.200.20/27
    }
}


vrrp_instance VI_6 {
    state MASTER
    interface eth0
    virtual_router_id 56
    priority 100
    virtual_ipaddress {
        192.168.200.21/27
    }
}

vrrp_instance VI_7 {
    state MASTER
    interface eth0
    virtual_router_id 57
    priority 100
    virtual_ipaddress {
        192.168.200.22/27
    }
}

vrrp_instance VI_8 {
    state MASTER
    interface eth0
    virtual_router_id 58
    priority 100
    virtual_ipaddress {
        192.168.200.23/27
    }
}

vrrp_instance VI_9 {
    state MASTER
    interface eth0
    virtual_router_id 59
    priority 100
    virtual_ipaddress {
        192.168.200.24/27
    }
}
```

### VRRP Script 配置

**分两步实现：** 

- 定义脚本 

  `vrrp_script`：自定义资源监控脚本，vrrp实例根据脚本返回值，公共定义，可被多个实例调用，定 义在vrrp实例之外的独立配置块，一般放在`global_defs`设置块之后,是和`global_defs`平级的语句块 

  通常此脚本用于监控指定应用的状态。一旦发现应用的状态异常，则触发对MASTER节点的权重减 至低于SLAVE节点，从而实现 VIP 切换到 SLAVE 节点

  **当 keepalived_script 用户存在时,会以此用户身份运行脚本,否则默认以root运行脚本**

  **注意: 此定义脚本的语句块一定要放在下面调用此语句`vrrp_instance`语句块的前面**

  ```powershell
  vrrp_script <SCRIPT_NAME> {
  	script <STRING>|<QUOTED-STRING>   	#此脚本返回值为非0时，会触发下面OPTIONS执行
  	OPTIONS 
  }
  ```

- 调用脚本

  `track_script`：调用vrrp_script定义的脚本去监控资源，定义在VRRP实例之内，调用事先定义的 `vrrp_script`

  ```powershell
  track_script {
  	SCRIPT_NAME_1
  	SCRIPT_NAME_2
  }
  ```

  

### 定义 VRRP script

```powershell
vrrp_script <SCRIPT_NAME> { 		#定义一个检测脚本，在global_defs 之外配置
	script <STRING>|<QUOTED-STRING> #shell命令或脚本路径
	interval <INTEGER> 				#间隔时间，单位为秒，默认1秒
	timeout <INTEGER> 				#超时时间
	weight <INTEGER:-254..254> 		#默认为0,如果设置此值为负数，当上面脚本返回值为非0时，会将此值与本节点权重相加可以降低本节点权重，即表示fall. 如果是正数，当脚本返回值为0，会将此值与本节点权重相加可以提高本节点权重，即表示 rise.通常使用负值
	fall <INTEGER> 					#执行脚本连续几次都失败,则转换为失败，建议设为2以上
	rise <INTEGER> 					#执行脚本连续几次都成功，把服务器从失败标记为成功
	user USERNAME [GROUPNAME] 		#执行监测脚本的用户或组      
	init_fail 						#设置默认标记为失败状态，监测成功之后再转换为成功状态
}
```

### 调用 VRRP script

```powershell
vrrp_instance VI_1 {
	...
	track_script {
		<SCRIPT_NAME>
	}
}
```

## 利用脚本实现主从角色切换

<img src="D:\桌面\mage.md\笔记\5day-png\25利用脚本实现主从角色切换.png" alt="image-20250112151955075" style="zoom:50%;" />

```powershell
#10.0.0.201：
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka1.example.com
   vrrp_mcast_group4 224.0.0.18
}

vrrp_script check_down {
    script "[ ! -f /etc/keepalived/down ]"		#/etc/keepalived/down存在时返回非0，触发权重-30
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth0
    }
    track_script {
        check_down								#调用前面定义的脚本
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
}

#10.0.0.202
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka2.example.com
}
include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
      10.0.0.10/24 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
}
```



## 实现单主模式的 Nginx 反向代理的高可用

<img src="D:\桌面\mage.md\笔记\5day-png\25单主模式的 Nginx 反向代理的高可用.png" alt="image-20250112172258664" style="zoom:50%;" />

```powershell
#10.0.0.201\10.0.0.202
#两台主机配置nginx反向代理
[root@ubuntu2404 conf.d]#cat webserver.conf 
upstream webserver {
    server 10.0.0.100:80 weight=1;
    server 10.0.0.101:80 weight=1;
    }
server {
    listen 80;
    server_name www.hzk.com;
    location /{
    proxy_pass http://webserver;
    }
}

10.0.0.201
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka1.example.com
   vrrp_mcast_group4 224.0.0.18
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"				#调用脚本路径
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_nginx
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/check_nginx.sh 
#!/bin/bash
/usr/bin/killall -0 nginx || systemctl restart nginx				#如果 nginx 进程不存在（killall -0 nginx 返回非零），则使用 systemctl restart nginx 重启服务。


#10.0.0.202
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka2.example.com
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"				#调用脚本路径
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_nginx
    }
    unicast_src_ip 192.168.10.202
    unicast_peer{
        192.168.10.201
    }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/check_nginx.sh
#!/bin/bash
/usr/bin/killall -0 nginx || systemctl restart nginx				#如果 nginx 进程不存在（killall -0 nginx 返回非零），则使用 systemctl restart nginx 重启服务。
```

## 实现双主模式 Nginx 反向代理的高可用

<img src="D:\桌面\mage.md\笔记\5day-png\25双主模式 Nginx 反向代理的高可用.png" alt="image-20250112175513885" style="zoom:50%;" />

```powershell
#10.0.0.201,10.0.0.202
#两台主机配置nginx反向代理
[root@ubuntu2404 conf.d]#cat /etc/nginx/conf.d/webserver.conf 
upstream webserver1 {
    server 10.0.0.100:80 weight=1;
    server 10.0.0.101:80 weight=1;
    }
upstream webserver2 {
    server 10.0.0.102:80 weight=1;
    server 10.0.0.103:80 weight=1;
    }
server {
    listen 80;
    server_name www.hzk.com;
    location /{
    proxy_pass http://webserver1;
    }
}
server {
    listen 80;
    server_name www.zk.com;
    location /{
    proxy_pass http://webserver2;
    }
}

#10.0.0.10/10.0.0.201
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf
global_defs {
   router_id ka1.example.com
   vrrp_mcast_group4 224.0.0.18
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf      
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_nginx
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state BACKUP
    interface eth1
    virtual_router_id 88
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
      10.0.0.20/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_nginx
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}

#10.0.0.20/10.0.0.202
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka2.example.com
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_nginx
    }
    unicast_src_ip 192.168.10.202
    unicast_peer{
        192.168.10.201
    }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 88
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
        10.0.0.20/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_nginx
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
}
```

## 实现 HAProxy 高可用



<img src="D:\桌面\mage.md\笔记\5day-png\25HAProxy 高可用" alt="image-20250112183844322" style="zoom:50%;" />

```powershell
#10.0.0.201
[root@ubuntu2404 conf.d]#cat /etc/haproxy/haproxy.cfg 
listen web_http
    bind 10.0.0.10:80
    server web1 10.0.0.100:80 check
    server web2 10.0.0.101:80 check
listen stats
    mode http
    bind 10.0.0.202:9999
    stats enable
    log global
    stats uri     /haproxy-status
    stats auth   haadmin:123456
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka1.example.com
   vrrp_mcast_group4 224.0.0.18
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_haproxy
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/check_haproxy.sh 
#!/bin/bash
/usr/bin/killall -0 haproxy || systemctl restart haproxy 


#10.0.0.202
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka2.example.com
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_haproxy
    }
    unicast_src_ip 192.168.10.202
    unicast_peer{
        192.168.10.201
    }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/check_haproxy.sh 
#!/bin/bash
/usr/bin/killall -0 haproxy || systemctl restart haproxy 
```



## 实现 MySQL 双主模式的高可用

![image-20250113190741542](D:\桌面\mage.md\笔记\5day-png\25MySQL 双主模式的高可用.png)

```powershell
#配置mysql双主模式
10.0.0.100
mkdir -p /data/mysql/logbin
chown mysql:mysql -R /data/mysql
vim /etc/apparmor.d/usr.sbin.mysqld
# Allow data files dir access
...
  /data/mysql/logbin/ rw,
  /data/mysql/logbin/** rw,
...
systemctl reload apparmor
vim /etc/mysql/mysql.conf.d/mysqld.cnf
...
bind-address        = 0.0.0.0
mysqlx-bind-address = 0.0.0.0
server-id = 100
log_bin = /data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
...
systemctl restart mysql.service

10.0.0.101
mkdir -p /data/mysql/logbin
chown mysql:mysql -R /data/mysql
vim /etc/apparmor.d/usr.sbin.mysqld
# Allow data files dir access
...
  /data/mysql/logbin/ rw,
  /data/mysql/logbin/** rw,
...
systemctl reload apparmor
vim /etc/mysql/mysql.conf.d/mysqld.cnf
...
bind-address        = 0.0.0.0
mysqlx-bind-address = 0.0.0.0
server-id = 101
log_bin = /data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
systemctl restart mysql.service

10.0.0.100
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       849 | No        |
+------------------+-----------+-----------+
1 row in set (0.00 sec)
mysql> create user repluser@'10.0.0.%' identified by '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> grant replication slave on *.* to repluser@'10.0.0.%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>  CHANGE MASTER TO 
    ->       MASTER_HOST='10.0.0.101',
    ->       MASTER_USER='repluser',
    ->       MASTER_PASSWORD='123456',
    ->       MASTER_PORT=3306,
    ->       MASTER_LOG_FILE='mysql-bin.000001', 
    ->       MASTER_LOG_POS=1038;
Query OK, 0 rows affected, 9 warnings (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.07 sec)

mysql> show slave status\G

10.0.0.101
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |      1038 | No        |
+------------------+-----------+-----------+
1 row in set (0.00 sec)
mysql> create user repluser@'10.0.0.%' identified by '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> grant replication slave on *.* to repluser@'10.0.0.%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> CHANGE MASTER TO 
    ->       MASTER_HOST='10.0.0.100',
    ->       MASTER_USER='repluser',
    ->       MASTER_PASSWORD='123456',
    ->       MASTER_PORT=3306,
    ->       MASTER_LOG_FILE='mysql-bin.000001', 
    ->       MASTER_LOG_POS=849;
Query OK, 0 rows affected, 9 warnings (0.01 sec)

mysql> 
mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> show slave status\G


#实现MySQL的健康性检测脚本1
vim /etc/keepalived/check_mysql.sh
#!/bin/bash
slave_is=( $(mysql -uroot -p123456 -h10.0.0.18 -e "show slave status\G" | grep "Slave_.*_Running:" | awk '{print $2}') )
if [ "${slave_is[0]}" = "Yes" -a "${slave_is[1]}" = "Yes" ];then
	exit 0
else
	exit 1
fi
#实现MySQL的健康性检测脚本2
[root@ka1 ~]#vi /etc/keepalived/check_mysql.sh
mysqladmin -uroot -p123456  ping &> /dev/null
#实现MySQL的健康性检测脚本3
[root@ka1 ~]#vi /etc/keepalived/check_mysql.sh
mysql  -uroot -p123456 -e 'status' &> /dev/null
#实现MySQL的健康性检测脚本4
[root@ka1 ~]#vi /etc/keepalived/check_mysql.sh
systemctl is-active mariadb &> /dev/null
#这四种健康性检查脚本

10.0.0.201
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka1.example.com
   vrrp_mcast_group4 224.0.0.18
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_mysql
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}

10.0.0.202
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka2.example.com
}

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 1
    weight -30
    fall 3
    rise 2
    timeout 2
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.202
    unicast_peer{
        192.168.10.201
    }
}
```





## 实现LVS-NAT模型的双vip

<img src="D:\桌面\mage.md\笔记\5day-png\25实现LVS-NAT模型的双vip.png" alt="image-20250113191237718" style="zoom:50%;" />

```powershell
10.0.0.100/10.0.0.101
vim /etc/sysctl.conf
sysctl -p
net.ipv4.ip_forward = 1

10.0.0.201
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka1.example.com
   vrrp_mcast_group4 224.0.0.18
}

vrrp_sync_group VG_1{
    group{
        VI_1
        VI_2
    }
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_mysql
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo rr
        lb_kind NAT
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster2.conf
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 88
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
      192.168.10.10/24 dev eth1 label eth1:0
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}

10.0.0.202
[root@ubuntu2404 conf.d]#cat /etc/keepalived/keepalived.conf
global_defs {
   router_id ka2.example.com
}

vrrp_sync_group VG_1{
    group{
        VI_1
        VI_2
    }
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.202
    unicast_peer{
        192.168.10.201
    }
}
virtual_server 10.0.0.10 80 {
        delay_loop 3
        lb_algo rr
        lb_kind NAT
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}
[root@ubuntu2404 conf.d]#cat /etc/keepalived/conf.d/cluster2.conf 
vrrp_instance VI_2 {
    state BACKUP
    interface eth1
    virtual_router_id 88
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234567
    }
    virtual_ipaddress {
        192.168.10.10/24 dev eth1 label eth1:0
    }
    unicast_src_ip 192.168.202
    unicast_peer{
        192.168.10.201
    }
}
```



## 实现单主的 LVS-DR 模式，利用FWM绑定成多个服务为一个集群服务

<img src="D:\桌面\mage.md\笔记\5day-png\25单主的 LVS-DR 模式，利用FWM绑定成多个服务为一个集群服务.png" alt="image-20250113211938893" style="zoom:50%;" />

```powershell
后端服务配置HTTPS服务

10.0.0.201
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka1.example.com
   vrrp_mcast_group4 224.0.0.18
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    track_interface {
        eth1
    }
    track_script {
        check_mysql
    }
    unicast_src_ip 192.168.10.201
    unicast_peer{
        192.168.10.202
    }
}
virtual_server fwmark 6 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}

10.0.0.202
[root@ubuntu2404 ~]#cat /etc/keepalived/keepalived.conf 
global_defs {
   router_id ka2.example.com
}

include /etc/keepalived/conf.d/*.conf
[root@ubuntu2404 ~]#cat /etc/keepalived/conf.d/cluster1.conf 
vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 66
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        10.0.0.10/24 dev eth0 label eth0:0
    }
    unicast_src_ip 192.168.10.202
    unicast_peer{
        192.168.10.201
    }
}
virtual_server fwmark 6 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 10.0.0.100 80 {
            weight 1
            HTTP_GET {
                url {
                    path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
        real_server 10.0.0.101 80 {
            weight 1
            TCP_CHECK {
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
                connect_port 80
            }
        }
}
```

