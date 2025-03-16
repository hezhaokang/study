# 21、ansible

## 软件部署

ubuntu

```powershell
更新软件源
apt update
apt install -y software-properties-common
安装ansible的专用软件源
add-apt-repository --yes --update ppa:ansible/ansible
安装ansible软件
apt install -y ansible
```

rocky/openeuler/centos

```powershell
yum install epel-release
yum install ansible
```



## 软件环境

```powershell
/usr/bin/ansible 			# ansible主命令
/usr/bin/ansible-config 	# ansible 配置管理命令
/usr/bin/ansible-console 	# ansible交互式命令
/usr/bin/ansible-doc 		# ansible模块帮助命令
/usr/bin/ansible-galaxy 	# ansible 扩展命令，用于获取别人的优秀的角色模板文件
/usr/bin/ansible-playbook 	# ansible 任务管理工具
/usr/bin/ansible-pull 		# ansible 反模式工作命令
/usr/bin/ansible-vault 		# ansible 文件专用加密工具
```

```powershell
通用格式：ansible <目标主机> 参数
专用格式：ansible <目标主机> -m 模块 -a 模块参数
命令方式：在命令行执行ansible的方式叫"Ad-Hoc",肯定还有其他执行方式。
主要功能：ansible的主要功能都是通过各种各样的插件模块来实现特有功能的。
拓展内容：ansible的默认模块叫command
```

```powershell
/etc/ansible/ansible.cfg
 主配置文件，配置ansible工作特性,也可以在项目的目录中创建此文件,当前目录下如果也有
ansible.cfg,则此文件优先生效,建议每个项目目录下,创建独有的ansible.cfg文件
/etc/ansible/hosts
 主机清单文件
/etc/ansible/roles/
 存放角色的目录
```

```powershell
ANSIBLE_CONFIG 		环境变量
./ansible.cfg   	当前目录下的ansible.cfg
~/.ansible.cfg 		当前用户家目录下的.ansible.cfg
/etc/ansible/ansible.cfg 	系统默认配置文件
注意：
 	对于ansible来说，他们的优先级是按照从上到下排列的，软件的根目录下的配置文件是最后的承载效果了。
```

## 主配置文件

```powershell
root@ubuntu24-13:~# grep -Ev '#|^$' ansible.cfg
[defaults] 					# 默认的配置段
[inventory] 				# 主机列表的配置段
[privilege_escalation] 		# 特权升级配置段
[paramiko_connection] 		# 插件连接配置段
[ssh_connection] 			# ssh连接配置段
[persistent_connection] 	# 长连接配置段
[accelerate] 				# 效率配置段
[selinux] 					# selinux安全配置段
[colors] 					# 颜色显示配置段
[diff] 						# 信息显示配置段
```

```powershell
该配置文件的基本格式如下
[配置段]
属性  = 值
```

## 实践

### command模块

```powershell
对于默认模块来说，我们不需要-m来指定模块，可以对目标主机使用"-a"传入一个命令参数，来执行查看本机上的信息，命令格式如下：
 	ansible <目标主机> -a 模块参数

结果显示：
 	基本信息：ansible操作是通过一个"provided hosts"的形式来检查的目标主机，只有找到主机后才执行
 	主机属性：默认的主机列表有两个属性：
 	all 默认表示所有主机列表
 	localhost 默认表示本机
 	
拓展信息：
 	主机列表文件是 /etc/ansible/hosts
 	- 绿色：执行成功并且不需要做改变的操作
 	- 黄色：执行成功并且对目标主机做变更
 	- 红色：执行失败
```

### ping模块

```powershell
ansible localhost -m ping
单行显示
ansible localhost -m ping -o
显示更多信息，采用 -vv 或者 -vvv
ansible localhost -m ping -vv
```

### ansible-doc命令

```powershell
常见参数
 -a 显示所有帮助信息，该参数已经被 --metadata-dump 参数替代
 -l 显示所有模块
 -s 显示简单的帮助信息
```



