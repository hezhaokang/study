# 26、zabbix

## 监控方法

Google的四个黄金指标

常用于在服务级别帮助衡量终端用户体验、服务中断、业务影响等层面的问题，适用于应用及服务监控

- 延迟(Latency)

  服务请求所需要的时长，例如HTTP请求平均延迟

  应用程序响应时间会受到所有核心系统资源（包括网络、存储、CPU和内存）延迟的影响

  需要区分失败请求和成功请求

- 流量(Traffic)，也称为吞吐量

  衡量服务的容量需求，例如每秒处理的HTTP请求数QPS或者数据库系统的事务数量TPS 

  吞吐量指标包括每秒Web请求、API调用等示例，并且被描述为通常表示为每秒请求数的需求

- 错误(Errors)

  失败的请求（流量)的数量，通常以绝对数量或错误请求占请求总数的百分比表示，请求失败的速 率，用于衡量错误发生的情况

  例如：HTTP 500错误数等显式失败，返回错误内容或无效内容等隐式失败，以及由策略原因导致 的失败(例如强制要求响应时间超过30毫秒的请求视为错误)

- 饱和度(Saturation)

  衡量资源的使用情况,用于表达应用程序有多"满"

  资源的整体利用率，包括CPU（容量、配额、节流)、内存(容量、分配)、存储（容量、分配和 I/O 吞吐量)和网络

  例如：内存、CPU、I/O、磁盘等资源的使用量

## 安装

### 安装zabbix服务端

#### 官网步骤安装

```
https://www.zabbix.com/cn/download?zabbix=7.0&os_distribution=ubuntu&os_version=24.04&components=server_frontend_agent&db=mysql&ws=apache
```

```powershell
#安装Zabbix repository源
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
sed -i 's@https://repo.zabbix.com/@https://mirrors.tuna.tsinghua.edu.cn/zabbix/@' /etc/apt/sources.list.d/*

apt update

#安装Zabbix server，Web前端，agent
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent zabbix-get

#安装数据库，创建初始数据库
apt install mysql-server -y

#如果MySQL和ZabbixServer在同一台主机,此项可不改
vim /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address        = 0.0.0.0

create database zabbix character set utf8mb4 collate utf8mb4_bin;
CREATE USER 'zabbix'@'10.0.0.%' IDENTIFIED BY '123456';
grant all privileges on zabbix.* to 'zabbix'@'10.0.0.%';
set global log_bin_trust_function_creators = 1;


#导入初始架构和数据，系统将提示您输入新创建的密码。
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -h10.0.0.203 -uzabbix -p123456 zabbix
	
#为Zabbix server配置数据库
#编辑配置文件 /etc/zabbix/zabbix_server.conf

DBPassword=password

#启动Zabbix server和agent进程
#启动Zabbix server和agent进程，并为它们设置开机自启：

systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2

http://host/zabbix

上面网页填写的内容保存在这里
#如果是CentOS下面文件
/etc/zabbix/web/zabbix.conf.php
#如果是Ubuntu则下面文件
/usr/share/zabbix/conf/zabbix.conf.php 实质也是/etc/zabbix/web/zabbix.conf.php软链接
登录账号密码默认是
Admin
zabbix

#安装中文包
apt -y install language-pack-zh-hans

#支持ttf和ttc后缀的字体文件
cd /usr/share/zabbix/assets/fonts/
mv graphfont.ttf graphfont.ttf.bak
mv /root ./
mv msyh.ttc graphfont.ttf

#注意:字体文件路径和名称的定义在文件/usr/share/zabbix/include/defines.inc.php中配置
#可以修改下面FONT_NAME指定新字体件,注意不需加文件后缀
[root@ubuntu2404 ~]#grep FONT_NAME /usr/share/zabbix/include/defines.inc.php
define('ZBX_GRAPH_FONT_NAME',           'graphfont'); // font file name
define('ZBX_FONT_NAME', 'graphfont');

#zabbix配置文件
[root@ubuntu2404 fonts]#cat /etc/zabbix/zabbix_server.conf | grep -Ev '^#|^$'
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_server.pid
SocketDir=/run/zabbix
DBHost=10.0.0.203
DBName=zabbix
DBUser=zabbix
DBPassword=123456
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
FpingLocation=/usr/bin/fping
Fping6Location=/usr/bin/fping6
LogSlowQueries=3000
StatsAllowedIP=127.0.0.1
EnableGlobalScripts=0
```

