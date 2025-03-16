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



## Jenkins分布式说明

<img src="5day-png/32jnkins分布式.png" alt="image-20250225091304641" style="zoom:50%;" />

采用 master/agent 架构，因而其节点可划分主节点(master)和代理节点(agent)两种类型,，代理节点也被称为从节点(slave)

主节点负责提供UI、处理HTTP请求及管理构建环境等，而代理节点则主要负责执行构建任务

- 主节点Master/Controller: 

  Jenkins的一个部署实例的核心控制系统，它能够完全访问所有Jenkins配置的选项和任务（job)列表，而且，若不存在其他代理节点，主节点也是默认的任务执行节点

- 代理节点Slave/Agent:

  在早先版本的Jenkins中，代理节点 (agent)也被称为从节点(slave),它代表着所有的非主节点

  这类节点由主节点管理，按需分配或指定执行特定的任务，例如不同的构建任务或测试

  脚本式流水线中,节点特指一个运行代理节点的系统,而在声明式流水线中,它则是分配的一个作为代理节点的特定节点

- 执行器（Executor):

  简单来说，Executor只是节点或代理节点用于执行任务的一个糟位

Executor的数量定义了该节点可以执行的并发任务量，一个节点上可以有任务数量的糟位，但也允行管理员按节点资源定义合适的数量

在主节点将任务分配给特定节点时，该节点上必须有可用的Executor来立即执行该任务,否则、只能等到有空闲槽位可用

![image-20250225091532986](5day-png/32jenkins分布式2.png)

**Jenkins主从架构图**

![image-20250225091644866](5day-png/32Jenkins主从架构图.png)

### 节点标签Label

Jenkins中的标签(tag)指的是节点上的标识符，而后可由pipeline中的agent指令等进行过滤和选择节点执行

当Agent节点较多时，基于方便管理的目的，通常应该给这些节点添加能够体现其某种特性或功能的标签，以便于在构建任务中能基于标签过滤出符合条件的agent来

一个 Agent 上可添加多个标签,一个标签也可以添加至多个 Agent, 可以在作业中通过标签表达式实现Agent的过滤

标签名称不允许使用空白字符，也不允许使用标签表达式中预留的关键字，例如: !、&、|、<、>、) 和（ 等

常用的标签纬度有如下几个

- 操作系统类型: Linux、Windows、MacOS
- 操作系统位数: 32bit、64bit
- 集成的工具链: jdk、Go、Python、Nodejs等

**标签表达式（label expressions）支持如下操作符**

- !expression：表达式条件取反

- a && b：表达式间“与” 关系

- a || b：表达式间“或” 关系

- a -> b：表示如果满足a表达式，则同时必须满足b表达式,但是如果不满足a,则不要求满足b,等同于“!a || b”

  示例: linux -> x64，意味着，如果操作系统为linux，则它也必须是x64的系统环境，如果不是linux，则无要求必须是x64

- a<->b：表示两个条件要么同时满足，要么同时都不满足，即等同于 “a && b || !a && !b”

- (expression)：表达式分组，常在需要改变操作符间的优先级顺序时使用

**Jenkins Master与Agent之间的通信方式**

<img src="5day-png/32静态agent.png" alt="image-20250225091920946" style="zoom:50%;" />

- Launch agent via SSH

  SSH连接, Agent端是SSH Server端

  **此方式需要安装SSH Build Agents插件**

  方式1: 

  在Jenkins Agent节点运行ssh服务,接收Master的远程连接

  在Controller端保存认证信息为Credential,可以口令认证和密钥认证

  运行者身份：普通用户jenkins，/home/jenkins/agent目录，作为Agent端的工作目录

  Controller ssh client --> Agent ssh server 

  方式2: 

  通过基于 jenkins/ssh-agent 镜像的容器运行

  此方式只支持密钥认证

  使用ssh-keygen生成一对密钥，并将公钥通过环境变量传递给 ssh-agent容器

  将私钥保存为 Jenkins上的凭据

- Launch agent by connecting it to the controller

  注意：此方式中文翻译为**通过** **Java Web** **启动代理**

  基于JNLP-HTTP 协议连接器实现

  在agent上以手动或系统服务的方式经由JNLP协议触发双向连接的建立

  要求：Controller端额外提供一个套接字以接收连接请求，默认使用tcp协议的50000端口，也支持使用随机端口（安全，可能会对服务端在防火墙开放该端口造成困扰），也可以使用websocket，基于默认8080端口建立集群通信连接

  Controller jnlp server <-- Agent jnlp client 

- Launch agent via execution of command on the controller

  在Controller上远程运行命令启动Agent

  在Master 上以远程运行命令的方式启动Agent,需要ssh服务

### **Agent** **分类**

Agent 可以分为静态和动态两种

- 静态Agent：

  固定的持续运行的Agent,即使没有任务,也需要启动Agent 

  以daemon形式运行的Jenkins

  每个Agent可以存在多个Executor，具体的数量应该根据Agent所在主机的系统资源来设定

  (1) Linux Jenkins (2) Windows Jenkins (3) Jenkins Container 方式

  注意：很多的构建步骤，有可能会通过运行shell命令进行，则必须要确保在Container内部有所调用的可用shell命令

- 动态Agent：

  按需动态创建和删除 Agent ,当无任务执行时,删除Agent

  可以基于Docker 和 Kubernetes 实现

  - Docker Plugin 

    在基于配置好的Docker Host上，按需要创建以容器方式运行的 Agent

    需要事先配置好容器模板

  - Kubernetes Plugin 

    基于配置好的Kubernetes集群环境，按需要创建以Pod方式运行Agent，需要事先配置Pod模板

    由Controller按Job的运行需要临时创建Agent，Agent数量可以动态伸缩, 且Job运行结束后会删除Agent

    可以把每个Agent视作一个动态的Executor

    依赖的环境：云，支持由Jenkins Controller通过API调用

    而 Jenkins 自身既可以部署在k8s上，也完全可以运行在k8s外



### 以Docker容器方式运行Agent

<img src="5day-png/32容器方式实现jenkins.png" alt="image-20250225095827157" style="zoom:50%;" />

### **基于** **SSH** **协议实现** **Jenkins** **分布式**

**Slave** **节点安装** **Java** **等环境确保和** **Master** **环境一致**

Slave 节点通过从Master节点自动下载的基于 JAVA 的 **remoting.jar** 程序包实现,所以需要安装JDK

Slave服务器需要创建与Master相同的数据目录，因为脚本中调用的路径只有相对于Master的一个路径，此路径在master与各node节点应该保持一致。任务中执行的脚本存放的路径和master也必须一致.

如果Slave需要执行编译或执行特定的job，则也需要配置Java或其它语言环境,安装 git、maven、go、ansible等与master相同的基础运行环境

**注意：Jenkins Agent 和Master的环境尽可能一致，包括软件的版本，路径，脚本，ssh key验证,域名解析等**

```bash
#在两个slave主机上执行下面操作，安装和Master节点相同版本的jdk
[root@ubuntu2404 ~]#apt update && apt install openjdk-17-jdk -y
```

**Master** **节点安装插件**

安装 **SSH Build Agents** 插件，实现 ssh 连接代理

**Master节点上配置连接的slave**

![image-20250225102012661](5day-png/32基于SSH协议实现Jenkins分布式.png)

**添加** **Master** **访问** **Slave** **认证凭据**

用于 Master 连接 Slave 节点的凭据

可以是用户密码的凭据,也可以配置Master节点到Slave节点SSH key 验证

以root 身份连接 Agent 

如果已经实现ssh key 验证，下面可以不配置

![image-20250225102102136](5day-png/32基于SSH协议实现Jenkins分布式1.png)

这是在agent服务器开启了一个java程序

```bash
[root@ubuntu2404 ~]#ss -ntp
State            Recv-Q            Send-Q                             Local Address:Port                              Peer Address:Port            Process                                  
ESTAB            0                 52                           [::ffff:10.0.0.201]:22                           [::ffff:10.0.0.1]:56887            users:(("sshd",pid=983,fd=4))           
ESTAB            0                 0                            [::ffff:10.0.0.201]:22                         [::ffff:10.0.0.204]:48790            users:(("sshd",pid=3455,fd=4))          
[root@ubuntu2404 ~]#ls /var/lib/jenkins/
remoting  remoting.jar
[root@ubuntu2404 ~]#ps aux | grep java
root        3478  3.4  5.5 3104980 109412 ?      Ssl  10:17   0:12 java -jar remoting.jar -workDir /var/lib/jenkins -jar-cache /var/lib/jenkins/remoting/jarCache
```

范例：

**1 ssh连接**

**ssh连接是输入yes和no的解决办法**

```bash
#方法一
[root@ubuntu2404 ~]#vim /etc/ssh/ssh_config
StrictHostKeyChecking no
```

方法二

![image-20250225103540310](5day-png/32基于SSH协议实现Jenkins分布式2.png)

**2 配置域名解析** **DNS**

```bash
[root@ubuntu2404 ~]#host gitlab.kang.com
gitlab.kang.com has address 10.0.0.205
```

**3 把在master上的脚本移动到agent上 并安装maven**

maven版本和master上的版本最好一样

```bash
[root@ubuntu2404 scripts]#scp -r /data/ 10.0.0.201:/
[root@ubuntu2404 ~]#apt install maven -y
```

**4 打通agent的root用户到远程服务器的root用户的key验证**

```bash
[root@ubuntu2404 ~]#ssh-keygen
[root@ubuntu2404 ~]#ssh-copy-id root@10.0.0.100
[root@ubuntu2404 ~]#ssh-copy-id root@10.0.0.101
```

![image-20250225115032719](5day-png/32基于SSH协议实现Jenkins分布式3.png)

![image-20250225115047121](5day-png/32基于SSH协议实现Jenkins分布式4.png)

![image-20250225115101356](5day-png/32基于SSH协议实现Jenkins分布式5.png)

```bash
bash -x /data/jenkins/scripts/spring-boot-helloworld.sh
```

```bash
[root@ubuntu2404 ~]#cat /data/jenkins/scripts/spring-boot-helloworld.sh
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



### **基于JNLP协议的Java Web启动代理**

**Launch agent by connecting it to the controller** **也称为 通过** **Java Web** **启动代理**

此方式无需安装插件，即可实现

**在Jenkins Master节点实现全局安全配置**

使用随机端口或者固定端口都可以



![image-20250225120354425](5day-png/32基于JNLP协议的Java Web启动代理1.png)

![image-20250225120629928](5day-png/32基于JNLP协议的Java Web启动代理2.png)

**代理 jenkins-agent2-jnlp**

**Run from agent command line: (Unix)** 

```
curl -sO http://jenkins.kang.com:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://jenkins.kang.com:8080/ -secret f7290b133482b54a7888c8e6e24c3ee4cca69c70d992d3fe1fa8544c31d74fd3 -name "jenkins-agent2-jnlp" -webSocket -workDir "/var/lib/jenkins"
```

**Run from agent command line: (Windows)** 

```
curl.exe -sO http://jenkins.kang.com:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://jenkins.kang.com:8080/ -secret f7290b133482b54a7888c8e6e24c3ee4cca69c70d992d3fe1fa8544c31d74fd3 -name "jenkins-agent2-jnlp" -webSocket -workDir "/var/lib/jenkins"
```

**Or run from agent command line, with the secret stored in a file: (Unix)** 

```
echo f7290b133482b54a7888c8e6e24c3ee4cca69c70d992d3fe1fa8544c31d74fd3 > secret-file
curl -sO http://jenkins.kang.com:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://jenkins.kang.com:8080/ -secret @secret-file -name "jenkins-agent2-jnlp" -webSocket -workDir "/var/lib/jenkins"
```

**Or run from agent command line, with the secret stored in a file: (Windows)** 

```
echo f7290b133482b54a7888c8e6e24c3ee4cca69c70d992d3fe1fa8544c31d74fd3> secret-file
curl.exe -sO http://jenkins.kang.com:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://jenkins.kang.com:8080/ -secret @secret-file -name "jenkins-agent2-jnlp" -webSocket -workDir "/var/lib/jenkins"
```

If you prefer to use TCP instead of WebSockets, remove the `-webSocket` option. Run `java -jar agent.jar -help` for more.

Note: PowerShell users must use curl.exe instead of curl because curl is a default PowerShell cmdlet alias for Invoke-WebRequest.



```bash
#在agint上执行
[root@ubuntu2404 ~]#curl -sO http://jenkins.kang.com:8080/jnlpJars/agent.jar
[root@ubuntu2404 ~]#ls
agent.jar
[root@ubuntu2404 ~]#java -jar agent.jar -url http://jenkins.kang.com:8080/ -secret f7290b133482b54a7888c8e6e24c3ee4cca69c70d992d3fe1fa8544c31d74fd3 -name "jenkins-agent2-jnlp" -webSocket -workDir "/var/lib/jenkins"
Feb 25, 2025 12:08:54 PM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /var/lib/jenkins/remoting as a remoting work directory
Feb 25, 2025 12:08:54 PM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /var/lib/jenkins/remoting
Feb 25, 2025 12:08:54 PM hudson.remoting.Launcher createEngine
INFO: Setting up agent: jenkins-agent2-jnlp
Feb 25, 2025 12:08:54 PM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 3283.v92c105e0f819
Feb 25, 2025 12:08:54 PM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /var/lib/jenkins/remoting as a remoting work directory
Feb 25, 2025 12:08:55 PM hudson.remoting.Launcher$CuiListener status
INFO: WebSocket connection open
Feb 25, 2025 12:08:55 PM hudson.remoting.Launcher$CuiListener status
INFO: Connected
```

![image-20250225125426319](5day-png/32基于JNLP协议的Java Web启动代理3.png)

![](5day-png/32基于SSH协议实现Jenkins分布式4.png)

![](5day-png/32基于SSH协议实现Jenkins分布式5.png)

```bash
bash -x /data/jenkins/scripts/spring-boot-helloworld.sh
```

```bash
[root@ubuntu2404 ~]#cat /data/jenkins/scripts/spring-boot-helloworld.sh
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



