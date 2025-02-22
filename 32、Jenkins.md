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

```
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
