# 20、Linux防火墙

## ips和ids

### 1. **IDS（入侵检测系统）**

#### 定义：

- IDS 是一种监控网络流量或系统活动的安全工具，旨在检测潜在的威胁或异常行为。
- 它通常是被动的，只负责报警，不会主动阻止攻击。

#### 工作原理：

- **签名检测**：通过预定义的攻击模式（签名）检测已知威胁。
- **异常检测**：通过建立基线来检测偏离正常行为的活动。

#### 特点：

- **检测为主**：只负责报警，通常不会干预网络流量。
- **部署位置**：可以部署在网络边界或主机上，类似监控摄像头。
- **实时性较弱**：通常需要人工或后续系统来分析报警。

#### 使用场景：

- 适用于需要监控、记录网络活动并生成警报的场合。
- 例如：发现未授权访问、暴力破解或恶意软件传播。

------

### 2. **IPS（入侵防御系统）**

#### 定义：

- IPS 是一种主动的网络安全工具，不仅可以检测威胁，还能自动采取措施阻止攻击。

#### 工作原理：

- **签名检测**：和 IDS 类似，通过已知模式检测威胁。
- **异常检测**：检测偏离正常行为的活动。
- **实时阻断**：一旦发现威胁，会采取措施阻止流量或隔离攻击。

#### 特点：

- **防御为主**：能够实时拦截和阻止攻击。
- **部署位置**：通常部署在网络路径中，类似安检门。
- **实时性强**：需要快速分析流量并采取行动。

#### 使用场景：

- 适用于需要自动阻断攻击的场合。
- 例如：防止 DDoS 攻击、病毒传播和网络漏洞利用。

------

### **对比总结**

| 特性         | IDS                              | IPS                          |
| ------------ | -------------------------------- | ---------------------------- |
| **功能**     | 检测威胁并报警                   | 检测威胁并主动防御           |
| **工作方式** | 被动监控，报警后人工响应         | 主动拦截，自动防御           |
| **部署方式** | 旁路部署（不会影响网络流量）     | 串联部署（需要处理网络流量） |
| **实时性**   | 仅检测，无实时阻断               | 实时阻断攻击                 |
| **适用场景** | 关注日志和攻击记录，进行后续分析 | 需要实时防御以阻止攻击       |

------

### **结合使用**

- **IDS 和 IPS 常常配合使用**：IDS 提供深入的监控和日志分析，而 IPS 负责第一时间阻断攻击，从而形成全面的防护体系。



****



## FW和WW

### FW：Firewall防火墙

```powershell
	防火墙系统是在内部网络和外部网络之间，专用网络与公共网络之间，可信网络和不可信网络之间建立屏障。它是一种 "隔离"  技术，是在两个网络通讯时执行的一种访问控制尺度，它能允许经你 “同意”的个人和数据进入你的网络，同时将你 “不同意” 的个人和数据拒之门外，最大限度的阻止非法的流量和数据进入系统或网络。同时防火墙也可以根据规则禁止本地网络向外发送数据报文，以及根据规则转发网络流量等。
	防火墙以串行的方式接入系统，所有流入流出的网络报文都要经由防火墙。根据工作模式不同，防火墙可以分为 网络层防火墙，应用层防火墙等。
```



### WW：Waterwall 防水墙

```powershell
	与防火墙相对，防水墙是一种防止内部信息泄漏的安全产品。它利用透明加解密，身份认证，访问控制和审计跟踪等技术手段，对涉密信息，重要业务数据和技术专利等敏感信息的存储，传播和处理过程，实施安全保护；最大限度地防止敏感信息泄漏、被破坏和违规外传，并完整记录涉及敏感信息的操作日志，以便日后审计。
	与防火墙关注外部流量进入的攻击行为，防水墙主要关注的是内部流量的隐患行为。
```



****



## 防火墙分类

主机防火墙

```
主机防火墙对单个主机进行防护，运行在终端主机上，主机防火墙可以阻止未授权的程序和数据出入计算，Windows和Linux 系统都有自带的主机防火墙。
```

网络防火墙

```
网络防火墙部署在整个系统的主线网路上，对整个系统的出入数据进行过滤。
```

### 按实现方式划分

硬件防火墙和软件防火墙

软件防火墙：

```
Centos/Rocky/Redhat系列：firewalld.service
Ubuntu系列：ufw.service
```

硬件防火墙

```
硬件防火墙生产厂商有 华为，天融信，启明星辰，绿盟，深信服、PaloAlto、fortinet、Cisco、checkpoint、NetScreen等。
```

### 按网络协议划分

```
网络层防火墙工作在OSI七层模型中的网络层，又称包过滤防火墙。
应用层防火墙工作在OSI七层模型中的应用层，又称网关防火墙或代理服务器防火墙。
```

