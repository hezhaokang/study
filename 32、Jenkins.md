# 31、Jenkins

## Jenkins部署与基本配置

jenkins是一款开源CI&CD软件，用于自动化各种任务，包括构建、测试和部署软件。

jenkins支持各种运行方式，可通过系统包、Docker或者通过一个独立的Java程序。

**官方网站**

```
https://jenkins.io/zh/
```

**官方文档**

```
https://www.jenkins.io/zh/doc/
```

Jenkins 是基于 Java 开发的一种开源的CI（Continuous integration持续集成）&CD (Continuous Delivery持续交付，Continuous Deployment持续部署)工具

Jenkins 只是一个调度平台,其本身并不能完成项目的构建部署

Jenkins 需要安装各种插件,可能还需要编写Shell,python脚本等才能调用和集成众多的组件来实现复杂的构建部署功能

**主要用途：**

- 持续、自动地构建/测试软件项目
- 监控一些定时执行的任务

**Jenkins特点：**

- 开源免费
- 跨平台，支持所有的平台
- master/slave支持分布式的build
- web形式的可视化的管理页面
- 安装配置简单
- 及时快速的提示和帮助
- 已有的1800+插件



Jenkins 项目产生两个发行线, 长期支持版本 (LTS) 和定期发布版本。

**稳定版 LTS**

LTS (长期支持) 版本每12周从常规版本流中选择，作为该时间段的稳定大版本。 每隔 4 周会更新迭代稳定的小版本，其中包括错误和安全修复反向移植。

下载链接

```
https://get.jenkins.io/
https://mirrors.tuna.tsinghua.edu.cn/jenkins/
```

- 常规版本

  每周都会发布一个新版本，为用户和插件开发人员提供错误修复和功能。

## Jenkins安装和启动

**Jenkins** **的安装**

Jenkins 支持多种部署和运行方式

Jenkins支持多种安装方法

- 包安装
- JAVA的WAR文件
- 容器运行

```
https://www.jenkins.io/zh/doc/book/installing/
```

**系统准备**、

```bash
#关闭防火墙和SELinux
略
#设置语言环境，防止后期Jenkins汉化出问题
[root@jenkins ~]# localectl set-locale LANG=en_US.UTF-8
```

### 安装

```bash
[root@ubuntu2404 ~]#apt update && apt install -y openjdk-17-jdk
[root@ubuntu2404 ~]#wget https://mirror.tuna.tsinghua.edu.cn/jenkins/debian-stable/jenkins_2.492.1_all.deb
[root@ubuntu2404 ~]#dpkg -i ./jenkins_2.492.1_all.deb
[root@ubuntu2404 ~]#systemctl status jenkins.service 
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-02-19 14:39:08 CST; 37s ago
   Main PID: 16864 (java)
      Tasks: 50 (limit: 4557)
     Memory: 552.2M (peak: 554.1M)
        CPU: 13.779s
     CGroup: /system.slice/jenkins.service
             └─16864 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080

Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: 8ad98603196d4ad7a7fc47bc85824f41
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: *************************************************************
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: *************************************************************
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: *************************************************************
Feb 19 14:39:08 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:08.045+0000 [id=33]        INFO        jenkins.InitReactorRunner$1#onAttained: Completed initial>Feb 19 14:39:08 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:08.074+0000 [id=23]        INFO        hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up a>Feb 19 14:39:08 ubuntu2404.wang.org systemd[1]: Started jenkins.service - Jenkins Continuous Integration Server.
Feb 19 14:39:13 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:13.422+0000 [id=48]        INFO        h.m.DownloadService$Downloadable#load: Obtained the updat>Feb 19 14:39:13 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:13.422+0000 [id=48]        INFO        hudson.util.Retrier#start: Performed the action check upd>lines 1-20/20 (END)

```

```bash
#密码存放路径
[root@ubuntu2404 ~]#cat /var/lib/jenkins/secrets/initialAdminPassword
8ad98603196d4ad7a7fc47bc85824f41
```

```bash
用浏览器访问： http://jenkins.kang.com:8080/
默认内置用户admin，其密码为随机字符，可以从文件中查到密码
```

**选择安装推荐的插件**会安装很慢，可以选择不安装，直接点右上角的X直接完成安装过程，后续再用离线方式安装插件

如果显示 jenkins 已离线 ，将**/var/lib/jenkins/hudson.model.UpdateCenter.xml**文件中的更新检查地址改成国内镜像地址,如清华大学地址，然后重启 jenkins 即可：

```
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
https://mirrors.aliyun.com/jenkins/updates/update-center.json
https://jenkins-zh.gitee.io/update-center-mirror/tsinghua/update-center.json
```

**范例： 解决离线问题**

```xml
[root@ubuntu1804 ~]#vim /var/lib/jenkins/hudson.model.UpdateCenter.xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
 <site>
   <id>default</id>
#修改此行为下面行 <url>https://updates.jenkins.io/update-center.json</url>
   <url>https://jenkins-zh.gitee.io/update-center-mirror/tsinghua/update-center.json</url>
 </site>
</sites>
```

**插件安装**

选 无 ,不安装任何插件

**创建** **Jenkins** **管理员**

用户信息保存在下面目录

```bash
[root@ubuntu2404 ~]#ls /var/lib/jenkins/users/
admin_950267871455663983  users.xml
```

### Jenkins基础配置

登录后需要立即修改密码

用户——Security——修改密码



**安装chinese插件**

系统管理——插件管理——Available plugins——在搜索框输入 chinese

