



# 1Liunx用户组和权限管理

## 1.1用户基础

### 1.1.1用户

- 管理员：root,0

- 普通用户：1-60000 自动分配

​				系统用户：对守护进程获取资源进行权限分配

​					1-999

​				登录用户：给用户进行交互式登录使用

​					1000+

### 1.1.2用户组

- 管理员组：root,0

- 普通用户组：

​				系统用户：对守护进程获取资源进行权限分配

​					1-999

​				登录用户：给用户进行交互式登录使用

​					1000+

- 用户和用户组：
  - 用户的主要组(primary group)
    - 用户必须属于一个且只有一个主组，默认创建用户时会自动创建和用户名同名的组，做为用户的主要组，由于此组中只有一个用户，又称为私有组
  - 用户的附加组(supplementary group)：  
    - 一个用户可以属于零个或多个辅助组，附属组

- 安全上下文:
  - SELinux 是 Linux 内核中的安全模块，使用 **强制访问控制 (MAC)** 策略来限制进程与资源之间的访问。SELinux 安全上下文由一组标识符组成，标识符用于描述对象（文件、进程、端口等）的安全属性

- 用户登录和退出

- 关于用户登录和退出，在linux系统中，会有对应的日志文件进行相关的记录动作：

  ```
  Rocky9的日志：
  	/var/log/secure
  	/var/log/audit/audit.log
  
  ubuntu的日志：
  	/var/log/auth.log
  ```

  

### 1.1.3配置文件

```
/etc/passwd：	#用户及其属性信息(名称、UID、主组ID等）
/etc/shadow：	#用户密码及其相关属性
/etc/group：		#组及其属性信息
/etc/gshadow：	#组密码及其相关属性

注意：
shadow 只有 root 才有读和写权限，这样就保证了密码文件的安全性
passwd和group 文件每个用户都有读权限
```

- passwd文件格式
  - 文件包含所有用户的基本信息，它用于存储每个用户的账号信息（不包括密码，因为密码保存在 `/etc/shadow` 文件中）

```
cat /etc/passwd
#root:x:0:0:root:/root:/bin/bash
	#每一列都是以 : 为分隔符
```

|    项     |      名称      | 含义                                                         | 备注 |
| :-------: | :------------: | ------------------------------------------------------------ | ---- |
|   root    |     用户名     | 这是账户的名称，`root` 是 Linux 系统的超级用户，拥有系统的全部权限 |      |
|     x     |      密码      | 这个位置通常是用户密码的存储位置。在现代 Linux 系统中，用户的密码通常存储在 `/etc/shadow` 文件中，因此这里显示为 `x` 表示密码在另一个地方存储 |      |
|     0     | UID（用户id）  | 这是用户的 UID（User ID），`0` 是 root 用户的 UID，表示该用户是系统的超级用户 |      |
|     0     |  GID（组id）   | 这是用户的 GID（Group ID），`0` 是 root 组的 GID，表示该用户所属的组是 root 组 |      |
|   root    | 用户全名或注释 | 这个字段通常用来存储用户的全名或其他信息。在这里，它再次显示为 `root` |      |
|   /root   |     主目录     | 这是该用户的主目录，`/root` 是 root 用户的默认家目录。普通用户的家目录通常是 `/home/username`，而 root 用户的家目录是 `/root` |      |
| /bin/bash |   默认 shell   | 这是用户登录后默认使用的 shell 程序，`/bin/bash` 表示使用 Bash 作为命令行解释器 |      |

- shadow文件格式
  - 文件用于存储用户的密码和其他敏感的用户信息，它只有 root 用户或管理员才有权限查看。

```
cat /etc/shadow
#root:$6$VftC9gu7HNAEIc79$VdxTdcGlQFjiP.RdyZG6js7fHnzj3WDIDb3v6JGqyD1zhh6Rw/J1ik7 M06UFNhZ9ihxjIiKpBwz2UEYc8pOsa.::0:99999:7:::
```

| 项             | 名称             | 含义                                                         |
| -------------- | ---------------- | ------------------------------------------------------------ |
| root           | 用户名           | 表示用户账户名称，与 `/etc/passwd` 文件中的用户名对应        |
| $6$VftC9gu7... | 密码             | 存储加密后的密码，如果该内容为空登录时不需要密码；如果为！！，表示禁止登录 |
| null           | 密码上次修改日期 | 这里为空，表示密码上次修改的日期未指定，通常以 Unix 时间（从 1970 年 1 月 1 日起的天数）表示。 |
| 0              | 最小密码修改间隔 | 指定两次密码修改的最小天数。在这里为 `0`，表示用户可以随时更改密码 |
| 99999          | 密码有效期       | 指定密码的有效天数。`99999` 表示密码几乎永久有效，用户不必定期更改密码 |
| 7              | 密码过期提醒     | 密码过期前多少天提醒用户。在这里是 `7`，表示在密码过期前 7 天开始提醒用户 |
| null           | 密码失效期       | 为空，表示没有设置密码失效日期                               |
| null           | 账户失效日期     | 为空，表示账户不设定失效日期                                 |
| null           | 保留字段         | 为空，保留字段，目前未使用                                   |