![image-20241218105116526](D:\桌面\mage.md\笔记\5day-png\20包过滤防火墙)

### 按实现细节角度划分

```
包过滤防火墙，
 	根据用户定制的关键字，检查数据包的网络信息，不检查具体数据，应用层控制比较弱。
应用网关防火墙：
 	根据用户定制的关键字，检查数据包的数据信息，检查IP、TCP报头，网络层控制比较弱。
状态检测防火墙：
 	基于简单包过滤防火墙，增强数据包的网络状态信息检测，应用层控制比较弱。
复合型防火墙：
 	综合性的防火墙，可以检查整个数据包内容，具备网络层和应用层控制，兼有会话层控制措施。
```

## netfilter/iptables原理![image-20241218110655425](D:\桌面\mage.md\笔记\5day-png\20ntfilter基础.png)

```
Netfilter/iptables是Linux平台下的包过滤防火墙系统，由Netfilter框架和iptables两部分组成，
```

```
Netfilter
 	是Linux内核中的一个框架，用于进行网络数据包的过滤和操作。它提供了强大的网络数据包处理功能，使系统管理员能够在Linux系统上实施高级的网络安全策略，包括防火墙、网络地址转换（NAT）、流量控制等。Netfilter的核心是一个包过滤器，它可以在数据包通过网络协议栈的不同阶段进行处理。
iptables
 	是基于Netfilter框架实现的报文选择系统，可以用于报文的过滤、网络地址转换和报文修改等功能。iptables本质上包含了五个规则表，而规则表则包含了一系列的报文的匹配规则以及操作目标。
	由于iptables涉及到底层的相关环境处理，而iptables的命令本身又是用户空间的工具，所以iptables又可以划分为两部分：iptables(内核空间) 和 iptables 命令行工具(用户空间)
```

### netfilter原理

```
1netfilter 在内核中某些位置放置了五个"勾子函数"（hook function），分别是 INPUT，OUTPUT，FORWARD，PREOUTING，POSTROUTING，管理员可以通过配置工具配置规则，让这五个函数按照自定义的规则进行工作，达到防火墙的功能。
```

![image-20241218113032453](D:\桌面\mage.md\笔记\5day-png\20netfilter.png)

三种报文流向

| 数据走向   | 具体过程                                |
| ---------- | --------------------------------------- |
| 流入本机   | PREROUTING --> INPUT --> 用户空间进程   |
| 流出本机   | 用户空间进程 --> OUTPUT --> POSTROUTING |
| 从本机转发 | PREROUTING --> FORWARD --> POSTROUTING  |

### iptables

iptables 中的链

```
内置链
对应内核中的每一个勾子函数 （INPUT，OUTPUT，FORWARD，PREROUTING，POSTROUTING）

自定义链
用于对内置链进行扩展或补充，可实现更灵活的规则组织管理机制；只有Hook钩子调用自定义链时，才会生效
```

iptables 中的表

```
表名 								备注
filter 			过滤规则表，根据预定义的规则过滤符合条件的数据包，默认表
nat 			network address translation 地址转换规则表
mangle 			修改数据标记位规则表，修改数据报文的。
raw 			关闭启用的连接跟踪机制，加快封包穿越防火墙速度
security		用于强制访问控制（MAC）网络规则，由Linux安全模块（如SELinux）实现，很少使用
```

表的优先级（从高到低）

```
security --> raw --> mangle --> nat --> filter
```

![image-20241218161752106](D:\桌面\mage.md\笔记\5day-png\20数据包过滤匹配流程.png)

#### iptables命令由四部分组成

```
字段 						说明
iptables 				命令
Table 			具体要操作的表，用 -t 指定，raw|mangle|nat|filter，默认 filter
Chain 			具体要操作的链，PREROUTING|INPUT|FORWARD|OUTPUT|POSTROUTING
Rule 			具体规则，由匹配条件和目标组成，如果满足条件，就执行目标中的规则，目标用 -j 指定
```

```powershell
iptables -t filter -A INPUT -s 192.168.0.1 -j DROP
```

