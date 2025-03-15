# 24、Linux Virtual Server（lvs）

## 集群Cluster和分布式

系统性能扩展方式：

Scale UP：垂直扩展，向上扩展,增强，性能更强的计算机运行同样的服务

Scale Out：水平扩展，向外扩展,增加设备，并行地运行多个服务调度分配问题，Cluster

### 集群

Cluster：集群,为解决某个特定问题将多台计算机组合起来形成的单个系统

Cluster 分为三种：

- LB：Load Balancing，负载均衡，多个主机组成，每个主机只承担一部分访问请求

- HA：High Availiablity，高可用，避免SPOF（single Point Of failure）单点故障

  MTBF:Mean Time Between Failure 平均无故障时间，正常时间

  MTTR:Mean Time To Restoration（ repair）平均恢复前时间，故障时间

  A = MTBF /（MTBF+MTTR） (0,1)：99%,99.5%,99.9%,99.99%,99.999%

  **SLA**：服务等级协议（简称：SLA，全称：service level agreement）。是指在一定开销下为保障服务的性能和可用性，服务提供商与用户间定义的一种双方认可的协定。通常这个开销是驱动提供服务质量的主要因素。在常规的领域中，总是设定所谓的三个9，四个9,N个9 等来进行表示，当没有达到这种水平的时候，就会有一些列的惩罚措施，而运维最主要的目标就是达成这种服务水平。

  停机时间又分为两种，一种是计划内停机时间，一种是计划外停机时间，而运维则主要关注计划外停机时间。

  ```bash
  1年 = 365天 = 8760小时
  90 = (1-90%)*365=36.5天
  99 = 8760 * 1% = 87.6小时
  99.9 = 8760 * 0.1% = 8760 * 0.001 = 8.76小时
  99.99 = 8760 * 0.0001 = 0.876小时 = 0.876 * 60 = 52.6分钟
  99.999 = 8760 * 0.00001 = 0.0876小时 = 0.0876 * 60 = 5.26分钟
  99.9999= (1-99.9999%)*365*24*60*60=31秒
  ```

- HPC：High-performance computing，高性能 http://www.top500.org

### 分布式

**分布式常见应用**

- 分布式应用-服务按照功能拆分，使用微服务

- 分布式静态资源--静态资源放在不同的存储集群上

- 分布式数据和存储--使用key-value缓存系统

- 分布式计算--对特殊业务使用分布式计算，比如Hadoop集群

分布式存储： Ceph，GlusterFS，FastDFS，MogileFS

分布式计算：hadoop，Spark

### 基于工作的协议层次划分

- 四层，传输层（通用）：DNAT 和 DPORT

  LVS:

  nginx：stream

  haproxy：mode tcp

- 七层， 应用层（专用）：针对特定协议，常称为 proxy server

  http：nginx, httpd, haproxy(mode http), ...

  fastcgi：nginx, httpd, ...

  mysql：mysql-proxy, mycat...

### 负载均衡的会话保持

- session sticky：同一用户调度固定服务器

  Source IP：LVS sh算法（对某一特定服务而言）

  Cookie

- session replication：每台服务器拥有全部session

  session multicast cluster

- session server：专门的session服务器

  Redis,Memcached 

### HA高可用集群实现

keepalived：vrrp协议

Ais：应用接口规范

- heartbeat

- cman+rgmanager(RHCS)

- coresync_pacemaker



## LVS

### **LVS** 工作原理

VS根据请求报文的目标IP和目标协议及端口将其调度转发至某RS，根据调度算法来挑选RS。LVS是内核级功能，工作在`INPUT`链的位置，将发往`INPUT`的流量进行“处理

### lvs集群架构

VS：Virtual Server，Director Server(DS), Dispatcher(调度器)，Load Balancer

RS：Real Server(lvs), upstream server(nginx), backend server(haproxy)

Client: 客户端

CIP：Client IP

VIP：Virtual server IP VS外网的IP

DIP：Director IP VS内网的IP

RIP：Real server IP 

访问流程：CIP <--> VIP == DIP <--> RIP

<img src="5day-png\24lvs流程.png" style="zoom:50%;" />

## LVS 集群的工作模式

- vs-nat：修改请求报文的目标IP,多目标IP的DNAT 

- lvs-dr：操纵封装新的MAC地址 

- lvs-tun：在原请求IP报文之外新加一个IP首部 

- lvs-fullnat：修改请求报文的源和目标IP,默认内核不支持,需要自行开发

###  LVS 的 NAT 模式

lvs-nat：本质是多目标IP的DNAT，通过将请求报文中的目标地址和目标端口修改为某挑出的RS的RIP和 PORT实现转发 

（1）RIP和DIP应在同一个IP网络，且应使用私网地址；RS的网关要指向DIP 

（2）请求报文和响应报文都必须经由Director转发，Director易于成为系统瓶颈 

（3）支持端口映射，可修改请求报文的目标PORT 

（4）VS必须是Linux系统，RS可以是任意OS系统

<img src="5day-png\24lvsnat.png" style="zoom:50%;" />

### LVS 的 DR 模式

<img src="5day-png\24lvsDR.png" style="zoom:33%;" />

LVS-DR：Direct Routing，直接路由，LVS默认模式,应用最广泛,通过为请求报文重新封装一个MAC首部 进行转发，源MAC是DIP所在的接口的MAC，目标MAC是某挑选出的RS的RIP所在接口的MAC地址；源 IP/PORT，以及目标IP/PORT均保持不变

<img src="5day-png\24lvsDR1.png" style="zoom:50%;" />

DR模式的特点： 

1. Director和各RS都配置有VIP 

2. 确保前端路由器将目标IP为VIP的请求报文发往Director 

   - 在前端网关做静态绑定VIP和Director的MAC地址 

   - 在RS上使用arptables工具

     ```
     arptables -A IN -d $VIP -j DROP
     arptables -A OUT -s $VIP -j mangle --mangle-ip-s $RIP
     ```

   - 在RS上修改内核参数以限制arp通告及应答级别

     ```
     /proc/sys/net/ipv4/conf/all/arp_ignore
     /proc/sys/net/ipv4/conf/all/arp_announce
     ```

3. RS的RIP可以使用私网地址，也可以是公网地址；RIP与DIP在同一IP网络；RIP的网关不能指向 DIP，以确保响应报文不会经由Director 
4. RS和Director要在同一个物理网络 
5. 请求报文要经由Director，但响应报文不经由Director，而由RS直接发往Client 
6.  不支持端口映射（端口不能修改） 
7. 无需开启 ip_forward 
8. RS可使用大多数OS系统

### LVS 的 TUN 模式

<img src="5day-png\24tun模式.png" alt="image-20250109100419438" style="zoom: 33%;" />

转发方式：不修改请求报文的IP首部（源IP为CIP，目标IP为VIP），而在原IP报文之外再封装一个IP首部 （源IP是DIP，目标IP是RIP），将报文发往挑选出的目标RS；RS直接响应给客户端（源IP是VIP，目标IP 是CIP）

<img src="5day-png\24tun模式1.png" alt="image-20250109100602009" style="zoom:50%;" />

**TUN模式特点**： 

1. RIP和DIP可以不处于同一物理网络中，RS的网关一般不能指向DIP,且RIP可以和公网通信。也就是 说集群节点可以跨互联网实现。DIP, VIP, RIP可以是公网地址
2. RealServer的tun接口上需要配置VIP地址，以便接收director转发过来的数据包，以及作为响应的报文源IP
3. Director转发给RealServer时需要借助隧道，隧道外层的IP头部的源IP是DIP，目标IP是RIP，而 RealServer响应给客户端的IP头部是根据隧道内层的IP头分析得到的，源IP是VIP，目标IP是CIP
4. 请求报文要经由Director，但响应不经由Director,响应由RealServer自己完成
5. 不支持端口映射
6. RS的OS须支持隧道功能



应用场景：

```
一般来说，TUN模式常会用来负载调度缓存服务器组，这些缓存服务器一般放置在不同的网络环境，可以就近折返给客户端。在请求对象不在Cache服务器本地命中的情况下，Cache服务器要向源服务器发送请求，将结果取回，最后将结果返回给用户。
LAN环境一般多采用DR模式，WAN环境虽然可以用TUN模式，但是一般在WAN环境下，请求转发更多的被haproxy/nginx/DNS等实现。因此，TUN模式实际应用的很少,跨机房的应用一般专线光纤连接或DNS调度
```

### LVS 的 FULLNAT 模式

<img src="5day-png\24fullnat.png" alt="image-20250109101907670" style="zoom:50%;" />

通过同时修改请求报文的源IP地址和目标IP地址进行转发 

CIP --> DIP  

VIP --> RIP  

fullnat模式特点： 

1. VIP是公网地址，RIP和DIP是私网地址，且通常不在同一IP网络；因此，RIP的网关一般不会指向 DIP 
2. RS收到的请求报文源地址是DIP，因此，只需响应给DIP；但Director还要将其发往Client 
3. 请求和响应报文都经由Director 
4. 相对NAT模式，可以更好的实现LVS-RealServer间跨VLAN通讯 
5. 支持端口映射 注意：此类型kernel默认不支持



### 对比

lvs-nat与lvs-fullnat： 

- 请求和响应报文都经由Director 

- lvs-nat：RIP的网关要指向DIP 

- lvs-fullnat：RIP和DIP未必在同一IP网络，但要能通信

lvs-dr与lvs-tun： 

- 请求报文要经由Director，但响应报文由RS直接发往Client 

- lvs-dr：通过封装新的MAC首部实现，通过MAC网络转发 

- lvs-tun：通过在原IP报文外封装新IP头实现转发，支持远距离通信

**总结 **

