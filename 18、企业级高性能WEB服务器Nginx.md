# 18企业级高性能WEB服务器Nginx

## nginx构架

![image-20241203185119875](5day-png\18nginx构架.png)

```
在Nginx中，Master进程和Worker进程分别承担不同的角色，共同协作以提供高性能的服务
Nginx通过灵活的配置文件来实现各种功能的地址
Nginx使用简洁而灵活的配置文件来定义服务器的行为。
配置文件通常包括全局配置、HTTP模块配置、Server配置以及Location配置。
```

### **master进程和worker进程**

master进程

```
负责管理Worker进程的生命周期。
处理来自管理员的信号。
负责配置文件的加载和重新加载。
启动、停止Worker进程，管理共享资源（如缓存）。

- 启动过程：
 	启动Nginx后，Master紧随启动。读取配置并初始化全局资源，并创建指定数量的Worker进程
- 创建Worker进程：
 	Master创建的Worker进程来处理实际的客户端请求。
 	每个Worker进程都是一个独立的进程，它们之间相互独立，互不影响
- 管理Worker进程：
 	Master进程负责监控Worker进程的状态。
 	若某个Worker进程异常退出，Master会重新启动一个新的Worker进程，确保服务可用
```

worker进程

```
- 处理客户端请求：
 	Worker进程是实际处理客户端请求的进程。
 	每个Worker进程是单线程的，但通过多进程的方式，Nginx能够同时处理多个请求，实现高并发
- 事件驱动和异步处理：
 	Worker进程采用事件驱动的异步模型。
 	它使用事件循环和非阻塞I/O操作，允许在不同的连接之间高效切换，以处理大量并发请求
- 负载均衡：
 	如果配置了负载均衡，多个Worker进程可以分担负载，均衡地分配请求，提高系统的整体性能
```

Master进程与Worker进程

```
在运行时，Master 进程负责监控 Worker 进程的状态，
 	如果某个 Worker 进程异常退出，Master 进程会重新启动一个新的 Worker 进程，确保服务的稳定性。
Master 进程和 Worker 进程之间通过信号进行通信，
	例如在重新加载配置时，Master 进程会向 Worker 进程发送信号，通知它们重新加载配置而无需停止服务。
	
这种Master-Worker模型使Nginx在高并发环境下表现出色，同时能够保持稳定性和可扩展性。
Master进程和Worker进程之间的协同工作是Nginx架构的关键之一。
```

### 设计原理

异步非阻塞的事件驱动模型

```
Nginx采用单线程（或说主线程和工作线程组合）的事件驱动模型，通过事件驱动的方式处理请求和连接。
使用非阻塞I/O来处理客户端的请求和与后端服务器的通信，以提高并发处理能力和系统性能。
所有I/O操作都通过事件通知机制完成，不会阻塞进程。
```

多级事件处理机制

```
Nginx使用多级事件处理机制来提高事件的处理效率。
它将事件分为读取事件、处理事件和写入事件三个阶段，每个阶段都有相应的事件处理模块，能够按需处理特定类型的事件，提高请求处理的效率。
```

基于事件的模块化架构

```
Nginx的架构是基于模块化设计的，每个模块负责处理特定类型的功能。
它包括核心模块、HTTP模块、邮件模块等，用户可以根据需要启用或禁用模块，灵活配置Nginx的功能。
核心模块实现Nginx的基本功能，如事件处理、内存管理、配置解析等。
标准HTTP模块提供HTTP服务的功能，如静态文件服务、反向代理、负载均衡等。
第三方模块由社区或开发者提供，用于扩展Nginx的功能，如Lua模块、Redis模块等。
```

高效的请求处理流程

```
Nginx的请求处理流程高度优化，能够高效地处理HTTP请求。
主要流程包括接收请求、解析请求、选择处理模块、生成响应和发送响应。
```

其他特性

```
高并发处理能力：Nginx能够高效地处理大量并发连接，适用于高负载环境。
低资源消耗：相比传统的服务器软件，Nginx的内存占用更低，能够在相同硬件上处理更多的请求。
可靠性和稳定性：Nginx经过了长时间的生产环境验证，稳定性高，能够处理高负载和长时间运行的工作负载。
灵活的负载均衡和反向代理：Nginx内置了负载均衡和反向代理功能，能够将请求分发到多个后端服务器，提供高可用性和可靠性。
```

## nginx模块

核心模块（Core Modules）

```
Nginx 的核心模块主要有三部份，分别是
    - Events Module： 处理与事件相关的操作，如连接的接受和关闭。包括事件驱动机制等
    - HTTP Module： 处理 HTTP 请求和响应，包括配置 HTTP 服务器、反向代理和负载均衡等
    - Mail Module：提供邮件代理服务顺的功能，支持 IMAP，POP3等协议
```

基础模块 （Base Modules）

```
基础模块包含众多子模块，用于实现Nginx的各种功能，这些模块主要是 HTTP 级别。
    - Core Module： 包含基本的 HTTP 功能，如配置服务器块、location 块等
    - Access Module： 处理访问控制，包括允许或拒绝特定的 IP 地址或用户
    - FastCGI Module： 支持 FastCGI 协议，用于与 FastCGI 进程通信
    - Proxy Module： 提供反向代理功能，用于将请求代理到后端服务器
    - Upstream Module： 用于定义负载均衡的后端服务器组
    - Rewrite Module：重写模块，用于修改客户端请求的URL。
    - SSL Module：用于支持HTTPS协议，提供加密传输和安全认证功能。
    - Gzip Module：压缩模块，用于对HTTP响应数据进行压缩，减少数据传输量，提高访问速度。
```

第三方模块（HTTP Modules）

```
Nginx提供了一套模块化的架构，允许开发者编写自定义的模块来扩展功能。这些第三方模块可以包括但不限于以下几类
    - HTTP 模块： 扩展HTTP模块的功能，添加自定义的处理逻辑
    - Filter 模块： 提供对HTTP请求和响应进行过滤的功能
    - Load Balancer 模块： 实现负载均衡策略
    - Access Control 模块： 控制对资源的访问权限
    - Security 模块： 提供安全性增强的功能，如防火墙、DDoS防护等
    - Authentication 模块： 处理用户认证逻辑
```

## nginx安装

ubuntu

```shell
软件安装
root@ubuntu24:~# apt install nginx -y
注意：
 	默认情况下会安装 nginx-common软件包，所以我们需要额外安装
 	nginx-core fcgiwrap nginx-doc
建议使用
root@ubuntu-42:~ # apt install nginx nginx-core fcgiwrap nginx-doc
```

rocky

```shell
软件安装
[root@rocky9 ~]# yum install nginx nginx-core -y
注意：
	默认情况下，会安装依赖包nginx-core、nginx-filesystem、rocky-logos-httpd
建议使用
root@rocky9-14:~ # yum install nginx nginx-core nginx-filesystem rocky-logos-httpd
```

### nginx源码编译安装

ubuntu

```shell
#编译环境准备
apt install build-essential gcc g++ libc6 libc6-dev libpcre3 libpcre3-dev libssl-dev libsystemd-dev zlib1g-dev
apt install libxml2 libxml2-dev libxslt1-dev php-gd libgd-dev geoip-database libgeoip-dev
#注意：
 	#第二条编译环境内容是 默认二进制环境开启功能所依赖的库环境
#创建运行用户
root@ubuntu-42:~ # useradd -r -s /usr/sbin/nologin nginx
#下载最新版源码并解压
root@ubuntu-42:~ # wget https://nginx.org/download/nginx-1.22.1.tar.gz
#定制配置
root@ubuntu-42:~ # tar xf nginx-1.22.1.tar.gz 
root@ubuntu-42:~ # cd nginx-1.22.1/
root@ubuntu-42:nginx-1.22.1 # ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  man  README  src
root@ubuntu-42:nginx-1.22.1 # ./configure --prefix=/data/server/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module
#编译安装
root@ubuntu-42:nginx-1.22.1 # make
root@ubuntu-42:nginx-1.22.1 # make install
#修改目录属主属组并查看
 root@ubuntu-42:nginx-1.22.1 # ls /data/server/nginx/
conf  html  logs  sbin
root@ubuntu-42:nginx-1.22.1 # ls /data/server/nginx/ -l
total 16
drwxr-xr-x 2 root root 4096 Dec  3 19:57 conf
drwxr-xr-x 2 root root 4096 Dec  3 19:57 html
drwxr-xr-x 2 root root 4096 Dec  3 19:57 logs
drwxr-xr-x 2 root root 4096 Dec  3 19:57 sbin
root@ubuntu-42:nginx-1.22.1 # chown -R nginx:nginx /data/server/nginx/
#创建软链接
root@ubuntu-42:nginx-1.22.1 # ln -sv /data/server/nginx/sbin/nginx /usr/sbin/nginx
'/usr/sbin/nginx' -> '/data/server/nginx/sbin/nginx'
#man手册导入
#导入手册
root@ubuntu-42:nginx-1.22.1 # cp man/nginx.8 /usr/share/man/man8/
#构建数据库
root@ubuntu-42:nginx-1.22.1 # mandb

#定制专属管理
#创建PID 目录
root@ubuntu-42:nginx-1.22.1 # mkdir /data/server/nginx/run
root@ubuntu-42:nginx-1.22.1 # chown -R nginx:nginx /data/server/nginx/run/
#修改配置文件，设置pid文件路径
root@ubuntu-42:nginx-1.22.1 # cat /data/server/nginx/conf/nginx.conf | grep "^pid.*"
pid     /data/server/nginx/run/nginx.pid;
#创建nginx服务脚本
root@ubuntu-42:nginx-1.22.1 # cat /usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/data/server/nginx/run/nginx.pid
ExecStart=/data/server/nginx/sbin/nginx -c /data/server/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target

#重载服务脚本
root@ubuntu42:~# systemctl daemon-reload 
#用服务启动
root@ubuntu42:~# systemctl start nginx
#查看状态
root@ubuntu42:~# systemctl status nginx.service
```

rocky

```shell
yum install gcc make gcc-c++ glibc glibc-devel pcre pcre-devel openssl openssl-devel systemd-devel zlib-devel 
yum install libxml2 libxml2-devel libxslt libxslt-devel php-gd gd-devel
#注意：
 	#第二条编译环境内容是 默认二进制环境开启功能所依赖的库环境
#Rocky系统编译安装
#获取软件
mkdir /softs; cd /softs
wget https://nginx.org/download/nginx-1.23.0.tar.gz 
tar xf nginx-1.23.0.tar.gz
#创建运行用户
useradd -r -s /usr/sbin/nologin nginx
cd nginx-1.23.0/
#定制配置
./configure --prefix=/data/server/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module
#编译软件
make
#安装软件
make install

#启动应用
#修改文件属性
chown -R nginx:nginx /data/server/nginx/
#启动nginx
/data/server/nginx/sbin/nginx 
#检查效果
/data/server/nginx/sbin/nginx -V

#查看帮助信息
#命令行处理
echo "export PATH=/data/server/nginx/sbin:$PATH" >> /etc/bashrc
source /etc/bashrc

#man文档处理
cd nginx-1.23.0/
cp man/nginx.8 /usr/share/man/man8/
man nginx
```

## nginx命令

```shell
Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
             [-e filename] [-c filename] [-g directives]

Options:
  -?,-h         : 帮助
  -v            : 显示版本并退出
  -V            : 显示版本并配置选项然后退出
  -t            : 测试配置并退出
  -T            : 测试配置，转储并退出
  -q            : 在配置测试期间抑制非错误消息
  -s signal     : 向主进程发送信号: stop, quit, reopen, reload
  -p prefix     : 设置前缀路径（默认：/data/server/nginx/）
  -e filename   : 设置错误日志文件（默认：logs/error.log）
  -c filename   : 设置配置文件（默认：conf/nginx.conf）
  -g directives : 从配置文件中设置全局指令
  
信号说明
 	stop 		#立即停止，没处理完的请求也会被立即断开，相当于信号 SIGTERM,SIGINT
    quit 		#优雅退出，不再接收新的请求，但己建立的连接不受影响，相当于信号 SIGQUIT
    reopen 		#重开一个新的日志文件，日志内容写新文件，相当于信号 SIGUSR1 
    reload 		#重新加载配置文件，重新生成worker 进程，不中断正在处理的请求，
   				#己建立的连接按旧配置处理，新连接用新配置处理
    			#同 systemctl reload nginx
    SIGUSR2 	#平滑升级二进制文件，适用于版本升级 
    SIGWINCH 	#优雅的停止工作进程，适用于版本升级
```

## 配置定制

```shell
#1 查看默认的配置文件
root@ubuntu-43:~ # grep -Ev "^#|^$" /etc/nginx/nginx.conf 
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;
events {
        worker_connections 768;
        # multi_accept on;
}
http {
        ##
        # Basic Settings
        ##
        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;
        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        ##
        # SSL Settings
        ##
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;
        ##
        # Logging Settings
        ##
        access_log /var/log/nginx/access.log;
        ##
        # Gzip Settings
        ##
        gzip on;
        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
        ##
        # Virtual Host Configs
        ##
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
root@ubuntu-43:~ # ps aux | grep nginx
root        4297  0.0  0.1  59304  2416 ?        Ss   13:45   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    4298  0.0  0.3  60100  6000 ?        S    13:45   0:00 nginx: worker process
www-data    4299  0.0  0.3  60100  6128 ?        S    13:45   0:00 nginx: worker process
root        4315  0.0  0.1   6544  2304 pts/3    S+   13:47   0:00 grep --color=auto nginx
root@ubuntu-43:~ # pstree -p | grep nginx
           |-nginx(4297)-+-nginx(4298)
           |              -nginx(4299)

#2 定制启动的进程信息
root@ubuntu-43:~ # grep "^# wo" /etc/nginx/nginx.conf 
# worker_processes auto;
root@ubuntu-43:~ # nginx -t
root@ubuntu-43:~ # systemctl restart nginx
root@ubuntu-43:~ # ps aux | grep nginx
root        4370  0.0  0.1  59304  2368 ?        Ss   13:49   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    4371  0.0  0.3  60100  6080 ?        S    13:49   0:00 nginx: worker process
root        4374  0.0  0.1   6544  2304 pts/3    S+   13:49   0:00 grep --color=auto nginx
root@ubuntu-43:~ # pstree -p | grep nginx
           |-nginx(4370)---nginx(4371)
           

#3 nginx命令使用参数启动服务
root@ubuntu-43:~ # systemctl stop nginx.service 
root@ubuntu-43:~ # nginx -g "worker_processes 3;"
root@ubuntu-43:~ # ps aux | grep nginx
root        4406  0.0  0.1  59304  2368 ?        Ss   13:53   0:00 nginx: master process nginx -g worker_processes 3;
www-data    4407  0.0  0.3  60100  5952 ?        S    13:53   0:00 nginx: worker process
www-data    4408  0.0  0.3  60100  5952 ?        S    13:53   0:00 nginx: worker process
www-data    4409  0.0  0.3  60100  5952 ?        S    13:53   0:00 nginx: worker process
root        4411  0.0  0.1   6544  2304 pts/3    S+   13:54   0:00 grep --color=auto nginx
root@ubuntu-43:~ # pstree -p | grep nginx
           |-nginx(4406)-+-nginx(4407)
           |             |-nginx(4408)
           |             `-nginx(4409)
```

## 信号stop&quit

```
在客户端下载的过程中服务端发送 stop 信号，客户端会立即断开
在客户端下载的过程中服务端发送 stop 信号，当占用nginx的进程工作结束后，所有的nginx进程就会丢弃，客户端断开连接。
```

 

## 平滑升级

```
平滑升级通常包括以下步骤：
- 新版本或配置准备：
 	准备好新版本的 Nginx 可执行文件或配置文件，并确保它们没有问题
- 新版本资源替换旧版本资源：
 	进行资源替换，此步骤要备份旧版本资源，可以用作回滚
- 发送信号：
 	使用 nginx -s SIGUSR2，nginx -s reload 等方式向 Nginx 主进程发送重新加载的信号
- 新的工作进程启动：
 	Nginx 主进程接收到重新加载信号后，会启动新的工作进程，并使用新的配置文件或软件版本
- 平滑过渡：
 	新的工作进程逐渐接管现有的连接。现有的连接会在旧的工作进程中继续处理，而新的连接会由新的工作进程处理
- 旧的进程退出：
 	当旧的工作进程不再有活动连接时，它会被关闭
```

```shell
#获取软件版本
root@ubuntu-42:~ # mkdir /data/server/nginx -p
root@ubuntu-42:~ # mkdir /data/softs -p
root@ubuntu-42:~ # cd /data/softs/
root@ubuntu-42:softs # wget http://nginx.org/download/nginx-1.25.0.tar.gz
#解压文件
root@ubuntu-42:softs # tar xf nginx-1.25.0.tar.gz 
root@ubuntu-42:softs # cd nginx-1.25.0
#per环境准备 - 基于apt方式安装的nginx的场景中才会出现
#结果显示，库文件都是携带版本号的，编译的时候，很多是不带版本号的，所以创建一个软连接
root@ubuntu-42:nginx-1.25.0 # find / -name libperl.so*
/usr/lib/x86_64-linux-gnu/libperl.so.5.38.2
/usr/lib/x86_64-linux-gnu/libperl.so.5.38
root@ubuntu-42:nginx-1.25.0 # ln -s /usr/lib/x86_64-linux-gnu/libperl.so.5.38 /usr/lib/x86_64-linux-gnu/libperl.so
#编译环境准备
root@ubuntu-42:nginx-1.25.0 # apt install build-essential gcc g++ libc6 libc6-dev libpcre3 libpcre3-dev libssl-dev libsystemd-dev zlib1g-dev libxml2 libxml2-dev libxslt1-dev php-gd libgd-dev geoip-database libgeoip-dev
#编译新版本 -- 注意：将 1.24.0 改为 1.25.0
root@ubuntu-42:nginx-1.25.0 # nginx -V
root@ubuntu-42:nginx-1.25.0 # ./configure 
root@ubuntu-42:nginx-1.25.0 # make

#新旧版本切换
#查看新旧版本
root@ubuntu-42:nginx-1.25.0 # nginx -v
nginx version: nginx/1.24.0 (Ubuntu)
root@ubuntu-42:nginx-1.25.0 # ./objs/nginx -v
nginx version: nginx/1.25.0
#新旧版本的执行文件
root@ubuntu-42:nginx-1.25.0 # ls ./objs/nginx /usr/sbin/nginx -l
-rwxr-xr-x 1 root root 6584000 Dec  4 12:57 ./objs/nginx
-rwxr-xr-x 1 root root 1313752 Sep 10 13:27 /usr/sbin/nginx
#备份就执行文件
root@ubuntu-42:nginx-1.25.0 # mv /usr/sbin/nginx /usr/sbin/nginx-1.24
#使用新版本执行文件
root@ubuntu-42:nginx-1.25.0 # chmod +x ./objs/nginx
root@ubuntu-42:nginx-1.25.0 # cp ./objs/nginx /usr/sbin/
root@ubuntu-42:nginx-1.25.0 # nginx -v
nginx version: nginx/1.25.0
#为了避免版本兼容性相关的问题，删除默认开启的功能模块
root@ubuntu-42:nginx-1.25.0 # rm -rf /etc/nginx/modules-enabled/*
root@ubuntu-42:~ # nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

#新旧双版本共存
#虽然nginx执行文件已经升级，但是目前承载的web服务依然是旧的版本
#向旧版本的主的master进程发起信号
root@ubuntu-42:~ # kill -USR2 2408
root@ubuntu-42:~ # pstree -p | grep nginx
           |-nginx(2408)-+-nginx(12423)-+-nginx(12424)
           |             |              `-nginx(12425)
           |             |-nginx(2409)
           |             `-nginx(2410)
