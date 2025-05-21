# 19、tomcat

## java组成

![image-20241210093344917](5day-png\19java组成.png)

```
语言、语法规范。关键字,如: if、for、class等
源代码 source code
依赖库，标准库(基础)、第三方库(针对某些应用)。底层代码太难使用且开发效率低，封装成现成的库
JVM虚拟机。将源代码编译为中间码即字节码后,再运行在JVM之上
```

```
ABI（Application Binary Interface，应用程序二进制接口）:
	一种定义了应用程序与操作系统或者硬件之间交互方式的接口标准，不同的操作系统，他的ABI接口是不一样的。
	软件要想在某个操作系统上运行，就必须符合该操作系统的ABI接口规范才可以。
	它提供了在不同平台上编写、编译和执行应用程序的一致性。
```

```
API（Application Programming Interface，应用程序接口）：
	API可以在各种不同的操作系统上实现给应用程序提供完全相同的接口，
 	POSIX可移植操作系统接口，定义了操作系统应该为应用程序提供的接口标准，
 	- 它是各种UNIX操作系统上运行的软件而定义的一系列API标准的总称。
 	- POSIX为我们提供了统一且强大的接口，方便跨平台开发，涵盖了以下内容：
 		系统接口、命令和实用程序、网络文件访问等。
```

## MVC

```
模型（Model）：
 	模型表示应用程序的数据和业务逻辑。它负责处理数据的存储、检索和更新，同时包含应用程序的业务规则。模型通常不包含有关用户界面或展示方式的信息，而是专注于处理数据。
视图（View）：
 	视图负责用户界面的呈现和展示。它接收来自模型的数据，并将其以用户友好的方式显示。视图不负责处理数据，而只关注展示和用户交互。
控制器（Controller）：
 	控制器充当模型和视图之间的协调者。它接收用户的输入（通常来自用户界面），然后根据输入更新模型和/或视图。控制器负责处理用户请求、更新模型的状态以及选择适当的视图进行展示。
```

![image-20241210094201307](5day-png\19MVC.png)

```
职责分析：
Controller：控制器
	取得表单数据
	调用业务逻辑
	转向指定的页面
Model：模型
	业务逻辑
	保存数据的状态
View：视图
	显示页面
处理流程：
1. 用户发请求
2. Servlet接收请求数据，并调用对应的业务逻辑方法
3. 业务处理完毕，返回更新后的数据给servlet
4. servlet转向到JSP，由JSP来渲染页面
5. 响应给前端更新后的页面

MVC模式也有以下不足：
1、每次请求必须经过"控制器->模型->视图"这个流程，用户才能看到最终的展现界面，这个过程似乎有些复杂
2、实际上视图是依赖于模型的，换句话说，如果没有模型，视图也无法呈现出最终的效果
3、渲染视图过程是在服务端来完成的，最终呈现给浏览器的是带有模型的视图页面，性能无法得到很好的优化
```

## REST

![image-20241210094519630](5day-png\19REST.png)

```
浏览器这一端视为前端，而服务器那一端视为后端的话，可以将以上改进后的MVC模式简化为以下前后端分离模式
```

![image-20241210094619846](5day-png\19前后端分离.png)

## JDK和JRE

![image-20241210095527033](5day-png\19JRE和JDK.png)

```
关联关系
JDK包含JRE：
 	JDK是Java开发工具包，它是Java开发的完整包。JDK包括了JRE，以及用于Java开发的一系列工具和库，如编译器（javac）、调试器（jdb）、文档生成器（javadoc）等。因此，可以说JRE是JDK的一个子集；
JRE用于运行Java程序：
 	JRE是Java运行时环境，提供了在计算机上运行Java应用程序所需的所有组件。它包括Java虚拟机（JVM）、Java类库、Java命令行工具和其他支持文件。当用户仅需要运行Java程序时，安装JRE就足够了；
JDK用于开发和运行Java程序：
 	JDK不仅包含了JRE，还包含了用于开发Java应用程序的工具和库。开发人员使用JDK中的工具编写、编译和调试Java代码。JDK是面向Java开发人员的完整工具包，而JRE主要面向终端用户，用于执行Java应用程序；
```

![image-20241210095738266](5day-png\19JDK vs JRE.png)

![image-20241210095859090](5day-png\19JDK JRE 运行.png)

```
Oracle JDK稳定版本
	JDK8
	JDK11
	JDK17
```

## 软件安装

### deb或rpm安装

```powershell
root@ubuntu-43:~ # mkdir /data/softs -p 
root@ubuntu-43:~ # cd /data/softs/
root@ubuntu-43:softs # wget https://download.oracle.com/java/23/latest/jdk-23_linux-x64_bin.deb
#root@rocky9-13:server # wget https://download.oracle.com/java/23/latest/jdk-23_linux-x64_bin.rpm
root@ubuntu-43:softs # dpkg -i jdk-23_linux-x64_bin.deb 
root@ubuntu-43:softs # whereis java
java: /usr/bin/java /usr/share/java /usr/share/man/man1/java.1
root@ubuntu-43:softs # ll /usr/bin/java
lrwxrwxrwx 1 root root 22 Sep 30 07:20 /usr/bin/java -> /etc/alternatives/java*
root@ubuntu-43:softs # ll /etc/alternatives/java
lrwxrwxrwx 1 root root 43 Sep 30 07:20 /etc/alternatives/java -> /usr/lib/jvm/jdk-23.0.1-oracle-x64/bin/java*
root@ubuntu-43:softs # ll /usr/lib/jvm/jdk-23.0.1-oracle-x64/bin/java
-rwxr-xr-x 1 10668 10668 12328 Sep 30 07:20 /usr/lib/jvm/jdk-23.0.1-oracle-x64/bin/java*
root@ubuntu-43:softs # java -version
java version "23.0.1" 2024-10-15
Java(TM) SE Runtime Environment (build 23.0.1+11-39)
Java HotSpot(TM) 64-Bit Server VM (build 23.0.1+11-39, mixed mode, sharing)

删除环境jdk
root@ubuntu-43:softs # apt purge jdk-23
#root@rocky9-13:server # yum remove jdk-23
```

### 软件包安装

**Ubuntu**

```powershell
root@ubuntu-42:~ # mkdir /data/server -p
root@ubuntu-42:~ # cd /data/server/
root@ubuntu-42:server # ls
root@ubuntu-42:server # ls
jdk-11.0.19_linux-x64_bin.tar.gz
root@ubuntu-42:server # tar xvf jdk-11.0.19_linux-x64_bin.tar.gz -C /usr/local/
root@ubuntu-42:local # ls
bin  etc  games  include  jdk-11.0.19  lib  man  sbin  share  src
root@ubuntu-42:local # ln -s jdk-11.0.19/ jdk
root@ubuntu-42:local # ls
bin  etc  games  include  jdk  jdk-11.0.19  lib  man  sbin  share  src
root@ubuntu-42:local # vim /etc/profile.d/jdk.sh
root@ubuntu-42:local # cat /etc/profile.d/jdk.sh
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
root@ubuntu-42:local # . /etc/profile.d/jdk.sh
root@ubuntu-42:local # java -version
java version "11.0.19" 2023-04-18 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.19+9-LTS-224)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.19+9-LTS-224, mixed mode)
```

**rocky**

```powershell
root@rocky9-13:server # yum list java*openjdk*
root@rocky9-13:server # yum -y install java-11-openjdk.x86_64
root@rocky9-13:server # whereis java
java: /usr/bin/java /usr/lib/java /etc/java /usr/share/java /usr/share/man/man1/java.1.gz
root@rocky9-13:server # ll /usr/bin/java
lrwxrwxrwx. 1 root root 22 12月 10 11:37 /usr/bin/java -> /etc/alternatives/java
root@rocky9-13:server # ll /etc/alternatives/java
lrwxrwxrwx. 1 root root 62 12月 10 11:37 /etc/alternatives/java -> /usr/lib/jvm/java-11-openjdk-11.0.25.0.9-3.el9.x86_64/bin/java
root@rocky9-13:server # ll /usr/lib/jvm/java-11-openjdk-11.0.25.0.9-3.el9.x86_64/bin/java
-rwxr-xr-x. 1 root root 18072 11月 13 06:32 /usr/lib/jvm/java-11-openjdk-11.0.25.0.9-3.el9.x86_64/bin/java
root@rocky9-13:server # java -version
openjdk version "11.0.25" 2024-10-15 LTS
OpenJDK Runtime Environment (Red_Hat-11.0.25.0.9-1) (build 11.0.25+9-LTS)
OpenJDK 64-Bit Server VM (Red_Hat-11.0.25.0.9-1) (build 11.0.25+9-LTS, mixed mode, sharing)
```

```powershell
root@rocky9-11:~ # mkdir /data/server -p
root@rocky9-11:~ # cd /data/server/
root@rocky9-11:server # ls
jdk-11.0.19_linux-x64_bin.tar.gz
root@rocky9-11:server # tar xfv jdk-11.0.19_linux-x64_bin.tar.gz -C /usr/local/
root@rocky9-11:server # cd /usr/local/
root@rocky9-11:local # ls
bin  etc  games  include  jdk-11.0.19  lib  lib64  libexec  sbin  share  src
root@rocky9-11:local # ln -s jdk-11.0.19/ jdk
root@rocky9-11:local # ls
bin  etc  games  include  jdk  jdk-11.0.19  lib  lib64  libexec  sbin  share  src
root@rocky9-11:local # vim /etc/profile.d/jdk.sh
root@rocky9-11:local # cat /etc/profile.d/jdk.sh
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
root@rocky9-11:local # . /etc/profile.d/jdk.sh 
root@rocky9-11:local # java -version
java version "11.0.19" 2023-04-18 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.19+9-LTS-224)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.19+9-LTS-224, mixed mode)
```

## tomcat

**特点**

```
- Servlet容器：
 	Tomcat是一个Servlet容器，负责处理和执行Java Servlet。Servlet是一种用Java编写的服务器端程序，用于处理Web请求和生成动态Web内容；
- JSP支持：
 	Tomcat还支持JavaServer Pages（JSP），一种将Java代码嵌入HTML中的技术，用于创建动态Web页面；
- 轻量级：
 	Tomcat被设计为轻量级的应用服务器，易于安装和配置。它专注于提供基本的Servlet和JSP支持，而不像一些其他应用服务器那样包含大量的附加功能；
- 开源：
 	Tomcat是开源软件，可以免费使用，并且其源代码是开 放的，允许用户根据需要进行修改和定制；
- 连接器支持：
 	Tomcat支持多种连接器，可以与不同的Web服务器进行集成。最常见的是与Apache HTTP服务器一起使用的AJP（Apache JServ Protocol）连接器；
- 管理工具：
 	Tomcat提供了Web-based管理工具，允许管理员轻松地配置和监控Tomcat服务器。这包括一个Web界面，可以用来管理Web应用程序、虚拟主机、连接器等；
- 安全性：
 	Tomcat内置了一些安全特性，包括对SSL/TLS的支持，以及一些安全的配置选项，帮助保护Web应用程序免受攻击；
- 跨平台性：
 	Tomcat是跨平台的，可以在不同的操作系统上运行，包括Windows、Linux、和macOS等
```

| **Servlet Spec** | **JSP Spec** | **EL Spec** | **WebSocket Spec** | **Authentication Spec (JASPIC)** | **Apache Tomcat Version** | **Latest Released Version** | **Supported Java Versions** |
| :--------------- | :----------- | :---------- | :----------------- | :------------------------------- | :------------------------ | :-------------------------- | :-------------------------- |
| 6.1              | 4.0          | 6.0         | 2.2                | 3.1                              | 11.0.x                    | 11.0.2                      | 17 and later                |
| 6.0              | 3.1          | 5.0         | 2.1                | 3.0                              | 10.1.x                    | 10.1.34                     | 11 and later                |
| 4.0              | 2.3          | 3.0         | 1.1                | 1.1                              | 9.0.x                     | 9.0.98                      | 8 and later                 |

## tomcat安装

### 基于包安装Tomcat

ubuntu

```powershell
root@ubuntu-42:local # apt list |grep tomcat
root@ubuntu-42:local # apt -y install tomcat10

```

rocky