### 基于Docker的动态Agent

**准备** **Docker Engine** **主机**

```bash
#安装docker并开启docker的远程连接 
[root@ubuntu2404 ~]#apt update && apt install docker.io 
[root@ubuntu2404 ~]#vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
[root@ubuntu2404 ~]#systemctl daemon-reload && systemctl restart docker.service
```

**安装** **Docker** **插件**

**创建** **Cloud**

管理 Jenkins -- Clouds --- New Cloud

![image-20250225131617527](5day-png/32基于Docker的动态Agent.png)

**添加** **Docker Agent templates**

```bash
#可以在Docker主机提前拉取镜像
[root@ubuntu2404 ~]#docker pull registry.cn-beijing.aliyuncs.com/wangxiaochun/inbound-agent:alpine-jdk17
[root@ubuntu2404 ~]#docker images
REPOSITORY                                                    TAG            IMAGE ID       CREATED        SIZE
registry.cn-beijing.aliyuncs.com/wangxiaochun/inbound-agent   alpine-jdk17   172c6050fbbb   3 months ago   138MB
[root@ubuntu2404 ~]#docker run --rm registry.cn-beijing.aliyuncs.com/wangxiaochun/inbound-agent:alpine-jdk17 env
LANGUAGE=en_US:en
USER=jenkins
HOSTNAME=4e1764a223f4
SHLVL=1
HOME=/home/jenkins
AGENT_WORKDIR=/home/jenkins/agent
PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
JAVA_HOME=/opt/java/openjdk
PWD=/home/jenkins
TZ=Etc/UTC
```

**使用jnlp协议**

![image-20250225132340975](5day-png/32基于Docker的动态Agent1.png)

![image-20250225132442609](5day-png/32基于Docker的动态Agent2.png)

**Container Setting** **配置名称解析**

JNLP协议需要Docker 主机主动连接 Jenkins主机，所以需要确保Docker Engine 主机可以解析jenkins.kang.com的名称

而SSH协议是 Jenkins 主机主动连接 Docker 主机，所以无需实现名称解析

域名解析实现两种

- 通过宿主机的DNS解析，容器可以继承宿主机的DNS配置
- 可以Container Setting 如下面添加名称解析

**如果基于JNLP协议，必须要实现Jenkins主机的名称解析，如下显示**

![image-20250225132745997](5day-png/32基于Docker的动态Agent3.png)

![image-20250225132818952](5day-png/32基于JNLP协议的Java Web启动代理4.png)

![image-20250225141041699](5day-png/32基于Docker的动态Agent4.png)**使用 SSH Agent**

![image-20250225141953117](5day-png/32基于Docker的动态Agent5.png)

![image-20250225141852451](5day-png/32基于Docker的动态Agent6.png)

![image-20250225141919171](5day-png/32基于Docker的动态Agent7.png)



## Jenkins Pipeline

**流水线和自由风格任务流程比较**

![image-20250225142339352](5day-png/32Pipeline.png)

- 一致性: Pipeline 用统一语法的代码的方式实现各个CICD的阶段的任务，不仅可以被纳入版本控制，还可以通过编辑代码实现目标效果
- 直观性: 构建过程中每一步都可以直接的图形化显示输出,比如每个阶段的执行时间,直观友好,pipeline 帮助我们快速的定位哪个阶段的任务出现错误
- 可持续性：Jenkins的重启或者中断后不影响已经执行的pipeline Job
- 支持暂停：Pipeline可以选择停止并等待人工输入或批准后再继续执行
- 支持回放: 如果失败,可以使用回放,进行临时性的修改 job ,再调试执行,如果成功,再真正修改任务即可
- 可扩展：通过Groovy的编程更容易的扩展插件
- 并行执行：通过Groovy脚本可以实现step，stage间的并行执行，和更复杂的相互依赖关系
- 多功能：支持复杂CD要求，包括fork/join子进程，条件判断，循环和并行执行工作的能力

### **Pipeline** **语法**

当前 Jenkins 2.X 支持两种语法的流水线： 脚本式（命令式）和声明式

- 脚本式Scripted Pipeline语法

  此语法是 Jenkins最先支持pipeline语法，采用命令式风格，直接在流水线脚本中定义逻辑和程序流程

- 声明式Declarative Pipeline语法

  后来CloudBees公司为Jenkins引入的一种“流水线即代码”的pipeline语法

  它允许用户在pipeline的定义中将更多的精力关注于期望pipeline的状态和输出之上，而非实现逻辑

声明式和脚本化的流水线从根本上是不同的。 声明式流水线的是 Jenkins 流水线更新一些的特性:

- 相比脚本化的流水线语法，它提供更丰富的语法特性
- 是为了使编写和读取流水线代码更容易而设计的

**Pipeline的基本结构**

- pipeline

  流水线的最外层结构，代表整条pipeline，包含着pipeline的完整逻辑;是声明式流水线语法的关健特征

- node 和 agent

  用于定义任务在哪里执行

  每个node都是一个 Jenkins 节点，可以是 Jenkins master也可以是 Jenkins agent，node是执行step的具体服务器。

  node 代码块也是脚本式pipeline语法的关健特性,声明式pipeline使用 agent 关健字

- stages

  用于包含所有stage的定义

- stage

  属于stages的子语句块

  指定 stage 的名称, 用于定义每个阶段 stage 的主要任务

  一个pipeline可以划分为若干个stage，每个stage都是一个完整的操作，比如: clone代码、代码编译、代码测试和代码部署，阶段是一个逻辑分组，可以跨多个node执行。

- steps

  属于stage的子语句块

  每个阶段stage中定义完成该阶段功能所需要经历的一系列步骤

  步骤 steps 是jenkins pipeline最基本的操作单元，从在服务器创建目录到构建容器镜像，由各类Jenkins 插件提供实现，例如： sh “make”

  能够把这些步骤steps 同该stage中的其它定义（如环境的定义,Post 等）分隔开

- post

  用在stage 代码块（和steps 同级）或整个pipeline执行完成后的附加步骤，此指令非必须项

**脚本式流水线语法**

```
https://www.jenkins.io/zh/doc/book/pipeline/
```

```bash
 node {  
    stage('Source') {
        //git clone 
    }
    stage('Build') {
        //mvn 
    }
    stage('Test') { 
        //mvn test
    }
    stage('Deploy') { 
        // scp 
        // java -jar 
    }
 }
 #特点：最外层是node {}     
```

**声明式流水线语法**

声明式流水线是在"Pipeline plugin"的2.5版本添加到 Jenkins 流水线的 ，它在流水线子系统之上提供了 一种更简单，更常见的语法。 

所有有效的声明式流水线必须包含在一个 pipeline 块中, 比如

```
 pipeline {
    /* insert Declarative Pipeline here */
 }
```

官方脚本

```
https://www.jenkins.io/zh/doc/book/pipeline/syntax/
```

**Pipeline 的基本结构** 

实线部分为必须段，虚线为可选段

![image-20250225142846592](5day-png/32Pipeline 的基本结构.png)

pipeline的定义有一个明确的、必须遵循的结构，它由一些directive和section组成，每一个section又可包含其它的section、directive和step，以及一些condlition的定义

Section:用于将那些在某个时间点需要一同运行的条目(item）组织在一起

- agent section:指定负责运行代码的节点

  在pipeline代码块的顶部，必须要有一个agent来指定“默认”的执行节点

  而一个stage的顶部也可以有一个agent的定义，用来指定负责运行该stage中的代码的节点

- stages section:组织---到多个stage

- steps section:组织一至多个DSL格式的步骤

- post section:在stage或整个pipeline的尾部封装--些需要被执行的步骤或者检验条件

Directive(指令)︰负责完成特定功能的语句或代码块，如environment、tools、triggers、input和when等

Steps : steps本身就是一个标识特定section的名称，其内部可以使用任何合法的DSL语句，例如git、sh、 bat和echo等

**Pipeline的声明式语法要点**

- steps内部的命令，每一条单独的命令都在当前任务的工作目录下执行。

  即使A命令切换到了一个新的目录，接下来的B命令并不会在对应的新目录中执行，而是在当前任务的工作目录下执行。如果非要在切换后的目录下执行命令B，那么采用she11中的&&符号将多条命令拼接在一起即可。

- 默认情况下，不支持shell里面的复杂语法，因为groovy有自己的条件表达式

- 如果jenkins的工作目录下存在同名目录，则获取失败

基本语法

```bash
pipeline {
   agent any 
   environment{
   url='http://www.wangxiaochun.com'
   }
   stages {
       stage('Source') {
   steps {
               // 
               echo "Access ${url}"
           }
   }
       stage('Build') { 
           steps {
               // 
           }
       }
       stage('Test') { 
           steps {
               // 
           }
       }
       stage('Deploy') { 
           steps {
               // 
           }
       }
   }
}
#特点：最外层是 pipeline {}
```

### Pipeline 常见指令说明

**agent** **可接受多种形式的参数**

- any:任何可用节点

- none:用于pipeline顶端时表示不定义默认的agent，每个stage就需要单独指定

- label { label ""}:具有指定的标签的节点均为可用节点

- node { label "" }:与label相似，但可以指定额外的参数customWorkspace

- docker:在指定的容器中运行pipeline或stage代码，该容器动态创建并运行于预配置的可运行容器的node上，或能够匹配到指定label的node上;可用参数如下

  image、label、args、rgistryUrl和rcgistryCredentialsId

- dockerfile:功能上类似于上面docker参数，但容器镜像通过指定的docker进行构建;该参数要求Jenkinsfile必须从Multibranch Pipeline或者Pipeline from SCM中加载;可用参数如下

  filename、dir、label、additionalBuildArgs、args、registryUrl和registryCredentialsId

- kubernetes:于Kubernetes集群上指定的Pod中运行stage或pipeline代码，该参数同样要求Jenkinsfile必须从Multibranch Pipeline或者Pipeline from SCM中加载

  需要在kubernetes参数中指定Pod模板

**Stages和Stage**

- stages是pipeline中最重要的section，stages负责描述pipeline中绝大部分的实际工作( work)
- Jenkins会按照stages中定义的顺序自卞而后执行各个stage 
- stages:封装了用于定义pipeline主体和逻辑的所有stage的定义，它包含一个或多个stage
- stages中至少需要包含一个stage，每个stage 指令来定义CD过程的每个离散部分，
- stage内部还支持再嵌套一个stages或一个parallel代码块，而后在这些代码块内部再嵌套stage，以指定stage的运行顺序
- stage内部嵌套stages，用于指定以顺序（串行)方式依次运行该stages内部的各stage ;而stage内部嵌套的parallel{}}，用于指定以并行方式运行该parallel号内部的各stage
- stage内部仅能定义steps、stages、parallel或matrix四者其中之一，且多层嵌套只能用在最后一个stage中;
- 对于本身已经嵌套在parallel或matrix内部的stage来说，不支持在其内部再使用parallel或matrix;但仍能使用agent、tools和when等其它指令，甚至是stages书配置段以顺序运行stage

**stage和steps**

- 每个stage都只包含一个steps
- Pipeline的基本结构决定了pipeline的整体流程，但真正“做事”是其内部一个个具体的step，因而steps是pipeline中最核心的组成部分
- steps 负责在stage中定义一到多个DSL语句，steps 中的语句负责完成该stage中特定的功能，例如构建、测试和部署等;但能够同其它的语句分隔开，如environment等
- 除了script，几乎所有的step在pipeline中都是不可拆分的原子操作
- pipeline内置了大量的step，具体请参考https:// www.jenkins.io/doc/pipeline/steps
- 除此之外，有相当一部分插件可直接当作step来用
- steps段中的script{}步骤负责将groovy脚本引入到steps{}配置段中，但它非为必要的步骤，且复杂的脚本应该单独组织为Shared Libraries，并由Pipeline导入后使用