- 生成随机密码

  ```
  tr -dc '[:alnum:]' < /dev/urandom | head -c 12
  #deAuRZYsk4Rer
  tr -dc 'aDhdkashdhkh.sdb,.s,;aw/,.f/af+' < /dev/urandom |head -c 12
  #,;hfsbd+wb,h
  		#/dev/urandom：这是一个随机数生成器，会不断生成随机数据。这里用它作为输入来产生随机字符
  		#tr -dc '[:alnum:]'：tr 是字符转换工具。-d 参数表示删除，-c 表示取反。[:alnum:] 表示“字母和数字”的字符集合。也就是说，这一部分的作用是过滤掉随机数据中的非字母和数字字符，只保留字母和数字
  		#head -c 12：从上一步过滤后的结果中取前12个字符
  		
  openssl rand -base64 9
  #OLfYYajtS7IR
  		#l生成9个字节的随机数，然后通过base64进行编码【3个字节转换成base64的4个字符】
  
  mkpasswd 接收信息后，生成一个密码串
  mkpasswd 040620
  $y$j9T$aYMakJayFV5rdpB4Bk2JV1$SLtphvsEpW35KbZoHTHjIpAcrNNLvoMYN8v3nKvH.x7
  ```

- group文件格式

  - 文件用于存储系统的组信息。它包含系统中所有组的基本配置和成员信息。

  ```
  cat /etc/group
  #root:x:0:
  #组名:密码:GID:用户1,用户2
  ```

  | 项   | 名称                | 含义                                                         |
  | ---- | ------------------- | ------------------------------------------------------------ |
  | root | 组名                | 组的名称。                                                   |
  | x    | 密码                | 这个字段通常用于存储密码，但通常表示为 `x`，因为在现代系统中不常用组密码。 |
  | 0    | 组的GID（组标识符） | 组的 GID（组标识符），这里是 `0`，表示这是最高权限的组       |
  | null | 组成员              | 后面没有显示组成员的内容，表示该组当前没有其他用户被直接列为其成员。通常，`0` 后面的字段用于列出属于该组的用户 |

- gshdow文件格式

  - 文件存储了组的安全信息和管理配置。与 `/etc/group` 不同，它包含加密的组密码和管理员信息，只有系统管理员可以读取或修改此文件。

  ```
  cat /etc/gshdow
  #root:::
  #组名:密码:管理员:组成员
  
  ```

  | 项   | 名称   | 含义                                                         |
  | ---- | ------ | ------------------------------------------------------------ |
  | root | 组名   | 组名，表示这是 `root` 组。                                   |
  | null | 密码   | 密码字段为空，表示没有设置组密码。通常，大多数系统不会使用组密码，因而保持为空。 |
  | null | 管理员 | 管理员字段为空，表示没有指定此组的管理员。                   |
  | null | 组成员 | 组成员字段为空，表示此组没有额外的成员。                     |

  

### 1.1.4配置操作

- vipw和vigr命令——编辑用户、用户组相关配置

  ```
  vipw|vigr 命令用于编辑/etc/passwd，/etc/group，/etc/shadow，/etc/gshadow
  	vipw 默认编辑 /etc/passwd 文件
  	vigr 默认编辑 /etc/group 文件
   
  常见选项：
      -g|--group 		#编辑 group 文件
      -p|--passwd 	#编辑 passwd 文件
      -s|--shadow 	#编辑 /etc/shadow 或 /etc/gshadow 文件
  ```

  

- pwck命令——检查用户相关配置

  ```
  命令使用：
  	pwck [options] [passwd_file | shadow_file]
  	pwck 选项 文件的路径
   	对用户相关配置文件进行检查，默认检查文件为 /etc/passwd
  选项
      -q|--quiet 				#只报告错误，忽略警告
      -r|--read-only 			#显示错误和警告，但不改变文件
      -R|--root CHROOT_DIR 	#chroot 到的目录
      -s|--sort 				#通过 UID 排序项目
  ```

  

- grpck命令——检查用户组相关配置

  ```
  命令使用：
  	grpck [options] [group [gshadow]]
  	grpck 选项 组名
  	对用户组相关配置文件进行检查，默认检查文件为 /etc/group，/etc/gshadow
   选项：
      -r|--read-only 			#显示错误和警告，但不改变文件
      -R|--root CHROOT_DIR 	#chroot 到的目录
      -s|--sort 				#通过 UID 排序项目
  ```



- |         |                  |
  | ------- | ---------------- |
  | vipw    | vim /etc/passwd  |
  | vigr    | vim /etc/group   |
  | vipw -s | vim /etc/shadow  |
  | vigr -s | vim /etc/gshadow |

  



## 1.2用户管理

### 1.2.1用户组管理（1）