```powershell
root@rocky9-11:~ # yum list |grep tomcat
root@rocky9-11:~ # yum -y install tomcat tomcat-admin-webapps tomcat-docs-webapp tomcat-webapps
 
```

### 源码部署安装

```powershell
安装jdk java环境
root@ubuntu-43:~ # apt install openjdk-11-jdk -y
root@ubuntu-43:~ # java -version
openjdk version "11.0.25" 2024-10-15
OpenJDK Runtime Environment (build 11.0.25+9-post-Ubuntu-1ubuntu124.04)
OpenJDK 64-Bit Server VM (build 11.0.25+9-post-Ubuntu-1ubuntu124.04, mixed mode, sharing)
root@ubuntu-43:~ # vim /etc/profile.d/java.sh
root@ubuntu-43:~ # cat /etc/profile.d/java.sh
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export JAVA_BIN=$JAVA_HOME/bin
export PATH=$JAVA_BIN:$PATH
root@ubuntu-43:~ # source /etc/profile.d/java.sh 
root@ubuntu-43:~ # mkdir /data/softs -p; cd /data/softs
root@ubuntu-43:softs # wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.97/bin/apache-tomcat-9.0.97.tar.gz
root@ubuntu-43:softs # mkdir /data/server
root@ubuntu-43:softs # tar xf apache-tomcat-9.0.97.tar.gz -C /data/server/
root@ubuntu-43:softs # cd /data/server/
root@ubuntu-43:server # ln -s apache-tomcat-9.0.97/ tomcat
root@ubuntu-43:server # ls
apache-tomcat-9.0.97  tomcat
定制系统环境变量 
root@ubuntu-43:server # vim /etc/profile.d/tomcat.sh
root@ubuntu-43:server # cat /etc/profile.d/tomcat.sh 
export CATALINA_BASE=/data/server/tomcat
export CATALINA_HOME=/data/server/tomcat
export CATALINA_TMPDIR=$CATALINA_HOME/temp
export CLASSPATH=$CATALINA_HOME/bin/bootstrap.jar:$CATALINA_home/bin/tomcat-juli.jar
export PATH=$CATALINA_HOME/bin:$PATH
root@ubuntu-43:server # source /etc/profile.d/tomcat.sh 
启动tomcat
root@ubuntu-43:server # catalina.sh start
root@ubuntu-43:server # ss -tnulp | grep java
tcp   LISTEN 0      100                                  *:8080            *:*    users:(("java",pid=3973,fd=43))                       
tcp   LISTEN 0      1                   [::ffff:127.0.0.1]:8005            *:*    users:(("java",pid=3973,fd=54))  
定制服务启动脚本
root@ubuntu-43:server # catalina.sh stop  
root@ubuntu-43:server # useradd -r -s /sbin/nologin tomcat
root@ubuntu-43:server # id tomcat
uid=999(tomcat) gid=988(tomcat) groups=988(tomcat)
root@ubuntu-43:server # chown -R tomcat:tomcat /data/server/tomcat
root@ubuntu-43:server # chown -R tomcat:tomcat /data/server/apache-tomcat-10.1.33/
root@ubuntu-43:server # vim /lib/systemd/system/tomcat.service
root@ubuntu-43:server # cat /lib/systemd/system/tomcat.service
[Unit]
Description=Tomcat
After=syslog.target network.target

[Service]
Type=forking
# Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
ExecStart=/data/server/tomcat/bin/startup.sh
ExecStop=/data/server/tomcat/bin/shutdown.sh
PrivateTmp=true
User=tomcat
Group=tomcat

[Install]
WantedBy=multi-user.target
root@ubuntu-43:server # systemctl daemon-reload
root@ubuntu-43:server # systemctl start tomcat
```



## tomcat目录结构

```shell
root@ubuntu-43:server # ll /data/server/apache-tomcat-9.0.97/
bin 		# 管理脚本文件目录
conf 		# 配置文件目录
lib 		# 库文件目录
logs 		# 日志目录
temp 		# 临时文件目录
webapps 	# 应用程序目录
work 		# jsp编译后的结果文件，建议提前预热访问，升级应用后，删除此目录数据才能更新
```

```shell
root@ubuntu-43:server # ls /data/server/tomcat/bin/*sh
/data/server/tomcat/bin/catalina.sh 	# 真实生效的管理脚本
...
/data/server/tomcat/bin/shutdown.sh 	# 停止tomcat服本
/data/server/tomcat/bin/startup.sh 		# 启动tomcat脚本
...
/data/server/tomcat/bin/version.sh 		# 输出版本信息
```

tomcat 和 catalina 的关系

```
Tomcat 的核心分为三个部份
    - Web 容器：用作处理静态页面
    - JSP 容器：把 jsp 翻译成一般的 servlet
    - catalina：是一个 servlet 容器，用于处理 servlet
```

![image-20241210164113569](5day-png\19tomcat目录结构.png)

| 组件      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| server    | 服务器，Tomcat 运行的进程实例，一个Server中可以有多个service，但通常就一个 |
| service   | 服务，用来组织Engine和Connector的对应关系，一个service中只有一个Engine |
| connector | 连接器，负责客户端的HTTP、HTTPS、A JP等协议连接，一个Connector只属于某一个Engine |
| engine    | 引擎，用来响应并处理用户请求。一个Engine上可以绑定多个Connector |
| host      | 虚拟主机，可以实现多虚拟主机,例如使用不同的主机头区分        |
| context   | 应用的上下文，配置特定url路径映射和目录的映射关系：url => directory |

```
Server组件
 	是Tomcat的顶级管理组件，代表着整个Tomcat的运行实例。它定义了全局服务器配置。
Service组件：
 	是Server组件的子元素，是对外提供服务的实体。
Connector组件: 用于接收客户端请求
Engine组件: 用于响应用户请求然后转交给后面的真实服务来处理请求
```







## jpress部署-war包部署

```powershell
root@ubuntu-42:softs # mkdir -p /data/app
root@ubuntu-42:softs # ll
total 783364
drwxr-xr-x 2 root root      4096 Dec 11 11:18 ./
drwxr-xr-x 5 root root      4096 Dec 11 11:16 ../
-rw-r--r-- 1 root root 802149260 Dec 11 11:18 jpress.tar.gz
root@ubuntu-42:softs # tar xf jpress.tar.gz
root@ubuntu-42:softs # ll jpress/starter-tomcat/target/
total 125372
drwxr-xr-x 5 root root      4096 Nov 20 12:13 ./
drwxr-xr-x 4 root root      4096 Nov 20 12:13 ../
drwxr-xr-x 2 root root      4096 Nov 20 12:13 classes/
drwxr-xr-x 2 root root      4096 Nov 20 12:13 maven-archiver/
drwxr-xr-x 6 root root      4096 Nov 20 12:13 starter-tomcat-5.0/
-rw-r--r-- 1 root root      4458 Nov 20 12:13 starter-tomcat-5.0-classes.jar
-rw-r--r-- 1 root root 128350601 Nov 20 12:13 starter-tomcat-5.0.war
root@ubuntu-42:softs # cd /data/softs/jpress/starter/target/starter-5.0/
root@ubuntu-42:starter-5.0 # vim jpress.
jpress.bat  jpress.sh   
root@ubuntu-42:starter-5.0 # vim jpress.sh 
# 取消下行注释
JAVA_OPTS="-Dundertow.port=80 -Dundertow.host=0.0.0.0 -Dundertow.devMode=false"
启动
root@ubuntu-42:starter-5.0 # ./jpress.sh start
关闭
root@ubuntu-42:starter-5.0 # ./jpress.sh stop
war方式部署
root@ubuntu-42:target # mkdir /data/app/jpress/ -p
root@ubuntu-42:target # cd  /data/server/jpress/starter-tomcat/target
root@ubuntu-42:target # cp starter-tomcat-5.0.war /data/app/jpress/ROOT.war
root@ubuntu-42:target # chown -R tomcat:tomcat /data/app/jpress/
root@ubuntu-42:target # vim /data/server/tomcat/conf/server.xml
      ...  # 更改appBase的属性路径为jpress的路径
      <Host name="localhost"  appBase="/data/app/jpress"
            unpackWARs="true" autoDeploy="true">
root@ubuntu-42:target # ll /data/app/jpress/
total 125356
drwxr-xr-x 3 tomcat tomcat      4096 Dec 11 11:48 ./
drwxr-xr-x 3 root   root        4096 Dec 11 11:43 ../
drwxr-x--- 6 tomcat tomcat      4096 Dec 11 11:48 ROOT/
-rw-r--r-- 1 tomcat tomcat 128350601 Dec 11 11:44 ROOT.war
数据库配置
root@ubuntu-42:target # apt install mariadb-server -y
root@ubuntu-42:target # vim /etc/mysql/mariadb.conf.d/50-server.cnf
bind-address=0.0.0.0
root@ubuntu-42:target # systemctl restart mariadb.service 
root@ubuntu-42:target # ss -tnulp | grep 3306
tcp   LISTEN 0      80                0.0.0.0:3306      0.0.0.0:*    users:(("mariadbd",pid=17281,fd=21)) 
root@ubuntu-42:target # mysql
MariaDB [(none)]> create database jpress;
MariaDB [(none)]> create user 'jpresser'@'localhost';
MariaDB [(none)]> alter user 'jpresser'@'localhost' identified by '123456';
MariaDB [(none)]> grant all on jpress.* to jpresser@'localhost';
MariaDB [(none)]> exit

```

## halo部署-jar包部署

```powershell
root@ubuntu-42:halo # java -jar halo.jar 
```

## Nginx反向代理Tomcat

```
nginx		10.0.0.42
tomcat		10.0.0.42
tomcat		10.0.0.43
tomcat		10.0.0.11
客户端		  10.0.0.13
```

```powershell
42
root@ubuntu-42:webapps # apt install nginx -y
root@ubuntu-42:conf.d # vim /data/server/tomcat/conf/server.xml 
<Host name="ubuntu42.hzk.com"  appBase="/data/web/webapps"
root@ubuntu-42:conf.d # systemctl restart tomcat.servic
root@ubuntu-42:~ # vim /etc/nginx/conf.d/www.conf 
root@ubuntu-42:~ # cat /etc/nginx/conf.d/www.conf 
upstream tomcat {
  server 10.0.0.42:8080;
  server 10.0.0.43:8080;
  server 10.0.0.11:8080;
}
server{
  listen 80;
  server_name hzk.com;
  location / {
    proxy_pass http://tomcat;
    proxy_set_header host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
root@ubuntu-42:~ # nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@ubuntu-42:~ # systemctl restart nginx

43
root@ubuntu-43:~ # vim /data/server/tomcat/conf/server.xml
        <Host name="hzk.com"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
root@ubuntu-43:~ # vim /data/server/tomcat/webapps/ROOT/test.jsp 
Tomcat jsp page from ubuntu43<br />
SessionID = <span style="color:blue"><%=session.getId() %>


11
root@rocky9-11:~ # cat /var/lib/tomcat/webapps/ROOT/test.jsp 
Tomcat jsp page from rocky9<br />
SessionID = <span style="color:blue"><%=session.getId() %>
root@rocky9-11:~ # cat /usr/share/tomcat/conf/server.xml 
      <Host name="hzk.com"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
            
13
root@rocky9-13:~ # curl hzk.com/test.jsp
Tomcat jsp page from ubuntu42<br />
SessionID = <span style="color:blue">0A605E3537E2233F2933497B03A26E6E
root@rocky9-13:~ # curl hzk.com/test.jsp
Tomcat jsp page from rocky9<br />
SessionID = <span style="color:blue">EA3F76FC9D5232D03BF02E346FFD4B08
root@rocky9-13:~ # curl hzk.com/test.jsp
Tomcat jsp page from ubuntu43<br />
SessionID = <span style="color:blue">BF83E41337BA3C6B5077426E353B652E
```

### nginx实现https

```powershell
vim /etc/nginx/conf.d/www.conf
upstream tomcat {
    # ip_hash;
    # hash $cookie_JSESSION consistent;
 	# hash $remote_addr; # 三选一可以实现会话保持功能
 	server 10.0.0.42:8080;
 	server 10.0.0.43:8080;
 	server 10.0.0.11:8080;
}
server{
 	listen 80;
 	server_name hzk.com;
 	return 302 https://$host$request_uri;
}

server{
 	listen 443 ssl;
 	server_name hzk.com;
 	
 	ssl_certificate /usr/share/easy-rsa/pki/hzk.com.pem;
 	ssl_certificate_key /usr/share/easy-rsa/pki/private/hzk.com.key;
 	ssl_session_cache shared:sslcache:20m;
 	ssl_session_timeout 10m;
 	location ~* \.jsp$ {
 		proxy_pass http://tomcat;
 		proxy_set_header host $http_host;
 	}
 }
```