#### 脚本安装

```powershell
#!/bin/bash

ZABBIX_MAJOR_VER=7.0
ZABBIX_VER=${ZABBIX_MAJOR_VER}-1

URL="mirror.tuna.tsinghua.edu.cn/zabbix"

MYSQL_HOST=localhost
MYSQL_ZABBIX_USER="zabbix@localhost"

#MYSQL_HOST=10.0.0.100
#MYSQL_ZABBIX_USER="zabbix@'10.0.0.%'"

MYSQL_ZABBIX_PASS='123456'
MYSQL_ROOT_PASS='123456'

FONT=msyh.ttc

ZABBIX_IP=`hostname -I|awk '{print $1}'`
GREEN="echo -e \E[32;1m"
END="\E[0m"

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

install_mysql () {
    [ $MYSQL_HOST != "localhost" ] && return 
    if [ $ID = "centos" -o $ID = "rocky" ] ;then
            VERSION_ID=`echo $VERSION_ID | cut -d . -f1`
            if [ ${VERSION_ID} == "8" ];then
            yum  -y install mysql-server
            systemctl enable --now mysqld
                elif [ ${VERSION_ID} == "7" ];then
                    yum -y install mariadb-server
                        systemctl enable --now mariadb
                else
                    color "不支持的操作系统,退出" 1
                fi 
    else
        apt update
        apt -y install mysql-server
        sed -i "/^bind-address.*/c bind-address  = 0.0.0.0" /etc/mysql/mysql.conf.d/mysqld.cnf
        systemctl restart mysql
    fi
    mysqladmin -uroot password $MYSQL_ROOT_PASS
    mysql -uroot -p$MYSQL_ROOT_PASS <<EOF
set global log_bin_trust_function_creators = 0;
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user $MYSQL_ZABBIX_USER identified by "$MYSQL_ZABBIX_PASS";
grant all privileges on zabbix.* to $MYSQL_ZABBIX_USER;
set global log_bin_trust_function_creators = 1;
quit
EOF
    if [ $? -eq 0 ];then
        color "MySQL数据库准备完成" 0
    else
        color "MySQL数据库配置失败,退出" 1
        exit
    fi
}

install_zabbix () {
    if [ $ID = "centos" -o $ID = "rocky" ] ;then 
        rpm -Uvh https://${URL}/zabbix/${ZABBIX_MAJOR_VER}/rhel/${VERSION_ID}/x86_64/zabbix-release-${ZABBIX_VER}.el${VERSION_ID}.noarch.rpm
        if [ $? -eq 0 ];then
                color "YUM仓库准备完成" 0
        else
            color "YUM仓库配置失败,退出" 1
                    exit
            fi
            sed -i "s#repo.zabbix.com#${URL}#" /etc/yum.repos.d/zabbix.repo
            if [[ ${VERSION_ID} == 8 ]];then 
                    yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent zabbix-get langpacks-zh_CN
        else 
                   yum -y install zabbix-server-mysql zabbix-agent  zabbix-get
                  yum -y install centos-release-scl
                   rpm -q yum-utils  || yum -y install yum-utils
                 yum-config-manager --enable zabbix-frontend
                yum -y install zabbix-web-mysql-scl zabbix-apache-conf-scl
        fi
    else 
                wget https://${URL}/zabbix/${ZABBIX_MAJOR_VER}/ubuntu/pool/main/z/zabbix-release/zabbix-release_${ZABBIX_VER}+ubuntu${VERSION_ID}_all.deb
            if [ $? -eq 0 ];then
                color "APT仓库准备完成" 0
            else
                color "APT仓库配置失败,退出" 1
            exit
        fi
        dpkg -i zabbix-release_${ZABBIX_VER}+ubuntu${VERSION_ID}_all.deb
        sed -i.bak "s#repo.zabbix.com#${URL}#" /etc/apt/sources.list.d/zabbix.list
        apt update
        apt -y install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent zabbix-get language-pack-zh-hans 
    fi
}

config_mysql_zabbix () {
        if [ -f "$FONT" ] ;then 
            mv /usr/share/zabbix/assets/fonts/graphfont.ttf{,.bak}
                cp  "$FONT" /usr/share/zabbix/assets/fonts/graphfont.ttf
        else
                color "缺少字体文件!" 1
        fi
        if [ $MYSQL_HOST = "localhost" ];then
       zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p$MYSQL_ZABBIX_PASS -h$MYSQL_HOST zabbix
       mysql -uroot -p$MYSQL_ROOT_PASS -e "set global log_bin_trust_function_creators = 0"
        fi
        sed -i -e "/.*DBPassword=.*/c DBPassword=$MYSQL_ZABBIX_PASS" -e "/.*DBHost=.*/c DBHost=$MYSQL_HOST" /etc/zabbix/zabbix_server.conf
        if [ $ID = "centos" -o $ID = "rocky" ];then
            if [[ ${VERSION_ID} == 8 ]];then            
           sed -i -e "/.*date.timezone.*/c php_value[date.timezone] = Asia/Shanghai" -e "/.*upload_max_filesize.*/c php_value[upload_max_filesize] = 20M" /etc/php-fpm.d/zabbix.conf
           systemctl enable --now zabbix-server zabbix-agent httpd php-fpm
                else
           sed -i "/.*date.timezone.*/c php_value[date.timezone] = Asia/Shanghai" /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
           systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
           systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
                fi
        else
        sed -i "/date.timezone/c php_value date.timezone Asia/Shanghai" /etc/apache2/conf-available/zabbix.conf
        chown -R www-data.www-data /usr/share/zabbix/
        systemctl enable  zabbix-server zabbix-agent apache2
        systemctl restart  zabbix-server zabbix-agent apache2
    fi
    if [ $?  -eq 0 ];then  
        echo 
        color "ZABBIX-${ZABBIX_VER}安装完成!" 0
        echo "-------------------------------------------------------------------"
        ${GREEN}"请访问: http://$ZABBIX_IP/zabbix"${END}
    else
        color "ZABBIX-${ZABBIX_VER}安装失败!" 1
        exit
    fi
}

install_mysql

install_zabbix

config_mysql_zabbix
```