### ansible-config命令

```powershell
ansible-config命令用于专门管理ansible的配置信息
ansible-config -h
结果显示：
 	view用于查看ansible配置
 	dump用于查看ansible的默认环境变量
 	list用于罗列ansible详细的配置项
 	init用于创建初始化配置项
```

### 主机清单

```powershell
在主机列表文件中，有这么几个特殊的属性值：
    all 表示所有主机
    ungrouped 所有没有组的主机
    localhost 表示本机
这几个常见的特殊属性值，可以用于我们的ansible指定目标主机时候的过滤条件。
```

```powershell
#免密码认证脚本
#!/bin/bash
# 设置环境变量
USER_NAME='root'
USER_HOME="/${USER_NAME}/.ssh"
SSH_CONFIG_FILE='/etc/ssh/ssh_config'
USER_PASSWD='040620'
HOSTADDR_PRE='10.0.0'
HOST_LIST="${HOSTADDR_PRE}.45 ${HOSTADDR_PRE}.44 ${HOSTADDR_PRE}.14"
HOSTNAME_LIST='ubuntu24-13 rocky9-12 ubuntu24-16'
HOSTS_FILE='/etc/hosts'
# 准备基本环境
base_env(){
  apt install expect -y
  [ -d ${USER_HOME} ] && rm -rf ${USER_HOME}
  ssh-keygen -t rsa -P "" -f ${USER_HOME}/id_rsa
  sed -i '/ask/{s/#/ /; s/ask/no/}' ${SSH_CONFIG_FILE}
}
# expect自动化交互过程
expect_auto(){
  remote_host=$1
  expect -c "
    spawn ssh-copy-id -i ${USER_HOME}/id_rsa.pub $1
    expect {
      \"*yes/no*\" {send \"yes\r\"; exp_continue}
      \"*password*\" {send \"$USER_PASSWD\r\"; exp_continue}
      \"*Password*\" {send \"$USER_PASSWD\r\";}
    } "
}
# 跨主机免认证环境
auth_auto(){
  for i in ${HOST_LIST} ${HOSTNAME_LIST}
  do
    expect_auto ${USER_NAME}@$i
    scp -rp ${HOSTS_FILE} ${USER_NAME}@$i:${HOSTS_FILE}
  done
}
# 设定主机名
hostname_set(){
  for i in ${HOSTNAME_LIST}
  do
    ssh ${USER_NAME}@$i "hostnamectl set-hostname $i"
  done
}
# 主函数执行
main(){
  # 基本环境准备
  base_env
  # 跨主机免密认证
  auth_auto
  # 设定主机名
  hostname_set
}
# 执行主函数
main
```

### sudo

```powershell
sudo免密码
root@rocky-14:~ # grep '^%' /etc/sudoers
%wheel   ALL=(ALL:ALL) NOPASSWD:ALL
root@rocky-14:~ # useradd -m putong
root@rocky-14:~ # echo putong:123456 | chpasswd
root@rocky-14:~ # usermod -G wheel putong
root@rocky-14:~ # id putong
uid=1001(putong) gid=1001(putong) groups=1001(putong),10(wheel)
```



### 目标匹配规则

```powershell
对于目标的匹配规则来说，他们的获取方式和表现样式主要有这么几种：
    匹配所有：all
    正则匹配：*(通配符)
    逻辑或：:(并集)，一般用于主机组
    逻辑与：:&(交集)，一般用于主机组
    逻辑非：:!(补集)，一般用于主机组
 	综合实践：上面所有样式的整合
```

命令执行过程