- NAT 多目标的DNAT，四层，支持端口修改，请求报文和响应报文都要经过LVS 

- DR 默认模式，二层，只修改MAC，不支持端口修改，性能好，LVS负载比小，LVS和RS并在同一 网段，请求报文经过LVS，响应报文不经过LVS 

- TUNNEL 三层，添加一个新的IP头，支持LVS和RS并在不在同一网段，不支持端口修改，请求报文 经过LVS，响应报文不经过LVS 

- FULLNAT 多目标的SNAT+DNAT，四层，支持端口修改，请求报文和响应报文都要经过LVS



在使用 LVS（Linux Virtual Server）时，可以采用不同的转发模式实现负载均衡。以下是 NAT、DR（Direct Routing，RS模式）、TUN（IP Tunneling）、FULLNAT 的主要特点和对比：

------

#### 1. **NAT 模式（Network Address Translation）**

- 原理：
  - 负载均衡器（LB）通过修改报文的目标 IP 和端口，将请求转发到后端服务器（Real Server，RS）。
  - 后端服务器响应的流量回到负载均衡器，负载均衡器修改源地址后再发送给客户端。
- 优点：
  - 配置简单，无需修改后端服务器配置。
  - 适合小规模服务。
- 缺点：
  - 所有流量都经过负载均衡器，负载均衡器成为性能瓶颈。
  - 增加了服务器的网络负担。
- 适用场景：
  - 小规模的内网服务，负载不高。

------

#### 2. **DR 模式（Direct Routing，RS模式）**

- 原理：
  - 负载均衡器只修改报文的 MAC 地址，不改变 IP 地址。
  - 客户端发来的请求通过 LB 转发给后端服务器，后端服务器直接响应客户端（流量回程不经过 LB）。
- 优点：
  - 负载均衡器压力小，因为响应流量不经过它。
  - 高效，适合高性能场景。
- 缺点：
  - 后端服务器需要配置真实 IP 和虚拟 IP，共享 VIP。
  - 后端服务器必须和负载均衡器在同一网段。
  - 配置复杂，特别是在 ARP 广播管理上容易出问题。
- 适用场景：
  - 高性能场景，例如高并发 Web 服务。

------

#### 3. **TUN 模式（IP Tunneling）**

- 原理：
  - 负载均衡器将请求封装在 IP 隧道中（通过 GRE 或 IPIP 隧道），然后转发给后端服务器。
  - 后端服务器通过隧道接收请求，直接响应客户端（不经过 LB）。
- 优点：
  - 后端服务器可以分布在不同的网段，不要求和负载均衡器在同一网段。
  - 负载均衡器只处理入口流量，减少了负载。
- 缺点：
  - 后端服务器需要支持 IP 隧道（Linux 内核 2.2 以上支持）。
  - 配置复杂性较高。
- 适用场景：
  - 分布式后端服务器，不同网络区域部署的系统。

------

#### 4. **FULLNAT 模式（Full Network Address Translation）**

- 原理：
  - 负载均衡器修改报文的源 IP 和目标 IP 地址，同时支持后端服务器在不同网段部署。
  - 响应流量必须回到负载均衡器。
- 优点：
  - 后端服务器可以部署在任意网段，灵活性高。
  - 不需要后端服务器额外配置，只需配置负载均衡器。
- 缺点：
  - 负载均衡器处理所有流量，成为瓶颈。
  - 相比 NAT 模式复杂，性能略低。
- 适用场景：
  - 跨网段部署、多数据中心等灵活性要求高的场景。

------

#### 对比总结

| **模式** | **流量路径**                    | **性能** | **配置难度** | **适用场景**             |
| -------- | ------------------------------- | -------- | ------------ | ------------------------ |
| NAT      | 全部流量经过 LB                 | 中       | 简单         | 小规模服务，内网负载均衡 |
| DR       | 请求流量经过 LB，响应直发客户端 | 高       | 较复杂       | 高性能、高并发场景       |
| TUN      | 请求隧道转发，响应直发客户端    | 高       | 较复杂       | 分布式、多网段后端       |
| FULLNAT  | 全部流量经过 LB                 | 中       | 中等         | 跨网段、多数据中心部署   |



##  LVS 调度算法

### 静态方法

仅根据算法本身进行调度 

1. 轮询算法RR：roundrobin，轮询,较常用 
2. 加权轮询算法WRR：Weighted RR，较常用 
3. 源地址哈希算法SH：Source Hashing，实现session sticky，源IP地址hash；将来自于同一个IP地址 的请求始终发往第一次挑中的RS，从而实现会话绑定
4. 目标地址哈希算法DH：Destination Hashing；目标地址哈希，第一次轮询调度至RS，后续将发往 同一个目标地址的请求始终转发至第一次挑中的RS，典型使用场景是正向代理缓存场景中的负载均衡, 如: Web缓存

### 动态方法

主要根据每RS当前的负载状态及调度算法进行调度Overhead=value 较小的RS将被调

1. 最少连接算法LC：least connections 适用于长连接应用

   ```
   Overhead=activeconns*256+inactiveconns
   ```

2. 加权最少连接算法WLC：Weighted LC，默认调度方法,较常用

   ```
   Overhead=(activeconns*256+inactiveconns)/weight
   ```

3. 最短期望延迟算法SED：Shortest Expection Delay，初始连接高权重优先,只检查活动连接,而不考虑 非活动连接

   ```
   Overhead=(activeconns+1)*256/weight
   ```

4. 最少队列算法NQ：Never Queue，第一轮均匀分配，后续SED

5. 基于局部的最少连接算法LBLC：Locality-Based LC，动态的DH算法，使用场景：根据负载状态实现 正向代理,实现Web Cache等

6. 带复制的基于局部的最少连接算法LBLCR：LBLC with Replication，带复制功能的LBLC，解决LBLC 负载不均衡问题，从负载重的复制到负载轻的RS,,实现Web Cache等



```
overhead：开销
activeconns：表示当前服务器（或后端真实服务器，RS）处理的 活动连接数。即当前服务器正在处理的活跃请求数量。活跃连接数，即当前服务器正在处理的连接数。活跃连接通常指的是正在进行数据交换的连接，通常会占用较多的系统资源（如 CPU、内存等）。
inactiveconns：非活跃连接数，即当前已建立但没有进行数据交换的连接。通常这些连接不会占用太多的系统资源，但它们仍然会消耗一些资源，尤其是在连接保持的时间较长时。
weight：在加权负载均衡中，weight 代表了每台服务器的 "权重"。权重越大，表示该服务器能够处理更多的流量。负载均衡器会根据权重将流量分配给不同的服务器，权重高的服务器会承担更多的请求。

```

### 内核版本 4.15 版本后新增调度算法：FO和OVF

FO（Weighted Fail Over）调度算法,在此FO算法中，遍历虚拟服务所关联的真实服务器链表，找到还未 过载（未设置IP_VS_DEST_F_OVERLOAD标志）的且权重最高的真实服务器，进行调度,属于静态算法 

OVF（Overflow-connection）调度算法，基于真实服务器的活动连接数量和权重值实现。将新连接调度 到权重值最高的真实服务器，直到其活动连接数量超过权重值，之后调度到下一个权重值最高的真实服 务器,在此OVF算法中，遍历虚拟服务相关联的真实服务器链表，找到权重值最高的可用真实服务器。属于动态算法 



一个可用的真实服务器需要同时满足以下条件： 

- 未过载（未设置IP_VS_DEST_F_OVERLOAD标志） 

- 真实服务器当前的活动连接数量小于其权重值 

- 其权重值不为零



## LVS 相关软件

### ipvsadm

Unit File: ipvsadm.service

主程序：/usr/sbin/ipvsadm 

规则保存工具：/usr/sbin/ipvsadm-save 

规则重载工具：/usr/sbin/ipvsadm-restore

配置文件：/etc/sysconfig/ipvsadm-config 

ipvs调度规则文件：/etc/sysconfig/ipvsadm



### ipvsadm 命令

**ipvsadm核心功能：** 

- 集群服务管理：增、删、改 

- 集群服务的RS管理：增、删、改 

- 查看 

**ipvsadm 工具用法：**

```powershell
#管理集群服务
增加修改
ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]] [-M netmask] [--pe persistence_engine] [-b sched-flags]
ipvsadm -D -t|u|f service-address 	#删除
ipvsadm –C 			#清空
ipvsadm –R 			#重载,相当于ipvsadm-restore
ipvsadm -S [-n] 	#保存,相当于ipvsadm-save

#管理集群中的RS
增加修改
ipvsadm -a|e -t|u|f service-address -r server-address [-g|i|m] [-w weight] 
删除
ipvsadm -d -t|u|f service-address -r server-address
查看
ipvsadm -L|l [options]
清除所有虚拟服务
ipvsadm -Z [-t|u|f service-address]

-t service-address：指定虚拟服务（VIP）的 IP 地址和端口，格式为 <ip>:<port>。例如 10.0.0.1:80。
-u service-address：指定 UDP 类型的虚拟服务，类似于 -t，但使用 UDP 协议。
-f service-address：指定 FULLNAT 类型的虚拟服务，这通常用于需要全程进行 NAT（完全网络地址转换）的负载均衡场景。

-s 用来指定负载均衡的调度算法。常见的调度算法包括：
	rr：轮询（Round Robin）算法。
	wrr：加权轮询（Weighted Round Robin）算法。
	lc：最少连接（Least Connections）算法。
	wlc：加权最少连接（Weighted Least Connections）算法。
	lblc：基于负载的最少连接（Load Balancing Least Connections）算法。
	lblcr：基于负载的最少连接轮询（Load Balancing Least Connections with Round Robin）算法。

-p 用来指定 持久性超时时间。该选项设置虚拟服务的连接持久性，通常是与客户端会话相关的持续时间。单位是秒。
	如果不指定 timeout，则会使用默认的持久性超时。

-M 用来设置虚拟服务的 子网掩码。它用来确定目标服务的子网范围。一般用于当负载均衡的虚拟服务是面向子网时，指定网段的掩码。

--pe 用来指定持久性引擎（persistence engine），即持久会话的处理方式。持久性引擎用于确保同一客户端的后续请求始终被转发到相同的后端服务器。
常见的持久性引擎包括：
	source：基于源 IP 地址的持久性。
	dest：基于目标 IP 地址的持久性。
	both：同时基于源和目标 IP 地址的持久性。
	none：不使用持久性。
	
-b 用来指定调度算法的标志（flags）。不同的调度算法可能有不同的附加标志，可以用来定制负载均衡的行为。
```