root@ubuntu-42:~ # ps aux | grep nginx
root        2408  0.0  0.1  59304  2800 ?        Ss   13:00   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    2409  0.0  0.3  60100  6512 ?        S    13:00   0:00 nginx: worker process
www-data    2410  0.0  0.3  60100  6512 ?        S    13:00   0:00 nginx: worker process
root       12423  0.0  0.3  11060  6912 ?        S    13:48   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12424  0.0  0.2  12752  4408 ?        S    13:48   0:00 nginx: worker process
www-data   12425  0.0  0.2  12752  4408 ?        S    13:48   0:00 nginx: worker process
root       12445  0.0  0.1   6544  2304 pts/3    S+   13:50   0:00 grep --color=auto nginx
#查看每个进程专用的pid文件
root@ubuntu-42:~ # ls /var/run/nginx.pid* -l
-rw-r--r-- 1 root root 6 Dec  4 13:48 /var/run/nginx.pid
-rw-r--r-- 1 root root 5 Dec  4 12:28 /var/run/nginx.pid.oldbin
#查看新master 进程 PID
root@ubuntu-42:~ # cat /var/run/nginx.pid
12423
#查看旧master 进程 PID
root@ubuntu-42:~ # cat /var/run/nginx.pid.oldbin 
2408
#虽然生成了新的nginx主进程，但是提供web访问的依然是旧的版本
root@ubuntu-42:~ # curl 127.1 -I
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 04 Dec 2024 13:53:17 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 04 Dec 2024 12:27:05 GMT
Connection: keep-alive
ETag: "67504a99-267"
Accept-Ranges: bytes

```

```shell
#平滑升级
#对服务端进行信号传递，关闭旧worker进程
root@ubuntu-42:~ # kill -WINCH 2408
root@ubuntu-42:~ # ps aux | grep nginx
root        2408  0.0  0.1  59304  2800 ?        Ss   13:00   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
root       12423  0.0  0.3  11060  6912 ?        S    13:48   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12424  0.0  0.2  12752  4408 ?        S    13:48   0:00 nginx: worker process
www-data   12425  0.0  0.2  12752  4408 ?        S    13:48   0:00 nginx: worker process
root       12482  0.0  0.1   6544  2304 pts/3    S+   13:56   0:00 grep --color=auto nginx
#结果显示：
 	#处于wget工作状态的旧worker进程依然存在，而没有工作的旧worker进程已经被丢弃了
#现在已经是新版在工作了
root@ubuntu-42:~ # curl 127.1 -I
HTTP/1.1 200 OK
Server: nginx/1.25.0
Date: Wed, 04 Dec 2024 13:57:09 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 04 Dec 2024 12:27:05 GMT
Connection: keep-alive
ETag: "67504a99-267"
Accept-Ranges: bytes
#再次查看nginx的进程状态
root@ubuntu-42:~ # ps aux | grep nginx
root        2408  0.0  0.1  59304  2800 ?        Ss   13:00   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
root       12423  0.0  0.3  11060  6912 ?        S    13:48   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12424  0.0  0.2  12752  4792 ?        S    13:48   0:00 nginx: worker process
www-data   12425  0.0  0.2  12752  4920 ?        S    13:48   0:00 nginx: worker process
root       12487  0.0  0.1   6544  2304 pts/3    S+   13:58   0:00 grep --color=auto nginx
#结果显示：
 	#所有的旧 woker 进程全部退出，但是旧的master进程依然处于存活状态。
#如果业务没有问题，则可以继续发送信号，退出旧的 master 进程，至此升级完成
#最后发送信号退出旧的 master 进程，至此升级完成
root@ubuntu-42:~ # kill -QUIT 2408
root@ubuntu-42:~ # ps aux | grep nginx
root       12423  0.0  0.3  11060  6912 ?        S    13:48   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12424  0.0  0.2  12752  4792 ?        S    13:48   0:00 nginx: worker process
www-data   12425  0.0  0.2  12752  4920 ?        S    13:48   0:00 nginx: worker process
root       12493  0.0  0.1   6544  2304 pts/3    S+   14:01   0:00 grep --color=auto nginx
```

```shell
#平滑回退
root@ubuntu-42:~ # ps aux | grep nginx
root        2408  0.0  0.1  59304  2800 ?        Ss   13:09   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    2409  0.0  0.3  60100  6512 ?        S    13:09   0:00 nginx: worker process
www-data    2410  0.0  0.3  60100  6512 ?        S    13:09   0:00 nginx: worker process
root       12423  0.0  0.3  11060  6912 ?        S    13:57   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12424  0.0  0.2  12752  4408 ?        S    13:57   0:00 nginx: worker process
www-data   12425  0.0  0.2  12752  4408 ?        S    13:57   0:00 nginx: worker process
root       13213  0.0  0.1   6544  2304 pts/5    S+   14:03   0:00 grep --color=auto nginx
#快速进入到切换新主阶段
#平滑升级到 1.25.0版本
root@ubuntu-42:~ # kill -WINCH 2408
root@ubuntu-42:~ # ps aux | grep nginx
root        2408  0.0  0.1  59304  2800 ?        Ss   13:09   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    2410  0.0  0.3  60100  6512 ?        S    13:09   0:00 nginx: worker process is shutting down
root       12423  0.0  0.3  11060  6912 ?        S    13:57   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12424  0.0  0.2  12752  4408 ?        S    13:57   0:00 nginx: worker process
www-data   12425  0.0  0.2  12752  4408 ?        S    13:57   0:00 nginx: worker process
root       13431  0.0  0.1   6544  2304 pts/5    S+   14:09   0:00 grep --color=auto nginx
#查看当前nginx的版本效果
root@ubuntu-42:~ # curl 127.1 -I
HTTP/1.1 200 OK
Server: nginx/1.25.0
Date: Wed, 04 Dec 2024 14:10:31 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 04 Dec 2024 12:27:05 GMT
Connection: keep-alive
ETag: "67504a99-267"
Accept-Ranges: bytes

#经过一段时间运行后，发现 1.25.0 的新版本出现了问题，那么就需要版本回滚
#查看可执行文件
root@ubuntu-42:~ # ls /usr/sbin/nginx* -l
-rwxr-xr-x 1 root root 6584000 Dec  4 13:02 /usr/sbin/nginx
-rwxr-xr-x 1 root root 1313752 Sep 10 13:27 /usr/sbin/nginx-1.24
#切换nginx版本
#查看效果
root@ubuntu-42:~ # mv /usr/sbin/nginx /usr/sbin/nginx-1.25
root@ubuntu-42:~ # mv /usr/sbin/nginx-1.24 /usr/sbin/nginx
root@ubuntu-42:~ # nginx -v
nginx version: nginx/1.24.0 (Ubuntu)
#目前依然是新版本的nginx提供服务访问
root@ubuntu-42:~ # curl -I 127.1 -s | grep Server
Server: nginx/1.25.0
#向当前的nginx主进程发起 HUP信号
#发送信号，拉起旧的 worker 进程
root@ubuntu-42:~ # ps aux | grep nginx
root        2408  0.0  0.1  59304  2800 ?        Ss   13:09   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
root       12423  0.0  0.3  11060  6912 ?        S    13:57   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   12424  0.0  0.2  12752  4792 ?        S    13:57   0:00 nginx: worker process
www-data   12425  0.0  0.2  12752  4920 ?        S    13:57   0:00 nginx: worker process
www-data   13443  0.0  0.3  60100  6000 ?        S    14:14   0:00 nginx: worker process
www-data   13444  0.0  0.3  60100  6128 ?        S    14:14   0:00 nginx: worker process
#结果显示：
 	#目前的nginx旧master 又拉起了两个nginx的worker进程
#此时还是新master在工作
root@ubuntu-42:~ # curl 127.1 -I
HTTP/1.1 200 OK
Server: nginx/1.25.0
Date: Wed, 04 Dec 2024 14:16:41 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 04 Dec 2024 12:27:05 GMT
Connection: keep-alive
ETag: "67504a99-267"
Accept-Ranges: bytes
#继续发送信号，优雅的退出新版的 master 进程
root@ubuntu-42:~ # kill -QUIT 12423
root@ubuntu-42:~ # ps aux | grep nginx
root        2408  0.0  0.1  59304  2800 ?        Ss   13:09   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data   13443  0.0  0.3  60100  6000 ?        S    14:14   0:00 nginx: worker process
www-data   13444  0.0  0.3  60100  6128 ?        S    14:14   0:00 nginx: worker process
root       13458  0.0  0.1   6544  2304 pts/5    S+   14:17   0:00 grep --color=auto nginx
#结果显示：
 	#目前的nginx已经将新版本的nginx退出了，回退到了旧版本
root@ubuntu-42:~ # curl 127.1 -I
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 04 Dec 2024 14:18:59 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 04 Dec 2024 12:27:05 GMT
Connection: keep-alive
ETag: "67504a99-267"
Accept-Ranges: bytes
```

## 扩展nginx模块

### 扩展模块-echo

```
添加模块逻辑：
0 准备nginx源码
 	- 如果本地没有源码的话，可以从官网下载一个同版本的nginx源码
1 下载模块
2 编译安装
3 测试效果
```

```shell
0、准备工作
#查看当前版本
root@ubuntu-42:nginx-1.24.0 # nginx -V
nginx version: nginx/1.24.0
#获取相同版本信息
mkdir /data/softs -p;cd /data/softs	
wget http://nginx.org/download/nginx-1.24.0.tar.gz
#解压文件
tar xf nginx-1.24.0.tar.gz
1、下载模块
cd /data/softs
wget https://github.com/openresty/echo-nginx-module/archive/refs/tags/v0.63.tar.gz
#解压文件
tar xf v0.63.tar.gz
mv echo-nginx-module-0.63 /usr/local/
2、编译安装
cd /data/softs/nginx-1.24.0
nginx -V
./configure --with-cc-opt='-g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -........ --add-module=/usr/local/echo-nginx-module-0.63
#编译文件
make
#新旧版本nginx
nginx -v
	nginx version: nginx/1.24.0 (Ubuntu)
./objs/nginx -V
	nginx version: nginx/1.24.0
#新旧版本的执行文件
ls ./objs/nginx /usr/sbin/nginx -l
	-rwxr-xr-x 1 root root 6997664 Dec  5 01:10 ./objs/nginx
	-rwxr-xr-x 1 root root 1313752 Sep 10 13:27 /usr/sbin/nginx
#备份旧执行文件
mv /usr/sbin/nginx /usr/sbin/nginx-1.24
#应用新的nginx
chmod +x ./objs/nginx
cp ./objs/nginx /usr/sbin/
3、检查应用
nginx -v
	nginx version: nginx/1.24.0
4、功能使用
root@ubuntu-42:nginx # cat /etc/nginx/conf.d/vhost.conf
server {
  listen 80;
  location /echo{
  
    echo "request_uri: \$request_uri";
  }
}
root@ubuntu-42:nginx # rm -rf /etc/nginx/sites-enabled/default 
root@ubuntu-42:nginx # systemctl restart nginx
root@ubuntu-42:nginx # curl 10.0.0.42/echo
request_uri: \/echo

```

### 扩展模块-vts

```shell
#获取模块
cd /data/softs
wget https://github.com/vozlt/nginx-module-vts/archive/refs/tags/v0.2.2.tar.gz
tar xf v0.2.2.tar.gz -C /usr/local/
#编译安装
#生成配置文件
cd /data/softs/nginx
./configure ... --add-module=/usr/local/nginx-module-vts-0.2.2/
#编译安装
make && make install
#检测效果
nginx -v
```



## 全局配置

```shell
root@ubuntu-42:softs # ll /etc/nginx/
total 104
drwxr-xr-x   2 root root 4096 Dec  5 01:31 conf.d/
				#子配置文件目录，在主配置文件中被包含，默认为空
-rw-r--r--   1 root root 1125 Nov 30  2023 fastcgi.conf
				#FastCGI配置文件，定义相关配置项，被引用
-rw-r--r--   1 root root 1077 Dec  5 01:47 fastcgi.conf.default
				#包含默认的 FastCGI 配置。
-rw-r--r--   1 root root 1055 Nov 30  2023 fastcgi_params
				#FastCGI配置文件，定义相关配置项，被引用
-rw-r--r--   1 root root 1007 Dec  5 01:47 fastcgi_params.default
-rw-r--r--   1 root root 2837 Dec  5 01:47 koi-utf
				#用于转换KOI8-R编码的字符集到UTF-8
-rw-r--r--   1 root root 2223 Dec  5 01:47 koi-win
				#用于转换KOI8-U编码的字符集到UTF-8
-rw-r--r--   1 root root 5465 Nov 30  2023 mime.types
				#包含文件扩展名和相应MIME类型的映射
-rw-r--r--   1 root root 5349 Dec  5 01:47 mime.types.default
drwxr-xr-x   2 root root 4096 Sep 10 13:27 modules-available/
				#模块配置文件目录
drwxr-xr-x   2 root root 4096 Dec  4 13:03 modules-enabled/
				#当前生效的模块配置文件目录，用软链接指向真正的文件
-rw-r--r--   1 root root 1661 Dec  5 02:11 nginx.conf
				#主配置文件
-rw-r--r--   1 root root 2656 Dec  5 01:47 nginx.conf.default
				#是一个默认配置的备份。不要直接修改它；编辑主配置文件 nginx.conf
-rw-r--r--   1 root root  180 Nov 30  2023 proxy_params
				#反向代理配置文件
-rw-r--r--   1 root root  636 Nov 30  2023 scgi_params
				#SCGI配置文件，SCGI和FastCGI类似，都是一种协议
-rw-r--r--   1 root root  636 Dec  5 01:47 scgi_params.default
drwxr-xr-x   2 root root 4096 Dec  4 12:27 sites-available/
				#所有域名配置文件
drwxr-xr-x   2 root root 4096 Dec  5 01:33 sites-enabled/
				#生效的域名配置文件，用软连接指向site-available中的文件
drwxr-xr-x   2 root root 4096 Dec  4 12:27 snippets/
				#通用的配置片段目录，需要时被引用
-rw-r--r--   1 root root  664 Nov 30  2023 uwsgi_params
				#配置 Nginx uWSGI 模块的默认参数的文件
-rw-r--r--   1 root root  664 Dec  5 01:47 uwsgi_params.default
-rw-r--r--   1 root root 3610 Dec  5 01:47 win-utf
				#编码转换映射转化文件
```

配置文件格式

```
- 配置文件由指令与指令块构成
- 每条指令以 ; 分号结尾，指令与值之间以空格符号分隔
- 可以将多条指令放在同一行，用分号分隔即可，但可读性差，不推荐
- 指令块以{ }大括号将多条指令组织在一起，且可以嵌套指令块
- include 语句允许组合多个配置文件以提升可维护性
- 使用 # 号添加注释，提高可读性
- 使用 $ 符号使用变量
- 变量分为 Nginx 内置变量和自定义变量，自定义变量用 set 指令定义
- 部分指令的参数支持正则表达式
```



```shell
#启动Nginx工作进程的用户和组
user www-data;
#启动Nginx工作进程的数量,一般设为和CPU核心数相同
worker_processes [number | auto];
#将Nginx工作进程绑定到指定的CPU核心，默认Nginx是不进行进程绑定的，绑定并不是意味着当前nginx进程独占以一核心CPU，但是可以保证此进程不会运行在其他核心上，这就极大减少了nginx的工作进程在不同的cpu核心上的来回跳转，减少了CPU对进程的资源分配与回收以及内存管理等，因此可以有效的提升nginx服务器的性能。
worker_cpu_affinity 01 10 | auto ;
#pid文件保存路径
pid /run/nginx.pid;
#worker进程的优先级，工作进程优先级，-20~19 （值越小优先级越高）
worker_priority 0;
#错误日志记录配置，语法：error_log file [debug | info | notice | warn | error | crit | alert | emerg]
#error_log logs/error.log;
#error_log logs/error.log notice;
error_log /var/log/nginx/error.log;
#所有worker进程能打开的文件数量上限,包括:Nginx的所有连接（例如与代理服务器的连接等），而不仅仅是与客户端的连接,另一个考虑因素是实际的并发连接数不能超过系统级别的最大打开文件数的限制.最好与ulimit -n 或者limits.conf的值保持一致,默认不限制
worker_rlimit_nofile 65536;
#前台运行Nginx服务用于测试、或者以容器运行时，需要设为off
daemon off;
#是否开启Nginx的master-worker工作模式，仅用于开发调试场景,默认为on
master_process off|on;