运行脚本之前要先准备字体文件`msyh.ttc`

### 安装 Zabbix Agent

```powershell
#ubuntu2404
#在客户端上下载源
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
#更新为清华源
sed -i 's@https://repo.zabbix.com/@https://mirrors.tuna.tsinghua.edu.cn/zabbix/@' /etc/apt/sources.list.d/*
apt update

#ubuntu2204
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu22.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu22.04_all.deb
sed -i 's@https://repo.zabbix.com/@https://mirrors.tuna.tsinghua.edu.cn/zabbix/@' /etc/apt/sources.list.d/*
apt update

#安装软件
apt install zabbix-agent

systemctl restart zabbix-agent
systemctl enable zabbix-agent

#zabbix-agent配置文件
[root@ubuntu2404 ~]#cat /etc/zabbix/zabbix_agentd.conf | grep -Ev '^#|^$'
PidFile=/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.0.0.200
ListenPort=10050
ListenIP=0.0.0.0
ServerActive=10.0.0.200
Hostname=Zabbix server
Include=/etc/zabbix/zabbix_agentd.d/*.conf

```

### 安装 Zabbix Agent2

```powershell
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
sed -i 's@https://repo.zabbix.com/@https://mirrors.tuna.tsinghua.edu.cn/zabbix/@' /etc/apt/sources.list.d/*
apt update

apt install zabbix-agent2 zabbix-agent2-plugin-*

systemctl restart zabbix-agent2
systemctl enable zabbix-agent2

[root@ubuntu2404 ~]#grep -vE '^$|#' /etc/zabbix/zabbix_agent2.conf 
PidFile=/var/run/zabbix/zabbix_agent2.pid
LogFile=/var/log/zabbix/zabbix_agent2.log
LogFileSize=0
Server=10.0.0.200
ListenPort=10050
ListenIP=0.0.0.0
ServerActive=10.0.0.200
Hostname=Zabbix server
Include=/etc/zabbix/zabbix_agent2.d/*.conf
PluginSocket=/run/zabbix/agent.plugin.sock
ControlSocket=/run/zabbix/agent.sock
Include=/etc/zabbix/zabbix_agent2.d/plugins.d/*.conf
```

## 案例：将Zabbix Server的 MySQL 数据库迁移到独立的 MySQL 服务器