```powershell
指定表
-t|--table table 			#指定表 raw|mangle|nat|filter，如果不显式指定，默认是filter

操作链
-N|--new-chain chain 		#添加自定义新链
-X|--delete-chain [chain] 	#删除自定义链(要求链中没有规则)
-P|--policy chain target 	#设置默认策略，对filter表中的链而言，其默认策略有ACCEPT|DROP
-E|--rename-chain old-chain new-chain 	#重命名自定义链，引用计数不为0的自定义链不能被重命名
-L|--list [chain] 			#列出链上的所有规则
-S|--list-rules [chain] 	#列出链上的的有规则
-F|--flush [chain] 			#清空链上的所有规则，默认是所有链
-Z|--zero [chain [rulenum]] #置0，清空计数器，默认操作所有链上的所有规则

操作具体规则
-A|--append chain rule-specification 			#往链上追加规则
-I|--insert chain [rulenum] rule-specification 	#往链上插入规则，可以指定编号，默认插入到最前面
-C|--check chain rule-specification 			#检查链上的规则是否正确
-D|--delete chain rule-specification 			#删除链上的规则 
-D|--delete chain rulenum 						#根据编号删除链上的规则
-R|--replace chain rulenum rule-specification 	#根据链上的规则编号，使用新的规则替换原有规则

其他选项
-h|--help 		#显示帮助
-V|--version 	#显示版本
-v|--verbose 	#显示详细信息
-n|--numeric 	#以数字形式显示IP和端口，默认显示主机名和协议名，否则容易遭受hosts解析影响
--line-numbers 	#显示每条规则编号
-j|--jump 		# 决定了数据包在满足特定条件后的命运
 				# ACCEPT：允许数据包通过。
				# DROP：丢弃数据包，不给出任何回应。等客户端测试多次后，主动放弃。
				# REJECT：直接拒绝数据包，并向发送方发送一个错误响应。

```

```powershell
root@ubuntu-42:~ # iptables -vnL INPUT 
Chain INPUT (policy ACCEPT 4156 packets, 258K bytes)
 pkts bytes target     prot opt in     out     source               destination         
   70  5880 DROP       0    --  *      *       10.0.0.11            0.0.0.0/0       
```

```
结果显示：
    -v显示的内容比-n -L 显示的内容多了四个字段
    pkts 规则匹配到的报文数量的多少
    bytes 规则匹配到的报文内容量的大小
    in 规则匹配到的流入的接口，*代表任意接口
    out 规则匹配到的流出的接口,*代表任意接口
```

### 规则保存

```powershell
iptables-save > iptables.rules
```

**还原规则**

```powershell
iptables-restore < iptables.rules
```

### 规则清楚

```powershell
方法1：清除单个规则我们可以基于编号和内容进行删除
 	-D chain rulenum 	指定规则编号来删除
 	-D chain rule 		指定规则名字来删除
方法2：清除规则计数
    -Z 					清空所有规则计数
 	-Z INPUT 1 			清空第1条规则计数
方法3：清除所有规则
 	-F 					特指默认规则（清空规则）
 	-X 					特指自定义规则
```

```powershell
iptables -D INPUT 4							#指定编号删除
iptables -D INPUT -s 10.0.0.13 -j DROP		#指定规则名字删除
iptables -Z									#数据包全部清零
```

## 规则解析

自定义规则语句

```powershell
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
注意：
 	target 其实指的是，我们设定的规则对数据包进行的操作，需要达到什么样的目标，常见的目标有：
 		ACCEPT-接受、DROP-拒绝、REJECT-转发、等
 		后面的prot~destination，其实都是规则的基本描述信息。
```

默认规则语句

```powershell
默认语句，也就是iptables链范围中，默认启用的通用规则。表现形式如下：
 	Chain INPUT (policy ACCEPT)
```

```powershell
注意：
 	policy 表示该Chain的内置默认动作，每个chain只能有一个policy，
 	policy规则的target目标只能是DROP或ACCEPT。
```

```powershell
防火墙的命令规则其实就是：
 iptables -子命令 <链> <规则策略> [选项]
注意：
 	由于"[选项]"部分是可选的，所以我们的iptables的核心规则是
    iptables -参数 <链> <规则策略>
   		- 规则策略我们有时候也称为匹配条件
```

```powershell
iptable -子命令 <链>         <规则策略>             [选项]
iptable  -A    INPUT   -p icmp --icmp-type 8     -j DROP
iptable  -A    INPUT   -s 10.0.0.42              -j DROP
iptable  -L                                      -n
```

动作

```powershell
ACCEPT 		匹配到规则的数据包允许通过。
DROP 		匹配到规则的数据包不允许通过。
RETURN 		返回调用链：匹配到的A链中的B规则到此为止，然后返回到主链上，继续直接进行下一链规则匹配。
SNAT 		修改数据包的源地址，仅在nat表中有效，涉及到的链有 POSTROUTING 和 INPUT。
REDIRECT 	用于数据包的重定向动作，仅在nat表中有效，涉及到的链有 POSTROUTING 和OUTPUT。
REJECT 		终止数据包匹配并返回错误数据包，涉及到的链有 INPUT、FORWARD和OUTPUT。
MASQUERADE 	用于动态分配数据包的ip地址--即地址伪装，仅在nat表中有效，涉及到的链有POSTROUTING。
DNAT 		修改数据包的目标地址，仅在nat表中有效，涉及到的链有 PREROUTING 和OUTPUT。
```