```powershell
1. 加载ansible配置文件 
 	2.12版本，默认加载的是 /etc/ansible/ansible.cfg 
 	之后的版本，默认加载的是 /root/ansible.cfg 
2. 加载指定模块：command
3. 获取目标主机，与主机列表进行匹配认证 			/etc/ansible/hosts
4. ansible指挥目标主机执行命令
  	4.1 控制端生成临时执行文件 				~/.ansible/cp
  	4.2 通过连接插件，与目标主机建立连接 		paramiko_ssh
  	4.3 通过插件功能将控制端的临时执行文件传输至目标主机的临时目录 	~/.ansible/tmp $HOME/.ansible/tmp/ansible-tmp-数字/XXX.PY文件
  	4.4 指挥目标主机给临时执行文件添加执行权限
  	4.5 目标主机执行临时文件并将结果返回给控制主机
  	4.6 目标主机删除临时执行py文件，然后断开连接
5. ansible控制端显示结果，并退出。
```

```
注意：
 	查看创建连接的数量时候，会显示执行5条连接，但是这个跟forks设置的并发量无关。
 	即使我们在执行Ansible的过程中，通过Ctrl + C 中断连接，~/.ansible/tmp 目录下也会自动清理掉
```



## 模块实践

### command

```powershell
command： ansible的默认模块，可以运行远程权限所有的shell命令
```

```powershell
argv:           	# 以列表或者字符串格式输入命令
chdir:             	# 执行命令前，先切换工作路径.
creates:         	# 依赖存在对象的命令，若被创建的对象已存在，则不执行该命令
free_form:     		# 以自由格式输入参数，这是一个强制的选项
removes:            # 依赖移除对象的命令，若被移除的对象存在，则执行该命令
stdin:           	# 设定标准输入作为参数.
warn:             	# 警告信息，需要ansible的配置文件的 command_warnings的属性值是yes.

Command模块在ansible使用的时候，标准的写法是：
 	ansible <目标主机> -m command -a '可执行命令'
注意：
 	由于ansible的默认模块就是command，所以远程执行命令的时候，一般会忽略-m选项，效果如下：
 	ansible <目标主机> -a '可执行命令'
 	可执行命令，就是我们平常在命令行输入的一些执行成功的命令,但是不允许是别名
 	command不支持"可执行命令"中的自定义变量和特殊符号(< > | ;&等)
```

### shell

```powershell
shell：command对于某些特殊的符号命令执行效果不好，专用的shell模块可以满足要求
```

```powershell
chdir:				# 执行命令前，先切换工作路径.
creates:			# 依赖存在对象的命令，若被创建的对象已存在，则不执行该命令
executable:			# 设置执行命令的shell类型
free_form:			# 以自由格式输入参数，这是一个强制的选项
removes:			# 依赖移除对象的命令，若被移除的对象存在，则执行该命令
stdin:				# 设定标准输入作为参数.
warn:				# 警告信息，需要ansible的配置文件的 command_warnings的属性值是yes.

Shell模块在ansible使用的时候，标准的写法是：
ansible <目标主机> -m shell -a '可执行命令'

注意：
	可执行命令，就是我们平常在命令行输入的一些执行成功的命令
	shell支持"可执行命令"中的所有效果
	shell在远程执行脚本的时候，必须远程主机存在，否则的话会报错
	
	 调用bash执行命令 类似 cat /tmp/test.md | awk -F'|' '{print $1,$2}' &> /tmp/example.txt 这些复杂命令，即使使用shell也可能会失败。
 	解决办法：写到脚本时，copy到远程，执行，再把需要的结果拉回执行命令的机器
```

### scripts

```powershell
scripts：批量的系统命令可以写成一个脚本文件，然后基于scripts的方式来运行，类似scp+shell的功能。
专门用于执行复杂脚本场景的功能模块

decrypt:               # 使用Vault控制源文件的自动解密
其他的参数，与上面一样


script模块在ansible使用的时候，标准的写法是：
ansible <目标主机> -m script -a '可执行命令'
注意：
 	script就是不但可以执行目标主机上的脚本文件，而且还能执行目标主机不存在的控制端特有的脚本文件
 	对于控制端的脚本文件执行的时候，必须使用executable
 	此模块不具有幂等性
```



### hostname