## 会话实践



### 会话复制

```powershell
root@rocky9-11:~ # vim /usr/share/tomcat/conf/server.xml
root@ubuntu-43:~ # vim /data/server/tomcat/conf/server.xml
root@ubuntu-42:~ # vim /data/server/tomcat/conf/server.xml
	<!-- 在 server.xml 中 指定域名的 Host 标签内添加下列内容 -->
		  <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                 channelSendOptions="8">

          <Manager className="org.apache.catalina.ha.session.DeltaManager"
                   expireSessionsOnShutdown="false"
                   notifyListenersOnReplication="true"/>

          <Channel className="org.apache.catalina.tribes.group.GroupChannel">
            <Membership className="org.apache.catalina.tribes.membership.McastService"
                        address="228.0.0.4"
                        port="45564"
                        frequency="500"
                        dropTime="3000"/>
            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                      address="auto"
                      port="4000"
                      autoBind="100"
                      selectorTimeout="5000"
                      maxThreads="6"/>

            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
              <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
            </Sender>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatchInterceptor"/>
          </Channel>

          <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
                 filter=""/>
          <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>

          <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                    tempDir="/tmp/war-temp/"
                    deployDir="/tmp/war-deploy/"
                    watchDir="/tmp/war-listen/"
                    watchEnabled="false"/>

          <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
        </Cluster>
        


###
root@ubuntu-42:ROOT # cat /data/web/webapps/ROOT/WEB-INF/web.xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  ...
  <!-- 在 web.xml 中 web-app 标签内添加下列内容 -->
  <distributable /> # 以启用会话复制功能。
</web-app>
```

### 会话服务器

#### Memcached

#####  Sticky 模式

![image-20241212164417441](5day-png\19memcached.png)

```powershell
自动部署tomcat9脚本
root@ubuntu-42:~ # cat tomcat_install.sh 
#!/bin/bash
# 部署java环境
java_install(){
  apt install openjdk-11-jdk -y
}

java_config(){
  cat > /etc/profile.d/java.sh <<-eof
  export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
  export JAVA_BIN=\$JAVA_HOME/bin
  export PATH=\$JAVA_BIN:\$PATH
eof
  source /etc/profile.d/java.sh
}

get_tomcat(){
  [ -d /data/softs ] || mkdir -p /data/softs
  if [ ! -f /data/softs/apache-tomcat-9.0.97.tar.gz ]; then
    cd /data/softs
    wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.97/bin/apache-tomcat9.0.97.tar.gz
  fi
}

untar_tomcat(){
  [ -d /data/server/tomcat ] && rm -rf /data/server/tomcat* || mkdir -p /data/server
  tar xf /data/softs/apache-tomcat-9.0.97.tar.gz -C /data/server/
  ln -sv /data/server/apache-tomcat-9.0.97 /data/server/tomcat
  echo "Tomcat jsp page from $(hostname)<br />" > /data/server/tomcat/webapps/ROOT/test.jsp
  echo 'SessionID = <span style="color:blue"><%=session.getId() %>' >> /data/server/tomcat/webapps/ROOT/test.jsp
  sed -i "/webapps/s/localhost/sswang.magedu.com/" /data/server/tomcat/conf/server.xml
}

tomcat_config(){
  cat > /etc/profile.d/tomcat.sh <<- eof
  export CATALINA_BASE=/data/server/tomcat
  export CATALINA_HOME=/data/server/tomcat
  export CATALINA_TMPDIR=\$CATALINA_HOME/temp
  export CLASSPATH=\$CATALINA_HOME/bin/bootstrap.jar:\$CATALINA_HOME/bin/tomcatjuli.jar
  export PATH=\$CATALINA_HOME/bin:\$PATH
eof
  source /etc/profile.d/tomcat.sh
}

tomcat_user(){
  userdel -r tomcat >/dev/null 2>&1
  useradd -r -s /sbin/nologin tomcat
  chown tomcat:tomcat -R /data/server/tomcat/*
}

tomcat_service(){
  cat > /lib/systemd/system/tomcat.service <<-eof
[Unit]
Description=Tomcat
After=syslog.target network.target
[Service]
Type=forking
# Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
ExecStart=/data/server/tomcat/bin/startup.sh
ExecStop=/data/server/tomcat/bin/shutdown.sh
PrivateTmp=true
User=tomcat
Group=tomcat
[Install]
WantedBy=multi-user.target
eof
  systemctl daemon-reload
  systemctl enable --now tomcat
}

main(){
  java_install
  java_config
  get_tomcat
  untar_tomcat
  tomcat_config
  tomcat_user
  tomcat_service
}

main
```

```
nginx								 10.0.0.42	ubuntu
Tomcat - 1，Memcached - 2 			10.0.0.43	ubuntu
Tomcat - 2，Memcached - 1 			10.0.0.11	rocky
```

1 server.xml 取消 Cluster 配置段
2 web.xml 取消 <distributable /> 配置条目

```powershell
43
root@ubuntu-43:~ # apt install memcached 
root@ubuntu-43:~ # vim /etc/memcached.conf
-l 0.0.0.0 # 修改为监听所有的IP地址
```

```powershell
11
root@rocky9-11:~ # yum -y install memcached
root@rocky9-11:~ # cat /etc/sysconfig/memcached 
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 0.0.0.0,::1"
root@rocky9-11:~ # systemctl restart memcached
```

```powershell
42
root@ubuntu-42:~ # apt install python3-pip
root@ubuntu-42:~ # apt install python3-memcache
root@ubuntu-42:~ # cat test.py 
#!/usr/bin/python3
import memcache
client = memcache.Client(['10.0.0.43:11211','10.0.0.11:11211'],debug=True)
for i in client.get_stats('items'):
    print(i)
print('-' * 35)
for i in client.get_stats('cachedump 4 0'):
    print(i)

root@ubuntu-42:~ # python3 test.py 
('10.0.0.43:11211 (1)', {})
('10.0.0.11:11211 (1)', {})
-----------------------------------
('10.0.0.43:11211 (1)', {})
('10.0.0.11:11211 (1)', {})
```

```powershell
42
root@ubuntu-42:~ # cat /etc/nginx/conf.d/www.conf 
upstream tomcat {
        # ip_hash;
        # hash $cookie_JSESSION consistent;
        # hash $remote_addr; # 三选一可以实现会话保持功能
        # server 10.0.0.42:8080;
        server 10.0.0.43:8080;
        server 10.0.0.11:8080;
}
server{
        listen 80;
        server_name hzk.com;
        location ~* \.jsp$ {
                proxy_pass http://tomcat;
                proxy_set_header host $http_host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
 }
root@ubuntu-42:~ # nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@ubuntu-42:~ # systemctl restart nginx.service 
```

```powershell
获取MSM集群依赖文件在memcached服务器上部署
mkdir /data/lib/ -p ;cd /data/lib/
wget https://repo1.maven.org/maven2/org/ow2/asm/asm/5.2/asm-5.2.jar
wget https://repo1.maven.org/maven2/com/esotericsoftware/kryo/3.0.3/kryo-3.0.3.jar
wget https://repo1.maven.org/maven2/de/javakaffee/kryo-serializers/0.45/kryo-serializers-0.45.jar
wget https://repo1.maven.org/maven2/de/javakaffee/msm/memcached-session-manager/2.3.2/memcached-session-manager-2.3.2.jar
wget https://repo1.maven.org/maven2/de/javakaffee/msm/memcached-session-manager-tc9/2.3.2/memcached-session-manager-tc9-2.3.2.jar
wget https://repo1.maven.org/maven2/com/esotericsoftware/minlog/1.3.1/minlog-1.3.1.jar
wget https://repo1.maven.org/maven2/de/javakaffee/msm/msm-kryo-serializer/2.3.2/msm-kryo-serializer-2.3.2.jar
wget https://repo1.maven.org/maven2/org/objenesis/objenesis/2.6/objenesis-2.6.jar
wget https://repo1.maven.org/maven2/com/esotericsoftware/reflectasm/1.11.9/reflectasm-1.11.9.jar
wget https://repo1.maven.org/maven2/net/spy/spymemcached/2.12.3/spymemcached-2.12.3.jar
```

```powershell
root@ubuntu-43:lib # cp *.jar /data/server/tomcat/lib/
root@rocky9-11:lib # cp *.jar /usr/share/tomcat/lib/
```

```powershell
43
ubutnu24-43的tomcat进行session共享的配置
root@ubuntu-43:lib # vim /data/server/tomcat/conf/context.xml
<Context>
...
    <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="m1:10.0.0.43:11211,m2:10.0.0.11:11211"
    failoverNodes="m2"		# 当前是 10.0.0.43先写 10.0.0.11，失败写m2
    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"

    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
    />
...
root@ubuntu-43:lib # systemctl restart tomcat
```

```powershell
11
rocky9-11的tomcat进行session共享的配置
root@rocky9-11:lib # vim /usr/share/tomcat/conf/context.xml
<Context>
...
    <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="m1:10.0.0.43:11211,m2:10.0.0.11:11211"
    failoverNodes="m1"  	# 当前是 10.0.0.11，先写 10.0.0.43，失败写m1
    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"

    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
    />
...
root@rocky9-11:lib # systemctl restart tomcat
```

```powershell
查看效果
root@ubuntu24-13:~# tail /data/server/tomcat/logs/catalina.out -n 17
27-Nov-2024 23:45:19.815 信息 [main] 
de.javakaffee.web.msm.MemcachedSessionService.startInternal --------
-  finished initialization:
- sticky: true
- operation timeout: 1000
- node ids: [m2]
- failover node ids: [m1]
- storage key prefix: null
- locking mode: null (expiration: 5s)
--------
...
root@rocky9-11:lib # tail -20 /usr/share/tomcat/logs/catalina.2024-12-12.log
12-Dec-2024 15:07:42.728 INFO [main] de.javakaffee.web.msm.MemcachedSessionService.startInternal --------
-  finished initialization:
- sticky: true
- operation timeout: 1000
- node ids: [m1]
- failover node ids: [m2]
- storage key prefix: null
- locking mode: null (expiration: 5s)
--------
```

```powershell
浏览器访问，查看效果
root@ubuntu-42:~ # python3 test.py
('10.0.0.43:11211 (1)', {'items:4:number': '1', 'items:4:number_hot': '0', 'items:4:number_warm': '0', 'items:4:number_cold': '1', 'items:4:age_hot': '0', 'items:4:age_warm': '0', 'items:4:age': '125', 'items:4:mem_requested': '183', 'items:4:evicted': '0', 'items:4:evicted_nonzero': '0', 'items:4:evicted_time': '0', 'items:4:outofmemory': '0', 'items:4:tailrepairs': '0', 'items:4:reclaimed': '0', 'items:4:expired_unfetched': '0', 'items:4:evicted_unfetched': '0', 'items:4:evicted_active': '0', 'items:4:crawler_reclaimed': '0', 'items:4:crawler_items_checked': '3', 'items:4:lrutail_reflocked': '100', 'items:4:moves_to_cold': '42', 'items:4:moves_to_warm': '41', 'items:4:moves_within_lru': '2', 'items:4:direct_reclaims': '0', 'items:4:hits_to_hot': '0', 'items:4:hits_to_warm': '2', 'items:4:hits_to_cold': '43', 'items:4:hits_to_temp': '0'})
('10.0.0.11:11211 (1)', {})
-----------------------------------
('10.0.0.43:11211 (1)', {'3929B358826D806717371DA20A878007-m1': '[89 b; 1733989340 s]'})
('10.0.0.11:11211 (1)', {})
```

```powershell
mv /usr/share/tomcat/lib/jedis-3.0.0.jar /tmp/
mv /usr/share/tomcat/lib/redisson-all-3.27.2.jar /tmp/
mv /usr/share/tomcat/lib/redisson-tomcat-9-3.27.2.jar /tmp/
rm -rf /usr/share/tomcat/work/*
rm -rf /usr/share/tomcat/temp/*
systemctl restart tomcat
ls -l /usr/share/tomcat/lib/reflectasm*.jar
```