```powershell
root@ubuntu-42:~ # iptables -vnL
Chain INPUT (policy ACCEPT 1230 packets, 75527 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  114  9576 DROP       0    --  *      *       10.0.0.11            0.0.0.0/0           

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination 
```

```
字段 					说明
pkts 			在当前规则中，被命中的数据包数量
bytes 			在当前规则中，被命中的总流量大小
target 			动作，目标，DROP 表示丢弃
prot 			具体协议
opt 			选项
in 				数据从哪个网络设备进来
out 			数据从哪个网络设备上出去
source 			源，可以是主机IP，网段，anywhere 表示不受限
destination 	目标，可以是主机IP，网段，anywherer 表示不受限
```

```powershell
iptables -t filter -A INPUT -j REJECT						不指定匹配条件，则全部拒绝
iptables -t filter -I INPUT 2 -s 127.0.0.1 -j ACCEPT		在第二行插入规则
iptables -D INPUT -s 127.0.0.1 -j ACCEPT					按照名字清除规则
iptables -t filter -I INPUT 2 -i lo -j ACCEPT				指定入口设备是本地回环网卡
iptables -t filter -R INPUT 2 -s 127.0.0.1 -j ACCEPT		替换第二条规则

```

![image-20241218175742371](D:\桌面\mage.md\笔记\5day-png\20、命令梳理.png)

```powershell
iptables -t filter -A INPUT -d 10.0.0.110 -j REJECT			目标ip地址规则限制
iptables -t filter -R INPUT 3 ! -d 10.0.0.110 -j REJECT		目标ip地址规则限制取反
iptables -t filter -A INPUT -d 10.0.0.110 -p icmp -j REJECT	通过 -p 选项对目标ip地址进行协议级别的限制
```

```powershell
使用tcp协议和端口拒绝服务
iptables -t filter -A INPUT -s 10.0.0.11 -d 10.0.0.110 -p tcp --dport 21:23 -j REJECT
在一条规则中指定两个端口
iptables -t filter -A INPUT -s 10.0.0.11 -d 10.0.0.110 -p tcp -m multiport --dports 22,80 -j REJECT
拒绝 10.0.0.11-10.0.0.100 这89个IP 的80端口的访问
iptables -t filter -A INPUT -m iprange --src-range 10.0.0.11-10.0.0.100 -p tcp --dport 80 -j REJECT
仅有 mac 地址是 00:50:56:22:fc:2c 的主机才能访问
iptables -t filter -A INPUT -d 10.0.0.110 -m mac --mac-source 00:50:56:22:fc:2c -j ACCEPT
iptables -t filter -A INPUT -j REJECT
设置出口规则，在返回的数据包中，跳过前62字节的报文头，如果内容中出现 baidu，则拒绝返回
iptables -t filter -A OUTPUT -m string --algo kmp --from 62 --string "baidu" -j REJECT\
在ubuntu中设置本地时间 9:00 - 12:00 ，UTC时间是 1:00 - 4:00
iptables -t filter -A INPUT -m time --timestart 01:00 --timestop 4:00 -s 10.0.0.11 -j REJECT
并发量不允许超过2,超过就拒绝
iptables -t filter -A INPUT -m connlimit --connlimit-above 2 -j REJECT
添加 icmp 放行规则，前10个数据包不处理，后面每分钟放行20个，另需要默认的拒绝策略规则
iptables -t filter -A INPUT -p icmp -m limit --limit 20/minute --limit-burst 10 -j ACCEPT
iptables -t filter -A INPUT -p icmp -j REJECT
```

```powershell
己连接的客户端可以继续通信
iptables -t filter -A INPUT -m state --state ESTABLISHED -j ACCEPT
iptables -t filter -A INPUT -m state --state NEW -j REJECT
```

## 面试题：有一台服务器无法建立新的连接，查看系统日志有如下提示，请问是什么问题，应该如何解决

```powershell
[root@rocky ~]# tail -3 /var/log/messages
Nov 16 08:50:22 rocky kernel: nf_conntrack: nf_conntrack: table full, dropping packet
Nov 16 08:50:23 rocky kernel: nf_conntrack: nf_conntrack: table full, dropping packet
Nov 16 08:50:23 rocky kernel: nf_conntrack: nf_conntrack: table full, dropping packet
```

答案

```powershell
1 加大连接限制
vi /etc/sysctl.conf
net.nf_conntrack_max = 393216
net.netfilter.nf_conntrack_max = 393216

2 降低 nf_conntrack timeout时间
vi /etc/sysctl.conf
net.netfilter.nf_conntrack_tcp_timeout_established = 300
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
......
```





## SNAT&DNAT