```powershell
该模块可以获取主机名相关的信息

hostname模块在ansible使用的时候，标准的写法是：
ansible <目标主机> -m hostname -a 'name=主机名'
注意：
 	主机名的设置一定要规范
 	这种方式更改的主机名，是立刻生效而且是永久生效的
```



### user

```powershell
user模块，主要是用于对远程主机的用户进行干预操作

常见属性
 	comment:		# 用户的描述信息.
    group:			# 设定用户的主组信息
    home:			# 设置用户家目录.
    password:		# 设置登录密码
    shell:			# 设定用户的专用shell类型
    state:			# 设定用户的存活状态
    system:			# 设定用户是否为系统用户
    uid:			# 设定用户uid
 	remove:			# 设定删除用户及家目录等数据,默认remove=no，推荐用yes
注意：
 	上面属性是常用的属性，其他属性我们会随着课程的展开，在特定的场景下遇到。
 	state主要有两个属性值absent(没有), present(存在)
 	groups 可以有多个值，多个值之间可以用逗号隔开
 	
hostname模块在ansible使用的时候，标准的写法是：
 	ansible <目标主机> -m user -a '属性1=值1 属性2=值2 ... 属性n=值n'
注意：
 	用户名的设置一定要提前做好规划
 	在工作中，uid的值一定要提前做好规划，而且必须唯一。
```



### group

```powershell
该模块应用于用户组的管理

group模块在ansible使用的时候，标准的写法是：
 	ansible <目标主机> -m group -a '属性1=值1 属性2=值2 ... 属性n=值n'
注意：
 	主机名的设置一定要规范，gid一定要唯一。
```



### cron

```powershell
cron模块主要做一些，类似于周期性任务相关的任务执行动作

常见属性
	day:		# 设定天效果 ( 1-31, *, */2, etc )
	disabled:	# 结合`state=present`实现禁用定时任务.
	hour:		#设定时效果 ( 0-23, *, */2, etc )
	job:		# 如果是env，内容是变量，如果是state=present，内容是定时任务.
	minute:		# 设定分钟效果 ( 0-59, *, */2, etc )
	month:		# 设定月效果 ( 1-12, *, */2, etc )
	name:		# 如果有 env，则是变量名，如果有state=present则是定时任务名.
	weekday:	# 周表示样式 ( 0-6 for Sunday-Saturday, *, etc )
注意：
 	上面是常用的属性
 	name和job最好作为强制选项

cron模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m cron -a '属性1=值1 属性2=值2 ... 属性n=值n'
注意：
	job的命令一定要规范，最好使用绝对路径。
	state=absent 表示删除
	disabled=yes|no|true|false 表示禁用或启用
```

### setup

```powershell
setup 主要是用来收集目标主机的属性信息的。

常用参数
	fact_path:		# 设定facts的文件路径
	filter:			# 指定过滤格式，可以正则形式.
注意：
 	常用的属性，也就上面标红的两个。
 	setup这个模块是通过调用ansible的facts组件来获取主机是所有属性信息的
 	
setup模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m setup -a '属性1=值1 属性2=值2 ... 属性n=值n'
注意：
	不加 -a的话，表示获取目标主机的所有属性信息
	过滤信息的规则一定要合理的设计。
	这个功能很重要，尤其是后期自动化管理的时候
```

### selinux

```powershell
selinux 模块用作对远程主机的 selinux 机制进行管理
```

```powershell
常用选项
configfile=/path/file 					#selinux 配置文件路径，默认/etc/selinux/config
policy=targeted|minimum|mls 			#在state值不为 disabled 时必选
state=disabled|enforcing|permissive 	#具体设置 
```



###  mount