##### 非 Sticky 模式

![image-20241212164602445](5day-png\19memcached非sticky.png)

```powershell
43
root@ubuntu-43:~ # vim /data/server/tomcat/conf/context.xml 
    <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="m1:10.0.0.43:11211,m2:10.0.0.11:11211"
    sticky="false"
    sessionBackupAsync="false"
    lockingMode="uriPattern:/path1|/path2"

    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"

    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
    />
root@ubuntu-43:~ # systemctl start memcached.service 
```

```powershell
42
root@rocky9-11:lib # vim /usr/share/tomcat/conf/context.xml 
    <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="m1:10.0.0.43:11211,m2:10.0.0.11:11211"
    sticky="false"
    sessionBackupAsync="false"
    lockingMode="uriPattern:/path1|/path2"

    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"

    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
    />
root@rocky9-11:lib # systemctl restart memcached
```

```powershell
root@ubuntu-42:~ # python3 test.py
-----------------------------------
('10.0.0.43:11211 (1)', {'bak:2EE18A82036B6EB917EDC29CB3C6823F-m2': '[89 b; 1733997164 s]'})
('10.0.0.11:11211 (1)', {'2EE18A82036B6EB917EDC29CB3C6823F-m2': '[89 b; 1733997164 s]'})
```

#### 基于 MSM 实现 Redis Session 共享

![image-20241212165458028](5day-png\19MSM 实现 Redis Session 共享.png)

```powershell
nginx		10.0.0.42	ubuntu
tomcat		10.0.0.43	ubuntu
tomcat		10.0.0.11	rocky
redis		10.0.0.13	rocky
```

```powershell
13
root@rocky9-13:~ # yum -y install redis
root@rocky9-13:~ # vim /etc/redis/redis.conf
# bind 127.0.0.1 -::1
bind 0.0.0.0
root@rocky9-13:~ # systemctl restart redis
root@rocky9-13:~ # ss -tnulp | grep redis
tcp   LISTEN 0      511                             0.0.0.0:6379       0.0.0.0:*    users:(("redis-server",pid=9152,fd=6)) 
root@rocky9-13:~ # redis-cli keys "*"
(empty array)
```

```powershell
43
root@ubuntu-43:~ # cd /data/softs/
root@ubuntu-43:softs #  wget https://repo1.maven.org/maven2/redis/clients/jedis/3.0.0/jedis-3.0.0.jar
root@ubuntu-43:softs # cp jedis-3.0.0.jar /data/server/tomcat/lib/
root@ubuntu-43:softs # vim /data/server/tomcat/conf/context.xml
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="redis://10.0.0.13"
    sticky="false"
    sessionBackupAsync="false"
    lockingMode="uriPattern:/path1|/path2"
    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"
    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
    />
root@ubuntu-43:softs # systemctl restart tomcat.service 
```

```powershell
11
root@rocky9-11:lib # cd /data/softs/
root@rocky9-11:softs # wget https://repo1.maven.org/maven2/redis/clients/jedis/3.0.0/jedis-3.0.0.jar
root@rocky9-11:softs # cp jedis-3.0.0.jar /usr/share/tomcat/lib/
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="redis://10.0.0.13"
    sticky="false"
    sessionBackupAsync="false"
    lockingMode="uriPattern:/path1|/path2"
    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"
    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
    />
root@rocky9-11:softs # systemctl restart tomcat
```

浏览器访问

```powershell
13查看
root@rocky9-13:softs # redis-cli keys "*"
1) "validity:9B4BAEB3A8E4C6E50C8C15DCC360C7BF"
2) "9B4BAEB3A8E4C6E50C8C15DCC360C7BF"
```

#### 基于 Redisson 实现 Session 共享

![image-20241212171330540](5day-png\19基于 Redisson 实现 Session 共享.png)

```
nginx		10.0.0.42	ubuntu
tomcat		10.0.0.43	ubuntu
tomcat		10.0.0.11	rocky
redis		10.0.0.13	rocky
```

```powershell
环境还原
root@ubuntu-43:softs # systemctl restart tomcat.service 
root@rocky9-11:softs # systemctl restart tomcat.service
root@rocky9-13:softs # redis-cli flushall
OK
root@rocky9-13:softs # redis-cli keys "*"
(empty array)
```

```powershell
43
root@ubuntu-43:softs # cd /data/softs/
root@ubuntu-43:softs # wget https://repo1.maven.org/maven2/org/redisson/redisson-all/3.27.2/redisson-all-3.27.2.jar
root@ubuntu-43:softs # wget https://repo1.maven.org/maven2/org/redisson/redisson-tomcat-9/3.27.2/redisson-tomcat-9-3.27.2.jar
root@ubuntu-43:softs # cp redisson-* /data/server/tomcat/lib/
root@ubuntu-43:softs # vim /data/server/tomcat/conf/context.xml 
    <Manager className="org.redisson.tomcat.RedissonSessionManager"
             configPath="/data/server/tomcat/conf/redisson.conf"
             readMode="REDIS" updateMode="DEFAULT"
             broadcastSessionEvents="false"
             keyPrefix=""
    />
root@ubuntu-43:softs # vim /data/server/tomcat/conf/redisson.conf
singleServerConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  address: "redis://10.0.0.13:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 24
  connectionPoolSize: 64
  database: 0
  dnsMonitoringInterval: 5000
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
root@ubuntu-43:softs # chown -R tomcat:tomcat /data/server/tomcat/conf/redisson.conf 
root@ubuntu-43:softs # ll /data/server/tomcat/conf/redisson.conf 
-rw-r--r-- 1 tomcat tomcat 511 Dec 12 09:27 /data/server/tomcat/conf/redisson.conf
root@ubuntu-43:softs # systemctl restart tomcat.service
```

```powershell
11
root@rocky9-11:softs # cd /data/softs/
root@rocky9-11:softs # wget https://repo1.maven.org/maven2/org/redisson/redisson-all/3.27.2/redisson-all-3.27.2.jar
root@rocky9-11:softs # wget https://repo1.maven.org/maven2/org/redisson/redisson-tomcat-9/3.27.2/redisson-tomcat-9-3.27.2.jar
root@rocky9-11:softs # cp redisson-* /usr/share/tomcat/lib/
root@rocky9-11:softs # vim /usr/share/tomcat/conf/context.xml 
    <Manager className="org.redisson.tomcat.RedissonSessionManager"
             configPath="/usr/share/tomcat/conf/redisson.conf"
             readMode="REDIS" updateMode="DEFAULT"
             broadcastSessionEvents="false"
             keyPrefix=""
    />
root@rocky9-11:softs # vim /usr/share/tomcat/conf/redisson.conf
singleServerConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  address: "redis://10.0.0.13:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 24
  connectionPoolSize: 64
  database: 0
  dnsMonitoringInterval: 5000
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
root@rocky9-11:softs # chown -R tomcat:tomcat /usr/share/tomcat/conf/redisson.conf
root@rocky9-11:softs # systemctl restart tomcat
```

```powershell
浏览器访问测试
root@rocky9-13:~ # redis-cli flushall
OK
root@rocky9-13:~ # redis-cli keys "*"
(empty array)
root@rocky9-13:~ # redis-cli keys "*"
1) "redisson:tomcat_session:C4B931C5BAC47897FF85BD53D094D797"
```



## JVM

![image-20241214091944524](5day-png\jvmjava.png)

![image-20241214091849075](5day-png\19jvm.png)

线程共享

```
Method Area：
 	方法区是所有线程共享的内存空间，存放已加载的类信息(构造方法，接口定义)，常量(final)，静态变量(static)， 运行时常量池等。但实例变量存放在堆内存中. 从JDK8开始此空间由永久代改名为元空间metaspace
```

```
heap：
 	堆在虚拟机启动时创建，存放创建的所有对象信息。如果对象无法申请到可用内存将抛出OOM异常，堆是靠GC垃圾回收器管理的，通过-Xmx -Xms 指定最大堆和最小堆空间大小

```

线程私有

```
Java stack：
 	Java栈是每个线程会分配一个栈，存放java中8大基本数据类型，对象引用，实例的本地变量，方法参数和返回值等，基于FILO（First In Last Out）,每个方法为一个栈帧
```

```
Program Counter Register：
 	PC寄存器就是一个指针，指向方法区中的方法字节码，每一个线程用于记录当前线程正在执行的字节码指令地址。由执行引擎读取下一条指令，因为线程需要切换，当一个线程被切换回来需要执行的时候，知道执行到哪里了
```

```
Native Method stack ：
 本地方法栈为本地方法执行构建的内存空间，存放本地方法执行时的局部变量、操作数等
```

### 垃圾回收

```
- 哪些是垃圾要回收
- 怎么回收垃圾
- 什么时候回收垃圾
```

Garbage 垃圾确定方法

```
一般通过 引用计数和根搜索算法来进行确定

引用计数：
 	每一个堆内对象上都添加一个私有引用计数器，记录着被引用的次数，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1。任何时刻计数器为0的对象，就是垃圾对象。当引用计数清零，该对象所占用堆内存就可以被回收。
 	尽管实现简单且效率高，但循环引用的对象，无法将引用计数归零，那么也就无法被清除。因此，Java虚拟机（JVM）中并不常用此算法。
 	
根搜索 (可达) 算法：Root Searching
 	这是Java虚拟机的主流垃圾对象识别方法。
 	它通过一系列的称为“GC Roots”的对象作为起始点，这些起始点包括虚拟机栈中引用的对象、方法区中的类静态属性引用的对象、本地方法栈中JNI引用的对象等。
 	从GC Roots开始，通过对象之间的引用关系向下搜索，能够到达的对象被认为是“可达对象”，否则就是“垃圾对象”。
 	能够有效解决循环引用的问题，并且可以准确地识别出不再被使用的对象。

```

![image-20241214094425811](5day-png\19垃圾回收机制根可达.png)

### GC算法

**GC算法-标记清除**

![image-20241214094608174](5day-png\19GC标记清楚.png)

```
算法简单，不会浪费内存空间，效率较高，但会形成内存碎片
```

**GC算法-标记压缩**

![image-20241214094700438](5day-png\19GC压缩清楚.png)

**GC算法-复制**

![image-20241214094752095](5day-png\19GC复制.png)

```
好处是没有碎片，复制过程中保证对象使用连续空间，且一次性清除所有垃圾，所以即使对象很多，收回效率也很高，缺点是比较浪费内存，只能使用原来一半内存，因为内存对半划分了，复制过程毕竟也是有代价。
```

![image-20241214094845875](5day-png\19GC汇总.png)

```
没有最好的算法，在不同场景选择最合适的算法
- 效率：复制算法>标记清除算法> 标记压缩算法
- 内存整齐度：复制算法=标记压缩算法> 标记清除算法
- 内存利用率：标记压缩算法=标记清除算法>复制算法
```

```
对于大多数垃圾回收算法而言，GC线程工作时，停止所有工作的线程，进行内存整理工作，称为Stop The World（STW）。GC 完成时，内存整理完毕，开始恢复其他工作线程运行。这也是 JVM 运行中最头疼的问题
```

###  内存GC策略

![image-20241214095137343](5day-png\19内存清理策略.png)

```
年轻代（Young Generation）：
 	用于存放新创建的对象。它通常被进一步划分为Eden伊甸园区和两个Survivor幸存区（From和To），2个幸存区大小相等、地位相同、可互换，通过Minor GC（新生代垃圾回收）来清理不再使用的对象。

老年代（Old Generation）：
 	用于存放经过多次Minor GC后仍然存活的对象。这些对象通常是比较稳定的，不会被频繁回收。
 	Major GC（老年代垃圾回收）会清理年老代中不再使用的对象。

永久代/元空间（PermGen/Metaspace）：
 	用于存放保存 JVM 自身的类的元数据信息、常量池、存储 JAVA 运行时的环境信息等。在Java 8及以后的版本中，永久代被MetaSpace元空间所替代。
 	元空间位于本地内存中，而不是堆内存中。关闭JVM会释放此区域内存，所以此空间不存在垃圾回收。
 	元空间在物理上不属于heap内存，但逻辑上归属于heap内存。
```