**post section**

在stage或pipeline的尾部定义一些step，并根据其所在stage或pipeline的完成情况来判定是否运行这些step

post section支持的condition如下

- always:总是运行
- changed:其所处的stage或pipeline同前一次运行具有不同状态时，才运行该post
- fixed: stage或pipeline本次运行成功，但前一次为failed或unstable时，才运行该post
- regression: stage或pipeline前一次运行成功，但本次为failure、unstable或aborted时，才运行该post
- aborted: stage或pipeline的运行状态为aborted时，才运行该post;在Web UI中灰色显示
- failure: stage或pipeline的运行状态为failed时，才运行该post
- success: stage或pipeline的运行状态为success时，才运行该post
- unstable:因测试失败或代码冲突导致stage或pipeline的运行状态为unstable时，才运行该post;在Web UI中以黄色显示
- unsuccessful: stagc或pipeline的运行不成功时，才运行该post
- cleanup:在其它所有的post的条件均被评估后（无论stage或pipeline的状态为何）才运行该post

**Jenkins Pipeline** **支持常用指令**

无论是脚本式语法还是声明式语法，本质上都是执行各种命令，对于不同的命令需要采用专用的语法来实现指定的功能，常见的语法命令及其样式如下:

- echo: 输出信息， echo "Building"

- sh: 执行命令，sh 'command' sh([script: 'echo hello'])，用三个单号可以支持多行命令，即：sh ''' 多行shell命令 '''

- git: 克隆代码，git branch: 'develop', credentialsId: 'd7e3bd', url: 'git@gitlab.wang.org:example/myapp.git'

- env: 设置变量， env.PATH="/usr/local/java/bin:$PATH"

- environmet:设定环境变量，可用于stage或pipeline代码块中;支持credentialsl)函数，用于通过标识符访问预定义的凭证

- tools:指定需要在agent上下载并配置的工具，例如git、maven、jdk等，这些工具可经由PATH环境变量指定的位置访问到;可用于stage或pipeline中

- parameters:用户在触发pipeline时应该提供的参数列表;仅可用于pipeline级别

- options:仅可用在pipeline级别来配置pipeline自身的选项，支持的参数可由pipeline自身提供，也可由其它插件(例如timestamps）提供

  例如“retry(2)”允许在pipeline失败时重试两次

- triggers:用于指定负责自动启动pipeline的触发器，对于集成了Github或Gitlab等自带触发机制的系统场景，triggers并非必须的指令;仅可用于pipeline级别

- libraries:当前pipeline可以导入的共享库，该共享库内部的代码则可被该pipeline调用

- input: stagc中的专用指令，用于暂停pipeline并提示用户输入内容后继续

- when: stage中的专用指令，用于设定该stage的运行条件

**文件/目录相关指令**

- isUnix :判断是否为类Unix系统
- deleteDir:删除当前目录
- dir(" / path/ to/dir"):切换到指定目录
- fileExists (" /path/ to/dir"):判断文件是否存在
- pwd:打印当前目录
- writeFile:将内容写入指定的文件中，支持如下几个参数
  - file:文件路径，支持相对路径和绝对路径，
  - text:要写入的内容;
  - encoding:目标文件的编码，空值为系统默认的编码;支持base64编码格式;可选参数

- readFile:读取文件的内容;支持如下几个参数;
  - file:文件路径，支持相对路径和绝对路径;
  - encoding:读取文件内容时使用的编码格式;可选参数;

**消息或控制指令**

- echo("message"'):打印指定的消息;
- error("message"):主动报错，并中止当前pipeline;
- retry(count):重复执行count次在{}中定义的代码块
- sleep: 让pipeline休眠一段时间，支持如下参数;
  - time:整数值，休眠时长
  - unit:时间单位，支持NANOSECONDS，MICROSECONDS、MILLISECONDS、SECONDS、MINUTES、HOURS和DAYS，可选参数
- timeout:代码块的超时时长，支持如下参数
  - time:整数值，休眠时长
  - unit:时间单位，支持NANOSECONDS、MICROSECONDS,MILLISECONDS、SECONDS、MINUTES、HOURS和DAYS，可选参数
  - activity:布尔类型，值为true时，表示在该代码块不再有月志活动时才算真正超时;可选参数
- waitUntil:等待指定的条件满足时执行定义的代码块
  - initialRecurrencePeriod:初始的重试周期，即测试条件是否满足的重试周期，默认为250ms;可选参数
  - quiet:是否禁止将每次的条件测试都记入日志，默认为false，即记入日志;可选参数

**发送通知指令**

- mail:向指定邮箱发送邮件
- subject:邮件标题;√不body:邮件正文
- from (optional):发件人地址列表，逗号分隔
- cc (optional) : CC email地址列表，逗号分隔
- bcc (optional): BCC email地址列表，逗号分隔
- charset(optional):编码格式
- mimeType (optional): Email正文的MIME类型，默认为text/plain
- replyTo (optional):回件地址，默认为Jenkins设置的全局配置中的邮箱地址

**Node和Process相关指令**

- bat: Windows的批处理脚本

- powershell:运行指定的PowerShell脚本，支持Microsoft PowerShell 3+

- pwsh: PowerShell Core Script

- node:在指定的节点上运行后续的脚本

- ws :分配工作空间

- sh:运行shell脚本，支持的参数如下

  - script:脚本代码块，支持指定脚本解释器，例如“#!/usr/bin/python3，否则将使用系统默认的解释器，且使用了-xe选项

  - encoding (optional):脚本执行后输出的日志信息的编码格式，未定义时使用系统默认编码格式label(optional):显示在Web UI中的详细信息

  - returnStdout (optional):布尔型值，true表示任务的标准输出将作为step的返回值，而不是打印到日志中;若有错误,依然会记入日志

  - returnStatus (optional):正常情况了命令执行失败会返回非零状态码，设定该参数值为true时，表示将返回该step的结果，而非状态码

**Pipeline** **简单案例**

范例: 脚本式

```bash
node {
 	stage('Get code') {
		echo '获取代码'
 		//git clone
 	}
 	stage('Build') {
 		echo '构建项目代码'
 	}
 	tage('Test') {
 		echo '测试项目功能'
 	}
 	stage('Deploy') {
 		echo '部署项目'
 	}
}
```

范例: 声明式

```bash
pipeline {
 	agent any
 	stages {
 		stage('获取代码') {
 			steps {
 				echo '获取代码'
 			}
 		}
 		stage('构建代码') {
 			steps {
 				echo '构建项目代码'
 			}
 		}
 		stage('代码测试') {
			steps {
 				echo '测试项目功能'
 			}
 		}
 		stage('项目部署') {
 			steps {
 				echo '部署项目'
 			}
 		}
 	}
}
```

### 常用插件

pipeline

Pipeline Stage View

Blue Ocean

### **声明式** **Pipeline**

steps 内部的命令，每一条单独的命令都在当前任务的工作目录下执行。

```groovy
pipeline {
    agent any
    environment {
        APP = "testapp"
    }
    stages {
        stage ('cmd test') {
            steps {
                echo '命令测试'
                sh '''
                    pwd
                    rm -rf *
                    mkdir -p testdir1/testdir2
                    cd testdir1/testdir2 && pwd
                    pwd && tree
                    echo $WORKSPACE
                    echo $JOB_NAME
                    mkdir -p $WORKSPACE/$JOB_NAME
                    touch $WORKSPACE/$JOB_NAME/${APP}.log
                    pwd && tree
                '''
            }
        }
    }
}
```

**脚本解析**

1. **环境变量**：
   - `APP = "testapp"`：定义了一个环境变量 `APP`，值为 `testapp`。
2. **Agent**：
   - `agent any`：表示该 Pipeline 可以在任何可用的 Agent 上运行。
3. **阶段（Stage）**：
   - 只有一个阶段 `cmd test`，用于测试 Shell 命令。
4. **步骤（Steps）**：
   - `echo '命令测试'`：输出提示信息。
   - `sh 'pwd'`：打印当前工作目录。
   - `sh 'rm -rf *'`：删除当前目录下的所有文件和文件夹。
   - `sh 'mkdir testdir1'`：创建目录 `testdir1`。
   - `sh 'cd testdir1 && pwd && mkdir testdir2 && cd testdir2 && pwd'`：
     - 进入 `testdir1` 目录。
     - 打印当前目录。
     - 在 `testdir1` 中创建 `testdir2` 目录。
     - 进入 `testdir2` 目录并打印当前目录。
   - `sh 'pwd && tree'`：打印当前目录并显示目录树结构。
   - `sh 'echo $WORKSPACE'`：打印 Jenkins 工作目录路径。
   - `sh 'echo $JOB_NAME'`：打印当前任务的名称。
   - `sh 'mkdir $WORKSPACE/$JOB_NAME'`：在 Jenkins 工作目录下创建一个以任务名称命名的目录。
   - `sh 'touch $WORKSPACE/$JOB_NAME/${APP}.log'`：在刚创建的目录中创建一个日志文件，文件名为 `testapp.log`。
   - `sh 'pwd && tree'`：再次打印当前目录并显示目录树结构。

### 变量

Jenkins环境变量可分为内置变量和用户自定义变量两类

pipeline和stage得中用于定义环境变量的指令是environment，但定义位置的不同，也意味着其作用域的不同

定义在pipeline顶部的环境变量可被其后的各stage所引用

Jenkins全局环境变量可被所有的pipeline引用，它们以“env.”为前缀

引用全局环境变量格式有四种:

```bash
${env.ENV_VAR_NAME}
$env.ENV_VAR_NAME
$ENV_VAR_NAME
${ENV_VAR_NAME}

#注意：变量引用有时要加双引号引起来，如："${env.<ENV_VAR_NAME>}"
```

范例：声明和使用变量

```groovy
pipeline {
    agent any    
    environment {
        NAME = "hezhaokang"
        CLASS = "M61"
    }
    stages {
        stage('declare var') {
            steps {
                script {
                    env.LOGIN = sh(returnStdout: true, script: "who | wc -l").trim()
                }
            }
        }
        stage('get var') {
            steps {
                echo "NAME=${env.NAME}"
                echo "NAME=$NAME"
                echo "NAME=${NAME}"
                echo "NAME=$env.NAME"
                echo "LOGIN=$LOGIN"
                echo "LOGIN=${env.LOGIN}"
            }
        }
    }
}
```

### **使用凭据** **Credential**

**此依赖于Credential Binding Plugin， 该插件通常在安装社区推荐的插件时会由Jenkins自行部署**

- Username with password

  用户名和密码使用变量名可自行指定，Jenkins都会通过credentialsID从指定的凭证提取出来用户名和密码并赋值给指定对应的变量

  ```bash
  withCredentials([usernamePassword(credentialsID: '<ID>',
  usernameVariable: '<variable to hold username>',
  passwordVariable: '<variable to hold password>')])
  ```

- SSH密钥

  ```
  withCredentials(sshUserPrivateKey(credentialsId: '<credentials-id>’,
  keyFileVariable: 'MYKEYFILE'
  passphraseVariable: 'PASSPHRASE’，
  usernameVariable: USERNAME')])
  { // some block }
  ```

  若安装了插件 SSH Agent Plugin，还可以使用“sshagent([]) { }”代码块

  ```
  stage("Update Source'){
    sshagent(["gitlab-ssh-private-key-root"]) {
      sh "git tag -a ${env.BUILD_TAG} -m 'test'"
      sh "git push ${sshRepodef} --tags"
    }
  }
  ```



范例：基于变量构建helloworld-spring-boot项目和推送Docker镜像

```groovy
pipeline {
    agent any
    //tools {
    //    maven 'maven-3.6.3'
    //}
    environment {
        codeRepo="git@gitlab.kang.com:devops/spring-boot-helloworld.git"
        credential="gitlab-kang-key"
        harborServer='harbor.kang.com'
        projectName='spring-boot-helloworld'
        imageUrl="${harborServer}/example/${projectName}"
        imageTag="${BUILD_ID}"
        harborUserName="kang"
        harborPassword="Hh123456"
    }
    stages {
        stage('Source') {
            steps {
                git branch: 'main', credentialsId: "${credential}", url: "${codeRepo}"
            }
        }
        stage('Test') {
           steps {
                //注意:不要修改hello()函数,否则会导致下面失败
                sh 'mvn test'
            }
        }        
        stage('Build') {
            steps {
                //sh 'mvn -B -DskipTests clean package'
                 sh 'mvn clean package -Dmaven.test.skip=true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build . -t "${imageUrl}:${imageTag}"'
            }           
        }
        stage('Push Docker Image') {
            steps {
                //sh "echo ${harborPassword} | docker login -u ${harborUserName} --password-stdin ${harborServer}"
                sh "docker login -u ${harborUserName} -p ${harborPassword} ${harborServer}"
                sh "docker push ${imageUrl}:${imageTag}"
            }   
        }
        stage('Run Docker ') {
            steps {
                sh 'ssh root@10.0.0.100 "docker rm -f ${projectName} ; docker run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"'
                sh 'ssh root@10.0.0.101 "docker rm -f ${projectName} ; docker run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"'
                //sh "docker -H 10.0.0.101 rm -f ${projectName} ; docker -H 10.0.0.101 run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"
                //sh "docker -H 10.0.0.102 rm -f ${projectName} ; docker -H 10.0.0.102 run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"
          
            }   
        }  		
    }
}
```

### 使用凭据提取密码

```groovy
pipeline {
    agent any
    //tools {
    //    maven 'maven-3.6.3'
    //}
    environment {
        codeRepo="git@gitlab.kang.com:devops/spring-boot-helloworld.git"
        credential="gitlab-kang-key"
        harborServer='harbor.kang.com'
        projectName='spring-boot-helloworld'
        imageUrl="${harborServer}/example/${projectName}"
        imageTag="${BUILD_ID}"
        //harborUserName="magedu"
        //harborPassword="Magedu123"
    }
    stages {
        stage('Source') {
            steps {
                git branch: 'main', credentialsId: "${credential}", url: "${codeRepo}"
            }
        }
        stage('Test') {
           steps {
                //注意:不要修改hello()函数,否则会导致下面失败
                sh 'mvn test'
            }
        }        
        stage('Build') {
            steps {
                //sh 'mvn -B -DskipTests clean package'
                 sh 'mvn clean package -Dmaven.test.skip=true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build . -t "${imageUrl}:${imageTag}"'
            }           
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'login-harbor-passwd', \
                        usernameVariable: 'harborUserName',passwordVariable: 'harborPassword')]) {
                    //sh "echo ${harborPassword} | docker login -u ${env.harborUserName} --password-stdin ${harborServer}"
                    sh "docker login -u ${env.harborUserName} -p ${harborPassword} ${harborServer}"
                    sh "docker push ${imageUrl}:${imageTag}"
                    echo "username=${env.harborUserName}"
                    echo "password=${harborPassword}"
                }
            }
        }
        
        stage('Run Docker ') {
            steps {
                sh 'ssh root@10.0.0.100 "docker rm -f ${projectName} ; docker run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"'
                sh 'ssh root@10.0.0.101 "docker rm -f ${projectName} ; docker run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"'
                //sh "docker -H 10.0.0.101 rm -f ${projectName} ; docker -H 10.0.0.101 run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"
                //sh "docker -H 10.0.0.102 rm -f ${projectName} ; docker -H 10.0.0.102 run --name ${projectName} --restart always -p 80:8888 -d ${imageUrl}:${imageTag}"
          
            }   
        }  		
    }
}
```

![image-20250225161536620](5day-png/32Pipeline凭据.png)

### **参数选项和密码**

声明式Pipeline中，parameters指令用于为Pipeline声明参数

该指令用于pipeline之中，且常见于agent指令之后

其功用与Freestyle Job上的参数类似

**常用的参数类型有如下这些**

- string

  字符串类型的参数

  例如: parameters { string(name: 'DEPLOY_ENV, defaultValue 'staging‘ ,description:'description demo'}}

- text

  文本类型的参数，支持多行文本

  示例: parameters { text (name:'DEPLOY_TEXT,defaultValue:'One\nTwo\nThree\n',description:'description demo' )}

- booleanParam

  布尔型参数

  示例: parameters { booleanParam(name:'DEBUG_BUILD', defaultValue: true,description:'description demo')}

  值传递也是string类型

- choice

  选项型参数

  示例: parameters { choice(name: 'CHOICES', choices: 'one' 'two', 'three'],description:'description demo' )}

- password

  密码参数

  示例: parameters { password(name:'PASSWORD', defaultValue:'123456',description 'Ai secretpassword')}

**注意:第一个执行PipleLine没有Build with Parameters的提示,第二次执行才会出现**

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"
                echo "Biography: ${params.BIOGRAPHY}"
                echo "Toggle: ${params.TOGGLE}"
                echo "Choice: ${params.CHOICE}"
                // Avoid printing password in logs
                echo "Password is provided but hidden for security."
            }
        }
    }
}