```powershell
mount 模块用于管理远程主机的挂载

src=/path/device 									#要挂载的设备路径，可以是网络地址
path=/path/point 									#挂载点
state=absent|mounted|present|unmounted|remounted  	# absent 取消挂载,并删除永久挂载中的配置
 									# mounted 永久挂载,立即生效，挂载点不存在会自动创建
									# present 永久挂载，写配置文件，但不会立即生效
									# unmounted 临时取消挂载，不改变配置文件
									# remounted 重新挂载，但不会改变配置文件
fstab=/path/file 			#指定挂载配置文件路径，默认 /etc/fstab
fstype=str 					#设备文件系统 xfs|ext4|swap|iso9660...
opts=str 					#挂载选项

```



### sysctl

```powershell
sysctl 模块用来修改远程主机上的内核参数

name=str 					#参数名称
val=str 					#参数值
reload=yes|no 				#默认yes，调用 /sbin/sysctl -p 生效
state=present|absent 		#是否保存到文件，默认present
sysctl_file=/path/file 		#指定保存文件路径，默认 /etc/sysctl.conf
sysctl_set=yes|no 			#是否使用systctl -w 校验，默认no
```



### copy

```powershell
copy模块主要涉及到文件的拷贝动作

常见参数
backup:		# 创建一个包含时间戳信息的备份文件
dest:		# 设置目标的路径(强制选项)，格式与src一致
group:		# 设定文件属组信息.
mode:		# 设定文件属性信息.
owner:		# 设定文件属主信息.
src:		# 源文件的路径，平常使用的话，推荐使用相对路径，敏感文件推荐用
绝对路径
 
注意：
 	常用的属性，也就上面标红的六个。
 	备份的时候，使用默认的时间戳效果。
 	如果源文件是目录的话，则必须携带/后缀

copy模块在ansible使用的时候，标准的写法是：
 	ansible <目标主机> -m copy -a '属性1=值1 属性2=值2 ... 属性n=值n'
注意：
 	src源文件路径和dest目标文件的路径，最好格式一致
```

### fetch

```powershell
该模块的作用于copy的作用正好相反，它是从远程主机拉取文件到本地目录

常见参数
	dest:		# 设定目标文件的地址(强制选项)
	flat:		# 允许拼接方式创建文件名
	src:		# 设定源文件的地址(强制选项)
注意：
     常用的属性，也就上面标红的三个。
 	源文件和目标文件的格式最好一致。
 	src 当前的版本，只支持抓取远程的单个文件
 	dest 支持只能是本机的目录，抓取后的格式： "/dest/" + "目标主机/path/to/src"
 	
fetch模块在ansible使用的时候，标准的写法是：
ansible <目标主机> -m fetch -a '属性1=值1 属性2=值2 ... 属性n=值n'
```

### file

```powershell
file模块，用于管理远程主机的文件，操作远程文件的各种动作。

常见属性
mode:		# 设定文件权限属性.
owner:		# 设定文件属主.
path:		# 指定文件的路径(强制选项).之前版本可以用name和dest来代替
src:		# 指定文件的源路径.
state:		# 设定文件的状态，directory、file、absent、touch、link

file模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m file -a '属性1=值1 属性2=值2 ... 属性n=值n'
 
注意：
	在创建相关文件的话，如果涉及到父目录，必须确保父目录存在，否则会失败
```

### stat

```powershell
stat模块主要作用的场景是 文件状态查看等

常见参数
 	path:                 # 获取文件的路径，强制选项

stat模块在ansible使用的时候，标准的写法是：
 	ansible <目标主机> -m stat -a 'path=文件路径 属性1=值1 ... 属性n=值n'
注意：
 	文件的路径一定要正确。
```

###  Get_url

```powershell
用于将文件从http、https或ftp下载到被管理机节点上

常见参数：
 	url: 下载文件的URL,支持HTTP，HTTPS或FTP协议
 	dest: 下载到目标路径（绝对路径）
 	force: 如果yes，dest不是目录，将每次下载文件，如果内容改变，替换文件，否则只有文件不存在时才下载
 	checksum:  对目标文件在下载前进行摘要计算，只有文件内容正确，才会进行文件下载。

get_url模块在ansible使用的时候，标准的写法是：
 	ansible <目标主机> -m get_url -a 'url=文件的http路径 属性1=值1 ... 属性n=值n'
注意：
 	文件的url路径一定要正确。
```