```bash
#插件存放路径
[root@ubuntu2404 ~]#ls /var/lib/jenkins/plugins/
bouncycastle-api      instance-identity      javax-activation-api      javax-mail-api      localization-support      localization-zh-cn
bouncycastle-api.jpi  instance-identity.jpi  javax-activation-api.jpi  javax-mail-api.jpi  localization-support.jpi  localization-zh-cn.jpi
```



**修改jenkins以root账号运行**

```bash
[root@ubuntu2404 ~]#vim /lib/systemd/system/jenkins.service
#修改下面两行为root,或注释下面两行
User=jenkins
Group=jenkins
```



**jenkins命令行**

```bash
[root@ubuntu2404 ~]#java -jar jenkins-cli.jar -s http://admin:123456@jenkins.kang.com:8080/ help
  add-job-to-view
    Adds jobs to view.
  build
    Builds a job, and optionally waits until its completion.
  cancel-quiet-down
    Cancel the effect of the "quiet-down" command.
  clear-queue
    Clears the build queue.
  connect-node
    Reconnect to a node(s)
  console
    Retrieves console output of a build.
  copy-job
    Copies a job.
  create-job
    Creates a new job by reading stdin as a configuration XML file.
  create-node
    Creates a new node by reading stdin as a XML configuration.
  create-view
    Creates a new view by reading stdin as a XML configuration.
  delete-builds
    Deletes build record(s).
  delete-job
    Deletes job(s).
  delete-node
    Deletes node(s)
  delete-view
    Deletes view(s).
  disable-job
    Disables a job.
  disable-plugin
    Disable one or more installed plugins.
  disconnect-node
    Disconnects from a node.
  enable-job
    Enables a job.
  enable-plugin
    Enables one or more installed plugins transitively.
  get-job
    Dumps the job definition XML to stdout.
  get-node
    Dumps the node definition XML to stdout.
  get-view
    Dumps the view definition XML to stdout.
  groovy
    Executes the specified Groovy script. 
  groovysh
    Runs an interactive groovy shell.
  help
    Lists all the available commands or a detailed description of single command.
  install-plugin
    Installs a plugin either from a file, an URL, or from update center. 
  keep-build
    Mark the build to keep the build forever.
  list-changes
    Dumps the changelog for the specified build(s).
  list-jobs
    Lists all jobs in a specific view or item group.
  list-plugins
    Outputs a list of installed plugins.
  offline-node
    Stop using a node for performing builds temporarily, until the next "online-node" command.
  online-node
    Resume using a node for performing builds, to cancel out the earlier "offline-node" command.
  quiet-down
    Quiet down Jenkins, in preparation for a restart. Don’t start any builds.
  reload-configuration
    Discard all the loaded data in memory and reload everything from file system. Useful when you modified config files directly on disk.
  reload-job
    Reload job(s)
  remove-job-from-view
    Removes jobs from view.
  restart
    Restart Jenkins.
  safe-restart
    Safe Restart Jenkins. Don’t start any builds.
  safe-shutdown
    Puts Jenkins into the quiet mode, wait for existing builds to be completed, and then shut down Jenkins.
  session-id
    Outputs the session ID, which changes every time Jenkins restarts.
  set-build-description
    Sets the description of a build.
  set-build-display-name
    Sets the displayName of a build.
  shutdown
    Immediately shuts down Jenkins server.
  stop-builds
    Stop all running builds for job(s)
  update-job
    Updates the job definition XML from stdin. The opposite of the get-job command.
  update-node
    Updates the node definition XML from stdin. The opposite of the get-node command.
  update-view
    Updates the view definition XML from stdin. The opposite of the get-view command.
  version
    Outputs the current version.
  wait-node-offline
    Wait for a node to become offline.
  wait-node-online
    Wait for a node to become online.
  who-am-i
    Reports your credential and permissions.
```

### 找回忘记的密码

**方法一：**

刚开始安装 Jenkins，没有修改过密码

直接在initialAdminPassword文件中查看密码即可

```
cat /var/lib/jenkins/secrets/initialAdminPassword
c72aae9f6418442b979b10de4c8d2759
```

**方法二：**

```bash
#停止服务
systemctl stop jenkins

#删除jenkins主目录中config.xml的如下内容
vim /var/lib/jenkins/config.xml
.......
################################################################################
##
 <useSecurity>true</useSecurity> 
 <authorizationStrategy 
class="hudson.security.FullControlOnceLoggedInAuthorizationStrategy"> 
   <denyAnonymousReadAccess>true</denyAnonymousReadAccess> 
 </authorizationStrategy> 
 <securityRealm class="hudson.security.HudsonPrivateSecurityRealm"> 
   <disableSignup>true</disableSignup> 
   <enableCaptcha>false</enableCaptcha> 
 </securityRealm> 
################################################################################

.......

#重启jenkins
systemctl start jenkins  

#重新无需验证即可登录，修改安全配置为Jenkins's own user database(Jenkins专有用户数据库),保存后
Dashboard ---系统管理 --- 全局安全配置

系统管理”,发现此时出现“管理用户”
点admin的配置，重新输入两次新密码

系统管理--- 全局安全配置 --- 授权策略
将任何用户可以做任何事(没有任何限制) 修改为登录用户可以做任何事
```



## Jenkins实现CICD

**Jenkins 实现传统的 CICD 流程**

![image-20250222144821152](5day-png/32Jenkins 实现传统的 CICD 流程.png)

**Docker 环境的CICD**

![image-20250222144912881](5day-png/32Docker 环境的CICD.png)