- groupadd——命令可以创建用户组

  ```
  格式
  	groupadd [options] GROUP
  	groupadd 选项 组名
  选项
  	-r|--system		#创建一个系统组
  	-f|--force		#如果组已存在，则成功退出
  	-g|--gid GID	#新建组时未指定组ID，是系统自动分配的；加上-g后指定组ID。
  ```

  ```
  groupadd -r hhh			#创建一个系统组hhh（组ID<1000）。
  groupadd -r zzz			#创建一个组zzz，如果组存在则退出。
  groupadd kkk -g 10086	#创建一个组kkk,指定组ID为10086
  ```

  

- groupmod——命令用于修改group属性

  ```
  格式
  	groupmod [options] GROUP
  	groupmod 选项 组名
  选项
  	-g|--gid GID #将组 ID 改为 GID
      -n|--new-name NEW_GROUP #改名为 NEW_GROUP
      -o|--non-unique #允许使用重复的 GID
      -p|--password PASSWORD #将密码更改为(加密过的) PASSWORD
  ```

  ```
  groupmod -g 10087 group		#更改组group的id为10087
  groupmod -n hhz group		#更改组group的名字为hhz
  
  ```

  

- groupdel——命令可以删除用户组

  ```
  格式
  	groupdel [options] GROUP
  	groupdel 选项 用户组名
  选项
  	-f|--force #强制删除，即使是用户的主组也强制删除组,但会导致无主组的用户不可用无法登录		
  ```

  ```
  groupdel group		#删除组group
  groupdel -f group1 	#强制删除组group1
  ```

- getent——查看用户，用户组

  ```
  格式
  getent group h		#等同于 cat /etc/group | grep h
  			#查看用户h
  ```

  

### 1.2.2用户创建

- id——用于查看用户的基本信息

  ```
  格式
  	id [username]
  ```

- useradd—— 命令可以创建新的Linux用户

  ```
  命令使用
      useradd [options] LOGIN
      uesradd 选项 用户名
      useradd -D # 查看默认的配置属性
  常用选项
      -u|--uid UID 				#指定UID
      -g|--gid GID 				#指定用户组，-g groupname|--gid GID
      -m|--create-home 			#创建家目录，一般用于登录用户（在rocky中不家m也创建家目录）
      -p|--password PASSWORD		#设置密码，这里的密码是以明文的形式存在于/etc/shadow 文件中
      -r|--system 				#创建系统用户
      -d|--home-dir HOME_DIR 		#指定家目录，可以是不存在的，指定家目录，并不代表创建家目录
      -M|--no-create-home 		#不创建家目录，一般用于不用登录的用户（在Ubuntu中不加m就不创建家目录）
      -D							#查看用户创建的基本属性
      -s|--shell SHELL 			#指定 shell，可用shell在/etc/shells 中可以查看
      -k|--skel SKEL_DIR #指定家目录模板，创建家目录，会生成一些默认文件，如果指定，就从该目录
  复制文件，默认是/etc/skel/，要配合-m
  	-l|--no-log-init #不将用户添加到最近登录和登录失败记录，前面讲到的3a认证审计，就在此处   
  lastlog|lastb|cat /var/log/secure
  ```

  ```
  useradd z			#创建用户z
  useradd k -u 1066	#创建用户k,并指定UID为1066
  useradd zk -g kkkk	#创建用户zk,并指定用户组为kkkk
  useradd -m zkk		#创建用户zkk,并创建用户的家目录（rocky可以不加-m,也创建家目录）
  ```

  ```
  ls -l /etc/skel/		#里面的文件也新增到新创建的用户的家目录下
  ```

  ```
  useradd -D				#查看用户创建的基本属性
  useradd -D -s /bin/sh	#更改用户属性信息
  ```

  

- newusers——批量创建用户

  ```
  vim user.txt		#编辑创建用户的文件
  #sswang1:123456:1024:1024::/home/sswang1:/bin/bash
  #sswang2:123456:1025:1025::/home/sswang2:/bin/bash
  newusers user.txt	#批量创建用户
  ```

  

### 1.2.3密码拆解

```
root@rocky9:~ $ getent shadow hzk
hzk:$6$egRirfE7u0oq6uO7$1HZu2Dah3IJtspBYoPbp2bXpIIID2SBXME8h9CCdJJxt02rh9NpHDAzBx224xV3rHJQeVZnqQLz62KfTTwJL40::0:99999:7:::

密码格式：
$6$egRirfE7u0oq6uO7$1HZu2Dah3IJtspBYoPbp2bXpIIID2SBXME8h9CCdJJxt02rh9NpHDAzBx224xV3rHJQeVZnqQLz62KfTTwJL40
第一部分: 指定加密算法
	$6
第二部分: 指定二次校验选项 - salt 盐值 
	$egRirfE7u0oq6uO7
第三部分：通过 加密算法(salt值 + 密码) 加密后的一段字符串
	$1HZu2Dah3IJtspBYoPbp2bXpIIID2SBXME8h9CCdJJxt02rh9NpHDAzBx224xV3rHJQeVZnqQLz62KfTTwJL40
```