```powershell
#停止 Zabbix 服务
[root@ubuntu2404 ~]#systemctl stop zabbix-server.service
#备份数据库
[root@ubuntu2404 ~]#mysqldump -uzabbix -p123456 -B zabbix -f --single-transaction > /data/zabbix.sql
[root@ubuntu2404 ~]#ls /data/
zabbix.sql
[root@ubuntu2404 ~]#scp /data/zabbix.sql 10.0.0.100:

[root@ubuntu2404 ~]#systemctl disable --now mysql

#在独立数据库服务器上安装并恢复数据库
apt update && apt -y install mysql-server
vim /etc/mysql/mysql.conf.d/mysqld.cnf
#注释两行
#bind-address       = 127.0.0.1
#mysqlx-bind-address   = 127.0.0.1
#或者
[root@ubuntu2204 ~]#sed -i '/127.0.0.1/s/^/#/' /etc/mysql/mysql.conf.d/mysqld.cnf

[root@ubuntu2204 ~]#systemctl restart mysql.service 

#重新授权用户允许远程连接zabbix数据库
[root@ubuntu2204 ~]#mysql
mysql> create user zabbix@'10.0.0.%' identified by '123456';
mysql> grant all privileges on zabbix.* to zabbix@'10.0.0.%';

[root@ubuntu2204 ~]#mysql -e 'set global log_bin_trust_function_creators=1'
[root@ubuntu2204 ~]#mysql -uroot < zabbix.sql
[root@ubuntu2204 ~]#mysql -e 'set global log_bin_trust_function_creators=0'

#将php的配置指向新的数据库服务器IP
[root@ubuntu2404 ~]#vim /usr/share/zabbix/conf/zabbix.conf.php
$DB['SERVER']           = '10.0.0.101';

#将Zabbix Server的配置指向新的数据库服务器IP
[root@ubuntu2204 ~]#vim /etc/zabbix/zabbix_server.conf
DBHost=10.0.0.101
DBPort=3306

#重启服务生效
systemctl start zabbix-server.service
```



## 监控 Nginx 服务

```powershell
#Ubuntu系统
[root@ubuntu2204 ~]#apt update && apt -y install nginx
[root@ubuntu2204 ~]#vim /etc/nginx/sites-enabled/default
server {
   .....
#添加下面三行，Zabbix默认监控/basic_status,此处为/status，需要和zabbix的模板定义的路径要保持一致  
   location /status {
       stub_status;
   }
   ......
}
[root@ubuntu2204 ~]#nginx -s reload

#rocky系统
[root@centos8 ~]#yum -y install nginx 
[root@centos8 ~]#vim /etc/nginx/nginx.conf
http {
   server {
 location / {
       }
        #添加下面三行，Zabbix默认监控/basic_status,此处为/status，需要和zabbix的模板定义的路径要保持一致  
       location = /status {    
           stub_status;
         }

[root@centos8 ~]#systemctl enable --now nginx

```

#### 1 使用Template App Nginx by HTTP

![image-20250114171706109](5day-png\26使用http监控nginx1.png)

![image-20250114171754233](5day-png\26使用http监控nginx2.png)







#### 2 使用Template App Nginx by Zabbix agent

![image-20250114172639652](5day-png\26使用agent监控nginx1.png)

![image-20250114172724041](5day-png\26使用agent监控nginx2.png)







## 监控 PHP-FPM 服务

```powershell
#Ubuntu24安装配置PHP-FPM
[root@ubuntu2404 ~]#apt update && apt -y install php-fpm nginx
[root@ubuntu2404 ~]#vim /etc/php/8.3/fpm/pool.d/www.conf
listen = /run/php/php8.3-fpm.sock
pm.status_path = /status
ping.path = /ping
[root@ubuntu2404 ~]#systemctl restart php8.3-fpm.service

#Ubuntu22安装配置PHP-FPM
[root@ubuntu2204 ~]#vim /etc/php/8.1/fpm/pool.d/www.conf
;listen = 127.0.0.1:9000
listen = /run/php/php8.1-fpm.sock
pm.status_path = /php_status                #Zabbix系统默认监控路径/status
ping.path = /ping
[root@ubuntu2204 ~]#systemctl restart php8.1-fpm.service 

#红帽系统安装配置PHP-FPM
[root@centos8 ~]#yum -y install php-fpm nginx
[root@centos8 ~]#vim /etc/php-fpm.d/www.conf
listen=127.0.0.1:9000
pm.status_path = /php_status                #Zabbix系统默认监控路径/status
ping.path = /ping
[root@centos8 ~]#systemctl enable --now php-fpm.service


#修改nginx的配置
#ubuntu24.04
[root@ubuntu2404 ~]#vim /etc/nginx/sites-enabled/default
 server {
   .......
   location ~ ^/(ping|status)$ {
       include fastcgi_params;
       fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        #fastcgi_pass 127.0.0.1:9000;
        #fastcgi_param PATH_TRANSLATED $document_root$fastcgi_script_name;
       fastcgi_param SCRIPT_FILENAME  $document_root$fastcgi_script_name;
   }
[root@ubuntu2404 ~]#nginx -s reload
```