**Kubernetes 环境的 CICD**

![image-20250222144951195](5day-png/32Kubernetes 环境的 CICD.png)

**Jenkins架构**

![image-20250222145238457](5day-png/32Jenkins 架构.png)

Jenkins根据业务场景的不同,提供了多种风格的任务，默认是自由风格任务，通过安装插件，还可以支持其它风格的插件

Job的风格分类

- 自由风格freestyle：支持实现各种开发语言的不同场景的风格，以Shell为主要技术，内部有各种灵活的配置属性，默认只有此类型
- 流水线 pipeline：重点掌握的风格，使用专用语法
- Maven 项目：仅适用于 JAVA 项目



**jenkins工作目录**

```bash
[root@ubuntu2404 ~]#ls /var/lib/jenkins/workspace
freestyle-demo1  freestyle-wheel  freestyle-wheel@tmp
```



范例1：幸运大装盘

1 打通jenkins用户的主机和后端服务器主机的root用户之间的key验证

2 在jenkins上安装gitlab插件，并添加http验证或ssh验证

![image-20250222160755763](5day-png/32大转盘1.png)

3 在jenkins上新建项目，在Build Steps执行shell脚本

```bash
bash -x /data/jenkins/scripts/freestyle-wheel.sh
```

```
[root@ubuntu2404 scripts]#cat freestyle-wheel.sh
#!/bin/bash
#
HOST_LIST="
10.0.0.100
10.0.0.101
"
for i in $HOST_LIST;do
    scp -r * root@$i:/var/www/html/
done
```



### 参数化构建

jenkins支持参数化构建，类似于脚本中的参数，可以实现灵活的构建任务

Jenkins 支持多种参数类型,比如:Boolean,Choice选项,字符串,Multi_line字符串,文件类型等

**参数类型说明**

参数化构建的目标在于为流水线提供基于参数值的灵活构建机制，从而让一个流水线的定义可以适用于多种需求情形

- 其功能与引用方式与环境变量类似
- 在触发作业运行之时，需要向各参数赋值
- 参数在使用时实际上也表现为变量，可以通过变量的调用方式使用参数
- 注意: 参数化功能无需安装插件即可支持

常用的参数类型

- 选项参数 Choice Parameter
- 布尔值参数 Boolean Parameter
- 字符参数 String Parameter
- 文本参数 Multi-line String Parameter
- 凭据参数
- 密码参数
- 文件参数
- 运行时参数

范例1：幸运大装盘撤销功能 参数化构建

1 关闭首次连接询问yes|no的过程

```
[root@ubuntu2404 ~]#vim /etc/ssh/ssh_config
#修改下面行
StrictHostKeyChecking no
```

2 设置参数化构建

![image-20250222165109381](5day-png/32参数化构建.png)

3 部署任务

![image-20250222165352648](5day-png/32Git部署.png)

![image-20250222165437316](5day-png/32执行shell.png)

```bash
bash -x /data/jenkins/scripts/freestyle-wheel-deploy.sh $ACTION
```

```bash
[root@ubuntu2404 scripts]#cat /data/jenkins/scripts/freestyle-wheel-deploy.sh
#!/bin/bash
#
HOST_LIST="
10.0.0.100
10.0.0.101
"
APP=wheel
APP_PATH=/var/www/html
DATA_PATH=/opt
DATE=`date +%F_%H-%M-%S`


deploy () {
    for i in ${HOST_LIST};do
        ssh root@$i "rm -f  ${APP_PATH} && mkdir -pv ${DATA_PATH}/${APP}-${DATE}"
        scp -r * root@$i:${DATA_PATH}/${APP}-${DATE}
        ssh root@$i "ln -sv ${DATA_PATH}/${APP}-${DATE} ${APP_PATH}"
    done
}

rollback() {
    for i in ${HOST_LIST};do
        CURRENT_VERISION=$(ssh root@$i "readlink $APP_PATH") 
        CURRENT_VERISION=$(basename ${CURRENT_VERISION})
        echo ${CURRENT_VERISION}
        PRE_VERSION=$(ssh root@$i "ls -1 ${DATA_PATH} | grep -B1 ${CURRENT_VERISION}|head -n1 ")
        echo $PRE_VERSION
        ssh root@$i "rm -f  ${APP_PATH}&& ln -sv ${DATA_PATH}/${PRE_VERSION} ${APP_PATH}"
    done
}


case $1 in 
deploy)
    deploy
    ;;
rollback)
    rollback
    ;;
*)
    exit
    ;;
esac
```



### 实现Java应用源码编译并部署

**jar包部署**

java 程序需要使用构建工具,如: maven,ant,gradle等进行构建打包才能部署,其中maven比较流行

以下以 maven 为例实现 Java 应用部署

![image-20250222170639046](5day-png/32Java应用源码编译并部署.png)

1 后端web服务器安装java

```bash
#安装java
[root@ubuntu2204 ~]#apt install openjdk-17-jre
```

2 在jenkins中部署

![image-20250222171830734](5day-png/32spring-boot-helloworld.png)

![image-20250222171904279](5day-png/32spring-boot-helloworld2.png)