###  archive

```powershell
archive模块，用于打包压缩保存在被管理节点

archive模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m archive -a '属性1=值1 ... 属性n=值n'
```

### unarchive

```powershell
unarchive模块主要用于压缩包的解压功能

常见参数
	copy：默认为yes，
		当copy=yes，拷贝的文件是从ansible主机复制到远程主机上，
		如果设置为copy=no，会在远程主机上寻找src源文件
	remote_src：和copy功能一样且互斥，yes表示在远程主机，不在ansible主机，no表示文件在ansible主机上
	src：源路径，可以是ansible主机上或远程主机上的路径，如果是远程主机上的路径，则需要设置copy=no
	dest：远程主机上的目标路径
	mode：设置解压缩后的文件权限
	
注意：
 	有时候，会因为ansible版本的问题，导致无法远程解压，可以通过 copy + shell方式来解决

unarchive模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m unarchive -a '属性1=值1 ... 属性n=值n'
```



### Lineinfile

```powershell
	ansible在使用sed进行替换时，经常会遇到需要转义的问题，而且ansible在遇到特殊符号进行替换时，存在问题，无法正常进行替换 。其实在ansible自身提供了两个模块：lineinfile模块和replace模块，可以方便的进行替换
	一般在ansible当中去修改某个文件的单行进行替换的时候需要使用lineinfile模块
重要选项 
 	regexp参数：使用正则表达式匹配对应的行，当替换文本时，如果有多行文本都能被匹配，则只有最后面被匹配到的那行文本才会被替换，当删除文本时，如果有多行文本都能被匹配，这么这些行都会被删除。
 	如果想进行多行匹配进行替换需要使用replace模块
 	
lineinfile模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m lineinfile -a '属性1=值1 ... 属性n=值n'
```



### Replace

```powershell
Replace模块相较于lineinfile模块来说，更倾向于sed命令的操作，尤其是多行匹配进行替换操作

replace模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m replace -a '属性1=值1 ... 属性n=值n'
```

### apt

```powershell
yum模块和apt模块分别用于 centos|redhat 和 ubuntu|debian 系统下的软件安装

这两个模块的核心选项基本上都是一样的，常见的选项有
    name:				# 指定安装软件.
    state:				# 软件状态：安装(present|installed)
   						# 移除(absent|removed)
						# 升级到最新版本latest
						# build-dep 安装依赖包
						# fix 修复
注意：
	安装多个包，将包名之间使用逗号隔开即可
	特殊语法：所有包都升级到最新版本
	ansible 10.0.0.16 -m apt -a "name=* state=latest"
其他选项
autoclean=yes|no 		#清除本地安装包，只删除己卸载的软件的 deb包
deb=/path/file.deb 		#指定deb包，可以是本地的，也可以是网络的
autoremove=yes|no 		#卸载依赖包,默认no
only_upgrade=yes|no 	#仅更新，不安装
update_cache=yes|no 	#更新索引
```

### apt_repository

```powershell
这个模块，主要用于配置远程客户端主机上的软件源信息的。

repo=str 				#具体源
filename=/path/file 	#具体文件名，默认加到 /etc/apt/sources.list 中
state=absent|present 	#absent 删除,present 新增，默认 present
update_cache=yes|no 	#yes 更新源，相当于执行 apt-get update
```



### apt_key

```powershell
此模块实现 apt 仓库的key 的配置管理，仅于用 debian 系列的环境中

url=URL 				#key文件路径
validate_certs=yes|no 	#是否校验https 的 ssl 证书，默认yes
state=absent|present 	#absent 删除，present 新增，默认 present
```



### yum