### PHP-FPM by HTTP

![image-20250114174340875](5day-png\26使用http监控php-fpm1.png)

![image-20250114174422168](5day-png\26使用http监控php-fpm2.png)





### PHP-FPM by Zabbix agent

![image-20250114181748967](5day-png\26使用agent监控php-fpm1.png)

![image-20250114181849827](5day-png\26使用agent监控php-fpm2.png)

 



## 自定义模板 Templates 和监控项 Items

客户端可以自定义监控项，在Zabbix Agent 配置文件添加内容,格式如下

```powershell
#cat /etc/zabbix/zabbix_agentd.conf
#cat /etc/zabbix/zabbix_agent2.conf
UserParameter=<key>,<shell command>
#支持参数
UserParameter=<key>[*],<shell command> $1 $2 .....

Include=/etc/zabbix/zabbix_agentd.d/*.conf
#或者创建独立的自定义文件
#cat /etc/zabbix/zabbix_agentd.d/*.conf
#cat /etc/zabbix/zabbix_agent2.d/*.conf
UserParameter=<key>,<shell command>
#支持参数
UserParameter=<key>[*],<shell command> $1 $2 .....
```

- key 必须整个系统唯一。注意大小写是敏感的, Key名允许的字符如下

  ```
  0-9a-zA-Z_-.
  ```

  key 使用 [*] 用于定义该key接受括号内的参数。参数需在配置监控项时给出；参数禁止使用下列字符：

  ```
  \ ’ ” ` * ? [ ] { } ~ $ ! & ; ( ) <>
  ```

- Command：命令用于生成key对应的值。

  可以在命令中使用位置引用$1 … $9来引用监控项Key中的相应参数。Zabbix解析监控项Key的[]中包含的参数，并相应地替换$1，…，$9。$0会替换为完整的原始命令（在对$0，…，$9执行替换之前的命令）运行。不管位置参数（$0,…,$9)是用双引号( “ )还是单引号( ’ )括起来，都会解析位置引用

**测试监控项**

在Zabbix Agent 上执行测试

```powershell
#不需要重启服务：
zabbix_agentd -t "在客户端定义的key名[arg1,arg2,...]"
zabbix_agent2 -t "在客户端定义的key名[arg1,arg2,...]"
```

在Zabbix Server上可以使用zabbix_get工具获取自定义监控项

```powershell
#需要重启服务：systemctl restart zabbix-agent2.service
zabbix_get -s 客户端IP -p 10050 -k "在客户端定义的key名[arg1,arg2,...]"
```

**注意:** 

- 如果用脚本实现首行必须加shebang机制 

- 如果编译安装的程序,需要写程序的完整路径

**宏Macros**

另外 Zabbix 支持用户自定义宏,即支持变量定义 

自定义宏格式为: {$macrosz_name}



### 自定义监控项配置案例

**范例：取根文件系统的空间利用率**

```powershell
[root@ubuntu2404 ~]#cat /etc/zabbix/zabbix_agentd.d/test.conf 
UserParameter=root_filesystem_use,df|awk -F' +|%' '$7 == "/" {print $5 }'

#客户端测试
[root@ubuntu2404 ~]#zabbix_agentd -t root_filesystem_use
root_filesystem_use                           [t|7]

#服务端测试
[root@ubuntu2404 ~]#zabbix_get -s 10.0.0.201 -p 10050 -k "root_filesystem_use"
7
```

**范例：自定义监控项实现连接数**

```powershell
[root@ubuntu2404 ~]#cat /etc/zabbix/zabbix_agentd.d/test.conf
UserParameter=tcp_state_estab,ss -ant|grep -c ESTAB