在 Linux 中，`SNAT` (Source Network Address Translation) 和 `DNAT` (Destination Network Address Translation) 都是由 `iptables` 或 `nftables` 实现的网络地址转换 (NAT) 功能。它们用于改变 IP 包头中的源地址或目标地址。这两个 NAT 操作在不同的链上工作，其工作方式和位置与流量的流动方向密切相关。

### 1. **SNAT（源地址转换）**

**链：`POSTROUTING` 链**

- **为什么在 `POSTROUTING` 链上工作：** `SNAT` 用于修改出站流量的源 IP 地址。也就是说，它适用于那些离开本地主机或本地网络的数据包，改变这些包的源 IP 地址以便进行路由或与外部网络进行通信。
  - 当数据包从主机发送到外部网络时，`SNAT` 在数据包离开本地网络之前执行。这通常发生在内核的路由决定之后、数据包即将离开接口之前。因此，`SNAT` 必须在数据包离开系统之前进行修改，所以它位于 `POSTROUTING` 链上。
- **应用场景：**
  - **Internet 共享**：例如，一台内部网络中的机器访问外部 Internet 时，使用 `SNAT` 来将其源地址改为路由器的公共 IP 地址。
  - **负载均衡**：当多个服务器共享一个公共 IP 地址时，可以使用 `SNAT` 来修改源地址，使其看起来像是同一个公共 IP 地址在进行通信。

**例子：**

```
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 192.168.1.1
```

这条规则会将出站的流量的源地址转换为 `192.168.1.1`，其中 `-o eth0` 表示该规则应用于通过 `eth0` 网卡发出的流量。

------

### 2. **DNAT（目标地址转换）**

**链：`PREROUTING` 链**

- **为什么在 `PREROUTING` 链上工作：** `DNAT` 用于修改进入本地主机的流量的目标 IP 地址。它发生在数据包进入网络并且路由决策之前。
  - 数据包刚到达路由器或主机时，`DNAT` 就会执行，它修改目标地址以决定将数据包转发到哪个内部服务器或目的地。
  - 在数据包到达系统之前，`DNAT` 可以改变目标 IP 地址，以便将流量引导到正确的主机或端口。
- **应用场景：**
  - **端口转发**：例如，路由器将从外部网络到达 `eth0` 的某个端口的流量转发到内网的特定主机上。
  - **负载均衡**：多个内部服务器共享一个公共 IP 地址，通过 `DNAT` 将进入的流量转发到不同的服务器上，通常是通过端口号或其他标识。

**例子：**

```
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10
```

这条规则会将到达 `80` 端口的流量重定向到内部地址 `192.168.1.10`。

------

### 总结：

- **SNAT**（源地址转换）通常在 `POSTROUTING` 链上工作，因为它处理离开主机的流量，并改变源地址，目的是使出站流量能正确地通过路由器或防火墙。
- **DNAT**（目标地址转换）通常在 `PREROUTING` 链上工作，因为它在数据包进入主机或网络前修改目标地址，目的是将流量导向内部网络的正确目标。

这两种 NAT 操作都是为了使流量能够正确地流动到预期的目标，并且它们的执行时机与数据包流向网络的方向和所需的修改类型密切相关。



## 通过iptables抓取TCP三次握手的SYN_SENT状态和SYN-REV状态？

在 TCP 握手过程中，客户端和服务器之间的连接建立涉及到多个状态，包括 `SYN_SENT` 和 `SYN-RECEIVED` 等。在 `iptables` 中，我们可以通过结合 `conntrack` 模块来监控和抓取这些连接状态。`conntrack` 是 Linux 内核提供的一种用于跟踪和管理网络连接的机制。

### 监控 TCP 状态

- **SYN_SENT**: 该状态出现在客户端，它表示客户端已经发送了一个 `SYN` 包并等待服务器的 `SYN-ACK` 响应。
- **SYN_RECEIVED**: 该状态出现在服务器端，它表示服务器已经接收到客户端的 `SYN` 包，并且发送了一个 `SYN-ACK` 包。

### 通过 `iptables` 抓取这两个状态的流量

1. **启用 `conntrack` 模块**： 首先确保你的系统启用了 `conntrack` 模块。通常，现代的 Linux 系统都已经启用它。
2. **抓取 SYN_SENT 状态**： 对于 `SYN_SENT` 状态，我们需要匹配尚未收到 `SYN-ACK` 包的连接。可以通过 `conntrack` 的 `state` 匹配来实现。
3. **抓取 SYN_RECV 状态**： 对于 `SYN_RECV` 状态，我们可以监视服务器接收到客户端的 `SYN` 包时的状态。

### 具体步骤：

#### 1. 抓取 `SYN_SENT` 状态

在客户端向服务器发起连接时，客户端会进入 `SYN_SENT` 状态，等待 `SYN-ACK` 包。

```
iptables -A INPUT -p tcp --syn -m state --state NEW -j LOG --log-prefix "SYN_SENT: "
```