```
默认JVM试图分配最大内存的总内存的1/4，初始化默认总内存为总内存的1/64，年轻代中heap的1/3，老年代占2/3
```

### 年轻代回收 Minor GC

```
1. 起始时，所有新建对象(特大对象直接进入老年代)都出生在eden，当eden满了，启动GC。这个称为Young GC 或者 Minor GC
2. 先标记eden存活对象，然后将存活对象复制到s0（假设本次是s0，也可以是s1，它们可以调换），eden剩余所有空间都清空。GC完成
```

![image-20241214095951288](5day-png\19年轻代GC回收.png)

```
3. 继续新建对象，当eden再次满了，启动GC
4. 先同时标记eden和s0中存活对象，然后将存活对象复制到s1。将eden和s0清空，此次GC完成
```

![image-20241214100028975](5day-png\19年轻代GC回收2.png)

```
5. 继续新建对象，当eden满了，启动GC
6. 先标记eden和s1中存活对象，然后将存活对象复制到s0。将eden和s1清空,此次GC完成
7. 以后就重复上面的步骤
```

![image-20241214100111958](5day-png\19年轻代GC回收3.png)

### 老年代回收 Major GC

![image-20241214100236413](5day-png\19老年代GC回收.png)

```
	进入老年代的数据较少，所以老年代区被占满的速度较慢，所以垃圾回收也不频繁。如果老年代也满了，会触发老年代GC，称为Old GC或者 Major GC。
 	由于老年代对象一般来说存活次数较长，所以较常采用标记-压缩算法。
 	由于老年代对象也可以引用新生代对象，所以先进行一次Minor GC，然后在Major GC会提高效率。所以，我们一般认为，回收老年代的时候完成了一次FullGC。
 	所以，一般情况下，我们可以认为 MajorGC = FullGC
```

### GC 触发条件

![image-20241214100402881](5day-png\19GC触发条件.png)

```
年轻代和老年代的GC
	Minor GC 触发条件：当eden区满了触发
	Full GC 触发条件：老年代满了；System.gc()手动调用（不推荐）
```

```
年轻代：存活时长低；适合复制算法
老年代：区域大，存活时长高；适合标记压缩算法
```

```
	Minor GC 可能会引起短暂的STW暂停。当进行 Minor GC 时，为了确保安全性，JVM 需要在某些特定的点上暂停所有应用程序的线程，以便更新一些关键的数据结构。这些暂停通常是非常短暂的，通常在毫秒级别，并且很少对应用程序的性能产生显著影响。
 	Major GC的暂停时间通常会比Minor GC的暂停时间更长，因为老年代的容量通常比年轻代大得多。这意味着在收集和整理大量内存时，需要更多的时间来完成垃圾收集操作
 	尽管Major GC会引起较长的STW暂停，但JVM通常会尽量优化垃圾收集器的性能，以减少这些暂停对应用程序的影响。例如，通过使用并行或并发垃圾收集算法，可以减少STW时间，并允许一部分垃圾收集工作与应用程序的线程并发执行。
```

### tomcat的java属性参数调整

![](5day-png\19tomcat的java属性参数调整.png)

![image-20241214111213231](5day-png\19堆内存设置相关参数.png)

```powershell
root@ubuntu43:~ # vim /data/server/tomcat/bin/catalina.sh
# -----------------------------------------------------------------------------
# 只要是头部增加即可
CATALINA_OPTS='-Xms1g -Xmx1g -XX:NewRatio=2 -XX:SurvivorRatio=6'
#添加此行，使用 JAVA_OPTS 也可以
#堆内存初始值1G，最大值1G，年轻代和老年代内存比例为 1:2，eden区和幸存区的内存比例为6:1:1
# OS specific support
root@ubuntu43:~ # systemctl restart tomcat.service 
```

## jvisualvm 监控内存

```powershell
安装依赖
root@ubuntu43:~ # apt install libxrender1 libxrender1 libxtst6 libxi6 fontconfig -y
root@ubuntu43:softs #  wget https://github.com/oracle/visualvm/releases/download/2.1.8/visualvm_218.zip
root@ubuntu43:softs # unzip visualvm_218.zip
root@ubuntu43:softs # cd ../apps/
root@ubuntu43:apps # export DISPLAY=10.0.0.1:0.0
root@ubuntu43:apps # ./visualvm_218/bin/visualvm
```



## 垃圾收集方式

```
	垃圾收集（Garbage Collection，GC）是Java等编程语言中用于自动管理内存的一种机制。它通过回收不再使用的对象所占用的内存，以避免内存泄漏和确保程序的稳定运行。
 	针对这些垃圾回收方式的实施特点，我们可以将其划分为不同的类型方式。
```

```
按工作模式不同：指的是GC线程和工作线程是否一起运行
独占垃圾回收器：
 	只要GC在工作，STW 一直进行到回收完毕，工作线程才能继续执行
并发垃圾回收器：
 	让GC线程垃圾回收的某些阶段 可以和 工作线程一起进行，
 	如：标记阶段并行，回收阶段仍然串行
```

```
按回收线程数：指的是GC线程是否串行或并行执行
串行垃圾回收器：
 	一个GC线程完成内存资源的回收工作
并行垃圾回收器：
 	多个GC线程同时一起完成内存资源的回收工作，充分利用CPU的多核并发执行的计算资源
```

![image-20241214105356436](5day-png\19垃圾回收策略串行和并行.png)

```
- 减少 STW 时长，串行变并行
- 减少 GC 次数，要分配合适的内存大小，这样的话，可以实现数据填充的速度慢一些，从而减少GC次数。
```

### 新生代垃圾回收器

![image-20241214105551990](5day-png\19新生代垃圾回收器.png)

```
新生代串行收集器Serial：
	单线程、独占式串行，采用复制算法0，简单高效但会造成STW

新生代并行回收收集器PS(Parallel Scavenge)：---吞吐量
多线程并行、独占式，会产生STW, 使用复制算法关注调整吞吐量，此收集器关注点是达到一个可控制的吞吐
量：
    - 吞吐量 = 运行用户代码时间/（运行用户代码时间+垃圾收集时间），
   		比如虚拟机总共运行100分钟，其中垃圾回收花掉1分钟，那吞吐量就是99%。
    - 高吞吐量可以高效率利用CPU时间，尽快完成运算任务，
   		主要适合在后台运算而不需要太多交互的任务。
    - 除此之外，Parallel Scavenge 收集器具有自适应调节策略，
   		它可以将内存管理的调优任务交给虚拟机去完成。
   		自适应调节策略也是Parallel Scavenge与 ParNew 收集器的一个重要区别。
注意：此为默认的新生代的垃圾回收器

新生代并行收集器ParNew：---用户交互
	Serial 收集器的多线程版，将单线程的串行收集器变成了多线程并行、独占式，使用复制算法，
 	- 相当于PS的改进版,但是与PS不一样的在于，ParNew可以和老年代的CMS组合使用。
```

### 老年代垃圾回收器

```
老年代串行收集器Serial Old
	Serial Old是Serial收集器的老年代版本,单线程、独占式串行，回收算法使用标记压缩‘
老年代并行回收收集器Parallel Old
	多线程、独占式并行，回收算法使用标记压缩，关注调整吞吐量。它是默认的新老年代的垃圾回收器。
		- Parallel Old收集器是Parallel Scavenge 收集器的老年代版本
 			这个收集器是JDK1.6之后才开始提供。
注意：
 Parallel Scavenge 收集器无法与CMS收集器配合工作,因为一个是为了吞吐量，一个是为了客户体验（也就是暂停时间的缩短）
```

```
JVM 1.8 默认的垃圾回收器：PS + ParallelOld，所以大多数都是针对此进行调优
```

####  CMS收集器

![image-20241214122145273](5day-png\19CMS收集器.png)

```
初始标记：
 	此过程需要STW（Stop The Word），只标记一下GC Roots能直接关联到的对象，速度很快。
 
并发标记：
 	就是GC Roots进行扫描可达链的过程，为了找出哪些对象需要收集。
 	这个过程远远慢于初始标记，但它是和用户线程一起运行的，不会出现STW，所有用户并不会感受到。
重新标记：
 	为了修正在并发标记期间，用户线程产生的垃圾，
 	这个过程会比初始标记时间稍微长一点，但是也很快，和初始标记一样会产生STW。
并发清理：
 	在重新标记之后，清理删除标记阶段已被判处死亡的对象，由于基于清除算法，不需要移动存活对象。
 	和并发标记一样也是和用户线程一起运行的，耗时较长（和初始标记比的话），不会出现STW。
 
 	由于整个过程中，耗时最长的并发标记和并发清理都是与用户线程一起执行的，所以总体上来说，CMS收集器的内存回收过程是与用户线程一起并发执行的

```

### 无代限制的收集器

```
- 基于标记压缩算法，不会产生大量的空间碎片，有利于程序的长期执行
- 分为4个阶段：
 	初始标记、并发标记、最终标记、筛选回收。并发标记并发执行，其它阶段STW只有GC线程并行执行
- G1收集器是面向服务端的收集器，
 	它的思想就是首先回收尽可能多的垃圾（这也是Garbage-First名字的由来）
- G1能充分的利用多CPU，
 	多核环境下的硬件优势，使用多个CPU来缩短STW停顿的时间(10ms以内)
- 可预测的停顿：
 	这是G1相对于CMS的另一大优势，G1和CMS一样都是关注于降低停顿时间，但是G1能够让使用者明确的指定在一个M毫秒的时间片段内，消耗在垃圾收集的时间不得超过N毫秒
- 通过此选项指定：+UseG1GC

```

## JVM相关工具

### openjdk

```powershell
root@ubuntu43:~ # ll /usr/bin/java
lrwxrwxrwx 1 root root 22 Oct 17 17:21 /usr/bin/java -> /etc/alternatives/java*
root@ubuntu43:~ # ll /etc/alternatives/java
lrwxrwxrwx 1 root root 43 Oct 17 17:21 /etc/alternatives/java -> /usr/lib/jvm/java-11-openjdk-amd64/bin/java*
root@ubuntu43:~ # ll /usr/lib/jvm/java-11-openjdk-amd64/bin/java
-rwxr-xr-x 1 root root 14528 Oct 17 17:21 /usr/lib/jvm/java-11-openjdk-amd64/bin/java*
root@ubuntu43:~ # ll /usr/lib/jvm/java-11-openjdk-amd64/bin
 	jconsole 		#图形工具
 	jinfo 			#查看进程的运行环境参数，主要是jvm命令行参数
    jmap 			#查看jvm占用物理内存的状态
    jps 			#查看所有jvm进程
    jstack 			#查看所有线程的运行状态
    jstat 			#对jvm应用程序的资源和性能进行实时监控
```

### Oracle JDK

```powershell
root@ubuntu43:softs # tar xf jdk-11.0.16_linux-x64_bin.tar.gz -C ../server/
root@ubuntu43:softs # cd ../server/
root@ubuntu43:server # ls
apache-tomcat-9.0.97  jdk-11.0.16  tomcat
root@ubuntu43:server # ln -s /data/server/jdk-11.0.16 /data/server/java
root@ubuntu43:server # ls
apache-tomcat-9.0.97  java  jdk-11.0.16  tomcat
root@ubuntu43:server # vim /etc/profile.d/java.sh
root@ubuntu43:server # . /etc/profile.d/java.sh 
root@ubuntu43:server # java -version
java version "11.0.16" 2022-07-19 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.16+11-LTS-199)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.16+11-LTS-199, mixed mode)
```

## 故障代码定位