- crpy——主要用于加密和解密文件或密码。

- openssl——验证用户密码

  - getent shadow hzk
    hzk:$6$egRirfE7u0oq6uO7$1HZu2Dah3IJtspBYoPbp2bXpIIID2SBXME8h9CCdJJxt02rh9NpHDAzBx224xV3rHJQeVZnqQLz62KfTTwJL40::0:99999:7:::
  - openssl passwd -salt egRirfE7u0oq6uO7 -6 040620
    $6$egRirfE7u0oq6uO7$1HZu2Dah3IJtspBYoPbp2bXpIIID2SBXME8h9CCdJJxt02rh9NpHDAzBx224xV3rHJQeVZnqQLz62KfTTwJL40

  

### 1.2.4更改密码

```
命令使用	
	passwd 用户名
```

```
在rocky 和 centos系列的系统中，可以使用如下方式：
 echo '密码' | passwd --stdin 用户名
 passwd --stdin 用户名 <<<密码
```

- 批量修改密码

  - ```
    vim passwd.txt		#编辑修改密码的用户和密码
    #hzk:040620
    #kk:040620
    chpasswd < file		#用chpasswd命令，修改密码
    
    echo hzk:123456 | chpasswd		#单行修改密码
    ```

    

- centos修改密码

  - ```
    echo 123456 | passwd --stdin kk			#修改kk密码为123456
    passwd --stdin kk <<<123456 			#修改kk密码为123456
    ```

    

### 1.2.5用户属性

- usermod——命令用于进行用户属性的修改动作

```
格式：
	usermod [options] LOGIN
	usermod 选项 用户名
选项：
	-g|--gid GROUP 					#修改组
	-L|--lock 						#锁定用户帐号，在/etc/shadow 密码栏的增加 !
	-d|--home HOME_DIR 				#修改家目录
	-e|--expiredate EXPIRE_DATE 	#修改过期的日期，YYYY-MM-DD 格式
	-f|--inactive INACTIVE 			#密码过期之后，账户被彻底禁用之前的天数，0 表示密码过期立即禁
用，-1表示不使用此功能
	-u|--uid UID 					#修改 UID
	-s|--shell SHELL 				#修改 shell
	-m|--move-home 					#将家目录内容移至新位置，和 -d 一起使用
	-U|--unlock 					#解锁用户帐号，将 /etc/shadow 密码栏的!拿掉
	-c|--comment COMMENT 			#修改注释
	-a|--append GROUP 				#将用户追加至上边 -G 中提到的附加组中，并不从其它组中删除此用户
```

### 1.2.6用户删除

- userdel——可以对linux系统用户进行删除操作

  ```
  格式：
  	userdel [options] LOGIN
  	userdel 选项 用户名
  选项
      -f|--force #强制删除，哪怕用户正在登录状态
      -r|--remove #删除家目录和邮件目录
  注意：
  	正在使用的用户不可以删除
  ```

  

### 1.2.7用户切换

- ```
  样式1：su 用户名
  	非登录式切换，即会读取目标用户的配置文件，不改变当前工作目录，即不完全切换如果不加用户名，表示切换到root用户
  样式2：su - 用户名
  	登录式切换，会读取目标用户的配置文件，切换至自已的家目录，即完全切换表示启动一个新的shell进行登录，然后使用 指定用户的环境信息
  	如果普通用户是 /bin/false 或者 /sbin/nologin | /usr/sbin/nologin 的时候，无法切换但是，如果通过 -s 指定 /bin/bash，可以登录，无论是rocky还是ubuntu
  ```

  

### 1.2.8 密码策略

- chage 可以修改用户密码策略

```
格式
	chage [options] LOGIN
	chage 选项 用户名
选项
    -d LAST_DAY 				#更改密码的时间
    -m|--mindays MIN_DAYS		#设置密码或凭证在更改后必须等待的最少天数
    -M|--maxdays MAX_DAYS		#设置密码或凭证的最大有效天数。超过该天数，系统可能会要求用户更改密码。
    -W|--warndays WARN_DAYS		#设置警告天数，提醒用户即将到期的密码或凭证。
    -I|--inactive INACTIVE 		#密码过期后的宽限期
    -E|--expiredate EXPIRE_DATE #用户的有效期
    -l 							#显示密码策略
```

```
chage hzk						#修改密码策略
chage -l hzk					#查看修改后的密码策略

```



### 1.2.9 用户组管理（2）

- gpasswd——更改组成员和密码 - 站在用户的角度

  ```
  格式：
  	gpasswd [option] GROUP
  	gpasswd 选项 用户名 组名
  选项：
      -a|--add USER 			#向组中添加用户
      -d|--delete USER		#从组中移除用户
      -r|--delete-password	#删除组密码
      -R｜--restrict			#向其成员限制访问组 GROUP
      -M｜--members USER,... 	#批量加组
      -A｜--administrators ADMIN,... #批量设组管理员
  注意：
  	组没有密码的情况下，加组只能由root来操作
  ```

  ```
  gpasswd web				#为组添加密码
  gpasswd -d hzk web		#从组里面移除用户
  ```

  