#客户端测试
[root@ubuntu2404 ~]#zabbix_agentd -t tcp_state_estab
tcp_state_estab                               [t|2]

#服务端测试
[root@ubuntu2404 ~]#zabbix_get -s 10.0.0.201 -p 10050 -k "tcp_state_estab"
3
```

**范例：实现自定义监控项的参数**

```powershell
[root@ubuntu2404 ~]#cat /etc/zabbix/zabbix_agentd.d/test.conf
UserParameter=test[*],echo $1

#服务端测试
[root@ubuntu2404 ~]#zabbix_get -s 10.0.0.201 -p 10050 -k "test[this is a testitem]"
this is a testitem
```

**范例：利用自定义监控项的参数功能监控MySQL的存活状态**

```powershell
[root@ubuntu2404 ~]#apt install mysql-server
[root@ubuntu2404 ~]#systemctl enable --now mysql
[root@ubuntu2404 ~]#mysql
mysql> create user test@'localhost' identified by '123456';
[root@ubuntu2404 ~]#mysqladmin -utest -p123456 ping
mysqld is alive
[root@ubuntu2404 ~]#cat /etc/zabbix/zabbix_agentd.d/test.conf 
UserParameter=mysql.ping[*],mysqladmin -u$1 -p$2 ping 2>/dev/null | grep -c alive

#服务端测试
[root@ubuntu2404 ~]#zabbix_get -s 10.0.0.201 -p 10050 -k "mysql.ping[test,123456]"
1

#客户端关闭mysql
[root@ubuntu2404 ~]#systemctl stop mysql.service 

#服务端测试
[root@ubuntu2404 ~]#zabbix_get -s 10.0.0.201 -p 10050 -k "mysql.ping[test,123456]"
0
```

### 自定义模板

**自定义模板使用流程** 

- 创建模板，模板必须属于某个主机组(一般属于主机组Templates) 

- 在模板中创建监控项、图形、触发器 

- 创建需要监控的主机，然后关联对应的模板 

- 更改模板的监控项目，所以使用模板的都会自动更改 

- 导出模板，后期可以至其他系统继续使用

## 监控项的值映射 Value Mapping

为了接收到的值能更“人性化”的显示，可以通过值映射方式，将数值与字符串之间进行关系绑定 

示例:  

http 响应码 

'200' → 'OK' 

'403' → 'Forbidden' 

'404' → 'Not Found'

### 创建值映射

- 打开对应主机或者指定的模板配置表单

- 前往 值映射 标签 

- 点击 增加 来增加一个新映射 

- 点击一个已存在的值映射名字来进行编辑

**范例: 将net.tcp.listen[80]内置的监控项为1时,映射为up,为0时,映射为down**

![image-20250114194112166](5day-png\26创建映射值1.png)

![image-20250114194235762](5day-png\26创建映射值2.png)

### 使用值映射

zabbix7没有此配置

添加监控项时,在查看值处选中上面创建的值映射名称



## 触发器 Triggers

### 触发器介绍

触发器其实就是一些条件的定义,一个触发器是根据一个监控项的返回值,将之与预先设置的阈值进行对 比，当监控项返回了不符合预定义的值范围后,就进行触发下一步操作的警戒线,一般要对创建的监控项设 置触发器以及触发方式和值的大小

可以在指定主机上创建触发器,只是针对指定主机有效

也可以在指定模板上创建触发器,则使用此模板的所有主机都有效,一个模板中可以有多触发器

触发器中使用的表达式是非常灵活的。可以使用它们去创建关于监控统计的复杂逻辑测试。

### 触发器严重性

触发器严重性表示触发器的重要程度

Zabbix支持下列6种触发器的严重程度：

| 严重性   | 颜色   |
| -------- | ------ |
| 未分类   | 灰色   |
| 信息     | 浅蓝色 |
| 警告     | 黄色   |
| 一般严重 | 橙色   |
| 严重     | 浅红色 |
| 灾难     | 红色   |

**严重性功能**

- 通过不同的颜色区分不同的严重程度

- 报警音频，不同的音频代表不同的严重程度 用户媒介，不同的用户媒介（通知渠道）代表不同的严重程度。例如，短信 - 高严重性，email - 其他。

- 不同的严重性通过触发器执行对应的条件动作

### 触发器表达式格式

**一个简单的表达式格式：**

```powershell
#Zabbix6.0版本后
function(/host/key,parameter)<operator><constant>