```bash
[root@ubuntu2404 scripts]#cat /data/jenkins/scripts/spring-boot-helloworld.sh 
#!/bin/bash
#
#提前在目标服务器上手动创建下面目录
APP=spring-boot-helloworld
APP_PATH=/data/${APP}

HOST_LIST="
10.0.0.100
10.0.0.101
"
PORT=8888

mvn clean package -Dmaven.test.skip=true

for host in $HOST_LIST;do
    ssh root@$host "[ -e $APP_PATH ] || mkdir -p $APP_PATH"
    ssh root@$host killall -9 java &> /dev/null
    scp target/${APP}-*-SNAPSHOT.jar  root@$host:${APP_PATH}/${APP}.jar
    #ssh root@$host "java -jar ${APP_PATH}/${APP}.jar --server.port=8888 &"
    ssh root@$host "nohup java -jar  ${APP_PATH}/${APP}.jar --server.port=$PORT  &>/dev/null & "&
done
```

**war包部署**

```bash
#两台Ubuntu系统，分别在上面安装Tomcat9
#安装方法1
#如果先安装JAVA，后安装Tomcat9，会导致tomcat无法启动，原因找不到JAVA，解决方法
[root@ubuntu2204 ~]#apt update && apt -y install openjdk-17-jre
[root@ubuntu2204 ~]#apt update && apt -y install tomcat9 
[root@ubuntu2204 ~]#vim /lib/systemd/system/tomcat9.service
[Service]
# Configuration 加下面
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/"
[root@ubuntu2204 ~]#systemctl daemon-reload
[root@ubuntu2204 ~]#systemctl start tomcat9
```

```xml
[root@jenkins ~]#apt -y install maven

#如果项目本身没有加速，需要在maven的配置实现镜像加速
[root@jenkins ~]#vim /etc/maven/settings.xml
 <mirror>
   <id>nexus-aliyun</id>
   <mirrorOf>*</mirrorOf>
   <name>Nexus aliyun</name>
   <url>http://maven.aliyun.com/nexus/content/groups/public</url>
 </mirror>
</mirrors>
```

**web服务为tomcat服务器,目录结构规划如下**

```bash
/var/lib/tomcat9/webapps #tomcat工作目录,存放上面目录中子目录的软链接
```

脚本

```bash
[root@ubuntu2404 scripts]#cat /data/jenkins/scripts/hello-world-war.sh
#!/bin/bash
#
APP=hello-world-war
APP_PATH=/var/lib/tomcat9/webapps

HOST_LIST="
10.0.0.100
10.0.0.101
"

mvn clean package -Dmaven.test.skip=true

for host in $HOST_LIST;do
    ssh root@$host systemctl stop tomcat9
    scp target/${APP}-*.war  root@$host:${APP_PATH}/hello.war
    ssh root@$host systemctl start tomcat9
done
```

![image-20250222195048362](5day-png/23java-war包部署1.png)

![image-20250222195135306](5day-png/23java-war包部署2.png)

![image-20250222195154232](5day-png/23java-war包部署3.png)

### 利用Git Parameter插件实现拉取指定版本

1 创建新项目freestyle-spring-boot-git-parameter

2 配置

![image-20250222192127822](5day-png/32git parameter插件部署.png)

![image-20250222192236583](5day-png/32git parameter插件部署1.png)

![image-20250222192258428](5day-png/32git parameter插件部署2.png)

3 执行部署

![image-20250222192348918](5day-png/32git parameter插件部署3.png)

​     

### **实现Golang应用源码编译并部署** 

**在Jenkins安装Golang环境**

```bash
[root@ubuntu2404 ~]#apt update && apt -y install golang
[root@ubuntu2404 ~]#go version
go version go1.22.2 linux/amd64
```

**准备Golang源代码和数据库环境**

```bash
#数据库环境
#安装 MySQL和Redis,按如下配置用户和密码
[root@gitlab ginweb]#cat conf/ginweb.ini
[mysql]
host = "127.0.0.1"
port = 3306
databases = "ginweb"
user = "ginweb"
passwd = "123456"
[redis]
host = "127.0.0.1"
port = 6379
passwd = "123456"
[root@ubuntu2404 ~]#apt update && apt -y install mysql-server redis

#修改MySQL配置
#方法一
[root@ubuntu2404 ~]#sed -i '/127.0.0.1/s/^/#/' /etc/mysql/mysql.conf.d/mysqld.cnf
#方法二
[root@ubuntu2404 ~]#vim /etc/mysql/mysql.conf.d/mysqld.cnf
#bind-address = 127.0.0.1
#mysqlx-bind-address = 127.0.0.1

[root@ubuntu2404 ~]#systemctl restart mysql.service

#配置MySQL环境
[root@ubuntu2404 ~]#mysql
mysql> create database ginweb;
mysql> create user ginweb@'10.0.0.%' identified by '123456';
mysql> grant all on ginweb.* to ginweb@'10.0.0.%';
#导入表结构
[root@ubuntu2404 ginweb]#mysql -uginweb -p123456 -h10.0.0.205 ginweb < ginweb.sql
mysql: [Warning] Using a password on the command line interface can be insecure.

#准备redis
#方法1：非交互
[root@ubuntu2404 ~]#sed -i -e '/^bind/c bind 0.0.0.0' -e '$a requirepass 123456' /etc/redis/redis.conf
[root@ubuntu2404 ~]#systemctl restart redis
#方法2：交互式
[root@ubuntu2204 ~]#vim /etc/redis/redis.conf
bind 0.0.0.0
requirepass 123456
[root@ubuntu2204 ~]#systemctl restart redis
```

准备脚本