#### **管理集群服务：增、改、删**

**增、修改：**

```bash
ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]]
```

```powershell
service-address：
-t|u|f：
 	-t: TCP协议的端口，VIP:TCP_PORT 如: -t 10.0.0.100:80
    -u: UDP协议的端口，VIP:UDP_PORT
    -f：firewall MARK，标记，一个数字       
[-s scheduler]：指定集群的调度算法，默认为wlc
```



**删除：**

```bash
ipvsadm -D -t|u|f service-address
```



#### **管理集群上的RS：增、改、删**

**增、改：**

```bash
ipvsadm -a|e -t|u|f service-address -r server-address [-g|i|m] [-w weight]
```

**删：**

```bash
ipvsadm -d -t|u|f service-address -r server-address
```



```powershell
server-address：
     rip[:port] 如省略port，不作端口映射
选项：
lvs类型：
    -g: gateway, dr类型，默认
    -i: ipip, tun类型
    -m: masquerade, nat类型        
-w weight：权重
```



**清空定义的所有内容：**

```bash
ipvsadm -C
```

**清空计数器：**

```bash
ipvsadm -Z [-t|u|f service-address]
```

**查看**

```bash
ipvsadm -L|l [options]
```

```powershell
--numeric, -n：以数字形式输出地址和端口号
--exact：扩展信息，精确值     
--connection，-c：当前IPVS连接输出
--stats：统计信息
--rate ：输出速率信息
```



**ipvs规则：**

```
/proc/net/ip_vs
```

**ipvs连接：**

```
/proc/net/ip_vs_conn
```



**保存：建议保存至/etc/sysconfig/ipvsadm**

```bash
ipvsadm-save > /PATH/TO/IPVSADM_FILE
ipvsadm -S > /PATH/TO/IPVSADM_FILE
systemctl stop ipvsadm.service  	#会自动保存规则至/etc/sysconfig/ipvsadm
```

**重载：**

```bash
ipvsadm-restore < /PATH/FROM/IPVSADM_FILE
systemctl  start ipvsadm.service  	#会自动加载/etc/sysconfig/ipvsadm中规则
```



范例： Ubuntu系统保存规则和开机加载规则

```bash
#保存规则
#方法1
[root@ubuntu2204 ~]#service ipvsadm save
 * Saving IPVS configuration...                                                   
                                              [ OK ]
[root@ubuntu2204 ~]#cat /etc/ipvsadm.rules
# ipvsadm.rules
-A -t 192.168.10.100:80 -s wlc
-a -t 192.168.10.100:80 -r 10.0.0.7:80 -g -w 1
-a -t 192.168.10.100:80 -r 10.0.0.17:80 -g -w 1

#方法2
[root@ubuntu2004 ~]#ipvsadm-save -n > /etc/ipvsadm.rules
#开机自启
[root@ubuntu2004 ~]#vim /etc/default/ipvsadm
AUTO="true"
```

范例：红帽系统保存规则和开机加载规则

```bash
#保存规则
[root@rocky8 ~]#ipvsadm-save -n > /etc/sysconfig/ipvsadm
#开机自启
[root@rocky8 ~]#systemctl enable ipvsadm.service
```



### 防火墙标记

在 **LVS（Linux Virtual Server）** 中，防火墙标记（Firewall Mark，也称为 fwmark）是用来匹配数据包的一种机制，结合 `iptables` 和 IPVS，可以实现更灵活的负载均衡策略。

通过给特定的流量打上标记，IPVS 可以根据这些标记将流量分发到对应的虚拟服务（VIP）。这种方法特别适用于场景中需要对多种协议或端口的流量进行统一的负载均衡。

------

#### 防火墙标记的工作原理

1. 使用 **`iptables`** 对特定的流量（基于协议、IP 地址、端口等）打上标记。
2. 使用 **`ipvsadm`** 将标记与虚拟服务关联。
3. IPVS 根据标记，将流量分发到对应的后端真实服务器（RS）。

------

#### 配置步骤

##### 1. 给流量打标记

使用 `iptables` 的 **`-j MARK`** 规则来给匹配的流量设置防火墙标记。

```bash
iptables -t mangle -A PREROUTING -d $vip -p $proto -m multiport --dports $port1,$port2,… -j MARK --set-mark NUMBER
```

**示例：**

```
iptables -t mangle -A PREROUTING -d 10.0.0.10 -p tcp -m multiport --dports 80,443 -j MARK --set-mark 10
```

- **第一条规则**：给目标端口为 `80` 的流量设置标记为 `1`。

------

##### 2. 配置 IPVS 虚拟服务

使用 `ipvsadm` 命令将防火墙标记与虚拟服务绑定。

**命令格式：**

```bash
ipvsadm -A -f <fwmark> -s <scheduler>
```

**示例：**

```bash
ipvsadm -A -f 10 -s rr
```

- **第一条命令**：将防火墙标记 `1` 绑定到一个虚拟服务，使用轮询算法（`rr`）。

------

##### 3. 添加真实服务器（RS）

为每个虚拟服务添加后端真实服务器。

**命令格式：**

```bash
ipvsadm -a -f <fwmark> -r <real-server-ip>:<port> -m
```

**示例：**

```bash
ipvsadm -a -f 10 -r 10.0.0.201
ipvsadm -a -f 10 -r 10.0.0.202
```

------

##### 4. 检查配置

检查防火墙标记和 IPVS 配置是否正确。

**查看防火墙标记：**

```bash
iptables -t mangle -L -n -v
```

**查看 IPVS 配置：**

```bash
ipvsadm -L -n
```

------

#### 示例场景

假设有一个服务需要对 HTTP（端口 `80`）和 HTTPS（端口 `443`）流量进行负载均衡：

1. HTTP 流量转发到后端 `192.168.1.101:80` 和 `192.168.1.102:80`，使用轮询算法。
2. HTTPS 流量转发到后端 `192.168.1.101:443` 和 `192.168.1.102:443`，使用轮询算法。

**完整配置：**

```bash
[root@lvs ~]#iptables -t mangle -A PREROUTING -d 10.0.0.10 -p tcp -m 
multiport --dports 80,443 -j MARK --set-mark 10 
[root@lvs ~]#ipvsadm -C
[root@lvs ~]#ipvsadm -A -f 10 -s rr
[root@lvs ~]#ipvsadm -a -f 10 -r 10.0.0.201 -g
[root@lvs ~]#ipvsadm -a -f 10 -r 10.0.0.202 -g
[root@lvs ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
FWM  10 rr
  -> 10.0.0.201:0                   Route   1      0          0         
  -> 10.0.0.202:0                 Route   1      0          0         
[root@lvs ~]#cat /proc/net/ip_vs
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port Forward Weight ActiveConn InActConn
FWM 0000000A rr 
  -> 0A000201:0000     Route   1      0          9         
  -> 0A000202:0000     Route   1      0          9


[root@lvs ~]#ipvsadm -A -f 10 
[root@lvs ~]#ipvsadm -a -f 10 -r 10.0.0.201 -g
[root@lvs ~]#ipvsadm -a -f 10 -r 10.0.0.202-g
[root@lvs ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
FWM  10 wlc
  -> 10.0.0.201:0               Route   1      0          0         
  -> 10.0.0.201:0               Route   1      0          0
  
[root@LVS ~]#cat /proc/net/ip_vs
```

------

#### 注意事项

1. **防火墙标记生效范围**：防火墙标记在 **mangle 表** 的 `PREROUTING` 链中设置。这是因为流量在路由之前需要被标记。

2. **确保内核支持**：LVS 和 iptables 的防火墙标记功能依赖于内核模块，确保已加载相关模块（如 `ip_vs`、`nf_conntrack`）。

3. **流量监控**：可以使用 `ipvsadm -L -n` 或 `iptables -t mangle -L -n -v` 监控流量标记和转发情况。

4. 清理规则

   ：修改配置前需要清除旧的规则，避免冲突。例如：

   ```bash
   ipvsadm -C  # 清除所有 IPVS 配置
   iptables -F -t mangle  # 清空 mangle 表规则
   ```

------

通过防火墙标记，可以灵活地将不同协议、端口或子网的流量转发到指定的后端服务器，实现复杂场景的负载均衡。







###  LVS 持久连接

session 绑定：对共享同一组RS的多个集群服务，需要统一进行绑定，lvs sh算法无法实现 

持久连接（ lvs persistence ）模板：实现无论使用任何调度算法，在一段时间内（默认360s ），能够 实现将来自同一个地址的请求始终发往同一个RS

```bash
ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]]
```

持久连接实现方式： 

- 每端口持久（PPC）：每个端口定义为一个集群服务，每集群服务单独调度 

- 每防火墙标记持久（PFWMC）：基于防火墙标记定义集群服务；可实现将多个端口上的应用统一 调度，即所谓的 port Affinity 