include /etc/nginx/modules-enabled/*.conf;
events {
        worker_connections 768;
        	#设置单个工作进程的最大并发连接数，默认512，生产建议根据性能修改更大的值。
        use epoll; 
        	#使用epoll事件驱动，Nginx支持众多的事件驱动，比如:select、poll、epoll，只能设置在events模块中设置。
        accept_mutex on; 
        	#mutex互斥为on表示同一时刻一个请求轮流由worker进程处理,而防止被同时唤醒所有worker,避免多个睡眠进程被唤醒的设置，可以避免多个 worker 进程竞争同一连接而导致性能下降,也可以提高系统的稳定性,默认为off，新请求会唤醒所有worker进程,此过程也称为"惊群"，在高并发的场景下多个worker进程可以各自同时接受多个新的连接请求，如果是多CPU和worker进程绑定,就可以提高吞吐量。
        multi_accept on; 
        	#on时Nginx服务器的每个工作进程可以同时接受多个新的网络连接，此指令默认为off，即默认为一个工作进程只能一次接受一个新的网络连接，打开后几个同时接受多个。建议设置为on。
}
http {
        sendfile on;
        tcp_nopush on;
        	#开启sendfile的情况下，合并请求后统一发送给客户端,必须开启sendfile
        types_hash_max_size 2048;
        	#设置 MIME 类型哈希表的最大大小。
			#Nginx 使用 MIME 类型哈希表快速确定文件扩展名与 MIME 类型的映射。此配置可以优化性能，特别是 MIME 类型较多时。
        include /etc/nginx/mime.types;
        	#引入 MIME 类型定义文件。
			#这个文件包含了文件扩展名与 MIME 类型的映射，例如 .html 映射为 text/html，.jpg 映射为 image/jpeg。
			#用于 default_type 和 types 指令来确定响应的 Content-Type 头。
        default_type application/octet-stream;
        	#当无法匹配到 MIME 类型时，Nginx 会将文件的类型设置为 application/octet-stream。
			#适合用来处理未知文件类型，提示浏览器将其作为二进制文件下载。
        ssl_prefer_server_ciphers on;
        	#启用后，优先使用服务器定义的加密套件，而不是客户端的。
			#提高安全性，确保服务器控制使用的加密算法。
        access_log /var/log/nginx/access.log;
        	#指定访问日志文件的位置。
			#Nginx 将所有请求的相关信息（如 IP 地址、时间、请求路径）记录到该日志文件中。
        gzip on;
        	#开启 Gzip 压缩功能。
			#Nginx 会压缩响应数据，从而减少传输的体积，提升网页加载速度。
        include /etc/nginx/conf.d/*.conf;
        	#加载 /etc/nginx/conf.d/ 目录下所有以 .conf 结尾的文件。
			#常用于存放模块配置或虚拟主机配置。
        include /etc/nginx/sites-enabled/*;
        	#加载 /etc/nginx/sites-enabled/ 目录下的所有文件。
			#通常用于管理站点的虚拟主机配置文件。通过在 sites-available/ 和 sites-enabled/ 之间建立符号链接，可以灵活启用或禁用站点。
}
```

## 文件描述符 & nginx最大并发连接数

查看systemd的默认限制

```shell
root@ubuntu-42:~ # systemctl show | grep FILE
DefaultLimitNOFILE=524288			#默认硬限制
DefaultLimitNOFILESoft=1024			#默认软限制，软限制不能超过硬限制
```

定制nginx的文件描述符

```shell
设置 worker 进程文件描述符数量
root@ubuntu42:~# cat /etc/nginx/nginx.conf
worker_rlimit_nofile 60000;

关于nginx的master进程文件描述符的限制
root@ubuntu24:~# cat /lib/systemd/system/nginx.service
...
LimitNOFILE=10000:50000 #soft 10000， hard 50000
```

## http配置

### 修改nginx的配置，增加字符集属性

```shell
root@ubuntu42:~# cat /etc/nginx/nginx.conf
http {
 charset utf8;
root@ubuntu-43:nginx # curl -s -I 127.1 
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Thu, 05 Dec 2024 03:48:49 GMT
Content-Type: text/html; charset=utf8
Content-Length: 615
Last-Modified: Wed, 04 Dec 2024 11:04:13 GMT
Connection: keep-alive
ETag: "6750372d-267"
Accept-Ranges: bytes
```

### 隐藏属性信息

```shell
root@ubuntu42:~# cat /etc/nginx/nginx.conf
http {
	...
 	server_tokens off;
 	...
 	}

root@ubuntu-42:nginx # curl -s -I 127.1 
HTTP/1.1 200 OK
Server: nginx
Date: Thu, 05 Dec 2024 03:51:56 GMT
Content-Type: text/html; charset=utf8
Content-Length: 615
Last-Modified: Wed, 04 Dec 2024 11:04:13 GMT
Connection: keep-alive
ETag: "6750372d-267"
Accept-Ranges: bytes
```

### Server & Location

**server配置**

```shell
ssl_certificate  file; 				#当前虚拟主机证书和CA机构证书，放在一个文件中，默认值为空
ssl_certificate_key file; 			#私钥文件路径，默认为空
ssl_session_cache off|none|[builtin[:size]] [shared:name:size];
 									#是否启用 ssl 缓存，默认none, off 不使用 ssl 缓存
 									#none 通知客户端，当前支持 ssl session cache
									#builtin[:size]使用 openssl 内建缓存,为每个worker 进程私有
									#shared:name:size woker 进程共享缓存
ssl_session_timeout N; 				#客户端连接可以复用ssl session cache中缓存的有效时长，默认5m
ssl_ciphers ciphers; 				#指定 SSL/TLS 加密算法，默认值 HIGH:!aNULL:!MD5;
 									#默认值表示加密优先级高，排除匿名加密算法，排除 MD5 加密算法
ssl_prefer_server_ciphers on|off; 	#默认off，表示在SSL/TLS握手时优先使用客户端加密套件，
 									#on表示优先使用服务端套件
root path; 							#当前虚拟主机网页文件目录，写在location中，文件路径是root+location
index file ...; 					#当前虚拟主机的默认页面，可以写多个，按顺序生效，默认 index.html
server_name name ...; 				#虚拟主机域名，可以写多个值，客户端请求头中的 Host 字段会与此处匹配
 									#默认值是""，用来匹配请求头中 Host 字段值为空的请求
 									#还可以写成 _,@ 等用来表示无效域名，和listen中的default_server配合

```

**location配置段**

```
配置样式
    location [ = | ~ | ~* | ^~ ] uri { ... }
    location @name { ... }
```

```shell
配置段匹配规则
    = 		#用于标准uri前，需要请求字串与uri精确匹配，区分大小写
    ^~ 		#用于标准uri前，表示包含正则表达式，匹配以特定字符串开始，区分大小写
    ~ 		#用于标准uri前，表示包含正则表达式，区分大小写
    ~* 		#用于标准uri前，表示包含正则表达式，不区分大写
    /str 	#不带符号 匹配以 str 开头的 uri，/ 也是一个字符
    \ 		#用于标准uri前，表示包含正则表达式并且转义字符，可以将 . * ?等转义为普通符号
    @name 	#定义
```

```shell
常用配置项
alias path; 		#定义路径别名，把访问路径重新映射到其它目录
root path; 			#定义网站在文件系统上的家目录，server_root/location_root
index file ...; 	#当前配置段的默认页面，可以写多个，依次生效，默认 index.html
return code [text]|code URL|URL; 	#直接给客户端返回状态码+字符串或URL
proxy_pass URL; 	#设置反向代理，将符合规则的请求转发到后端服务器进行处理
fastcgi_pass address; 				#通过fastCGI协议将请求转发到后端服务器进行处理
fastcgi_index name; 				#fastCGI协议的默认资源，默认为空
fastcgi_param parameter value [if_not_empty];	#设置传给fastCGI服务器的参数 key val 格式
deny address|CIDR|unix:|all; 		#拒绝访问的客户端，黑名单
allow address|CIDR|unix:|all; 		#允许访问的客户端，白名单，可以是具体IP 
try_files file ... uri|file ... =code; 			#按顺序查询文件是否存在，并返回匹配到的信息
```

### server实践

```
定制应用页面
mkdir /data/server/nginx/web{1..3} -p
echo "nginx web1" > /data/server/nginx/web1/index.html
echo "nginx web2" > /data/server/nginx/web2/index.html
echo "nginx web3" > /data/server/nginx/web3/index.html

ip a a 10.0.0.186/24 dev ens33
ip a a 10.0.0.187/24 dev ens33
ip a a 10.0.0.188/24 dev ens33
```

#### 基于端口站点实践

```shell
root@ubuntu-43:conf # cat /apps/nginx/conf/conf.d/vhost.conf 
server {
  listen 80;
  root /data/server/nginx/web1;
}
server {
  listen 81;
  root /data/server/nginx/web2;
}
server {
  listen 82;
  root /data/server/nginx/web3;
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service
```

#### 基于域名实现多server

```shell
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  server_name www.a.com;
  root /data/server/nginx/web1;
}

server {
  listen 80;
  server_name www.b.com;
  root /data/server/nginx/web2;
}

server {
  listen 80;
  server_name www.c.com;
  root /data/server/nginx/web3;
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # rndc flush
root@ubuntu-43:conf # systemctl restart nginx.service
```

### root & alias

```
root 和 alias 所起的作用都是指定响应请求所用文件的路径，只是他们有些许的区别
 	root 表示 location 匹配内容的相对路径
 	alias 表示 一个绝对路径,而且必须以"/"结尾

www.a.com
/data/server/nginx/web1/root/a/c/d/e/f/h/index.html
root /data/server/nginx/web1
www.a.com/root/a/c/d/e/f/h/index.html

/app/
alias /data/server/nginx/web1/root/a/c/d/e/f/h/
www.a.com/app/index.html
```

```shell
root实践
mkdir /data/server/nginx/web1/dir{1,2}
echo "nginx web1 dir1" > /data/server/nginx/web1/dir1/index.html
echo "nginx web1 dir2" > /data/server/nginx/web1/dir2/index.html
echo "nginx web1 filea" > /data/server/nginx/web1/afile
root@ubuntu-43:~ # tree /data/server/nginx/web1
/data/server/nginx/web1
├── afile
├── dir1
│   └── index.html
├── dir2
│   └── index.html
└── index.html
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /dir1/ {
      root /data/server/nginx/web1;
  }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service
root@ubuntu-43:conf # curl 10.0.0.43
nginx web1
root@ubuntu-43:conf # curl 10.0.0.43/dir1/
nginx web1 dir1
root@ubuntu-43:conf # curl 10.0.0.43/dir2/
nginx web1 dir2
root@ubuntu-43:conf # curl 10.0.0.43/afile
nginx web1 filea
```

```shell
alias
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /web2/ {
      alias /data/server/nginx/web2/;
  }
  location /web3/ {
      alias /data/server/nginx/web3/;
  }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service
root@ubuntu-43:conf # curl 10.0.0.43
nginx web1
root@ubuntu-43:conf # curl 10.0.0.43/web2/
nginx web2
root@ubuntu-43:conf # curl 10.0.0.43/web3/
nginx web3
```

### location优先级

```shell
配置段匹配规则
    = 			#用于标准uri前，需要请求字串与uri精确匹配，区分大小写
    ^~ 			#用于标准uri前，表示包含正则表达式，匹配以特定字符串开始，区分大小写
    ~ 			#用于标准uri前，表示包含正则表达式，区分大小写
    ~* 			#用于标准uri前，表示包含正则表达式，不区分大写
    /str 		#不带符号 匹配以 str 开头的 uri，/ 也是一个字符
    \ 			#用于标准uri前，表示包含正则表达式并且转义字符，可以将 . * ?等转义为普通符号
    @name 		#定义
   【空格】		#如果没有其他匹配规则，则使用默认匹配
    
规则优先级
= > ^~ > ~* > ~ > /str  
```

```shell
root@ubuntu-43:~ # cat /apps/nginx/conf/conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  #精确匹配 /
  location = / {
default_type text/html;
return 200 'location = /\n';
  }
  #匹配以 /开始
  location / {
default_type text/html;
return 200 'location /\n';
  }
  #匹配以 /document/ 开始，区分大小写
  location /documents/ {
default_type text/html;
return 200 'location /documents/\n';
  }
  #匹配以 /images/ 开始，区分大小写
  location ^~ /images/ {
default_type text/html;
return 200 'location ^~ /images/\n';
  }
  #匹配以 .gif .jpg .jpeg 结尾，不区分大小写
  location ~* \.(gif|jpg|jpeg)$ {
default_type text/html;
return 200 'location ~* \.(gif|jpg|jpeg)\n';
  }
}

root@ubuntu-43:conf # curl 10.0.0.43
location = /
root@ubuntu-43:conf # curl 10.0.0.43/
location = /
root@ubuntu-43:conf # curl 10.0.0.43/index.html
location /
root@ubuntu-43:conf # curl 10.0.0.43/abc
location /
root@ubuntu-43:conf # curl 10.0.0.43/documents
location /
root@ubuntu-43:conf # curl 10.0.0.43/documents/
location /documents/
root@ubuntu-43:conf # curl 10.0.0.43/images
location /
root@ubuntu-43:conf # curl 10.0.0.43/images/ab
location ^~ /images/
root@ubuntu-43:conf # curl 10.0.0.43/images/ab.jpg
location ^~ /images/
root@ubuntu-43:conf # curl 10.0.0.43/images/ab.DIF
location ^~ /images/
root@ubuntu-43:conf # curl 10.0.0.43/123.jpg
location ~* \.(gif|jpg|jpeg)
root@ubuntu-43:conf # curl 10.0.0.43/image/abc.jpg
location ~* \.(gif|jpg|jpeg)
```

### @重定向实践

```shell
root@ubuntu-43:~ # cat /apps/nginx/conf/conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  #精确匹配 /
  location = / {
        default_type text/html;
        return 200 'location = /\n';
  }
  error_page 404 @error;
  location @error {
        default_type text/html;
        return 200 "page not found\n";
  }
}

root@ubuntu-43:~ # nginx -t
root@ubuntu-43:~ # systemctl restart nginx.service
root@ubuntu-43:conf # curl 10.0.0.43
location = /
root@ubuntu-43:conf # curl 10.0.0.43/aa
page not found
```

### 四层控制

#### 黑白名单属性

```shell
访问控制属性：
deny address|CIDR|unix:|all; 		# 拒绝访问的客户端，黑名单，
 									# 可以是具体IP，网段，socket(1.5.1版本以上)，所有
allow address|CIDR|unix:|all; 		# 允许访问的客户端，白名单，
 									# 可以是具体IP，网段，socket(1.5.1版本以上)，所有
注意：
 	deny 和 allow 指令同时存在时，按加载顺序生效 
 	作用域：http, server, location, limit_except 
```

##### 黑白名单实践

```shell
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  location /web1/ {
    alias /data/server/nginx/web1/;
    # 只允许 127.1 访问，但实际上并不起作用，因为没有配套的deny
    allow 127.0.0.1;
  }
  location /web2/ {
    alias /data/server/nginx/web2/;
    # 所有客户端都不能访问
    deny all;
    # deny 写在前面了,所以 allow 无效
    allow 127.0.0.1;
  }
  location /web3/ {
    alias /data/server/nginx/web3/;
    allow 127.0.0.1;    # 放行127.1
    deny 10.0.0.43;     # 拒绝 10.0.0.42
    allow 10.0.0.0/24;  # 放行 10.0.0.0/24 所有IP，但 10.0.0.13 在上一条被拒绝
    deny all;           # 拒绝所有，兜底规则
  }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service
root@ubuntu-43:conf # curl 10.0.0.43/web1/
nginx web1
root@ubuntu-43:conf # curl 10.0.0.43/web2/ -I
HTTP/1.1 403 Forbidden
root@ubuntu-42:nginx # curl 10.0.0.43/web2/ -I
HTTP/1.1 403 Forbidden
root@ubuntu-43:conf # curl 10.0.0.43/web3/ -I
HTTP/1.1 403 Forbidden
root@ubuntu-43:conf # curl 127.1/web3/
nginx web3
root@ubuntu-42:nginx # curl 10.0.0.43/web3/
nginx web3
```

#### 身份认证

```shell
身份验证属性
    auth_basic string|off; 		# 身份验证时对话框上的提示信息，新版浏览器几乎不支持,
   								# 默认off，即不启用验证功能
    auth_basic_user_file file; 	# 指定身份认证的用户名和密码的文件路径，
   								# 密码要使用 htpasswd 命令生成
注意：
 作用域 http, server, location, limit_except
```



##### 身份认证实践

```shell
#安装密码工具
rocky
[root@rocky9 ~]# yum -y install httpd-tools
ubuntu
root@ubuntu-43:conf # apt install apache2-utils -y
#创建用户名和密码
#交互式新增用户密码
root@ubuntu-43:conf # htpasswd -c /apps/nginx/conf/conf.d/httpuser tom
New password: 
Re-type new password: 
Adding password for user tom
#非交互式新增用户密码
root@ubuntu-43:conf # htpasswd -b /apps/nginx/conf/conf.d/httpuser jerry 123456
Adding password for user jerry
#查看文件，在生产环境中，基于安全考虑，可以将文件设为隐藏文件，修改属主属组，并配置读写权限
root@ubuntu-43:conf # cat conf.d/httpuser 
tom:$apr1$IwbWqOMw$8tEJjJQMyLHS9kYpgcJrg0
jerry:$apr1$WK1wDGtF$h1ahHESGAfH.hMVV4oc72.
#配置实践
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  location /web1/ {
    alias /data/server/nginx/web1/;
    auth_basic "请输入你的名字";
    # 一旦登录后，再访问其它资源的时候，无需重复验证
    auth_basic_user_file conf.d/httpuser;
  }
  location /web2/ {
    alias /data/server/nginx/web2/;
  }
  location /web3/ {
    alias /data/server/nginx/web3/;
  }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service
root@ubuntu-43:conf # curl 10.0.0.43/web1/ -I
HTTP/1.1 401 Unauthorized
Server: nginx
Date: Thu, 05 Dec 2024 08:13:51 GMT
Content-Type: text/html; charset=utf8
Content-Length: 172
Connection: keep-alive
WWW-Authenticate: Basic realm="请输入你的名字"

root@ubuntu-43:conf # curl http://tom:123456@10.0.0.43/web1/
nginx web1
root@ubuntu-43:conf # curl -u jerry:123456 --basic 10.0.0.43/web1/
nginx web1
```

### 错误页面

#### error_page属性

```shell
错误配置页面
 	error_page code ... [=[response]] uri; 	# 定义指定状态码的错误页面，可以多个状态码共用一个资源
注意：
 	作用域 http, server, location, if in location 
```

##### error_page实践

```shell
#定制错误页面
root@ubuntu-43:conf # mkdir -p /data/server/nginx/errors/
root@ubuntu-43:conf # vim /data/server/nginx/errors/error.html
root@ubuntu-43:conf # cat /data/server/nginx/errors/error.html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>错误页面</title>
    <style>
        body, html {
            height: 100%;
            margin: 0;
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: #f0f0f0;
            color: #333;
        }
        .error-container {
            text-align: center;
            padding: 20px;
            border: 1px solid #ccc;
            border-radius: 8px;
            background-color: #fff;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 300px;
        }
        .error-code {
            font-size: 3em;
            margin-bottom: 10px;
            color: #d9534f;
        }
        .error-message {
            font-size: 1.5em;
            margin-bottom: 20px;
            font-weight: bold;
        }
        .error-description {
            font-size: 1em;
            line-height: 1.5;
        }
        .error-description a {
            color: #d9534f;
            text-decoration: none;
            border-bottom: 1px solid #d9534f;
        }
        .error-description a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <div class="error-container">
        <div class="error-code">404</div>
        <div class="error-message">页面未找到</div>
        <div class="error-description">
            很抱歉，您访问的页面不存在。请检查URL是否正确，或返回<a href="/">首页</a>。
        </div>
    </div>
</body>
</html>
#定制自定义指定状态码的错误页面配置
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  error_page 400 404 502 503 504 error.html; #自定义指定状态码的错误页面
  location = /error.html {
    root /data/server/nginx/errors/;
  }
  location = /error2.html {
    # 通过302 重定向到 index.hmtl
    error_page 400 404 =302 /index.html;
  }
}

root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # curl 10.0.0.43/error.html -I
HTTP/1.1 200 OK
Server: nginx
Date: Thu, 05 Dec 2024 08:29:13 GMT
Content-Type: text/html; charset=utf8
Content-Length: 1696
Last-Modified: Thu, 05 Dec 2024 08:18:45 GMT
Connection: keep-alive
ETag: "675161e5-6a0"
Accept-Ranges: bytes

root@ubuntu-43:conf # curl 10.0.0.43/error1.html -I
HTTP/1.1 302 Moved Temporarily
Server: nginx
Date: Thu, 05 Dec 2024 08:29:17 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: error.html

root@ubuntu-43:conf # curl 10.0.0.43/error2.html -I
HTTP/1.1 302 Moved Temporarily
Server: nginx
Date: Thu, 05 Dec 2024 08:29:20 GMT
Content-Type: text/html
Content-Length: 615
Connection: keep-alive
ETag: "6750372d-267"

root@ubuntu-43:conf # curl 10.0.0.43/error3.html -I
HTTP/1.1 302 Moved Temporarily
Server: nginx
Date: Thu, 05 Dec 2024 08:29:24 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: error.html
```

#### @重定向实现异常页面展示

```shell
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  error_page 404 @error;
  location @error {
        default_type text/html;
        return 200 "page not found\n";
  }
  error_page 503 @custom_503;
  location @custom_503 {
        add_header Content-Type text/plain;
        return 200 "<h1> Error Server Page</h1>\n";
    }
  # 模拟 503 错误（仅用于测试）
  location /test-503 {
        return 503;
  }
  
  location = /404 {
    # 临时重定向
    error_page 404 =302 http://www.baidu.com; 
  }
}

root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # curl 10.0.0.43/nihao
page not found
root@ubuntu-43:conf # curl 10.0.0.43/test-503
<h1> Error Server Page</h1>
root@ubuntu-43:conf # curl 10.0.0.43/404
<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
root@ubuntu-43:conf # curl 10.0.0.43/404 -I
HTTP/1.1 302 Moved Temporarily
Server: nginx
Date: Thu, 05 Dec 2024 08:34:59 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: http://www.baidu.com
```

### 日志管理

```shell
默认配置示例
 	access_log /var/log/nginx/access.log;
 	error_log /var/log/nginx/error.log;
配置解读：
 	error_log file [level]; # 指定保存错误日志的文件和记录级别，
 		# 如果不需要记录日志，值可以写在 /dev/null
		# 默认值 error_log logs/error.log error
注意：
 	作用域 main, http, mail, stream, server, location 
```

#### 定制错误日志

```shell
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx;
  error_log /apps/nginx/logs/error.log;
  
  # 此资源报错不记录日志
  location /test-503 {
    error_log /dev/null;
  }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # curl 10.0.0.43/ddddd
root@ubuntu-43:conf # curl 10.0.0.43/test-503
root@ubuntu-43:conf # curl 10.0.0.43/ssaaasasas
root@ubuntu-43:conf # cat /apps/nginx/logs/error.log | tail -2
2024/12/05 08:42:01 [error] 12962#0: *1 open() "/data/server/nginx/ddddd" failed (2: No such file or directory), client: 10.0.0.43, server: , request: "GET /ddddd HTTP/1.1", host: "10.0.0.43"
2024/12/05 08:42:17 [error] 12962#0: *3 open() "/data/server/nginx/ssaaasasas" failed (2: No such file or directory), client: 10.0.0.43, server: , request: "GET /ssaaasasas HTTP/1.1", host: "10.0.0.43"
```

#### 访问日志

```
属性格式：
 	access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
	access_log off;
属性解析：
 	path 日志路径
 	format 日志格式
 	buffer=size 定义 buffer 表示启用缓冲区，会异步落盘
 	gzip[=level] 启用gzip 压缩，此参数会自动启用buffer，默认缓冲区大小为 64k，默认压缩级别为1
 	flush=time 强制落盘时间频率，在缓冲区满了，或者此参数规定的时间到了都会定磁盘
 	[if=condition] 条件判断，返回true 才会记日志
 	off 表示不启用日志
 	默认值  access_log logs/access.log combined;
注意：
 	作用域 http, server, location, if in location, limit_except
```

#### 日志格式定制

```shell
日志格式
 	log_format name [escape=default|json|none] string ...;
属性解析：
 	name 日志格式名称，供 access_log 指令调用
 	[escape=default|json|none] 设置变量的字符转义，默认default
 	string ... 当前定义的日志格式要记录的具体内容
 	默认值 log_format combined "...";
 
注意：
 	作用域 http 

默认 log_format 内容
log_format combined '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';
                    

自定义信息变量
#定义日志格式常用的变量
$remote_addr 				#客户端的IP地址
$remote_user 				#使用 HTTP 基本身份验证时的远程用户
$time_local 				#服务器本地时间
$request 					#客户端请求的 HTTP 方法、URI 和协议
$status 					#服务器响应的状态码
$body_bytes_sent 			#发送给客户端的字节数，不包括响应头的大小
$http_referer 				#客户端跳转前的来源页面
$http_user_agent 			#客户端的用户代理字符串
$http_x_forwarded_for 		#通过代理服务器传递的客户端真实 IP 地址
$host 						#请求的主机头字段
$server_name 				#服务器名称
$request_time 				#请求处理时间
$upstream_response_time 	#从上游服务器接收响应的时间
$upstream_status 			#上游服务器的响应状态码
$time_iso8601 				#ISO 8601 格式的本地时间
$request_id 				#用于唯一标识请求的 ID
$ssl_protocol 				#使用的 SSL 协议
$ssl_cipher 				#使用的 SSL 加密算法
```

##### 定制日志格式实践

```shell
http {
...
    #自定义访问日志格式 - 字符串
        log_format basic '\$remote_addr - \$remote_user [\$time_local] "\$request" '
                  '\$status \$body_bytes_sent "\$http_referer" '
                  '"\$http_user_agent" "\$http_x_forwarded_for"';
    log_format json_basic '{"remote_addr": "\$remote_addr", '
                  '"remote_user": "\$remote_user", '
                  '"time_local": "\$time_local", '
                  '"request": "\$request", '
                  '"status": "\$status", '
                  '"body_bytes_sent": "\$body_bytes_sent", '
                  '"http_referer": "\$http_referer", '
                  '"http_user_agent": "\$http_user_agent", '
                  '"http_x_forwarded_for": "\$http_x_forwarded_for"}';


    access_log  /apps/nginx/logs/access.log;
...
}
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  access_log /apps/nginx/logs/${host}_access.log basic;
  
  location /web1/ {
    alias /data/server/nginx/web1/;
  }
  # 此资源记录json日志
  location /json{
    access_log /apps/nginx/logs/${host}_json_access.log json_basic;
    return 200 "json\n";
  }
  #不记录access log
  location /test{ 
    access_log off; 
    return 200 "test\n";
  }
}
#测试效果
curl 10.0.0.43
curl 10.0.0.43/web1/
curl 10.0.0.43/json
curl 10.0.0.43/test
root@ubuntu-43:conf # ls ../logs/
10.0.0.43_access.log  10.0.0.43_json_access.log  access.log  error.log  nginx.pid
root@ubuntu-43:conf # cat ../logs/10.0.0.43_json_access.log 
{"remote_addr": "\10.0.0.43", "remote_user": "\-", "time_local": "\05/Dec/2024:09:30:35 +0000", "request": "\GET /json HTTP/1.1", "status": "\200", "body_bytes_sent": "\5", "http_referer": "\-", "http_user_agent": "\curl/8.5.0", "http_x_forwarded_for": "\-"}
root@ubuntu-43:conf # cat ../logs/10.0.0.43_access.log 
\10.0.0.43 - \- [\05/Dec/2024:09:30:44 +0000] "\GET /web1/ HTTP/1.1" \200 \11 "\-" "\curl/8.5.0" "\-"
\10.0.0.43 - \- [\05/Dec/2024:09:30:48 +0000] "\GET / HTTP/1.1" \200 \615 "\-" "\curl/8.5.0" "\-"
```

### 状态检测

#### try_files

```
针对nginx预访问的资源进行条件判断，如果匹配到，则进行下一项，如果没有匹配到，则返回其他动作。
```

```
配置属性
   try_files file ... uri; 		#按顺序查询资源是否存在，返回第一个匹配到的，
   								#如果没有匹配到，会内部重定向到最后一个资源
   try_files file ... =code; 	#最后还可以加一个状态码
```

##### 配置定制

```shell
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  # 先找 uri，往后依次是 uri.html uri/index.html /index.html

  try_files $uri $uri.html $uri/index.html /index.html;
  
  #如果有 web1 或 web1.html 会匹配到，返回状态码是200，如果没有则返回 500错误页面
  location /web1 {
  try_files $uri $uri.html;
  }
  location /web1/ {
  alias /data/server/nginx/web1/;
  try_files $uri $uri.html;
  }
  #如果有 b 或 b.html 会匹配到，返回状态码是200，如果没有则返回 418错误页面
  location /b{
  try_files $uri $uri.html =418;
  }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 

#测试效果
#先找 xyz，往后依次是 xyz.html xyz/index.html /index.html
root@ubuntu-43:conf # curl 10.0.0.43/xxh
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
#如果有 web1 或 web1.html 会匹配到，返回状态码是200，如果没有则返回 500错误页面
root@ubuntu-43:conf # curl 10.0.0.43/web1
<html>
<head><title>500 Internal Server Error</title></head>
<body>
<center><h1>500 Internal Server Error</h1></center>
<hr><center>nginx</center>
</body>
</html>
root@ubuntu-43:conf # curl 10.0.0.43/web1/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
#如果有 index.html 会匹配到，返回状态码是200，如果没有则返回 500错误页面
root@ubuntu-43:conf # curl 10.0.0.43/web1/index.html
nginx web1
#如果有 b 或 b.html 会匹配到，返回状态码是200，如果没有则返回 418错误页面
root@ubuntu-43:conf # curl 10.0.0.43/b 
root@ubuntu-43:conf # curl 10.0.0.43/b -I
HTTP/1.1 418 
Server: nginx
Date: Thu, 05 Dec 2024 11:08:39 GMT
Content-Length: 0
Connection: keep-alive
```

#### 状态页

```
stub_status; 	# 添加此指令后可开启 Nginx 状态页，作用域 server, location
```

```shell
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
    listen 80 default_server;

    location /status {
        stub_status;
        allow 10.0.0.43;
        deny all;
    }
}
root@ubuntu-43:conf # nginx -t
oot@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # curl 10.0.0.43/status
Active connections: 1 
server accepts handled requests
 1 1 1 
Reading: 0 Writing: 1 Waiting: 0 
```

```
Nginx状态页输出的关键指标包括：
    Active connections：当前处于活动状态的客户端连接数，包括正在读取、写入和等待的连接数。
    accepts：Nginx自启动后已经接受的客户端请求的总数。
    handled：Nginx自启动后已经处理完成的客户端请求总数，通常等于accepts，除非有因worker_connections限制等被拒绝的连接。
    requests：Nginx自启动后客户端发来的总的请求数。
    Reading：当前正在读取客户端请求报文首部的连接的连接数。数值越大，说明排队现象严重，性能不足。
    Writing：当前正在向客户端发送响应报文过程中的连接数。数值越大，说明访问量很大。
    Waiting：当前正在等待客户端发出请求的空闲连接数。在开启keep-alive的情况下，这个值等于active – (reading+writing)。
```

#### 内置变量属性

```shell
在 Nginx 中，除了内置变量外，我们还可以使用 set 指令来自定义变量
配置解读
 set $variable value; 	# 自定义变量，变量名以$开头，变量值可以从其它变量中获取，也可以直接指定
 						# 作用域 server, location, if
常用内置变量
$remote_addr 		# 客户端公网IP，如果经多层Nginx代理，那最后的Nginx 通过此变量无法获取客户端IP
$proxy_add_x_forwarded_for 	# 代理IP和真实客户端IP
$args 				# URL中的所有参数
$is_args 			# 是否有参数，有参数该变量值为 ?，没有参数值为空
$document_root 		# 当前资源的文件系统路径
$document_uri 		# 当前资源不包含参数的URI
$host 				# 请求头中的host值
$remote_port 		# 客户端随机问口
$remote_user 		# 经过 auth basic 验证过的用户名
$request_body_file 	# 作为反向代理发给后端服务器的本地资源名称
$request_method 	# 当前资源的请求方式 GET/PUT/DELETE 等
$request_filename 	# 当前资源在文件系统上的绝对路径
$request_uri 		# 不包含的主机名的URI，包含请求的资源和参数
$scheme 			# 请求协议， HTTP/HTTPS/FTP 等
$server_protocol 	# 客户端请求的协议版本 HTTP/1.0， HTTP/1.1， HTTP/2.0 等
$server_addr 		# 服务器IP
$server_name 		# 服务器主机名
$server_port 		# 服务器端口号
$http_user_agent 	# 客户端UA
$http_cookie 		# 客户端所有COOKIE
$cookie_<name> 		# 获取指定COOKIE
$http_<name> 		# 获取指定的请求头中的字段，如果字段有中划线要替换成下划线，大写转成小写
$sent_http_<name> 	# 获取指定的响应头中的字段，如果字段有中划线要替换成下划线，大写转成小写
$arg_<name> 		# 获取指定参数
```

#####  内置变量属性实践

```shell
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
    listen 80;
    location /vars{
        default_type text/html;
        return 200 "request: \$request;
       proxy_add_x_forwarded_for: \$proxy_add_x_forwarded_for;
       args: \$args;
       document_uri: \$document_uri;
       request_uri: \$request_uri;
       document_root: \$document_root;
       host: \$host;
       request_method: \$request_method;
       request_filename: \$request_filename;
       scheme: \$scheme;
       UA: \$http_User_Agent;
       all_cookies: \$http_cookie;
       cookie_uname: \$cookie_uname;\n";
 }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service
root@ubuntu-43:conf # curl --cookie "age=10,uname=jerry" "10.0.0.43/vars?id=123&name=tom"
request: \GET /vars?id=123&name=tom HTTP/1.1;
       proxy_add_x_forwarded_for: \10.0.0.43;
       args: \id=123&name=tom;
       document_uri: \/vars;
       request_uri: \/vars?id=123&name=tom;
       document_root: \/apps/nginx/html;
       host: \10.0.0.43;
       request_method: \GET;
       request_filename: \/apps/nginx/html/vars;
       scheme: \http;
       UA: \curl/8.5.0;
       all_cookies: \age=10,uname=jerry;
       cookie_uname: \jerry;
```

### 自定义变量

```shell
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
    listen 80;

    set $var1 1234;  # 直接赋值，数字
    set $var2 "hello world";  # 直接赋值，字符串，中间有空格
    set $var3 $host;  # 间接赋值，从内置变量中获得值
    
    location /set {
        set $var4 $var1;  # 间接赋值，从内置变量中获得值
        return 200 "
        var1:  $var1;
        var2:  $var2;
        var3:  $var3;
        var4:  $var4;\n";
    }
}
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
    listen 80;

    set $var1 1234;  # 直接赋值，数字
    set $var2 "hello world";  # 直接赋值，字符串，中间有空格
    set $var3 $host;  # 间接赋值，从内置变量中获得值
    
    location /set {
        set $var4 $var1;  # 间接赋值，从内置变量中获得值
        return 200 "
        var1:  $var1;
        var2:  $var2;
        var3:  $var3;
        var4:  $var4;\n";
    }
}

root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # curl 10.0.0.43/set

        var1:  1234;
        var2:  hello world;
        var3:  10.0.0.43;
        var4:  1234;
```

### 长连接

```shell
keepalive_timeout timeout [header_timeout]; 	#TCP握手建立连接后，会话可以保持多长时间，
 												#在此时间内，可以继续传送数据，而不用再次握手
												#默认值 keepalive_timeout 75s
												#header_timeout 用作响应头中显示
                                                
keepalive_requests number; 						#一次请求不断开连接的情况下最多可以传送多少个资源
 												#默认值 keepalive_requests 1000;
												#在请求过程中以上两项达到一项阀值，连接就会断开
注意：
 作用域 http, server, location
 
示例配置：
 keepalive_timeout 15 30; 		# 在当前server 中修改配置，服务端真实时长是15S，响应头中显示30S
```

#### 默认实践

```shell
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /web2/ {
     alias /data/server/nginx/web2/;
  }
  location /web3/ {
     alias /data/server/nginx/web3/;
  }
  location / {
    return 200 "uri: \$uri\n";
  }
}
```

#### 长连接默认实践

```shell
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  keepalive_requests 2;			#连续获取两次资源，退出。
  root /data/server/nginx/web1;
  location /web2/ {
     alias /data/server/nginx/web2/;
  }
  location /web3/ {
     alias /data/server/nginx/web3/;
  }
  location / {
    return 200 "uri: $uri\n";
  }
}

```

### 文件下载

```
文件上传功能通常要求支持多文件和文件夹的选择与上传，支持大文件批量上传（如20G以上），以及断点续传等功能。
文件下载功能通常要求支持文件的批量下载、断点续传以及支持文件夹结构的下载等。
```

```shell
下载服务器功能配置
    autoindex on|off; 						#是否显示目录内容列表,默认 off
    autoindex_exact_size on|off; 			#是否以友好的方式显示文件大小，
   											#默认 on，表示显示文件具体字节数
    autoindex_format html|xml|json|jsonp; 	#数据显示格式，默认 html
    autoindex_localtime on|off 				#是否以本地时区显示文件时间属性，
   											#默认off，以UTC时区显示
注意：
 以上指令作用域均为 http, server, location
 autoindex 所在root的目录下面，不要存在 index.html文件，否则直接显示首页
 
 上传服务器功能配置
    client_max_body_size size; 		#设置允许客户端上传单个文件最大值，超过此值会响应403，默认1m
    client_body_buffer_size size 	#用于接收每个客户端请求报文的body部分的缓冲区大小，
   									# 默认 16k，超过此大小时，
                                    #数据将被缓存到由 client_body_temp_path 指令所指定的位置
    
    client_body_temp_path path [level1 [level2 [level3]]]; 
                                    #存储客户端请求报文的body部分的临时存储路径及子目录结构和数量
                                    #子目录最多可以设三级，数字表示当前级别创建几个子目录，16进制
                                    # level1 的目录名字，0-9 a-f
									# level2 的目录名字，00-99，aa-ff
```

### 文件下载服务器实践

```shell
创建文件
mv /data/server/nginx/web1/index.html{,-bak}
dd if=/dev/zero of=/data/server/nginx/web1/10M.img bs=10M count=1
dd if=/dev/zero of=/data/server/nginx/web1/10M.img bs=20M count=1
cp /etc/fstab /data/server/nginx/web1/
cp /var/log/auth.log /data/server/nginx/web1/dir/logs/
echo "sanguoyanyi" > /data/server/nginx/web1/三国演义.txt
root@ubuntu-43:conf # tree /data/server/nginx/web1/
/data/server/nginx/web1/
├── 10M.img
├── 20M.img
├── afile
├── dir
│   └── logs
│       └── auth.log
├── fstab
├── index.html-bak
└── 三国演义.txt

3 directories, 7 files

#定制下载服务器
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  
  location /download {
    alias /data/server/nginx/web1;
    charset utf8;
    autoindex on;
    autoindex_exact_size off; #以友好格式显示文件大小
    autoindex_localtime on;   #以服务器时区显示文件时间
    # 设置下载文件的默认 MIME 类型（可选）
    default_type application/octet-stream;
  }
}

root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 
```

### 文件上传服务器实践

```
#安装php
root@ubuntu-43:conf # apt install php php-fpm php-mysql php-curl php-zip php-xml php-mbstring -y
#查看服务
root@ubuntu-43:conf # systemctl is-active php8.3-fpm.service
active

```



### 限速实践

```
目前 Nginx 中主要的三种限速操作分别是：限制请求数（request），限制连接数（connection），限制响应速度（rate），对应在 Nginx 中的模块指令分别是 limit_req，limit_conn 和  limit_rate 三部分。
```

```
限制单一连接下载速度
    limit_rate rate; 			#对单个客户端连接限速，默认单位为字节，其它单位需要显式指定，
   								#表示每秒的下载速度，限速只对单一连接而言，同一客户端两个连接，
								#总速率为限速2倍，默认值0，表示不限制
    limit_rate_after size; 		#在传输了多少数据之后开始限速，默认值0，表示一开始就限速
注意：
	作用域范围 http, server, location, if in location
```

限制客户端并发连接数

```shell
limit_conn_zone key zone=name:size; # 定义一个限速规则，供 limit_req 指令调用
 									# key 定义用于限速的关键字，表示以什么为依据来限制并发连接数
 									# zone=name:size name 表示规则名称
 									# size 表示使用多大内存空间存储 key 对应的内容，指桶大小
								# 数据会在多个worker 进程共享，空间耗尽后会使用LRU算法淘汰旧的数据
 								# 作用域 http
limit_conn zone number; 		# zone 表示要调用的规则，number 表示要限制的连接数，不能写变量
 								# 作用域 http, server, location
```

```shell
定义一个名为perip的共享内存区域，用于存储基于客户端二进制IP地址的连接状态，内存大小为10MB。
 	limit_conn_zone $binary_remote_addr zone=perip:10m;

限制每个客户端最多允许20个并发连接。
 	limit_conn perip 20;
```

```shell
root@ubuntu-43:conf # dd if=/dev/zero of=/data/server/nginx/web1/100M.img bs=100M count=1
root@ubuntu-43:conf # dd if=/dev/zero of=/data/server/nginx/web1/200M.img bs=200M count=1
root@ubuntu-43:conf # ll /data/server/nginx/web1/*.img -h
-rw-r--r-- 1 root root 100M Dec  5 12:27 /data/server/nginx/web1/100M.img
-rw-r--r-- 1 root root 200M Dec  5 12:27 /data/server/nginx/web1/200M.img
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
limit_conn_zone \$binary_remote_addr zone=mylimit:10m;
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /100M.img {
    limit_rate 10k; #每秒下载速度为10K
    limit_rate_after 1m; #前1M不开启限速
  }
  location /200M.img {
    limit_rate 10k; #每秒下载速度为10K
    limit_conn mylimit 2; #限制每个客户端最多允许2个并发连接。
  }
}

root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # ps aux | grep wget
root       28849  0.0  0.2  14752  4736 pts/0    S    12:38   0:00 wget 10.0.0.43/200M.img
root       28850  0.0  0.2  14752  4736 pts/0    S    12:38   0:00 wget 10.0.0.43/200M.img
root       28929  0.0  0.1   6544  2304 pts/0    S+   12:40   0:00 grep --color=auto wget
结果显示：
 只能同时存在两个下载
```

### 请求限制

```shell
limit_req_zone key zone=name:size rate=rate [sync]; 	# 定义一个限速规则，供limit_req 调用
 										# key 定义用于限速的关键字，表示以什么为依据来限速
										# zone=name:size name 表示规则名称
 										# size 表示使用多大内存空间存储 key 对应的内容，指桶大小
										# 数据会在多个worker 进程共享
										# 空间耗尽后会使用LRU算法淘汰旧的数据
										# rate=rate 限制请求速率，
										# 同一个key 每秒可以请求多少次或每分钟可以请求多少次
										# [sync] 实现共享内存的区域同步，商业版支持
										# 作用域 http，定义好后需要在其它地方进行调用才生效
limit_req zone=name [burst=number] [nodelay|delay=number]
 							# zone=name 表示调用哪个 limit_req_zone
							# [burst=number] 超过rate后能接收多少个请求，这些请求会慢慢处理
 							# [nodelay|delay=number] 
							# nodelay 表示rate匹配到的请求直接处理而且返回 503
 							# delay=number 表示有多少个请求直接处理，剩下的加队列慢慢处理
 							# 作用域 http, server, location
```

```
示例：
 	定义mylimit共享内存区域，存储客户端二进制IP地址的连接状态，内存大小为10MB,限制每秒请求数为10个。
 	10r/s 完整写法是 10requests/secends
 	因为nginx 是毫秒级别的控制粒度，10r/s 意味着对同一客户端在 100ms 只能处理一个请求
 	limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
 
 	允许额外的突发请求数量为20个，当请求速率超过限制时，Nginx会延迟或拒绝请求，并返回适当的错误响应。
 	limit_req zone=mylimit burst=20;
```

```shell
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
# 定制mylimit内存区域，用于存储客户端二进制IP地址的连接状态，内存大小为10MB。限制每秒请求数为2个。
# 2r/s，也就是说nginx在 500ms里面可以处理1个请求
limit_req_zone \$binary_remote_addr zone=mylimit:10m rate=2r/s;

server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location / {
    limit_req zone=mylimit;             # 调用请求规则
  }
}

root@ubuntu-43:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # for i in {1..5};do curl 10.0.0.43;sleep 0.4;done
hello hi 
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
hello hi 
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
hello hi 
root@ubuntu-43:conf # for i in {1..5};do curl 10.0.0.43;sleep 0.5;done
hello hi 
hello hi 
hello hi 
hello hi 
hello hi 
```

#### 请求超配实践

```shell
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
# 定制mylimit内存区域，用于存储客户端二进制IP地址的连接状态，内存大小为10MB。限制每秒请求数为2个。
# 2r/s，也就是说nginx在 500ms里面可以处理1个请求
limit_req_zone \$binary_remote_addr zone=mylimit:10m rate=2r/s;

server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location / {
    limit_req zone=mylimit burst=3;             # 调用请求规则
  }
}
#配置解析：
   #rate 速率的请求数为2r/s，500ms内仅处理1个请求，burst=3 允许额外的突发请求数量为20个，
   #假如接收5个请求，先处理1个，剩余的放到队列里面，
   #每500ms 处理一个，最多处理4个，第5个请求直接返回503 
root@ubuntu-43:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # for i in {1..5};do curl 10.0.0.43;done
hello hi 
hello hi 
hello hi 
hello hi 
hello hi 
root@ubuntu-43:conf # vim /root/test_nginx_limit.sh
root@ubuntu-43:conf # /bin/bash /root/test_nginx_limit.sh
hello hi 
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
hello hi 
hello hi 
hello hi 
root@ubuntu-43:conf # tail -5 ../logs/access.log 
10.0.0.43 - - [05/Dec/2024:12:54:59 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:12:54:59 +0000] "GET / HTTP/1.1" 503 190 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:12:55:00 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:12:55:00 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:12:55:01 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
结果显示：
 	因为并发请求，至于谁先到，谁后到，这个无法控制
```

#### 拒绝超配实践

```shell
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
# 定制mylimit内存区域，用于存储客户端二进制IP地址的连接状态，内存大小为10MB。限制每秒请求数为2个。
# 2r/s，也就是说nginx在 500ms里面可以处理1个请求
limit_req_zone \$binary_remote_addr zone=mylimit:10m rate=2r/s;

server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location / {
    limit_req zone=mylimit burst=3 nodelay;             # 调用请求规则
  }
}
#配置解析：
   #nodelay 表示无延时队列，排在队列越后面的请求等待时间越久，
   #如果请求数过多，可能会等到超时也不会被处理
root@ubuntu-43:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # for i in {1..5};do curl 10.0.0.43;sleep 0.1;done
hello hi 
hello hi 
hello hi 
hello hi 
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
root@ubuntu-43:conf # tail -5 ../logs/access.log 
10.0.0.43 - - [05/Dec/2024:13:00:31 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:13:00:32 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:13:00:32 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:13:00:32 +0000] "GET / HTTP/1.1" 200 10 "-" "curl/8.5.0"
10.0.0.43 - - [05/Dec/2024:13:00:32 +0000] "GET / HTTP/1.1" 503 190 "-" "curl/8.5.0"
#结果显示：
 	#按照顺序处理，最后一个不管了。
```

### 压缩功能

```shell
gzip on|off; 				#启用或禁用压缩功能，默认off，
 							# 作用域 http, server, location, if in location
gzip_buffers number size; 	#Nginx在压缩时要向服务器申请的缓存空间个数和每个缓存大小，
 							#默认 32 4k 或 16 8k
							#作用域 http, server, location
gzip_comp_level level; 		#压缩比，默认 1，
 							#作用域 http, server, location
gzip_disable regex ...; 	#根据客户端请求头中的UA字段内容来确定在某些情况下禁用压缩，
 							#可以支持正则表达式，gzip_disable "MSIE [1-6]\."; 
							#这种写法就表示 IE6 浏览器禁用压缩，默认为空
							#作用域 http, server, location
gzip_http_version 1.0|1.1; 	#启用gzip 压缩的最小版本，
 							#默认 1.1，即http/1.1 及更高版本才使用压缩
 							#作用域 http, server, location
gzip_min_length length; 	#资源体积多大才启用压缩，默认20，
 							#作用域 http, server, location
gzip_proxied off|expired|no-cache|no-store|private|no_last_modified|no_etag|auth|any ...;
 							#nginx作为反向代理时是否压缩后端返回数据
							#根据请求头中的Via 字段来判断，默认值 off，不压缩
 							#expired 如果请求头中包含 Expires，则压缩
							#no-cache 如果请求头中包含 Cache-Control:no-cache 则压缩
 							#no-store 如果请求头中包含 Cache-Control:no-store 则压缩
 							#private 如果请求头中包含 Cache-Control:private 则压缩
 							#no_last_modified 如果请求头中不包含 Last-Modified 则压缩
 							#no_etag 如果请求头中不包含 ETag 则压缩
							#auth 如果请求头中包含 Authorization 则压缩
							#any 任何情况都压缩
							#作用域 http, server, location
gzip_types mime-type ...; 	#指定要压缩的资源类型，默认 text/html，* 表示所有类型
 							#text/html 类型只要开启gzip 都会被压缩
							# 作用域 http, server, location
gzip_vary on|off; 			#是否在响应头中添加 Vary: Accept-Encoding，默认 off
 							#作用域 http, server, location
```

```shell
root@ubuntu-43:~ # cp /var/log/auth.log /data/server/nginx/web1/a.html
root@ubuntu-43:~ # cp /var/log/auth.log /data/server/nginx/web1/b.html
root@ubuntu-43:~ # cp /var/log/auth.log /data/server/nginx/web1/c.html
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  gzip on;                  # 开启压缩
  gzip_types text/html text/plain application/javascript application/xjavascript text/css application/xml text/javascript application/x-httpd-php 
image/gif image/png;
  gzip_vary on;             # 在响应头中添加 Vary头信息
        
  location /a.html {
    gzip_comp_level 5;      # 定制压缩比为5
  }
  location /b.html {
    gzip off;               # 关闭压缩功能
  }
}
root@ubuntu-43:conf # nginx -t
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # curl 10.0.0.43/a.html -I --compressed
HTTP/1.1 403 Forbidden
Server: nginx
Date: Thu, 05 Dec 2024 14:03:34 GMT
Content-Type: text/html; charset=utf8
Connection: keep-alive
Vary: Accept-Encoding
Content-Encoding: gzip

root@ubuntu-43:conf # curl 10.0.0.43/b.html -I --compressed
HTTP/1.1 403 Forbidden
Server: nginx
Date: Thu, 05 Dec 2024 14:03:51 GMT
Content-Type: text/html; charset=utf8
Content-Length: 146
Connection: keep-alive

root@ubuntu-43:conf # curl 10.0.0.43/c.html -I --compressed
HTTP/1.1 403 Forbidden
Server: nginx
Date: Thu, 05 Dec 2024 14:11:50 GMT
Content-Type: text/html; charset=utf8
Connection: keep-alive
Vary: Accept-Encoding
Content-Encoding: gzip
```

### 图标配置

```shell
oot@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
          
  location = /favicon.ico {
    root /data/server/nginx/web1/static;
    expires 7d;            # 缓存过期时间为7天
    log_not_found off;      # 未找到资源的请求,不记录日志
    access_log off;         # 不记录访问日志
  }
}
root@ubuntu-43:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf # systemctl restart nginx.service 
root@ubuntu-43:conf # mkdir /data/server/nginx/web1/static -p
root@ubuntu-43:conf # wget https://www.logosc.cn/uploads/output/2021/10/19/aaabb9adc7fad4ed3268b7e5ce05ba37.jpg -O /data/server/nginx/web1/static/favicon.ico
```

### HTTPS

```shell
root@ubuntu-43:conf # apt install easy-rsa -y
root@ubuntu-43:conf # cd /usr/share/easy-rsa/
root@ubuntu-43:easy-rsa # ./easyrsa init-pki
root@ubuntu-43:easy-rsa # tree pki/
pki/
├── inline
├── openssl-easyrsa.cnf
├── private
└── reqs

4 directories, 1 file
root@ubuntu-43:easy-rsa # ./easyrsa build-ca nopass
root@ubuntu-43:easy-rsa # ./easyrsa gen-req sswang.magedu.com nopass
root@ubuntu-43:easy-rsa # ./easyrsa sign-req server sswang.magedu.com
Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
root@ubuntu-43:easy-rsa #  cat pki/issued/sswang.magedu.com.crt pki/ca.crt > pki/sswang.magedu.com.pem
root@ubuntu-43:easy-rsa #  chmod +r pki/private/sswang.magedu.com.key 
root@ubuntu-43:easy-rsa # tree pki
pki
├── ca.crt
├── certs_by_serial
│   └── 30981053E951083B1AA0CD6C82E97593.pem
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── inline
├── issued
│   └── sswang.magedu.com.crt
├── openssl-easyrsa.cnf
├── private
│   ├── ca.key
│   └── sswang.magedu.com.key
├── reqs
│   └── sswang.magedu.com.req
├── revoked
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── serial
├── serial.old
└── sswang.magedu.com.pem

10 directories, 14 files

root@ubuntu-43:easy-rsa # vim /apps/nginx/conf/conf.d/vhost.conf 
root@ubuntu-43:easy-rsa # cat /apps/nginx/conf/conf.d/vhost.conf 
server {
  listen 80 default_server;
  server_name sswang.magedu.com;
  root /data/server/nginx/web1;
  # return 301 https://$host$request_uri; # 301重定向
  rewrite ^(.*) https://\$server_name\$1 permanent; #rewrite 重定向，二选一
}
server{
  listen 443 ssl;
  server_name sswang.magedu.com;
  root /data/server/nginx/web1;
  # 定制ssl的能力
  ssl_certificate /usr/share/easy-rsa/pki/sswang.magedu.com.pem;
  ssl_certificate_key /usr/share/easy-rsa/pki/private/sswang.magedu.com.key;
  ssl_session_cache shared:sslcache:20m;
  ssl_session_timeout 10m;
}
root@ubuntu-43:easy-rsa # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:easy-rsa # systemctl restart nginx.service 
root@ubuntu-43:easy-rsa # curl 10.0.0.43 -I
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Thu, 05 Dec 2024 14:38:43 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
Location: https://\sswang.magedu.com\/

root@ubuntu-43:easy-rsa # curl https://10.0.0.43 -I
curl: (60) SSL certificate problem: self-signed certificate in certificate chain
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
root@ubuntu-43:easy-rsa # curl https://10.0.0.43 -k
hello hi 
```

### 防盗链

```



```



### Rewrite

```powershell
break; 			#中断当前相同作用域(location)中的其它 ngx_http_rewrite_module 模块的配置和指令，
 				#返回到上一作用域继续执行
 				#该指令后的其它指令和配置还会执行，只中断 ngx_http_rewrite_module 指令，
 				# 作用域 server, location, if
return code [text];
return code URL;
return URL; 	# 不写code ，默认值为302
 				# 直接向客户端返回状态码，字符串，或者URL，
 				# 如果返回的字符串中包含空格，要加引号，如果返回URL，要写完整
 				# 此指令后的其它指令或配置将不再执行，作用域 server, location, if
rewrite_log on|off; 	#是否记录 ngx_http_rewrite_module 模块产生的日志到 error_log中，
 						#默认值 off, 如果开启，需要将 error_log 的级别设为 notice，
 						# 作用域 http, server, location, if
set $variable value; 	# 自定义变量，变量名以$开头，变量值可以从其它变量中获取，也可以直接指定
 						# 作用域 server, location, if
```

PCRE 风格正则表达式

```powershell
PCRE（Perl Compatible Regular Expressions）风格的正则表达式在设计上兼容Perl语言的正则表达式语法，具有灵活且功能强大的特点。下面是一些 PCRE 风格正则表达式中常见的元字符和功能。

元字符
	  . 	#匹配除换行符外的任意字符
    \w 		#匹配字母数字下划线中文字
    \s 		#匹配任意空白字符
    \d 		#匹配任意数字，相当于[0-9]
    \b 		#匹配单词开始或结束
    [] 		#匹配括号内的任意一个字符
            # [abc] 表示abc中的任意一个字符
    [^] 	#匹配除了括号内字符之外的任意一个字符
            # [^abc] 表示任意一个除 abc 之外的字符
    ^ 		#匹配内容的开始位置
    $ 		#匹配内容的结束位置
    * 		#匹配前面的字符零次或多次
    + 		#匹配前面的字符一次或多次
    ? 		#匹配前面的字符零次或一次
    {n} 	#匹配前面的字符n次
    {n,} 	#匹配前面的字符至少n次
    {n,m} 	#匹配前面的字符n次到m次
    {,m} 	#匹配前面的字符最多m次
    | 		#或，用于在模式中指定多个备选项
    () 		#分组，用于将多个模式组合在一起，并捕获匹配的文本 $1 $2 $n 后向引用
    \ 		#转义字符，用于取消元字符的特殊意义，或引入某些特殊字符
    
注意：我们平常所说的正则，主要指的是 普通正则和扩展正则。不同的正则，需要有不同的正则表达式解析引擎。
```

```powershell
1、编译echo
cd /usr/local/src/
wget https://github.com/openresty/echo-nginx-module/archive/refs/tags/v0.63.tar.gz
tar xf v0.63.tar.gz
root@ubuntu:nginx-1.22.1 # nginx -V
nginx version: nginx/1.22.1
built by gcc 13.2.0 (Ubuntu 13.2.0-23ubuntu4) 
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module
root@ubuntu:nginx-1.22.1 # ./configure --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=/usr/local/src/echo-nginx-module-0.63
root@ubuntu:nginx-1.22.1 # make
root@ubuntu:nginx-1.22.1 # nginx -V
nginx version: nginx/1.22.1
built by gcc 13.2.0 (Ubuntu 13.2.0-23ubuntu4) 
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module
root@ubuntu:nginx-1.22.1 # ./objs/nginx -V
nginx version: nginx/1.22.1
built by gcc 13.2.0 (Ubuntu 13.2.0-23ubuntu4) 
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=/usr/local/src/echo-nginx-module-0.63
root@ubuntu:nginx-1.22.1 # ls ./objs/nginx /usr/sbin/nginx -l
ls: cannot access '/usr/sbin/nginx': No such file or directory
-rwxr-xr-x 1 root root 6180088 Dec  7 02:04 ./objs/ngin
root@ubuntu:nginx-1.22.1 # find / -name nginx
/apps/nginx
/apps/nginx/sbin/nginx
/usr/local/src/nginx-1.22.1/objs/nginx
/usr/local/sbin/nginx
root@ubuntu:nginx-1.22.1 # ls ./objs/nginx /usr/local/sbin/nginx -l
-rwxr-xr-x 1 root root 6180088 Dec  7 02:04 ./objs/nginx
lrwxrwxrwx 1 root root      22 Dec  7 01:56 /usr/local/sbin/nginx -> /apps/nginx/sbin/nginx
root@ubuntu:nginx-1.22.1 # rm -rf /apps/nginx/sbin/nginx.1.22.1.bak
root@ubuntu:nginx-1.22.1 # cp ./objs/nginx /apps/nginx/sbin/
root@ubuntu:nginx-1.22.1 # ls /apps/nginx/sbin/
nginx
root@ubuntu:nginx-1.22.1 # nginx -V
nginx version: nginx/1.22.1
built by gcc 13.2.0 (Ubuntu 13.2.0-23ubuntu4) 
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=/usr/local/src/echo-nginx-module-0.63

2、
root@ubuntu:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /break{
    set $var1 magedu;
    echo $var1;
    break;
    # break命令之后，ngx_http_rewrite_module 模块其他指令则不再生效，比如set 和 return
    set $var2 1234;
    echo "$var1 -- $var2";
    return 200 "hello break";
  }
  location /nobreak{
    set $var1 magedu;
    set $var2 1234;
    echo "$var1 -- $var2";
    return 211 "hello nobreak\n";
  }
}
root@ubuntu:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu:conf # systemctl restart nginx.service 
root@ubuntu:conf # curl 10.0.0.43/back
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.22.1</center>
</body>
</html>
root@ubuntu:conf # curl 10.0.0.43/break
magedu
magedu -- 
root@ubuntu:conf # curl 10.0.0.43/break -I
HTTP/1.1 200 OK
Server: nginx/1.22.1
Date: Sat, 07 Dec 2024 02:23:26 GMT
Content-Type: application/octet-stream
Connection: keep-alive
root@ubuntu:conf # curl 10.0.0.43/nobreak
hello nobreak
root@ubuntu:conf # curl 10.0.0.43/nobreak -I
HTTP/1.1 211 
Server: nginx/1.22.1
Date: Sat, 07 Dec 2024 02:35:36 GMT
Content-Type: application/octet-stream
Content-Length: 14
Connection: keep-alive
```



#### if条件属性

```
if (condition) { ... }
 			# 允许在配置中使用条件判断，使用正则表达式(pcre风格)对变量进行匹配，匹配成功返true，执行后续指令
 			# if指令仅能做单次判断，不支持 if else 多分支
            # if ($var){ } 这种写法，如果变量对应的值是空字符串或0，就返回false
			# nginx 1.0.1之前$变量的值如果以0开头的任意字符串会返回false
 			# 作用域 server, location
 			# 支持的运算符
 			# = 比较变量和字符串是否相等
 			# != 比较变量和字符串是否不相等
 			# ~ 区分大小写，是否匹配正则，包含
 			# !~ 区分大小写，是否不匹配正则，包含
 			# ~* 不区分大小写，是否匹配正则，包含
 			# !~* 不区分大小写，是否不匹配正则，不包含
		 	# -f|!-f 判断文件是否存在|不存在 
 			# -d|!-d 判断目录是否存在|不存在
 			# -x|!-x 判断文件是否可执行|不可执行
 			# -e|!-e 判断文件(包括文件，目录，软链接)是否存在|不存在
```

```powershell
root@ubuntu:conf # mkdir -p  /data/server/nginx/web{1..3}
root@ubuntu:conf # echo file-log > /data/server/nginx/web1/file.log
root@ubuntu:conf # echo file-txt > /data/server/nginx/web1/file.txt
oot@ubuntu:conf # cat > conf.d/vhost.conf <<-eof
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /file {
    if (!-e \$request_filename){
      return 200 "\$request_filename 文件不存在";
    }
  }
}
eof
root@ubuntu:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu:conf # systemctl restart nginx.service 
root@ubuntu:conf # curl 10.0.0.43/file.txt
file-txt
root@ubuntu:conf # curl 10.0.0.43/file.html
/data/server/nginx/web1/file.html 文件不存在
root@ubuntu:conf # curl 10.0.0.43/file.log
file-log
root@ubuntu:conf # curl 10.0.0.43/file
/data/server/nginx/web1/file 文件不存在
```

#### set 配合 if实践

```powershell
root@ubuntu:conf # dd if=/dev/zero of=/data/server/nginx/web1/noslow.img bs=10M count=1
1+0 records in
1+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.0121535 s, 863 MB/s
root@ubuntu:conf # dd if=/dev/zero of=/data/server/nginx/web1/slow.img bs=10M count=1
1+0 records in
1+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.00619364 s, 1.7 GB/s
root@ubuntu:conf # ls /data/server/nginx/web1/*.img
/data/server/nginx/web1/noslow.img  /data/server/nginx/web1/slow.img
root@ubuntu:conf # ls /data/server/nginx/web1/*.img -lh
-rw-r--r-- 1 root root 10M Dec  7 02:47 /data/server/nginx/web1/noslow.img
-rw-r--r-- 1 root root 10M Dec  7 02:47 /data/server/nginx/web1/slow.img
root@ubuntu:conf # cat > conf.d/vhost.conf <<-eof
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /noslow.img {
    break;
    set \$slow 1;
    if (\$slow) {
      limit_rate 10k;
    }
  }
  location /slow.img{
    set \$slow 1;
    if (\$slow) {
      limit_rate 10k;
    }
  }
}
eof
root@ubuntu:conf # systemctl restart nginx
root@ubuntu:conf # wget 10.0.0.43/noslow.img
--2024-12-07 02:50:18--  http://10.0.0.43/noslow.img
Connecting to 10.0.0.43:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10485760 (10M) [application/octet-stream]
Saving to: ‘noslow.img’

noslow.img                                          100%[===================================================================================================================>]  10.00M  --.-KB/s    in 0.005s  

2024-12-07 02:50:18 (1.98 GB/s) - ‘noslow.img’ saved [10485760/10485760]

root@ubuntu:conf # wget 10.0.0.43/slow.img
--2024-12-07 02:50:45--  http://10.0.0.43/slow.img
Connecting to 10.0.0.43:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10485760 (10M) [application/octet-stream]
Saving to: ‘slow.img’

slow.img                                              0%[                                                                                                                    ]  72.00K  11.9KB/s    eta 14m 12s
```

### 重定向

```
rewrite regex replacement [flag]; 
 		# 通过正则表达式匹配来改变URI，在一个配置段中可以有一条或多条，按照顺序从上下往下匹配
 		# 作用域 server, location, if
 		# 如果有多条规则，被某一条规则命中并替换后，会用新的URI再从头开始逐一匹配，
 		# 直到没有被命中为止，但是重复匹配次数不能超过 10次，否则会报500
 		# regex PCRE 风格的正则表达式，表示要查找的内容
 		# replacement 用于替换的字符串
[flag] 标志位，用于控制 rewrite 指令的行为 last|break|redirect|permanent
 	- last 如果被当前 rewrite 规则匹配上，替换后结束本轮替换，开始下一轮替换
 	- break 如果被当前 rewrite 规则匹配上，替换后结束当前代码段的重写替换，后续所有rewrite 都不执行
 	- redirect   如果被当前 rewrite 规则匹配上，替换后执行 302 临时重定向
 	- permanent  如果被当前 rewrite 规则匹配上，替换后执行 301 永久重定向
 	- last 和 break 在服务器内部实现跳转，客户端浏览器地址栏中的信息不会发生变化
 	- redirect 和 permanent 在客户端实现跳转，客户端浏览器地址栏中的信息会发生变化
```

**last和break**

```
相同点：
 	- 无论是break还是last，它们都会中止当前 location 块的处理，并跳出该块，客户端浏览器地址栏中的信息不会发生变化
不同点：
    - break：终止当前代码段中的所有 rewrite 匹配
    - last：中止当前location中的 rewrite 匹配，用替换后的URI继续从第一条规则开始执行下一轮rewrite
```

```powershell
root@ubuntu:conf # cat conf.d/vhost.conf 
server {
    listen 80 default_server;
    root /data/server/nginx/web1;

    location /1.html {
        rewrite /1.html /2.html; # 访问1跳转到2
        rewrite /2.html /3.html; # 访问2跳转到3
    }

    location /2.html {
        rewrite /2.html /a.html; # 访问2跳转到a
    }

    location /3.html {
        rewrite /3.html /b.html; # 访问3跳转到b
    }

    location /c.html {
        rewrite /c.html /3.html; # 访问c跳转到3
        rewrite /3.html /1.html; # 访问3跳转到1
        rewrite /1.html /c.html; # 访问1跳转到c
    }
}

root@ubuntu:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu:conf # systemctl restart nginx.service 
root@ubuntu:conf # curl 10.0.0.43/1.html
bbbbb
root@ubuntu:conf # curl 10.0.0.43/2.html
aaaaa
root@ubuntu:conf # curl 10.0.0.43/3.html
bbbbb
root@ubuntu:conf # curl 10.0.0.43/c.html		# 无限循环，最终返回 500 服务器内部错误
<html>
<head><title>500 Internal Server Error</title></head>
<body>
<center><h1>500 Internal Server Error</h1></center>
<hr><center>nginx/1.22.1</center>
</body>
</html>
```

#### break & last 重定向实践

```powershell
root@ubuntu:conf # cat > conf.d/vhost.conf <<-eof
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /1.html {
    # break 结束当前 server 配置段中的所有 rewrite
    rewrite /1.html /a.html break; # 访问1跳转到a
    rewrite /a.html /3.html; # 访问a跳转到3
  }
  location /2.html {
    # last 结束当前 location 中的本轮 rewrite，继续执行下一轮 rewrite
    rewrite /2.html /a.html last; # 访问2跳转到a
    rewrite /a.html /3.html; # 访问a跳转到3
  }
  location /a.html {
    rewrite /a.html /b.html; # 访问a跳转到b
  }
}
eof
root@ubuntu:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu:conf # systemctl restart nginx.service 
root@ubuntu:conf # curl 10.0.0.43/1.html	#访问1跳转到a, 遇到break，不再跳了	
aaaaa
root@ubuntu:conf # curl 10.0.0.43/2.html	#访问2跳转到a, 访问a跳转到b
bbbbb
root@ubuntu:conf # curl 10.0.0.43/a.html	#访问a跳转到b.
bbbbb
```

#### redirect & permanent 重定向

```powershell
root@ubuntu:conf # cat > conf.d/vhost.conf <<-eof
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /1.html {
    rewrite /1.html /c.html redirect; # 访问1跳转到c, 客户端重定向
  }
  location /a.html {
    rewrite /a.html /b.html permanent; # 访问a跳转到b，永久重定向
  }
}
eof
root@ubuntu:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu:conf # systemctl restart nginx.service 
root@ubuntu:conf # curl 10.0.0.43/1.html -L
ccccc
root@ubuntu:conf # curl 10.0.0.43/1.html -I
HTTP/1.1 302 Moved Temporarily
Server: nginx/1.22.1
Date: Sat, 07 Dec 2024 03:29:38 GMT
Content-Type: text/html
Content-Length: 145
Location: http://10.0.0.43/c.html
Connection: keep-alive
root@ubuntu:conf # curl 10.0.0.43/a.html -I
HTTP/1.1 301 Moved Permanently
Server: nginx/1.22.1
Date: Sat, 07 Dec 2024 03:30:34 GMT
Content-Type: text/html
Content-Length: 169
Location: http://10.0.0.43/b.html
Connection: keep-alive

root@ubuntu:conf # curl 10.0.0.43/a.html -L
bbbbb
```

## nginx反向代理

```
代理分为两种，分别是正向代理和反向代理。正向代理（Forward Proxy） 和 反向代理（Reverse Proxy） 是两种常见的代理服务器，它们用于处理网络通信中的不同方向和用途。
```

![image-20241207113134384](5day-png\18nginx代理图解.png)

正向代理（Forward Proxy）

```
特点
    - 代理服务器位于客户端和目标服务器之间
    - 客户端向代理服务器发送请求，代理服务器将请求发送到目标服务器，并将目标服务器的响应返回给客户端
    - 目标服务器不知道客户端的存在，它只知道有一个代理服务器向其发送请求
    - 客户端通过正向代理访问互联网资源时，通常需要配置客户端来使用代理
用途
    - 突破访问限制：用于绕过网络访问限制，访问受限制的资源
    - 隐藏客户端身份：客户端可以通过正向代理隐藏其真实 IP 地址
```

反向代理（Reverse Proxy）

```
特点
    - 代理服务器位于目标服务器和客户端之间
    - 客户端向代理服务器发送请求，代理服务器将请求转发给一个或多个目标服务器，并将其中一个目标服务器的响应返回给客户端
    - 目标服务器不知道最终客户端的身份，只知道有一个代理服务器向其发送请求
    - 用于将客户端的请求分发给多个服务器，实现负载均衡
```

```
用途
    - 负载均衡：通过将流量分发到多个服务器，确保服务器的负载均匀分布
    - 缓存和加速：反向代理可以缓存静态内容，减轻目标服务器的负载，并提高访问速度
    - 安全性：隐藏真实服务器的信息，提高安全性，同时可以进行 SSL 终止（SSL Termination）
```

相同和不同

```
相同点
    - 中间层：正向代理和反向代理都是位于客户端和目标服务器之间的中间层。
    - 代理功能：它们都充当了代理的角色，处理请求和响应，使得通信更加灵活和安全
不同点
    - 方向：正向代理代理客户端，反向代理代理服务器
    - 目的：正向代理主要用于访问控制和隐藏客户端身份，反向代理主要用于负载均衡、缓存和提高安全性
    - 配置：客户端需要配置使用正向代理，而反向代理是对服务器透明的，客户端无需感知

```

### Nginx 和 LVS

```
Nginx 还是 LVS 取决于具体的应用需求和复杂度。Nginx 更适合作为 Web 服务器和应用层负载均衡器，而 LVS 更适用于传输层负载均衡。
相同点
    - 负载均衡：Nginx 和 LVS 都可以作为负载均衡器，将流量分发到多个后端服务器，提高系统的可用
性和性能。
    - 性能：Nginx 和 LVS 都具有高性能的特点，能够处理大量并发连接和请求
不同点
    - 层次：Nginx 在应用层进行负载均衡和反向代理，而 LVS 在传输层进行负载均衡
    - 功能：Nginx 除了负载均衡外，还可以作为反向代理和静态文件服务器；而 LVS 主要专注于负载均衡，实现简单而高效的四层分发
    - 配置和管理：Nginx 配置相对简单，易于管理，适用于各种规模的应用；LVS 需要深入了解 Linux 内核和相关配置，适用于大规模和对性能有更高要求的场景。
LVS 不监听端口，不处理请求数据，不参与握手流程，只会在内核层转发数据报文
Nginx 需要在应用层接收请求，根据客户端的请求参数和Nginx中配置的规则，再重新作为客户端向后端服务器发起请求
LVS 通常做四层代理，Nginx 做七层代理
```

![image-20241207113518518](5day-png\18lvs和nginx.png)

### 反向代理

```shell
proxy_pass URL; 					# 转发的后端服务器地址，可以写主机名,域名，IP地址，
 									# 也可以额外指定端口，
									# 作用域 location, if in location, limit_except
proxy_pass_header field; 			# 显式指定要回传给客户端的后端服务器响应头中的字段
proxy_pass_request_body on|off; 	# 是否向后端服务器发送客户端 http 请求的 body 部份，默认
proxy_pass_request_headers on|off; 	# 是否向后端服务器发送客户端 http 请求的头部信息，默认 on
proxy_hide_header field; 			# Nginx 默认不会将后端服务器的 Date,Server,Xxxx... 
 									# 这些响应头信息传给客户端，除了这些之外的响应头字段会回传，
									# 可以使用 proxy_hide_header 显式指定不回传的响应头字段
proxy_connect_timeout time; 		# Nginx与后端服务器建立连接超时时长,超时返回504
proxy_read_timeout time; 			# Nginx 等待后端服务器返回数据的超时时长,超时返回504
proxy_send_timeout time; 			# Nginx 向后端服务器发送请求的超时时长,超时返回408
proxy_set_body value; 				# 重新定义传给后端服务器的请求的正文，可以包含文本，变量等
proxy_set_header field value; 		# 更改或添加请求头字段并发送到后端服务器
proxy_http_version 1.0|1.1; 		# 设置向后端服务器发送请求时的 http 协议版本，默认值1.0
proxy_ignore_client_abort on|off; 	# 客户端中断连接，Nginx 是否继续执行与后端的连接，
 									# 默认值 off，客户端中断，Nginx 也会中断后端连接，
 									# on 表示 客户端中断，nginx 还会继续处理与后端在连接
proxy_headers_hash_bucket_size size; # 当配置了 proxy_hide_header和proxy_set_header的时候，
 									# 用于设置nginx保存HTTP报文头的hash表的大小，默认值 64
proxy_headers_hash_max_size size; 	# 上一个参数的上限，默认值 512
proxy_next_upstream error|timeout|invalid_header|...| off ...;
 									# 当前配置的后端服务器无法提供服务时，去请求下一个后端服务器
 									# 默认值 error timeout，表示因超时错误时，去请求
注意：
 以上配置属性的作用域范围均是 http, server, location
```

```powershell
root@ubuntu:conf # vim conf.d/vhost.conf 
root@ubuntu:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location / {
    proxy_pass http://10.0.0.43:81; # 交给后端的81端口服务
    proxy_connect_timeout 2s; # 设置连接超时时间为2s
  }
  location /1 {
    proxy_pass http://10.0.0.43:88; # 交给后端的88端口服务
  }
}
server {
  listen 81 default_server;
  root /data/server/nginx/web2;
  access_log /var/log/nginx/access_client.log;
}
root@ubuntu:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu:conf # systemctl restart nginx.service 
注意：
 如果后端服务不可用，从客户端访问会返回502
 如果连接后端服务器超时，会报504
root@ubuntu:conf # curl 10.0.0.43
nginx web2
root@ubuntu:conf # curl 10.0.0.43/1
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.22.1</center>
</body>
</html>
添加防火墙规则 - 禁止10.0.0.13 向 81端口发送请求
root@ubuntu:conf # iptables -A INPUT -p tcp --dport 81 -s 10.0.0.43 -j DROP
root@ubuntu:conf # curl 10.0.0.43
<html>
<head><title>504 Gateway Time-out</title></head>
<body>
<center><h1>504 Gateway Time-out</h1></center>
<hr><center>nginx/1.22.1</center>
</body>
</html>
删除规则
root@ubuntu:conf # iptables -F
root@ubuntu:conf # curl 10.0.0.43
nginx web2
```

### 动静分离

```
静态资源处理：
    Nginx会将静态资源（如HTML、CSS、JavaScript文件、图片等）的请求直接处理并返回给客户端。
    静态资源可以缓存在Nginx服务器上，减少对后端服务器的请求次数，提高响应速度。
动态资源处理：
    对于动态资源（如JSP页面、Servlet程序等）的请求，Nginx会将其转发给专门的后端服务器进行处理。
    后端服务器处理完动态内容后，将结果返回给Nginx，再由Nginx返回给客户端。
```

```
优势和好处
提高性能：
 	通过动静分离，Nginx可以专注于处理静态资源，而后端服务器则专注于处理动态资源。这种分工可以提高整个系统的并发处理能力和响应速度。
节省资源：
 	静态资源不需要经过后端应用服务器处理，直接由Nginx返回给客户端，从而节省了服务器的计算和内存资源。
简化开发与维护：
 	将动态资源和静态资源分开处理，可以使开发者更专注于业务逻辑的开发和维护，同时方便对静态资源的管理和部署。
```

```powershell
proxy_cache_path /tmp/nginx/cache levels=1:2 keys_zone=mycache:10m max_size=10g inactive=60m;
levels=1:2 				#指定了缓存文件存储的目录结构。
keys_zone=mycache:10m 	#为缓存区域分配了 10MB 的内存。
max_size=10g 			#设置了缓存目录的最大容量为 10GB。
inactive=60m 			#设置了 60 分钟后未访问的缓存项会过期。
设置缓存过期时间
在配置缓存时，确保你为不同的响应状态设置了合适的缓存过期时间。比如：
proxy_cache_valid 200 302 301 5m;  	# 对于 200, 302, 301 状态码，缓存 5 分钟
proxy_cache_valid any 2m;           # 对于其他状态码，缓存 2 分钟
```



```
client 			10.0.0.42
proxy server 	10.0.0.43
api server 		10.0.0.44
static server	10.0.0.60
```

```powershell
static server	10.0.0.60
root@openEuler-60:~ # yum -y install nginx
root@openEuler-60:~ # echo "Static Web Server" > /usr/share/nginx/html/index.html
root@openEuler-60:~ # cat > /etc/nginx/default.d/simple.conf<<-eof
location / {
  add_header X-Host \$host;
}
eof
root@openEuler-60:~ # 
root@openEuler-60:~ # systemctl restart nginx.service 
root@openEuler-60:~ # curl localhost
Static Web Server
```

```python
api server 		10.0.0.44
root@ubuntu-44:~ # apt -y install python3
root@ubuntu-44:~ # vim simple_http_server.py 
root@ubuntu-44:~ # cat simple_http_server.py 
from http.server import BaseHTTPRequestHandler, HTTPServer
class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # 获取请求的URL路径
        url_path = self.path
        # 获取请求头中的Host信息
        host = self.headers.get('Host')
        # 设置响应状态码
        self.send_response(200)
        # 设置响应头
        self.send_header('Content-type', 'text/plain')
        self.end_headers()
        # 准备响应内容，包含换行信息
        response_content = f"API Server: {url_path}, Host Header: {host}\n"
        # 发送响应内容：请求的URL路径
        # self.wfile.write(url_path.encode('utf-8'))
        self.wfile.write(response_content.encode('utf-8'))
def run(server_class=HTTPServer, handler_class=SimpleHTTPRequestHandler, 
port=8080):
    server_address = ('', port)
    httpd = server_class(server_address, handler_class)
    print(f'Starting httpd server on port {port}')
    httpd.serve_forever()
if __name__ == '__main__':
    run(port=8080)
    
root@ubuntu-44:~ # python3 simple_http_server.py >/dev/null 2>&1 &
```

```powershell
proxy server 	10.0.0.43
root@ubuntu-43:conf # systemctl restart nginx
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf # cat conf.d/vhost.conf 
server {
  listen 80 default_server;
  server_name hzk.com;
  root /data/server/nginx/web1;
  location /api {
    proxy_pass http://10.0.0.44:8080;
    proxy_set_header Host "api.magedu.com";
  }
  location /static {
    rewrite ^/static(.*)$ /index.html break;  # 重写url
    proxy_pass http://10.0.0.60;
    proxy_set_header Host "static.magedu.com";
  }
  location /static1/ {
    # /static1/index.html 转发给后端 /index.html
    proxy_pass http://10.0.0.60/;    # 以/为结尾
    proxy_set_header Host "static.magedu.com";
  }
}
root@ubuntu-43:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf # systemctl restart nginx

```

```powershell
client 			10.0.0.42
root@ubuntu-42:~ # tail -1 /etc/hosts
10.0.0.43 hzk.com
root@ubuntu-42:~ # curl hzk.com/api
API Server: /api, Host Header: api.magedu.com
root@ubuntu-42:~ # curl hzk.com/apidwsfaswd
API Server: /apidwsfaswd, Host Header: api.magedu.com
root@ubuntu-42:~ # curl hzk.com/static
Static Web Server
root@ubuntu-42:~ # curl hzk.com/static1
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.22.1</center>
</body>
</html>
root@ubuntu-42:~ # curl hzk.com/static1 -L
Static Web Server
root@ubuntu-42:~ # curl hzk.com/static1 -I
HTTP/1.1 301 Moved Permanently
Server: nginx/1.22.1
Date: Sat, 07 Dec 2024 05:33:32 GMT
Content-Type: text/html
Content-Length: 169
Location: http://hzk.com/static1/
Connection: keep-alive
```

### 代理缓存

```powershell
proxy_cache zone|off; 				# 是否启用代理缓存，默认 off,不启用，zone 指缓存名称
proxy_cache_path path [args]; 		# 开启代理缓存后指定缓存数据的存放路径
proxy_cache_key string; 			# 指定缓存数据的key，不同的key 对应不同的缓存文件
 									# 默认值 $scheme$proxy_host$request_uri
proxy_cache_valid [code ...] time;  # 为不同响应状态码的数据设置不同的缓存时长，可设置多条，
 									# 默认不设置,此处超时不会删除文件，会重新生成后覆盖
proxy_cache_use_stale error|timeout|invalid_header|...|off ...; 
									# 在后端服务器报哪些错误的情况下，直接使用过期缓存数据响应请求
    								# 默认off
proxy_cache_methods GET|...; 		# 缓存哪些请求类型的数据，默认值 GET HEAD
注意：
 以上配置属性的作用域范围均是 http, server, location
```

```powershell
配置实例
proxy_cache_path /tmp/proxycache levels=1:2 keys_zone=proxycache:20m inactive=60s max_size=1g;
 	这条proxy_cache_path指令配置了一个名为/tmp/proxycache的缓存目录，具有两级目录结构，一个20MB的内存区域用于存储缓存键和元数据，缓存项在不活跃60秒后可以被删除，并且整个缓存目录的最大大小为1GB。
```

```powershell
前置条件：各服务器时间和时区先统一，方便测试
```

```powershell
root@ubuntu-43:~ # cat /apps/nginx/conf/conf.d/vhost.conf 
proxy_cache_path /var/cache/nginx/ levels=1:2 keys_zone=proxycache:20m inactive=60s max_size=1g;
server {
  listen 80 default_server;
  server_name hzk.com;
  root /data/server/nginx/web1;
  location /api {
    proxy_pass http://10.0.0.44:8080;
    proxy_set_header Host "api.com";
  }
  location /static {
    rewrite ^/static(.*)$ /index.html break;  # 重写url
    proxy_pass http://10.0.0.60;
    proxy_set_header Host "static.com";
    proxy_cache proxycache;                    # 使用缓存
    proxy_cache_key $request_uri;
    proxy_cache_valid 200 302 301 90s;         # 缓存 90 秒
    proxy_cache_valid any 2m;                  # 其他状态码缓存 2 分钟
}

}
root@ubuntu-43:conf # mkdir /var/cache/nginx
root@ubuntu-43:conf # chown nginx:nginx /var/cache/nginx/ -R
root@ubuntu-43:conf # vim conf.d/vhost.conf 
root@ubuntu-43:conf # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf # systemctl restart nginx
root@ubuntu-43:~ # tree /var/cache/nginx/
/var/cache/nginx/
└── 9
    └── cf
        └── 7e2b55c4c38e99561caa378268f87cf9

3 directories, 1 file
root@ubuntu-43:~ # file /var/cache/nginx/9/cf/7e2b55c4c38e99561caa378268f87cf9 
/var/cache/nginx/9/cf/7e2b55c4c38e99561caa378268f87cf9: data
root@ubuntu-43:~ # cat /var/cache/nginx/9/cf/7e2b55c4c38e99561caa378268f87cf9
"6753dd3e-12"��^Y
KEY: /static
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Sat, 07 Dec 2024 06:38:59 GMT
Content-Type: text/html
Content-Length: 18
Last-Modified: Sat, 07 Dec 2024 05:29:34 GMT
Connection: close
ETag: "6753dd3e-12"
X-Host: static.com
Accept-Ranges: bytes

Static Web Server

90s后
root@ubuntu-43:~ # tree /var/cache/nginx/
/var/cache/nginx/
└── 9
    └── cf

3 directories, 0 files
```



### 缓存命中配置

```
Nginx的缓存命中配置涉及多个方面，包括缓存路径的设置、缓存有效期的配置、缓存键的定义等。
proxy_cache_path path ... 			# 设置缓存数据存放的根路径
proxy_cache_valid [code ...] time; 	# 设置对应状态码的缓存有效期。
proxy_cache_key string; 			# 定义缓存的key格式
proxy_cache zone | off; 			# 是否启用定义的缓存区域。
add_header name value; 				# 为缓存添加的HTTP响应头的名称。
```

```powershell
配置实例
http {
    # 设置缓存路径和相关参数
    #levels=1:2 指定了缓存文件存储的目录结构。
	#keys_zone=mycache:10m 为缓存区域分配了 10MB 的内存。
	#max_size=10g 设置了缓存目录的最大容量为 10GB。
	#inactive=60m 设置了 60 分钟后未访问的缓存项会过期。
    proxy_cache_path /tmp/nginx/cache levels=1:2 keys_zone=mycache:10m max_size=10g inactive=60m;
    server {
        ...
        location / {
            proxy_pass http://backend_server;  # 后端服务器地址
            proxy_cache mycache;  # 引用前面定义的缓存区域
            proxy_cache_valid 200 302 10m;  # 设置200和302状态码的缓存有效期为10分钟
            proxy_cache_valid 404 1m;  # 设置404状态码的缓存有效期为1分钟
            proxy_cache_key "$scheme$proxy_host$request_uri$is_args$args";  # 定义缓存的key格式
            add_header Nginx-Cache "$upstream_cache_status";  # 添加缓存状态参数
        }
    }
}
```

```powershell
root@ubuntu-43:conf.d # vim vhost.conf 
root@ubuntu-43:conf.d # cat vhost.conf 
proxy_cache_path /tmp/proxycache levels=1:2 keys_zone=proxycache:20m inactive=60s max_size=1g;
server {
  listen 80 default_server;
  server_name hzk.com;
  root /data/server/nginx/web1;
  location /api {
    proxy_pass http://10.0.0.44:8080;
    proxy_set_header Host "api.com";
  }
  location /static {
    rewrite ^/static(.*)$ /index.html break;  # 重写url
    proxy_pass http://10.0.0.60;
    proxy_set_header Host "static.com";
    proxy_cache proxycache;                    #使用缓存
    proxy_cache_key $request_uri;
    proxy_cache_valid 200 302 301 90s;         # 缓存90s
    proxy_cache_valid any 2m;                  #此处一定要写，否则缓存不生效
    add_header X-Cache $upstream_cache_status; #添加响应头，可以用来查看是否命中缓存
  }
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf.d # systemctl restart nginx

root@ubuntu-42:~ # curl hzk.com/static/wdasf -I
HTTP/1.1 200 OK
Server: nginx/1.22.1
Date: Sat, 07 Dec 2024 06:59:24 GMT
Content-Type: text/html
Content-Length: 20
Connection: keep-alive
Last-Modified: Sat, 07 Dec 2024 06:43:42 GMT
ETag: "6753ee9e-14"
X-Host: static.com
X-Cache: MISS
Accept-Ranges: bytes

root@ubuntu-42:~ # curl hzk.com/static/wdasf -I
HTTP/1.1 200 OK
Server: nginx/1.22.1
Date: Sat, 07 Dec 2024 06:59:34 GMT
Content-Type: text/html
Content-Length: 20
Connection: keep-alive
Last-Modified: Sat, 07 Dec 2024 06:43:42 GMT
ETag: "6753ee9e-14"
X-Host: static.com
X-Cache: HIT
Accept-Ranges: bytes
```

### IP透传

```
Nginx的IP地址透传是一个在网络传输中保留和传递源IP地址和目标IP地址的功能。IP透传，即IP透明传输，是指在网络传输过程中，源IP地址和目标IP地址能够完整地保留和传递，中间的网络设备（如代理服务器、负载均衡器等）不会修改这些IP地址。
```

```
记录真实客户端IP地址的日志：
 	在Nginx作为反向代理服务器时，默认情况下，后端服务器只能看到Nginx的IP地址，而无法看到客户端的真实IP地址。通过IP透传，后端服务器可以记录客户端的真实IP地址，便于日志分析和问题排查。
访问控制：
 	在一些应用场景中，需要根据客户端的IP地址进行访问控制。通过IP透传，可以确保后端服务器能够获取到客户端的真实IP地址，从而进行准确的访问控制。
```

配置方法

```powershell
server {
    listen 80;
    server_name example.com;
    location / {
        # 用于记录客户端的真实IP地址。
        proxy_set_header X-Real-IP $remote_addr;
        # 用于记录经过的代理服务器的IP地址列表，最左边的为客户端IP，右边为经过的代理服务器IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # 用于记录原始请求的协议（http或https）
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```powershell
root@ubuntu-43:conf.d # vim vhost.conf 
root@ubuntu-43:conf.d # cat vhost.conf 
proxy_cache_path /tmp/proxycache levels=1:2 keys_zone=proxycache:20m inactive=60s max_size=1g;
server {
  listen 80 default_server;
  server_name hzk.com;
  root /data/server/nginx/web1;
  location /api {
    proxy_pass http://10.0.0.44:8080;
    proxy_set_header Host "api.com";
  }
  location /static {
    rewrite ^/static(.*)$ /index.html break;  # 重写url
    proxy_pass http://10.0.0.60;
    proxy_set_header Host "static.magedu.com";
    # 将客户端IP追加请求报文中X-Real-IP,记录客户端的真实IP地址。
    proxy_set_header X-Real-IP $remote_addr;
    # 将客户端IP追加请求报文中X-Forwarded-For首部字段
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
  }
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf.d # systemctl restart nginx
root@ubuntu-42:~ # curl hzk.com/static/wdas
客户端：10.0.0.43---真实ip: 10.0.0.42----地址列表:\10.0.0.42
```

#### 多级穿透

```powershell
用于记录客户端的真实IP地址。注意: X-Real-IP通常只记录上一级的IP地址，如果有多级代理，则可能不准确。
proxy_set_header X-Real-IP $remote_addr;
用于记录经过的代理服务器IP地址列表，以及客户端的真实IP地址（列表最左侧）。这是多级IP透传中最常用的字段。
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;：
```

```powershell
Proxy Server - First
root@ubuntu-43:conf.d # vim vhost.conf
root@ubuntu-43:conf.d # cat vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /static {
    proxy_pass http://10.0.0.151;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf.d # systemctl restart nginx
```

```powershell
Proxy Server - Second 
root@ubuntu-151:conf.d # vim vhost.conf 
root@ubuntu-151:conf.d # cat vhost.conf 
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  location /static {
    proxy_pass http://10.0.0.60;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
root@ubuntu-151:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-151:conf.d # systemctl restart nginx
```

```powershell
client
root@ubuntu-42:~ # curl 10.0.0.43/static
客户端：10.0.0.151---真实ip: 10.0.0.43----地址列表:10.0.0.42, 10.0.0.43
```

## 负载均衡

```
upstream name { server address [parameters]; } 	# 定义一个后端服务器组，可以包含多台服务器，
 												# 定义好后在 proxy_pass 指令中引用
												# 作用域 http
server address [parameters]; 		# 在 upstream 中定义一个具体的后端服务器，作用域upstream
 									# address 指定后端服务器 可以是IP地址|主机名|Socket
```

```powershell
parameters 是可选参数，具体有以下几个属性
weight=number 指定该 server 的权重，默认值都是1
max_conns=number 该 Server 的最大活动连接数，达到后将不再给该 Server 发送请求，
 					可以保持的连接数=代理服务器的进程数乘以max_conns,默认值0，表示不限制
max_fails=number 后端服务器的下线条件,后端服务器连续进行检测多少次失败,而后标记为不可用，默认为1
					当客户端访问时，才会利用TCP触发对探测后端服务器健康性检查，而非周期性的探测
fail_timeout=time 后端服务器的上线条件,后端服务器连续进行检测多少次成功,而后标记为可用,默认为10秒
	backup 标记该 Server 为备用，当所有后端服务器不可用时，才使用此服务器
	down 标记该 Server 临时不可用，可用于平滑下线后端服务器，新请求不再调度到此服务器，原有连接不受影响
```

负载均衡调度算法

```powershell
hash key [consistent]; 	# 使用自行指定的 Key 做 hash 运算后进行调度，Key 可以是变量，
 						# 比如请求头中的字段，URI等，如果对应的server条目配置发生了变化，
 						# 会导致相同的 key 被重新 hash
 						# consistent 表示使用一致性 hash，
						# 此参数确保该 upstream 中的 server 条目发生变化时，
 						# 尽可能少的重新 hash，适用于做缓存服务的场景，提高缓存命中率
 						# 作用域 upstream
ip_hash; 				# 源地址hash调度方法，基于的客户端的remote_addr做hash计算，
 						# 以实现会话保持，作用域 upstream
least_conn; 			# 最少连接调度算法，优先将客户端请求调度到当前连接最少的后端服务器,
						# 相当于LVS中的 LC 算法，配合权重，能实现类似于 LVS 中的WLC 算法
						# 作用域 upstream
keepalive connections;	# 为每个 worker 进程保留最多多少个空闲保活连接数，
 						# 超过此值，最近最少使用的连接将被关闭
						# 默认不设置，作用域 upstream
keepalive_time time; 	# 空闲连接保持的时长，超过该时间，一直没使用的空闲连接将被销毁
 						# 默认值 1h，作用域 upstream
```

### 负载均衡案例

```
Client					10.0.0.42
Proxy Server			10.0.0.43
Real Server - 1 		10.0.0.151
Real Server - 2 		10.0.0.60
```

```powershell
43
root@ubuntu-43:conf.d # vim vhost.conf 
root@ubuntu-43:conf.d # cat vhost.conf 
upstream group1 {
 server 10.0.0.151;
 server 10.0.0.60;
}
server {
  listen 80 default_server;
  location / {
    proxy_pass http://group1;
  }
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf.d # systemctl restart nginx
```

```powershell
151
root@ubuntu-151:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-151:conf.d # systemctl restart nginx.service 
```

```powershell
60
root@openEuler-60:~ # cat > /etc/nginx/default.d/simple.conf<<-eof
location / {
  return 200 "RealServer-2\n";
}
eof
root@openEuler-60:~ # nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@openEuler-60:~ # systemctl restart nginx.service
```

```powershell
42
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-1
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-1
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
结果显示
 	客户端访问，轮循调度到后端服务器
```

###  多负载Upstream

```powershell
root@ubuntu-43:conf.d # cat vhost.conf 
upstream group1 {
 server 10.0.0.60;
}
upstream group2 {
 server 10.0.0.151;
}
server {
  listen 80 default_server;
  location / {
    proxy_pass http://group1;
    }
}
server {
  listen 80 ;
  location / {
    proxy_pass http://group2;
    }
}
root@ubuntu-43:conf.d # nginx -t
root@ubuntu-43:conf.d # systemctl restart nginx
root@ubuntu-42:~ # curl -H "Host: hzk1.com" 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl -H "Host: hzk2.com" 10.0.0.43
RealServer-1
```

### 权重

```
server address [weight]; 	# 在 upstream 中定义一个具体的后端服务器，作用域 upstream
 							# address 指定后端服务器 可以是IP地址|主机名|Socket
 							# weight=number 指定该 server 的权重，默认值都是1
```

```powershell
root@ubuntu-43:conf.d # vim vhost.conf
root@ubuntu-43:conf.d # cat vhost.conf 
upstream group1 {
 server 10.0.0.60 weight=3;
 server 10.0.0.151;
}
server {
  listen 80 default_server;
  location / {
    proxy_pass http://group1;
    }
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf.d # systemctl restart nginx

root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-1
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-1
```

### 限制最大活动连接数

```
server address [max_conns]; 	# 在 upstream 中定义一个具体的后端服务器，作用域 upstream
 								# address 指定后端服务器 可以是IP地址|主机名|Socket
 								# max_conns=number 该 Server 的最大活动连接数，达到后将不再给
 								# 该 Server 发送请求，默认值0，表示不限制
 								# 主机可保持的连接数=进程数 * max_conns
```

```powershell
root@ubuntu-43:conf.d # vim vhost.conf
root@ubuntu-43:conf.d # cat vhost.conf 
upstream group1 {
 server 10.0.0.60;
 server 10.0.0.151 max_conns=2;
}
server {
  listen 80 default_server;
  location / {
    proxy_pass http://group1;
    }
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf.d # systemctl restart nginx


root@ubuntu-151:conf.d # cat > vhost.conf <<-eof
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  limit_rate 10k;
}
eof
root@ubuntu-151:conf.d #  dd if=/dev/zero of=/data/server/nginx/web1/10.img bs=10M count=1
1+0 records in
1+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.0153955 s, 681 MB/s
root@ubuntu-151:conf.d # systemctl restart nginx.service 

root@openEuler-60:~ # cat > /etc/nginx/default.d/simple.conf<<-eof
limit_rate 10k;
eof
root@openEuler-60:~ # systemctl start nginx
root@openEuler-60:~ # dd if=/dev/zero of=/usr/share/nginx/html/10.img bs=10M count=1
输入了 1+0 块记录
输出了 1+0 块记录
10485760 字节 (10 MB, 10 MiB) 已复制，0.00242069 s，4.3 GB/s


root@ubuntu-42:~ # for i in {1..20}; do wget 10.0.0.43/10.img & done
root@ubuntu-151:conf.d # ss -tnep | grep 80
ESTAB 0      0               10.0.0.151:80           10.0.0.43:35308 users:(("nginx",pid=3673,fd=11)) uid:999 ino:42587 sk:1013 cgroup:/system.slice/nginx.service <->    
ESTAB 0      0               10.0.0.151:80           10.0.0.43:35306 users:(("nginx",pid=3673,fd=3)) uid:999 ino:42586 sk:1014 cgroup:/system.slice/nginx.service <->  
```

###  备用主机backup

```
backup 标记该 Server 为备用，当所有后端服务器不可用时，才使用此服务器
```

```powershell
root@ubuntu-43:conf.d # vim vhost.conf 
root@ubuntu-43:conf.d # cat vhost.conf 
upstream group1 {
        #10.0.0.60 备用
 server 10.0.0.60 backup;
 server 10.0.0.151;
}
server {
  listen 80 default_server;
  location / {
    proxy_pass http://group1;
    }
}
root@ubuntu-43:conf.d # nginx -t
root@ubuntu-43:conf.d # systemctl restart nginx

root@ubuntu-42:~ # curl 10.0.0.151
RealServer-1
root@ubuntu-42:~ # curl 10.0.0.151
RealServer-1
root@ubuntu-42:~ # curl 10.0.0.151
RealServer-1
root@ubuntu-42:~ # curl 10.0.0.151

停掉151主机后
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
```

### 应用平滑下线

```
down 标记该 Server 临时不可用，可用于平滑下线后端服务器，新请求不再调度到此服务器，原有连接不受影响
```

### 源IP地址hash

```
ip_hash; 	# 源地址hash调度方法，基于的客户端的remote_addr做hash计算，
 			# 以实现会话保持，作用域 upstream

ip_hash 算法只使用 IPV4 的前 24 位做 hash 运算，如果客户端IP前24位一致，则会被调度到同一台后端服务器
```

```
oot@ubuntu-43:conf.d # vim vhost.conf 
root@ubuntu-43:conf.d # cat vhost.conf 
upstream group1 {
  ip_hash;
  server 10.0.0.60;
  server 10.0.0.151;
}
  server {
    listen 80 default_server;
    location / {
      proxy_pass http://group1;
      }
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:conf.d # systemctl restart nginx


root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
root@ubuntu-42:~ # curl 10.0.0.43
RealServer-2
```

### 使用自行指定的 key 做 hash 调度

```
hash key [consistent]; 	# 使用自行指定的 Key 做 hash 运算后进行调度，Key 可以是变量，
 						# 比如请求头中的字段，URI等，如果对应的server条目配置发生了变化，
 						# 会导致相同的 key 被重新 hash
 						# consistent 表示使用一致性 hash，
						# 此参数确保该 upstream 中的 server 条目发生变化时，
						# 尽可能少的重新 hash，适用于做缓存服务的场景，提高缓存命中率
 						# 作用域 upstream
```

### 普通hash

```
hash $remote_addr;
						#权重一样的hash配置示例
```

加权重的hash效果

```
三台 server 权重不一样，调度算法 hash($remoute_addr)%(1+2+3)，值为 0，1，2，3，4，5
 0 调度到 111
 1，2调度到 112
 3，4，5调度到 113
upstream group1{
 hash $remote_addr;
 server 10.0.0.111 weight=1;
 server 10.0.0.112 weight=2;
 server 10.0.0.113 weight=3;
}
```

### 一致性哈希

```
一致性哈希（Consistent Hashing）是一种用于分布式系统中数据分片和负载均衡的算法，其中的"hash环"是该算法的核心概念之一。
	在一致性哈希中，所有可能的数据节点或服务器被映射到一个虚拟的环上。这个环的范围通常是一个固定的哈希空间，比如0到2^32-1，每个数据节点或服务器被映射到环上的一个点，通过对其进行哈希计算得到。这个哈希值的范围也是在0到2^32-1之间。
 	在这个环上，数据会被分散到最接近它的节点。当有新的数据要存储时，首先通过哈希计算得到该数据的哈希值，然后在环上找到离这个哈希值最近的节点，将数据存储在这个节点上。同样，当要查询数据时，也是通过哈希计算得到数据的哈希值，然后找到最近的节点进行查询。
 	由于哈希环是一个环形结构，节点的添加和删除对整体的影响相对较小。当添加或删除节点时，只有相邻的节点受到影响，而其他节点保持不变。这使得一致性哈希算法在分布式系统中能够提供较好的负载均衡性能，同时减小了数据迁移的开销。
 	总的来说，一致性哈希中的哈希环是通过哈希计算将数据节点映射到环上，以实现数据分片和负载均衡的分布式算法。
```

```powershell
upstream group1{
 hash $remote_addr consistent; #添加一致性 hash 参数
 server 10.0.0.210;
 server 10.0.0.159;
 server 10.0.0.213;
}
```

| hash($remoute_addr) | hash($remoute_addr)%(2^32-1) | Node | Server       |
| ------------------- | ---------------------------- | ---- | ------------ |
| 1                   | 1                            | 1    | 顺时针第一个 |
| 2                   | 2                            | 2    | 顺时针第一个 |
| 3                   | 3                            | 3    | 顺时针第一个 |
| 4                   | 4                            | 4    | 顺时针第一个 |
| 5                   | 5                            | 5    | 顺时针第一个 |
| ......              |                              |      |              |

![image-20241207185341884](5day-png\18hash环.png)

环偏移

```powershell
	在一致性哈希中，哈希环可能会面临的一个问题是环偏移（Ring Wrapping）。环偏移指的是哈希环上的某个区域过于拥挤，而其他区域相对空闲，这可能导致负载不均衡。为了解决这个问题，一些改进的一致性哈希算法引入了虚拟节点（Virtual Nodes）的概念。
	虚拟节点是对物理节点的一种扩展，通过为每个物理节点创建多个虚拟节点，将它们均匀地分布在哈希环上。这样一来，每个物理节点在环上的位置会有多个副本，而不是只有一个位置。这样一来，即使哈希环上的某个区域过于拥挤，也可以通过调整虚拟节点的数量来使得负载更均衡。
```

## 四层代理和负载

```powershell
	Nginx在1.9.0版本开始支持tcp模式的负载均衡，在1.9.13版本开始支持udp协议的负载，udp主要用于DNS的域名解析，其配置方式和指令和http 代理类似，其基于ngx_stream_proxy_module模块实现tcp负载，另外基于模块ngx_stream_upstream_module实现后端服务器分组转发、权重分配、状态监测、调度算法等高级功能。
	在 Nginx 配置中，stream 指令应该位于与 http 指令平行的顶层上下文中。默认情况下，stream 指令和 http 指令都是顶层指令，它们不能嵌套在彼此内部，也不能位于 server 或 location 块中。
注意：
 如果编译安装，需要指定 --with-stream 选项才能支持 ngx_stream_proxy_module模块
 ./configure 的 相关介绍如下：
  --with-stream                      	enable TCP/UDP proxy module
  --with-stream=dynamic              	enable dynamic TCP/UDP proxy module

```

准备Mysql环境

```powershell
root@ubuntu-44:~ # apt update;apt install mysql-server
root@ubuntu-44:~ # vim /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address            = 10.0.0.44
mysqlx-bind-address     = 10.0.0.44
skip-name-resolve       #跳过主机名反解
root@ubuntu-44:~ # systemctl start mysq
root@ubuntu-44:~ # mysql
mysql> create user proxyer@'10.0.0.%' identified by '123456';
mysql> flush privileges;
root@ubuntu-44:~ # mysql -h 10.0.0.44 -uproxyer -p123456 -e "select version();"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------------------+
| version()               |
+-------------------------+
| 8.0.40-0ubuntu0.24.04.1 |
+-------------------------+
root@ubuntu-44:~ # ss -tnulp | grep 3306
tcp   LISTEN 0      70                           10.0.0.44:33060      0.0.0.0:*    users:(("mysqld",pid=13479,fd=21))                     
tcp   LISTEN 0      151                          10.0.0.44:3306       0.0.0.0:*    users:(("mysqld",pid=13479,fd=23)) 
```

准备redis环境

```powershell
root@openEuler-60:~ # yum -y install redis -y
root@openEuler-60:~ # vim /etc/redis.conf
# bind 127.0.0.1 		# 注释掉
protected-mode no    	#关闭保护模式
root@openEuler-60:~ # systemctl restart redis
root@openEuler-60:~ # ss -tnulp | grep 6379
tcp   LISTEN 0      511          0.0.0.0:6379       0.0.0.0:*    users:(("redis-server",pid=21404,fd=7))                                           
tcp   LISTEN 0      511             [::]:6379          [::]:*    users:(("redis-server",pid=21404,fd=6)) 
```

客户端准备工具

```powershell
apt install mysql-client redis-tools
或
yum install mysql redis
mysql环境测试
root@ubuntu-42:~ # mysql -h 10.0.0.44 -uproxyer -p123456 -e "select version();"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------------------+
| version()               |
+-------------------------+
| 8.0.40-0ubuntu0.24.04.1 |
+-------------------------+
root@ubuntu-42:~ # redis-cli -h 10.0.0.60 -p 6379 info server
# Server
redis_version:4.0.14
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:6eb529b77a3f2394
redis_mode:standalone
os:Linux 6.6.0-28.0.0.34.oe2403.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:12.3.1
process_id:21404
run_id:d303dffe0324407f38cf0bbdc5869b889fa75e6a
tcp_port:6379
uptime_in_seconds:149
uptime_in_days:0
hz:10
lru_clock:5517083
executable:/usr/bin/redis-server
config_file:/etc/redis.conf
```

定制代理配置

```powershell
root@ubuntu-43:conf.d # nginx -V
nginx version: nginx/1.22.1
built by gcc 13.2.0 (Ubuntu 13.2.0-23ubuntu4) 
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/apps/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module --add-module=/usr/local/src/echo-nginx-module-0.63
root@ubuntu-43:conf.d # mkdir -p /apps/nginx/conf/stream.d
root@ubuntu-43:conf.d # mv /apps/nginx/conf/conf.d/vhost.conf /apps/nginx/conf/stream.d/
root@ubuntu-43:conf.d # vim ../nginx.conf
stream {
    include /apps/nginx/conf/stream.d/*.conf;
}
root@ubuntu-43:conf.d # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:stream.d # cd /apps/nginx/conf/stream.d/
root@ubuntu-43:stream.d # cat vhost.conf 
upstream mysqlserver {
    server 10.0.0.43:3306;
}

upstream redisserver {
    server 10.0.0.60:6379;
}

server {
    listen 3306;
    proxy_pass mysqlserver;
}

server {
    listen 6379;
    proxy_pass redisserver;
}
root@ubuntu-43:stream.d # nginx -t
root@ubuntu-43:stream.d # systemctl restart nginx
root@ubuntu-43:stream.d # systemctl restart nginx
root@ubuntu-43:stream.d # ss -tnulp | grep nginx
tcp   LISTEN 0      511                            0.0.0.0:6379      0.0.0.0:*    users:(("nginx",pid=3819,fd=7),("nginx",pid=3818,fd=7),("nginx",pid=3817,fd=7))
tcp   LISTEN 0      511                            0.0.0.0:3306      0.0.0.0:*    users:(("nginx",pid=3819,fd=6),("nginx",pid=3818,fd=6),("nginx",pid=3817,fd=6))
```

客户端测试

```powershell
root@ubuntu-42:~ # mysql -h 10.0.0.43 -uproxyer -p123456 -e "select version();"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------------------+
| version()               |
+-------------------------+
| 8.0.40-0ubuntu0.24.04.1 |
+-------------------------+
root@ubuntu-42:~ # redis-cli -h 10.0.0.43 -p 6379 info server
# Server
redis_version:4.0.14
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:6eb529b77a3f2394
redis_mode:standalone
os:Linux 6.6.0-28.0.0.34.oe2403.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:12.3.1
process_id:21404
run_id:d303dffe0324407f38cf0bbdc5869b889fa75e6a
tcp_port:6379
uptime_in_seconds:1806
uptime_in_days:0
hz:10
lru_clock:5518740
executable:/usr/bin/redis-server
config_file:/etc/redis.conf
```

## FastCGI代理

```
fastcgi_index name; 		# 后端 FastCGI 服务器默认资源，默认值为空
 							# 作用域 http, server, location
fastcgi_pass address; 		# 指定后端 FastCGI 服务器地址，可以写 IP:port，也可以指定socket 文件
 							# 作用域 location, if in location
fastcgi_param parameter value [if_not_empty];
 							# 设置传递给FastCGI服务器的参数值，
							# 可用于将Nginx的内置变量赋值给自定义key
 							# 作用域 http, server, location
```

1、准备php环境

```powershell
root@ubuntu-43:stream.d # apt install php -y
root@ubuntu-43:stream.d # apt install php8.3-fpm
确认版本
root@ubuntu-43:stream.d # php -v
PHP 8.3.6 (cli) (built: Sep 30 2024 15:17:17) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.6, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.6, Copyright (c), by Zend Technologies
php工作目录
root@ubuntu-43:stream.d # ls /etc/php/8.3/
apache2  cli  mods-available
root@ubuntu-43:stream.d # ls /etc/php/8.3/apache2/
conf.d  php.ini
检查服务状态
root@ubuntu-43:stream.d # systemctl is-active php-fpm.service
inactive
root@ubuntu-43:stream.d # vim /etc/php/8.3/fpm/pool.d/www.conf
修改监听端口
; listen = /run/php-fpm/www.sock
listen = 127.0.0.1:9000
root@ubuntu-43:stream.d # systemctl restart php8.3-fpm.service
root@ubuntu-43:stream.d # ss -tnulp | grep php
tcp   LISTEN 0      4096                         127.0.0.1:9000      0.0.0.0:*    users:(("php-fpm8.3",pid=12184,fd=10),("php-fpm8.3",pid=12183,fd=10),("php-fpm8.3",pid=12182,fd=8))
root@ubuntu-43:nginx # vim conf/conf.d/vhost.conf
root@ubuntu-43:nginx # cat conf/conf.d/vhost.conf 
root@ubuntu-43:nginx # cat conf/conf.d/vhost.conf
upstream phpserver {
    server 127.0.0.1:9000;
}
server {
  listen 80 default_server;
  root /data/server/nginx/web1;
  #反向代理 php
  location ~ \.php$ {
    fastcgi_pass phpserver;
    fastcgi_index index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
  #定制首页
  location / {       
     index index.php;
     autoindex on;
  }
}
root@ubuntu-43:nginx # nginx -t
nginx: the configuration file /apps/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /apps/nginx/conf/nginx.conf test is successful
root@ubuntu-43:nginx # systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
这里nginx启动不成功,查看端口，安装php会把apache2启动
root@ubuntu-43:nginx # netstat -tulnp | grep :80
tcp6       0      0 :::80                   :::*                    LISTEN      10859/apache2       
root@ubuntu-43:nginx # systemctl stop apache2
root@ubuntu-43:nginx # systemctl restart nginx
echo '<?php phpinfo(); ?>' > /data/server/nginx/web1/index.php
echo '<?php echo "test.php"; ?>' > /data/server/nginx/web1/test.php
root@ubuntu-43:nginx # systemctl restart nginx
root@ubuntu-43:nginx # curl 10.0.0.43/test.php
test.php
```

部署Discuz

```powershell
root@ubuntu-43:nginx # mkdir discuz
root@ubuntu-43:discuz # wget https://gitee.com/Discuz/DiscuzX/attach_files/1773967/download -O Discuz_X3.5_SC_UTF8_20240520.zip
root@ubuntu-43:discuz # unzip Discuz_X3.5_SC_UTF8_20240520.zip -d discuz/
root@ubuntu-43:discuz # rm -rf /data/server/nginx/web1/*
root@ubuntu-43:discuz # mv discuz/upload/ /data/server/nginx/web1/
root@ubuntu-43:discuz # chown -R nginx:nginx /data/server/nginx/web1/
mysql> create database discuz;
mysql> create user 'discuzer'@'10.0.0.%' identified by '123456';
mysql> grant all on discuz.* to 'discuzer'@'10.0.0.%';
mysql>  flush privileges;

root@ubuntu-43:discuz # mysql -h 10.0.0.44 -p123456 -u discuzer -e "select user();"
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| user()             |
+--------------------+
| discuzer@10.0.0.43 |
+--------------------+
root@ubuntu-43:discuz # apt install mysql-client -y
root@ubuntu-43:discuz # mysql -h 10.0.0.43 -p123456 -u discuzer -e "select user();"
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| user()             |
+--------------------+
| discuzer@10.0.0.43 |
+--------------------+
root@ubuntu-43:discuz # apt install php8.3-cli php8.3-xml php8.3-mysqli
```

## Tenginx

```powershell
root@ubuntu-42:~ # apt update
root@ubuntu-42:~ # apt install gcc libssl-dev libpcre3-dev zlib1g-dev make
root@ubuntu-42:~ # groupadd -r -g 888 nginx
root@ubuntu-42:~ # useradd -r -g nginx -u 888 -s /sbin/nologin nginx
root@ubuntu-42:~ # id nginx
uid=888(nginx) gid=888(nginx) groups=888(nginx)
root@ubuntu-42:~ # mkdir /data/softs
root@ubuntu-42:~ # cd /data/softs
root@ubuntu-42:softs # wget https://tengine.taobao.org/download/tengine-3.1.0.tar.gz
编译安装
root@ubuntu-42:softs # tar xf tengine-3.1.0.tar.gz 
root@ubuntu-42:softs # ls
tengine-3.1.0  tengine-3.1.0.tar.gz
root@ubuntu-42:softs # cd tengine-3.1.0/
root@ubuntu-42:tengine-3.1.0 # ls
AUTHORS.te  auto  CHANGES  CHANGES.cn  CHANGES.te  conf  configure  contrib  docs  html  LICENSE  man  modules  packages  README.markdown  src  tests  THANKS.te
root@ubuntu-42:tengine-3.1.0 # ./configure --prefix=/data/server/tengine --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre
root@ubuntu-42:tengine-3.1.0 # make && make install
root@ubuntu-42:tengine-3.1.0 # cd /data/server/tengine/
root@ubuntu-42:tengine # ls
conf  html  logs  sbin
root@ubuntu-42:tengine # ./sbin/nginx -V
Tengine version: Tengine/3.1.0
nginx version: nginx/1.24.0
built by gcc 13.2.0 (Ubuntu 13.2.0-23ubuntu4) 
built with OpenSSL 3.0.13 30 Jan 2024
TLS SNI support enabled
configure arguments: --prefix=/data/server/tengine --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre
启动
root@ubuntu-42:tengine # /data/server/tengine/sbin/nginx
关闭
root@ubuntu-42:tengine # /data/server/tengine/sbin/nginx -s stop
```

## openresty

```powershell
root@ubuntu-42:~ # apt update
root@ubuntu-42:~ # apt install gcc libssl-dev libpcre3-dev zlib1g-dev make
root@ubuntu-42:~ # groupadd -r -g 888 nginx
root@ubuntu-42:~ # useradd -r -g nginx -u 888 -s /sbin/nologin nginx
root@ubuntu-42:~ # id nginx
uid=888(nginx) gid=888(nginx) groups=888(nginx)
root@ubuntu-42:~ # mkdir /data/softs
root@ubuntu-42:~ # cd /data/softs
root@ubuntu-42:softs # wget https://openresty.org/download/openresty-1.25.3.1.tar.gz
root@ubuntu-42:softs # tar xf openresty-1.25.3.1.tar.gz 
root@ubuntu-42:softs # cd openresty-1.25.3.1/
root@ubuntu-42:openresty-1.25.3.1 # ls
bundle  configure  COPYRIGHT  patches  README.markdown  README-windows.txt  util
root@ubuntu-42:openresty-1.25.3.1 # ./configure --prefix=/data/server/openresty --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module
root@ubuntu-42:openresty-1.25.3.1 # make && make install 
root@ubuntu-42:openresty-1.25.3.1 # ls /data/server/openresty/
bin  COPYRIGHT  luajit  lualib  nginx  pod  resty.index  site
root@ubuntu-42:~ # cd /data/server/openresty/nginx/sbin/
root@ubuntu-42:sbin # ./nginx
root@ubuntu-42:sbin # mkdir /data/server/openresty/lua -p
cd /data/server/openresty/lua
echo 'ngx.say("Hello, OpenResty with Lua!")' > test.lua
root@ubuntu-42:lua # cat > /data/server/openresty/nginx/conf/nginx.conf <<-eof
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    # Lua模块配置
    lua_package_path "/data/server/openresty/lua/?.lua;;";
    lua_shared_dict my_cache 10m;
    server {
        listen 80;
        server_name localhost;
        location /test {
            # 指定要执行的Lua脚本文件
            content_by_lua_file /data/server/openresty/lua/test.lua;
        }
        location /lua {
 default_type text/html;
            content_by_lua_block {
                ngx.say("<p>hello, world with lua</p>") 
            }
 }
    }
}
eof
root@ubuntu-42:lua # /data/server/openresty/nginx/sbin/nginx -s reload
root@ubuntu-42:lua # curl 10.0.0.42/test
Hello, OpenResty with Lua!
root@ubuntu-42:lua # curl 10.0.0.42/lua
<p>hello, world with lua</p>
```