- `--syn`：匹配带有 `SYN` 标志的数据包。
- `-m state --state NEW`：表示连接正在尝试建立，且该连接尚未建立。
- `LOG --log-prefix "SYN_SENT: "`：将匹配到的数据包记录到日志，前缀为 `SYN_SENT: `。

#### 2. 抓取 `SYN_RECV` 状态

当服务器收到客户端的 `SYN` 包，并回复 `SYN-ACK` 包时，它会进入 `SYN_RECV` 状态。

```
iptables -A INPUT -p tcp --syn -m state --state ESTABLISHED -j LOG --log-prefix "SYN_RECV: "
```

- `--syn`：匹配带有 `SYN` 标志的数据包。
- `-m state --state ESTABLISHED`：表示这是一个已建立的连接，但由于我们要捕获的是 `SYN-ACK` 包，因此匹配该状态也是有效的。
- `LOG --log-prefix "SYN_RECV: "`：将匹配到的数据包记录到日志，前缀为 `SYN_RECV: `。

#### 3. 其他状态（例如 `SYN_ACK`）：

如果你希望捕获 TCP 握手的完整过程，可以进一步细化状态匹配。例如，如果你希望查看 `SYN-ACK` 数据包，可以匹配：

```
iptables -A INPUT -p tcp --syn -m state --state ESTABLISHED -j LOG --log-prefix "SYN-ACK: "
```

#### 4. 查看连接追踪（`conntrack`）

如果你想更深入地了解连接的状态，可以使用 `conntrack` 命令。`conntrack` 允许你查看当前的网络连接状态。

```
# 查看当前的连接状态
conntrack -L
```

或者，使用以下命令过滤特定的 TCP 连接状态：

```
conntrack -L | grep SYN_RECV
```

这将列出所有当前处于 `SYN_RECV` 状态的连接。

#### 5. 使用 `iptables` 和 `conntrack` 组合监控

你也可以在 `iptables` 中结合 `conntrack` 模块监控特定状态的连接。例如：

```
iptables -A INPUT -p tcp -m conntrack --ctstate NEW,ESTABLISHED -j LOG --log-prefix "TCP Handshake: "
```

这将捕获所有新的和已建立的连接，并将它们记录到日志中。你可以根据需要查看状态。

### 总结：

- SYN_SENT: 客户端发起连接时处于此状态。
  - 使用 `iptables` 捕获带有 `SYN` 标志的 `NEW` 状态包。
- SYN_RECV: 服务器收到 SYN 包并回应时处于此状态。
  - 使用 `iptables` 捕获带有 `SYN` 标志的 `ESTABLISHED` 状态包。

通过这些方式，你可以使用 `iptables` 配合 `conntrack` 模块来捕获和监视 TCP 三次握手过程中的 `SYN_SENT` 和 `SYN_RECV` 状态。







## iptables的四表五链中，表的概念是什么，有哪些表，作用是什么，链的概念是什么，有哪些链，作用是什么？

在 `iptables` 中，**表**和**链**是两个核心的概念，它们共同构成了 Linux 防火墙的工作机制。理解这些概念对配置和管理 `iptables` 非常重要。

### 1. **表（Table）** 的概念

在 `iptables` 中，**表**是用于组织规则的容器。每个表有一个特定的用途，负责处理特定类型的网络流量。`iptables` 支持多个表，每个表都包含一组规则（链）来处理流量。常见的表有 **filter**、**nat**、**mangle**、**raw** 和 **security** 表。每个表的作用和特点如下：

#### 常见的表

1. **filter 表**（默认表）

   - **作用**：`filter` 表是 `iptables` 中最常用的表，负责对进入或离开主机的数据包进行过滤和处理。
   - **主要用途**：控制哪些数据包能够进入或离开网络接口。
   - **默认链**：`INPUT`（输入链）、`OUTPUT`（输出链）、`FORWARD`（转发链）。

   **示例**：通过 `filter` 表定义的规则可以实现如“允许或拒绝某个 IP 地址或端口的访问”之类的功能。

2. **nat 表**

   - **作用**：`nat` 表用于处理与网络地址转换（Network Address Translation, NAT）相关的操作。
   - **主要用途**：修改数据包的源 IP 地址或目的 IP 地址，通常用于实现内网和外网之间的通信（如 SNAT、DNAT）。
   - **默认链**：`PREROUTING`（路由前）、`POSTROUTING`（路由后）、`OUTPUT`（输出链）。

   **示例**：通过 `nat` 表可以设置 SNAT（源地址转换）或 DNAT（目的地址转换），例如实现端口转发或反向代理。