- 每客户端持久（PCC）：基于0端口（表示所有服务）定义集群服务，即将客户端对所有应用的请 求都调度至后端主机，必须定义为持久模式





在 **LVS（Linux Virtual Server）** 中，**持久连接**（Persistence Connection）是一种确保同一客户端的多次请求在一定时间内始终被转发到同一个后端服务器的机制。它在处理需要会话保持的应用（例如购物车、登录状态等）时非常重要。

------

#### 持久连接的工作原理

1. LVS 会记录客户端和后端服务器之间的连接信息，包括客户端的源 IP 地址。
2. 当客户端再次发起请求时，LVS 根据之前的记录，将请求转发到之前分配的后端服务器，而不是重新应用负载均衡算法。
3. 记录的持久性连接有一个超时时间（Persistence Timeout），超过这个时间后，LVS 会重新应用负载均衡算法。

------

#### 配置持久连接

##### 1. **基础命令**

使用 `ipvsadm` 配置虚拟服务时，通过 `-p` 选项启用持久连接，并设置超时时间。

**命令格式：**

```bash
ipvsadm -A|E -t|u|f service-address -s scheduler -p [timeout]
```

- **`-p`**：启用持久连接。
- **`timeout`**：持久连接的超时时间，单位为秒（默认为 360 秒）。

------

2. ##### **配置示例**

**示例场景：**

- VIP 地址为 `10.0.0.1:80`（HTTP 服务）。
- 后端真实服务器为：
  - `192.168.1.101:80`
  - `192.168.1.102:80`
- 使用加权轮询调度算法（`wrr`），持久连接超时时间设置为 600 秒。

**配置步骤：**

1. 添加虚拟服务并启用持久连接：

   ```bash
   ipvsadm -A -t 10.0.0.1:80 -s wrr -p 600
   ```

2. 添加真实服务器：

   ```bash
   ipvsadm -a -t 10.0.0.1:80 -r 192.168.1.101:80 -m
   ipvsadm -a -t 10.0.0.1:80 -r 192.168.1.102:80 -m
   ```

3. 检查配置：

   ```bash
   ipvsadm -L -n
   ```

   **输出示例：**

   ```bash
   arduino复制代码IP Virtual Server version 1.2.1 (size=4096)
   Prot LocalAddress:Port Scheduler Flags
     -> RemoteAddress:Port Forward Weight ActiveConn InActConn
   TCP  10.0.0.1:80 wrr persistent 600
     -> 192.168.1.101:80 Masq    1      10         5
     -> 192.168.1.102:80 Masq    1      8          7
   ```

------

#### 持久连接的重要参数

1. **`-p timeout`**
   - 设置持久连接的超时时间。
   - 超过超时时间后，LVS 会重新应用负载均衡算法。
   - 如果不指定，默认超时为 360 秒。
2. **持久连接记录**
   - LVS 会记录客户端与后端服务器的映射关系。此记录基于客户端的源 IP 地址。
3. **调度算法**
   - 持久连接不影响调度算法。例如，初次请求时仍然会根据配置的调度算法（如轮询、最少连接等）选择后端服务器。

------

#### 示例应用场景

1. **Web 应用的会话保持**
   - 假设用户在登录后与后端服务器 A 建立会话，后续请求必须保持到服务器 A，否则会话状态丢失。
   - 使用持久连接确保用户的所有请求在会话超时前始终被转发到同一个后端服务器。
2. **购物车功能**
   - 如果负载均衡器每次随机将用户请求分配到不同的后端服务器，购物车中的商品数据可能无法保持。
   - 持久连接将用户请求始终指向同一个后端，确保购物车功能正常。
3. **混合协议场景**
   - 需要同时负载均衡 HTTP（80）和 HTTPS（443）流量的场景，可以结合防火墙标记（fwmark）和持久连接，确保同一客户端的 HTTP 和 HTTPS 请求指向同一个后端服务器。

------

#### 持久连接与其他负载均衡机制的关系

1. **与调度算法结合**
   - 持久连接与调度算法不冲突。初次请求根据调度算法选择后端服务器，后续请求直接依据持久连接记录转发。
2. **与防火墙标记结合**
   - 防火墙标记可以用来区分不同的服务类型，持久连接可以确保每种服务的流量指向固定的后端服务器。
3. **与连接跟踪结合**
   - LVS 的持久连接依赖连接跟踪机制（`conntrack`），需要确保内核支持并启用该功能。

------

#### 注意事项

1. **持久连接的成本**
   - LVS 需要为每个客户端记录持久连接的映射关系，映射记录占用内存资源。如果有大量客户端请求，可能会导致内存不足。
   - 调整系统内核参数（如连接跟踪表大小）来优化性能。
2. **负载均衡的均匀性**
   - 持久连接可能导致负载均衡不均匀。例如，大量请求来自同一个客户端时，可能使某个后端服务器的负载过高。
3. **超时时间的设置**
   - 设置合理的超时时间。如果时间过长，持久连接记录会占用大量内存；如果时间过短，可能导致会话中断。
4. **DNS 解析**
   - 如果客户端使用动态 IP 或 DNS 解析频繁变化，可能导致持久连接失效。

------

通过配置持久连接，LVS 可以更好地支持需要会话保持的应用场景，同时灵活结合其他机制（如防火墙标记、调度算法）实现复杂的负载均衡策略。



## LVS实践

### LVS-NAT模式

<img src="5day-png\24LVS-NAT.png" alt="image-20250109120613346" style="zoom:50%;" />

```powershell
192.168.10.200\10.0.0.200:
[root@ubuntu2404 ~]#vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
[root@ubuntu2404 ~]#sysctl -p
net.ipv4.ip_forward = 1
[root@ubuntu2404 ~]#ipvsadm -A -t 192.168.10.200:80 -s wrr
[root@ubuntu2404 ~]#ipvsadm -a -t 192.168.10.200:80 -r 10.0.0.201:80 -m
[root@ubuntu2404 ~]#ipvsadm -a -t 192.168.10.200:80 -r 10.0.0.202:80 -m
[root@ubuntu2404 ~]#ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  ubuntu2404.wang.org:http wrr
  -> 10.0.0.201:http              Masq    1      0          0         
  -> 10.0.0.202:http              Masq    1      0          0

[root@ubuntu2204 ~]#while :;do curl 192.168.10.200;sleep 0.5;done
10.0.0.201 RS1
10.0.0.202 RS2
10.0.0.201 RS1
10.0.0.202 RS2


ipvsadm -Ln --stats
=
cat /proc/net/ip_vs

ipvsadm -Lnc
=
cat /proc/net/ip_vs_conn

#保存规则
[root@ubuntu2404 ~]#mkdir /etc/sysconfig
[root@ubuntu2404 ~]#ipvsadm -Sn > /etc/sysconfig/ipvsadm
[root@ubuntu2404 ~]#cat /etc/sysconfig/ipvsadm 
-A -t 192.168.10.200:80 -s wrr
-a -t 192.168.10.200:80 -r 10.0.0.201:80 -m -w 1
-a -t 192.168.10.200:80 -r 10.0.0.202:80 -m -w 1
[root@ubuntu2404 ~]#systemctl enable --now ipvsadm.service
```

###  LVS-DR模式单网段

DR模型中各主机上均需要配置VIP，解决地址冲突的方式有三种：

1. 在前端网关做静态绑定
2. 在各RS使用arptables 
3. 在各RS修改内核参数，来限制arp响应和通告的级别

限制响应级别：arp_ignore 

- 0：默认值，表示可使用本地任意接口上配置的任意地址进行响应 
- 1：仅在请求的目标IP配置在本地主机的接收到请求报文的接口上时，才给予响应

限制通告级别：arp_announce 

- 0：默认值，把本机所有接口的所有信息向每个接口的网络进行通告 
- 1：尽量避免将接口信息向非直接连接网络进行通告 
- 2：必须避免将接口信息向非本网络进行通告

配置要点 

1. Director 服务器采用双IP桥接网络，一个是VIP，一个DIP 
2. Web服务器采用和DIP相同的网段和Director连接
3. 每个Web服务器配置VIP 
4. 每个web服务器可以出外网

<img src="5day-png\24LVS-DR模式单网段.png" alt="image-20250109145058976" style="zoom:50%;" />

```powershell
#route:10.0.0.101\192.168.10.101
[root@ubuntu2204 ~]#vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1
[root@ubuntu2204 ~]#sysctl -p
net.ipv4.ip_forward = 1

#RS1，SR2：10.0.0.201 && 10.0.0.202
[root@ubuntu2404 ~]#echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
[root@ubuntu2404 ~]#echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
[root@ubuntu2404 ~]#echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
[root@ubuntu2404 ~]#echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/all/arp_ignore
1
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/all/arp_announce 
2
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/lo/arp_ignore
1
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/lo/arp_announce 
2

===
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce

#LVS，SR1，SR2
ip a a 10.0.0.10/32 dev lo

#LVS：10.0.0.200
[root@ubuntu2404 ~]#apt install ipvsadm
[root@ubuntu2404 ~]#ipvsadm -A -t 10.0.0.10:80 -s rr
[root@ubuntu2404 ~]#ipvsadm -a -t 10.0.0.10:80 -r 10.0.0.201:80 -g
[root@ubuntu2404 ~]#ipvsadm -a -t 10.0.0.10:80 -r 10.0.0.202:80 -g
[root@ubuntu2404 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.10:80 rr
  -> 10.0.0.201:80                Route   1      0          0         
  -> 10.0.0.202:80                Route   1      0          0 

[root@ubuntu2204 ~]#while :;do curl 10.0.0.10;sleep 0.5;done
10.0.0.202 RS2
10.0.0.201 RS1
10.0.0.202 RS2
10.0.0.201 RS1
```