- groupmems ——可以管理附加组的成员关系-站在用户组的角度

  ```
  格式：
  	groupmems [options] [action]
  	groupmens 选项 用户组 用户名
  一般选项
      -g|--group groupname 	#更改为指定组 (只有root)
      -a|--add username 		#指定用户加入组
      -d|--delete username 	#从组中删除用户
      -p|--purge 				#从组中清除所有成员
      -l|--list 				#显示组成员列表
  ```

  ```
  groupmems -g web -l			#查看web组有哪些用户
  groupmems -g web -a hzk		#为web组添加用户hzk
  groupmems -g web -d hzk		#从web删除用户hzk
  ```

  

- groups ——可查看用户组关系

  ```
  格式：
  	groups [OPTION].[USERNAME]...
  	groups 选项 用户名
  ```

  ```
  groups hzk					#查看组hzk
  ```

  

### 1.2.10其他配置

- ```
  用户登录配置文件 /etc/login.defs
  grep -Ev '#|^$' /etc/login.defs
  ```

- ```
  查看创建用户的配置文件 /etc/default/useradd
  cat /etc/default/useradd
  ```

- ```
  定制新用户基本配置和属性
  ls /etc/skel/
  注意：
  	该目录保存了创建新用户后需要拷贝到 /home 目录下所需的相关文件。
  ```

- ```
  定制用户登录的显示页面
  cat /etc/motd
  	用于设置用户登录系统后显示给用户的提示信息，该文件默认为空
  ```

  

## 1.3权限管理

### 1.3.1权限体系

```
默认情况下，Linux系统中的umask值通常为022。这个值意味着：
	对于新创建的文件，默认权限为644（即所有者具有读写权限，所属组和其他用户只有读权限）。
	对于新创建的目录，默认权限为755（即所有者具有读、写、执行权限，所属组和其他用户具有读和执行权限）
```

文件操作权限

```
Linux文件权限主要分为三种类型：
读权限（Readable）：
	允许用户读取文件内容 或 查看目录中的文件列表。
	简写：r、4
写权限（Writable）：
	允许用户修改文件内容 或 在目录中创建、删除或重命名文件。
	简写：w、2
执行权限（eXcutable）：
	对于文件，表示该文件是可执行的程序；对于目录，表示用户可以进入该目录。
	简写：x、1
```

文件用户权限

```
文件权限被分为三个级别，分别对应不同的用户群体：
	所有者（Owner）：
		文件或目录的创建者，拥有对文件或目录的最高权限。
		简写：u
    所属组（Group）：
    	文件或目录被分配到的用户组，组内的所有成员将继承该组对该文件或目录的权限。
    	简写：g 
    其他用户（Others）：
		既不是文件所有者也不是所属组成员的所有其他用户。
 		简写：o
```

```
程序访问文件时的权限，取决于此程序的发起者
    进程的发起者，同文件的属主：则应用文件属主权限
    进程的发起者，属于文件属组；则应用文件属组权限
    应用文件“其它”权限
```

查看文件属性信息：

```
-rw-r--r--. 1 root root 2706 Oct 24 11:24 /etc/bashrc
```

| -rw-r--r--.        | 1      | root         | root         | 2706                   | Oct 24 11:24       | /etc/bashrc      |
| ------------------ | ------ | ------------ | ------------ | ---------------------- | ------------------ | ---------------- |
| 文件权限和类型信息 | 链接数 | 文件的所有者 | 文件的所属组 | 文件大小，以字节为单位 | 文件的最后修改时间 | 文件的路径和名称 |

### 1.3.2属性操作

- chown ——命令可以修改文件的属主，也可以修改文件属组

```
命令格式：
    chown [OPTION]... [OWNER][:[GROUP]] FILE...
    chown 选项 文件新所有者 文件路径
    chown 选项 文件新所有者 文件的新所属组 要更改所有者或组的文件或目录（可以是一个或多个文件或目录）
    chown [OPTION]... --reference=RFILE FILE...
    	[OPTION]... 选项
    	--reference=RFILE将目标文件 (FILE...) 的所有者和组设置为与 RFILE 相同的所有者和组。
		FILE...要更改所有者和组的目标文件或目录，可以是一个或多个文件或目录。
注意：
 owner 表示所有者
 :group 和 .group 表示归属组
 owner. 和 owner: 表示所有者和归属组一样
 
 选项：
 	-R|--recursive #递归操作
```

```
chown hzk /root/a/a.txt		#将a.txt的所有者改为hzk
chown hzk: /root/a/b.txt	#将b.txt的所有者和归属组改为hzk
chown 1000 /root/a/c.txt	#将c.txt的所有者改为UID为1000的用户
chown -R hzk /root/a		#将目录下的所有文件的所属组都改为hzk
```