#示例
min(/Zabbix server/net.if.in[eth0,bytes],5m)>100K

#Zabbix5.0版本前
{<server|template>:<key>.<function>(<parameter>)}<operator><constant>
```

**触发器表达式常用函数**

| 函数名称 | 说明                       | 范例                                                         |
| -------- | -------------------------- | ------------------------------------------------------------ |
| avg()    | 监控项的平均值             | avg(#5) → 最新5个值的平均值<br />avg(1h) → 最近一小时的平均值 <br />avg(1h,1d) → 一天前的一小时内的平均值 |
| min()    | 监控项的最小值             | CPU使用率最近5分钟的最小值大于6<br /> system.cpu.load.min(5m)>6<br /> CPU最近5次最小的值大于2<br /> system.cpu.load.min(#5)>2 |
| max()    | 监控项的最大值             | max(#5) → 最新5个值的最大值 <br />max(1h) → 最近一小时的最大值 |
| last()   | 最后的第几个值             | 注意last的 #num 参数和在其它函数中的作用不同 <br />例如:返回值 3, 7, 2, 6, 9 <br />last() 通常等同于 last(#1)<br />last(#5) - 第五个最新值 ,注意:不是五个最新值 <br />last(#2)将返回值为7，last(#5)返回值为9 <br />last(#10,30) 表示每隔30s采样一次,取倒数第10个值 |
| diff()   | 比对上一次文件的内容       | 返回值为1,表示发生变化,返回值为0,表示没有变化                |
| nodata() | 监控一段时间内是否返回数据 | 时间不少于30秒，因为timer处理器每30秒调用一次 <br />返回1 - 指定评估期没有接收到数据 <br />返回0 - 其它 |



```powershell
#示例
www.zabbix.com 主机的处理器负载过高
#zabbix6.0
last(/Zabbix server/system.cpu.load[all,avg1])>5

{www.zabbix.com:system.cpu.load[all,avg1].last()}>5
'www.zabbix.com:system.cpu.load[all,avg1]' 给出了被监控参数的简短名称。它指定了服务器是“www.zabbix.com”，监控项的键值是“system.cpu.load[all,avg1]”。通过使用函数“last()”获取最新的值。最后，“>5”意味着当www.zabbix.com最新获取的处理器负载值大于5时触发器就会处于异常状态。

#示例
www.zabbix.com is overloaded
{www.zabbix.com:system.cpu.load[all,avg1].last()}>5 or 
{www.zabbix.com:system.cpu.load[all,avg1].min(10m)}>2 
当前处理器负载大于5或者最近10分钟内最小值大于2，表达式为true。

#示例
/etc/passwd文件被修改

使用函数diff：

{www.zabbix.com:vfs.file.cksum[/etc/passwd].diff()}=1
当文件/etc/passwd的checksum值与最近的值不同时，表达式为true。

类似的，表达式可以用于监控重要文件的修改, 如/etc/passwd, /etc/inetd.conf, /kernel等

#示例
有用户正在从互联网上下载一个大文件

使用min函数:

{www.zabbix.com:net.if.in[eth0,bytes].min(5m)}>100K
在过去5分钟内，eth0上接收字节数大于100kb时，表达式为true。

#示例

SMTP服务群集的两个节点都停止。 注意在一个表达式中使用两个不同的主机:

{smtp1.zabbix.com:net.tcp.service[smtp].last()}=0 and 
{smtp2.zabbix.com:net.tcp.service[smtp].last()}=0
当SMTP服务器smtp1.zabbix.com和smtp2.zabbix.com都停止，表达式为true

#示例
Zabbix agent需要升级

使用str()函数:

{www.zabbix.com:agent.version.str("beta8")}=1
如果Zabbix agent版本是beta8（可能是1.0beta8），则表达式为真。

#示例
服务器无法访问

{www.zabbix.com:icmpping.count(30m,0)}>5
当主机“www.zabbix.com”在30分钟内超过5次不可达，则表达式为真。

#示例
3分钟内没有心跳检查

使用nodata()函数:

{www.zabbix.com:tick.nodata(3m)}=1
要使用这个触发器，'tick'必须定义成一个
Zabbix[:manual/config/items/itemtypes/trapper|trapper]]监控项。主机应该使用
zabbix_sender定期发送这个监控项的数据。