```powershell
yum|apt模块在ansible使用的时候，标准的写法是：
 	ansible <目标主机> -m yum|apt -a '属性1=值1 ... 属性n=值n'
 
注意：
 	在安装本地包的时候，使用的是name属性，而且目标主机的本地必须有相应的包
 	在安装本地包的时候，有时候可能会因为软件依赖的问题，我们可以基于disable_gpg_check=yes来忽略
```

```powershell
name=packagename 			#指定包名 name1,name2
state=absent|installed|latest|present|removed 																		#absent|removed 删除,installed|present 安装,latest 升级到最新版
list=packagename|installed|updates|available|repos 																	#此选项与name选项互斥，
 							#写具体包名是相当于执行 yum list --showduplicates packagename
download_dir=/path 			#指定下载目录
download_only=yes|no 		#只下载不安装，默认 no
update_only=yes|no 			#yes 仅更新,默认no
use_backend=auto|yum|yum4|dnf 
							#指定真实执行的命令,默认 auto
autoremove=yes|no 			#卸载依赖，仅在卸载时生效，默认no
disablerepo=repoid 			#排除某些仓库 repoid1,repoid2
enablerepo=repoid 			#从指定仓库中安装 repoid1,repoid2
validate_certs=yes|no 		#是否对包进行校验，默认yes
disable_gpg_check=yes|no 	#是否不对包进行校验，默认no
update_only=yes|no 			#只更新不安装,默认no

注意：yum 模块本身没有直接的选项来刷新缓存，但你可以通过执行 yum clean all 和 yum makecache 命令来达到这一目的。

```

### yum_repository

```powershell
此模块用来对远程主机上的 yum 仓库配置进行管理
常用选项
name=repoid 			#repoid
description=desc 		#描述信息
baseurl=url 			#仓库地址
enabled=yes|no 			#是否启用
gpgcheck=yes|no 		#是否启用gpgcheck，没有默认值，默认跟随 /etc/yum.conf 中的全局配置
gpgkey=/path/key 		#gpgkey路径
state=absent|present 	#absent删除， present安装，默认present
timeout=30 				#超时时长，默认30s
```

### service

```powershell
service模块的作用就是操作应用服务的。
主要选项
	enabled:	# 设置开机自启动
	name:		# 服务名称(强制选项).
	state:		# 设置服务的状态started|stopped|restarted|reloaded

service模块在ansible使用的时候，标准的写法是：
ansible <目标主机> -m service -a '属性1=值1 ... 属性n=值n'
```

### Setup

```powershell
setup 模块来收集主机的系统信息，这些 facts 信息可以直接以变量的形式使用

重要选项
	fact_path:		# 设定facts的文件路径
	filter:			# 指定过滤格式，可以正则形式.
注意
	setup这个模块是通过调用ansible的facts组件来获取主机是所有属性信息的
	如果主机较多，会影响执行速度，可以使用`gather_facts: no` 来禁止 Ansible 收集 facts 信息
	
setup模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m setup -a '属性1=值1 属性2=值2 ... 属性n=值n'
注意：
	不加 -a的话，表示获取目标主机的所有属性信息
	过滤信息的规则一定要合理的设计。
	这个功能很重要，尤其是后期自动化管理的时候
```



### debug

```powershell
debug模块的功能主要是我们调试环境时候，用到的一个模块，帮助我们获取一些更详细的信息

重要选项
	msg:			# 打印自定义消息
	var:			# 调试变量名称，与msg互斥
	verbosity:		# 显示更详细信息
注意：
	这个功能在命令行模式下作用不大，但是在高级playbook方法中使用的场景作用很大。

debug模块在ansible使用的时候，标准的写法是：
	ansible <目标主机> -m debug -a 'name=主机名'
```



### debug & msg

```powershell
debug里面无法将变量和msg同时使用
```







## playbook

```powershell
语法检查
ansible-playbook yaml-test.yaml --syntax-check
文件执行
ansible-playbook yaml-test.yaml
检查效果
ansible 10.0.0.13 -m shell -a "netstat -tnulp | egrep 'Proto|apache' "
```











20250316