```powershell
root@ubuntu43:~ # cat CPUIntensiveTask.java 
public class CPUIntensiveTask{
  public static void main(String[]args){
    int numberOfThreads = Runtime.getRuntime().availableProcessors();//获取可用的处理器核心数
    for(int i=0; i < numberOfThreads; i++){
      Thread thread = new Thread(new IntensiveTask());
      thread.start();
    }
  }
  static class IntensiveTask  implements Runnable {
    @Override
    public void run(){
      boolean flag = true;
      while(flag){
        ;
      }
      System.out.println("Thread " + Thread.currentThread().getId() + " completed.");
    }
  }
}
root@ubuntu43:~ # javac CPUIntensiveTask.java
root@ubuntu43:~ # ls CPUIntensiveTask*
CPUIntensiveTask.class  CPUIntensiveTask.java
root@ubuntu43:~ # java CPUIntensiveTask.java
root@ubuntu43:~ # ps aux | grep java
root        3420  199  0.7 3536224 29344 pts/0   Sl+  21:31   2:02 java CPUIntensiveTask
root@ubuntu43:~ # pstree -p 3420
root@ubuntu43:~ # top -H -p 3420
root@ubuntu43:~ # printf '0x%x\n' 3437
0xe07
root@ubuntu43:~ # jstack -l 3420 | grep -A10 0xe07
"Thread-0" #10 prio=5 os_prio=0 cpu=67495.02ms elapsed=68.20s 
tid=0x00007f8d241a5800 nid=0xe07 runnable [0x00007f8d0c2fb000]
   java.lang.Thread.State: RUNNABLE
        at CPUIntensiveTask$IntensiveTask.run(CPUIntensiveTask.java:15)
        at java.lang.Thread.run(java.base@11.0.16/Thread.java:834)
   Locked ownable synchronizers:
        - None
"Thread-1" #11 prio=5 os_prio=0 cpu=67366.13ms elapsed=68.20s 
tid=0x00007f8d241a7800 nid=0xe08 runnable [0x00007f8d0c1fb000]
   java.lang.Thread.State: RUNNABLE
        at CPUIntensiveTask$IntensiveTask.run(CPUIntensiveTask.java:15)
当然也可以将所有的过程记录，放到一个文件中，等待后续查看
root@ubuntu43:~ # jstack -l 3420 > stack.log
定位出15行问题
```



## JConsole图形化工具

**jconsole.exe工具**

### tomcat启用jmx监控

```powershell
root@ubuntu43:~ # vim /data/server/tomcat/bin/catalina.sh 
# -----------------------------------------------------------------------------
CATALINA_OPTS="$CATALINA__OPTS \
-Dcom.sun.management.jmxremote \
-Djava.rmi.server.hostname=10.0.0.43 \
-Dcom.sun.management.jmxremote.port=12345 \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.ssl=false"
# OS specific support. $var _must_ be set to either true or false.
```

### Tomcat 性能优化常用配置

```powershell
内存空间优化
root@ubuntu43:~ # vim /data/server/tomcat/bin/catalina.sh
JAVA_OPTS="-server -Xms4g -Xmx4g -Xss512k -Xmn1g -XX:CMSInitiatingOccupancyFraction=65 -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:+DisableExplicitGC -XX:MaxTenuringThreshold=10 -XX:NewRatio=2 -XX:PermSize=128m -XX:MaxPermSize=512m -XX:CMSFullGCsBeforeCompaction=5 -XX:+ExplicitGCInvokesConcurrent -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods"
线程池调整
root@ubuntu43:~ # vim /data/server/tomcat/conf/server.xml
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="2000"
               />
```

## java程序编译

![image-20241214150856871](5day-png\19程序编译.png)

![image-20241214150927139](5day-png\19javeJSP.png)

```
java项目编译流程
1. 获取 Java 程序源码
2. 进入源码目录，运行 mvn 命令，mvn 会根据项目中的 pom.xml 文件和编译参数编译源码，得到 .war  或 .jar 结尾的目标文件
3. 部署运行
```

```powershell
https://github.com/apache/dubbo-admin
获取源代码:
 	git clone https://github.com/apache/dubbo-admin.git
更改属性：
 	dubbo-admin-server/src/main/resources/application.properties
构建
 	mvn clean package -Dmaven.test.skip=true
启动项目
 	mvn --projects dubbo-admin-server spring-boot:run or
 	cd dubbo-admin-distribution/target; java -jar dubboadmin-${project.version}.jar
确认效果
 	访问 http://localhost:38080
    默认的 username 和 password 都是 root
```

```powershell
root@ubuntu43:softs # git clone https://github.com/apache/dubbo-admin.git
查看dubbo-admin-develop项目的maven仓库
root@ubuntu43:softs # cat dubbo-admin/pom.xml
   <repositories>
        <repository>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
            <id>apache.snapshots.https</id>
            <name>Apache Development Snapshot Repository</name>
            <url>https://repository.apache.org/content/repositories/snapshots</url>
        </repository>
    </repositories>
```

## maven仓库使用流程

![image-20241214154611058](5day-png\19maven仓库使用流程.png)

```
1. 首先，在本地repo搜索指定的依赖，而后是Maven的repo服务
2. 若配置了远程repo服务，Maven还是搜索该repo
3. Maven默认使用的是其官方存储库，在国内的下载速度较慢，因而建议使用国内的Maven镜像仓库，例如阿里的镜像仓库等
```

```
本地库说明
- Maven 的本地仓库存储了 maven 需要的插件和你项目中需要用到的依赖 jar 包
 	当 maven 使用到上述内容的时候，如果本地仓库中已经存在了，则直接使用本地仓库中内容，
 	如果本地仓库中不存在，则需要从中央仓库下载到本地然后才可以继续使用，
 	因此 maven 的本地仓库会存储很多的文件
- Maven的默认配置的本地仓库路径 ： ${HOME}/.m2/repository
- 可以在maven的配置文件 settings.xml 文 中指定本地仓库路径
```

```
Maven 的资源坐标
- Groupld：组织ID，一般是反写的公司域名com.example ,同一个公司的Groupld都是相同的
- Artifactld：制器ID，一般是项目名，如myapp，也是生成包jar/war的名
- Version：版本，比如：1.2.3
```

安装maven前必须安装java 环境

```
- Maven 3.3 要求 JDK 1.7 或以上
- Maven 3.2 要求 JDK 1.6 或以上
- Maven 3.0/3.1 要求 JDK 1.5 或以上
注意：
 Maven 所安装 JDK 的版本不影响编译应用的所依赖的 JDK 版本，比如 Maven 安装 JDK11，而java 应用 Jpress 可以使用 JDK8
```

## apt/yum安装maven

```powershell
apt list maven
apt install maven -y
root@ubuntu-42:~ # which mvn
/usr/bin/mvn
root@ubuntu-42:~ # mvn --version
Apache Maven 3.8.7
Maven home: /usr/share/maven
Java version: 21.0.5, vendor: Ubuntu, runtime: /usr/lib/jvm/java-21-openjdk-amd64
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "6.8.0-47-generic", arch: "amd64", family: "unix"
改为国内源
root@ubuntu-42:~ # vim /etc/maven/settings.xml
    <mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>https://maven.aliyun.com/repository/public</url>
      <blocked>true</blocked>
    </mirror>
  </mirrors>
```

二进制安装maven

```powershell
root@ubuntu43:softs # wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.zip
root@ubuntu43:softs # unzip apache-maven-3.9.6-bin.zip 
root@ubuntu43:softs # mv apache-maven-3.9.6 ../server/
root@ubuntu43:softs # cd ../server/
root@ubuntu43:server # ls maven/bin/
m2.conf  mvn  mvn.cmd  mvnDebug  mvnDebug.cmd  mvnyjp
root@ubuntu43:server # maven/bin/mvn --version
Apache Maven 3.9.6 (bc0240f3c744dd6b6ec2920b3cd08dcc295161ae)
Maven home: /data/server/maven
Java version: 11.0.16, vendor: Oracle Corporation, runtime: /data/server/jdk-11.0.16
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "6.8.0-49-generic", arch: "amd64", family: "unix"
加PATH
root@ubuntu43:server # ln -s /data/server/maven/bin/mvn /usr/local/bin/
root@ubuntu43:server # vim maven/conf/settings.xml
    <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>central</mirrorOf>
      <name>阿里云公共仓库</name>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
		
```

```powershell
root@ubuntu43:~ # tree /data/server/maven/ -L 1
/data/server/maven/
├── bin
├── boot
├── conf
├── lib
├── LICENSE
├── NOTICE
└── README.txt
root@ubuntu43:~ # ll /data/server/maven/conf/
-rw-r--r-- 1 root root 10736 Dec 14 08:04 settings.xml

settings.xml
全局 settings.xml 是 maven 的全局配置文件，一般位于
${maven.home}/conf/settings.xml，即 maven 文件夹下的 conf 中。

settings.xml
用户 setting 是 maven 的用户配置文件，一般位于 ${user.home}/.m2/settings.xml，即每位用户都有一份配置文件。
默认情况下，该目录是不存在的。

pom.xml
pom.xml 文件是项目配置文件，一般位于项目根目录下或子目录下

配置优先级
- 配置优先级从高到低：pom.xml > 本地 settings > 全局 settings
- 如果这些文件同时存在，在应用配置时，会合并它们的内容，如果有重复的配置，优先级高的配置会覆盖优先级低的。
```

```
settings.xml 用来配置 maven 项目中的各种参数文件，包括本地仓库、远程仓库、私服、认证等信息
/data/server/maven/conf/settings.xml	源码安装
/etc/maven/settings.xml					包安装
```

##  Maven 命令

```powershell
clean 					#清理之前己构建和编译的产物
compile 				#将源码编译成class字节码文件
test 					#行项目的单元测试
package 				#将项目打包成 war 包 或 jar 包
install 				#将项目打包并安装到本地 Maven 仓库，以便其他项目可以使用
deploy 					#将项目发布到远程仓库，通常是用于发布到公司或公共 Maven仓库
dependency:resolve 		#解析并下载项目依赖
dependency:tree 		#显示项目依赖树
clean install 			#先清理，再编译安装
```

命令示例

```
清理，打包，并安装
mvn clean install package
清理，打包，并安装，跳过单元测试部份
mvn clean install package -Dmaven.test.skip=true

4线程编译
mvn -T 4 clean install package -Dmaven.test.skip=true
2倍CPU核心线程数编译
mvn -T 2C clean install package -Dmaven.test.skip=true

多线程编译
mvn clean install package -Dmaven.test.skip=true -Dmaven.compile.fork=true
构建docker镜像，要在pom.xml中配置
mvn -f mall-gateway clean package dockerfile:build
```

## RuoYi项目编译案例 **-** 代码编译

```powershell
root@ubuntu43:codes # apt update;apt install mysql-server -y
root@ubuntu43:codes # vim /etc/mysql/mysql.conf.d/mysqld.cnf 
#bind-address = 127.0.0.1
#mysqlx-bind-address = 127.0.0.1
root@ubuntu43:codes # systemctl restart mysql.service
root@ubuntu43:codes # mysql
mysql> create database ruoyi;
Query OK, 1 row affected (0.00 sec)

mysql> create user ruoyier@'10.0.0.%' identified with mysql_native_password by '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> grant all on ruoyi.* to ruoyier@'10.0.0.%';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

root@ubuntu43:codes # git clone https://gitee.com/y_project/RuoYi.git
root@ubuntu43:codes # vim RuoYi/ruoyi-admin/src/main/resources/application-druid.yml
            # 主库数据源
            master:
                url: jdbc:mysql://10.0.0.43:3306/ruoyi?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
                username: ruoyier
                password: 123456
注意：
 修改的数据库地址：jdbc:mysql://10.0.0.13:3306/ruoyi  地址、端口、数据库名都不能错。
root@ubuntu43:codes # ls RuoYi/sql/
quartz.sql  ruoyi.html  ruoyi.pdm  ry_20240601.sql
root@ubuntu43:codes # mysql -uruoyier -h10.0.0.43 -p123456
mysql> use ruoyi;
mysql> source /data/codes/RuoYi/sql/quartz.sql;
mysql> source /data/codes/RuoYi/sql/ry_20240601.sql;
查看效果
mysql> show tables;
root@ubuntu43:codes # cd RuoYi/
root@ubuntu43:RuoYi # mvn clean package -Dmaven.test.skip=true
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for ruoyi 4.7.9:
[INFO] 
[INFO] ruoyi .............................................. SUCCESS [  3.712 s]
[INFO] ruoyi-common ....................................... SUCCESS [01:16 min]
[INFO] ruoyi-system ....................................... SUCCESS [  0.693 s]
[INFO] ruoyi-framework .................................... SUCCESS [ 12.784 s]
[INFO] ruoyi-quartz ....................................... SUCCESS [  1.472 s]
[INFO] ruoyi-generator .................................... SUCCESS [  1.225 s]
[INFO] ruoyi-admin ........................................ SUCCESS [ 33.099 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  02:21 min
[INFO] Finished at: 2024-12-14T09:03:19Z
[INFO] ------------------------------------------------------------------------
root@ubuntu43:RuoYi # ls ruoyi-admin/target/
classes  generated-sources  maven-archiver  maven-status  ruoyi-admin.jar  ruoyi-admin.jar.original
运行项目 - 第一次启动较慢
root@ubuntu43:RuoYi # java -jar ./ruoyi-admin/target/ruoyi-admin.jar --server.port=9999
```