LVS的eth0的网关可否不配置？如果随便配置，发现什么问题？如果不配 置，怎么解决？

```bash
#默认值
[root@ubuntu2204 ~]#cat /proc/sys/net/ipv4/conf/all/rp_filter
2
[root@centos8 ~]#cat /proc/sys/net/ipv4/conf/all/rp_filter
1
#Ubuntu22.04修改内核参数为0
[root@ubuntu2204 ~]#echo "0" > /proc/sys/net/ipv4/conf/all/rp_filter
[root@ubuntu2204 ~]#echo "0" > /proc/sys/net/ipv4/conf/eth0/rp_filter
#Rocky8修改内核参数为0
[root@lvs ~]#echo "0" > /proc/sys/net/ipv4/conf/all/rp_filter
#说明:参数rp_filter用来控制系统是否开启对数据包源地址的校验。
#0标示不开启地址校验
#1表开启严格的反向路径校验。对每一个收到的数据包，校验其反向路径是否是最佳路径。如果反向路径不是最佳路径，则直接丢弃该数据包；
#2表示开启松散的反向路径校验，对每个收到的数据包，校验其源地址是否可以到达，即反向路径是否可以ping通，如反向路径不通，则直接丢弃该数据包。

```

 LVS的VIP可以配置到lo网卡,但必须使用32位的netmask,为什么?

```
lo（本地回环）网卡的子网掩码只能配置为 32 位的原因，主要与其用途和功能定位有关。以下是具体原因和背景：
1. 本地回环设备的特殊性
本地通信的设备：lo 是一个虚拟网络接口，用于本地通信（主机自身）。它不需要与其他设备通信，也不会转发或接收来自外部的流量。
点对点的特性：回环设备仅用于主机内部的网络通信，IP 数据包通过 lo 网卡发送后，会直接回到本地主机。因此，它实际上是一个点对点的通信，不需要定义一个子网范围。
2. /32 掩码的作用
表示单一 IP：子网掩码 /32 表示一个单一的 IP 地址。例如，10.0.0.100/32 仅指向 10.0.0.100，不涉及其他地址。
本地访问限制：对于回环设备，配置 /32 掩码表示只有指定的单个 IP 地址有效，避免了误操作导致与其他 IP 地址或网络的冲突。
3. 避免广播和路由开销
如果允许设置其他掩码（如 /24），会引入广播地址、网络地址以及可能的路由问题：
广播地址：例如 10.0.0.0/24 会生成 10.0.0.255 的广播地址，但对于回环设备来说，广播完全没有意义。
路由开销：较大的子网掩码可能引入额外的路由表项，而这些在回环设备中都是不需要的。
4. 网络范围的无意义
本地回环设备不需要连接外部网络，因此没有必要定义一个子网范围。例如：
配置 10.0.0.0/24 意味着允许访问 10.0.0.1-10.0.0.254，这些地址完全没有意义，因为回环流量仅限于主机内部。
5. 一致性与规范
操作系统规范：现代操作系统对回环设备的 IP 地址设置有严格限制，通常要求子网掩码为 /32，以确保不会引入任何无意义的配置或冲突。
一致性设计：其他虚拟接口（如 tun、tap）可以灵活配置掩码，但 lo 被设计为单一 IP 的通信接口，这种一致性简化了系统的设计和维护。
总结
回环设备的子网掩码只能配置为 /32 是因为其作为本地通信的特殊用途，仅支持单一的 IP 地址。这种设计避免了不必要的广播、路由开销和潜在的配置问题，同时也符合操作系统和网络规范的要求。
```

### LVS-DR模式多网段

<img src="5day-png\24lvs-DR模式多网段.png" alt="image-20250109145634853" style="zoom:50%;" />

```bash
#route:10.0.0.101\192.168.10.101
[root@ubuntu2204 ~]#vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1
[root@ubuntu2204 ~]#sysctl -p
net.ipv4.ip_forward = 1
ip a a 172.30.0.101/24 dev eth0:1

#RS1，SR2：10.0.0.201 && 10.0.0.202
[root@ubuntu2404 ~]#echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
[root@ubuntu2404 ~]#echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
[root@ubuntu2404 ~]#echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
[root@ubuntu2404 ~]#echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/all/arp_ignore
1
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/all/arp_announce 
2
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/lo/arp_ignore
1
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/lo/arp_announce 
2

===
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce

#LVS，SR1，SR2
ip a a 172.30.0.10/32 dev lo

#LVS：10.0.0.200
[root@ubuntu2404 ~]#apt install ipvsadm
[root@ubuntu2404 ~]#ipvsadm -A -t 172.30.0.10:80 -s rr
[root@ubuntu2404 ~]#ipvsadm -a -t 172.30.0.10:80 -r 10.0.0.201:80 -g
[root@ubuntu2404 ~]#ipvsadm -a -t 172.30.0.10:80 -r 10.0.0.202:80 -g
[root@ubuntu2404 ~]#ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.30.0.10:80 wrr
  -> 10.0.0.201:80                Route   1      0          0         
  -> 10.0.0.202:80                Route   1      0          0 

[root@ubuntu2204 ~]#while :;do curl 172.30.0.10;sleep 0.5;done
10.0.0.202 RS2
10.0.0.201 RS1
10.0.0.202 RS2
10.0.0.201 RS1
```



#### RS脚本

```powershell
#!/bin/bash

vip=172.30.0.10
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

```

#### VS脚本

```powershell
#!/bin/bash

vip='172.30.0.10'
iface='lo:1'
mask='255.255.255.255'
port='80'
rs1='10.0.0.201'
rs2='10.0.0.202'
scheduler='wrr'
type='-g'
#rpm -q ipvsadm &> /dev/null || yum -y install ipvsadm &> /dev/null

case $1 in
start)
    ifconfig $iface $vip netmask $mask #broadcast $vip up
    iptables -F
 
    ipvsadm -A -t ${vip}:${port} -s $scheduler
    ipvsadm -a -t ${vip}:${port} -r ${rs1} $type -w 1
    ipvsadm -a -t ${vip}:${port} -r ${rs2} $type -w 1
    echo "The VS Server is Ready!"
    ;;
stop)
    ipvsadm -C
    ifconfig $iface down
    echo "The VS Server is Canceled!"
    ;;
*)
    echo "Usage: $(basename $0) start|stop"
    exit 1
    ;;
esac

```

```bash
#方法一
ip a a 10.0.0.10/32 dev lo:1
#方法二
ifconfig lo:1 10.0.0.100/32
#方法三
nmcli connection modify eth0 +ipv4.addresses 172.30.0.10/24 
nmcli connection reload
nmcli connection up eth0
```



### LVS-TUNNEL隧道模式

![image-20250109153331119](5day-png\24LVS-TUNNAL隧道模式.png)

```powershell
#LVS,RS1,RS2
开启tunnel网卡配置
ifconfig tunl0 10.0.0.10 netmask 255.255.255.255 up
或者下面方法,注意设备名必须为tunl0
ip addr add 10.0.0.10/32 dev tunl0
ip link set up tunl0
自动加载了ipip模块
[root@ubuntu2404 ~]#lsmod | grep ipip
ipip                   20480  0
tunnel4                12288  1 ipip
ip_tunnel              32768  1 ipip

#正常在生产环境中不用修改内核参数，因为LVS-TUNNEL隧道模式用于跨网段的。
#修改内核参数
echo "1" > /proc/sys/net/ipv4/conf/tunl0/arp_ignore        
echo "2" > /proc/sys/net/ipv4/conf/tunl0/arp_announce
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce

#以下参数用来控制系统是否开启对数据包源地址的校验。
#0表示不开启地址校验
#1表示开启严格的反向路径校验。对每一个进行的数据包，校验其反向路径是否是最佳路径。如果反向路径不是最佳路径，则直接丢弃该数据包；
#2标示开启松散的反向路径校验，对每个进行的数据包，校验其源地址是否可以到达，即反向路径是否可以ping通，如反向路径不通，则直接丢弃该数据包。

#默认值，Ubuntu默认值可不做修改
[root@centos7 ~]#cat /proc/sys/net/ipv4/conf/all/rp_filter
1
root@rocky9-10:~ # cat /proc/sys/net/ipv4/conf/all/rp_filter
0
[root@ubuntu2404 ~]#cat /proc/sys/net/ipv4/conf/all/rp_filter
2

#lvs
ipvsadm -A -t 10.0.0.10:80 -s rr
ipvsadm -a -t 10.0.0.10:80 -r 10.0.0.201 -i 
ipvsadm -a -t 10.0.0.10:80 -r 10.0.0.202 -i 
ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.10:80 rr
  -> 10.0.0.201:80                Tunnel  1      0          0         
  -> 10.0.0.202:80                Tunnel  1      0          0 
 
#client:
while :;do curl 10.0.0.10;sleep 0.3;done
10.0.0.202 10.0.0.10 RS2
10.0.0.201 10.0.0.10 RS1
10.0.0.202 10.0.0.10 RS2
10.0.0.201 10.0.0.10 RS1
```

<img src="5day-png\24LVS-TUNNEL隧道双IP头.png" alt="image-20250109164919246" style="zoom:50%;" />



## 高可用

**LVS 不可用时： **

Director不可用，整个系统将不可用；SPoF Single Point of Failure 

解决方案：高可用，keepalived、heartbeat/corosync 

**RS 不可用时：** 

某RS不可用时，Director依然会调度请求至此RS 

解决方案： 由Director对各RS健康状态进行检查，失败时禁用，成功时启用 

**常用解决方案：** 

- keepalived 

- heartbeat/corosync  