- chgrp—— 命令可以只修改文件的属组

```
命令格式：
    chgrp [OPTION]... GROUP FILE...
    	GROUP	指定新的组名称，将目标文件或目录的组归属更改为该组。
    	FILE...	指定要更改组的文件或目录，支持多个文件或目录。
    chgrp [OPTION]... --reference=RFILE FILE...
    	--reference=RFILE	指定参考文件 RFILE，将目标文件的组归属更改为与此参考文件相同的组。
    	FILE...		要更改组归属的文件或目录，可以是一个或多个文件或目录。
常用选项
 -R|--recursive #递归操作
```

```
chgrp hzk /root/a/a.txt		#修改a.txt的属组为hzk
```



- chmod 命令可以修改文件的操作权限

  ```
  命令格式
      chmod [OPTION]... MODE[,MODE]... FILE...
      chmod [OPTION]... OCTAL-MODE FILE...
      chmod [OPTION]... --reference=RFILE FILE...
  		MODE：指定文件权限的模式，可以使用符号模式（如 u+r）或八进制模式（如 644）来表示。
  			符号模式：指定文件权限的增减。
  				u：用户（所有者）
  				g：组
  				o：其他人
  				a：所有用户（相当于 ugo）
  				+：添加权限
  				-：移除权限
  				=：设置权限
  					例如：u+r 表示给所有者添加读权限。
  			八进制模式：直接指定文件的权限。
  				4 表示读权限 (r)
  				2 表示写权限 (w)
  				1 表示执行权限 (x)
  			权限组合成一个 3 位数，例如 644 表示所有者有读写权限，组和其他人只有读权限
  		FILE...
  			要更改权限的文件或目录，支持多个文件和目录。
  常用选项
   	-R|--recursive #递归操作
   操作实例：
   	u+r # 属主加读权限
      g-x # 属组去掉执行权限
      ug=rx # 属主属组权限改为读和执行
      o= # other用户无任何权限
      a=rwx # 所有用户都有读写执行权限
      u+r,g-x # 同时指定
  ```

  ```
  chmod 777 /root/a/dir1
  chmod +x /root/a/dir2
  chmod u+x,o-x /root/a/dir3
  chmod u=rwx /root/a/dir4
  chmod ug=rx /root/a/dir5
  ```

  

|      | 目录                       | 文件                       |
| ---- | -------------------------- | -------------------------- |
| r    | 具有ls命令权限             | 具有查看这个文件内容的权限 |
| w    | 创建和删除的权限           | 编辑文件的权限             |
| x    | 没有进入资格其余的权限无效 | 通过路径执行的权限         |

### 1.3.3默认权限

​		linux系统在创建文件的时候，会有一个默认的权限。普通文件的权限是 644，普通目录文件的权限是755。在Linux系统中，文件创建时的默认权限与umask（用户文件创建掩码）紧密相关。umask决定了在创建新文件或目录时哪些权限位将被屏蔽（即不允许），从而间接决定了新文件或目录的默认权限。

​		默认情况下，Linux系统中的umask值通常为022。这个值意味着：对于新创建的文件，默认权限为644（即所有者具有读写权限，所属组和其他用户只有读权限）。对于新创建的目录，默认权限为755（即所有者具有读、写、执行权限，所属组和其他用户具有读和执行权限）

​		文件：-rw-rw-rw-（即666，所有用户都具有读写权限,不需要执行）

​		目录：drwxrwxrwx（即777，所有用户都具有读、写、执行权限）

当创建新文件或目录时，实际权限是默认权限减去umask值后的结果。例如，

​	如果umask值为022，

​		则新文件的权限为666 - 022 = 644，

​		新目录的权限为777 - 022 = 755。

修改umask方法

```
格式：
	umask [-p] [-S] [mode]
	mode:要修改的mode值
选项 
    -p #如果省略 MODE 模式，以可重用为输入的格式输入
    -S #以字符显示
```



### 1.3.4特殊权限

```
	在Linux文件系统中，除了基本的读（r）、写（w）、执行（x）权限外，还存在三种特殊权限，它们分
别是SUID（Set User ID）、SGID（Set Group ID）和Sticky Bit。这些特殊权限用于在特定情况下提
供更灵活的权限控制。
```

d rwx r-x rw-

 

|      |                 |              |
| ---- | --------------- | ------------ |
| rwx  | x——SUID的位置   | 所有者权限   |
| r-x  | x——GUID的位置   | 归属组权限   |
| rw-  | -——STICKY的位置 | 其他用户权限 |

- SUID:

- 解读：

  - 以/usr/bin/passwd文件为例，Linux系统中的/usr/bin/passwd文件就具有SUID权限。这使得普通用户能够修改自己的密码，因为passwd程序在执行时会暂时获得root权限，从而能够写入只有root才能访问的/etc/shadow文件。

  - ```
    chmod u+s /usr/bin/cat
    #切换到普通用户，发现普通用户也可以执行cat命令，查看/etc/shadow文件
    ```