```bash
[root@ubuntu2404 scripts]#cat /data/jenkins/scripts/freestyle-ginweb.sh
#!/bin/bash
#
APP=ginweb
APP_PATH=/opt
DATE=`date +%F_%H-%M-%S`
HOST_LIST="
10.0.0.100
10.0.0.101
"

build () {
    #go env 可以查看到下面变量信息，如下环境变量不支持相对路径，只支持绝对路径
    #root用户运行脚本
    #export GOCACHE="/root/.cache/go-build"
    #export GOPATH="/root/go"
    #Jenkins用户运行脚本
    export GOCACHE="/var/lib/jenkins/.cache/go-build"
    export GOPATH="/var/lib/jenkins/go"
    #go env -w GOPROXY=https://goproxy.cn,direct
    export GOPROXY="https://goproxy.cn,direct"
    CGO_ENABLED=0 go build -o ${APP}
}

deloy () {
    for host in $HOST_LIST;do
        ssh root@$host "mkdir -p $APP_PATH/${APP}-${DATE}"
        scp -r *  root@$host:$APP_PATH/${APP}-${DATE}/
        ssh root@$host "killall -0 ${APP} &> /dev/null  && killall -9 ${APP}; rm -f ${APP_PATH}/${APP} && \
                   ln -s ${APP_PATH}/${APP}-${DATE} ${APP_PATH}/${APP}; \
                   cd ${APP_PATH}/${APP}/ && nohup ./${APP}&>/dev/null" &
    done
}

build

deloy
```

![image-20250222204500140](5day-png/32golang编译部署1.png)

![image-20250222204532411](5day-png/32golang编译部署2.png)

![image-20250222205031652](5day-png/32golang编译部署3.png)

### 集成Ansible的任务构建

![image-20250223131118694](5day-png/32集成Ansible的任务构建.png)

```bash 
#在jenkins上安装 Ansible 环境
[root@ubuntu2404 html]#apt update && apt -y install ansible

#打通key验证
Jenkins账号到远程主机的root用户的key验证
jenkins@ubuntu2404:~$ ssh-keygen
jenkins@ubuntu2404:~$ ssh-copy-id 10.0.0.100
jenkins@ubuntu2404:~$ ssh-copy-id 10.0.0.101

#Ubuntu24.04默认没有配置文件，可以手动创建一个空文件，使用默认值即可
[root@ubuntu2404 ~]#ansible --version
ansible [core 2.16.3]
  config file = None	#缺少配置文件
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.3 (main, Feb  4 2025, 14:48:35) [GCC 13.3.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
  
[root@ubuntu2404 ~]#mkdir -p /etc/ansible/ && touch /etc/ansible/ansible.cfg
[root@ubuntu2404 ~]#ansible --version
ansible [core 2.16.3]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.3 (main, Feb  4 2025, 14:48:35) [GCC 13.3.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

编写hosts文件并测试

```bash
[root@ubuntu2404 ~]#cat /etc/ansible/hosts 
[webservers]
10.0.0.100 ansible_ssh_user=root

[appservers]
10.0.0.101 ansible_ssh_user=root
```

```bash
jenkins@ubuntu2404:~$ ansible all -m ping
10.0.0.101 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
10.0.0.100 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

**在jenkins中安装ansible插件**

**Invoke Ansible Ad-Hoc Command**

![image-20250223155643883](5day-png/32Invoke Ansible Ad-Hoc Command.png)



**Invoke Ansible Playbook**

![image-20250223162454799](5day-png/32Invoke Ansible Playbook.png)

```bash
[root@ubuntu2404 ansible]#cat /data/ansible/test.yaml 
- hosts: webservers
  remote_user: root

  tasks:
    - name: Execute command to get host IP
      shell: hostname -I
      register: result  # 修正：register 的值必须是合法的变量名

    - name: Show result
      debug:
        msg: "{{ result.stdout }}"  # 修正：显示命令的输出内容

    - name: Create new file
      file:
        path: /tmp/test.log
        state: touch  # 修正：state 参数应为 touch，而不是 status
```

**使用Ansible Playbook基于参数化实现任务测试和生产多套不同环境的部署**

```bash
#创建两个hosts文件代表不同的环境
[root@ubuntu2404 ansible]#cat /etc/ansible/hosts-product 
[webservers]
10.0.0.100 ansible_ssh_user=root
[root@ubuntu2404 ansible]#cat /etc/ansible/hosts-test 
[webservers]
10.0.0.101 ansible_ssh_user=root
```

![image-20250223163240726](5day-png/32Ansible Playbook基于参数化实现任务测试和生产多套不同环境的部署.png)

![image-20250223163313114](5day-png/32Ansible Playbook基于参数化实现任务测试和生产多套不同环境的部署1.png)

![image-20250223163342876](5day-png/32Ansible Playbook基于参数化实现任务测试和生产多套不同环境的部署2.png)

```bash
[root@ubuntu2404]#cat /data/ansible/test.yaml 
- hosts: webservers
  remote_user: root

  tasks:
    - name: Execute command to get host IP
      shell: hostname -I
      register: result  # 修正：register 的值必须是合法的变量名

    - name: Show result
      debug:
        msg: "{{ result.stdout }}"  # 修正：显示命令的输出内容

    - name: Create new file
      file:
        path: /tmp/test.log
        state: touch  # 修正：state 参数应为 touch，而不是 status
```



**使用Ansible Playbook实现向Playbook中传参功能**

**编写Playbook文件**

```
[root@ubuntu2404 ansible]#cat /data/ansible/test.yaml
- hosts: "{{ ansible_hosts }}"
  remote_user: root

  tasks:
    - name: Execute command to get host IP
      shell: hostname -I
      register: result  # 修正：register 的值必须是合法的变量名

    - name: Show result
      debug:
        msg: "{{ result.stdout }}"  # 修正：显示命令的输出内容

    - name: Create new file
      file:
        path: /tmp/test.log
        state: touch  # 修正：state 参数应为 touch，而不是 status
```