## 使用Nexus构建私有仓库









# **1. JVM 参数优化**

Tomcat 的性能高度依赖 JVM，因此调整 JVM 参数是优化的核心：

#### **堆内存分配**

- 根据服务器内存大小合理配置：

  ```
  -Xms2g -Xmx2g
  ```

  - `-Xms`：初始堆大小。
  - `-Xmx`：最大堆大小。

#### **GC 调优**

- 选择适合的垃圾回收器：

  ```
  -XX:+UseG1GC
  ```

  - 对于高并发应用，G1 GC 提供更好的暂停时间控制。

- 配置垃圾回收参数：

  ```
  -XX:MaxGCPauseMillis=200 -XX:+ParallelRefProcEnabled
  ```

#### **线程栈大小**

- 避免线程栈占用过多内存：

  ```
  -Xss512k
  ```

------

### **2. 连接器优化**

Tomcat 使用的 HTTP 连接器（如 NIO、APR）可以通过调整参数提升性能。

#### **启用 NIO 或 APR**

- 修改 server.xml：

  ```
  <Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" 
              maxThreads="500" 
              minSpareThreads="50" 
              acceptCount="200" 
              connectionTimeout="20000"
              enableLookups="false" 
              URIEncoding="UTF-8"/>
  ```

  - `maxThreads`：最大工作线程数，控制并发能力。
  - `minSpareThreads`：空闲线程数，减少频繁创建线程的开销。
  - `acceptCount`：请求队列最大长度。
  - `connectionTimeout`：连接超时时间。

#### **开启 Keep-Alive**

- 保持长连接以减少 TCP 建立的开销：

  ```
  <Connector port="8080" protocol="HTTP/1.1" 
              maxKeepAliveRequests="100" 
              keepAliveTimeout="5000"/>
  ```

  - `maxKeepAliveRequests`：限制单个连接允许的最大请求数。
  - `keepAliveTimeout`：Keep-Alive 超时时间。

------

### **3. 内存和线程池优化**

#### **优化线程池**

- 配置线程池大小以适应负载：

  - 调整 server.xml 的 Executor 元素：

    ```
    <Executor name="tomcatThreadPool" 
              namePrefix="catalina-exec-" 
              maxThreads="300" 
              minSpareThreads="20"/>
    ```

#### **控制会话内存**

- 配置会话持久化策略：

  - 在 context.xml 中禁用会话持久化：

    ```
    <Manager pathname="" />
    ```

  - 对于分布式部署，使用外部会话存储（如 Redis）。

------

### **4. 静态资源优化**

#### **配置静态资源缓存**

- 在 web.xml 中启用缓存：

  ```
  xml复制代码<servlet>
      <servlet-name>default</servlet-name>
      <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
      <init-param>
          <param-name>cacheMaxSize</param-name>
          <param-value>40960</param-value>
      </init-param>
      <init-param>
          <param-name>cacheTTL</param-name>
          <param-value>60000</param-value>
      </init-param>
  </servlet>
  ```

------

### **5. 压缩传输**

#### **启用 Gzip 压缩**

- 在 server.xml 中添加压缩支持：

  ```
  xml复制代码<Connector port="8080" protocol="HTTP/1.1"
              compression="on"
              compressionMinSize="1024"
              noCompressionUserAgents="gozilla, traviata"
              compressableMimeType="text/html,text/xml,text/plain,text/css,application/json,application/javascript"/>
  ```

  - `compression="on"`：启用压缩。
  - `compressionMinSize`：设置触发压缩的最小数据大小。
  - `compressableMimeType`：指定需要压缩的 MIME 类型。

------

### **6. 日志管理**

#### **减少日志开销**

- 调整日志级别：

  - 在  logging.properties 中设置日志级别为 INFOINFO

    ```
    properties
    org.apache.catalina.level = INFO
    ```

#### **分离访问日志**

- 将访问日志分离以减少 I/O 压力：

  - 修改 server.xml

    ```
    <Valve className="org.apache.catalina.valves.AccessLogValve"
           directory="logs"
           prefix="access_log"
           suffix=".txt"
           pattern="%h %l %u %t &quot;%r&quot; %s %b" />
    ```

------

### **7. 安全性优化**

#### **限制管理功能访问**

- 配置管理界面访问 IP：

  - 修改 conf/tomcat-users.xml

    ```
    <role rolename="manager-gui"/>
    <user username="admin" password="securepassword" roles="manager-gui"/>
    ```

  - 使用防火墙限制管理功能访问。

#### **禁用未使用的服务**

- 删除 `webapps` 中不必要的应用（如 `examples`、`docs`）。

#### **启用 HTTPS**

- 配置 HTTPS 连接：

  ```
  <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
              SSLEnabled="true" 
              maxThreads="200" 
              scheme="https" 
              secure="true"
              keystoreFile="conf/keystore.jks" 
              keystorePass="changeit"/>
  ```

------

### **8. 系统级优化**

- 文件描述符限制

  - 确保系统允许足够的文件描述符数：

    ```
    ulimit -n 65535
    ```

- TCP 调优

  - 调整操作系统 TCP 参数：

    ```
    echo "net.ipv4.tcp_fin_timeout=30" >> /etc/sysctl.conf
    ```

# 2、什么是OOM，Java程序如何解决OOM问题

**OOM（Out of Memory）** 是 Java 中常见的运行时错误，全称是 **OutOfMemoryError**，表示程序运行时无法为对象分配足够的内存。通常由内存不足或内存管理问题引发。

**常见的 OOM 类型**

1. **Java Heap Space**
   - 堆内存不足，无法为新对象分配空间。
   - 原因：
     - 创建了过多的对象且未被及时回收。
     - 堆内存配置不足。
     - 内存泄漏（对象不被回收）。
2. **GC Overhead Limit Exceeded**
   - 垃圾回收耗时过多但未能释放足够内存。
   - 原因：
     - 内存不足，导致 GC 频繁触发。
     - 不断创建新对象，GC 来不及回收。
3. **Metaspace**（JDK 8+）
   - Metaspace 内存不足，无法加载类元数据。
   - 原因：
     - 动态加载类过多。
     - 元空间设置不足。
4. **Direct Buffer Memory**
   - NIO 的直接内存不足。
   - 原因：
     - 分配的直接内存超过 JVM 允许范围。
     - 未正确释放直接内存。
5. **Unable to Create New Native Thread**
   - 无法创建新的线程，通常由于系统资源限制。
   - 原因：
     - 线程数过多。
     - 系统级资源（如文件描述符、内存）耗尽。

------

### **如何解决 OOM 问题**

#### **1. Java Heap Space**

**解决思路：**

- **增加堆内存**：
  使用 JVM 参数调整堆大小，例如：

  ```
  -Xms2g -Xmx4g
  ```

  - `-Xms`：初始堆大小。
  - `-Xmx`：最大堆大小。

- **定位内存泄漏**：

  - 使用工具分析堆转储文件（Heap Dump），例如：
    - **VisualVM**。
    - **Eclipse MAT（Memory Analyzer Tool）**。
  - 检查长时间占用内存的对象及其引用关系。

- **优化代码**：

  - 避免无用对象（如大集合）长期占用内存。
  - 减少大对象使用或及时清理。

------

#### **2. GC Overhead Limit Exceeded**

**解决思路：**

- **增加堆大小**：

  - 提高最大堆大小以减少 GC 频率：

    ```
    -Xmx4g
    ```

- **优化垃圾回收器**：

  - 选择适合的 GC 策略：

    - 对吞吐量要求高的场景：`-XX:+UseParallelGC`。
    - 对低延迟要求高的场景：`-XX:+UseG1GC`。

  - 调整 GC 参数，例如：

    ```
    -XX:MaxGCPauseMillis=200
    ```

- **检查内存使用**：

  - 确保没有过度创建短生命周期的对象。

------

#### **3. Metaspace**

**解决思路：**

- 调整元空间大小：

  - 增加元空间限制：

    ```
    -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m
    ```

- 检查类加载逻辑：

  - 避免频繁动态生成类（如使用 CGLIB、Javassist 动态代理）。
  - 确保不重复加载相同的类。

------

#### **4. Direct Buffer Memory**

**解决思路：**

- 调整直接内存大小：

  - 增加直接内存限制：

    ```
    -XX:MaxDirectMemorySize=512m
    ```

- 及时释放直接内存：

  - 确保使用的 `ByteBuffer` 在不用时手动清理。
  - 使用 `sun.misc.Cleaner` 工具清理直接内存。

------

#### **5. Unable to Create New Native Thread**

**解决思路：**

- **减少线程数**：

  - 检查线程池配置，避免线程过多。

  - 使用线程池代替直接创建线程，例如：

    ```
    ExecutorService executor = Executors.newFixedThreadPool(10);
    ```

- **增加系统资源限制**：

  - 调整系统对线程数量的限制：

    - 修改 ulimit 配置：

      ```
      ulimit -u 65535
      ```

    - 增加 /etc/security/limits.conf中的用户线程数限制：

      ```
      *  soft  nproc  65535
      *  hard  nproc  65535
      ```

------

### **监控和预防 OOM**

#### **1. 启用 JVM 监控**

- 使用工具监控内存使用情况：
  - JConsole。
  - VisualVM。
  - Prometheus + Grafana。

#### **2. 定期导出 Heap Dump**

- 当程序出现 OOM 时，自动生成堆转储文件：

  ```
  -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path/to/dump
  ```

#### **3. 代码优化**

- 避免滥用单例、静态集合等长期存储数据的结构。
- 减少频繁的短生命周期对象创建。

------

### **总结**

OOM 是 Java 应用内存管理中的常见问题，可以通过调整 JVM 参数、优化代码逻辑以及定期监控内存来解决和预防。及时分析堆转储文件和运行日志是定位问题的关键步骤。



# 3、基于客户端的分布式机集群和基于服务端的分布式集群有什么区别

基于**客户端**的分布式集群和基于**服务端**的分布式集群在架构设计、数据处理方式、负载均衡以及开发复杂性等方面有明显的区别。以下是两者的主要区别：

------

### **1. 集群架构**

- **基于客户端的分布式集群：**
  - 客户端直接与集群中的各节点交互。
  - 客户端负责选择目标节点，并管理与节点之间的通信。
  - 常见于分布式数据库（如 MongoDB、Cassandra）和分布式缓存（如 Redis 的 Cluster 模式）。
- **基于服务端的分布式集群：**
  - 客户端只与一个或多个服务端入口（如负载均衡器、网关）通信。
  - 服务端（或负载均衡器）负责请求的分发和处理。
  - 常见于 Web 服务（如 Nginx 反向代理集群）和分布式文件系统（如 Hadoop HDFS）。

------

### **2. 请求路由和负载均衡**

- **基于客户端的分布式集群：**
  - 客户端自行实现负载均衡，通常通过以下方式：
    - 哈希算法（如一致性哈希）。
    - 节点列表（通常通过服务注册发现获取）。
  - 客户端需要有足够的逻辑来处理节点故障、超时重试等情况。
- **基于服务端的分布式集群：**
  - 请求首先被服务端的负载均衡器或代理分发，负载均衡方式包括：
    - 轮询、加权轮询。
    - 最小连接数。
    - 动态响应时间。
  - 客户端无需感知集群内部细节。

------

### **3. 数据一致性管理**

- **基于客户端的分布式集群：**
  - 数据一致性主要由客户端负责：
    - 客户端需要知道数据存储在哪些节点。
    - 客户端可能需要处理分片（sharding）或副本（replication）。
  - 数据模型的复杂性会增加，特别是在需要事务支持的场景下。
- **基于服务端的分布式集群：**
  - 数据一致性由集群服务端统一管理：
    - 服务端可以实现分布式事务或强一致性协议（如 Raft、Paxos）。
    - 客户端只需关心请求结果，不需要参与数据一致性逻辑。