3. **mangle 表**

   - **作用**：`mangle` 表用于对数据包进行特殊的修改，如修改包的 TTL（Time-to-Live）、TOS（Type of Service）等字段。
   - **主要用途**：用来进行数据包标记、改变 IP 包头的特性等。
   - **默认链**：`PREROUTING`（路由前）、`POSTROUTING`（路由后）、`FORWARD`（转发）、`INPUT`（输入）、`OUTPUT`（输出）。

   **示例**：通过 `mangle` 表可以修改数据包的 IP 标记，用于 QoS（服务质量）管理。

4. **raw 表**

   - **作用**：`raw` 表用于处理原始数据包，主要用于配置连接跟踪（connection tracking）的规则。
   - **主要用途**：主要用于关闭或调整连接跟踪机制，绕过某些连接跟踪操作，适用于非常规操作。
   - **默认链**：`PREROUTING`（路由前）、`OUTPUT`（输出链）。

   **示例**：通过 `raw` 表可以关闭连接跟踪，以避免某些数据包被无意义地跟踪。

5. **security 表**

   - **作用**：`security` 表用于与 Linux 安全模块（如 SELinux）进行交互，管理与安全相关的规则。
   - **主要用途**：用于访问控制或安全策略，适用于更高级的安全配置。
   - **默认链**：`INPUT`（输入链）、`OUTPUT`（输出链）、`FORWARD`（转发链）。

   **示例**：通过 `security` 表，可以为数据包应用基于安全策略的控制。

### 2. **链（Chain）** 的概念

**链**（Chain）是规则的实际执行路径。每个表包含一个或多个链，链由一系列的规则组成，规则会逐个检查数据包，直到满足某个规则条件为止。如果没有规则匹配，则会根据链的默认策略来决定数据包的处理方式。每个链的作用和功能不同。

#### 常见的链

1. **INPUT 链**
   - **作用**：用于处理进入本机的数据包。
   - **适用场景**：当数据包目标是本机时，`INPUT` 链会处理这些数据包。
2. **OUTPUT 链**
   - **作用**：用于处理从本机发出的数据包。
   - **适用场景**：当数据包的源地址是本机时，`OUTPUT` 链会处理这些数据包。
3. **FORWARD 链**
   - **作用**：用于处理转发的数据包，即本机并不是源地址或目标地址，而是中转的路由器。
   - **适用场景**：适用于路由器或网关，数据包不是发往本机的，而是被转发到其他地方。
4. **PREROUTING 链**
   - **作用**：在路由决策之前，对数据包进行处理。
   - **适用场景**：适用于 `nat` 和 `mangle` 表，用于修改数据包的目标地址或源地址等。
5. **POSTROUTING 链**
   - **作用**：在路由决策之后，对数据包进行处理。
   - **适用场景**：适用于 `nat` 和 `mangle` 表，用于修改数据包的源地址或进行其他操作。
6. **OUTPUT 链**（与 `OUTPUT` 链同名）
   - **作用**：处理本机发出的数据包。区别在于，`OUTPUT` 链是用于数据包经过路由后返回到用户空间前的处理。

#### 例子：

- **filter 表的链**：`INPUT`、`OUTPUT`、`FORWARD`。
- **nat 表的链**：`PREROUTING`、`POSTROUTING`、`OUTPUT`。
- **mangle 表的链**：`PREROUTING`、`POSTROUTING`、`FORWARD`、`INPUT`、`OUTPUT`。

### 3. **表与链的关系**

- **表**是用来组织规则的集合，表示不同类型的网络处理（如过滤、NAT、包修改等）。
- **链**是规则的顺序执行路径，每个链包含了一系列的规则，这些规则根据数据包的特征逐一检查并处理。

### 4. **总结：**

- 表（Table）：
  - `filter`：默认表，用于过滤流量。
  - `nat`：用于处理网络地址转换（NAT）。
  - `mangle`：用于修改数据包头部信息。
  - `raw`：用于原始数据包处理，主要与连接跟踪机制有关。
  - `security`：与安全相关的表，通常与 SELinux 配合使用。
- 链（Chain）：
  - 每个表包含一个或多个链，每个链由一系列规则组成。
  - 常见链有：`INPUT`、`OUTPUT`、`FORWARD`、`PREROUTING`、`POSTROUTING`。

这些表和链的组合使得 `iptables` 能够灵活地进行各种网络流量处理，确保网络的安全和流量的有效管理。





## iptables中表与链之间的关系是怎样的



在 `iptables` 中，**表**和**链**是两个核心的概念，它们相互配合工作，共同实现流量的处理和过滤。理解它们之间的关系对于有效地配置和管理防火墙规则至关重要。

### 1. **表（Table）和链（Chain）的关系**

- **表**（Table）是用于组织和管理规则的容器，每个表负责特定类型的流量处理。
- **链**（Chain）是规则的实际执行路径，定义了如何处理数据包。每个表包含多个链，每个链包含多个规则，规则按照顺序逐个检查数据包，直到找到匹配的规则为止。