```

### **交互输入实现确认和取消**

input 指令支持中断当前任务,以待确认和取消

input步骤是Pipeline与用户交互的接口，用于实现根据用户输入改变pipeline的行为遇到input步骤时，Pipeline会暂停下来并等待用户的响应

input步骤是特殊的参数化pipeline的方法，它常用于实现简易的审批流程，或者是手动实施后续的步骤等;

为了接收用户输入的不同数据，Jenkins提供了不同类型的参数

input步骤默认打印出的表单是打印一条消息并为用户提供--个选择:Proceed（继续）或者Abort

input步骤中，通过参数接收到的用户输入可以保存在变量中，而后进行调用

变量作用域仅为当前stage，例如，下面示例中的userInput变量仅能于当前步骤中调用

若要跨stage使用变量，则需要在pipeline代码外部先用def声明变量，而后再于stage中使用该变量;

input步骤的可用参数

- message，String类型

  其内容将打印给用户，并要求用户选择Proceed或Abort

  若input仅提供了该参数时，还可以省略message参数名称

- ok (optional) ,String类型

  用于自定义Proceed按钮的标签名称例如，下面的示例中，Proceed按钮的名称为“Yes"

input message: '', ok: 'Yes'

- id (optional)，String类型

  每个input都有一个惟一的ID标识，用于生成专用的URL，以便于根据用户输入继续或中止pipeline <JENKINS_URL>/job/l[job_name]/[build_id]/input/[input_id]/

  该URL可通过POST方法进行请求，后跟proceedEmpty表示空输入并继续，而abort则表示中止

  未定义ID时，Jenkins将自动为input生成lD

- parameters (optional)

  要求用户手动输入一个或多个参数列表,即可在INPUT中嵌套参数选项

  parameters指令支持的参数类型仅是input步骤支持的参数类型的-一个子集，因而，那些参数类型都会被input步骤所支持

- submitter(optional) , String类型

  可以进行后续操作的用户的ID或用户组列表，彼此间以逗号分隔，且前后不允许出现空格

- submitterParameter (optional)，String类型

  用于保存input步骤的实际操作者的用户名

范例

```groovy
stage('User Confirmation') {
    steps {
        script {
            def userInput = input(
                message: "Should we continue?", // 交互提示信息
                ok: "Yes, we should.", // 确认按钮文本
                submitter: "user1,user2", // 允许提交的用户列表
                parameters: [ // 交互输入的参数
                    string(name: 'REASON', defaultValue: 'No reason', description: 'Why should we continue?')
                ]
            )
            echo "User input received: ${userInput}"
        }
    }
}