- SGID

  - 功能：SGID权限可以应用于可执行文件或目录。对于可执行文件，它与执行SUID文件类似，但影响的是执行者=的组身份。然而，在实际应用中，SGID更多地被用于目录，以便在该目录下创建的新文件自动继承目录的组身份。表现： 在ls -l命令的输出中，如果"目录的组"执行权限位是s（小写），则表示该目录具有SGID权限。

  - ```
    chmod g+s a/hzk/hzk1
    #可以发现在hzk1目录下创建文件，文件的所属组都是hzk1的所属组
    ```

- STICKY

  - Sticky Bit权限仅对目录有效。当一个目录被设置为Sticky Bit时，只有该目录的所有者、文件的所有者或root用户才能删除或重命名该目录下的文件。这有助于防止其他用户删除或移动他们不拥有但可能有权访问的文件。表现：在ls -l命令的输出中，如果目录的"其他用户"执行权限位是t（小写），则表示该目录具有Sticky Bit权限。

  | 权限   | 字符表示 | 八进制表示 | 备注                                               |
  | ------ | -------- | ---------- | -------------------------------------------------- |
  | SUID   | s        | 4          | 如果原属主没有可执行权限，再加SUID权限，则显示为s  |
  | SGID   | s        | 2          | 如果原属组没有可执行权限，再加SGID权限，则显示为s  |
  | STICKY | t        | 1          | 如果other没有可执行权限，再加STICKY权限，则显示为t |

  ```
  SUID设定方式: u+s、u-s、4xxx
  SGID设定方式: g+s、g-s、2xxx
  STICKY设定方式: o+t、o-t、1xxx
  ```

  ```
  chmod o-t /sticky/dir
  #目录添加了STICKY属性，那么用户只能删除自己的文件（root账号除外）
  ```

  

### 1.3.8特殊权限

- chattr——命令用于改变文件或目录的扩展属性。这些属性可以增强文件的安全性，防止被意外修改、删除

  或重命名。

  ```
  格式
  	chattr [+-=][ASacdistu...] 文件或目录名称
  选项
  	-p project # 设置文件项目编号
      -R # 递归执行
      -V # 显示过程，并输出chattr 版本
      -f # 不输出错误信息
      -v version # 设置版本
  常用动作
  	+表示添加属性，-表示移除属性，=表示设置唯一属性，覆盖其他所有属性。
  常用属性
      a：只允许追加数据，不能删除或修改文件内容。
      i：设置文件为不可修改，即不能删除、重命名、修改内容或添加链接。
      s：同步写入磁盘，修改立即生效，而非默认的非同步方式。
      u：当文件被删除时，其内容仍保留在磁盘上，直到空间被其他文件覆盖。
      A：访问文件时不更新访问时间。
      c：自动压缩文件。
      d：防止目录被dump备份。
  ```

- lsattr——命令用于查看文件或目录的扩展属性。

  - ```
    格式
    	lsattr [选项] 文件或目录
    常用选项
    	-a：显示所有文件和目录的属性，包括以.开头的隐藏文件。
        -d：如果目标是目录，则只显示目录本身的属性，而不显示目录内文件的属性。
        -R：递归地列出目录及其所有子目录和文件的属性。
    ```

  - 注意事项：
    - 使用chattr和lsattr命令时，需要确保有足够的权限，通常是以root用户身份执行。
    - 某些文件系统可能不支持所有chattr属性，具体取决于文件系统的类型和版本。
    - 修改文件或目录的特殊属性可能会对系统的正常操作产生影响，因此请谨慎使用这些命令。

### 1.3.9 ACL

```
传统意义层面的文件系统级别的 User + Group + Other + RWX 的授权体系，主要针对的对象是以
文件为基本单位的。其实并不能实现 一个文件针对不同的用户设定形式各异的操作权限。
 	Linux系统的ACL（Access Control Lists，访问控制列表）是一种用于控制对文件和目录访问权限
的机制。它扩展了传统的文件权限模型，为系统管理员提供了更加灵活和精细的权限控制手段。
 	- CentOS7 默认创建的xfs和ext4文件系统具有ACL功能
 	- CentOS7 之前版本，默认手工创建的ext4文件系统无ACL功能，需手动增加
 	- 对于ubuntu系统来说，如果没有的话，需要安装 acl 命令
```

- setfacl 可设置ACL权限