------

### **4. 故障处理**

- **基于客户端的分布式集群：**
  - 客户端需要负责节点状态检测和故障切换（Failover）。
  - 如果一个节点不可用，客户端需选择其他节点重试。
  - 客户端逻辑更复杂，可能会导致实现成本上升。
- **基于服务端的分布式集群：**
  - 故障处理通常由服务端完成：
    - 负载均衡器可动态移除故障节点。
    - 服务端可以实现自动恢复机制。
  - 客户端对集群的容错感知较弱，但系统可靠性更高。

------

### **5. 复杂性与维护成本**

- **基于客户端的分布式集群：**
  - 优点：
    - 适用于高性能场景，客户端直接与目标节点通信，减少了中间代理的性能损耗。
    - 扩展性好，新增节点后客户端可动态感知。
  - 缺点：
    - 客户端需要实现复杂的逻辑，如服务发现、负载均衡、故障切换。
    - 不同的客户端实现可能导致不一致行为。
- **基于服务端的分布式集群：**
  - 优点：
    - 客户端逻辑简单，仅需与服务端通信。
    - 服务端可集中管理集群，简化了运维和管理。
  - 缺点：
    - 代理层可能成为性能瓶颈（需要优化负载均衡器性能）。
    - 增加了额外的网络跳转开销。

------

### **6. 使用场景**

- **基于客户端的分布式集群：**
  - 分布式数据库（如 MongoDB、Cassandra）。
  - 分布式缓存（如 Redis Cluster 模式）。
  - 分布式消息队列（如 Kafka 的客户端分区选择）。
- **基于服务端的分布式集群：**
  - Web 服务集群（如使用 Nginx 或 Envoy 做反向代理）。
  - 分布式文件系统（如 Hadoop HDFS）。
  - 大型微服务架构中的 API 网关（如 Spring Cloud Gateway）。

------

### **总结**

| **特点**       | **基于客户端的分布式集群**   | **基于服务端的分布式集群**         |
| -------------- | ---------------------------- | ---------------------------------- |
| 请求路由方式   | 客户端负责                   | 服务端（负载均衡器）负责           |
| 负载均衡       | 客户端实现（如哈希）         | 服务端实现（如轮询、最小连接数等） |
| 故障处理       | 客户端感知节点故障，处理切换 | 服务端动态移除故障节点             |
| 数据一致性管理 | 客户端负责分片和一致性逻辑   | 服务端统一管理                     |
| 复杂性         | 客户端复杂，需实现更多逻辑   | 服务端复杂，客户端简单             |
| 使用场景       | 分布式数据库、分布式缓存等   | Web 服务、文件系统、微服务网关等   |

两者的选择应根据系统架构需求、性能目标以及开发和维护成本权衡。



# 4、在工作中监控过Java程序的哪些指标

在工作中监控 Java 程序是保障应用稳定性和性能的重要任务。以下是常见的 Java 程序监控指标及其意义：

------

### **1. JVM 内存指标**

#### **堆内存使用情况（Heap Memory Usage）**

- 监控内容：
  - 当前堆内存使用量。
  - 堆内存最大值。
  - 老年代（Old Generation）和新生代（Young Generation）的内存使用。
- 意义：
  - 判断是否存在内存泄漏。
  - 评估 GC 的效果，防止内存不足导致 OOM。
- 工具：
  - JConsole、VisualVM、Prometheus（结合 JVM Exporter）。

------

#### **非堆内存使用情况（Non-Heap Memory Usage）**

- 监控内容：
  - 元空间（Metaspace）使用情况。
  - 直接内存（Direct Memory）的分配和使用。
- 意义：
  - 确保类加载和 NIO 分配足够的内存，避免 OOM。

------

#### **垃圾回收（GC）指标**

- 监控内容：
  - GC 次数（Minor GC 和 Full GC）。
  - GC 时间（每次和总计）。
  - GC 停顿时间（Stop-the-World）。
- 意义：
  - 过多的 GC 次数和时间表明内存使用或配置可能有问题。
- 工具：
  - jstat、GC 日志（通过 `-Xlog:gc` 开启 GC 日志）。
  - APM 工具（如 Prometheus、Grafana）。

------

### **2. 线程指标**

#### **线程状态**

- 监控内容：
  - 各类线程状态的数量（运行中、阻塞、等待、终止等）。
  - 死锁检测。
- 意义：
  - 确保线程没有因锁或资源争用导致阻塞或死锁。
- 工具：
  - jstack。
  - VisualVM。

#### **线程池监控**

- 监控内容：
  - 线程池中的当前活跃线程数。
  - 任务队列长度。
  - 拒绝的任务数。
- 意义：
  - 确保线程池配置合理，避免线程资源耗尽或任务积压。
- 工具：
  - 自定义线程池监控，通过 `ThreadPoolExecutor` 提供的监控方法。

------

### **3. CPU 使用**

#### **应用级 CPU 使用**

- 监控内容：
  - Java 进程的 CPU 占用。
  - 每个线程的 CPU 时间（热点线程）。
- 意义：
  - 检查是否存在性能瓶颈或 CPU 使用过高的问题。
- 工具：
  - `top`/`htop` 命令。
  - Java Mission Control。
  - APM 工具。

#### **线程级 CPU 使用**

- 监控内容：
  - 找出高 CPU 占用的线程。
  - 检查线程是否处于无限循环或其他异常情况。
- 工具：
  - 使用 `jstack` 获取线程堆栈。
  - `ps -L` 查看线程的 CPU 占用。

------

### **4. I/O 指标**

#### **文件 I/O**

- 监控内容：
  - 磁盘读写速率。
  - 文件描述符使用数。
- 意义：
  - 评估文件操作性能，避免文件描述符耗尽。
- 工具：
  - lsof。
  - APM 工具。

#### **网络 I/O**

- 监控内容：
  - 网络连接数。
  - 每秒数据包发送和接收速率。
- 意义：
  - 检查是否存在连接泄漏或网络性能问题。
- 工具：
  - Netstat。
  - Tcpdump。

------

### **5. 请求和事务指标**

#### **请求吞吐量（TPS/QPS）**

- 监控内容：
  - 每秒处理的请求数量。
- 意义：
  - 反映系统的处理能力。

#### **响应时间**

- 监控内容：
  - 平均响应时间。
  - 最大响应时间（P99、P95）。
- 意义：
  - 确保系统响应符合性能要求。

#### **错误率**

- 监控内容：
  - 请求的失败率。
  - 常见异常的数量。
- 意义：
  - 评估系统的可靠性。
- 工具：
  - APM 工具（如 Prometheus、Skywalking）。

------

### **6. 应用级指标**

#### **类加载**

- 监控内容：
  - 已加载类的总数。
  - 类卸载的数量。
- 意义：
  - 监控动态类加载情况，避免类加载器泄漏。

#### **锁争用**

- 监控内容：
  - 锁等待的线程数量。
  - 锁等待时间。
- 意义：
  - 确保没有线程因锁争用导致性能问题。
- 工具：
  - `jstack`。
  - Java Flight Recorder。

------

### **7. 自定义业务指标**

- 根据业务需求监控：
  - 队列长度（如消息队列、任务队列）。
  - 缓存命中率（如 Redis、Guava 缓存）。
  - 数据库查询时间和失败率。

------

### **监控工具推荐**

#### **开源工具**

- Prometheus + Grafana：通用监控平台，可监控 JVM 和业务指标。
- ELK（Elasticsearch, Logstash, Kibana）：日志监控和分析。
- Zabbix：传统监控工具。

#### **专业 APM 工具**

- Skywalking：分布式链路跟踪。
- Pinpoint：轻量级 Java 应用性能监控。
- New Relic 或 AppDynamics：商业化 APM 工具。

#### **JVM 专属工具**

- VisualVM：监控 JVM 性能。
- Java Mission Control：JDK 自带，适合分析 GC 和性能瓶颈。
- JConsole：轻量级的 JVM 监控工具。

------

### **总结**

Java 程序监控涵盖 JVM 内存、GC、线程、CPU、I/O、请求响应等多个方面。通过适当的工具和监控指标，可以快速定位性能瓶颈或异常，提升应用的稳定性和运行效率。





# 5、java对象进入老年代的原因

在 JVM 的垃圾回收机制中，**堆内存**通常分为 **年轻代（Young Generation）** 和 **老年代（Old Generation）**。对象从年轻代进入老年代的主要原因与对象的生命周期、内存使用和垃圾回收策略有关。以下是对象进入老年代的常见原因：

------

### **1. 对象年龄（Age）超过晋升阈值**

- 机制：

  - 新生对象一般分配在年轻代的 **Eden 区**。
  - 当对象在年轻代的垃圾回收（Minor GC）中幸存一次，其年龄（Age）加 1。
  - 当对象的年龄达到 **晋升阈值（默认 15）** 时，就会被移入老年代。

- 参数控制：

  - `-XX:MaxTenuringThreshold`：设置对象年龄阈值。

  - 示例：

    ```
    -XX:MaxTenuringThreshold=10
    ```

------

### **2. 老年代空间不足**

- 机制：

  - 如果一个对象体积较大，且年轻代无法容纳，则直接分配到老年代（**大对象直接进入老年代**）。
  - 常见于大型数组或字符串对象。

- 参数控制：

  - `-XX:PretenureSizeThreshold`：设置超过此大小的对象直接分配到老年代。

  - 示例：

    ```
    -XX:PretenureSizeThreshold=1M
    ```

------

### **3. 动态年龄判断**

- 机制：
  - JVM 会根据实际情况动态调整对象的晋升条件。
  - 如果某一年龄段的对象累计大小超过 **Survivor 区的一半**，则该年龄及以上的所有对象会直接晋升到老年代。
- 目的：
  - 避免 Survivor 区被占满，提升 GC 效率。

------

### **4. Survivor 区不足**

- 机制：

  - 新生对象在 Eden 区分配，经过 Minor GC 后存活的对象被移动到 Survivor 区（S0 或 S1）。
  - 如果 Survivor 区空间不足，存活的对象会直接晋升到老年代。

- 参数控制：

  - `-XX:SurvivorRatio`：调整 Eden 区和 Survivor 区的比例。

  - 示例：

    ```
    -XX:SurvivorRatio=8
    ```

------

### **5. 长期存活的对象**

- 机制：
  - 长生命周期的对象，如缓存、静态对象等，一旦被判断为长期存活，就会被晋升到老年代。
  - 常见于：会话对象（Session）、数据库连接池等。

------

### **6. Full GC 后幸存**

- 机制：
  - 在 Full GC（针对整个堆的 GC）时，如果一个对象存活并且年轻代没有足够空间接纳，最终会被移入老年代。

------

### **7. 直接分配到老年代**

- 机制：
  - 某些情况下，JVM 会直接将对象分配到老年代：
    - 启用了 `-XX:+UseSerialGC` 或其他垃圾收集器策略。
    - 应用场景中频繁分配的大对象，超过年轻代的分配能力。
- 参数控制：
  - `-XX:PretenureSizeThreshold`（见第 2 点）。

------

### **8. 垃圾收集器的特性**

不同的垃圾收集器对老年代分配和晋升的策略可能不同：

- G1 垃圾收集器：
  - 对象按分区管理，可能直接从年轻代晋升到老年代的特定分区。
- CMS 垃圾收集器：
  - 老年代使用并发标记清除，晋升策略与默认垃圾收集器类似。

------

### **参数优化建议**

- 调整晋升阈值：
  - **适合短生命周期对象：** 增大 `MaxTenuringThreshold`，延迟对象晋升。
  - **适合长生命周期对象：** 减小 `MaxTenuringThreshold`，加快晋升速度。
- 配置 Survivor 区：
  - 增大 Survivor 区的比例，避免对象过早晋升。
- 管理大对象：
  - 使用 `-XX:PretenureSizeThreshold`，防止大对象频繁触发 GC。
- 选择合适的垃圾收集器：
  - 针对高吞吐量选择 G1，针对低延迟选择 ZGC 或 Shenandoah。

------

通过监控 GC 日志和调整 JVM 参数，可以优化对象的分配和晋升策略，从而降低 GC 开销，提升程序性能。









20250316