#### 每个表包含的链

1. **filter 表**：

   - 默认表，用于过滤数据包。

     filter表有三个主要链：

     - `INPUT`：处理进入本机的数据包。
     - `OUTPUT`：处理从本机发出的数据包。
     - `FORWARD`：处理转发的数据包（当数据包通过本机转发到其他主机时）。

2. **nat 表**：

   - 主要用于处理网络地址转换（NAT）。

     nat表有三个主要链：

     - `PREROUTING`：在路由决策之前处理数据包。通常用于目的地址转换（DNAT）。
     - `POSTROUTING`：在路由决策之后处理数据包。通常用于源地址转换（SNAT）。
     - `OUTPUT`：处理本机发出的数据包，通常用于设置源地址转换（SNAT）。

3. **mangle 表**：

   - 主要用于修改数据包的头部信息（如修改 TOS、TTL 等）。

     mangle表有五个链：

     - `PREROUTING`：在路由决策之前处理数据包，常用于改变数据包的标记、TTL 等。
     - `POSTROUTING`：在路由决策之后处理数据包，常用于修改数据包的标记、TTL 等。
     - `FORWARD`：处理转发的数据包。
     - `INPUT`：处理进入本机的数据包。
     - `OUTPUT`：处理从本机发出的数据包。

4. **raw 表**：

   - 主要用于关闭或调整连接跟踪机制，处理原始数据包。
   - raw表有两个链：
     - `PREROUTING`：处理进入的数据包。
     - `OUTPUT`：处理从本机发出的数据包。

5. **security 表**：

   - 用于与 Linux 安全模块（如 SELinux）交互，处理安全相关的规则。

     security表有三个链：

     - `INPUT`：处理进入本机的数据包。
     - `OUTPUT`：处理从本机发出的数据包。
     - `FORWARD`：处理转发的数据包。

### 2. **表和链的执行顺序**

- 当数据包到达 `iptables` 时，首先会根据 **路由表**（或目标 IP 地址）来选择对应的 **链**。根据所选链的规则和配置，数据包会逐条检查并执行规则。
- 每个表和链的执行流程如下：
  1. filter 表：
     - 数据包经过 `INPUT`、`OUTPUT` 或 `FORWARD` 链，按照规则进行过滤。
  2. nat 表：
     - 数据包首先进入 `PREROUTING` 链，可能会经过 DNAT（目的地址转换）。然后，如果数据包需要发送出去，会进入 `POSTROUTING` 链，可能会经过 SNAT（源地址转换）。如果数据包是从本机发出的，它也会进入 `OUTPUT` 链。
  3. mangle 表：
     - 数据包会根据需要进入 `PREROUTING`、`POSTROUTING`、`FORWARD`、`INPUT` 或 `OUTPUT` 链进行修改，通常用于修改 TTL、TOS 等头部信息。
  4. raw 表：
     - 处理原始数据包，通常用于关闭或调整连接跟踪。数据包会进入 `PREROUTING` 或 `OUTPUT` 链。
  5. security 表：
     - 用于安全策略的应用，数据包会进入 `INPUT`、`OUTPUT` 或 `FORWARD` 链进行处理。

### 3. **表与链的关系示例**

#### 示例 1：过滤流量（使用 `filter` 表）

- **表**：`filter`
- 链：INPUT、OUTPUT、FORWARD
  - 数据包会根据目标判断是进入 `INPUT`（目标为本机）、`OUTPUT`（源地址为本机）还是 `FORWARD`（经过本机转发）。

#### 示例 2：源地址转换（SNAT，使用 `nat` 表）

- **表**：`nat`

- 链：

  ```
  POSTROUTING
  ```

  - 数据包经过路由决策后进入 `POSTROUTING` 链，源地址被修改（源地址转换，SNAT）。

#### 示例 3：修改 TTL 值（使用 `mangle` 表）

- **表**：`mangle`

- 链：

  ```
  PREROUTING
  ```

  - 数据包在路由决策前进入 `PREROUTING` 链，TTL（生存时间）被修改。

### 4. **总结**

- **表**是对不同类型流量处理的分类，每个表有不同的作用（如过滤流量、网络地址转换、数据包修改等）。

- 链

  是每个表中定义的规则链，用于逐一检查数据包，并根据规则决定是否接受或修改数据包。

  - 每个表包含多个链，常见链有 `INPUT`、`OUTPUT`、`FORWARD`、`PREROUTING`、`POSTROUTING` 等。
  - 每个链针对不同类型的流量（如输入、输出、转发、路由前后等）进行处理。

这种结构使得 `iptables` 非常灵活，能够根据需要定制不同类型的网络流量处理策略。