**创建主机清单文件**

```bash
[root@ubuntu2404 ansible]#cat /etc/ansible/hosts-product 
[webservers]
10.0.0.100 ansible_ssh_user=root

[appservers]
10.0.0.101 ansible_ssh_user=root
[root@ubuntu2404 ansible]#cat /etc/ansible/hosts-test 
[webservers]
10.0.0.101 ansible_ssh_user=root

[appservers]
10.0.0.100 ansible_ssh_user=root
```

![image-20250223164740834](5day-png/32Ansible Playbook实现向Playbook中传参功能1.png)

![image-20250223164806029](5day-png/32Ansible Playbook实现向Playbook中传参功能2.png)

![image-20250223164831537](5day-png/32Ansible Playbook实现向Playbook中传参功能3.png)

高级——》

![image-20250223164924278](5day-png/32Ansible Playbook实现向Playbook中传参功能4.png)

```
Value
$hosts_list
```





### 构建后通知

#### 邮件通知

Mailer 和 Email Extension 插件都可以实现邮件通知功能

**配置发送邮件的邮件通知**

Jenkins—系统管理—系统设置

注意:必须安装插件才能出现下面的SMTP配置

配置邮件通知信息如下:

- 用户名必须要和上面的系统管理员邮件地址相同
- 用户默认邮件后缀，可为空
- 启用"使用SSL协议"
- SMTP 端口可以为空,默认为465，QQ邮箱必须加密，163可以不加密使用25端口
- Reply-To Address 可以为空

![image-20250223171206507](5day-png/32邮件通知.png)

![image-20250223171249772](5day-png/32邮件通知2.png)



![image-20250223172112965](5day-png/32邮件通知3.png)



#### 钉钉

创建群组——添加机器人——生成码

![image-20250223172705022](5day-png/32钉钉1.png)

![image-20250223172715312](5day-png/32钉钉2.png)

**Jenkins** **安装** **DingTalk** **插件**

**安装完插后，建议重启，否则可以通知失败**

```bash
#提示失败，可以重启jenkins解决
[钉钉插件]发送消息时报错: java.lang.NullPointerException: Cannot invoke "io.jenkins.plugins.DingTalkUserProperty.getMobile()" because the return value of "hudson.model.User.getProperty(java.lang.Class)" is nullFinished: SUCCESS
```

![image-20250223172842026](5day-png/32钉钉3.png)

Manage Jenkins --> 钉钉

![image-20250223172916567](5day-png/32钉钉4.png)

![image-20250223173611486](5day-png/32钉钉5.png)

![image-20250223173633921](5day-png/32钉钉6.png)

**配置任务实现钉钉通知**

![image-20250223173813738](5day-png/32钉钉7.png)

![image-20250223173857977](5day-png/32钉钉8.png)

手机号支持多个,每个手机号一行,也可选 atall 即所有群里的人员(不必再输入手机号，如果输入手机号，仍然会@相关人员)

自定义内容需要使用Markdown格式，比如:

```bash
- 构建ID: ${BUILD_ID}
- 部署项目: ${JOB_NAME}
- 项目URL: ${JOB_DISPLAY_URL}
- 部署目录: ${WORKSPACE}
```

![image-20250223174052657](5day-png/32钉钉9.png)

![image-20250223174104271](5day-png/32钉钉10.png)

#### **微信通知**

**注册企业微信添加WebHook机器**

![image-20250223174236672](5day-png/32微信通知1.png)

**Jenkins** **安装** **Qy Wechat Notification** **插件**

**添加构建后操作**

![image-20250223174334432](5day-png/32微信通知2.png)

通知UserID

```
ALL 表示所有人
hezhaokang,zhangsan #用户名大小写不敏感
```

更多消息定制消息格式

```
- 构建ID: ${BUILD_ID}
- 部署项目: ${JOB_NAME}
- 部署目录: ${WORKSPACE}
```

![image-20250223174636202](5day-png/32微信通知3.png)



### 自动化构建

两种自动化构建方式

- 周期性定时构建
- Webhook 触发构建

#### 定时和SCM构建

![image-20250223180614849](5day-png/32定时和SCM构建.png)

- 定时构建: 按时间周期性的触发构建

- 轮询SCM(Source Code Management): 

  指的是定期到代码仓库检查代码是否有变更，存在代码变更时就运行pipeline;为了能够从CI中得到更多的收益，轮询操作越频繁越好;显然，这会给SCM带去无谓的压力,所以构建的触发由SCM负责通知Jenkins最为理想;但在外部的SCM无法通知到局域网中的Jenkins时，可以采轮询SCM方式倒也不失为一种选择

  首次任务会进行构建，后续查看SCM代码是否变化，来决定是否构建



Jenkins cron语法遵循Unix cron语法的定义,但在细节上略有差别

一项cron的定义包含由空白字符或Tab分隔的5个字段，用于定义周期性的时间点

H 符号可用于任何字段,且它能够在一个时间范围内对项目名称进行散列值计算出一个唯一的偏移量，以避免所有配置相同cron值的项目在同一时间启动;比如:triggers { cron(H(0,30)) }，表示每小时的前半小时的某一分钟进行构建