- ldirectord 

**检测方式：** 

- 网络层检测，icmp 

- 传输层检测，端口探测 

- 应用层检测，请求某关键资源 

RS全不用时：backup server, sorry server 

## 面试题

### Linux 集群有哪些分类 

Linux 集群是一种通过多台服务器协作，来实现高性能、高可用性、负载均衡或其他特定功能的技术架构。根据其用途和实现目标，Linux 集群通常可以分为以下几类：

------

#### 一、高性能计算集群（HPC，High-Performance Computing Cluster）

##### **功能特点**：

- 提供强大的计算能力，通过多台服务器并行计算完成复杂任务。
- 主要用于科学研究、工程计算、大数据处理等需要高算力的场景。

##### **核心技术**：

- **任务分割**：将计算任务分解为多个小任务，分配到不同节点。
- **通信协议**：使用 **MPI（消息传递接口）** 或 **PVM（并行虚拟机）** 进行节点间的通信。
- **作业调度系统**：使用 **Slurm**、**PBS** 或 **Hadoop** 管理和调度任务。

##### **典型应用**：

- 天文学、气象建模、分子动力学仿真、基因组学分析。

------

#### 二、高可用性集群（HA，High Availability Cluster）

##### **功能特点**：

- 提供故障容错能力，保证服务在硬件或软件故障时仍能继续运行。
- 主要解决单点故障问题，通过主备切换或故障转移实现。

##### **核心技术**：

- **心跳机制**：通过心跳信号检测集群中节点的状态（如 **Heartbeat**、**Corosync**）。
- **资源管理**：使用 **Pacemaker** 管理服务和资源的启动、停止及迁移。
- **共享存储**：如 **DRBD** 或 **NFS**，用于多个节点共享相同数据。

##### **典型应用**：

- 数据库高可用（如 MySQL 主备切换）、Web 服务器高可用（如 Nginx+Keepalived）、文件服务（如 GlusterFS）。

------

#### 三、负载均衡集群（LB，Load Balancing Cluster）

##### **功能特点**：

- 将用户请求分配到多台服务器上，提升系统的并发能力和响应速度。
- 避免单点压力过大，提高整体吞吐量和稳定性。

##### **核心技术**：

- **负载均衡器**：如 **LVS（Linux Virtual Server）**、**HAProxy**、**Nginx**。
- **调度算法**：轮询（Round Robin）、最少连接（Least Connection）、加权分配（Weighted Round Robin）等。
- **健康检查**：检测后端服务器的运行状态，确保请求分配给正常服务的节点。

##### **典型应用**：

- Web 服务器、应用服务器集群（如 LVS+Nginx+Tomcat）、电子商务平台。

------

#### 四、存储集群

##### **功能特点**：

- 实现海量数据的存储、管理和访问，提供高可用性和扩展性。
- 支持分布式存储、块存储、文件存储等模式。

##### **核心技术**：

- **分布式存储系统**：如 **Ceph**、**GlusterFS**、**HDFS**。
- **冗余和复制**：通过数据冗余提高可靠性（如 RAID 或副本机制）。
- **一致性算法**：如 **Paxos**、**Raft** 确保数据一致性。

##### **典型应用**：

- 数据备份与归档、大数据存储（Hadoop HDFS）、对象存储（如 Ceph RADOS）。

------

#### 五、大数据集群

##### **功能特点**：

- 专门设计用于处理海量数据，通过分布式计算和存储架构实现数据分析和挖掘。
- 以高吞吐量为主要目标，支持批处理和流处理。

##### **核心技术**：

- **分布式计算框架**：如 **Hadoop MapReduce**、**Apache Spark**。
- **分布式存储框架**：如 **HDFS**、**Hive**、**HBase**。
- **调度与资源管理**：如 **YARN** 或 **Mesos**。

##### **典型应用**：

- 数据仓库、实时流处理、机器学习模型训练。

------

#### 六、云计算集群

##### **功能特点**：

- 提供虚拟化资源和弹性扩展能力，实现 IaaS（基础设施即服务）、PaaS（平台即服务） 和 SaaS（软件即服务）。
- 通过集群技术管理和分配物理资源。

##### **核心技术**：

- **虚拟化技术**：如 KVM、Xen、VMware。
- **云平台**：如 **OpenStack**、**Kubernetes**、**Docker Swarm**。
- **分布式网络与存储**：如 Open vSwitch、Ceph。

##### **典型应用**：

- 私有云、公有云、混合云部署。

------

#### 七、数据库集群

##### **功能特点**：

- 提高数据库的并发处理能力、可用性和容错能力。
- 支持主从复制、分布式存储和分片。

##### **核心技术**：

- **主从复制**：如 MySQL/MariaDB 的 Master-Slave。
- **分布式数据库**：如 **MongoDB**、**TiDB**、**CockroachDB**。
- **中间件**：如 **ProxySQL**、**MyCat**。

##### **典型应用**：

- 数据库读写分离、大型分布式数据库集群。

------

#### 八、应用集群

##### **功能特点**：

- 提供特定应用的集群服务，通常用于企业的定制化需求。
- 支持多实例部署，提高性能和可靠性。

##### **核心技术**：

- **应用级负载均衡**：如 HAProxy、Nginx。
- **服务发现与编排**：如 Consul、etcd、Zookeeper。

##### **典型应用**：

- 微服务架构、多实例的应用服务器集群。

------

#### 总结

Linux 集群根据功能可以分为以上几类，各有其适用场景。在实际部署中，不同类型的集群可能会结合使用。例如，一个网站可能既需要负载均衡集群（LVS+Nginx），也需要高可用集群（数据库 HA），甚至需要存储集群（Ceph）。选择合适的集群架构取决于业务需求和资源状况。

### 正向代理和反向代理区别 

**正向代理**和**反向代理**是代理服务器的两种常见使用方式。两者的区别主要体现在**服务对象**和**应用场景**上：

------

#### **一、正向代理（Forward Proxy）**

##### **定义**：

正向代理是客户端的代理，代理客户端向外部服务器发送请求并将响应返回给客户端。

##### **工作原理**：

- 客户端无法直接访问目标服务器。
- 客户端通过正向代理服务器发送请求。
- 正向代理服务器向目标服务器发送请求，并将目标服务器的响应返回给客户端。

##### **特征**：

1. **代理对象**：客户端。

2. **隐藏**：隐藏客户端信息，对目标服务器来说，所有请求都来自代理服务器。

3. 用途

   ：

   - **突破访问限制**：如访问被防火墙限制的外部资源（科学上网）。
   - **缓存请求**：减少对目标服务器的访问次数。
   - **隐私保护**：隐藏客户端的真实 IP 地址。

##### **示例场景**：

- 公司内网用户通过代理服务器访问外网。
- 使用 VPN 或 Shadowsocks 客户端代理访问受限网站。

##### **示意图**：

```
[客户端] → [正向代理] → [目标服务器]
```

------

#### **二、反向代理（Reverse Proxy）**

##### **定义**：

反向代理是服务器的代理，代理外部客户端向内部服务器发送请求。

##### **工作原理**：

- 客户端访问反向代理服务器，认为其是目标服务器。
- 反向代理服务器将请求转发到后端的真实服务器。
- 真实服务器处理请求后，反向代理将响应返回给客户端。

##### **特征**：

1. **代理对象**：服务器。

2. **隐藏**：隐藏真实服务器信息，对客户端来说，所有请求都发给代理服务器。

3. 用途

   ：

   - **负载均衡**：将请求分发到多个后端服务器。
   - **安全防护**：隐藏真实服务器地址，防止直接攻击。
   - **缓存优化**：缓存静态资源，减少后端服务器负载。
   - **SSL 卸载**：由反向代理处理 HTTPS，加快后端服务。

##### **示例场景**：

- 使用 Nginx 或 Apache 配置反向代理。
- 通过 CDN（如 Cloudflare）优化和保护网站。

##### **示意图**：

```
[客户端] → [反向代理] → [真实服务器]
```

------

#### **三、对比总结**

| **对比项**   | **正向代理**                           | **反向代理**                           |
| ------------ | -------------------------------------- | -------------------------------------- |
| **代理对象** | 客户端                                 | 服务器                                 |
| **功能**     | 帮助客户端访问目标服务器               | 帮助服务器接收客户端请求               |
| **隐藏信息** | 隐藏客户端信息，对服务器不可见         | 隐藏真实服务器信息，对客户端不可见     |
| **常用场景** | 科学上网、突破访问限制、缓存客户端请求 | 负载均衡、隐藏后端服务器、缓存静态资源 |
| **用户感知** | 客户端需要配置代理服务器               | 客户端无需配置，透明使用               |

------

#### **四、实际应用示例**

##### **正向代理实例**：

1. 用户 A 使用 Shadowsocks 配置代理，访问受限网站。
2. 公司内网通过 Squid 配置正向代理，实现对员工访问互联网的控制。

##### **反向代理实例**：

1. 网站通过 Nginx 配置反向代理，将流量分发到多个后端 Web 服务。
2. 使用 CDN（如 Cloudflare）为网站提供反向代理服务，保护源站。

------

总结来说，**正向代理面向客户端，帮助客户端访问目标服务器；反向代理面向服务器，帮助服务器接收客户端请求**。两者的用途不同，但在网络架构中经常结合使用，提升系统的灵活性和安全性。

### 四层代理和七层代理的区别 

四层代理和七层代理是网络负载均衡和代理技术中常见的两种实现方式，它们基于 **OSI 模型**的不同层次来处理数据流量，主要区别在于代理操作的网络协议层级和处理能力。

------

#### **一、四层代理（基于传输层代理）**

##### **定义**：