```

范例

```groovy
pipeline{
    agent {
       any
    }
	stages{
        stage("get code"){
            steps{
                echo "git code from scm"
            }
        }
        stage("build"){
            steps{
                input message: '确定执行吗？', ok: '确定'
                echo "build code"
            }
        }
        stage("package"){
            steps{
                echo "package code"
            }
        }
        stage("deploy"){
            steps{
                echo "deploy package to nodes"
            }
        }
    }
}
```

![image-20250225163656361](5day-png/32Pipeline交互式.png)

### **条件判断**

```
https://www.jenkins.io/doc/book/pipeline/syntax/#flow-control
```

对于pipeline来说，使用if语句或者try语句,或者when来进行条件的流程控制，这两种方式效果相似

when是用在stage段中的指令，用于限定当前stage的运行条件

when指令中至少要含有一个条件

若同时定义了多个条件，它们之间的隐含关系为“与”，即必须同时满足，才会运行当前stage{}

when也允许嵌套定义条件的逻辑操作: not、allOf和anyOf

Jenkins内置支持的常用条件

- branch:满足给定的分支模式时运行当前stage{}

  格式为“branch pattern:, comnparator. ”，该指令仅适用于多分支流水线

  equals :即完全相同，例如“branch equals : main”

  glob:文件名路径格式的通配符，默认值，例如“branch glob: origin/ref/*”

  regexp:正确表达式模式，例如“branch regexp: release-\d+

- environment

  给定的变量满足指定的值时，运行当前stage{}

  格式: “environment name: , value:" 

  示例: when {environment name: 'DEPLOY_IO', value: 'production'}

- expression

  给定的groovy表达式结果为true时，运行当前stage{}

  表达式需要为布尔型，或者返回值为true或false的字符串

  示例: expression {BRANCH_NAME ==~ /(production | staging)/ }

- tag

  给定的TAG_NAME匹配指定的模式时，运行当前stage{}

  示例: when { tag "release-*"}

- changeRequest

  当前构建因某个指定的“change request（例如Pull Request或Merge Request)”触发时，运行当前分支

  空的参数值，表示任意的change request均满足条件，例如when { changeRequest() }

  支持使用过滤器属性，包括id, target, branch, fork, url, title, author, authorDisplayName和authorEmail

  示例: when { changeRequest target: 'master'}

  在过滤器属性上，还支持使用可选的comparator参数

  示例: when { changeRequest authorEmail: "[ \w_-.]+@example.com", comparator: ’REGEXP’}

- not:嵌套的条件结果为false时执行当前stage {}

  示例: when { not { branch 'master' }}

- allOf:嵌套的所有条件均为true执行当前stage

  示例: when { allOfbranch 'master'; environment name: 'DEPLOY_TO', value: 'production'} }

- anyOf:嵌套的条件至少有一个为true时执行当前stage{}

  示例: when { anyOf{branch 'master'; branch 'staging' }}

- triggeredBy

  当前的构建Job由指定的Trigger触发时运行当前stage{}

  示例: when { triggeredBy 'BuildUpstreamCause’ }

- 在进入stage内的agent之前评估when条件

  默认如果在stage定义了agent，则在进入该stage {}的agent后才会评估其内部的when条件

  但也可以显式将beforeAgent选项的值设定为true来更改此默认行为，即先评估when条件，且结果为true时才进入agent

- 在运行stage{}内的input之前评估when条件

  默认情况下,如果在stage{}定义了input，则在运行该stage{}的input后才会评估其内部的when条件

  但也可以显式将beforeInput选项的值设定为true来更改此默认行为，即先评估when条件，且结果为true时才执行input

- 在运行stage{}内的options之前评估when条件

  默认情况下,如果在stage{}定义了options，则在运行该stage{}的options后才会评估其内部的when条件

  但也可以显式将beforeOptions选项的值设定为true来更改此默认行为，即先评估when条件，且结果为true时才执行options

范例 分支捕获和异常捕获

```groovy
node {
    stage('Branch Check') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }

    stage('Error Handling') {
        try {
            sh 'exit 1' // 这里会触发异常
        } catch (exc) {
            echo 'Something failed, I should sound the klaxons!'
            throw // 继续抛出异常，让 Jenkins 任务失败
        }
    }
}
```

```bash
#在if中使用的条件值，一般使用she11命令的执行后的状态返回值来进行获取，由于jenkinsfile的特殊性，普通she11命令的状态返回值必须以文件的方式落地，也就是说stdout的内容重定向到文件，在jenkins的sh语法中，需要开启 return status: true属性
#格式如下
result = sh returnstdout: true ,script: "<shell command>"
result = sh(script: "<shell command>"，returnstdout: true)
#默认情况下，sh命令的状态返回值是О或者非0，如果想要指定状态返回值的内容的话，可以使用如下格式:
result = sh(script: "<shell command> && echo 'true' || echo 'false' " , returnstdout: true)
```

范例：when 条件

```groovy
pipeline {
    agent none
    stages {
        stage('Example Build') {
            agent any  // 这里需要指定 agent，否则会报错
            steps {
                echo 'Hello World'
            }
        }

        stage('Example Deploy') {
            agent any
            when {
                beforeInput true  // 先评估 when 条件，如果为 true 才进入 input
                branch 'main' // 只有 main 分支才会执行
            }
            input {
                message "Deploy to production?" // 用户确认框
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

范例：参数化和when条件执行

```groovy
pipeline {
    agent any
    parameters {
        booleanParam(name:'pushImage', defaultValue: 'true', description: 'Push Image to Harbor?')
    }    
    tools {
        maven 'maven-3.8.7'
    }
    stages {
        stage('Source') {
            steps {
                echo "source"
            }
        }
        stage('Build') {
            steps {
                echo "Build"
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Build Docker image"
            }           
        }
        stage('Push Docker Image') {
           agent any
           when {
                expression { params.pushImage }
                //expression { "${params.pushImage}" == 'true' }
            }
            steps {
            	echo "Push"
            }   
        }
        stage('Run Docker ') {
            steps {
				echo "run docker"
            }   
        }  		
    }
}
```

![image-20250225170843097](5day-png/32条件判断.png)

### **并行**

并行执行

```
pipeline {
    agent any
    stages {
        stage( 'Deploy') {
            parallel {
                stage('deploy_proxy'){
                    steps {
                        echo "部署反向代理服务"
                    }
                }
                stage('deploy app1') {
                    steps {
                        echo "部署deploy_app1应用"
                    }
                }
                stage ('deploy app2'){
                    stages {
                        stage ('delete container') {
                            steps {
                                 echo "删除旧容器"
                            }
                        }
                        stage('start container') {
                            steps {
                                echo "启动新容器"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

![image-20250225171305584](5day-png/32Pipeline并行.png)

### **触发器**

在Pipeline中实现GitLab trigger，注意：需要安装gitlab插件

在Pipeline代码中配置GitLab Trigger会用到如下几个触发条件

- triggerOnPush: GitLab仓库推送push事件时是否执行构建
- triggerOnMergeRequest:GitLab仓库推送mergeRequest事件时，是否执行构建
- branchFilterType:只有符合条件的分支才会被触发;必选配置，否则将无法实现触发，支持如下值
  - NameBaseFilter:基于分支名进行过滤，多个分支名彼此间以逗号分隔
  - RegexBaseFilter:基于正则表达式模式对分支名过滤
  - All:所有分支都会被触发
- includeBranchSpec:基于branchFilterType值，输入期望包括的分支的规则
- excludeBranchSpec:基于branchFilterType值，输入期望排队的分支的规则
- secretToken:在回调时使用的token

GitLab触发器配置格式：

```bash
triggers {
    gitlab(
        triggerOnPush: false,  // 不在每次推送时触发
        triggerOnMergeRequest: true,  // 合并请求触发构建
        triggerOpenMergeRequestOnPush: "never",  // 不会自动创建合并请求
        triggerOnNoteRequest: true,  // 允许通过评论触发构建
        noteRegex: "Jenkins please retry a build",  // 通过评论触发构建
        skipWorkInProgressMergeRequest: true,  // 跳过 WIP 合并请求
        ciSkip: false,  // 禁止使用 [ci skip] 跳过构建
        setBuildDescription: true,  // 自动设置构建描述
        addNoteOnMergeRequest: true,  // 添加构建结果注释
        addCiMessage: true,  // 添加 CI 消息
        addVoteOnMergeRequest: true,  // 合并请求上添加投票
        acceptMergeRequestOnSuccess: false,  // 构建成功时不自动接受合并请求
        branchFilterType: "NameBasedFilter",  // 基于分支名称过滤
        includeBranchesSpec: "release/*",  // 触发所有 release 分支
        excludeBranchesSpec: "",  // 不排除任何分支
        pendingBuildName: "Jenkins",  // 等待中的构建显示为 Jenkins
        cancelPendingBuildsOnUpdate: false,  // 不取消等待中的构建
        secretToken: "62dad2cd1d9ae62686ada8dc4cdOae66"  // 安全令牌
    )
}
```

#### **配置项说明**

1. **`triggerOnPush: false`**
   - 禁止在每次代码推送时触发构建。
2. **`triggerOnMergeRequest: true`**
   - 启用合并请求时触发构建，即每次创建或更新合并请求时都会触发构建。
3. **`triggerOpenMergeRequestOnPush: "never"`**
   - 在推送代码时 **不会** 自动创建合并请求。
4. **`triggerOnNoteRequest: true`**
   - 允许通过 **GitLab 评论** 来触发构建。
5. **`noteRegex: "Jenkins please retry a build"`**
   - 只有评论内容匹配该正则时（例如：`Jenkins please retry a build`），才会触发构建。
6. **`skipWorkInProgressMergeRequest: true`**
   - 如果合并请求是 "工作中"（WIP）状态，构建会被跳过。
7. **`ciSkip: false`**
   - 禁用 CI 跳过（即不允许在提交消息中添加 `[ci skip]` 来跳过构建）。
8. **`setBuildDescription: true`**
   - 在构建开始时自动设置构建的描述。
9. **`addNoteOnMergeRequest: true`**
   - 在 GitLab 合并请求上添加构建结果的注释。
10. **`addCiMessage: true`**
    - 将 CI 消息添加到 GitLab 的合并请求。
11. **`addVoteOnMergeRequest: true`**
    - 在合并请求上投票，通常用于集成测试状态。
12. **`acceptMergeRequestOnSuccess: false`**
    - 构建成功时不会自动接受合并请求。
13. **`branchFilterType: "NameBasedFilter"`**
    - 根据分支名称来过滤触发的条件。
14. **`includeBranchesSpec: "release/qat"`**
    - 仅对 `release/qat` 分支触发构建。
15. **`excludeBranchesSpec: ""`**
    - 没有指定排除分支，因此所有分支都会被考虑（不过你指定了 `includeBranchesSpec`，所以不会触发其他分支）。
16. **`pendingBuildName: "Jenkins"`**
    - 当构建处于等待状态时，在 GitLab 合并请求页面上显示的名字。
17. **`cancelPendingBuildsOnUpdate: false`**
    - 如果有新的提交推送， **不会取消正在等待的构建**。
18. **`secretToken: "62dad2cd1d9ae62686ada8dc4cdOae66"`**
    - 用于验证 GitLab 与 Jenkins 的通信安全性。

范例

```groovy
pipeline {
    agent any
    tools {
        maven 'maven-3.8.7'
    }
    triggers {
        gitlab(triggerOnPush: true,
            acceptMergeRequestOnSuccess: true,
            //triggerOnMergeRequest: true,
            branchFilterType: 'All',
            secretToken: '62dad2cd1d9ae62686ada8dc4cdOae66')
        }
    parameters {
        booleanParam(name: "PUSH", defaultValue: true)
        }
    environment {
        GitRepo="http://gitlab.kang.com/devops/spring-boot-helloWorld.git"
        credential="gitlab-kang-passeord"
        HarborServer='harbor.kang.com'        
        ImageUrl="devops/spring-boot-helloworld"
        ImageTag="latest"
        }
    stages {
        stage('Source'){
            steps { 
                git branch: 'main', credentialsId: "${credential}", url: "${GitRepo}"
                //git branch: 'main', url: "${GitRepo}"
            }
        }
    }
}
```

在gitlab上配置wobhooks

```
http://gitlab.kang.com/devops/spring-boot-helloWorld.git
62dad2cd1d9ae62686ada8dc4cdOae66
```

### **构建后操作**

流水线也提供了构建后的动作，常可用于消息通知

这个动作就在post 配置段中

post部分定义了一个或多个在管道或阶段运行完成后运行的附加步骤(取决于post 部分在管道中的位置)。

post可以支持以下任何后置条件块:

always、changed、fixed、regression、aborted、 failure、successunstable、 unsuccessful、cleanup。

这些条件块允许根据管道或阶段的完成状态在每个条件内执行步骤。条件块按如下所示的顺序执行。格式说明

```bash
#在stages同级别的位置，添加一个post配置段
post {
	always {
		echo '任执行结束后执行此操作'
	}
}
```

**邮件通知**

```groovy
pipeline {
    agent any
    parameters {
        booleanParam(name:'pushImage', defaultValue: 'true', description: 'Push Image to Harbor?')
    }    
    tools {
        maven 'maven-3.8.7'
    }
    stages {
        stage('Source') {
            steps {
                echo "source"
            }
        }
        stage('Build') {
            steps {
                echo "Build"
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Build Docker image"
            }           
        }
        stage('Push Docker Image') {
           agent any
           when {
                 expression { params.pushImage }
                //expression { "${params.pushImage}" == 'true' }
            }
            steps {
            	echo "Push"
            }   
        }
        stage('Run Docker ') {
            steps {
				echo "run docker"
				//sh 'false'
            }   
        }
    }
    post {
        always {
            mail to: 'zhaokang2004@outlook.com',
            subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
    }  
}
```

钉钉和微信

```bash
    post{
        success{
            dingtalk(
                robot: 'dingtalk',
                type: 'TEXT',
                text: [
                    "Status of pipeline: ${currentBuild.fullDisplayName}",
                    "${env.BUILD_URL} has result ${currentBuild.result}."
                ]
            )
        failure{
            qyWechatNotification failNotify: true, webhookUrl: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=58ac51d5-b10a-4637-8658-88ff1b8b2afa'
        }
    }
```

 范例：微信

```
pipeline {
    agent any
    parameters {
        booleanParam(name:'pushImage', defaultValue: 'true', description: 'Push Image to Harbor?')
    }    
    tools {
        maven 'maven-3.8.7'
    }
    stages {
        stage('Source') {
            steps {
                echo "source"
            }
        }
        stage('Build') {
            steps {
                echo "Build"
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Build Docker image"
            }           
        }
        stage('Push Docker Image') {
           agent any
           when {
                 expression { params.pushImage }
                //expression { "${params.pushImage}" == 'true' }
            }
            steps {
            	echo "Push"
            }   
        }
        stage('Run Docker ') {
            steps {
				echo "run docker"
				//sh 'false'
            }   
        }
    }
    post {
        always {
           qyWechatNotification failNotify: true, webhookUrl: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=58ac51d5-b10a-4637-8658-88ff1b8b2afa'
        }
    }  
}
```







### Pipeline script from SCM

![image-20250225183819311](C:/Users/zhaok/AppData/Roaming/Typora/typora-user-images/image-20250225183819311.png)

![image-20250225183954142](5day-png/32Pipeline script from SCM2.png)

## Jenkins 权限管理

**创建新用户**

Jenkins—系统管理—管理用户— 新建用户

### 安装插件

Role-based Authorization Strategy

![image-20250225184823787](5day-png/32jenkins权限管理.png)

 

### 角色管理

![image-20250225185614524](5day-png/32角色管理.png)

![image-20250225185649718](5day-png/32角色管理1.png)



### 项目管理

![image-20250225185915626](5day-png/32项目管理1.png)

![image-20250225190147620](5day-png/32项目管理2.png)

![image-20250225190358622](5day-png/32项目管理3.png)

![image-20250225190327721](5day-png/32项目管理4.png)

![image-20250225190451603](5day-png/32项目管理5.png)

## 视图管理

![image-20250225190959962](5day-png/32视图管理.png)

![image-20250225190936732](5day-png/32视图管理1.png)

### **Pipeline** **视图**

安装 build Pipeline 插件

![image-20250225191421672](5day-png/32pipeline视图.png)





## 代码质量检测 SonarQube

SonarQube 是一个开源平台，用于管理源代码的质量。

Sonar 不只是一个质量数据报告工具，更是代码质量管理平台。

支持的语言包括：Java、Go、Python、PHP、C、C++C#、C#、JavaScripts、Scala、HTML、PL/SQL、Swift、Ruby等29种语言。

SonarQube是一种自动代码审查工具，用于检测代码中的错误漏洞和代码异味，它集成到现有的工作流

程,以便在项目分支和拉取(PR)请求之间进行连续的代码检查

SonarQube 支持多种插件,实现和 Jenkins 等 CICD 工具的集成

**七个维度检测代码质量**

1. 可维护性（maintainability）

   所谓“代码易维护”就是指，在不破坏原有代码设计、不引入新的 bug 的情况下，能够快速地修改或者添

加代码。

2. 可读性（readability）

   在编写代码的时候，时刻要考虑到代码是否易读、易理解。除此之外，代码的可读性在非常大程度上会

影响代码的可维护性。

​		看代码是否符合编码规范、命名是否达意、注释是否详尽、函数是否长短合适、模块划分是否清晰、是

否符合高内聚低耦合等等。

​		code review 是一个很好的测验代码可读性的手段

3. 可扩展性（extensibility）

   表示代码应对未来需求变化的能力。跟可读性一样，代码是否易扩展也很大程度上决定代码是否易维

护.

​		代码的可扩展性表示，在不修改或少量修改原有代码的情况下，通过扩展的方式添加新的功能代码

4. 灵活性（flexibility）

   如果一段代码易扩展、易复用或者易用，都可以称这段代码写得比较灵活

5. 简洁性（simplicity）

   KISS ( Keep It Simple, Stupid)原则:尽量保持代码简单。代码简单、逻辑清晰，也就意味着易读、易维护.

6. 可复用性（reusability）

   代码的可复用性可以简单地理解为，尽量减少重复代码的编写，复用已有的代码

7. 可测试性（testability）

   代码可测试性的好坏，能从侧面上非常准确地反应代码质量的好坏。代码的可测试性差，比较难写单元测试，那基本上就能说明代码设计得有问题

**SonarQube** **架构**

基于C/S结构

**SonarQube** **四个主要组件**

![image-20250225191925794](5day-png/32SonarQube构架.png)

- SonarQube Server 包括三个主要部分
  - Web Server: UI 界面
  - Search Server :为UI提供搜索功能,基于 ElasticSearch 实现
  - Compute Engine Server：处理代码分析报告,并将之存储到 SonarQube Database
- SonarQube Database: 负责存储 SonarQube 的配置，以及项目的质量快照等
- SonarQube Plugin: 可以在 SonarQube Server 安装丰富的插件，实现支持各种开发语言、SCM、集成、身份验证和治理等功能
- Code analysis Scanners: 代码扫描器,是SonarQube Server的客户端, 将代码扫描后得出报告提交给 SonarQube Server

**SonarQube** **版本说明**

SonarQube 分为: 社区版,开发版,企业版和数据中心版

其中只有社区版是开源免费的

**SonarQube** **分两种版本****: LTS** **和非** **LTS** **版**

SonarQube 的 LTS (Long Term Support长期支持版本) 在其约 **18** **个月**的生命周期内提供组织稳定性和

错误修复。

**LTS** **新版称为** **LTA Long Term Active version**

生产建议使用 LTS 版

官方LTS版本说明

```
https://www.sonarqube.org/downloads/lts/
```

```powershell
LTS 版有如下版本
9.9
8.9
7.9
6.7
5.6
4.5
3.7
```

各种版本下载

```
https://www.sonarsource.com/products/sonarqube/downloads/historical-downloads/
https://www.sonarqube.org/downloads/
```

**安装环境准备**

**硬件要求**

硬件需求

- 小型应用至少需要2GB的RAM
- 磁盘空间取决于SonarQube分析的代码量
- 必须安装在读写性能较好的磁盘, 存储数据的目录中包含ElasticSearch的索引,服务器启动并运行时，将会在该索引上进行大是I/O操作
- 不支持32位操作系统

**系统内核优化**

新版要求

- vm.max_map_count is greater than or equal to 524288
- fs.file-max is greater than or equal to 131072
- the user running SonarQube can open at least 131072 file descriptors
- the user running SonarQube can open at least 8192 threads

You can set them dynamically for the current session by running the following commands as root :

```bash
#mv.max_map_count 用于限制一个进程可以拥有的VMA(虚拟内存区域)的数量,Ubuntu22.04默认值65530
sysctl -w vm.max_map_count=524288

#设置系统最大打开的文件描述符数
sysctl -w fs.file-max=131072

#每个用户可以打开的文件描述符数
ulimit -n 131072

#每个用户可以打开的线程数
ulimit -u 8192
```

注意: 必须修改内核限制，否则在启动时会报以下错误

**创建用户和修改内核配置**

```bash
[root@ubuntu2404 ~]#sysctl  vm.max_map_count
vm.max_map_count = 1048576
[root@ubuntu2404 ~]#sysctl fs.file-max
fs.file-max = 9223372036854775807

[root@ubuntu2404 ~]#ulimit -a
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 15184
max locked memory           (kbytes, -l) 495120
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1024
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 15184
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited

#Ubuntu22.04，此文件可不改,可选
[root@SonarQube-Server ~]# vim /etc/security/limits.conf
sonarqube - nofile 131072
sonarqube - nproc  8192

#如果以systemd 运行SonarQube,需要在service文件配置
[servcie]
.....
LimitNOFILE=131072
LimitNPROC=8192
......
```

**数据库环境依赖说明**

```
https://docs.sonarqube.org/7.9/requirements/requirements/
```

注意：SonarQube 7.9 不再支持MySQL，可以选择安装 PostgreSQL

**官方如下说明: 7.9.x 版本不再支持MySQL**

```
https://docs.sonarqube.org/7.9/setup/upgrade-notes/
```

**SonarQube 7.9** **以上版本不再支持** **java 11**

![image-20250225193605064](5day-png/32SonarQube java.png)

安装

```bash
#安装JAVA
[root@ubuntu2204 ~]#apt update && apt -y install openjdk-17-jdk

#创建用户
[root@ubuntu2204 ~]#useradd -s /bin/bash -m sonarqube
[root@ubuntu2204 ~]#id sonarqube
uid=1001(sonarqube) gid=1001(sonarqube) groups=1001(sonarqube)

#安装PostgreSQL
[root@ubuntu2204 ~]#apt list postgresql
Listing... Done
postgresql/jammy-proposed 14+238ubuntu0.1 all
N: There is 1 additional version. Please use the '-a' switch to see it
[root@ubuntu2204 ~]#apt -y install postgresql

#安装时自动生成用户postgres
[root@ubuntu2204 ~]#id postgres
uid=115(postgres) gid=121(postgres) groups=121(postgres),120(ssl-cert)

#默认监听在127.0.0.1的5432端口，需要修改监听地址
[root@ubuntu2204 ~]#ss -ntlp|grep post
LISTEN 0      244        127.0.0.1:5432       0.0.0.0:*    users:(("postgres",pid=129134,fd=5))    

#修改监听地址支持远程连接（如果sonarqube和PostgreSQL在同一台主机，可不做修改）
[root@ubuntu2204 ~]#vim /etc/postgresql/14/main/postgresql.conf 
listen_addresses = '*' 或者 '0.0.0.0'

[root@ubuntu2204 ~]#vim /etc/postgresql/14/main/pg_hba.conf
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             0.0.0.0/0               scram-sha-256

[root@ubuntu2204 ~]#systemctl restart postgresql

```

**创建数据库和用户授权**

```bash
[root@ubuntu2204 ~]#su - postgres 
#登录postgresql数据库
postgres@ubuntu2204:~$ psql -U postgres 
psql (14.15 (Ubuntu 14.15-0ubuntu0.22.04.1))
Type "help" for help.
#安全起见,修改数据库管理员postgres用户的密码,可选
postgres=# ALTER USER postgres WITH ENCRYPTED PASSWORD '123456';
ALTER ROLE

#创建用户和数据库并授权
postgres=# CREATE USER sonarqube WITH ENCRYPTED PASSWORD '123456';
CREATE ROLE
# OWNER sonarqube可选
postgres=# CREATE DATABASE sonarqube OWNER sonarqube;
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonarqube;
GRANT
#前面如果已经指定数据库的OWNER,则可以不执行下面命令,可选
postgres=# ALTER DATABASE sonarqube OWNER TO sonarqube;
ALTER DATABASE

#查看数据库是否创建,相当于MySQL中 show databases;
postgres=# \l
                                  List of databases
   Name    |   Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+-----------+----------+-------------+-------------+-----------------------
 postgres  | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 sonarqube | sonarqube | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |           |          |             |             | postgres=CTc/postgres
 template1 | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |           |          |             |             | postgres=CTc/postgres
(4 rows)

```



**下载sonarQube**

```
https://www.sonarqube.org/downloads/
https://www.sonarsource.com/products/sonarqube/downloads/historical-downloads/
```

```bash
[root@ubuntu2204 apps]#ls
sonarqube-9.9.8.100196.zip
[root@ubuntu2204 apps]#unzip sonarqube-9.9.8.100196.zip
[root@ubuntu2204 apps]#ln -s /data/apps/sonarqube-9.9.8.100196 /usr/local/sonarqube
#设置属性,注意：/usr/local/sonarqube/ 最后有斜线 /
[root@ubuntu2204 sonarqube]#chown -R sonarqube:sonarqube /usr/local/sonarqube/
```

**设置** **SonarQube** **连接数据库**

```bash
#修改SonarQube配置用于连接postgresql数据库
[root@ubuntu2204 apps]#vim /usr/local/sonarqube/conf/sonar.properties
#修改连接postgresql数据库的账号和密码,和前面的配置必须匹配
sonar.jdbc.username=sonarqube
sonar.jdbc.password=123456

#修改数据库相关的信息，这里必须和此前配置的postgresql内容相匹配，其中localhost为DB服务器的地址，而sonarqube为数据库名称
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
#默认配置如下，必须删除?currentSchema=my_schema
##sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube?currentSchema=my_schema

#设置 SonarQube 的提供的 Web Server监听的地址和端口,可选
sonar.web.host=0.0.0.0 #此为默认值,可不做修改
sonar.web.port=9000    #此为默认值,可不做修改

#按需要修改SonarQube存储数据的目录位置，以下两个目录为相对路径，相对于sonarqube的安装目录，也可以使用绝对路径
sonar.path.data=data   #默认值,可不做修改
sonar.path.temp=temp   #默认值,可不做修改
```

**启动SonarQube**

**注意:SonarQube需要调用Elasticsearch，而且默认需要使用普通用户启动，如果以 root 启动会报错**

```bash
#以sonarqube用户身份启动服务
#方法1:
[root@ubuntu2204 sonarqube]#su - sonarqube 
[sonarqube@ubuntu2204 ~]$/usr/local/sonarqube/bin/linux-x86-64/sonar.sh start
/usr/bin/java
Starting SonarQube...
Started SonarQube.

#方法2：或者执行下面启动SonarQube
[root@SonarQube-Server ~]#su - sonarqube -c '/usr/local/sonarqube/bin/linux-x86-64/sonar.sh start'

#查看状态
[sonarqube@ubuntu2204 ~]$/usr/local/sonarqube/bin/linux-x86-64/sonar.sh status
/usr/bin/java
Removed stale pid file: ./SonarQube.pid
SonarQube is not running.

#查看是否9000/tcp端口
[root@ubuntu2404 ~]#ss -tnulp | grep java
tcp   LISTEN 0      4096   [::ffff:127.0.0.1]:9001             *:*    users:(("java",pid=10592,fd=81))         
tcp   LISTEN 0      25                      *:9000             *:*    users:(("java",pid=10741,fd=12))         
tcp   LISTEN 0      4096   [::ffff:127.0.0.1]:44193            *:*    users:(("java",pid=10592,fd=79))     
```



**无法启动的主要的可能原因**

- 用户和权限
- java 版本不对或没有安装
- 内核参数没有优化
- 数据库及权限



### 创建service文件

先停止sonarqube

```bash
[root@ubuntu2404 ~]#su - sonarqube 
[sonarqube@ubuntu2404 sonarqube]$bin/linux-x86-64/sonar.sh stop
/usr/bin/java
Gracefully stopping SonarQube...
Stopped SonarQube.
```

```bash
[root@ubuntu2404 ~]#vim /usr/lib/systemd/system/sonarqube.service
[Unit]
Description=SonarQube service
After=syslog.target network.target
[Service]
Type=simple
User=sonarqube
Group=sonarqube
PermissionsStartOnly=true
ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-9.9.8.100196.jar
#ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -
Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-
9.9.7.96285.jar
#ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -
Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-
8.9.2.46101.jar
#ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -
Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-7.9.6.jar
StandardOutput=syslog
LimitNOFILE=131072
TimeoutStartSec=5
Restart=always
[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#systemctl daemon-reload
[root@ubuntu2404 ~]#systemctl enable --now sonarqube.service
[root@ubuntu2404 ~]#systemctl status sonarqube.service 
```

### 登录Web页面

**默认用户名和密码都是** **admin**

**安装中文支持**

中文插件

```
Chinese PackLocalization
```

安装完后,点 Restart Server 重启服务

查看多了一个文件

```bash
[root@ubuntu2404 ~]#ls /usr/local/sonarqube/extensions/plugins/
README.txt  sonar-l10n-zh-plugin-9.9.jar
```



插件可以在github上找到最新版本

```
https://github.com/SonarSource
```



### 权限管理

### 允许匿名访问

新版默认取消了匿名用户访问,可以在下面配置中打开匿名访问即关闭认证

![image-20250226195856475](5day-png/32Sonarqube权限管理.png)

**关闭上面开关后还要打开下面的两个选项**

![image-20250226200202771](5day-png/32Sonarqube权限管理1.png)



### **不允许匿名访问**

如果不允许匿名访问,就需要给 Jenkins 创建访问sonarqube 所使用的用户的访问令牌

可以创建新用户或使用默认的admin用户

**新建用户并授权**

- 在SonarQube上创建用户账号（不建议使用admin账号）

  配置 →权限 →用户

- 为用户账号赋予相应的权限，例如执行分析和置备项目

  配置 →权限 →全局权限

创建jenkins用户

![image-20250226201617339](5day-png/32Sonarqube权限管理4.png)

对 jenkin 用户创建 token

![image-20250226201004846](5day-png/32Sonarqube权限管理2.png)

**生成token令牌**

![image-20250226201314126](5day-png/32Sonarqube权限管理3.png)

```
squ_4a00c1e42a4ec29e17e60eab4ed6bd31a71d89a0
```

给用户分配置"执行分析"和"置备项目"权限

![image-20250226201709248](5day-png/32Sonarqube权限管理5.png)



### **部署代码扫描器** **sonar-scanner**

**sonar-scanner 安装方法1：手动下载安装**

下载地址

```bash
https://docs.sonarqube.org/latest/analyzing-source-code/scanners/sonarscanner/
https://binaries.sonarsource.com/?prefix=Distribution/sonar-scanner-cli/
https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/
```

```
[root@ubuntu2404 ~]#wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.0.2.4839-linux-x64.zip
```

**下载和配置较新的版本并配置**

```bash
[root@ubuntu2404 ~]#wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 ~]#mv sonar-scanner-cli-7.0.2.4839-linux-x64.zip /usr/local/src
[root@ubuntu2404 ~]#cd /usr/local/src
[root@ubuntu2404 local]#ls /src
sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 local]#cd src/
[root@ubuntu2404 src]#unzip src/sonar-scanner-cli-7.0.2.4839-linux-x64.zip 
[root@ubuntu2404 src]#ls
sonar-scanner-7.0.2.4839-linux-x64  sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 src]#ln -s /usr/local/src/sonar-scanner-7.0.2.4839-linux-x64 /usr/local/sonar-scanner
[root@ubuntu2404 src]#ls ..
bin  etc  games  include  lib  man  sbin  share  sonar-scanner  src

#查看版本，无需安装java
[root@ubuntu2404 src]#/usr/local/sonar-scanner/jre/bin/java --version
openjdk 17.0.13 2024-10-15
OpenJDK Runtime Environment Temurin-17.0.13+11 (build 17.0.13+11)
OpenJDK 64-Bit Server VM Temurin-17.0.13+11 (build 17.0.13+11, mixed mode, sharing)

[root@ubuntu2404 sonar-scanner]#ln -s /usr/local/sonar-scanner/bin/sonar-scanner /usr/local/bin
```

范例：根据上面创建用户和token信息修改sonar-scanner的配置文件

```bash
[root@ubuntu2404 src]#cat /usr/local/sonar-scanner/conf/sonar-scanner.properties
# Configure here general information about the environment, such as the server connection details for example
# No information about specific project should appear here

#----- SonarQube server URL (default to SonarCloud)
#sonar.host.url=https://mycompany.com/sonarqube

#sonar.scanner.proxyHost=myproxy.mycompany.com
#sonar.scanner.proxyPort=8002
#指向sonarqube服务器的地址和端口
sonar.host.url=http://sonarqube.kang.com:9000 
sonar.sourceEncoding=UTF-8

#如果登录sonarqube server需要验证，还要加下面两行sonarqube的预先创建的用户信息
#登录sonarqube服务器的验证信息

#方式1：基于用户名和密码
#sonar.login=admin
#sonar.password=123456
#sonar.login=jenkins
#密码方式未来会淘汰
#sonar.password=123456

#方式2：建议使用Token方式
sonar.login=squ_4a00c1e42a4ec29e17e60eab4ed6bd31a71d89a0
```

```
https://docs.sonarsource.com/sonarqube-server/latest/analyzing-source-code/scanners/sonarscanner/
```

```bash
[root@ubuntu2404 ginweb]#sonar-scanner 
21:02:10.846 INFO  Scanner configuration file: /usr/local/src/sonar-scanner-7.0.2.4839-linux-x64/conf/sonar-scanner.properties
21:02:10.860 INFO  Project root configuration file: NONE
21:02:10.897 INFO  SonarScanner CLI 7.0.2.4839
21:02:10.902 INFO  Java 17.0.13 Eclipse Adoptium (64-bit)
21:02:10.908 INFO  Linux 6.8.0-48-generic amd64
21:02:10.984 INFO  User cache: /root/.sonar/cache
21:02:15.364 INFO  Communicating with SonarQube Server 9.9.8.100196
21:02:16.753 INFO  Load global settings
21:02:17.273 INFO  Load global settings (done) | time=522ms
21:02:17.280 INFO  Server id: 86E1FA4D-AZU9eJm08VCYutpXTM7F
21:02:17.294 INFO  User cache: /root/.sonar/cache
21:02:17.306 INFO  Load/download plugins
21:02:17.306 INFO  Load plugins index
21:02:17.791 INFO  Load plugins index (done) | time=485ms
21:02:18.125 INFO  Plugin [l10nzh] defines 'l10nen' as base plugin. This metadata can be removed from manifest of l10n plugins since version 5.2.
21:02:23.689 INFO  Load/download plugins (done) | time=6384ms
21:02:25.786 INFO  Process project properties
21:02:25.821 INFO  EXECUTION FAILURE
21:02:25.828 INFO  Total time: 14.990s
21:02:25.830 ERROR Error during SonarScanner CLI execution
21:02:25.831 ERROR You must define the following mandatory properties for 'Unknown': sonar.projectKey
21:02:25.835 ERROR 
21:02:25.835 ERROR Re-run SonarScanner CLI using the -X switch to enable full debug logging.
```

错误解决方法

方法一

```bash
[root@ubuntu2404 ginweb]#cat sonar-project.properties 
# must be unique in a given SonarQube Server instance
sonar.projectKey=jinweb

# --- optional properties ---

# defaults to project key
#sonar.projectName=My project
# defaults to 'not provided'
#sonar.projectVersion=1.0
 
# Path is relative to the sonar-project.properties file. Defaults to .
#sonar.sources=.
 
# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8
```

方法二

```bash
#通过选项实现，无需配置文件sonar-project.properties
[root@jenkins spring-boot-helloWorld]#sonar-scanner -Dsonar.projectName=myapp -Dsonar.projectKey=myapp -Dsonar.sources=./ -Dsonar.java.binaries=./ -Dsonar.login=<token>
```

```
[root@ubuntu2404 ginweb]#sonar-scanner
```



### **Jenkins** **和** **SonarQube** **集成实现代码扫描**

<img src="5day-png/32Jenkins 和 SonarQube 集成实现代码扫描.png" alt="image-20250227102234191" style="zoom:50%;" />

Jenkins借助于SonarQube Scanner插件将SonarQube提供的代码质量检查能力集成到pipeline上,从而确保质量阈检查失败时，能够避免继续进行后续的操作，例如发布等

通常的流程如下

- Jenkins Pipeline启动
- SonarQube Scanner分析代码,并将报告发送至SonarQubeServer
- SonarQube Server分析代码检测的结果是否符合预定义的质量阈
- SonarQube Server将通过(passed)或者失败（failed)的结果发送回Jenkins上的SonarQube Scanner插件暴露的 Webhook
- 质量阈相关的阶段成功通过或可选地失败时Jenkins pipeline继续后面的Stage,否则pipeline将终止

### **SonarQube** **质量阈**

- 质量阙是一组预定义的评估条件
- 代码质量扫描结果可满足这组条件时,项目才会被标记为“passed”
- 管理员也可以在SonarQube上按需自定义并调用质量阈

**创建新的质量阈**

![image-20250227102738158](5day-png/32sinaeqube创建质量阀.png)

![image-20250227102928393](5day-png/32Sonarqube质量阀.png)



在SonarQube上添加webhook（网络调用），以便于Jenkins通过SonarQube Quality Gate插件（此插件无需安装）调用其“质量阈”信息

配置 --- 网络调用 webhook --- 创建

![image-20250227103603175](5day-png/32sonarqube-webhook.png)

注意：此处的密码没有复杂度和长度要求，但建议使用符合安全的密码

输入名称和下面 Jenkins的URL地址

注意: 此密码只用于防止攻击使用, 所以可以随意输入

```bash
http://jenkins.kang.com:8080/sonarqube-webhook

#生成随机Secret密码
#方式一
[root@ubuntu2404 ginweb]#openssl rand -base64 21
sF7bK1GYcK4FcxBHymVgGhs8F4Ir
#方式二
[root@ubuntu2404 ginweb]#openssl rand -hex 21
2521e05b8d3227e5a0cda5a1a73b1f60a69ac74fea
```

**jenkins服务器安装 sonar-scanner**

```bash
[root@ubuntu2404 ~]#cd /usr/local/src/
[root@ubuntu2404 src]#wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 src]#unzip sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 src]#ln -s /usr/local/src/sonar-scanner-7.0.2.4839-linux-x64 /usr/local/sonar-scanner
[root@ubuntu2404 src]#ln -s /usr/local/sonar-scanner/bin/sonar-scanner /usr/local/bin
[root@ubuntu2404 src]#cat /usr/local/sonar-scanner/conf/sonar-scanner.properties 
# Configure here general information about the environment, such as the server connection details for example
# No information about specific project should appear here

#----- SonarQube server URL (default to SonarCloud)
#sonar.host.url=https://mycompany.com/sonarqube

#sonar.scanner.proxyHost=myproxy.mycompany.com
#sonar.scanner.proxyPort=8002
#指向sonarqube服务器的地址和端口
sonar.host.url=http://sonarqube.kang.com:9000 
sonar.sourceEncoding=UTF-8
sonar.login=squ_4a00c1e42a4ec29e17e60eab4ed6bd31a71d89a0
```

### 案例2:基于ipeLine实现JAVA项目集成SonarQube代码检测通知Jenkins

![image-20250227105300907](5day-png/32综合案例1.png)

![image-20250227105323210](5day-png/32综合案例2.png)

```bash
10.0.0.206 harbor.kang.com		DNS
10.0.0.205 gitlab.kang.com
10.0.0.204 jenkins.kang.com
10.0.0.203 sonarqube.kang.com
```

#### 1 安装软件

安装DNS服务脚本

```bash
[root@ubuntu2404 ~]#cat install_dns.sh 
#!/bin/bash
#

#在HOST_LIST输入FQDN和IP的对应关系
HOST_LIST="
harbor 10.0.0.206
gitlab 10.0.0.205
jenkins 10.0.0.204
sonarqube 10.0.0.203"

DOMAIN=kang.com

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

```bash
[root@ubuntu2404 ~]#cat /etc/bind/kang.com.zone
$TTL 1D
@       IN SOA  master admin (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS       master
master  A        10.0.0.206         
harbor  A        10.0.0.206
gitlab  A        10.0.0.205
jenkins         A        10.0.0.204
sonarqube       A        10.0.0.203
```

##### **1 安装harbor**

```bash
[root@ubuntu2404 ~]#apt update;apt install -y docker.io docker-compose python3-pip
[root@ubuntu2404 ~]#ls
harbor-offline-installer-v2.12.2.tgz
[root@ubuntu2404 ~]#tar xf harbor-offline-installer-v2.12.2.tgz -C /usr/local/
[root@ubuntu2404 ~]#cd /usr/local/harbor/
[root@ubuntu2404 harbor]#cp harbor.yml.tmpl harbor.yml
[root@ubuntu2404 harbor]#ls
common.sh  harbor.v2.12.2.tar.gz  harbor.yml  harbor.yml.tmpl  install.sh  LICENSE  prepare
[root@ubuntu2404 harbor]#vim harbor.yml
hostname: harbor.kang.com
harbor_admin_password: 123456
data_volume: /data/harbor
#如果不使用https，还需要将下面行注释掉
# https related config
#https:
  # https port for harbor, default is 443
  #port: 443
  # The path of cert and key files for nginx
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path
  # enable strong ssl ciphers (default: false)
  # strong_ssl_ciphers: false
[root@ubuntu2404 harbor]#mkdir /data/harbor -p
[root@ubuntu2404 harbor]#bash install.sh 
[root@ubuntu2404 harbor]#cat /lib/systemd/system/harbor.service 
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor
[Service]
Type=simple
Restart=on-failure
RestartSec=5
ExecStart=/usr/bin/docker-compose -f  /usr/local/harbor/docker-compose.yml up
ExecStop=/usr/bin/docker-compose -f /usr/local/harbor/docker-compose.yml down
[Install]
WantedBy=multi-user.target

[root@ubuntu2404 harbor]#systemctl daemon-reload 
[root@ubuntu2404 harbor]#systemctl enable harbor.service 
Created symlink /etc/systemd/system/multi-user.target.wants/harbor.service → /usr/lib/systemd/system/harbor.service.
[root@ubuntu2404 harbor]#systemctl restart harbor.service 
[root@ubuntu2404 harbor]#systemctl status harbor.service 
```

##### **2 安装gitlab**

```bash
[root@ubuntu2404 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/noble/main/g/gitlab-ce/gitlab-ce_17.8.2-ce.0_amd64.deb
[root@ubuntu2404 ~]#apt install ./gitlab-ce_17.8.2-ce.0_amd64.deb
[root@ubuntu2404 gitlab]#vim /etc/gitlab/gitlab.rb 
external_url 'http://gitlab.kang.com'
[root@ubuntu2404 gitlab]#gitlab-ctl status
run: alertmanager: (pid 14690) 19s; run: log: (pid 14492) 71s
run: gitaly: (pid 14657) 21s; run: log: (pid 13994) 204s
run: gitlab-exporter: (pid 14667) 21s; run: log: (pid 14425) 88s
run: gitlab-kas: (pid 14214) 188s; run: log: (pid 14224) 187s
run: gitlab-workhorse: (pid 14633) 22s; run: log: (pid 14356) 101s
run: logrotate: (pid 13916) 217s; run: log: (pid 13924) 216s
run: nginx: (pid 14647) 22s; run: log: (pid 14373) 99s
run: node-exporter: (pid 14653) 22s; run: log: (pid 14408) 93s
run: postgres-exporter: (pid 14698) 18s; run: log: (pid 14514) 65s
run: postgresql: (pid 14045) 194s; run: log: (pid 14060) 191s
run: prometheus: (pid 14676) 20s; run: log: (pid 14469) 77s
run: puma: (pid 14275) 116s; run: log: (pid 14284) 112s
run: redis: (pid 13951) 211s; run: log: (pid 13960) 210s
run: redis-exporter: (pid 14669) 21s; run: log: (pid 14448) 83s
run: sidekiq: (pid 14297) 109s; run: log: (pid 14305) 108s
```

##### **3 安装jenkins**

```bash
[root@ubuntu2404 ~]#apt update && apt install -y openjdk-17-jdk
[root@ubuntu2404 ~]#wget https://mirror.tuna.tsinghua.edu.cn/jenkins/debian-stable/jenkins_2.492.1_all.deb
[root@ubuntu2404 ~]#dpkg -i ./jenkins_2.492.1_all.deb
[root@ubuntu2404 ~]#systemctl status jenkins.service 
#密码存放路径
[root@ubuntu2404 ~]#cat /var/lib/jenkins/secrets/initialAdminPassword
80b6d9dec9b5411c8604d990ba86c57a
```



##### **4 安装Sonarqube**

```bash
[root@ubuntu2404 ~]#apt update && apt -y install openjdk-17-jdk
[root@ubuntu2404 ~]#useradd -s /bin/bash -m sonarqube
[root@ubuntu2404 ~]#apt -y install postgresql
[root@ubuntu2404 ~]#systemctl restart postgresql
[root@ubuntu2404 ~]#su - postgres
postgres@ubuntu2404:~$ psql -U postgres
psql (16.6 (Ubuntu 16.6-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# ALTER USER postgres WITH ENCRYPTED PASSWORD '123456';
ALTER ROLE
postgres=# CREATE USER sonarqube WITH ENCRYPTED PASSWORD '123456';
CREATE ROLE
postgres=# CREATE DATABASE sonarqube OWNER sonarqube;
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonarqube;
GRANT
postgres=# ALTER DATABASE sonarqube OWNER TO sonarqube;
ALTER DATABASE
postgres=# \l
                                                         List of databases
   Name    |   Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |    Access privileges    
-----------+-----------+----------+-----------------+-------------+-------------+------------+-----------+-------------------------
 postgres  | postgres  | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 sonarqube | sonarqube | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =Tc/sonarqube          +
           |           |          |                 |             |             |            |           | sonarqube=CTc/sonarqube
 template0 | postgres  | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres            +
           |           |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres  | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres            +
           |           |          |                 |             |             |            |           | postgres=CTc/postgres
(4 rows)

postgres=# \q
postgres@ubuntu2404:~$ 
logout

[root@ubuntu2404 ~]#mkdir /data/apps -p
[root@ubuntu2404 ~]#cd /data/apps/
[root@ubuntu2404 apps]#ls
sonarqube-9.9.8.100196.zip
[root@ubuntu2404 apps]#ln -s /data/apps/sonarqube-9.9.8.100196 /usr/local/sonarqube
[root@ubuntu2404 apps]#chown -R sonarqube:sonarqube /usr/local/sonarqube/
[root@ubuntu2404 apps]#unzip sonarqube-9.9.8.100196.zip
[root@ubuntu2204 apps]#vim /usr/local/sonarqube/conf/sonar.properties
#修改连接postgresql数据库的账号和密码,和前面的配置必须匹配
sonar.jdbc.username=sonarqube
sonar.jdbc.password=123456

#修改数据库相关的信息，这里必须和此前配置的postgresql内容相匹配，其中localhost为DB服务器的地址，而sonarqube为数据库名称
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

#编写server文件
[root@ubuntu2404 ~]#vim /usr/lib/systemd/system/sonarqube.service
[Unit]
Description=SonarQube service
After=syslog.target network.target
[Service]
Type=simple
User=sonarqube
Group=sonarqube
PermissionsStartOnly=true
ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-9.9.8.100196.jar
#ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -
Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-
9.9.7.96285.jar
#ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -
Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-
8.9.2.46101.jar
#ExecStart=/usr/bin/nohup /usr/bin/java -Xms32m -Xmx32m -
Djava.net.preferIPv4Stack=true -jar /usr/local/sonarqube/lib/sonar-application-7.9.6.jar
StandardOutput=syslog
LimitNOFILE=131072
TimeoutStartSec=5
Restart=always
[Install]
WantedBy=multi-user.target

[root@ubuntu2404 ~]#systemctl daemon-reload
[root@ubuntu2404 ~]#systemctl enable --now sonarqube.service
[root@ubuntu2404 ~]#systemctl status sonarqube.service 
```



### 2 软件部署

**在Jenkins服务器安装Docker,并配置连接Harbor**

```bash
[root@ubuntu2404 ~]#apt update && apt -y install docker.io
[root@ubuntu2404 ~]#cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://si7y70hh.mirror.aliyuncs.com"],
  "insecure-registries": ["harbor.kang.com"]
}

#测试登录harbor
[root@ubuntu2404 ~]#docker login harbor.kang.com
Username: kang
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```



**在** **Jenkins** **创建凭据连接** **Harbor**

![image-20250227133902613](C:/Users/zhaok/AppData/Roaming/Typora/typora-user-images/image-20250227133902613.png)

**在** **Jenkins** **安装** **Maven** **工具**

```bash
[root@ubuntu2404 ~]#apt update && apt install maven -y
[root@ubuntu2404 ~]#vim /etc/maven/settings.xml
    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
      <blocked>true</blocked>
    </mirror>
```

**在** **Jenkins** **上配置** **Maven** **环境**

![image-20250227140056139](5day-png/32配置mvn环境.png)

**在** **Jenkins** **创建连接** **GitLab** **的凭据**

![image-20250227140422563](5day-png/32jenkins连接gitlab的凭据)

**在jenkins上修改jenkins用户使用docker的权限**

```bash
[root@ubuntu2404 ~]#usermod -G docker jenkins
[root@ubuntu2404 ~]#id jenkins 
uid=108(jenkins) gid=108(jenkins) groups=108(jenkins),109(docker)

#必须重启jenkins服务才能生效
[root@ubuntu2404 ~]#systemctl restart jenkins.service 
```



**在 Sonarqube 创建用户并授权**

创建jenkins用户，并生成jenkins令牌 token

```
squ_247a95927838f6bf00377b280cb1a10888167179
```

**在** **SonarQube** **添加** **Jenkins** **的回调接口** **webhook**

```bash
#url
http://jenkins.kang.com:8080/sonarqube-webhook
#生成Secet
[root@ubuntu2404 ~]#openssl rand -base64 21
uxdI3H9kvuCVGCrS8AN1wan8ZiL6
```

**在** **Jenkins** **创建访问** **Sonarqube** **的令牌凭据**

```bash
#添写前面创建的Sonarqube中用户令牌
squ_247a95927838f6bf00377b280cb1a10888167179
#指定凭据的ID
ID:sonarqube-jenkins-token
```

![image-20250227143143481](5day-png/32在 Jenkins 创建访问 Sonarqube 的令牌凭据.png)

**在** **Jenkins** **上配置系统的** **SonarQube** **服务器信息**

**注意:Name 名称区分大小写**

**注意：http://sonarqube.kang.com:9000 地址最后不能加/** ，新版无此有要求

**否则会报错误** **hudson.remoting.ProxyException: net.sf.json.JSONException: Invalid JSON String**

Manage Jenkins -- Configure System

注意这里的大小写 Sonarqube-Server

![image-20250227143851945](5day-png/32在 Jenkins 上配置系统的 SonarQube 服务器信息)

**在** **Jenkins** **上安装** **SonarQube Scanner**

Jenkins 上安装 SonarQube Scanner 以便在构建任务中调用,有两种方式

- 在 Jenkins 手动安装 sonarqube-scanner 

- Jenkins 全局工具配置中配置安装 sonar-scanner

  Manage Jenkins >全局工具配置

```bash
#在 Jenkins 手动安装 sonarqube-scanner
[root@ubuntu2404 ~]#wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 ~]#mv sonar-scanner-cli-7.0.2.4839-linux-x64.zip /usr/local/src
[root@ubuntu2404 ~]#cd /usr/local/src
[root@ubuntu2404 src]#ls /src
sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 src]#unzip src/sonar-scanner-cli-7.0.2.4839-linux-x64.zip 
[root@ubuntu2404 src]#ls
sonar-scanner-7.0.2.4839-linux-x64  sonar-scanner-cli-7.0.2.4839-linux-x64.zip
[root@ubuntu2404 src]#ln -s /usr/local/src/sonar-scanner-7.0.2.4839-linux-x64 /usr/local/sonar-scanner
[root@ubuntu2404 src]#ls ..
bin  etc  games  include  lib  man  sbin  share  sonar-scanner  src
[root@ubuntu2404 src]#ln -s /usr/local/sonar-scanner/bin/sonar-scanner /usr/local/bin
```

```bash
[root@ubuntu2404 src]#cat /usr/local/sonar-scanner/conf/sonar-scanner.properties
# Configure here general information about the environment, such as the server connection details for example
# No information about specific project should appear here

#----- SonarQube server URL (default to SonarCloud)
#sonar.host.url=https://mycompany.com/sonarqube

#sonar.scanner.proxyHost=myproxy.mycompany.com
#sonar.scanner.proxyPort=8002
sonar.host.url=http://sonarqube.kang.com:9000 
sonar.sourceEncoding=UTF-8
```

**准备微信机器人**

```
https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=0eadce83-eff6-499d-9cbb-b16134afaf6b
```

**创建任务使用**

```bash
pipeline {
    agent any
    tools {
        maven 'maven-3.8.7'
    }
    environment {
        codeRepo="http://gitlab.kang.com/devops/spring-boot-helloworld.git"
        credential='gitlab-kang-password'    
        harborServer='harbor.kang.com'
        projectName='spring-boot-helloworld'
        imageUrl="${harborServer}/example/${projectName}"
        imageTag="${BUILD_ID}"
    }
    stages {
        stage('Source') {
            steps {
                git branch: 'main', credentialsId: "${credential}", url: "${codeRepo}"
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }        
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('Sonarqube-Server') {
                    sh 'mvn sonar:sonar'
                    //sh 'mvn sonar:sonar -Dsonar.java.binaries=./'
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }	
        
        stage('Build') {
            steps {
                //sh 'mvn -B -DskipTests clean package'
                sh 'mvn -Dmaven.test.skip=true clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker image build . -t "${imageUrl}:${imageTag}"'
                // input(message: '镜像已经构建完成，是否要推送？')
            }           
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-kang-password', passwordVariable: 'harborPassword', usernameVariable: 'harborUserName')]) {
                    sh "docker login -u ${env.harborUserName} -p ${env.harborPassword} ${harborServer}"
                    sh "docker image push ${imageUrl}:${imageTag}"
                }
            }   
        }
        stage('Run Docker') {
            steps {
                sh 'ssh root@10.0.0.100 "docker rm -f ${projectName} && docker run --name ${projectName} -p 80:8888 -d ${imageUrl}:${imageTag}"'
                sh 'ssh root@10.0.0.101 "docker rm -f ${projectName} && docker run --name ${projectName} -p 80:8888 -d ${imageUrl}:${imageTag}"'
                //sh "docker -H 10.0.0.100:2375 rm -f ${projectName} && docker -H 10.0.0.202:2375 run --name ${projectName} -p 80:8888 -d ${imageUrl}:${imageTag}"
                //sh "docker -H 10.0.0.101:2375 rm -f ${projectName} && docker -H 10.0.0.203:2375 run --name ${projectName} -p 80:8888 -d ${imageUrl}:${imageTag}"
            }   
        }          
    }
  post {
        success {
            mail to: 'zhaokang2004@outlook.com',
            subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
        failure{
            qyWechatNotification failNotify: true, webhookUrl: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=0eadce83-eff6-499d-9cbb-b16134afaf6b'
        }
   } 
}
```



在后端服务器上部署docker 打通key验证 配置http





## 面试题

- 如何找回 Jenkins 密码
- 简述什么是版本控制系统（VCS）？
- 简述什么是 DevOps？
- 简述软件交付过程？四种常见场景：传统方式，基于Ansible，Docker 容器，Kubernetes













20250316