```bash
Examples:
# Every fifteen minutes (perhaps at :07, :22, :37, :52):
H/15 * * * *    
# Every ten minutes in the first half of every hour (three times, perhaps at :04, :14, :24):
H(0-29)/10 * * * *
# Once every two hours at 45 minutes past the hour starting at 9:45 AM and finishing at 3:45 PM every weekday:
45 9-16/2 * * 1-5
# Once in every two hour slot between 8 AM and 4 PM every weekday (perhaps at 9:38 AM, 11:38 AM, 1:38 PM, 3:38 PM):
H H(8-15)/2 * * 1-5
# Once a day on the 1st and 15th of every month except December:
H H 1,15 1-11 *
```

范例: 每小时构建

```
H * * * *
```

每3分钟构建一次,如:在2:55,2:58,3:01,3:04时间点进行构建

```
H/3 * * * *
```

范例: 每分钟执行SCM 构建

```
* * * * *
```

**注意：SCM任务会在左侧多出一个“Git轮询日志，可以看到轮询的记录信息**

观察Git 轮询日志可以看到当有变化时才会构建,否则不会执行构建

![image-20250223175959422](5day-png/32定时构建.png)

![image-20250223180403749](5day-png/32SCM构建.png)

#### **构建Webhook触发器**

构建触发器(webhook)，也称为钩子，实际上是一个HTTP回调，其用于在开发人员向gitlab提交代码后能够触发jenkins自动执行代码构建操作。

![image-20250223180639840](5day-png/32Webhook 触发器.png)

常见场景: 

只有在开发人员向develop分支提交代码的时候会自动触发代码构建和部署至测试环境，而向主分支提交的代码不会自动构建，需要运维人员手动部署代码到生产环境。

**可以使用多种方式实现 Webhook 触发构建**

- 触发远程构建: 此方式无需安装插件

- Build when a change is pushed to GitLab. GitLab webhook URL: 需要安装Gitlab插件
- Generic Webhook Trigger : 需要安装 Generic Webhook Trigger Plugin 插件

##### 触发远程构建

![image-20250223181314286](5day-png/32触法远程构建.png)

```
http://jenkins.kang.com:8080/job/freestyle-wheel-deploy/build?token=123456
```

在gitlab服务器上操作

![image-20250223191949825](5day-png/32Webhook 触发器1.png)

添加webhooks

![image-20250223192136845](5day-png/32Webhook 触发器2.png)



![image-20250223192201370](5day-png/32Webhook 触发器3.png)

**注意：新版本不再允许用户的密码方式，curl测试成功，但gitlab测式会提示403错误，只支持Token**

**解决办法**

在jenkins上新建一个账号

![image-20250223192430704](5day-png/32Webhook 触发器4.png)

登录到kang账号，创建一个token

```
1198d1fb13a43e6a3e281c52c8d65642de
```

![image-20250223192645567](5day-png/32Webhook 触发器5.png)



```bash
#在命令行可以触发
[root@ubuntu2404 ~]#curl http://kang:123456@jenkins.kang.com:8080/job/freestyle-wheel-deploy/build?token=123456
```

**在gitlab上面修改webhook**

```
http://kang:1198d1fb13a43e6a3e281c52c8d65642de@jenkins.kang.com:8080/job/freestyle-wheel-deploy/build?token=123456
```

![image-20250223194033365](5day-png/32Webhook 触发器6.png)





```
http://kang:1198d1fb13a43e6a3e281c52c8d65642de@jenkins.kang.com:8080/job/freestyle-wheel/build?token=123456
```

```
http://kang:1198d1fb13a43e6a3e281c52c8d65642de@jenkins.kang.com:8080/job/tag/build?token=123456
```



**第二种插件方式**

jenkins上配置

![image-20250223201827977](5day-png/32Webhook 触发器7.png)

![image-20250223201842873](5day-png/32Webhook 触发器8.png)

```
http://jenkins.kang.com:8080/project/tag
fa6c9119a5765e8119b96a51ee67360e
```

在gitlab上还做上面的步骤，建立账号，设置令牌，开启出站请求

设置Webhooks 添加刚才生成的路径和token

![image-20250223202302983](5day-png/32Webhook 触发器9.png)



### **构建前后多个项目关联自动触发任务执行**

用于多个 Job 相互关联，需要同行执行多个job的场景,比如:如果job1后希望自动构建job2

可以用两种方法实现

- 在前面任务中利用构建后操作关联后续任务
- 在后面任务中利用构建触发器关联前面任务

注意:

上面两种方法,都需要在前面任务执行后才能自动关联执行后续任务

不要实现任务的环路，会导致死循环



1 在先执行的任务中配置构建后操作实现

![image-20250223202942644](5day-png/32任务里配置构建后操作.png)

2 在执行任务中配置其他工程构建后触发

![image-20250223203334374](5day-png/32在执行任务中配置构建前操作.png)



### **实现容器化的Docker任务**

![image-20250223203541427](5day-png/32实现容器化的Docker任务.png)

当前越来越多的组织以容器形式运行应用, 应用交付形式统一为Container Image

交付的Container Image由Registry存储和分发,应用以容器化形式由Docker，Kubernetes进行编排运行

jenkins的多款插件都能实现容器镜像Image构建和推送

- docker-build-step 
- Docker 
- CloudBees Docker Build and Publish
- Docker Pipeline Plugin：这个插件允许在Jenkins Pipeline中使用Docker来构建、发布和管理容器。它提供了一组用于在Pipeline脚本中执行Docker相关操作的步骤。
- Docker Slaves Plugin：这个插件允许Jenkins使用Docker容器作为构建代理（agent）。它可以动态地启动和停止Docker容器来扩展Jenkins的构建能力