如果在180秒内没有接收到数据，则触发值变为异常状态。

注意:nodata可以在任何类型的监控项中使用。

#示例
夜间的CPU负载

使用time()函数:

{zabbix:system.cpu.load[all,avg1].min(5m)}>2 and 
{zabbix:system.cpu.load[all,avg1].time()}>000000 and 
{zabbix:system.cpu.load[all,avg1].time()}<060000
仅在夜间(00:00-06:00)，触发器状态变可以变为真。

#示例
检查客户端本地时间是否与Zabbix服务器时间同步

使用fuzzytime()函数:

{MySQL_DB:system.localtime.fuzzytime(10)}=0
当MySQL_DB服务器的本地时间与Zabbix server之间的时间相差超过10秒，触发器将变为异常状态。

#示例
比较今天的平均负载和昨天同一时间的平均负载（使用第二个“时间偏移”参数）。

{server:system.cpu.load.avg(1h)}/{server:system.cpu.load.avg(1h,1d)}>2
如果最近一小时平均负载超过昨天相同小时负载的2倍，触发器将触发。

#示例

使用了另一个监控项的值来获得触发器的阈值
{Template PfSense:hrStorageFree[{#SNMPVALUE}].last()}<{Template 
PfSense:hrStorageSize[{#SNMPVALUE}].last()}*0.1
如果剩余存储量下降到10%以下，触发器将触发。

#示例
使用评估结果获取超过阈值的触发器数量:

({server1:system.cpu.load[all,avg1].last()}>5) +
({server2:system.cpu.load[all,avg1].last()}>5) +
({server3:system.cpu.load[all,avg1].last()}>5)>=2
如果表达式中至少有两个触发器大于5，触发器将触发。

```



### 滞后(恢复表达式)

有时我们需要一个OK和问题状态之间的区间值，而不是一个简单的阈值。 

例如，我们希望定义一个触发器，当机房温度超过20C时，触发器会出现异常，我们希望它保持在那种 状态，直到温度下降到15C以下。

为了做到这一点，首先定义问题事件的触发器表达式。然后在事件成功迭代中选择‘恢复表达式’，并为 OK事件输入恢复表达式。 

注意，只有首先解决问题事件才会评估恢复表达式。如果问题条件仍然存在，则不能通过恢复表达式来 

解决问题。 

示例 1 

机房温度过高。 

问题表达式:

```
{server:temp.last()}>20
```

恢复表达式

```
{server:temp.last()}<=15
```

示例 2 

磁盘剩余空间过低。 

问题表达式: it is less than 10GB for last 5 minutes

```
{server:vfs.fs.size[/,free].max(5m)}<10G
```

恢复表达式: it is more than 40GB for last 10 minutes

```
{server:vfs.fs.size[/,free].min(10m)}>40G
```

### 开启 Zabbix Server的声音提示

![image-20250114202447795](5day-png\26开启 Zabbix Server的声音提示.png)





### 自定义触发器

#### 配置单条件触发器

自定义单条件触发器：设置内存低于 30% 进行告警，点击对应主机→ 创建触发器获取内存还剩余的百 分比： 

剩余 30% 可用，则需要告警通知；剩余 50% 可用，就算恢复； 

编辑触发器表达式

```powershell
#问题表达式: 
{Webserverb:mem_use_percent.last()}<30

#恢复表达式: 
{Webserver:mem.use_percent.last()}>60

#测试内存
dd if=/dev/zero of=/dev/null bs=1000M count=1024
```

### 配置多条件触发器

自定义多条件触发器：设置空闲内存低于 30%并且swap使用大于1% 进行告警

```powershell
#增加swap的自定义监控项
UserParameter=swap_use,free |awk '/^Swap/{print $3*100/$2}'

#编辑触发器表达式
#问题表达式：
{Webserver:mem_unuse_percent.last()}<=30 and {Webserver:swap_use.last()}>=1

#恢复表达式：
{Webserver:mem_unuse_percent.last()}>=50

#使用命令压测
dd if=/dev/zero of=/dev/null bs=500M count=1024
#只满足内存低于30%，所以不会警告

dd if=/dev/zero of=/dev/null bs=1000M count=1024
#内存低于30%，并且swap使用超过1%
```