```
命令格式：
	setfacl [-bkndRLPvh] [{-m|-x} acl_spec] [{-M|-X} acl_file] file ...
常见选项
    -m|--modify=acl         #修改acl权限
    -x|--remove=acl         #删除文件acl 权限
    -b|--remove-all         #删除文件所有acl权限
    --set=acl               #用新规则替换旧规则，会删除原有ACL项，用新的替代，
							#一定要包含UGO的设置，不能象 -m一样只有 ACL    
一般选项
    -M|--modify-file=file 	#从文件读取规则
    -X|--remove-file=file 	#从文件读取规则
    -k|--remove-default     #删除默认acl规则
    --set-file=file         #从文件读取新规则
    --mask                	#重新计算mask值
    -n|--no-mask           	#不重新计算mask值
    -d|--default          	#在目录上设置默认acl
    -R|--recursive       	#递归执行
    -L|--logical          	#将acl 应用在软链接指向的目标文件上，与-R一起使用
    -P|--physical        	#将acl 不应用在软链接指向的目标文件上，与-R一起使用
注意：
	ACL 也可以对 mask 进行操作，但是它的作用想要有效的限制还是比较高的。
		 - mask只影响除所有者和other的之外的人和组的最大权限
		 - mask需要与用户的权限进行逻辑与运算后，才能变成有限的权限
         - 用户或组的设置必须存在于mask权限设定范围内才会生效也就是说，对于mask，acl 不欢迎你，也没有什么意义。
```

- getfacl—— 可查看设置的ACL权限

```
命令格式：
	getfacl file ...
```

- 设置ACL

```
root@rocky9:/ $ mkdir /a/acl
root@rocky9:/ $ echo "acl" > /a/acl/acl.txt
root@rocky9:/ $ getfacl /a/acl/acl.txt
getfacl: Removing leading '/' from absolute path names
# file: a/acl/acl.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--

root@rocky9:/ $ setfacl -m u:hzk:- /a/acl/acl.txt
root@rocky9:/ $ getfacl /a/acl/acl.txt
getfacl: Removing leading '/' from absolute path names
# file: a/acl/acl.txt
# owner: root
# group: root
user::rw-
user:hzk:---
group::r--
mask::r--
other::r--

root@rocky9:/ $ ll /a/acl/acl.txt
-rw-r--r--+ 1 root root 4 Oct 29 19:52 /a/acl/acl.txt
root@rocky9:/ $ ll /a/acl/acl.txt
-rw-r--r--+ 1 root root 4 Oct 29 19:52 /a/acl/acl.txt
root@rocky9:/ $ su - kk1 -c "cat /a/acl/acl.txt"
/etc/profile: line 79: 1111: command not found
acl

```

- 修改ACL写权限

```
root@rocky9:/ $ setfacl -m u:kk1:rw /a/acl/acl.txt
root@rocky9:/ $ getfacl /a/acl/acl.txt
getfacl: Removing leading '/' from absolute path names
# file: a/acl/acl.txt
# owner: root
# group: root
user::rw-
user:hzk:---
user:kk1:rw-
group::r--
mask::rw-
other::r--

[root@rocky9 ~]# su - kk1 -c "cat /a/acl/acl.txt"
acl
[root@rocky9 ~]# su - hzk -c "echo 123 >> /a/acl/acl.txt"
/etc/profile: line 79: 1111: command not found
-bash: line 1: /a/acl/acl.txt: Permission denied

```

- 给组加ACL实践

  ```
  root@rocky9:/ $ getent group kkkk
  kkkk:x:1005:hzk
  root@rocky9:/ $ setfacl -m g:hzk:rwx /a/acl/acl.txt
  root@rocky9:/ $ getfacl /a/acl/acl.txt
  getfacl: Removing leading '/' from absolute path names
  # file: a/acl/acl.txt
  # owner: root
  # group: root
  user::rw-
  user:hzk:rwx
  user:kk1:rw-
  group::r--
  group:hzk:rwx
  mask::rwx
  other::r--
  
  root@rocky9:/ $ su - hzk -c "echo 123 >> /a/acl/acl.txt"
  root@rocky9:/ $ su - hzk -c "cat /a/acl/acl.txt"
  acl
  123
  123
  ```

- ACL规则的移除

  ```
  root@rocky9:/ $ setfacl -x u:hzk /a/acl/acl.txt
  root@rocky9:/ $ setfacl -x g:hzk /a/acl/acl.txt
  root@rocky9:/ $ setfacl -x u:kk1 /a/acl/acl.txt
  root@rocky9:/ $ getfacl /a/acl/acl.txt
  getfacl: Removing leading '/' from absolute path names
  # file: a/acl/acl.txt
  # owner: root
  # group: root
  user::rw-
  group::r--
  mask::r--
  other::r--
  
  ```

  

- 移除所有定制ACL权限 - 恢复默认的acl权限

  ```
  [root@rocky9 ~]# setfacl -b /a/acl/acl.txt
  ```

  

- 同时定制多属性信息

  ```
  通过 --set 同时定制多个属性信息
  root@rocky9:/ $ setfacl --set u::rw,u:hzk:rw-,u:kk1:r--,g::-,o::- /a/acl/acl.txt
  root@rocky9:/ $ getfacl /a/acl/acl.txt
  getfacl: Removing leading '/' from absolute path names
  # file: a/acl/acl.txt
  # owner: root
  # group: root
  user::rw-
  user:hzk:rw-
  user:kk1:r--
  group::---
  mask::rw-
  other::---
  
  
  
  ```










20250316