四层代理工作在 **OSI 模型的传输层（L4）**，基于 **TCP** 或 **UDP** 协议的端口和 IP 地址进行流量转发。

##### **工作原理**：

- 四层代理只关注 TCP/UDP 的源地址、目标地址、源端口、目标端口。
- 不解析应用层协议（如 HTTP、FTP）。
- 使用 IP 和端口号来转发流量到后端服务器。

##### **特点**：

1. **性能高**：四层代理只需处理传输层协议，不需要解析应用层数据，性能更高。
2. **透明代理**：客户端通常不知道请求被代理，因为四层代理不修改请求内容。
3. 有限的负载均衡能力：
   - 支持简单的调度算法（如轮询、最小连接）。
   - 无法基于 URL、Cookie 等应用层信息做复杂的流量分发。

##### **应用场景**：

- **LVS（Linux Virtual Server）**：一种高性能的四层负载均衡技术。
- 基于 TCP 的代理服务，如 MySQL、SMTP 等。
- 高性能要求的场景。

##### **示意图**：

```
[客户端] → [四层代理] → [真实服务器]
```

------

#### **二、七层代理（基于应用层代理）**

##### **定义**：

七层代理工作在 **OSI 模型的应用层（L7）**，可以深入解析应用层协议（如 HTTP、HTTPS、FTP）并基于应用层信息（如 URL、Cookie、Header 等）进行流量分发。

##### **工作原理**：

- 七层代理解析并理解应用层协议（如 HTTP 的请求方法和头信息）。
- 根据业务需求，代理服务器可以对请求内容进行修改、缓存或智能路由。

##### **特点**：

1. 灵活性强：
   - 支持基于 URL、主机名（Host）、Cookie 等高级规则进行流量分发。
   - 可实现动态内容与静态内容分离。
2. 智能功能：
   - 提供 HTTPS 卸载。
   - 具备缓存、压缩、过滤、重写等功能。
3. 性能开销高：
   - 需要解析应用层协议，处理性能相对较低。
4. 可见性：
   - 可以对应用层请求和响应进行深度监控和日志记录。

##### **应用场景**：

- Nginx、HAProxy 的七层代理模式。
- Web 服务负载均衡、API 网关。
- 基于 HTTPS 的内容分发和安全防护。

##### **示意图**：

```
[客户端] → [七层代理] → [真实服务器]
```

------

#### **三、四层代理与七层代理的对比**

| **对比项**   | **四层代理**                     | **七层代理**                       |
| ------------ | -------------------------------- | ---------------------------------- |
| **工作层次** | OSI 模型的传输层（L4，TCP/UDP）  | OSI 模型的应用层（L7，HTTP/HTTPS） |
| **处理内容** | IP 地址、端口号                  | URL、Cookie、Header 等应用层信息   |
| **性能**     | 性能更高，开销较小               | 性能稍低，开销较大                 |
| **智能化**   | 功能简单，只能基于网络层信息路由 | 功能强大，支持复杂的路由规则       |
| **隐藏性**   | 客户端无法感知代理               | 客户端可能感知代理（如重写的头部） |
| **用途**     | TCP/UDP 应用负载均衡，低延迟场景 | Web 服务负载均衡，复杂路由场景     |
| **常见工具** | LVS、F5 Big-IP                   | Nginx、HAProxy                     |

------

#### **四、实际应用场景的选择**

1. **使用四层代理的场景**：
   - 系统对性能要求高，且无需复杂的路由规则。
   - 需要代理的服务是基于 TCP/UDP 的（如数据库、邮件服务、文件传输）。
   - 大规模的负载均衡部署（如 LVS 四层负载均衡）。
2. **使用七层代理的场景**：
   - 系统需要基于 HTTP/HTTPS 等应用层协议做智能流量分发。
   - 需要对用户请求进行深度解析（如 URL 重写、缓存静态资源、SSL 卸载）。
   - 实现复杂的负载均衡功能，如动态内容和静态内容的分离。

------

总结来说，四层代理适合对性能和延迟敏感的场景，七层代理适合需要复杂路由和应用层处理的场景。实际部署中，可能会结合使用这两种代理技术以满足不同的业务需求。

### LVS的工作模式有哪些，有什么特点 

LVS（Linux Virtual Server）是一种高性能、高可用的负载均衡技术，主要用于实现网络服务的负载均衡。它提供三种主要的工作模式，每种模式的实现原理和特点如下：

------

#### **1. NAT 模式（Network Address Translation）**

##### **工作原理**：

- 负载均衡器（Director）通过网络地址转换（NAT）技术，将请求从客户端转发到后端服务器（RS，Real Server）。
- 后端服务器将响应数据返回给负载均衡器，再由负载均衡器转发给客户端。

##### **特点**：

1. 网络要求：
   - 所有真实服务器和负载均衡器需要在同一个私有网络中（同一网段）。
   - 负载均衡器充当网关角色。
2. 数据路径：
   - 请求和响应数据都需要经过负载均衡器，增加了负载均衡器的压力。
3. 优点：
   - 真实服务器无需配置额外的公网 IP，便于管理。
   - 适合小规模集群部署。
4. 缺点：
   - 性能受限于负载均衡器，负载均衡器可能成为瓶颈。
   - 数据双向传输导致负载均衡器的网络和 CPU 开销较大。

##### **应用场景**：

- 小规模内部网络，网络带宽要求不高的场景。

------

#### **2. DR 模式（Direct Routing，直接路由模式）**

##### **工作原理**：

- 客户端请求发送到负载均衡器，负载均衡器根据调度算法选择一台真实服务器并将请求直接转发给它。
- 后端服务器直接将响应数据通过客户端网关返回给客户端，不经过负载均衡器。

##### **特点**：

1. 网络要求：
   - 负载均衡器和真实服务器必须在同一网段。
   - 真实服务器的网卡需绑定虚拟 IP（VIP）。
2. 数据路径：
   - 请求流量经过负载均衡器，响应流量直接返回给客户端。
3. 优点：
   - 减轻了负载均衡器的压力（响应流量不回流）。
   - 性能较高，适合大规模集群部署。
4. 缺点：
   - 配置复杂，真实服务器需绑定 VIP，并调整网络栈（如禁用 ARP）。
   - 真实服务器和负载均衡器需在同一物理网络中。

##### **应用场景**：

- 高并发场景，需要处理大量请求的 Web 服务。

------

#### **3. TUN 模式（IP Tunneling，隧道模式）**

##### **工作原理**：

- 客户端请求通过负载均衡器转发给真实服务器。
- 请求通过隧道（IPIP 协议）转发，真实服务器处理后直接将响应返回给客户端。

##### **特点**：

1. 网络要求：
   - 负载均衡器和真实服务器可以在不同的网络（跨网段部署）。
   - 真实服务器需支持 IPIP 隧道协议。
2. 数据路径：
   - 请求流量经过负载均衡器，通过 IP 隧道传递给真实服务器。
   - 响应流量直接返回客户端。
3. 优点：
   - 支持跨网段部署，灵活性高。
   - 减轻了负载均衡器的网络负载。
4. 缺点：
   - 配置复杂，真实服务器需支持并启用 IPIP 隧道。
   - IP 隧道可能增加一定的处理开销。

##### **应用场景**：

- 分布式集群，真实服务器分布在不同的网络或数据中心。

------

#### **4. FULLNAT 模式（Full Network Address Translation）**

##### **工作原理**：

- 客户端请求经过负载均衡器时，负载均衡器对源 IP 和目标 IP 地址都进行转换。
- 响应数据也需回到负载均衡器，由其还原目标地址后再转发给客户端。

##### **特点**：

1. 网络要求：
   - 无需真实服务器和负载均衡器在同一网段。
2. 数据路径：
   - 请求和响应数据均需经过负载均衡器。
3. 优点：
   - 真实服务器无需绑定 VIP。
   - 适合跨网段部署。
4. 缺点：
   - 数据双向传输，增加负载均衡器的压力。
   - 性能较 NAT 稍高，但仍受限于负载均衡器。

##### **应用场景**：

- 跨网段、小规模部署。

------

##### **模式对比**

| **特性**     | **NAT 模式**     | **DR 模式**    | **TUN 模式**   | **FULLNAT 模式**   |
| ------------ | ---------------- | -------------- | -------------- | ------------------ |
| **请求转发** | 负载均衡器转发   | 负载均衡器转发 | 负载均衡器转发 | 负载均衡器转发     |
| **响应回流** | 回流到负载均衡器 | 直接到客户端   | 直接到客户端   | 回流到负载均衡器   |
| **性能**     | 较低             | 高             | 高             | 较低               |
| **复杂度**   | 简单             | 较复杂         | 复杂           | 较简单             |
| **网络限制** | 必须同网段       | 必须同网段     | 支持跨网段     | 支持跨网段         |
| **适用场景** | 小规模集群       | 高并发场景     | 分布式部署     | 跨网段、小规模场景 |

------

##### **总结**

1. **NAT 模式**：简单易用，适合小规模集群，但性能有限。
2. **DR 模式**：高性能，适合高并发场景，但要求负载均衡器和真实服务器在同一网段。
3. **TUN 模式**：灵活性高，支持跨网段，但配置复杂。
4. **FULLNAT 模式**：兼顾灵活性和易用性，但性能和效率较 NAT 有一定提升。

根据业务需求、网络环境和性能要求选择合适的模式，可以充分发挥 LVS 的负载均衡能力。

### LVS的调度算法 

#### 静态调度算法

##### 1、**轮叫调度（Round Robin，RR）**

- **原理**：将请求依次分配给后端服务器，每台服务器都分配到相同数量的请求。
- 特点：
  - 简单高效。
  - 适合后端服务器性能均等的场景。