**实现自由风格任务实现Docker镜像制作并运行**

![image-20250223203810746](5day-png/32实现自由风格任务实现Docker镜像制作并运行.png)

**在harbor.wang.org主机上安装Harbor**

```
[root@ubuntu2404 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/noble/main/g/gitlab-ce/gitlab-ce_17.8.2-ce.0_amd64.deb

```



**在部署应用的目标主机安装Docker，并且打开远程连接端口，并且信任harbor**

```bash
#在Jenkins主机及应用主机上安装Docker
[root@ubuntu2404 ansible]#apt update && apt -y install docker.io
#方法1:并信任harbor，
[root@jenkins ~]#vim /lib/systemd/system/docker.service
.....
#修改此行
ExecStart=/usr/bin/dockerd -H fd:// -H 127.0.0.1:2375 --
containerd=/run/containerd/containerd.sock --insecure-registry harbor.wang.org
.....
#方法2：信任harbor
[root@jenkins ~]#cat /etc/docker/daemon.json 
{
  "insecure-registries": ["harbor.wang.org"]
}
#需要重启jenkins上面的权限才能生效
[root@ubuntu2204 ~]#systemctl restart jenkins

#默认jenkins用户无法访问docker服务器
#方法1：
#修改jenkins用户权限可以访问docker的socket文件
[root@ubuntu2204 ~]#usermod -aG docker jenkins
[root@ubuntu2204 ~]#id jenkins 
uid=115(jenkins) gid=120(jenkins) 组=120(jenkins),119(docker)
#需要重启jenkins上面的权限才能生效
[root@ubuntu2204 ~]#systemctl restart jenkins

[root@ubuntu2204 ~]#systemctl daemon-reload 
[root@ubuntu2204 ~]#systemctl restart docker.service
#方法2：打开监听端口2375/tcp支持Jenkins用户直接连接,
[root@jenkins ~]#vim /lib/systemd/system/docker.service
.....
#修改此行
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375 ......
```

用jenkins和后端的两台服务器上登录harbor

```bash
[root@ubuntu2404 docker]#su - jenkins 
jenkins@ubuntu2404:~$ docker login -ukang -pHh123456 harbor.kang.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /var/lib/jenkins/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
jenkins@ubuntu2404:~$ 
logout
[root@ubuntu2204 ~]#docker login -ukang -pHh123456 harbor.kang.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

```bash
[root@ubuntu2404 scripts]#cat docker-spring-book.sh
#!/bin/bash
#
REGISTRY=harbor.kang.com
PORT=80

HOSTS="
10.0.0.100
10.0.0.101"

mvn clean package -Dmaven.test.skip=true

docker build -t ${REGISTRY}/example/myapp:v$BUILD_ID  . 
docker push ${REGISTRY}/example/myapp:v$BUILD_ID

for i in $HOSTS;do
    ssh root@$i "docker rm -f myapp ; docker  run -d  -p $PORT:8888 --name myapp --restart always ${REGISTRY}/example/myapp:v$BUILD_ID"
    
done
```



### 实现参数容器化docker任务

```bash
root@ubuntu2404 scripts]#cat docker-spring-book.sh
#!/bin/bash
#
REGISTRY=harbor.kang.com
PORT=80

HOSTS="
10.0.0.100
10.0.0.101"

mvn clean package -Dmaven.test.skip=true

docker build -t ${REGISTRY}/example/myapp:$TAG  . 
docker push ${REGISTRY}/example/myapp:$TAG

for i in $HOSTS;do
    ssh root@$i "docker rm -f myapp ; docker  run -d  -p $PORT:8888 --name myapp --restart always ${REGISTRY}/example/myapp:$TAG"
    
done
```

![image-20250224194237821](5day-png/32参数化容器构建1.png)

![image-20250224194051177](5day-png/32参数化容器构建2.png)

![image-20250224194111688](5day-png/32参数化容器构建3.png)

```bash
#方法二
#开启后端服务器的远程连接
[root@ubuntu2204 ~]#vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
```

修改脚本

```bash
[root@ubuntu2404 scripts]#cat docker-spring-book-docker.sh
#!/bin/bash
#
REGISTRY=harbor.kang.com
PORT=80

HOSTS="
10.0.0.100
10.0.0.101"

mvn clean package -Dmaven.test.skip=true

docker build -t ${REGISTRY}/example/myapp:$TAG  . 
docker push ${REGISTRY}/example/myapp:$TAG

for i in $HOSTS;do
    #ssh root@$i "docker rm -f myapp ; docker  run -d  -p $PORT:8888 --name myapp --restart always ${REGISTRY}/example/myapp:$TAG"
    docker -H $i rm -f myapp
    docker -H $i run -d  -p ${PORT}:8888 --restart always --name myapp ${REGISTRY}/example/myapp:$TAG
done
```



### **基于Docker插件实现自由风格任务实现Docker镜像制作**

**安装插件** **docker-build-step**

在Jenkins安装Docker并配置 Docker 插件

```bash
#本地Docker Engine
unix:///var/run/docker.sock
unix://localhost:2375   #Jenkins-2.244.1 不支持
#远程Docker Engine
tcp://10.0.0.101:2375
```

![image-20250224213142418](5day-png/32基于Docker插件实现自由风格任务实现Docker镜像制作.png)

**在** **Jenkins** **创建连接** **Harbor** **的凭证**

![image-20250224213659661](5day-png/32基于Docker插件实现自由风格任务实现Docker镜像制作1.png)