- **缺点**：不考虑服务器负载情况，可能导致性能不均。

##### **2、加权轮叫调度（Weighted Round Robin，WRR）**

- **原理**：根据服务器的权重分配请求，权重高的服务器分配更多请求。
- 特点：
  - 可以根据服务器的性能（如 CPU、内存）设置权重。
  - 适合服务器性能差异较大的场景。
- **缺点**：仍不考虑实时负载变化。

#### 3、**源地址哈希调度（Source Hashing，SH）**

- **原理**：根据客户端的源 IP 地址计算哈希值，并将请求分配到固定的后端服务器。
- 特点：
  - 客户端请求总是分配到相同的服务器（源地址不变时）。
  - 适合需要持久会话（Session Persistence）的场景。
- **缺点**：负载均衡性可能较差，容易出现热点服务器。

#### 4、**目标地址哈希调度（Destination Hashing，DH）**

- **原理**：根据目标 IP 地址计算哈希值，将请求分配到固定的后端服务器。
- 特点：
  - 适合服务代理和缓存系统等场景。
- **缺点**：负载均衡性依赖目标地址的分布。

#### 动态调度算法

##### **1、最小连接数调度（Least Connection，LC）**

- **原理**：将请求分配给当前活跃连接数最少的服务器。
- 特点：
  - 动态调整，适合服务器性能相近的场景。
  - 可以更好地分散负载。
- **缺点**：无法考虑服务器实际性能。

#### 2、**加权最小连接数调度（Weighted Least Connection，WLC）**

- **原理**：在最小连接数调度的基础上，引入权重作为调度依据。

- 计算公式：

  负载 = 活跃连接数 / 权重\text{负载 = 活跃连接数 / 权重}负载 = 活跃连接数 / 权重

  - 将请求分配给负载值最小的服务器。

- 特点：

  - 动态调度，考虑了服务器性能和负载。
  - 适合服务器性能和负载波动较大的场景。

#### 3、**最短期望延迟调度（Shortest Expected Delay，SED）**

- **原理**：在 WLC 基础上进一步优化，通过预估的响应延迟分配请求。
- **计算公式**： 负载 = 活跃连接数 + 1×256/权重\text{负载 = 活跃连接数 + 1} \times 256 / \text{权重}负载 = 活跃连接数 + 1×256/权重
- 特点：
  - 优化了 WLC 在处理响应延迟上的不足。
  - 适合对延迟敏感的场景。

#### 4、**最少队列调度（Never Queue，NQ）**

- **原理**：如果服务器当前没有等待处理的连接，则直接分配请求；否则采用 SED 算法。
- 特点：
  - 保证实时性，尽量减少请求等待队列的长度。
  - 适合对实时性要求较高的场景。

#### 5、**LBLC（基于局部性的最少连接调度）**

##### **工作原理**：

- **局部性优先**：根据目标 IP 地址将请求尽量分配到之前处理过该目标的服务器。
- **最少连接优先**：如果之前没有服务器处理过该目标，或者之前的服务器负载过高，则将请求分配给连接数最少的服务器。

##### **特点**：

1. 局部性优化：
   - 同一个目标地址（如缓存中的某一数据对象）尽量由固定的服务器处理。
   - 避免不同服务器重复加载同一目标的数据，提升缓存命中率。
2. 负载均衡：
   - 在局部性条件下，尽量平衡服务器之间的负载。
3. 优先级逻辑：
   - 优先考虑局部性分配（目标地址匹配），如果无法满足则按最少连接原则分配。

#### 6、**LBLCR（基于局部性的最少连接调度，带复制功能）**

##### **工作原理**：

- 在 LBLC 的基础上增加了**复制功能**。
- **局部性优先**：优先将目标地址分配到之前处理过该目标的服务器。
- **负载复制**：如果之前的服务器负载过高，则会将该目标的负载“复制”到另一台负载较轻的服务器。

##### **特点**：

1. 局部性优化：
   - 和 LBLC 一样，同一目标地址尽量由固定的服务器处理。
2. 负载复制：
   - 当目标地址的访问量突然增加，原本的服务器无法承受时，将目标地址的请求分配到另一台服务器，并复制目标数据。
   - 减轻热点服务器的压力。
3. 高可用性：
   - 即使局部性被打破，系统依然能保证负载均衡和高性能。

##### **适用场景**：

- 和 LBLC 类似，适用于目标地址具有局部性访问特性的场景，但需要应对**突发流量或热点资源问题**。
- 例如，某一资源突然成为热点，导致访问量激增。

##### **适用场景**：

- 目标地址的请求具有**明显的局部性特征**（如访问同一目标地址的频率较高）。
- 例如，CDN 系统或缓存系统中，同一目标文件或资源被频繁访问。

### LVS和Nginx,Haproxy 的区别

LVS、Nginx 和 HAProxy 都是常见的负载均衡工具，它们的作用相似，但实现方式、性能特点和使用场景各不相同。以下从多个维度分析它们的区别：

------

##### **1. 工作层次**

- LVS：
  - 工作在 **网络层（第 4 层）**，基于 IP 和端口进行负载均衡。
  - 处理的是传输层协议（TCP/UDP）。
- Nginx：
  - 工作在 **应用层（第 7 层）**，基于 HTTP、HTTPS、WebSocket 等协议进行负载均衡。
  - 可以解析 HTTP 请求内容（如 URL、头部信息等）。
- HAProxy：
  - 主要工作在 **第 4 层（传输层）** 和 **第 7 层（应用层）**。
  - 提供更灵活的调度策略，支持基于请求内容的路由。

------

##### **2. 性能对比**

- LVS：
  - 性能最高，转发效率极高，适合大规模流量场景。
  - 基于内核模块实现（IPVS），系统资源占用低。
- Nginx：
  - 性能次之，处理复杂 HTTP 请求时性能有所下降。
  - 支持丰富的功能，如静态文件处理、缓存等。
- HAProxy：
  - 性能介于 LVS 和 Nginx 之间。
  - 专注于负载均衡，效率高，支持复杂的负载均衡算法。

------

##### **3. 功能特性**

| 功能             | **LVS**                          | **Nginx**                    | **HAProxy**                    |
| ---------------- | -------------------------------- | ---------------------------- | ------------------------------ |
| **负载均衡层次** | 网络层（4 层）                   | 应用层（7 层）               | 传输层（4 层）和应用层（7 层） |
| **支持协议**     | TCP、UDP                         | HTTP、HTTPS、WebSocket       | TCP、HTTP、HTTPS               |
| **健康检查**     | 简单（基于 IP 层连接检测）       | 复杂（支持 HTTP 状态码检测） | 复杂（支持 HTTP、TCP 检测）    |
| **会话保持**     | 基于 IP 地址                     | 基于 Cookie、URL 或 Session  | 基于 Cookie 或自定义规则       |
| **流量分发算法** | 静态、动态（如 RR、WLC、SED 等） | 基于 URL、Header、IP 等      | 基于 URL、Cookie、IP 等        |
| **扩展性**       | 高，可用于大规模集群             | 中，适合小中型应用           | 中，适合中型负载场景           |
| **SSL 支持**     | 不直接支持（需结合 Keepalived）  | 支持                         | 支持                           |
| **静态内容处理** | 不支持                           | 支持                         | 不支持                         |
| **缓存功能**     | 不支持                           | 支持                         | 不支持                         |

------

##### **4. 使用场景**

##### **LVS**：

- 适合场景：
  - 超大规模、高并发场景（如电商网站、视频流媒体）。
  - 需要 7*24 小时稳定运行的核心服务。
- 不足：
  - 不支持复杂的 HTTP 请求处理。
  - 配置复杂，对技术人员要求较高。

##### **Nginx**：

- 适合场景：
  - Web 应用前端，尤其是需要处理静态资源、缓存等场景。
  - 小中型网站，或对 HTTP 请求有复杂分发需求的场景。
- 不足：
  - 性能不如 LVS，对于超高并发的场景不如 LVS 高效。

##### **HAProxy**：

- 适合场景：
  - 需要高效处理 TCP 或 HTTP 流量，且负载均衡规则较为复杂的场景。
  - 中大型网站，或对负载均衡的灵活性有较高要求的场景。
- 不足：
  - 不支持静态文件处理和缓存。
  - 对 7 层功能支持比 Nginx 稍弱。

------

#### **5. 部署与实现**

- LVS：
  - 工作于 Linux 内核层，需通过 ipvsadm 工具配置。
  - 一般结合 Keepalived 提供高可用。
- Nginx：
  - 软件级别实现，安装配置较简单。
  - 可结合 Lua 模块扩展功能。
- HAProxy：
  - 也是软件级别实现，配置稍复杂。
  - 提供更细粒度的日志和监控能力。

------

#### **6. 对比总结**

| **工具**    | **优点**                               | **缺点**                         | **适用场景**                       |
| ----------- | -------------------------------------- | -------------------------------- | ---------------------------------- |
| **LVS**     | 高性能、低资源占用、稳定性强           | 配置复杂、功能单一               | 大规模流量场景、高性能要求         |
| **Nginx**   | 灵活、支持多功能（缓存、静态资源）     | 性能不及 LVS，对高并发支持有限   | 小中型网站、复杂 HTTP 分发需求     |
| **HAProxy** | 性能高、规则灵活、支持高级负载均衡算法 | 不支持静态资源处理，缓存功能缺失 | 中大型网站、复杂规则或高效负载场景 |

------

#### **如何选择**

1. **大规模流量、高并发**：选择 LVS。
2. **小中型 Web 应用**：选择 Nginx，提供负载均衡和静态资源支持。
3. **复杂负载均衡规则或 TCP/HTTP 应用**：选择 HAProxy。