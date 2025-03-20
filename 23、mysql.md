

# 23、mysql

## 数据库管理系统特点

```powershell
- 数据库管理系统中采用了复杂的数据模型表示数据结构，数据冗余小，易扩充，实现了数据共享。
- 具有较高的数据和程序独立性。
	数据库的独立性表现在物理独立性和逻辑独立性两个方面。
- 数据库管理系统为用户和应用程序提供了方便和统一的查询接口。
- 数据库管理系统的存在，使得数据可以和应用程序解耦。
- 数据库管理系统还具有并发控制，数据备份和恢复，完整性和安全性保障等功能。
```



## CRUD

| 操作描述 | 作用     | SQL关键字 |
| -------- | -------- | --------- |
| create   | 增加数据 | insert    |
| read     | 读取数据 | select    |
| update   | 更新数据 | update    |
| delete   | 删除数据 | delete    |



## 数据库范式

```powershell
第一范式 1NF (确保每列保持原子性)

第二范式 2NF (确保表中的每列都和主键相关)

第三范式 3NF (确保每列都和主键列直接相关，而不是间接相关)

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。
- 必须先满足第一范式才能满足第二范式，
- 必须同时满足第一第二范式才能满足第三范式。
```



## MySQL执行查询语句执行流程

![image-20241227152709081](5day-png\23MySQL数据库中的SQL查询语句执行流程.png)

```powershell
1 接收查询语句：
 	用户通过客户端或编程语言框架里面的驱动模块，经由专门的通信协议，向MySQL服务器发送SQL查询语
句。
 		如果是mysql服务端，这里的通信协议端口是 3306
 	数据库服务端内部的连接器接收查询请求后，首先到本地的缓存记录中检索信息
 		如果有信息，则直接返回查询结果
 		如果没有信息，则将信息直接传递给后端的数据库解析器来进行处理
2 词法分析和语法分析：
 	MySQL服务器首先会对接收到的查询语句进行词法分析，将其分解成一系列的词法单元。
 	接着，进行语法分析，检查这些词法单元是否符合SQL语法规则。如果语法错误，MySQL会立即返回错误信息。
3 语义分析：
    在语法分析之后，MySQL会进行语义分析，检查查询语句中的表、列、函数等是否存在，以及用户是否有足够的权限执行该查询。
    语义分析还会检查查询语句中的数据类型是否匹配，以及是否存在违反数据库完整性的约束。
4 查询优化：
    MySQL的查询优化器会对查询语句进行优化，决定最优的执行查询计划 -- 也就是怎么走会效率更快。
    优化过程包括选择最佳的访问路径、索引、连接顺序等，以最小化查询的执行时间和资源消耗。
    优化器还会考虑统计信息，如表的行数、索引的分布等，以做出更明智的决策。
5 查询执行：
    优化后的查询计划被传递给执行器进行执行。执行器根据查询计划访问存储引擎，获取数据。
   		存储引擎将文件系统里面的数据库文件数据，加载到内存，然后在内存中做一次聚合
   		然后，临时放到查询缓存中，便于下次遇到相同的查询语句重复过来。
    在执行过程中，MySQL可能会使用缓存来加速查询的执行。从MySQL 8.0开始，查询缓存已被移除。
6 结果集返回：
 	执行器将查询结果集返回给客户端。结果集可以包含零行或多行数据，具体取决于查询的内容。
 	如果查询涉及聚合函数（如SUM、AVG等）或分组操作（如GROUP BY），执行器还会在返回结果之前对这些数据进行相应的处理。
7 错误处理和日志记录：
 	如果在执行过程中遇到任何错误（如权限不足、表不存在等），MySQL会返回相应的错误信息给客户端。同时，MySQL还会记录查询的执行日志，以便进行性能分析和故障排查。

```



MySQL 的查询语句执行流程可以分为以下几个主要步骤，从用户发送查询到返回结果，涉及多个组件和阶段。以下是详细执行流程：

------

### 1. **客户端连接**

- **连接建立：** 客户端通过网络与 MySQL 服务器建立连接（通常使用 TCP/IP 或本地套接字）。
- **身份验证：** MySQL 使用用户名、密码、主机等信息对客户端进行身份验证，并检查权限。
- 如果身份验证失败，直接拒绝连接。

------

### 2. **查询接收**

- 客户端发送查询语句（例如 `SELECT * FROM table_name WHERE id = 1`）。
- MySQL 服务器接收查询并存储在查询缓存（如果开启了查询缓存）。

------

### 3. **查询缓存（可选）**

- 缓存检查：

   如果启用了查询缓存，MySQL 会检查是否有完全相同的查询已经执行过。

  - 如果缓存命中，直接返回缓存结果，跳过后续所有步骤。
  - 如果未命中，进入查询解析阶段。

- **注意：** MySQL 8.0 及以上版本已移除查询缓存。

------

### 4. **语法解析与语义分析**

- **词法解析：** 将查询语句拆分成关键字、标识符（如表名、列名）等基础元素。
- **语法解析：** 检查 SQL 语句是否符合语法规则，生成解析树（Parse Tree）。
- **语义分析：** 验证表、列是否存在，以及权限是否满足查询要求。

------

### 5. **查询优化器**

- 逻辑优化：
  - 重写查询语句以生成等价但更高效的执行计划。
  - 例如，将子查询转换为连接（JOIN）、合并 WHERE 条件等。
- 物理优化：
  - 根据表的索引、统计信息、表的大小等，选择最优的执行计划（如使用全表扫描还是索引扫描）。
  - 生成执行计划（Execution Plan）。

执行计划会包含：

- 表的访问顺序
- 使用的索引（如果有）
- 联合查询的连接算法（如 Nested Loop Join）
- 数据排序方式等

------

### 6. **权限检查**

- 检查用户是否对查询涉及的表、列具有相应的权限（如 SELECT 权限）。

------

### 7. **执行器**

- 执行器根据优化器生成的执行计划逐步执行查询：
  1. **存储引擎接口调用：** 执行器调用底层存储引擎（如 InnoDB 或 MyISAM）的 API。
  2. **读取数据：** 存储引擎负责从磁盘或内存中读取数据，返回给执行器。
  3. **过滤与处理：** 执行器对数据进行过滤、排序、聚合等操作，生成最终结果。

------

### 8. **结果返回**

- 查询结果从执行器返回到客户端。
- 数据传输时会分批（chunk）发送，以提高性能。

------

### 9. **查询日志记录**

- **慢查询日志：** 如果查询超过了 `long_query_time` 的阈值，会记录在慢查询日志中。
- **通用查询日志：** 如果启用，会记录所有执行的 SQL 语句。

------

### 图示（简化流程）

```
客户端 -> 连接管理 -> 查询缓存（可选） -> 解析器 -> 优化器 -> 执行器 -> 存储引擎 -> 数据返回
```

------

### 重点优化环节

1. **查询优化器：** 决定了执行计划的优劣，直接影响性能。
2. **存储引擎：** 不同存储引擎（如 InnoDB、MyISAM）的性能和功能差异显著。
3. **索引设计：** 优化查询时至关重要。





## mysql安装

### 二进制安装MySQL

#### 包管理器进行安装

```powershell
mysql
yum -y install mysql-server
apt -y install mysql-server
mariadb
yum -y install mariadb-server
apt -y install mariadb-server

家目录/var/lib/mysql
```



#### 二进制包方式安装Mysql

```powershell
1 安装依赖
Rocky系统：
[root@rocky9 ~]# yum -y install libaio numactl-libs ncurses-compat-libs
Ubuntu系统：
root@ubuntu24:~# apt install libaio-dev numactl libnuma-dev libncurses-dev

注意：ubuntu24系统没有libaio1的包，需要单独去下载安装
curl -O http://launchpadlibrarian.net/646633572/libaio1_0.3.113-4_amd64.deb
dpkg -i libaio1_0.3.113-4_amd64.deb

2 创建组和用户
groupadd -r mysql
useradd -r -g mysql -s /sbin/nologin mysql

3 获取软件
mkdir /data/softs -p;cd /data/softs
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.4.0-linux-glibc2.28-x86_64.tar.xz

4 解压至指定目录（这个目录只能写 /usr/local/,不用增加修改环境变量）
tar xf mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz
mv mysql-8.4.0-linux-glibc2.28-x86_64 /usr/local/mysql

5 创建环境变量
echo 'PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
source /etc/profile.d/mysql.sh
echo $PATH
更改文件属性
chown -R mysql:mysql /usr/local/mysql/
6 创建数据目录
mkdir /usr/local/mysql/data -p
mkdir /usr/local/mysql/etc -p
mkdir /usr/local/mysql/log -p

7 创建主配置文件
root@ubuntu-42:softs # vim /usr/local/mysql/etc/my.cnf
root@ubuntu-42:softs # cat /usr/local/mysql/etc/my.cnf
[mysql]
port = 3306
socket = /usr/local/mysql/data/mysql.sock
[mysqld]
port = 3306
mysqlx_port = 33060
mysqlx_socket = /usr/local/mysql/data/mysqlx.sock
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
socket = /usr/local/mysql/data/mysql.sock
pid-file = /usr/local/mysql/data/mysqld.pid
log-error = /usr/local/mysql/log/error.log

# 注意：大家在互联网上经常看到的认证属性已经被移除了
# default-authentication-plugin = mysql_native_password

8.1 初始化,本地root用户 - 使用密码
使用 --initialize 选项会生成随机密码，要去 /usr/local/mysql/mysql.log中查看
cd /var/local/mysql
bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
tail -f /usr/local/mysql/log/error.log
2024-12-27T09:07:02.861894Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: -9hsUqId_eLt
密码：-9hsUqId_eLt

8.2 初始化,本地root用户 - 使用空密码
使用 --initialize-insecure -选项会生成空密码

执行完8.1和9后要先关闭mysql服务后再清理历史文件再执行8.2
systemctl stop mysqldpkill mysql （忘记关闭删除完文件后可执行这条命令关闭8.1的mysql服务）
rm -rf /usr/local/mysql/data/*
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

9 定制启动脚本
该脚本不是systemd风格的脚本，但是可以被 systemd兼容
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
重载配置文件
systemctl daemon-reload
启动mysqld服务
/etc/init.d/mysqld start
会自动生成服务管理文件
查看自动生成的服务管理文件
systemctl cat mysqld.service
systemctl status mysqld
```

#### 通过脚本安装MySQL

```powershell

```







## mariadb多实例

```powershell
1 安装软件
yum install -y mariadb-server mariadb-pam

2 检测软件环境
检测mysql用户
id mysql
uid=27(mysql) gid=27(mysql) groups=27(mysql)
如果没有该用户，可以手工创建、
groupadd -g 27 mysql
useradd -s /usr/sbin/nologin -u 27 -g 27 mysql
查看默认的配置文件
grep -Ev '#|^$' /etc/my.cnf.d/mariadb-server.cnf
检测服务是否关闭
systemctl status mariadb.service

3 创建目录
mkdir -pv /mysql/{3306,3307,3308}/{data,etc,socket,log,bin,pid}
为目录赋予权限
chown -R mysql:mysql /mysql/

4 生成三个实例的初始数据
mysql_install_db --user=mysql --datadir=/mysql/3306/data
mysql_install_db --user=mysql --datadir=/mysql/3307/data
mysql_install_db --user=mysql --datadir=/mysql/3308/data

5 创建配置文件
cat > /mysql/3306/etc/my.cnf <<-eof
[mysqld]
port=3306
datadir=/mysql/3306/data
socket=/mysql/3306/socket/mysql.sock
log-error=/mysql/3306/log/mysql.log
pid-file=/mysql/3306/pid/mysql.pid
eof
给每个实例配置文件
for i in 7 8
do
  cp -a /mysql/3306/etc/my.cnf /mysql/330$i/etc/my.cnf
  sed -i "s#3306#330$i#g" /mysql/330$i/etc/my.cnf
done

6 创建mysql的启动脚本
vim /mysql/3306/bin/mysqld
#!/bin/bash
PORT=3306
USER="root"
PWD="Magedu"
CMD_PATH="/usr/bin"
BASE_DIR="/mysql"
SOCKET="${BASE_DIR}/${PORT}/socket/mysql.sock"
mysql_start(){
  if [ ! -e "$SOCKET" ];then
    echo "Starting MySQL..."
    ${CMD_PATH}/mysqld_safe --defaults-file=${BASE_DIR}/${PORT}/etc/my.cnf 
&>/dev/null &
  else
    echo "MySQL is running..."
    exit
  fi
}
mysql_stop(){
  if [ ! -e "$SOCKET" ];then
    echo "MySQL is stoped..."
    exit
  else
    echo "Stop MySQL..."
    ${CMD_PATH}/mysqladmin -u ${USER} -p${PWD} -S ${SOCKET} shutdown
  fi
}
mysql_restart(){
  echo "Starting MySQL..."
  mysql_stop
  sleep 2
  mysql_start
}
usage_msg(){
  echo "Usage: ${BASE_DIR}/${PORT}/bin/mysqld {start|stop|restart}"
}
case $1 in
  start)
    mysql_start;;
  stop)
    mysql_stop;;
  restart)
    mysql_restart;;
  *)
    usage_msg;;
esac

给每个实例配置启动文件
for i in 7 8
do
  cp -a /mysql/3306/bin/mysqld /mysql/330$i/bin/mysqld
  sed -i "s#3306#330$i#g" /mysql/330$i/bin/mysqld
done
加执行权限
chmod +x /mysql/3306/bin/mysqld
chmod +x /mysql/3307/bin/mysqld
chmod +x /mysql/3308/bin/mysqld

7 启动mysql服务 
/mysql/3306/bin/mysqld
Usage: /mysql/3306/bin/mysqld {start|stop|restart}

8 配置启动别名
alias mysqld3306='/mysql/3306/bin/mysqld'
alias mysqld3307='/mysql/3307/bin/mysqld'
alias mysqld3308='/mysql/3308/bin/mysqld'

9 登录数据库
mysql -S /mysql/3306/socket/mysql.sock
配置登录数据库别名
alias mysql3306='mysql -S /mysql/3306/socket/mysql.sock'
alias mysql3307='mysql -S /mysql/3307/socket/mysql.sock'
alias mysql3308='mysql -S /mysql/3308/socket/mysql.sock'

10 测试
root@rocky9-14:~ # netstat -tnulp | grep 330
tcp6       0      0 :::3306                 :::*                    LISTEN      32631/mariadbd
tcp6       0      0 :::3307                 :::*                    LISTEN      32775/mariadbd
tcp6       0      0 :::3308                 :::*                    LISTEN      32890/mariadbd
```



## 关系数据库的构成

| 组件       | 关键字           | 说明                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| 数据库     | databse          | 表的集合，一个数据库中可以有多个表，在文件系统中表现出就是一 个目录 |
| 表         | table            | 在数据库中以二维表的形式出现，有行和列，数据库中的数据就是存 放于表中的 |
| 索引       | index            | 索引通常建立在一个列上，用以加快数据查询速度                 |
| 视图       | view             | 用SQL语言构建的虚拟表，可以临时把两个或多个表以逻辑关系关联 上，对外提供查询 |
| 存储过程   | procedure        | 存储过程是一组为了完成特定功能的SQL语句集合，客户端可以直接 调用 |
| 存储函数   | function         | 存储函数和存储过程一样，都是SQL语句集合，但可以使用参数      |
| 触发器     | trigger          | 触发器也是有SQL语句的集合组成，但需要达到触发条件才能调用    |
| 事件调度器 | envent scheduler | 数据库中的计划任务                                           |
| 用户       | user             | 连接服务端时的用户名                                         |
| 权限       | privilege        | 每个用户可以对哪些数据库或表进行操作，在什么IP能连接         |

### @符号

```powershell
- 当你执行 SELECT @@hostname; 时，你实际上是在查询名为 hostname 的系统变量的值。这个变量通常包含了数据库服务器所在计算机的主机名。
 - @ 符号单独使用时，通常用于引用用户定义的会话变量（session variables），这些变量是在当前用户会话的上下文中定义的。
 - @@ 符号则用于引用全局系统变量或会话系统变量，具体取决于上下文。如果没有明确指定会话级别（使用 SET SESSION），则默认引用全局系统变量。
```

### 注释

```powershell
单行注释
-- select version()\G;
# select version()\G;
多行注释
/*
select version()\G;
select version()\G;
select version()\G;
select version()\G;
*/
```



## sql语句

| SQL语句类型 | 说明                                      | 具体语句                           |
| ----------- | ----------------------------------------- | ---------------------------------- |
| DDL         | Data Defination Language 数据定义语言     | CREATE, DROP, ALTER                |
| DML         | Data Manipulation Language 数据操纵语言   | INSERT, DELETE, UPDATE             |
| DQL         | Data Query Language 数据查询语言          | SELECT                             |
| DCL         | Data Control Language 数据控制语言        | GRANT, REVOKE                      |
| TCL         | Transaction Control Language 事务控制语言 | BEGIN, COMMIT, ROLLBACK, SAVEPOINT |



## DDL

### 数据库操作

```sql
查看当前数据库
	SHOW DATABASES;
创建数据库
	CREATE DATABASE 数据库名;
切换到数据库
	USE 数据库名;
查看当前在那个数据库里面
	SELECTDATABSE();
删除数据库
	DROP DATABASE 数据库名;
```

### 数据表操作

```powershell
查看当前数据库下的所有表
	SHOW TABLES;
创建表
	直接创建
		CREATE TABLE 表名 (字段 字段类型,字段 字段类型);
	查询创建
		create table 表名 select Host,User,Password from 要查询的数据库名.要查询的表名;
	复制指定表的表结构创建表-仅结构
		create table 表名 like 要复制的表名;	
查询表里面的数据
	select * from 表名;
	select 字段1,字段2 from 表名;
查看当前表里面的字段
	DESC 表名;
	SHOW columns from 表名;
查询表的建表语句
	SHOW CREATE TABLE 表名;
	SHOW CREATE TABLE 表名\G
	SHOW CREATE TABLE 数据库名.表名;
	SHOW CREATE TABLE 数据库名.表名\G
修改表 添加字段/修改字段类型/修改字段名称及类型/删除字段/修改表名
	ALTER TABLE 表名 ADD/MODIFY/CHANGE/DROP/RENAME TO ...;
修改表名
	alter table 要修改的表名 rename to 修改后的表名;
删除表名
	DROP TABLE 表名;
添加表字段
	向表 stu 中插入一个新的字段 phone，排在name字段后面
	alter table stu add phone 字段类型 after name;
删除字段
	alter table 表名 drop column 字段;
字段类型修改
	alter table 表名 modify 字段 类型;
修改字段名称和类型
	alter table 表名 change column 要修改的字段 修改后的字段 类型;
修改表字符集
	alter table 表名 character set utf8;
	在修改字符集的时候，同时修改字段的类型
		alter table 表名 change 要修改的字段 修改后的字段 类型 character set utf8;
字段默认值修改
	alter table 表名 alter column 字段 set default '值';
添加字段的时候，设定默认值
	alter table 表名 add 字段 bool default 0;
	alter table 表名 add 字段 bool default false;
修改字段的默认值
	alter table 表名 modify 字段 bool default true;
	alter table 表名 modify 字段 bool default 1;
为当前表添加主键
	为一个表的现有字段 增加|删除一个主键
 		ALTER TABLE 表名 ADD|DROP PRIMARY KEY (字段);
	为一个表的 增加一个 之前不存在的字段为主键
		ALTER TABLE 表名 ADD COLUMN 字段 INT UNSIGNED FIRST;
	将一个表的现有主键进行自增属性的增加
 		ALTER TABLE 表名 MODIFY COLUMN 字段 INT AUTO_INCREMENT PRIMARY KEY;
 	或，将上面命令拆解成两个
 		ALTER TABLE 表名 MODIFY COLUMN 字段 INT UNSIGNED AUTO_INCREMENT;
 		ALTER TABLE 表名 ADD PRIMARY KEY (字段);
在数据库外面查看所有表
	show tables from 数据库名;
	show tables in 数据库名;
	等价于
	use 数据库名;
	show tables;
查看表状态信息
	SHOW TABLE STATUS LIKE '表名';
	SHOW TABLE STATUS LIKE '表名'\G
查看库里面所有表的状态
	SHOW TABLE STATUS FROM 数据库名;
	SHOW TABLE STATUS FROM 数据库名\G
查看表中的自增id
	select `auto_increment` from information_schema.tables where table_schema = '数据库名' and table_name = '表名';
定制自增id起始值
	创建主键id
		alter table 表名 add column 字段 int unsigned first;
	将设定的主键自增
		alter table 表名 modify column 字段 int unsigned auto_increment primary key;
	将主键id的自增起始值设置为 20
		alter table 表名 auto_increment = 20;
```



## DML

### 添加数据

```sql
INSERT INTO 表名 (字段1,字段2,...)VALUES(值1,值2,...)[,(值1,值2,...)...];
```

### 修改数据

```sql
UPDATE 表名 SET 字段1=值1,字段2=值2[WHERE 条件];
单条匹配
把 id 字段大于15的修改为‘字段=值’
update 表名 set 字段=值 where id>15;
多条匹配
把 id 字段等于15或 name 字段为空的修改为‘字段=值’
update 表名 set 字段=值 where (id=15 || name is null);
把 id 字段等于17和 name 字段为xiaohong的修改为‘字段=值’
update 表名 set 字段=值 where (id=17 and name='xiaohong');
```

### 删除数据

```sql
DELETE FROM 表名 [WHERE 条件];
```

```powershell
TRUNCATE TABLE tbl_name; 		#DDL语句，不支持事务，效率更高
DELETE FROM tbl_name; 			#DML语句
```

## DCL

### 用户管理

```sql
创建用户
CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
修改用户密码
ALTER USER '用户名'@'主机名' IDENTIFIED WITH mysql_natile_password BY '密码';
ALTER USER '用户名'@'主机名' IDENTIFIED BY '密码';

删除用户
DROP USER '用户名'@'主机名'
```

```powershell
创建用户：CREATE USER
格式
 	CREATE USER 'USERNAME'@'HOST' [IDENTIFIED BY 'password'];
范例
    create user test@'10.0.0.0/255.255.255.0' identified by '123456';
    create user test2@'10.0.0.%' identified by '123456';
 
重命名用户：RENAME USER
格式
 	RENAME USER 'USERNAME'@'HOST' TO 'USERNAME'@'HOST';

删除用户：DROP USER
格式
 	DROP USER 'USERNAME'@'HOST'
范例，删除默认的空用户
 	DROP USER ''@'localhost';
```



### 权限控制

```sql
授权
GRANT 权限列表 ON 数据库名.表名 TO '用户名'@'主机名';
撤销权限
REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@'主机名';
```



## 数据类型

MySQL 支持多种数据类型，可以根据需要选择合适的类型来存储数据。这些数据类型大致分为以下几类：

### 1. **数值类型**

#### 1.1 整数类型

- **TINYINT**: 1字节，范围 -128 到 127（有符号）；0 到 255（无符号）。
- **SMALLINT**: 2字节，范围 -32,768 到 32,767（有符号）；0 到 65,535（无符号）。
- **MEDIUMINT**: 3字节，范围 -8,388,608 到 8,388,607（有符号）；0 到 16,777,215（无符号）。
- **INT/INTEGER**: 4字节，范围 -2,147,483,648 到 2,147,483,647（有符号）；0 到 4,294,967,295（无符号）。
- **BIGINT**: 8字节，范围 -2^63 到 2^63-1（有符号）；0 到 2^64-1（无符号）。

#### 1.2 浮点类型

- **FLOAT(M, D)**: 4字节，存储近似值，`M` 是总位数，`D` 是小数位数。
- **DOUBLE/REAL (M, D)**: 8字节，存储近似值，`M` 是总位数，`D` 是小数位数。
- **DECIMAL/NUMERIC (M, D)**: 精确浮点数，用字符串方式存储。`M` 是总位数，`D` 是小数位数。

------

### 2. **日期和时间类型**

- **DATE**: 日期，格式 `YYYY-MM-DD`，范围 `1000-01-01` 到 `9999-12-31`。
- **DATETIME**: 日期和时间，格式 `YYYY-MM-DD HH:MM:SS`，范围同 DATE。
- **TIMESTAMP**: UNIX 时间戳，格式 `YYYY-MM-DD HH:MM:SS`，范围 `1970-01-01 00:00:01` UTC 到 `2038-01-19 03:14:07` UTC。
- **TIME**: 时间，格式 `HH:MM:SS`，范围 `-838:59:59` 到 `838:59:59`。
- **YEAR**: 年份，格式 `YYYY`，范围 `1901` 到 `2155`。

------

### 3. **字符串类型**

#### 3.1 固定长度字符串

- **CHAR(M)**: 固定长度，最多 255 个字符。

#### 3.2 可变长度字符串

- **VARCHAR(M)**: 可变长度，最多 65535 个字符（取决于行大小和字符集）。

#### 3.3 文本类型

- **TINYTEXT**: 最多 255 个字符。
- **TEXT**: 最多 65,535 个字符。
- **MEDIUMTEXT**: 最多 16,777,215 个字符。
- **LONGTEXT**: 最多 4,294,967,295 个字符。

#### 3.4 二进制类型

- **TINYBLOB**: 最多 255 字节。
- **BLOB**: 最多 65,535 字节。
- **MEDIUMBLOB**: 最多 16,777,215 字节。
- **LONGBLOB**: 最多 4,294,967,295 字节。

#### 3.5 枚举与集合

- **ENUM**: 枚举类型，可以定义一组可能的值（最多 65,535 个）。
- **SET**: 集合类型，可以存储一个或多个预定义的值（最多 64 个）。

------

### 4. **空间数据类型 (GIS)**

MySQL 支持地理空间数据类型，例如：

- **GEOMETRY**: 任意几何值。
- **POINT**: 单个点。
- **LINESTRING**: 一系列点组成的线。
- **POLYGON**: 一系列点组成的多边形。

------

### 5. **JSON 数据类型**

- **JSON**: 存储 JSON 格式数据，从 MySQL 5.7.8 开始支持。





## 修饰符

在 MySQL 中，数据类型可以通过 **修饰符（Modifiers）** 进一步定义或调整，修饰符主要用于控制存储方式、精度、默认值等特性。以下是 MySQL 常用的修饰符分类与作用：

------

### 1. **NULL 和 NOT NULL**

- **NULL**: 表示字段可以存储 `NULL` 值（默认行为）。

- NOT NULL: 表示字段不能为空。

  ```sql
  CREATE TABLE example (
      column1 INT NOT NULL,
      column2 VARCHAR(100) NULL
  );
  ```

------

### 2. **DEFAULT 默认值**

- 为字段设置一个默认值，当插入数据时如果没有指定该字段的值，则使用默认值。

- 支持常量或函数（如 CURRENT_TIMESTAMP）。

  ```sql
  CREATE TABLE example (
      column1 INT DEFAULT 0,
      column2 TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  ```

------

### 3. **AUTO_INCREMENT 自增**

- 适用于整数类型，值会自动递增，常用于主键。

- 一个表只能有一个 AUTO_INCREMENT 列，且必须是索引或主键。

  ```sql
  CREATE TABLE example (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(100)
  );
  ```

------

### 4. **UNSIGNED 和 SIGNED**

- **UNSIGNED**: 表示无符号数，数值范围从 0 开始（不支持负数）。

- SIGNED: 默认行为，表示带符号数，可以存储正负值。

  ```sql
  CREATE TABLE example (
      column1 INT UNSIGNED,
      column2 SMALLINT SIGNED
  );
  ```

------

### 5. **ZEROFILL**

- 使用零填充来补全字段显示的宽度，常与 `UNSIGNED` 一起使用。

- ZEROFILL 会强制字段为 UNSIGNED。

  ```sql
  CREATE TABLE example (
      column1 INT(5) ZEROFILL
  );
  ```

  插入值 42，查询时显示 00042。

------

### 6. **CHARACTER SET 和 COLLATE**

- 用于指定字段的字符集和排序规则。

- **CHARACTER SET**: 定义字段的字符集，例如 `utf8` 或 `utf8mb4`。

- COLLATE: 定义字段的排序规则，例如 utf8_general_ci。

  ```sql
  CREATE TABLE example (
      column1 VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
  );
  ```

------

### 7. **ON UPDATE**

- 主要用于 TIMESTAMP或 DATETIME类型，定义字段在记录更新时自动修改的行为。

  ```sql
  CREATE TABLE example (
      last_modified TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  );
  ```

------

### 8. **宽度 (Display Width)**

> **注意：从 MySQL 8.0 开始，显示宽度对整数类型已被废弃！**

- 早期版本中，整数类型（如 INT、TINYINT）可以指定显示宽度（如 `INT(5)`），但不影响存储，只影响 `ZEROFILL` 时的显示格式。

------

### 9. **BINARY 修饰符**

- 强制字符串类型为二进制存储，区分大小写。

  ```sql
  CREATE TABLE example (
      column1 CHAR(10) BINARY
  );
  ```

------

### 10. **ENUM 和 SET 的修饰符**

- **ENUM**: 定义一个单选集合。

- SET: 定义一个多选集合。

  ```sql
  CREATE TABLE example (
      column1 ENUM('small', 'medium', 'large') NOT NULL,
      column2 SET('a', 'b', 'c', 'd') DEFAULT 'a,b'
  );
  ```



### 适用所有类型的修饰符

| 修饰符             | 说明                                               |
| ------------------ | -------------------------------------------------- |
| NULL               | 数据列可包含NULL值，默认值                         |
| NOT NULL           | 数据列不允许包含NULL值                             |
| DEFAULT            | 指定默认值                                         |
| PRIMARY KEY        | 主键，所有记录中此字段的值不能重复，且不能为NULL   |
| UNIQUE KEY         | 唯一键，所有记录中此字段的值不能重复，但可以为NULL |
| CHARACTER SET name | 指定一个字符集                                     |

### 适用数值型的修饰符

| 修饰符         | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| AUTO_INCREMENT | 自动递增，适用于整数类型, 必须作用于某个 key 的字段，比如primary key |
| UNSIGNED       | 无符号                                                       |





## 查询表所支持的引擎

```sqlite
show engines;
show engines\G
```

```powershell
Aria:
	Aria是MyISAM的改进版，提供了崩溃安全的表，并继承了MyISAM的特性。
 	它主要用于内部临时表和权限表。不支持事务。
MRG_MyISAM:
 	MRG_MyISAM是MyISAM表的集合，允许将多个MyISAM表合并为一个虚拟表。不支持事务。
MEMORY:
 	MEMORY存储引擎基于哈希表，数据存储在内存中，非常适合用于临时表。不支持事务。
BLACKHOLE:
 	BLACKHOLE引擎就像一个黑洞，任何写入它的数据都会消失，类似于Unix/Linux中的/dev/null。不支持事务。
MyISAM:
 	MyISAM是非事务性存储引擎，具有良好的性能和较小的数据占用空间。不支持事务。
CSV:
 	CSV存储引擎将表存储为CSV文件，非常适合与其他系统进行数据交换。不支持事务。
ARCHIVE:
 	ARCHIVE引擎通过gzip压缩表来减少存储占用空间，非常适合存储历史数据。不支持事务。
FEDERATED:
 	FEDERATED引擎允许访问其他MariaDB服务器上的表，支持事务和其他高级功能。
PERFORMANCE_SCHEMA:
 	PERFORMANCE_SCHEMA存储引擎用于性能监控和分析。不支持事务。
InnoDB:
 	InnoDB是MariaDB的默认存储引擎，支持事务、行级锁定、外键和表加密。是大多数应用场景的首选。
SEQUENCE:
 	SEQUENCE引擎生成填充有顺序值的表，非常适合需要生成唯一序列号的场景。支持事务。
```



## DQL

```powershell
字段显示可以使用别名：col1 AS alias1, col2 AS alias2，......
WHERE子句：指明过滤条件以实现"选择"的功能
    过滤条件：布尔型表达式
    算术操作符：+，-，*，/， %
    比较操作符：=，<=>（相等或都为空）,<>，!=(非标准SQL)，>，>=，<，<=
    逻辑操作符：NOT，AND，OR，XOR
    
范例查询：BETWEEN min_num AND max_num
不连续的查询：IN (element1，element2，...)  
空查询：IS NULL,，IS NOT NULL

DISTINCT 去除重复行，范例：SELECT DISTINCT gender FROM students;

模糊查询：LIKE 使用 % 表示任意长度的任意字符 _ 表示任意单个字符
RLIKE：正则表达式，索引失效，不建议使用
REGEXP：匹配字符串可用正则表达式书写模式，同上

GROUP BY：根据指定的条件把查询结果进行"分组"以用于做"聚合"运算
常见聚合函数： count()，sum()，max()，min()，avg()，注意：聚合函数不对null统计

HAVING：对分组聚合运算后的结果指定过滤条件
一旦分组 group by，select语句后只跟分组的字段，聚合函数

ORDER BY：根据指定的字段对查询结果进行排序
升序：ASC
降序：DESC

LIMIT [[offset,]row_count]：对查询的结果进行输出行数数量限制，跳过 offset，显示 row_count 行，offset 默为值为 0。
 
FOR UPDATE：写锁，独占或排它锁，只有一个读和写操作
LOCK IN SHARE MODE：读锁，共享锁，同时多个读操作
```

`count()`, `sum()`, `max()`, `min()`, 和 `avg()` 是常用的 **聚合函数**（aggregate functions），通常用于 SQL 查询或数据处理工具（如 Python 中的 Pandas）。这些函数用于对一组数据进行计算并返回单一的结果值。以下是这些函数的详细说明：

### 聚合函数

#### 1. **`count()`**

`count()` 函数用于返回给定列中非 `NULL` 的值的数量。它通常用于统计行数或特定值的出现次数。

- **语法：**

  ```
  SELECT COUNT(column_name) FROM table_name;
  ```

- **例子：**

  ```sql
  SELECT COUNT(*) FROM employees;  -- 统计 employees 表中所有行的数量
  SELECT COUNT(salary) FROM employees;  -- 统计 salary 列中非 NULL 值的数量
  ```

#### 2. **`sum()`**

`sum()` 函数用于计算给定列中所有数字的总和。该列通常应为数值类型（如整数、浮动点数等）。

- **语法：**

  ```
  SELECT SUM(column_name) FROM table_name;
  ```

- **例子：**

  ```
  SELECT SUM(salary) FROM employees;  -- 计算 employees 表中所有 salary 的总和
  ```

#### 3. **`max()`**

`max()` 函数返回给定列中的最大值。它通常用于找出最大数值或字母顺序的最大值。

- **语法：**

  ```
  SELECT MAX(column_name) FROM table_name;
  ```

- **例子：**

  ```
  SELECT MAX(salary) FROM employees;  -- 查找 employees 表中最高的 salary
  ```

#### 4. **`min()`**

`min()` 函数返回给定列中的最小值。它通常用于找出最小数值或字母顺序的最小值。

- **语法：**

  ```
  SELECT MIN(column_name) FROM table_name;
  ```

- **例子：**

  ```
  SELECT MIN(salary) FROM employees;  -- 查找 employees 表中最低的 salary
  ```

#### 5. **`avg()`**

`avg()` 函数返回给定列的平均值。它用于计算一组数值的算术平均。

- **语法：**

  ```
  SELECT AVG(column_name) FROM table_name;
  ```

- **例子：**

  ```
  SELECT AVG(salary) FROM employees;  -- 计算 employees 表中所有 salary 的平均值
  ```



## 密码

### mysql/mariadb

这些参数是 MariaDB 的 `validate_password` 插件的配置项，用于强制执行密码的复杂度要求。具体来说，这些配置项定义了密码的复杂性规则，例如密码的最小长度、必须包含的字符种类等。下面是每个配置项的详细解释：

#### 1. **`validate_password.changed_characters_percentage`**

- **值**：`0`
- **解释**：这个选项指定密码更改时，必须修改多少百分比的字符。如果为 `0`，则意味着没有强制要求修改一定比例的字符。密码可以完全相同地更改，也不受限制。

#### 2. **`validate_password.check_user_name`**

- **值**：`ON`
- **解释**：此选项决定是否禁止将用户名作为密码的一部分。如果为 `ON`，则密码不能包含与用户名相同的部分。这是为了防止密码过于简单或容易猜测。

#### 3. **`validate_password.dictionary_file`**

- **值**：空
- **解释**：该选项指定一个文件路径，文件中包含了一些常见的单词或密码，用来检查密码是否过于简单，容易被猜测。由于此项为空，说明没有配置字典文件，也就是说当前并未启用基于字典的密码强度检查。

#### 4. **`validate_password.length`**

- **值**：`8`
- **解释**：此选项指定密码的最小长度。当前设置为 8，表示密码必须至少有 8 个字符。

#### 5. **`validate_password.mixed_case_count`**

- **值**：`1`
- **解释**：此选项指定密码中必须至少包含多少个大写字母和小写字母的混合。当前设置为 `1`，表示密码必须至少有一个大写字母和一个小写字母。

#### 6. **`validate_password.number_count`**

- **值**：`1`
- **解释**：此选项指定密码中必须至少包含多少个数字字符。当前设置为 `1`，表示密码必须至少包含一个数字字符。

#### 7. **`validate_password.policy`**

- **值**：`MEDIUM`

- 解释

  ：此选项定义了密码的复杂度政策。MariaDB 支持三种复杂度等级：

  - **`LOW`**：密码可以包含少量字符，没有强制要求混合字符种类。
  - **`MEDIUM`**：密码必须包含大写字母、小写字母、数字和特殊字符等基本复杂性要求（即当前配置）。
  - **`STRONG`**：密码必须满足更严格的要求，除了包含大写字母、小写字母、数字和特殊字符外，还要求密码的长度更长，可能还需要不包含字典词汇等。

#### 8. **`validate_password.special_char_count`**

- **值**：`1`
- **解释**：此选项指定密码中必须至少包含多少个特殊字符。当前设置为 `1`，表示密码必须至少包含一个特殊字符（如 `!`, `@`, `#`, `$`, `%` 等）。

```powershell
mysql -e "SHOW VARIABLES LIKE 'validate_password%';"
mysql> SHOW VARIABLES LIKE 'validate_password%';
+-------------------------------------------------+-------+
| Variable_name                                   | Value |
+-------------------------------------------------+-------+
| validate_password.changed_characters_percentage | 0     |
| validate_password.check_user_name               | ON    |
| validate_password.dictionary_file               |       |
| validate_password.length                        | 8     |
| validate_password.mixed_case_count              | 1     |
| validate_password.number_count                  | 1     |
| validate_password.policy                        | LOW   |
| validate_password.special_char_count            | 1     |
+-------------------------------------------------+-------+
8 rows in set (0.00 sec)

mysql修改方式为弱密码
1 在 [mysqld] 部分添加或修改 validate_password.policy 配置项：
[mysqld]
validate_password.policy = LOW

2 进入mysql修改 
SET GLOBAL validate_password.policy = LOW;

mysql初始化root密码
mysql_secure_installation

mariadb初始化root密码（多实例）
mysql_secure_installation -S /mysql/3306/socket/mysql.sock
Switch to unix_socket authentication [Y/n] y # 是否切换成socket方式认证
Change the root password? [Y/n] y # 是否修改root密码
New password: # 输入新密码
Re-enter new password: # 输入确认新密码
Remove anonymous users? [Y/n] y # 是否移除匿名用户
Disallow root login remotely? [Y/n] y # 是否进制远程登录
Remove test database and access to it? [Y/n] y # 是否移除测试数据库
Reload privilege tables now? [Y/n] y # 是否重载权限表


```



### mysqladmin更改密码

```powershell
mysqladmin -uroot -p旧密码 password '新密码'
```



### 忘记密码

删库跑路式重置密码（数据不保留初始化数据库）

```powershell
systemctl stop mysql.service
ls /var/lib/mysql
rm -rf /var/lib/mysql/*
mysqld --initialize-insecure
ls /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql
chmod -R 750 /var/lib/mysql
systemctl start mysql.service
```



保留原有数据，清理密码思路

```powershell
1 启动时添加指定项
[mysqld]
skip-grant-tables 	# 启动MySQL服务器时跳过权限表的加载。任何用户都可以连接到
 						# MySQL服务器而无需提供密码，并且拥有对所有数据库的完全访问权限。
skip-networking 		# 启动MySQL服务器时禁用网络连接。skip-networking被启用时，
 						# MySQL服务器不会监听任何TCP/IP端口，只能通过套接字文件进行本地连接。
2 使用UPDATE命令修改管理员密码
3 移除配置项重启
```

mysql

```powershell
1 启动时添加指定项 重启服务
vim /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
skip-grant-tables
skip-networking
重启服务
systemctl restart mysql.service

2 登录数据库 修改密码
登录数据库
mysql
修改密码
update mysql.user set authentication_string='' where user='root' and host='localhost';
刷新权限
flush privileges;

3 移除配置项 重启服务
vim /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
# skip-grant-tables
# skip-networking
重启服务
systemctl restart mysql.service
```

mariadb

```powershell
1 启动时添加指定项 重启服务
vim /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
skip-grant-tables # 增加
skip-networking # 增加，禁止远程连接，仅允许本地连接
重启服务
systemctl restart mariadb.service

2 登录数据库 修改密码
登录数据库
mysql
修改密码
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('12345678');
或者
alter user root@'localhost' identified by '12345678';
刷新权限
flush privileges;

3 移除配置项 重启服务
vim /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
# skip-grant-tables
# skip-networking
重启服务
systemctl restart mariadb.service
```



## 权限管理

### 1. **基本语法**

#### 1.1 **GRANT 权限**

基本的 `GRANT` 命令格式如下：

```
GRANT <权限> ON <数据库>.<对象> TO '<用户名>'@'<主机>';
```

- **权限**：指定要授予的权限，如 `SELECT`、`INSERT`、`UPDATE`、`ALL PRIVILEGES` 等。
- **数据库**：指定数据库名。如果要为所有数据库授权，可以使用 `*`。
- **对象**：可以是数据库、表、列等。如果是数据库，则为 `<数据库>.*`；如果是表，则为 `<数据库>.<表>`；如果是列，则为 `<数据库>.<表>.<列>`。
- **用户名**：指定用户的名称。
- **主机**：指定用户可以从哪个主机登录，通常为 `localhost` 或 `%`（表示所有主机）。

### 2. **常见权限**

- `ALL PRIVILEGES`：授予所有权限。
- `SELECT`：授予查询权限。
- `INSERT`：授予插入权限。
- `UPDATE`：授予更新权限。
- `DELETE`：授予删除权限。
- `CREATE`：授予创建数据库、表等的权限。
- `DROP`：授予删除数据库、表等的权限。
- `GRANT OPTION`：授予该用户授予其他用户权限的能力。

### 3. **授予权限的示例**

#### 3.1 **授予所有权限**

为 `user1` 用户授予所有权限，允许其访问 `db1` 数据库中的所有表。

```
GRANT ALL PRIVILEGES ON db1.* TO 'user1'@'localhost';
```

#### 3.2 **授予单一权限**

为 `user1` 用户授予对 `db1` 数据库中所有表的 `SELECT` 权限：

```
GRANT SELECT ON db1.* TO 'user1'@'localhost';
```

#### 3.3 **授予对特定表的权限**

为 `user1` 用户授予对 `db1` 数据库中 `table1` 表的 `INSERT` 权限：

```
GRANT INSERT ON db1.table1 TO 'user1'@'localhost';
```

#### 3.4 **授予对特定列的权限**

为 `user1` 用户授予对 `db1.table1` 表中的 `column1` 和 `column2` 列的 `SELECT` 权限：

```
GRANT SELECT (column1, column2) ON db1.table1 TO 'user1'@'localhost';
```

#### 3.5 **授予多个权限**

为 `user1` 用户授予对 `db1` 数据库中所有表的 `SELECT`、`INSERT` 和 `UPDATE` 权限：

```
GRANT SELECT, INSERT, UPDATE ON db1.* TO 'user1'@'localhost';
```

#### 3.6 **授予跨主机权限**

为 `user1` 用户授予对 `db1` 数据库中所有表的 `SELECT` 权限，允许从任意主机连接：

```
GRANT SELECT ON db1.* TO 'user1'@'%';
```

### 4. **撤销权限**

如果需要撤销权限，可以使用 `REVOKE` 命令：

#### 4.1 **撤销所有权限**

```
REVOKE ALL PRIVILEGES ON db1.* FROM 'user1'@'localhost';
```

#### 4.2 **撤销特定权限**

撤销 `user1` 用户对 `db1.table1` 表的 `INSERT` 权限：

```
REVOKE INSERT ON db1.table1 FROM 'user1'@'localhost';
```

### 5. **刷新权限**

在修改权限后，MariaDB 会自动缓存权限，因此需要使用 `FLUSH PRIVILEGES` 来刷新权限，使其立即生效。

```
FLUSH PRIVILEGES;
```

### 6. **查看用户权限**

要查看某个用户的权限，可以使用以下命令：

```
SHOW GRANTS FOR 'user1'@'localhost';
```

### 7. **删除用户**

如果要删除某个用户，可以使用 `DROP USER` 命令：

```
DROP USER 'user1'@'localhost';
```

### 8. **创建用户并授权**

通常，创建用户后需要为该用户授权，下面是一个示例：

#### 8.1 **创建用户**

```
CREATE USER 'user1'@'localhost' IDENTIFIED BY 'password';
```

#### 8.2 **为用户授权**

```
GRANT ALL PRIVILEGES ON db1.* TO 'user1'@'localhost';
```

### 9. **常见权限错误和修复**

- **ERROR 1044 (42000): Access denied for user 'root'@'localhost'**
  这个错误通常表示当前用户没有足够的权限来执行所需的操作。请确保你使用的账户有足够的权限，或者使用 `GRANT` 命令为该用户授予更多权限。

### 总结

- `GRANT` 用于授予用户权限。

- `REVOKE` 用于撤销用户权限。

- `SHOW GRANTS` 用于查看用户的当前权限。

- 使用 `FLUSH PRIVILEGES` 刷新权限。

- 权限可以针对数据库、表、列和操作进行精细控制。



## InnoDB和MyISAM

`InnoDB` 和 `MyISAM` 都是 MySQL 数据库中常见的存储引擎。每个引擎有自己的特点、优缺点以及适用场景。下面是这两种存储引擎的比较：

### 1. **InnoDB 存储引擎**

`InnoDB` 是 MySQL 的默认存储引擎，并且它支持事务处理、外键约束和行级锁定等功能，适合高并发、多事务的应用场景。

#### 特点：

- **事务支持**：`InnoDB` 支持 **ACID** 事务（原子性、一致性、隔离性、持久性），能够确保数据的一致性和可靠性。
- **行级锁定**：使用行级锁定，可以提高并发性，尤其适合高并发的应用环境。
- **外键支持**：支持外键约束，能够确保数据完整性，防止出现无效或孤立的数据。
- **崩溃恢复**：`InnoDB` 内置崩溃恢复功能。即使发生崩溃，也能在恢复过程中恢复未提交的事务。
- **MVCC（多版本并发控制）**：通过多版本并发控制机制，可以减少读操作的锁定，从而提高并发性能。

#### 优点：

- 支持事务，适合需要高数据一致性的应用。
- 行级锁定提高了并发性。
- 内置崩溃恢复机制，保证数据的安全性。
- 支持外键约束，确保数据完整性。

#### 缺点：

- 相比于 `MyISAM`，`InnoDB` 的写操作稍微慢一些。
- 对磁盘空间的使用较为高效，但需要更多的内存来管理缓存。

#### 适用场景：

- 需要高并发和数据一致性的场景，比如金融系统、电商平台、社交媒体等。

------

### 2. **MyISAM 存储引擎**

`MyISAM` 是 MySQL 中较早的存储引擎，适用于不需要事务处理、并发较低的场景。它是基于表级锁定的引擎，通常比 `InnoDB` 更快，但缺乏一些关键的功能，如事务、外键等。

#### 特点：

- **表级锁定**：`MyISAM` 使用的是表级锁定，这在多用户并发写操作时可能会产生锁竞争，影响性能。
- **不支持事务**：`MyISAM` 不支持事务，因此不保证操作的原子性。
- **没有外键支持**：不能定义外键约束。
- **表级存储**：每个表都会有独立的文件，这使得表之间的管理比较简单。

#### 优点：

- **性能较好**：在查询操作频繁，且对数据一致性要求不高的场景中，`MyISAM` 提供了比 `InnoDB` 更高的性能。
- **适合只读或少量写操作的场景**：例如日志记录、统计数据等。
- **磁盘空间占用较少**：由于不支持事务和外键等机制，`MyISAM` 存储相对较小。

#### 缺点：

- **不支持事务**：数据一致性和完整性无法得到保证。
- **表级锁定**：会导致在高并发写操作时性能下降。
- **崩溃恢复较差**：如果系统崩溃，数据恢复较为困难，需要手动修复。
- **不支持外键约束**：无法进行数据完整性约束。

#### 适用场景：

- 适合只读或读多写少的应用，例如日志记录、数据仓库、静态数据存储等。

------

### 3. **对比总结**

| 特性             | **InnoDB**                             | **MyISAM**                                 |
| ---------------- | -------------------------------------- | ------------------------------------------ |
| **事务支持**     | 支持（ACID）                           | 不支持                                     |
| **锁定机制**     | 行级锁定                               | 表级锁定                                   |
| **外键支持**     | 支持                                   | 不支持                                     |
| **崩溃恢复**     | 自动恢复（日志和重做日志）             | 无自动恢复（需要手动修复）                 |
| **性能（查询）** | 适合高并发写操作，但读取性能稍低       | 适合查询频繁的应用，读取性能优越           |
| **性能（写入）** | 写操作稍慢（因为行级锁和事务处理）     | 写操作较快（没有事务和行级锁的开销）       |
| **并发性**       | 高并发（行级锁和事务支持）             | 较差（表级锁，写操作时性能下降）           |
| **适用场景**     | 需要高并发、事务支持、数据一致性要求高 | 读取多、写入少的场景，且数据一致性要求不高 |
| **默认存储引擎** | 默认（从 MySQL 5.5 起）                | 需要手动设置（MySQL 5.5 之前是默认引擎）   |

------

### 4. **如何选择存储引擎**

- **选择 InnoDB**：
  - 如果你的应用需要**事务支持**、**数据一致性**、**高并发**以及**外键约束**。
  - 适合需要写操作频繁或需要高可靠性的场景，如银行、电子商务、社交平台等。
- **选择 MyISAM**：
  - 如果你的应用主要是**只读或读多写少**，并且不需要事务支持和外键约束。
  - 适合日志记录、统计查询、搜索引擎等场景。

根据具体的业务需求和数据访问模式，选择适合的存储引擎，以便获得最佳的性能和数据安全保障。



### 5.相关文件

MyISAM 存储引擎相关文件

```
- tbl_name.frm 表格式定义
- tbl_name.sdi 表格式定义（mysql8.0开始，但是MariaDB目前依然在使用frm）
- tbl_name.MYD 数据文件
- tbl_name.MYI 索引文件
```

InnoDB 存储引擎相关文件

```
- tbl_name.frm 表格式定义
- tbl_name.ibd 数据和索引文件
```



## INDEX索引

### 优点/缺点

#### 优点

```powershell
- 大大加快数据的检索速度;
- 创建唯一性索引，保证数据库表中每一行数据的唯一性;
- 加速表和表之间的连接;
- 在使用分组和排序子句进行数据检索时，可以显著减少查询中分组和排序的时间。
```

#### 缺点

```powershell
- 索引需要占物理空间。
- 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，降低了数据的维护速度。
```



### 索引数据结构

```powershell
	MySQL索引的算法和结构主要取决于存储引擎。对于InnoDB存储引擎，最常用的索引结构是B+树索引。B+树索引具有以下特点：
    所有数据都会出现在叶子节点：叶子节点形成一个单向链表，便于范围搜索和排序。
    非叶子节点仅起到索引数据作用：不存储实际数据，只存储索引，使得树的层级更低，性能更高。
```

#### 类型

```powershell
- 聚簇（集）索引、非聚簇索引：数据和索引是否存储在一起
- 主键索引、二级（辅助）索引
- 稠密索引、稀疏索引：是否索引了每一个数据项
- 简单索引、组合索引：是否是多个字段的索引
- 左前缀索引：取前面的字符做索引
- 覆盖索引：从索引中即可取出要查询的数据，性能高
```



### 索引结构-树

按照有序性，可以分为有序树和无序树：

```powershell
- 无序树：树中任意节点的子结点之间没有顺序关系
- 有序树：树中任意节点的子结点之间有顺序关系
```

按照节点包含子树个数，可以分为B树和二叉树，二叉树可以分为以下几种：

```powershell
- 二叉树：每个节点最多含有两个子树的树称为二叉树
- 二叉查找树：
 	首先它是一颗二叉树，
 		- 若左子树不空，则左子树上所有结点的值均小于它的根结点的值，
 		- 若右子树不空，则右子树上所有结点的值均大于它的根结点的值，
 		- 左、右子树也分别为二叉排序树
- 满二叉树：叶节点除外的所有节点均含有两个子树的树被称为满二叉树
- 完全二叉树：如果一颗二叉树除去最后一层节点为满二叉树，且最后一层的结点依次从左到右分布
- 霍夫曼树：带权路径最短的二叉树
- 红黑树：
 	红黑树是一颗特殊的二叉查找树，
 		- 每个节点都是黑色或者红色，根节点、叶子节点是黑色。
 		- 如果一个节点是红色的，则它的子节点必须是黑色的
- 平衡二叉树(AVL)：一 棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树
```



#### **1. 二叉树 (Binary Tree)**

**定义**：
每个节点最多有两个子节点，分别称为左子节点和右子节点。

**特点**：

- **满二叉树**：所有非叶子节点都有两个子节点，且所有叶子节点在同一层。
- **完全二叉树**：除了最后一层，其他层是满的，最后一层的节点从左到右依次排列。
- **二叉搜索树 (BST)**：左子树的值小于根节点，右子树的值大于根节点。

**应用场景**：

- 二叉搜索树用于快速查找、插入和删除操作。

------

#### **2. 红黑树 (Red-Black Tree)**

**定义**：
一种自平衡的二叉搜索树，添加了红黑节点的着色规则。

**特点**：

1. 每个节点是红色或黑色。
2. 根节点是黑色。
3. 红色节点的子节点必须是黑色（不能有连续的红色节点）。
4. 从任意节点到其叶子节点的路径上，黑色节点数量相同。
5. 树的高度近似平衡（最大高度约为 `$\log_2(n)$`）。

**优点**：

- 查找、插入和删除的时间复杂度均为 O(log⁡n)O(\log n)O(logn)。
- 高度平衡，性能稳定。

**应用场景**：

- Java中的`TreeMap`、`TreeSet`，C++ STL中的`map`和`set`。

------

#### **3. B树**

**定义**：
一种多路搜索树（不是二叉树），广泛应用于数据库和文件系统。

**特点**：

1. 每个节点可以有多个子节点和多个关键字。
2. 关键字按顺序存储，子节点之间的值分布在对应的关键字范围内。
3. 树的所有叶子节点在同一层。
4. 内部节点的子节点数在一定范围内（最小度数`t`）：
   - 子节点数范围为 `[t,2t]`（根节点除外）。
5. 自平衡，树的高度较低。

**优点**：

- 减少磁盘I/O操作，适合大规模存储。

**应用场景**：

- 数据库索引（如MySQL中的`InnoDB`存储引擎）。

------

#### **4. B+树**

**定义**：
B树的变种，是数据库系统中最常用的数据结构之一。

**特点**：

1. 非叶子节点只存储索引，不存储数据。
2. 所有数据存储在叶子节点，叶子节点通过链表相连，便于范围查询。
3. 叶子节点的关键字按照顺序排列。
4. 非叶子节点的子节点数范围仍为 `[t,2t]`。

**优点**：

- **范围查询效率高**：通过叶子节点的链表直接扫描。
- **磁盘友好**：非叶子节点存储索引，降低内存占用，提高I/O性能。

**应用场景**：

- 数据库系统的主流索引结构（如MySQL的B+树索引）。

------

#### **对比总结**

| 特性     | 二叉树     | 红黑树     | B树                | B+树               |
| -------- | ---------- | ---------- | ------------------ | ------------------ |
| 子节点数 | 每节点 ≤ 2 | 每节点 ≤ 2 | 每节点范围 [t, 2t] | 每节点范围 [t, 2t] |
| 平衡性   | 无法保证   | 自平衡     | 自平衡             | 自平衡             |
| 应用场景 | 基础算法   | 内存存储   | 磁盘存储           | 数据库索引         |
| 范围查询 | 效率低     | 效率低     | 中等               | 高效               |
| 数据存储 | 所有节点   | 所有节点   | 所有节点           | 仅叶子节点         |



### Hash索引

```powershell
Hash索引：基于哈希表实现，只有精确匹配索引中的所有列的查询才有效，索引自身只存储索引列对应的哈希值和数据指针，索引结构紧凑，查询性能好。
Memory 存储引擎支持显式 hash 索引，InnoDB 和 MyISAM 存储引擎不支持。
适用场景：只支持等值比较查询，包括=, <>, IN()。
```

```powershell
不适合使用hash索引的场景
	- 不适用于顺序查询：索引存储顺序的不是值的顺序
	- 不支持模糊匹配
	- 不支持范围查询
	- 不支持部分索引列匹配查找：如A，B列索引，只查询A列索引无效
```



### 索引优化

```powershell
- 独立地使用列：尽量避免其参与运算，独立的列指索引列不能是表达式的一部分，也不能是函数的参数，在where条件中，始终将索引列单独放在比较符号的一侧，尽量不要在列上进行运算（函数操作和表达式操作）
- 左前缀索引：构建指定索引字段的左侧的字符数，要通过索引选择性（不重复的索引值和数据表的记录总数的比值）来评估，尽量使用短索引，如果可以，应该制定一个前缀长度
- 多列索引：AND操作时更适合使用多列索引，而非为每个列创建单独的索引
- 选择合适的索引列顺序：无排序和分组时，将选择性最高放左侧
- 只要列中含有NULL值，就最好不要在此列设置索引，复合索引如果有NULL值，此列在使用时也不会使用索引
- 对于经常在where子句使用的列，最好设置索引
- 对于有多个列where或者order by子句，应该建立复合索引
- 对于like语句，以 % 或者 _ 开头的不会使用索引，以 % 结尾会使用索引
- 尽量不要使用not in和<>操作，虽然可能使用索引，但性能不高
- 不要使用RLIKE正则表达式会导致索引失效
- 查询时，能不要*就不用*，尽量写全字段名，比如：select id,name,age from students;
- 大部分情况连接效率远大于子查询
- 在有大量记录的表分页时使用limit
- 对于经常使用的查询，可以开启查询缓存
- 多使用explain和profile分析查询语句
- 查看慢查询日志，找出执行时间长的sql语句优化
```



### 命令

```powershell
查看索引
SHOW INDEX FROM [db_name.]tbl_name;

查询表内容是否使用索引
explain select * from 表 where 字段=值\G
key: PRIMARY 使用主键索引
key: NULL 没有使用索引

创建索引
CREATE INDEX idx_name ON student(name); 
 	创建一个索引，该索引包含 name 列的完整值。
 	这意味着，无论 name 列中的字符串有多长，整个字符串都会被包含在索引中。
CREATE INDEX idx_name ON student(name(10)); 
 	创建一个前缀索引，该索引仅包含 name 列的前 10 个字符。
 	这意味着，索引是基于 name 列值的前 10 个字符创建的。
 	
删除索引
DROP INDEX index_name ON tbl_name;
ALTER TABLE tbl_name DROP INDEX index_name;
	index_name：要删除的索引的名称。
	tbl_name：包含该索引的表名。
	
	ALTER TABLE 方式更符合语义化修改表结构的场景。
	
	
select @@profiling; 	# 指示是否启用了查询性能分析（profiling）
set profiling=1;		# 开启性能分析查询
show profiles;			# 查看查询的性能分析
SHOW WARNINGS\G			# 查看警告信息
```



```powershell
B+树索引是左前缀特性，即左匹配可以使用索引，右匹配不使用索引，包含匹配也不使用索引
mysql> explain select * from student where name like 'g%'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: student
   partitions: NULL
         type: range
possible_keys: idx_name
          key: idx_name
      key_len: 82
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)

mysql> explain select * from student where name like '%g'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: student
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 20
     filtered: 11.11
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
mysql> explain select * from student where name like '%g%'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: student
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 20
     filtered: 11.11
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```



## 并发控制

### 锁机制

| 说明                                                         | 锁类型 |
| ------------------------------------------------------------ | ------ |
| 共享锁，也称为 S 锁，只读不可写（包括当前事务），多个读互不阻塞 | 读锁   |
| 独占锁，排它锁，也称为 X 锁，写锁会阻塞其它事务（不包括当前事务）的读和写 | 写锁   |

锁兼容 & 锁冲突

```powershell
	S 锁和 S 锁是兼容的，X 锁和其它锁都不兼容，举个例子:
	事务 T1 获取了一个行 r1 的 S 锁，另外事务 T2 可以立即获得行 r1 的 S 锁，此时 T1 和 T2 共同获得行 r1 的 S 锁，此种情况称为锁兼容，
	但是另外一个事务 T2 此时如果想获得行 r1 的 X 锁，则必须等待 T1 对行 r1 锁的释放，此种情况也称为锁冲突。
```



### 锁细节

锁是加在索引上的，而不是加在数据上的

锁粒度

| 存储引擎 | 锁粒度 |
| -------- | ------ |
| MyISAM   | 表级锁 |
| InnoDB   | 行级锁 |

锁的实现

| 实现方式 | 说明                             |
| -------- | -------------------------------- |
| 存储引擎 | 自行实现该引擎的锁策略和锁粒度   |
| 自行实现 | 在程序中或命令行下用命令显式实现 |

锁分类

| 锁类型 | 说明                 |
| ------ | -------------------- |
| 隐式锁 | 由存储引擎自动施加锁 |
| 显式锁 | 用户手动请求         |

加锁策略：在 "锁粒度" 及 "数据安全性" 寻求的平衡机制



#### 命令

```powershell
加读锁
lock tables 表名 read;
释放锁
unlock tables;
在那个终端上加的锁就要在那个终端上释放锁

加写锁（加写锁后，当前终端可以进行正常的查看和写操作）
lock table 表名 write;
释放锁
unlock tables ;
在那个终端上加的锁就要在那个终端上释放锁

添加全局读锁
flush tables with read lock;
退出全局锁
exit/quit/\q
```



## 事务

事务是数据库管理系统中保证数据一致性的重要机制。事务的特性通常用 **ACID** 来描述，这四个字母代表了事务必须具备的四个核心特性：**原子性 (Atomicity)**、**一致性 (Consistency)**、**隔离性 (Isolation)** 和 **持久性 (Durability)**。

------

### **1. 原子性 (Atomicity)**

**定义**：
事务是一个不可分割的操作单元，事务中的所有操作要么全部完成，要么全部不执行。

**例子**：
假设银行账户之间转账：

- 从账户A扣款100元；
- 向账户B存款100元。

如果事务中途失败（如数据库崩溃），账户A和账户B的状态不会出现不一致（扣款成功但存款未完成）。事务要么回滚所有更改，要么完成所有操作。

------

### **2. 一致性 (Consistency)**

**定义**：
事务执行前后，数据库必须处于一致的状态。这意味着，数据在事务完成时必须满足所有定义的完整性约束。

**例子**：
银行账户总余额在事务前后应该保持不变。如果转账操作完成后，数据库中账户的总余额发生了变化，就违反了一致性。

------

### **3. 隔离性 (Isolation)**

**定义**：
多个事务并发执行时，一个事务的执行不应被其他事务的中间状态所干扰。换句话说，事务之间是相互独立的。

**隔离级别**（从低到高）：

- **读未提交 (Read Uncommitted)**：允许读取未提交的数据，可能导致“脏读”。
- **读已提交 (Read Committed)**：只能读取已提交的数据，避免了脏读。
- **可重复读 (Repeatable Read)**：在一个事务中，多次读取同一数据返回的结果相同，避免了不可重复读。
- **可序列化 (Serializable)**：完全隔离，事务逐个顺序执行，避免了幻读。

**问题**：

- **脏读**：一个事务读取了另一个未提交事务的数据。
- **不可重复读**：一个事务两次读取同一数据，结果不一致。
- **幻读**：一个事务两次查询同一条件时，结果的记录数发生了变化。

------

### **4. 持久性 (Durability)**

**定义**：
一旦事务提交，其对数据库的修改将永久保存，即使系统崩溃也不会丢失数据。

**例子**：
当银行转账事务提交后，即使系统断电或崩溃，账户变动（如余额扣减和增加）也会被永久记录。

------

### **总结**

| 特性   | 描述                                                         | 作用             |
| ------ | ------------------------------------------------------------ | ---------------- |
| 原子性 | 事务中所有操作要么全部完成，要么全部回滚。                   | 确保事务完整性   |
| 一致性 | 数据在事务执行前后保持一致，遵守完整性约束。                 | 数据一致性保障   |
| 隔离性 | 事务之间相互独立，避免并发导致的问题（脏读、不可重复读、幻读）。 | 确保事务互不干扰 |
| 持久性 | 提交后的事务修改将永久保存，即使系统崩溃也不会丢失。         | 数据的可靠性保障 |



### 开启结束事务

```powershell
启用一个事务
BEGIN
BEGIN WORK
START TRANSACTION

结束事务
#提交执行
COMMIT
#回滚
ROLLBACK

truncate、drop不支持事务回滚

```

### 自动提交

```powershell
#查看是否开启自动提交
select @@autocommit;
#关闭自动提交
set autocommit=0;
#开启自动提交
set autocommit=1;
也可以修改配置文件修改配置文件
root@ubuntu24-16:~# vim /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
autocommit=0	#关闭自动提交 =1开启自动提交
```



### 合并提交

```powershell
mysql> source /root/testlog.sql;
Query OK, 1 row affected (0.00 sec)

Database changed
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> call sp_testlog;
Query OK, 1 row affected (1.32 sec)

mysql> commit
    -> ;
Query OK, 0 rows affected (0.05 sec)


上次执行
mysql> call sp_testlog;
Query OK, 1 row affected (3 min 2.92 sec)
```



### 事务保存点

```powershell
mysql> begin;			#开启事务
Query OK, 0 rows affected (0.00 sec)

mysql> insert into stu1 (name,age,gender)values('u1',11,'M');
Query OK, 1 row affected (0.00 sec)

mysql> savepoint p1;	#保存回滚点p1
Query OK, 0 rows affected (0.01 sec)

mysql> insert into stu1 (name,age,gender)values('u2',22,'M');
Query OK, 1 row affected (0.00 sec)

mysql> savepoint p2;	#保存回滚点p2
Query OK, 0 rows affected (0.00 sec)

mysql> insert into stu1 (name,age,gender)values('u3',33,'M');
Query OK, 1 row affected (0.00 sec)

mysql> savepoint p3;	#保存回滚点p3
Query OK, 0 rows affected (0.00 sec)

mysql> rollback to p2; 	#回滚到回滚点p2
Query OK, 0 rows affected (0.00 sec)

mysql> commit;
```



```powershell
查看当前正在进行的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX\G
```



### 死锁

![image-20250104212920653](5day-png\23死锁)

```powershell
在 MySQL中，两个或多个事务在同一资源相互占用，并请求锁定对方占用的资源的状态。
死锁并不是 MySQL 中独有的，只要出现资源互相竞争的情况都有可能出现死锁。
在 MySQL 中出现死锁时，MySQL 服务会自行处理，自行回滚一个事务，防止死锁的出现。

如果要提交事务1，因为依赖到事务2的第一条语句，所以，需要保证事务2先提交完毕
如果要提交事务2，因为依赖到事务1的第一条语句，所以，需要保证事务1先提交完毕
但是默认情况下，两个事务都是不会让对方的，所以就出现互不相让的效果，导致谁也无法提交，形成死锁的效果。
```

```powershell
查看正在进行中的事务
SELECT * FROM information_schema.INNODB_TRX;
查看锁
SELECT * FROM information_schema.INNODB_LOCKS;
MySQL8.0.13 及以后使用此语句查看锁
SELECT * FROM performance_schema.data_locks;
查看锁等待
SELECT * FROM information_schema.INNODB_LOCK_WAITS;
MySQL8.0.13及以后使用此语句查看锁等待
SELECT * FROM performance_schema.data_lock_waits;
查看事务锁的超时时长，默认50s, 也就是说，超过50秒之后，事务会自动断开，事务过程中的操作也会自动消失
mysql> show global variables like 'innodb_lock_wait_timeout';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_lock_wait_timeout | 50    |
+--------------------------+-------+
1 row in set (0.00 sec)
```



### 事务隔离

事务隔离级别（Transaction Isolation Level）是数据库管理系统（DBMS）中为了控制事务之间并发访问数据时的相互影响而设计的机制。它规定了一个事务与其他事务之间的可见性和影响范围，直接影响并发控制和性能。

在 SQL 标准中，事务隔离级别定义了 4 个等级，从低到高分别为：**读未提交（Read Uncommitted）**、**读已提交（Read Committed）**、**可重复读（Repeatable Read）** 和 **可串行化（Serializable）**。

------

#### **1. 读未提交（Read Uncommitted）**

- 特点：

  - 事务可以读取其他事务未提交的数据（脏数据）。
  - 不保证数据的一致性。

- 可能的问题：

  - **脏读**：一个事务读取到另一个事务未提交的数据，而这些数据随后可能被回滚。

- 适用场景：

  - 对一致性要求不高，但需要高性能的场景。

- 示例：

  ```
  SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
  ```

------

#### **2. 读已提交（Read Committed）**

- 特点：

  - 只能读取其他事务已提交的数据。
  - 避免了脏读。

- 可能的问题：

  - **不可重复读**：同一事务中的两次读取操作可能返回不同的数据，因为其他事务可能在中间修改并提交了数据。

- 适用场景：

  - 数据一致性要求一般的系统，广泛用于大多数场景。

- 示例：

  ```
  SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
  ```

------

#### **3. 可重复读（Repeatable Read）**

- 特点：

  - 保证同一事务中多次读取同一数据返回的结果是一致的，即使其他事务在此期间提交了数据修改。
  - 防止脏读和不可重复读。

- 可能的问题：

  - **幻读**：同一事务中两次查询范围数据时，结果可能因其他事务插入或删除数据而不同。

- 适用场景：

  - 需要更高一致性且允许一定程度并发的场景。MySQL 的 InnoDB 存储引擎默认使用此级别。

- 示例：

  ```
  SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
  ```

------

#### **4. 可串行化（Serializable）**

- 特点：

  - 最高的隔离级别，所有事务依次顺序执行，完全避免脏读、不可重复读和幻读。
  - 使用锁机制或 MVCC（多版本并发控制）实现。

- 可能的问题：

  - 性能开销极高，容易导致事务等待和死锁。

- 适用场景：

  - 需要强一致性的场景，如金融系统。

- 示例：

  ```
  SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  ```

------

#### **总结表格**

| 隔离级别 | 脏读   | 不可重复读 | 幻读   | 性能 |
| -------- | ------ | ---------- | ------ | ---- |
| 读未提交 | 可能   | 可能       | 可能   | 最高 |
| 读已提交 | 不可能 | 可能       | 可能   | 较高 |
| 可重复读 | 不可能 | 不可能     | 可能   | 中等 |
| 可串行化 | 不可能 | 不可能     | 不可能 | 最低 |

------

#### **MySQL 默认隔离级别**

- MySQL 的 InnoDB 存储引擎默认使用 **可重复读（Repeatable Read）** 隔离级别。

- 你可以通过以下命令查看当前隔离级别：

  ```
  SELECT @@transaction_isolation;
  ```

------

#### **事务隔离级别设置**

1. 设置全局隔离级别（对所有新连接生效）：

   ```
   SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
   ```

2. 设置会话隔离级别（仅对当前会话生效）：

   ```
   SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
   ```

3. 在事务内指定隔离级别：

   ```
   START TRANSACTION ISOLATION LEVEL SERIALIZABLE;
   ```





在事务并发操作中，由于多个事务对共享数据的同时访问，可能会导致数据一致性问题。这些问题包括 **脏读**、**不可重复读**、**幻读** 和 **丢失更新**。理解这些问题有助于选择合适的事务隔离级别，从而平衡性能和一致性。

------

#### **1. 脏读 (Dirty Read)**

**定义**：
一个事务读取了另一个未提交事务所修改的数据，而这些数据可能会被回滚。

**场景**：

- 事务 A 修改了数据，但未提交。
- 事务 B 读取了事务 A 修改的数据。
- 如果事务 A 回滚，事务 B 读取到的数据将是无效的。

**解决方法**：
通过隔离级别 **读已提交 (Read Committed)** 或更高级别来避免。

------

#### **2. 不可重复读 (Non-Repeatable Read)**

**定义**：
同一个事务中，两次读取同一数据，结果却不一致，因为其他事务在两次读取之间修改并提交了该数据。

**场景**：

- 事务 A 读取了一行数据。
- 事务 B 修改了这行数据并提交。
- 事务 A 再次读取时，结果与第一次读取不同。

**解决方法**：
通过隔离级别 **可重复读 (Repeatable Read)** 或更高级别来避免。

------

#### **3. 幻读 (Phantom Read)**

**定义**：
同一个事务中，两次查询返回的记录集合不同，因为其他事务在查询范围内插入或删除了数据。

**场景**：

- 事务 A 查询满足某条件的数据。
- 事务 B 插入了满足该条件的新数据并提交。
- 事务 A 再次查询时，发现了额外的记录（幻影数据）。

**解决方法**：
通过隔离级别 **可串行化 (Serializable)** 或特定的锁机制来避免。

------

#### **4. 丢失更新 (Lost Update)**

**定义**：
两个事务同时更新同一行数据，其中一个事务的更新被另一个事务覆盖，导致数据更新丢失。

**场景**：

- 事务 A 和事务 B 读取了同一行数据。
- 事务 A 更新数据并提交。
- 事务 B 更新同一行数据并提交，事务 A 的更新被覆盖。

**解决方法**：
通过锁机制或显式版本控制来避免。

------

#### **问题总结表格**

| 问题名称       | 定义                               | 出现原因                                 | 解决方法/隔离级别              |
| -------------- | ---------------------------------- | ---------------------------------------- | ------------------------------ |
| **脏读**       | 读取到未提交的事务数据             | 一个事务读取了另一个事务未提交的数据     | 读已提交或更高级别             |
| **不可重复读** | 同一事务中两次读取同一数据结果不同 | 其他事务修改了数据并提交                 | 可重复读或更高级别             |
| **幻读**       | 同一事务中两次查询返回的记录集不同 | 其他事务插入或删除了数据                 | 可串行化或锁机制               |
| **丢失更新**   | 一个事务的更新被另一个事务覆盖     | 多个事务同时修改同一数据，覆盖彼此的更新 | 乐观锁、悲观锁或串行化隔离级别 |

------

#### **事务隔离级别与并发问题的关系**

| 隔离级别 | 脏读   | 不可重复读 | 幻读   | 丢失更新 |
| -------- | ------ | ---------- | ------ | -------- |
| 读未提交 | 可能   | 可能       | 可能   | 可能     |
| 读已提交 | 不可能 | 可能       | 可能   | 可能     |
| 可重复读 | 不可能 | 不可能     | 可能   | 可能     |
| 可串行化 | 不可能 | 不可能     | 不可能 | 不可能   |

------

#### **如何选择合适的隔离级别**

- 如果对性能要求高，可以选择 **读已提交 (Read Committed)** 或 **读未提交 (Read Uncommitted)**，但需承担一定的并发问题风险。
- 如果需要强一致性，选择 **可重复读 (Repeatable Read)** 或 **可串行化 (Serializable)**，但需承担性能开销。

#### MVCC和事务的隔离级别

**多版本并发控制**（MVCC，Multi-Version Concurrency Control）是一种数据库并发控制机制，用于实现事务的隔离性。它通过维护数据的多个版本，使读取操作无需等待写入锁释放，从而提高系统性能。MVCC 与事务隔离级别密切相关，尤其是在常用的数据库（如 MySQL 的 InnoDB 存储引擎）中。

------

##### **MVCC 工作原理**

1. **数据版本**：
   每条记录会存储额外的元信息，例如：
   - 创建版本号（事务 ID）：记录该数据由哪个事务创建。
   - 删除版本号（事务 ID）：记录该数据在哪个事务中被标记为删除。
2. **事务 ID**：
   每个事务开始时会分配一个唯一的事务 ID，用于区分不同事务以及记录的版本。
3. **版本可见性规则**：
   - 每个事务只能看到在该事务开始之前提交的数据。
   - 事务运行过程中修改或删除的数据，对其他事务不可见，直到当前事务提交。
4. **快照读与当前读**：
   - **快照读**：读取的是符合版本规则的已提交数据（通过 MVCC 实现）。
   - **当前读**：读取最新数据，需要加锁（如 `SELECT ... FOR UPDATE` 或 `SELECT ... LOCK IN SHARE MODE`）。

------

##### **MVCC 与事务隔离级别的关系**

MVCC 在实现事务隔离级别时，主要影响的是 **读已提交 (Read Committed)** 和 **可重复读 (Repeatable Read)** 两种隔离级别。

##### **1. 读未提交（Read Uncommitted）**

- **实现方式**：
  不依赖 MVCC，直接读取最新的数据，包括未提交的数据。
- 特点：
  - 事务间没有隔离。
  - 可能发生脏读。

------

##### **2. 读已提交（Read Committed）**

- **实现方式**：
  每次查询都会创建一个新的快照，仅返回已提交的数据版本。
- 特点：
  - 避免脏读。
  - 存在不可重复读问题，因为同一事务的多次查询可能读取到不同的已提交数据。

------

##### **3. 可重复读（Repeatable Read）**

- **实现方式**：
  在事务开始时创建一个一致性快照，事务期间所有查询都基于该快照。
- 特点：
  - 避免脏读和不可重复读。
  - 可能存在幻读问题，但 MySQL 的 InnoDB 通过间隙锁（Gap Lock）解决了幻读问题。

------

##### **4. 可串行化（Serializable）**

- **实现方式**：
  不依赖 MVCC，而是通过强制加锁（如表锁或行锁）实现完全隔离。
- 特点：
  - 所有事务按照顺序执行，完全避免并发问题。
  - 性能开销高。

------

##### **MVCC 的优势**

1. **高并发性能**：
   - 读取操作不需要等待写锁释放，避免读写冲突。
   - 写入操作只影响当前事务，不会阻塞其他事务的读取。
2. **实现更高效的隔离级别**：
   - 通过维护多个版本，避免了锁争用，提高了事务隔离级别的实现效率。

------

##### **MySQL 中 MVCC 的应用**

MySQL 的 InnoDB 存储引擎默认使用 MVCC 实现隔离级别，主要用于 **读已提交 (Read Committed)** 和 **可重复读 (Repeatable Read)**：

- **快照读**：利用 MVCC 读取一致性视图（已提交版本）。
- **当前读**：直接操作最新数据版本，需要加锁。

##### **示例：快照读与当前读**

假设 `t` 表中有数据：

| id   | value | trx_id | delete_trx_id |
| ---- | ----- | ------ | ------------- |
| 1    | 100   | 10     | NULL          |

1. **快照读**：

   ```
   SELECT value FROM t WHERE id = 1;
   ```

   - 返回的是符合快照视图的数据版本，不加锁。

2. **当前读**：

   ```
   SELECT value FROM t WHERE id = 1 FOR UPDATE;
   ```

   - 返回最新数据，并加锁以防其他事务修改。

------

##### **MVCC 的局限性**

1. **空间开销**：
   - 数据版本需要额外存储，可能增加存储成本。
2. **一致性要求高的场景**：
   - 对于需要强一致性的事务（如金融系统），MVCC 可能不够，需使用更严格的隔离级别（如可串行化）。
3. **性能问题**：
   - 过多的旧版本可能导致版本链过长，增加性能开销。

------

##### **总结表格**

| 隔离级别 | 是否使用 MVCC  | 特点                       | 问题解决情况                   |
| -------- | -------------- | -------------------------- | ------------------------------ |
| 读未提交 | 否             | 直接读取最新数据           | 存在脏读、不可重复读、幻读     |
| 读已提交 | 是             | 每次查询读取最新提交的版本 | 避免脏读，存在不可重复读、幻读 |
| 可重复读 | 是             | 事务内快照一致             | 避免脏读、不可重复读，存在幻读 |
| 可串行化 | 否（加锁实现） | 事务完全串行化执行         | 避免所有问题                   |



#### 命令

```powershell
#MySQL8.0之前及MariaDB
SET tx_isolation='READ-UNCOMMITTED|READ-COMMITTED|REPEATABLEREAD|SERIALIZABLE'
#MySQL8.0及以后
SET transaction_isolation='READ-UNCOMMITTED|READ-COMMITTED|REPEATABLEREAD|SERIALIZABLE'

配置文件中更改
root@ubuntu24-16:~# vim /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
transaction-isolation=READ-UNCOMMITTED|READ-COMMITTED|REPEATABLEREAD|SERIALIZABLE
读未提交|读已提交|可重复读|可串行化
```





## 日志

MySQL 的日志系统是数据库的重要组成部分，用于记录各种事件和操作。以下是 MySQL 常见的日志类型及其功能：

### 1. **错误日志（Error Log）**

- **功能**: 记录服务器启动、运行、关闭过程中的错误信息、警告和通知。

- MySQL 中错误日志中记录的主要内容

  - ```powershell
    - mysqld 启动和关闭过程中输出的事件信息
    - mysqld 运行中产生的错误信息
    - event scheduler 运行一个 event 时产生的日志信息
    - 在主从复制架构中的从服务器上启动从服务器线程时产生的信息
    ```

- 查看MySQL默认的错误日志文件位置

  - ```sql
    select @@log_error;
    ```

- 用途:

  - 诊断 MySQL 启动失败或运行时的问题。
  - 记录重要事件，例如服务器重启或崩溃。

- 配置:

  - 可以通过 `log_error` 参数设置错误日志文件路径。

  - 示例:

    ```ini
    log_error = /var/log/mysql/error.log
    ```

- 定义错误日志级别

  - ```sql
    mysql 查看错误日志级别
    	select @@log_error_verbosity;
    mariadb 查看错误日志级别
    	select @@log_warnings;
    修改
    mysql
    	SET GLOBAL log_error_verbosity=2;
    mariadb
    	SET SESSION log_warnings = 1;
    ```

    ```init
    配置文件定制
    mysql
    vim /etc/mysql/mysql.conf.d/mysqld.cnf
    [mysqld]
    log_warnings = 1
    
    mariadb
    vim /etc/mysql/mariadb.conf.d/50-server.cnf
    [mysqld]
    log_warnings = 1
    ```

    | log_warnings值 | 日志记录的内容              |
    | -------------- | --------------------------- |
    | 1              | ERROR                       |
    | 2              | ERROR，WARNING              |
    | 3              | ERROR，WARNING，INFORMATION |

    

### 2. **常规查询日志（General Query Log）**（通用日志）

**功能**: 记录客户端对 MySQL 发出的所有查询或命令（包括连接和断开）。

用途:

- 调试目的，检查系统收到的具体 SQL 语句。
- 监控用户活动。

配置:

启用:

```sql
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/path/to/general.log';
```

默认关闭，启用可能会影响性能。

查看默认的配置选项 

- ```powershell
  #相关配置项
  #general_log=0|1 是否开启通用日志
  #general_log_file=/path/log_file 日志文件
  #log_output="FILE|TABLE|NONE" 记录到文件中还是记录到表中
  ```

- 开启通用日志功能

  - ```sql
    临时开启
    set global general_log=1;
    查看
    select @@general_log,@@general_log_file,@@log_output;
    修改参数，将日志记录到表中
    set global log_output="TABLE";
    
    永久开启
    MySQL# vim /etc/mysql/mysql.conf.d/mysqld.cnf
    mariadb# vim /etc/mysql/mariadb.conf.d/50-server.cnf
    [mysqld]
    general_log = ON
    general_log_file = /var/log/mysql/general.log
    ```

    

### 3. **慢查询日志（Slow Query Log）**

- **功能**: 记录执行时间超过指定阈值的 SQL 语句（即慢查询）。

- 用途:

  - 优化 SQL 语句，识别性能瓶颈。
  - 分析哪些查询耗时过长。

- 配置:

  - 相关参数:

    ```ini
    MySQL# vim /etc/mysql/mysql.conf.d/mysqld.cnf
    mariadb# vim /etc/mysql/mariadb.conf.d/50-server.cnf
    
    slow_query_log=0|1 		#是否开启记录慢查询日志
    log_slow_queries=0|1 	#同上，MariaDB 10.0/MySQL 5.6.1 版本后已删除
    slow_query_log_file=/path/log_file 	#日志文件路径
    long_query_time=N 		#慢查询的定义，默认10S，也就是说一条SQL查询语句，执行时长超过10S，将会被记录
    log_queries_not_using_indexes=0|1 	#只要查询语句中没有用到索引，或是全表扫描，都会记录，不管查询时长是否达到阀值
    
    log_slow_rate_limit=N 	#查询多少次才记录，MariaDB 特有配置项，默认值为1
    ```

    ```ini
    [mysqld]
    slow_query_log=1
    long_query_time=1
    log_queries_not_using_indexes=1
    ```

    

- mysqldumpslow-慢查询分析工具

  - ```powershell
    mysqldumpslow [ OPTS... ] [ LOGS... ]
    常用选项
    -v|--verbose 	#显示详细信息
    -d|--debug   	#调试模式
    --help     		#显示帮助      
    -s ORDER   		#排序
    al 				#根据平均加锁时长排序
    at 				#根据平均查询记录行数排序
    ar 				#根据平均查询时间排序
    c 				#根据查询结果行数排序
    l 				#根据加锁时长排序
    r 				#根据查询行数排序
    t 				#根据查询时长排序
    -r         		#根据排序规则反转
    -t N 			#只果看日志中前N行记录  
    -a         		#显示具体内容，不以抽象形式显示
    -g PATTERN 		#使用 grep 过滤
    -h HOSTNAME 	#根基主机名匹配日志文件
    -l         		#不从总时间中忽略锁时间
    
    ```

    

### 4. **二进制日志（Binary Log）**

- **功能**: 记录对数据库执行的所有更改（如 `INSERT`、`UPDATE`、`DELETE`），不包括查询的 `SELECT` 操作。

- 用途:

  - 数据恢复（基于二进制日志的增量恢复）。
  - 主从复制（Replication）的核心。
  - 审计数据库变化。

- 二进制日志三种格式

  ```powershell
  - Statement：基于语句的记录模式，日志中会记录原生执行的 SQL 语句，对于某些函数或变量，不会替换。
  - Row：基于行的记录模式，会将 SQL 语句中的变量和函数进行替换后再记录。
  - Mixed：混合记录模式，在此模式下，MySQL 会根据具体的 SQL 语句来分析采用哪种模式记录日志。
  
  #原始SQL语句
  update t1 set age=20 where id<5;
  #Statement 方式记录的日志，只记录语句
  update t1 set age=20 where id<5;
  #Row 方式记录的日志,会解析成所有的具体语句
  update t1 set age=20 where id=4;
  update t1 set age=20 where id=3;
  update t1 set age=20 where id=2;
  update t1 set age=20 where id=1;
  #Statement 比较节约磁盘空间，但无法百分百还原场景
  #Row 相对于 Satement 占用空间较大
  ```

- 配置:

  - 相关配置和服务器变量

    ```powershell
    sql_log_bin=1|0 			#是否开启二进制日志，可动态修改，是系统变量，而非服务器选项
    log_bin=/path/file_name 	#是否开启二进制日志，两项都要是开启状态才表示开启，此项是服务器选项，指定的是日志文件路径
    log_bin_basename=/path/log_file_name 		#binlog文件前缀
    log_bin_index=/path/file_name 				#索引文件路径
    binlog_format=STATEMENT|ROW|MIXED 			#log文件格式
    max_binlog_size=1073741824 	#单文件大小，默认1G，超过大小会自动生成新的文件,重启服务也会生成新文件
    binlog_cache_size=4m 		#二进制日志缓冲区大小，每个连接独占大小
    max_binlog_cache_size=512m 	#二进制日志缓冲区总大小，多个连接共享
    sync_binlog=1|0 			#二进制日志落盘规则，1表示实时写，0 表示先缓存，再批量写磁盘文件
    expire_logs_days=N 			#二进制文件自动保存的天数，超出会被删除，默认0，表示不删除
    ```

  - 启用:

    ```ini
    log_bin = /var/log/mysql/binlog
    binlog_format = ROW  # 推荐 ROW 格式
    ```

  - 格式:

    - **ROW**: 记录行的具体变化，适合大多数场景。
    - **STATEMENT**: 记录 SQL 语句本身。
    - **MIXED**: 结合了 ROW 和 STATEMENT 的优点。

- mysqlbinlog-查看二进制日志具体内容

  - ```
    mysqlbinlog [options] log-files
    常用选项
    -v|-vv|-vvv #详细显示
    --start-position=N #开始位置
    --stop-position=N #结束位置
    --start-datetime=VAL #开始时间，格式：YYYY-MM-DD hh:mm:ss
    --stop-datetime=VAL #结束时间
    --base64-output=VAL #是否输出base64编码过的内容，默认 auto, never|decode-rows|auto
    
    范例
    mysqlbinlog --start-position=起始位置 --stop-position=结束位置 /path/to/binlog.000001 -v
    mysqlbinlog --start-datetime="起始时间" --stop-datetime="结束时间" /path/to/binlog.000001 -v
    ```

- 二进制文件查看

  ```sql
  查看当前服务的二进制文件列表
  SHOW {BINARY | MASTER} LOGS
  查看正在使用的二进制文件
  SHOW MASTER STATUS
  ```

- 二进制日志中的事件

  - ```powershell
    *************************** 20. row ***************************
       Log_name: binlog.000001			#要查询的文件名，不指定默认第一个
            Pos: 1561					#事件开始位置
     Event_type: Xid					#事件类型
      Server_id: 1						#服务器标识
    End_log_pos: 1592					#事件结束位置
           Info: COMMIT /* xid=75 */	#具体内容
    ```

- 刷新二进制日志文件

  ```powershell
  - 重启服务
  systemctl restart mariadb|mysql
  - 手工刷新二进制文件
  flush logs;
  - mariadb专用的 mysqladmin 工具刷新
  mysqladmin flush-logs 			# 第一次刷新
  mysqladmin flush-binary-log 	# 第二次刷新
  ```

- 清理二进制日志

  ```powershell
  根据条件部份清理
  PURGE { BINARY | MASTER } LOGS { TO 'log_name' | BEFORE datetime_expr }
  示例：
   PURGE BINARY LOGS TO 'binlog.000002'; 			  # 删除binlog.000002之前的日志
   PURGE BINARY LOGS BEFORE '2022-12-31'; 		  # 清理 2022-12-31 之前的日志
   PURGE BINARY LOGS BEFORE '2023-01-01 21:00:00';  # 清理 2023-01-01 21:00:00 之前的日志
  全部清理，并重新生成日志文件，默认从1开始，一般在 master 主机第一次启动时执行此操作
  #MySQL8.0和 MariaDB 10.1.6 开始支持 to N,从指定文件开始
  RESET MASTER [TO N]; 		# 如果N不存在，则创建一个N
  示例：
   	reset master;
   	reset master to 10;
  ```

- 二进制日志远程同步 

  ```powershell
  1 客户端工具与服务端保持同一版本，且远程用户有所有权限，开启了二进制日志。
  服务端：
  vim /etc/mysql/mariadb.conf.d/50-server.cnf
  [mysqld]
  binlog_format=row
  log_bin=/data/mysql/logs/binlog
  ...
  bind-address            = 0.0.0.0 # 允许远程日志过来
  创建日志目录
  mkdir -p /data/mysql/logs
  chown mysql:mysql -R /data/mysql/
  重启服务
  systemctl daemon-reload
  systemctl restart mariadb
  创建登录用户
  CREATE USER 'root'@'10.0.0.%' IDENTIFIED BY '123456';
  GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.0.0.%' WITH GRANT OPTION;
  FLUSH PRIVILEGES;
  
  客户端
  mkdir -p /data/mysql/logs
  chown mysql:mysql -R /data/mysql/
  cd /data/mysql/logs/
  mysqlbinlog -R --host=10.0.0.43 --user=root --password=123456 --raw --stop-never binlog.000002
  ```

  

### 5. **中继日志（Relay Log）**

- **功能**: 从服务器在主从复制中使用的日志，记录主服务器传递过来的二进制日志内容。
- 用途:
  - 用于从服务器执行主服务器传递的操作。
- 配置:
  - 通常自动生成，与复制相关。

### 6. **事务日志（InnoDB Redo Log 和 Undo Log）**

- 功能:

  - **Redo Log**: 支持事务的持久性，记录修改以便在崩溃后恢复。（重做）
  - **Undo Log**: 支持事务的回滚和 MVCC（多版本并发控制）。（回滚）

- 用途:

  - 保证事务的 ACID 特性。
  - 数据一致性和恢复。

- 性能优化：

  - redo log 包含日志缓冲（redo log buffer）和磁盘上的日志文件（redo logfile）两部分。

  - ```powershell
    	MySQL 每执行新一条 DML 语句都会先将日志记录在 redo log buffer 中，然后再根据不同的配置项，使用不同的规则将 redo log buffer 中的数据落盘（写入到 redo log file）。
    	但是，redo log buffer 是用户空间的数据，无法直接写入磁盘，中间必须经过操作系统缓冲区（OS Buffer），因此，从 redo log buffer 到 redo log file，实际上会先写 OS Buffer，再调用fsync() 将其刷人入到 redo log file 中。
    ```

    <img src="5day-png\23事务日志性能" alt="image-20250105122252665" style="zoom:50%;" />

- 落盘规则

  - MySQL 通过 innodb_flush_log_at_trx_commit 配置来控制 redo log 的落盘规则。

  - ```powershell
    0：延迟写，
     	事务提交时会将数据写到 Log Buffer，但不会立即写 OS Buffer 和 Log File，而是每隔 1S，批量的将数据写到 OS Buffer 并调用 fsync() 写磁盘文件。
     	如果系统崩溃则会丢失 1S 的数据 - 因为数据在用户空间的 Log Buffer里面。
    1：实时写，
     	每次提交事务后都会将数据写入 OS Buffer 再写磁盘文件，这种方式几乎不会丢失数据，但性能较差。
    2：实时写，
     	延时落盘，每次提交事务都会将数据写入 OS Buffer，但写磁盘也是 1S 执行一次，这是一个相对折中的方案。
    ```

  - ```sql
    查看
    select @@innodb_flush_log_at_trx_commit;
    ```

    



### 事务日志和二进制日志的区别

```
- 事务日志可以看作是在线日志，二进制日志可以看作是离线日志
- 事务日志记录事务执行的过程，包括提交和未提交，二进制日志记录只记提交的过程
- 事务日志只支持 InnoDB 存储引擎，二进制支持 InnoDB 和 MyISAM 存储引擎
- 事务日志的容量是100M的大小，会滚动覆盖式更新，二进制的容量大小没有限制。
```



## 备份

### **1. 备份类型**

#### **（1）完全备份（Full Backup）**

- 描述：将整个数据库或系统的数据完整复制到备份存储。
- 优点：数据完整，可独立恢复。
- 缺点：耗时长，占用存储空间大。
- 使用场景：首次备份或需要快速完整恢复时。

#### **（2）增量备份（Incremental Backup）**

- 描述：只备份自上次备份（无论是完全还是增量）以来发生更改的数据。
- 优点：节省存储空间和时间。
- 缺点：恢复时需要合并多个增量备份。
- 使用场景：需要频繁备份，且更改量小。

#### **（3）差异备份（Differential Backup）**

- 描述：备份自上次完全备份以来所有更改的数据。
- 优点：恢复比增量备份简单，只需要完全备份和最近一次差异备份。
- 缺点：随着时间推移，备份文件变大。
- 使用场景：变化较少且恢复需求快速。

### **2. 备份方法**

#### **（1）物理备份（Physical Backup）**

- 描述：直接复制数据库文件、日志文件和配置文件。
- 优点：速度快，保留数据结构和文件权限。
- 缺点：依赖于相同的硬件和数据库版本。
- 工具：`rsync`、`tar`、`xtrabackup`。

#### **（2）逻辑备份（Logical Backup）**

- 描述：通过导出 SQL 语句或数据表结构和数据内容进行备份。
- 优点：跨平台兼容，可单独恢复部分数据。
- 缺点：备份和恢复速度慢。
- 工具：`mysqldump`、`mysqlpump`。

#### **（3）热备份（Hot Backup）**

- 描述：在数据库运行状态下进行的备份。
- 优点：不影响服务运行。
- 缺点：可能需要专用工具，复杂度高。
- 工具：`xtrabackup`、`MySQL Enterprise Backup`。

#### **（4）冷备份（Cold Backup）**

- 描述：停止数据库服务后进行的备份。
- 优点：简单、安全。
- 缺点：服务中断。
- 工具：文件级复制。

#### **（5）快照备份（Snapshot Backup）**

- 描述：使用存储设备或文件系统的快照功能记录数据的瞬时状态。
- 优点：快速，可恢复到特定时间点。
- 缺点：依赖于底层存储技术。
- 工具：LVM、ZFS、AWS Snapshots。

------

### **3. 备份内容**

#### **（1）数据文件**

- 数据库表、索引等核心数据存储。

#### **（2）日志文件**

- 二进制日志（Binlog）：用于恢复增量数据。
- 错误日志：记录错误信息，方便排查问题。
- 查询日志：记录所有查询语句。

#### **（3）配置文件**

- 数据库的配置参数（如 `/etc/my.cnf`）。

#### **（4）用户和权限信息**

- 数据库用户和权限分配。

------

### **4. 恢复类型**

#### **（1）完全恢复**

- 恢复整个数据库到备份时的状态。

#### **（2）部分恢复**

- 仅恢复部分数据库或表。

#### **（3）时间点恢复（Point-in-Time Recovery, PITR）**

- 利用二进制日志将数据库恢复到某个特定时间点。

#### **（4）闪回恢复**

- 通过快照或回滚技术将数据恢复到某个历史状态。

------

### **5. 备份策略**

1. **备份频率**：根据业务需求，设置每日、每周或实时备份。
2. **备份保留周期**：如保留最近 7 天或 1 个月的备份。
3. **异地备份**：将备份存储在异地以防灾难。
4. **定期测试**：验证备份的完整性和可恢复性。
5. **自动化**：使用脚本或工具（如 `cron`、`Ansible`）实现定时备份。

------

### **6. 备份工具**

#### **MySQL 专用工具**

- `mysqldump`：逻辑备份工具。
- `mysqlpump`：更快的逻辑备份工具。
- `xtrabackup`：物理热备工具。
- `MySQL Enterprise Backup`：商业版备份工具。

#### **通用备份工具**

- `rsync`：同步文件。
- `tar`：归档文件。
- `LVM`：快照技术。



### 备份工具

- `cp`、`tar`等复制归档工具，物理备份工具，适用于所有存储引擎，只支持冷备，完全和部分备份
- `LVM的快照`：先加读锁，做完快照后解锁，几乎热备；借助文件系统工具进行备份
- `mysqldump`：逻辑备份工具，适用于所有存储引擎，对MyISAM存储引擎进行温备；支持完全或部分备份；对InnoDB支持热备，结合binlog增量备份
- `xtrabackup`：由Percona提供支持对InnoDB做热备的工具，支持完全备份，增量备份
- `mysqlbackup`：热备份，MySQL Enterprise Edition组件

### 冷备和还原的实现

```powershell

# 先暂停服务
systemctl stop mysqld

#创建备份目录
mkdir /data/backup -p
cd /data/backup/

#备份打包数据
tar zcf base_data.tar.gz /var/lib/mysql
tar zcf binlog_data.tar.gz /data/mysql/logs

# 在远程主机上还原
# 暂停服务
systemctl stop mysqld

# 将原来的/var/lib/mysql/*全部清空
rm -rf /var/lib/mysql/*
rm -rf /data/mysql/logs/*

#从远端主机拉取解压数据
rsync root@10.0.0.43:/data/backup/*.gz ./
tar xf base_data.tar.gz
tar xf binlog_data.tar.gz

# 将之前备份的移动过去
mv ./var/lib/mysql/* /var/lib/mysql/
mv ./data/mysql/logs/* /data/mysql/logs/

# 更改权限
chown mysql:mysql -R /data/mysql/*
chown mysql:mysql -R /var/lib/mysql

# 重启服务
systemctl start mysqld
```

### mysqldump 备份工具

常用的逻辑备份工具包括`mysqldump`、`mysqldumper`、`phpMyAdmin`等

mysqldump是MySQL官方提供的客户端备份工具，通过mysql协议连接至mysql服务器进行备份，mysqldump命令是将数据库中的数据备份成一个文本文件，数据表的结构和数据都存储在文本文件中

- InnoDB建议备份策略

```powershell
#旧版命令
mysqldump -uroot -p123456 -A -F -E -R --triggers --single-transaction --master-data=2 --flush-privileges --default-character-set=utf8 --hex-blob > ${BACKUP}/fullbak_{BACK_TIME}.sql

# 新版8.0.26以上
mysqldump -uroot -p123456 -A -F -E -R --triggers --single-transaction --source-data=2 --flush-privileges --default-character-set=utf8 --hex-blob > ${BACKUP}/fullbak_{BACK_TIME}.sql

# 选项说明
-uroot               # MySQL用户名
-p123456             # MySQL密码
-A                   # 备份所有数据库
-F                   # 备份前先刷新日志
-E                   # 导出事件
-R                   # 导出存储过程和自定义函数
--triggers           # 导出触发器
--single-transaction # 以开启事务的方式备份数据
--source-data=2      # 备份日志
--flush-privileges   # 导出后刷新权限
--default-character-set=utf8  # 设置字符集
--hex-blob           # 使用十六进制转储二进制
```

- MyISAM建议备份策略

```powershell
mysqldump -uroot -p123456 -A -E -F -R --source-data=1 --flush-privileges --triggers --default-character-set=utf8 --hex-blob > ${BACKUP}/fullbak_{BACK_TIME}.sql
```

mysqldump备份和还原试验

#### 备份指定数据库里面的数据

```powershell
备份指定数据库
mysqldump db1 > /data/backup/db1-bak.sql
mysqldump -u root  -p123456 -h10.0.0.43 db1 > /data/backup/db1-bak.sql
还原数据库
mysql db1 < /data/backup/db1-bak.sql
还原数据库数据的时候，数据库名可以和原来的数据库不同名
mysql testdb2 < /data/backup/db1-bak.sql

在备份服务器还原，先创建数据库，然后将sql文件导入即可
create database db1;
source /data/backup/db1-bak.sql
```

#### 备份数据库结构和数据

```powershell
备份数据库结构
mysqldump -B db1 > /data/backup/db1-bak-stru.sql
mysqldump -p123456 -h10.0.0.43 -B db1 > /data/backup/db1-bak-stru.sql
还原数据库
mysql < /data/backup/db1-bak-stru.sql

还原数据库
source /data/backup/db1-bak-stru.sql
```

#### 备份所有数据库

```powershell
直接备份所有数据库
mysqldump -A > /data/backup/all-bak.sql
mysqldump -uroot -h 10.0.0.43 -p123456 -A > /data/backup/all-bak.sql
```

#### 数据库备份脚本

```shell
#!/bin/bash
UNAME=root
PWD=123456
HOST=10.0.0.43
IGNORE='Database|information_schema|performance_schema|sys'
YMD=`date +%F`
BACKUP_DIR='/data/backup'
if [ ! -d ${BACKUP_DIR}/${YMD} ];then
  mkdir -pv ${BACKUP_DIR}/${YMD}
fi
DBLIST=`mysql -e "show databases;" 2>/dev/null | grep -Ewv "$IGNORE"`
for db in ${DBLIST};do
  mysqldump -B $db 2>/dev/null 1>${BACKUP_DIR}/${YMD}/${db}_${YMD}.sql
  if [ $? -eq 0 ];then
    echo "${db}_${YMD} backup success"
  else
    echo "${db}_${YMD} backup fail"
  fi
done
```

#### mysqldump保存日志扩展信息

```powershell
在备份日志中记录备份时的二进制日志信息，后续通过此备份进行恢复，还是会缺少一部份数据，这一部份数据，则可以通过当前的二进制日志与备份文件中的二进制信息进行对比得到。
功能解决
很久以前可以通过：--source-data=1|2 属性来实现，额外属性的获取。在目前的mysql8和mariadb10版本里面需要用到的是：
 	--single-transaction --master-data=2 选项
	--master-data 选项用于在备份文件中包含二进制日志文件名和位置信息。这对于将来进行时间点恢复至关重要，因为它允许你指定恢复到特定的时间点。

--master-data=0：不记录二进制日志信息。
--master-data=1：在备份文件中包含 CHANGE MASTER TO 语句，用于指定二进制日志的位置。
--master-data=2：与 --master-data=1 类似，但 CHANGE MASTER TO 语句会被注释掉。这通常用于安全性更高的场景，因为注释掉的语句不会被直接执行。

注意：
	--master-data 选项不写，等于 --master-data=0
 	--master-data 选项，等于 --master-data=1
 	

mysqldump -B db1 > /data/backup/db1.sql
mysqldump -B db1 --single-transaction --master-data > /data/backup/db1-bak.sql
mysqldump -B db1 --single-transaction --master-data=1 > /data/backup/db1-bak1.sql
mysqldump -B db1 --single-transaction --master-data=2 > /data/backup/db1-bak2.sql
mysqldump -B db1 --single-transaction --master-data=0 > /data/backup/db1-bak3.sql
```



### xtrabackup备份工具

```
安装文档地址：https://docs.percona.com/percona-xtrabackup/8.0/yum-download-rpm.html
```

1. 在线热备份：Xtrabackup支持在线热备份，即在备份过程中不会影响数据库的读写操作，这对于生 产环境中的数据库尤为重要。 

2. 备份速度快：通过直接复制物理文件的方式来备份和恢复数据，这种方式速度非常快。 
2. 对数据库性能影响小：在备份过程中，对数据库的性能影响较小，不会增加太多的性能压力。 
2. 支持自动校验：Xtrabackup支持对备份的数据进行自动校验，确保备份数据的完整性。
2. 支持多种存储引擎：可以备份InnoDB、XtraDB、MyISAM等多种存储引擎的表。但需要注意的 是，对于MyISAM存储引擎的表，在备份时需要加读锁。
2. 支持增量和差异备份：Xtrabackup支持增量备份和差异备份，只备份自上次备份以来发生变化的数 据，这大大减少了备份时间和备份文件的大小。 
2. 版本兼容性强：Xtrabackup与不同版本的MySQL数据库具有良好的兼容性，可以在不同版本的 MySQL数据库之间进行数据迁移。 
2. 基于InnoDB crash recovery机制：在备份还原时利用redo log得到完整的数据文件，并通过全局 读锁保证InnoDB数据与非InnoDB数据的一致性。

```powershell
rocky
安装软件源
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
启用源仓库
percona-release enable pxb-80
安装软件
yum install percona-xtrabackup-80
rpm -ql percona-xtrabackup-80 | grep bin
ubuntu
mkdir /data/softs -p
cd /data/softs/
wget https://downloads.percona.com/downloads/Percona-XtraBackup-8.0/Percona-XtraBackup-8.0.35-31/binary/debian/noble/x86_64/percona-xtrabackup-80_8.0.35-31-1.noble_amd64.deb

apt install ./percona-xtrabackup-80_8.0.35-31-1.noble_amd64.deb
dpkg -L percona-xtrabackup-80 | grep bin
```

```powershell
xtrabackup 实现完全备份和还原
	xtrabackup --backup --target-dir=/backup/base
xtrabackup 实现增量备份和还原
 	xtrabackup --backup --target-dir=/backup/base
 	xtrabackup --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/base
xtrabackup 实现单表备份和还原
 	xtrabackup -uroot -p123456 --backup --target-dir=/backup/db1_stu --databases='db1' --tables='stu'
```

```powershell
cat /backup/base/xtrabackup_info
uuid = 15671bf9-c127-11ef-bb59-0050563bd0e3
name =
tool_name = xtrabackup 						#备份工具
tool_command = --backup --target-dir=/backup/base
tool_version = 8.0.35-31
ibbackup_version = 8.0.35-31 				#备份工具版本
server_version = 8.0.40-0ubuntu0.24.04.1 	#MySQL版本
start_time = 2024-12-23 20:11:46 			#开始备份时间
end_time = 2024-12-23 20:11:47 				#结束时间
lock_time = 0 								#锁表时长
binlog_pos = filename 'binlog.000004', position '157' 	#binlog文件和pos位置信息
innodb_from_lsn = 0 						#开始lsn号
innodb_to_lsn = 20556483 					#结束lsn号
partial = N 								#不是部份备份
incremental = N 							#不是增量备份
format = file 								#以文件形式备份
compressed = N 								#不是压缩格式
encrypted = N 								#不加密

cat /backup/base/xtrabackup_checkpoints
backup_type = full-backuped 				#完全备份
from_lsn = 0 								#备份开始lsn号
to_lsn = 20556483 							#备份结束lsn号
last_lsn = 20556483 						#备份结束时最大lsn号
flushed_lsn = 20556483
redo_memory = 0
redo_frames = 0
```

#### xtrabackup备份和还原实现

```powershell
#42:
mkdir -p /backup/base
xtrabackup --backup --target-dir=/backup/base
scp -r /backup root@10.0.0.44:/data/backup/
#44:
mkdir /data/backup/
cd backup/
du -sh /data/backup/backup/base
xtrabackup --prepare --target-dir=/data/backup/backup/base
du -sh /data/backup/backup/base
systemctl stop mysql.service
rm -rf /var/lib/mysql/*
ll /var/lib/mysql
xtrabackup --copy-back --target-dir=/data/backup/backup/base --datadir=/var/lib/mysql
chown -R mysql:mysql /var/lib/mysql/*
systemctl start mysql.service
```

#### xtrabackup 实现增量备份和还原

```powershell
#42
xtrabackup --backup --target-dir=/backup/base
du -sh /backup/*
72M     /backup/base
insert into db1.student(name,age,gender)values('u77',77,'M');
xtrabackup --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/base
du -sh /backup/*
72M     /backup/base
2.0M    /backup/inc1 		# 增量的数据

#44
清理旧有数据
rm -rf /data/backup/*
cd /data/backup/
从10.0.0.41 主机获取数据到本地
scp -r root@10.0.0.13:/backup/* ./
du -sh ./*
84M	./base
2.0M	./inc1
2.0M	./inc2
整理全量备份数据，不回滚
xtrabackup --prepare --apply-log-only --target-dir=/data/backup/base
du -sh /data/backup/*
104M	/data/backup/base		# 占用空间变大
2.0M	/data/backup/inc1
2.0M	/data/backup/inc2
整理第一次增量备份数据，不回滚
xtrabackup --prepare --apply-log-only --target-dir=/data/backup/base --incremental-dir=/data/backup/inc1
du -sh /data/backup/*
104M	/data/backup/base
10M	/data/backup/inc1			# 占用空间变大
2.0M	/data/backup/inc2
整理第二次增量备份数据，需要回滚
xtrabackup --prepare --target-dir=/data/backup/base --incremental-dir=/data/backup/inc2
du -sh /data/backup/*
104M	/data/backup/base
10M	/data/backup/inc1
34M	/data/backup/inc2			# 占用空间变大

数据还原操作
systemctl stop mysql.service
rm -rf /var/lib/mysql/*
开始还原数据
xtrabackup --copy-back --target-dir=/data/backup/base --datadir=/var/lib/mysql
开始启动mysql
chown -R mysql:mysql /var/lib/mysql/*
systemctl start mysql.service
```



## mysql集群

### 横向扩展和纵向扩展

**横向扩展（Horizontal Scaling）**和**纵向扩展（Vertical Scaling）**是系统架构中提升性能和处理能力的两种主要方法。以下是两者的定义、优缺点以及应用场景的对比分析。

------

#### **1. 横向扩展（Horizontal Scaling）**

##### **定义**

通过增加更多的独立服务器或节点来提升系统的性能和处理能力。例如，在负载均衡器后面添加更多的服务器来分担请求。

##### **特点**

- 每个新节点通常是一个独立的服务器。
- 使用集群或分布式系统技术来协调节点间的工作。
- 需要配置负载均衡器（如 Nginx、HAProxy）来分发流量。

##### **优点**

- **高可用性**：某些节点故障不会导致系统整体宕机，支持故障转移。
- **灵活性**：可以动态添加或移除节点以适应流量变化。
- **更高的性能上限**：理论上可通过增加节点无限扩展性能。
- **分布式存储**：适合大数据存储和分布式数据库，如 Cassandra、MongoDB。

##### **缺点**

- **复杂性高**：需要管理多个节点和分布式系统架构。
- **一致性问题**：数据分布在多个节点之间，可能面临一致性挑战（如 CAP 原则）。
- **延迟增加**：节点间的通信可能带来额外的延迟。

##### **应用场景**

- 网站流量剧增时扩容（如电商网站促销期间）。
- 分布式数据库或缓存（如 Redis 集群、分布式文件系统）。
- 云计算中的弹性扩展。

------

#### **2. 纵向扩展（Vertical Scaling）**

##### **定义**

通过增加单个服务器的硬件资源（如 CPU、内存、存储）来提升系统的性能和处理能力。例如，更换为更高配置的服务器或虚拟机。

##### **特点**

- 不改变系统架构，提升现有服务器的性能。
- 依赖更强大的硬件和高效的软件优化。

##### **优点**

- **简单易管理**：无需增加节点，管理成本较低。
- **无数据一致性问题**：所有数据在同一节点上。
- **性能立即可见**：增加资源后性能提升明显。

##### **缺点**

- **成本增加**：高性能硬件价格昂贵。
- **有性能瓶颈**：单个服务器的性能有上限。
- **单点故障风险**：如果服务器宕机，系统将不可用。

##### **应用场景**

- 中小型网站或业务，对性能要求较高但流量稳定。
- 数据库优化，如将内存扩展以容纳更多缓存。
- 应用层单点性能优化。

------

#### **3. 对比分析**

| 特性             | 横向扩展                     | 纵向扩展                 |
| ---------------- | ---------------------------- | ------------------------ |
| **实现方式**     | 增加更多节点                 | 增加现有节点的硬件资源   |
| **适用规模**     | 大规模系统，分布式场景       | 中小型系统，单机优化场景 |
| **可用性**       | 高，支持冗余和故障转移       | 低，单点故障风险         |
| **复杂性**       | 高，需要分布式架构和协调机制 | 低，硬件升级即可         |
| **成本**         | 可控，按需扩展               | 硬件成本较高，可能不划算 |
| **性能扩展上限** | 理论上无限                   | 有硬件瓶颈               |
| **一致性**       | 难以保证，需要协调           | 一致性强，无需额外协调   |

------

#### **4. 选择策略**

##### **优先使用横向扩展**

- 系统需要支持大规模用户和高并发（如电商平台、社交媒体）。
- 需要高可用性和容错能力。
- 能够接受分布式系统带来的管理复杂性。

##### **考虑纵向扩展**

- 流量稳定，中小型系统。
- 优化单机性能的瓶颈（如 CPU 密集型应用、数据库性能）。
- 短期解决性能问题，不需要复杂的架构变更。

------

#### **5. 实际场景例子**

- ##### **横向扩展**：

  - 使用 Kubernetes 增加 Pod 数量以应对流量激增。
  - 数据存储采用分布式数据库，如 Cassandra 或 HDFS。

- ##### **纵向扩展**：

  - 为 MySQL 数据库服务器增加内存和 CPU。
  - 更换为更高配置的云实例（如从 AWS EC2 t2.micro 升级到 t2.large）。

选择哪种扩展方法应基于具体业务需求、流量规模和预算限制。



### 主从复制

主从复制的优点

```
- 负载均衡读操作：将读操作进行分流，由另外一台或多台服务器提供查询服务，降低 MySQL 负载，提升响应速度
- 数据库备份：主从节点上都有相同的数据集，从而也实现了数据库的备份
- 高可用和故障切换：主从架构由两个或多个服务节点构成，在某个节点不可用的情况下，可以进行转移和切换，保证服务可用
- MySQL升级：当 MySQL 服务需要升级时，由于主从架构中有多个节点，可以逐一升级，而不停止服务
```

主从复制的缺点

```
- 数据延时：主节点上的数据需要经过复制之后才能同步到从节点上，这个过程需要时间，从而会造成主从节点之间的数据不一致
- 性能消耗：主从复制需要开启单独线程，会消耗一定资源
- 数据不对齐：如果主从复制服务终止，而且又没有第一时间恢复，则从节点的数据一直无法更新
```

#### 三个线程，两个日志

| 线程               | 节点   | 作用                                                  |
| ------------------ | ------ | ----------------------------------------------------- |
| binlog dump thread | Master | 为 Slave 节点的 I/O thread 提供本机的 binlog 中的数据 |
| I/O thread         | Slave  | 从 Master 节点获取数据，并存放于本地的 relay log 中   |
| SQL thread         | Slave  | 将本地的 relay log 中的数据写到数据文件中，完成重放   |

| 日志      | 节点   | 作用                                             |
| --------- | ------ | ------------------------------------------------ |
| binlog    | Master | slave 节点的数据源                               |
| relay log | Slave  | 中继日志，从 master 节点中同步过来的数据暂存于此 |

<img src="5day-png\23主从复制.png" alt="image-20250105200625478" style="zoom: 50%;" />

相关文件

```powershell
master.info 			#用于保存slave连接至master时的相关信息，例如账号、密码、服务器地址等
relay-log.info 			#保存在当前slave节点上已经复制的当前二进制日志和本地relay log日志的对应关系
mysql-relay-bin.00000N 	#中继日志,保存从主节点复制过来的二进制日志,本质就是二进制日志
```

```powershell
主节点
修改主节点配置文件
[mysqld]
log_bin=/data/logbin/mysql-bin #启用二进制日志 
server-id=N #为当前节点设置全局唯一ID
查看从二进制日志的文件和位置开始进行复制
show master status;
创建有复制权限的用户账号
GRANT REPLICATION SLAVE ON *.* TO 'repluser'@'HOST' IDENTIFIED BY 'replpass';
MySQL8.0 分成两步实现
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';

从节点
修改配置文件
[mysqld]
server_id=N 		#为当前节点设置一个全局惟的ID号
log-bin 			#开启从节点二进制日志
read_only=ON 		#设置数据库只读，针对supper user无效
relay_log=relay-log #relay log的文件路径，默认值hostname-relay-bin
relay_log_index=relay-log.index 	#默认值hostname-relay-bin.index

使用有复制权限的用户账号连接至主服务器
在从节点上执行下列SQL语，提供主节点地址和连接账号，用户名，密码，开始同步的二进制文件和位置等。
CHANGE MASTER TO MASTER_HOST='masterhost', 	#指定master节点
MASTER_USER='repluser', 					#连接用户
MASTER_PASSWORD='replpass', 				#连接密码
MASTER_LOG_FILE='mariadb-bin.xxxxxx', 		#从哪个二进制文件开始复制
MASTER_LOG_POS=123, 						#指定同步开始的位置
MASTER_DELAY = interval; 	#可指定延迟复制实现防止误操作,单位秒，这里可以用作延时同步，一般用于备份

START SLAVE [IO_THREAD|SQL_THREAD]; 		#启动同步线程
STOP SLAVE 					#停止同步
RESET SLAVE ALL 			#清除同步信息
SHOW SLAVE STATUS;			#查看从节点状态
SHOW RELAYLOG EVENTS in 'relay-bin.00000x'; 	#查看relaylog 事件
							#可以利用 MASTER_DELAY 参数设置从节点延时同步，用作主从备份，比如设置一小时的延时，则主节点上的误操作，要一小时后才会同步到从服务器，可以利用时间差保存从节点数据
```

```powershell
ubuntu中安装mysql做主从，在配置文件中添加
vim /etc/apparmor.d/usr.sbin.mysqld
/usr/sbin/mysqld {
...
# Allow data files dir access
  /data/mysql/logbin/ rw,
  /data/mysql/logbin/** rw,
...
}
systemctl reload apparmor
```



### 一主一从

<img src="5day-png\23一主一从.png" alt="image-20250105200927849" style="zoom: 50%;" />

```powershell
#master:11
yum install mysql-server -y
mkdir -pv /data/mysql/logbin
chown -R mysql:mysql /data/mysql/
vim /etc/my.cnf.d/mysql-server.cnf
server-id               = 101
log_bin                 = /data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
systemctl restart mysqld
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
mysql> flush privileges;
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       849 | No        |
+------------------+-----------+-----------+
1 row in set (0.00 sec)
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      849 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)


#slave:12
yum install mysql-server -y
mkdir -pv /data/mysql/logbin
chown -R mysql:mysql /data/mysql/
vim /etc/my.cnf.d/mysql-server.cnf
server-id               = 102
read-only
log_bin                 = /data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
systemctl restart mysql
mysql> CHANGE MASTER TO MASTER_HOST='10.0.0.10', MASTER_USER='repluser', MASTER_PASSWORD='123456', MASTER_PORT=3306,MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=849;
start slave;
show slave status\G
```

```powershell
#清理master


#清理slave
mysql> stop slave;
mysql> reset slave all;

```

```powershell
#有数据一主一从
从环境清空环境
systemctl stop mysqld
rm -rf /var/lib/mysql/*
rm -rf /data/mysql/logbin/*
systemctl start mysqld

#master
mysql> reset master;
mysql> show slave hosts;
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      157 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
mysqldump -A -F --source-data=1 --single-transaction >all.sql
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
scp all.sql 10.0.0.12:

#slave
vim all.sql
...
# CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=157; 
# 将此句补全为如下样式
CHANGE MASTER TO 
 MASTER_HOST='10.0.0.11',
 MASTER_USER='repluser',
 MASTER_PASSWORD='123456',
 MASTER_PORT=3306,
 MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=157;
...
mysql> set sql_log_bin=0;
mysql> source /root/all.sql
mysql> start slave;
mysql> show slave status\G
mysql> set @@sql_log_bin=1;
```



### 一主多从

<img src="5day-png\23一主多从.png" alt="image-20250105201018212" style="zoom:50%;" />

```powershell
在一主一从基础上做
#master:11
mysqldump -A --source-data=1 --single-transaction >all.sql
scp all.sql 10.0.0.13:

#slave1:12
#slave2:13
yum install mysql-server -y
mkdir -pv /data/mysql/logbin
chown -R mysql.mysql /data/mysql/
vim /etc/my.cnf.d/mysql-server.cnf
server-id               = 103
read-only
log_bin                 = /data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
systemctl restart mysqld
vim all.sql
# CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=453; #将此内容修改为如下所示
CHANGE MASTER TO 
 MASTER_HOST='10.0.0.11',
 MASTER_USER='repluser',
 MASTER_PASSWORD='123456',
 MASTER_PORT=3306,
 MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=453;
systemctl start mysqld.service 
mysql> set sql_log_bin=0;
mysql> source /root/all.sql;
mysql> set sql_log_bin=1;
mysql> start slave;
mysql> show slave status\G
```

### 级联复制 

<img src="5day-png\23级联复制.png" alt="image-20250105201106791" style="zoom:50%;" />

```powershell
#master:11

#slave1:12
#mysql8.0以后默认开启了此项，其它版本或 mariadb 需要手动开启，
#中间节点要保证: 1 开启二进制日志， 2 从主节点上同步过来的数据，要能写到二进制日志中
mysql> select @@log_slave_updates;
vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server-id=102
read-only
log_slave_updates 			# 增加此选项
log-bin=/data/mysql/logbin/mysql-bin
systemctl restart mysqld.service
mysqldump -A -F --single-transaction --source-data=1 > middle-all.sql
scp middle-all.sql 10.0.0.14:
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
mysql> flush privileges;

#slave1-1:14
如果14之前做过主从，要清理数据
mysql> stop slave;
mysql> reset slave all;
systemctl stop mysqld
rm -rf /var/lib/mysql/*
rm -rf /data/mysql/logbin/*
systemctl start mysqld.service
如果是新环境从这里开始
yum install mysql-server -y
mkdir -pv /data/mysql/logbin
chown -R mysql.mysql /data/mysql/
vim /etc/my.cnf.d/mysql-server.cnf
server-id               = 103
read-only
log_bin                 = /data/mysql/logbin/mysql-bin
vim middle-all.sql
# CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=157;
#修改上面语句为
CHANGE MASTER TO 
 MASTER_HOST='10.0.0.12',
 MASTER_USER='repluser',
 MASTER_PASSWORD='123456',
 MASTER_PORT=3306,
 MASTER_LOG_FILE='mysql-bin.000004', 
 MASTER_LOG_POS=157;
systemctl restart mysqld.service
mysql> set sql_log_bin=0;
mysql> source /root/middle-all.sql
mysql> set sql_log_bin=1;
mysql> start slave;
mysql> show slave status\G

```

### 双主架构互为主从

<img src="5day-png\23双主构架互为主从.png" alt="image-20250105201339142" style="zoom:50%;" />

```powershell
根据前面的实验，master-2 己经是 master-1 的从节点了，此处只需要把 master-1 配置为 master-2 的从节点即可。
master1:11
cat /etc/my.cnf.d/mysql-server.cnf
[mysqld]
...
server-id = 101
log_bin = /data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
...
systemctl restart mysqld
master2:12
cat /etc/my.cnf.d/mysql-server.cnf
[mysqld]
...
server-id               = 102
# read-only								# 停止只读服务
log_bin                 = /data/mysql/logbin/mysql-bin
# log_slave_updates						# 停止中间服务器的能力
default_authentication_plugin=mysql_native_password
...
systemctl restart mysqld
现在master2是master1的从
配置master1为master2的从
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
mysql> flush privileges;
mysql> show master logs;
master1:11
mysql> show slave status;
mysql> CHANGE MASTER TO 
      MASTER_HOST='10.0.0.12',
      MASTER_USER='repluser',
      MASTER_PASSWORD='123456',
      MASTER_PORT=3306,
      MASTER_LOG_FILE='mysql-bin.000007', 
      MASTER_LOG_POS=157;
mysql> start slave;
mysql> show slave status\G
```

```powershell
在双主架构中，如果同时在两个节点执行写操作，就可能会导致数据冲突，从而影响主从同步。
尝试在两个主机同时写入数据
mysql> insert into student(name,age,gender) values('user8', 80,'F');
mysql> show slave status\G
	这个报错
	Last_SQL_Error: Coordinator stopped because there were error(s) in the worker(s). The most recent failure being: Worker 1 failed executing transaction 'ANONYMOUS' at source log mysql-bin.000007, end_log_pos 1340. See error log and/or performance_schema.replication_applier_status_by_worker table for more details about this failure or others, if any. 
在主从同步失效的情况下，在master-1上的写操作，无法同步到 master-2

解决办法
忽略错误事件
#master1上执行
#停止主从
mysql> stop slave;
#跳过1个错误事件
mysql> set global sql_slave_skip_counter=1;
#再次开启主从同步
mysql> start slave;
#查看同步状态，己经恢复
mysql> show slave status\G

master2同样执行
```



### 多主一从

<img src="5day-png\23多主一从.png" alt="image-20250105201211989" style="zoom:50%;" />



### 环状复制，使用较少

<img src="5day-png\23环形复制.png" alt="image-20250105201421457" style="zoom:50%;" />



### 半同步复制原理解读

<img src="5day-png\23半同步复制.png" alt="image-20250107123252528" style="zoom: 33%;" />

```powershell
异步复制
#默认情况下，MySQL 中的复制是异步的，当客户端程序向主节点中写入数据后，主节点中数据落盘，写入binlog日志，然后将binlog日志中的新事件发送给从节点 ，便向客户端返回写入成功，而并不验证从节点是否接收完毕，也不等待从节点返回同步状态，这意味着客户端只能确认向主节点的写入是成功的，并不能保证刚写入的数据成功同步到了从节点。
#此复制策略下，如果主从同步出现故障，则有可能出现主从节点之间数据不一致的问题。甚至，如果在主节点写入数据后没有完成同步，主节点服务当机，则会造成数据丢失。
#异步复制不要求数据能成功同步到从节点，只要主节点完成写操作，便立即向客户端返回结果。

同步复制
#当客户端程序向主节点中写入数据后，主节点中数据落盘，写入binlog日志，然后将binlog日志中的新事件发送给从节点 ，等待所有从节点向主节点返回同步成功之后，主节点才会向客户端返回写入成功。
#此复制策略能最大限度的保证数据安全和主从节点之间的数据一致性，但此复制策略性能不高，需要在所有从节点上完成数据同步之后，客户端才能获得返回结果。
#此同步策略又称为全同步复制。

半同步复制
#当客户端程序向主节点中写入数据后，主节点中数据落盘，写入binlog日志，然后将binlog日志中的新事件发送给从节点 ，等待所有从节点中有一个从节点返回同步成功之后，主节点就向客户端返回写入成功。
#此复制策略尽可能保证至少有一个从节点中有同步到数据，也能尽早的向客户端返回写入状态。
#但此复制策略并不能百分百保证数据有成功的同步至从节点，因为可以在此策略下设至同步超时时间，如果超过等待时间，即使没有任何一个从节点返回同步成功的状态，主节点也会向客户端返回写入成功。
#“Waiting Slave dump”意味着主库的dump线程在发送完binlog后，正在等待从库的IO线程确认已经接收到这些日志。
```

#### MySQL5.7 之前的半同步复制默认策略 - 先commit之后，再等待slave的返回状态数据

```shell
rpl_semi_sync_master_wait_point=after_commit
```

<img src="5day-png\23mysql5.7之前半同步复制默认策略.png" alt="image-20250107124313095" style="zoom: 33%;" />

​	waiting Slave dump表示，等待slave返回成功的数据信息，如果返回的话，给客户端返回成功。如 果没有等后，等待10s后，变成异步模式，给客户端返回成功。 

​	特点在于，master节点自身能够保证数据是落地的 -- 也就是存储成功的。

在此半同步策略配置中，可能会出现下列问题：

- 幻读：当客户端提交一个事务，该事务己经写入 redo log 和 binlog，但该事务还没有写入从节 点，此时处在 Waiting Slave dump 处，此时另一个用户可以读取到这条数据，而他自己却不能。

  ```
  第一个客户端用户，过来添加了一条数据，主键为123
   	因为从节点异常导致，没有主上面的数据，
  第二个客户端用户，过来添加了一条数据，主键为123.
   	因为主节点上的123数据，已经存在了，所以 第二个客户端数据，操作会失败。
  ```

- 数据丢失：一个提交的事务在 Waiting Slave dump 处 crash后，主库将比从库多一条数据

  ```
  因为master节点数据已经落地了，但是因为slave节点异常，导致它比主数据库信息少了一条数据。
  ```

#### MySQL5.7 以及之后的半同步复制 - 先等待slave的返回状态数据，之后再commit。

```shell
rpl_semi_sync_master_wait_point=after_sync
```

<img src="5day-png\23MySQL5.7 以及之后的半同步复制.png" alt="image-20250107124700855" style="zoom:33%;" />

在此半同步策略配置中，客户端的写操作先不提交事务，而是先写二进制日志，然后向从库从步数据， 由于在主节点上的事务还没提交，所以此时其它进程查不到当前的写操作，不会出现幻读的问题，而且 主节点要确认至少有一个从节点的数据同步成功了，再会提交事务，这样也保证了主从之间的数据一致 性，不会存在丢失数据的情况。

#### mysql8.0半同步实践

```
master：11
slave-1：12
slave-2：13
```

```powershell
#master
#在 master节点上安装 master 插件
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
mysql> show plugins;
mysql> select * from mysql.plugin;
#安装完成后还需要手动开启插件
mysql> select @@rpl_semi_sync_master_enabled;
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server-id=101
log_bin=/data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
rpl_semi_sync_master_enabled 			# 增加功能，注意是 master关键字
systemctl restart mysqld.service

#slave
#在 slave 节点上安装 slave 插件
mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
mysql> show plugins;
vim /etc/my.cnf.d/mysql-server.cnf
......
[mysqld]
server-id=183
read-only
log-bin=/data/mysql/logbin/mysql-bin
rpl_semi_sync_slave_enabled # 增加该条属性，注意是 slave关键字
# log_slave_updates
default_authentication_plugin=mysql_native_password
systemctl restart mysqld;
#再次查看
mysql> select @@rpl_semi_sync_slave_enabled;

#master
mysql> show global variables like '%semi%';
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | ON         |#己开启	
| rpl_semi_sync_master_timeout              | 10000      |#超时时长,1W毫秒，10S
| rpl_semi_sync_master_trace_level          | 32         |
| rpl_semi_sync_master_wait_for_slave_count | 1          |#指定几台slave节点收到binlog后就给客户端返回
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |#当前同步策略
+-------------------------------------------+------------+



mysql>  show global status like '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 2     |#2个slave节点
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+
```

修改同步超时时长

```powershell
vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server-id=101
log_bin=/data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
rpl_semi_sync_master_enabled
rpl_semi_sync_master_timeout=3000 		# 设定时间为3秒
systemctl restart mysqld
mysql> show variables like '%semi_sync_master_timeout%';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| rpl_semi_sync_master_timeout | 3000  |
+------------------------------+-------+

```



### 复制过滤器

复制过滤器是指让从节点仅复制指定的数据库，或指定数据库的指定表。

复制过滤器的实现有两种方式：

```
- 在 master 节点上使用服务器选项配置来实现：在 master 节点上配置仅向二进制日志中写入与特定数据库相关的事件。
- 在 slave 节点上使用服务器选项或者是全局变量配置来实现：在 slave 节点上配置在读取 relay log 时仅处理指定的数据库或表。
```

#### 在 master 节点上配置实现

```
优点：只需要在 master 节点上配置一次即可，不需要在 salve 节点上操作；减小了二进制日志中的数据量，能减少磁盘IO和网络IO。
缺点：二进制日志中记录的数据不完整，如果当前节点出现故障，将无法使用二进制还原。
```

```powershell
binlog-do-db=db1 		#数据库白名单列表，不支持同时指定多个值，如果想实现多个数据库需多行实现
binlog-do-db=db2
binlog-ignore-db=db3 	#数据库黑名单列表，不支持同时指定多个值，如果想实现多个数据库需多行实现
binlog-ignore-db=db4 

#binlog-do-db,binlog-ignore-db 都是服务器选项
#配置白名单的方式表示仅往二进制日志中写指定的库
#配置黑名单的方式表示除了黑名单中的库或表，都写二进志日志
```

```powershell
master：11
slave：12
```

master 节点上配置db1,db2 不写二进制日志

```powershell
#master 节点上配置db1,db2 不写二进制日志
[root@rocky9-12 ~]# vim /etc/my.cnf.d/mysql-server.cnf
......
[mysqld]
server-id=101
log_bin=/data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
rpl_semi_sync_master_enabled
rpl_semi_sync_master_timeout=3000
binlog-ignore-db=db1 # 下面的两条
binlog-ignore-db=db2
...
systemctl restart mysqld
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000010 |      157 |              | db1,db2          |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

#### 在slave上配置实现部分同步

```
优点：master 节点中的所有数据都是全量写二进制日志，可以使用二进制日志恢复数据。
缺点：对于 slave 节点不需要保存的数据，也会通过网络传输过来，这样会对磁盘IO和网络IO都有影响，而且，在有多个 slave节点时，每一个 slave 节点都需要配置。
```

```
replicate-do-db=db1 					#指定复制库的白名单，每行指定一个库，多个库写多行
replicate-do-table=tab1 				#指定复制表的白名单，每行指定一个表，多个表写多行
replicate-ignore-db=db1 				#指定复制表的黑名单，每行指定一个库，多个库写多行
replicate-ignore-table=tab1 			#指定复制表的黑名单，每行指定一个表，多个表写多行
replicate-wild-do-table=foo%.bar% 		#指定复制表的白名单，支持通配符，每行指定一个规则，多个规则写多行 
replicate-wild-ignore-table=foo%.bar%	#指定复制表的黑名单，支持通配符，每行指定一个规则，多个规则写多行
```

```
master：11
slave：12
```

```powershell
#先重置环境，在slave 节点上停止主从，删除db1,db3
mysql> stop slave;
mysql> reset slave all;
mysql> drop database db1;
mysql> drop database db2;
mysql> drop database db3;
mysql> drop database db4;
#master节点上删除db1,db2,db3
mysql> drop database db1;
mysql> drop database db2;
mysql> drop database db3;
mysql> drop database db4;
#master主机取消过滤数据库的配置后，重启主机
vim /etc/my.cnf.d/mysql-server.cnf
#binlog-ignore-db=db1
#binlog-ignore-db=db2
systemctl restart mysqld;

#slave-1节点上配置
vim /etc/my.cnf.d/mysql-server.cnf
....
[mysqld]
server-id=183
read-only
log-bin=/data/mysql/logbin/mysql-bin
rpl_semi_sync_slave_enabled
# log_slave_updates
default_authentication_plugin=mysql_native_password
replicate-do-db=db1			# 同步数据库
replicate-do-db=db2			# 同步数据库
replicate-wild-do-table=db%.stu		# 同步数据表
#重启服务
systemctl restart mysqld
#重置主从同步
mysql> CHANGE MASTER TO
      MASTER_HOST='10.0.0.11',
      MASTER_USER='repluser',
      MASTER_PASSWORD='123456',
      MASTER_PORT=3306,
      MASTER_LOG_FILE='mysql-bin.000011', MASTER_LOG_POS=157;
Query OK, 0 rows affected, 9 warnings (0.03 sec)
mysql> start slave;
```



### GTID复制

GTID（global transaction ID）：全局事务ID

```
GTID 是一个己提交的事务的编号，由当前 MySQL 节点的 server-uuid 和每个事务的transacton-id 联合组成，每个事务的 transacton-id 唯一，但仅只在当前节点唯一，server-uuid 是在每个节点自动随机生成，能保证每个节点唯一。基于此，用 server-uuid 和 transacton-id 联合的 GTID 也能保证全局唯一。
```

GTID 具有幂等性特性，即多次执行结果是一样的。

GTID 优点：

```
- GTID 使用 master_auto_position=1 代替了基于 binlog 和 position 号的主从复制搭建的方式，更便于主从复制的搭建。
- GTID 可以知道事务在最开始是在哪个实例上提交的，保证事务全局统一
- 截取日志更加方便。跨多文件，判断起点终点更加方便
- 传输日志，可以并发传输。SQL 回放可以更高并发
- 判断主从工作状态更加方便
注意：不是所有的事务都可以实现并行，如果多个存在增删改依赖的动作发送到目标主机，在远程主机需要维持一个基本的顺序，从而保证服务能够正常的运行。
```

```
gtid_mode=ON 					#gtid模式
enforce_gtid_consistency=ON 	#保证GTID安全的参数
```

```powershell
slave：12

[mysqld]
...
server-id               = 102
read-only
log_bin                 = /data/mysql/logbin/mysql-bin
# log_slave_updates
default_authentication_plugin=mysql_native_password
#rpl_semi_sync_slave_enabled
#replicate-do-db=db1
#replicate-do-db=db2
#replicate-wild-do-table=db%.stu
# 增加以下两条配置
gtid_mode=ON
enforce_gtid_consistency=ON
...
关于半同步配置启用会导致mysql启动失败的原因，需要启用插件
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
systemctl restart mysqld


master：11
[mysqld]
server-id=101
log_bin=/data/mysql/logbin/mysql-bin
default_authentication_plugin=mysql_native_password
# rpl_semi_sync_master_enabled
# rpl_semi_sync_master_timeout=3000
# 增加以下两条配置
gtid_mode=ON
enforce_gtid_consistency=ON
...
systemctl restart mysqld
mysql> reset master;
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      157 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

slave:11
#在salve节点上设置主从同步
mysql> CHANGE MASTER TO
      MASTER_HOST='10.0.0.11',
      MASTER_USER='repluser',
      MASTER_PASSWORD='123456',
      MASTER_PORT=3306,
      MASTER_AUTO_POSITION=1; 				# 使用 该属性
mysql> start slave;
```

主从复制管理

```powershell
SHOW MASTER STATUS;
SHOW BINARY LOGS;
SHOW BINLOG EVENTS;
SHOW SLAVE STATUS;
SHOW PROCESSLIST;
SHOW SLAVE HOSTS;			# 在主节点上查看所有的从节点列表
```

### 常见问题和解决方案

数据损坏或丢失

```
- 如果是 slave 节点的数据损坏或丢失，重置数据库，重新同步复制即可
- 如果要防止 master 节点的数据损坏或丢失，则整个主从复制架构可以用 MHA+半同步来实现，在master 节点不可用时，提升一个 salve 节点为新的 master 节点
```

在环境中出现了不唯一的 server-id

```
可手动修改 server-id 至唯一，再次重新复制
```

主从复制出现延迟

```
- 升级到 MySQL5.7 以上版本(5.7之前的版本，没有开 GTID 之前，主库可以并发事务，但是 dump 传输时是串行)利用 GTID( MySQL5.6需要手动开启，MySQL5.7 以上默认开启)支持并发传输 binlog 及并行多个 SQL 线程。
- 减少大事务，将大事务拆分成小事务
- 减少锁
- sync_binlog=1 加快 binlog 更新时间，从而加快日志复制
- 需要额外的监控工具的辅助
- 多线程复制：对多个数据库复制
- 一从多主：Mariadb10 版后支持
```

主从节点数据不一致

常见原因

```
- 主库 binlog 格式为 Statement，同步到从库执行后可能造成主从不一致。
- 主库执行更改前有执行set sql_log_bin=0，会使主库不记录 binlog，从库也无法变更这部分数据。
- 从节点未设置只读，误操作写入数据
- 主库或从库意外宕机，宕机可能会造成 binlog 或者 relaylog 文件出现损坏，导致主从不一致
- 主从实例版本不一致，特别是高版本是主，低版本为从的情况下，主数据库上面支持的功能，从数据库上面可能不支持该功能
- 主从 sql_mode 不一致,也就是说，两方对于数据库执行的检查标准不一样 -- 参考服务器变量一节
- MySQL 自身 bug 导致
```

解决方案

```
- 将从库重新实现：虽然这是一种解决方法，但此方案恢复时间较慢，而且有时候从库也是承担一部分的查询操作的，不能贸然重建。
- 使用 percona-toolkit 工具辅助：PT 工具包中包含 pt-table-checksum 和 pt-table-sync 两个工具，主要用于检测主从是否一致以及修复数据不一致情况。这种方案优点是修复速度快，不需要停止主从辅助，缺点是需要会使用该工具，关于使用方法，可以参考下面链接：https://www.cnblogs.com/feiren/p/7777218.html
- 手动重建不一致的表：在从库发现某几张表与主库数据不一致，而这几张表数据量也比较大，手工比对数据
不现实，并且重做整个库也比较慢，这个时候可以只重做这几张表来修复主从不一致。这种方案缺点是在执行
导入期间需要暂时停止从库复制，不过也是可以接受的。
```

数据不一致处理

```powershell
#salve 节点停止同步
mysql>stop slave;
#master 节点导出 A,B,C的备份，并记录下同步的binlog和POS点
mysqldump -q --single-transaction --master-data=2 testdb A B C  > /backup/A_B_C.sql
#从备份文件中找到 master 节点截至到备份时的 mastr log 和 pos
-- MASTERLOGFILE='mysql-bin.888888', MASTERLOGPOS=666666; # 该条属性是注释的
#将 A_B_C.sql 文件 scp 到 slave 节点，并在 savle 节点上开启复制到指定位置再停下来
mysql>start slave until MASTERLOGFILE='mysql-bin.888888',MASTERLOGPOS=666666;
#在 slave 节点上导入备份数据
mysql
mysql>set sql_log_bin=0;
mysql>source /backup/A_B_C.sql
mysql>set sql_log_bin=1;
#导入完毕后 slave 节点继续同步
mysql>start slave;
```

如何避免主从不一致

- 主库 binlog 采用 ROW 格式 

- 主从实例数据库版本保持一致 

- 主库做好账号权限把控，不可以执行 set sql_log_bin=0 

- 从库开启只读，不允许人为写入 定期进行主从一致性检验

主从同步之间的延时如何处理

排查延时

```
1 在MySQL中，可以使用SHOW SLAVE STATUS命令查看Seconds_Behind_Master的值来判断延时情况。
 	Seconds_Behind_Master为0表示主从复制良好，正值表示已经出现延时，数字越大表示延时越严重。
2 对比主库的master_log_file和从库的relay_master_log_file，
 	以及Read_Master_Log_Pos和Exec_Master_Log_Pos，如果不同或相差较大，说明主从同步存在延时。
3 检查主库和从库的日志文件，
 	了解是否有错误或警告信息，这些信息可能有助于确定延时的原因。
```

解决方法

```
1 调整数据刷新到磁盘的方式。
 	1 安全性高，但是效率低；0或2，可能会提高性能但会降低数据安全性。
2 优化复制机制
 	考虑使用半同步复制，主库在写入数据后会等待至少一个从库确认写入成功后再返回响应。
 	这可以减少从库与主库之间的数据不一致情况，但会增加写操作的响应时间。
 	如果MySQL版本支持，可以开启多线程复制，允许多个线程同时复制不同的事务，从而提高复制速度。
3 优化网络配置
 	确保主从库之间有足够的网络带宽，尤其是在大量数据传输的情况下。
```

- 减少大事务，将大事务拆分成小事务
- 减少锁
- sync-binlog=1 加快binlog更新时间，从而加快日志复制
- 多线程复制，对多个数据库复制
- 一从多主：Mariadb10版本后支持



## MySQL 中间件代理服务器

数据库主要分为两大类：关系型数据库与 NoSQL 数据库。

```
	关系型数据库，是建立在关系模型基础上的数据库，其借助于集合代数等数学概念和方法来处理数据库中的数据。主流的 MySQL，Oracle，MS SQL Server 和 DB2 都属于这类传统数据库。
	NoSQL 数据库，全称为 Not Only SQL，意思就是适用关系型数据库的时候就使用关系型数据库，不适用的时候也没有必要非使用关系型数据库不可，可以考虑使用更加合适的数据存储。主要分为临时性键值存储（Redis，memcached），永久性键值存储（ROMA，Redis），面向文档的数据库（MongoDB，CouchDB），面向列的数据库（Cassandra，HBase），每种 NoSQL 都有其特有的使用场景及优点。

```

### RDBMS 优缺点

```powershell
关系型数据库特点：
	- 数据关系模型基于关系模型，结构化存储，完整性约束
	- 基于二维表及其之间的联系，需要连接，并，交，差，除等数据操作
	- 采用结构化的查询语言 (SQL) 做数据读写
	- 操作需要数据的一致性，需要事务甚至是强一致性
关系型数据库优点：
	- 保持数据的一致性(事务处理)
	- 可以进行 join 等复杂查询
	- 通用化，技术成熟
关系型数据库缺点：
	- 数据读写必须经过 sql 解析，大量数据，高并发下读写性能不足
	- 对数据做读写，或修改数据结构时需要加锁，影响并发操作
	- 无法适应非结构化存储
	- 扩展困难
	- 昂贵、复杂
```

### NoSQL 优缺点

```powershell
NoSQL 数据库特点：
	- 非结构化的存储
	- 基于多维关系模型
	- 具有特有的使用场景
NoSQL 数据库优点：
	- 高并发，大数据下读写能力较强
	- 基本支持分布式，易于扩展，可伸缩
	- 简单，弱结构化存储
NoSQL 数据库缺点：
	- join 等复杂操作能力较弱
	- 事务支持较弱
	- 通用性差
	- 无完整约束复杂业务场景支持较差
```

### 数据库切片

数据库切（分）片

```
数据库切（分）片简单来说，就是指通过某种特定的条件，将我们存放在同一个数据库中的数据分散存放到多个数据库（主机）中，以达到分散单台设备负载的效果。
```

数据的切分（Sharding）根据其切分规则的类型，可以分为两种切分模式 -- 垂直|水平。

```
	一种是按照不同的表（或者 Schema）来切分到不同的数据库（主机）之上，这种切分可以称为数据的垂
直（纵向）切分；
 	另外一种则是根据表中的数据的逻辑关系，将同一个表中的数据按照某种条件拆分到多台数据库（主机）上面，这种切分称为数据的水平（横向）切分。
```

```
	垂直切分的最大特点就是规则简单，实施也更为方便，尤其适合各业务之间的耦合度非常低，相互影响小，业务逻辑非常清晰的系统，在这种系统中，可以很容易做到将不同业务模块所使用的表分拆到不同的数据库中。根据不同的表来进行拆分，对应用程序的影响也更小，拆分规则也会比较简单清晰。
	水平切分于垂直切分相比，相对来说稍微复杂一些。因为要将同一个表中的不同数据拆分到不同的数据库中， 对于应用程序来说，拆分规则本身就较根据表名来拆分更为复杂，后期的数据维护也会更为复杂一些。
```

垂直切分优缺点

```
垂直切分的优点：
- 拆分后业务清晰，拆分规则明确
- 系统之间整合或扩展容易
- 数据维护简单
垂直切分的缺点：
- 部分业务表无法 join，只能通过接口方式解决，提高了系统复杂度
- 受每种业务不同的限制存在单库性能瓶颈，不易数据扩展跟性能提高
- 事务处理复杂
```

水平切分优缺点

```
优点
- 拆分规则抽象良好，join 操作基本都可以数据库完成
- 不存在单库大数据，高并发的性能瓶颈
- 应用端改造较少
- 提高了系统的稳定性跟负载能力
缺点
- 拆分规则难以抽象
- 分片事务一致性难以解决
- 数据多次扩展难度跟维护量极大
- 跨库 join 性能较差
```

 缺点小结

```
- 引入分布式事务的问题
- 跨节点 Join 的问题
- 跨节点合并排序分页问题
- 多数据源管理问题
```

### mycat

MyCat实现 MySQL 读写分离

#### Mycat工作原理

Mycat的原理中最重要的一个动词是“拦截”，它拦截了用户发送过来的SQL语句，首先对SQL语句做了一些特定的分析，比如分片分析，路由分析，读写分离分析，缓存分析等，然后将此SQL发往后端的真实数据库，并将返回的结果做适当的处理，最终再返回给用户

#### Mycat应用场景

Mycat适用的场景很丰富，以下是几个典型的应用场景

- 单纯的读写分离，此时配置最为简单，支持读写分离，主从切换
- 分表分库，对于超过1000万的表进行分片，最大支持1000亿的单表分片
- 多租户应用，每个应用一个库，但应用程序只连接Mycat，从而不改造程序本身，实现多租户化
- 报表系统，借助于Mycat的分表能力，处理大规模报表的统计
- 替代Hbase，分析大数据
- 作为海量数据实时查询的一种简单有效方案，比如100亿条频繁查询的记录需要在3秒内查询出来结果，除了基于主键的查询，还可能在范围查询或其他属性查询，此时Mycat可能是最简单有效的选择

#### Mycat的高可用

需要注意：在生产环境下，Mycat节点最好使用双节点，即双机热备环境，防止Mycat这一层出现单点故障

- Keepalived + Mycat + MySQL
- Keepalived + LVS + Mycat + MySQL
- Keepalived + Haproxy + Mycat + MySQL

#### Mycat实现MySQL读写分离

Mycat安装

```shell
# mycat是基于java语言开发，因此要先安装java环境
yum install -y java-1.8.0-openjdk

# 下载mycat
mkdir /data/softs -p;cd /data/softs/
wget https://www.mysticalrecluse.com/script/tools/Mycat-server-1.6.7.6-release-20220524173810-linux.tar.gz

# 解压
mkdir /apps;
tar xf Mycat-server-1.6.7.6-release-20220524173810-linux.tar.gz -C /apps
```

目录结构

```shell
bin           # mycat命令，启动，重启，停止等
catlet        # 扩展功能目录，默认为空
conf          # 配置文件目录
lib           # 引用的jar包
logs          # 日志目录，默认为空
version.txt   # 版本说明文件

# 日志
logs/wrapper.log   # Mycat启动日志
logs/mycat.log     # Mycat详细工作日志

# 常用配置文件
conf/server.xml    # Mycat软件本身相关的配置文件，设置账号，参数等
conf/schema.xml    # 对应的物理数据库和数据库表的配置，读写分离，高可用，分布式策略定制，节点控制
conf/rule.xml      # Mycat分片（分库分表）规则配置文件，记录分片规则列表，使用方法等。
```

```shell
# 定制环境变量写PATH
vim /etc/profile.d/mycat.sh
PATH=/apps/mycat/bin:$PATH

source /etc/profile.d/mycat.sh

# 启动
mycat start

# 连接mycat
mysql -uroot -p123456 -h 127.0.0.1 -p8066
```

配置文件说明

```shell
server.xml  # 存放mycat软件本身相关配置的文件，比如，链接mycat的用户，密码，数据库名等

# 相关配置
user        # 用户配置节点
name        # 客户端登录mycat的用户名
password    # 客户端登录mycat的密码
schema      # 数据库名，这里会和schema.xml中的相关配置关联，多个用逗号分开
privileges  # 配置用户针对表的增删改查的权限
readOnly    # mycat逻辑库所具有的权限，true为只读，false为读写都有，默认false
```

- server.xml文件里登录mycat的用户名和密码可以任意定义，这个账号和密码是为客户机登录mycat时使用的账号信息
- 逻辑库名(如上面的TESTDB，也就是登录mycat后显示的库名，切换这个库后，显示的就是代理的真实mysql数据库的表)要在`schema.xml`里面也定义，否则会导致mycat服务启动失败
- 这里只定义了一个标签，所以把多余的注释了，如果定义多个标签，即设置多个连接mycat的用户和密码，那么就需要在schema.xml文件中定义多个对应的库

Schema.xml文件配置

```shell
schema.xml       # 是最主要的配置项，此文件关联mysql读写分离策略，读写分离，分库分表策略，分片节点都在此文件中配置的
                 # myCat作为中间件，它只是一个代理，本身并不进行数据存储，需要连接后端的MySQL的物理服务器，此文件就是用来连接件MySQL服务器的

schema           # 数据库设置，此数据库为逻辑数据库，name与server.xml中schema对应
dataNode         # 分片信息，也就是分库相关配置
dataHost         # 物理数据库，真正存储数据的数据库

# 每个节点的属性逐一说明
schema
    name            # 逻辑数据库名，与server.xml中的schema对应
    checkSQL_schema # 数据库前缀相关的设置，这里为false
    sqlMax.limit    # select时默认的limit，避免查询全表

table
    name            # 表名，物理数据库中表名
    dataNode        # 表存储到哪些节点，多个节点用逗号分隔，节点为下文dataNode设置的name
    primaryKey      # 主键字段名，自动生成主键时需要设置
    autoIncrement   # 是否自增
    rule            # 分片规则名，具体规则下文rule详细介绍
  
dataNode
    name            # 节点名，与table中dataNode对应
    datahost        # 物理数据库名，与datahost中name对应
    database        # 物理数据库中的数据库名

dataHost
    name            # 物理数据库名，与dataNode中dataHost对应
    balance         # 均衡负载的方式
    writeType       # 写入方式
    dbType          # 数据库类型
    heartbeat       # 心跳检测语句，注意语句结束结尾的分号要加  
```

schema.xml 文件中有三点需要注意：balance="1"，writeType="0" ,switchType="1"

schema.xml 中的 balance 的取值决定了负载均衡对非事务内的读操作的处理。balance 属性负载均衡 类型，目前的取值有 4 种：

```
- balance="0"：
 	不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上，即读请求仅发送到writeHost 上。
- balance="1"：
 	一般用此模式，读请求随机分发到当前 writeHost 对应的 readHost 和 standby 的 writeHost 上。即全部的 readHost 与stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1 ->S1 ， M2->S2，并且 M1 与 M2 互为主备)，正常情况下， M2,S1, S2 都参与 select语句的负载均衡。
- balance="2"：
 	读请求随机分发到当前 dataHost 内所有的 writeHost 和 readHost 上。即所有读操作都随机的在 writeHost， readhost 上分发。
- balance="3"：
 	读请求随机分发到当前 writeHost 对应的 readHost上。即所有读请求随机的分发到 wiriterHost 对应的 readhost 执行, writerHost 不负担读压力，注意 balance=3 只在 1.4 及其以后版本有，1.3 没有。
```



#### 读写分离的实现

前置工作：管理Selinux，关闭防火墙

配置MySQL主从

配置Mycat

```
master：11
slave:12
mycat:13
```

```powershell
master：11
在master上创建mycat用户
create user 'mycater'@'10.0.0.%' identified by '123456';
grant all on db1.* to 'mycater'@'10.0.0.%';
flush privileges;
```

修改server.xml配置mycat连接后端数据库的账号密码

```shell
vim /apps/mycat/conf/server.xml
<?xml version="1.0" encoding="UTF-8"?>
 <!DOCTYPE mycat:server SYSTEM "server.dtd">
 <mycat:server xmlns:mycat="http://io.mycat/">
 <system>
 <property name="useHandshakeV10">1</property>
 <property name="serverPort">3306</property>         

 </system>
<user name="root">              
<property name="password">123456</property>         
<property name="schemas">db1</property>
 <property name="defaultSchema">db1</property>
 </user>
</mycat:server>
```

修改schema.xml实现读写分离策略

```shell
vim /apps/mycat/conf/schema.xml
<?xml version="1.0"?>
 <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
 <mycat:schema xmlns:mycat="http://io.mycat/">
 <schema name="db1" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
 </schema>
    <dataNode name="dn1" dataHost="localhost1" database="db1" />
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="1" writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
        <heartbeat>select user();</heartbeat>
        <writeHost host="host1" url="10.0.0.11:3306" user="mycater" password="123456"> 
<readHost host="host2" url="10.0.0.12:3306" user="mycater" password="123456" />
        </writeHost>
    </dataHost>
</mycat:schema>
```

```shell
#重启mycat
mycat restart
mycat status
#连接mycat
mysql -uroot -p123456 -h127.1

#在master和slave上开启通用日志
vim /etc/my.cnf.d/mysql-server.cnf
...
general_log			#添加这行
...
systemctl restart mysqld.service
```



#### 实验过程中的问题

```shell
# mycat使用3306端口的时候，同主机上的mysql服务要停止，防止抢占端口

# 主从服务器上的mycater账号的权限问题
# Mycat 可能不支持 MySQL 8.0 默认的 caching_sha2_password 认证插件。如果你使用的是 MySQL 8.0 或更高版本，可能需要将用户的认证插件更改为 mysql_native_password。
ALTER USER 'mycater'@'%' IDENTIFIED WITH 'mysql_native_password' BY '123456';

# 关闭防火墙
```

### ProxySQL实现MySQL读写分离

ProxySQL: MySQL中间件
两个版本：官方版和percona版，percona版是基于官方版基础上修改
C++语言开发，轻量级但性能优异，支持处理千亿级数据

具有中间件所需要的绝大多数功能，包括

- 多种方式的读写分离
- 定制基于用户、基于schema、基于语句的规则对于SQL语句进行路由
- 缓存查询结果
- 后端节点监控

#### proxySQL体系结构

库介绍

- main: ProxySQL最主要的库，修改配置时使用的库，它是一个`内存数据库系统`。所以，修改main库中的配置后，必须将其持久化到disk上才能永久保存。
- disk: 磁盘数据库，该数据库结构和内存数据库完全一致。当持久化内存数据库中的配置时，其实就是写入disk库中。磁盘数据库的默认路径为$DATADIR/proxysql.db
- stats: 统计信息库。这个库包含了ProxySQL收集的关于内部功能的指标。通过这个数据库，可以知道触发某个计数器的频率，通过ProxySQL的SQL执行次数等
- monitor: 监控后端MySQL节点的相关的库，该库中只有几个log类的库，监控模式收集的监控信息全部存放到对应的log表中（心跳监控，主从复制监控，主从延时监控，读写监控）
- stats_history: 用于存放历史统计数据。

```shell
mysql> show databases;
+-----+---------------+-------------------------------------+
| seq | name          | file                                |
+-----+---------------+-------------------------------------+
| 0   | main          |                                     |
| 2   | disk          | /var/lib/proxysql/proxysql.db       |
| 3   | stats         |                                     |
| 4   | monitor       |                                     |
| 5   | stats_history | /var/lib/proxysql/proxysql_stats.db |
+-----+---------------+-------------------------------------+
# 可以看出main,stats,monitor都是内存库

# 查看内存库的方法
show tables from main;

mysql> show tables from main;
# 使用use的时候，默认是进入main库
# 比如：use stats,进入stats库中，然后查询表，实际情况是
# 查看的是main库的表，所以必须要使用show tables from stats来查看对应库的表
```

#### ProxySQL部署

```powershell
#配置软件源
cat > /etc/yum.repos.d/proxysql.repo << EOF
[proxysql]
name=ProxySQL YUM repository
baseurl=https://repo.proxysql.com/ProxySQL/proxysql-2.7.x/centos/\$releasever
gpgcheck=1
gpgkey=https://repo.proxysql.com/ProxySQL/proxysql-2.7.x/repo_pub_key
EOF
yum makecache
#安装软件
yum install proxysql


rpm -ql proxysql
/etc/logrotate.d/proxysql
/etc/proxysql.cnf			#配置文件
/etc/systemd/system/proxysql-initial.service
/etc/systemd/system/proxysql.service	#服务脚本
/usr/bin/proxysql			#主程序
/usr/lib/.build-id
/usr/lib/.build-id/45
/usr/lib/.build-id/45/4b03fef98200802ba260c5462676ccc84b666a
/usr/share/proxysql/tools/proxysql_galera_checker.sh
/usr/share/proxysql/tools/proxysql_galera_writer.pl

#启动服务
systemctl start proxysql
systemctl is-active proxysql
active
```

#### 配置文件内容

```powershell
vim /etc/proxysql.cnf
# 数据库目录
datadir="/var/lib/proxysql"
# 日志 （用于平时定位问题）
errorlog="/var/lib/proxysql/proxysql.log"
# 和管理相关的
admin_variables=
{
	admin_credentials="admin:admin"
#	mysql_ifaces="127.0.0.1:6032;/tmp/proxysql_admin.sock"
	mysql_ifaces="0.0.0.0:6032"
#	refresh_interval=2000
#	debug=true
}
#和mysql相关
mysql_variables=
{
	threads=4
	max_connections=2048
	default_query_delay=0
	default_query_timeout=36000000
	have_compress=true
	poll_timeout=2000
#	interfaces="0.0.0.0:6033;/tmp/proxysql.sock"
	interfaces="0.0.0.0:6033"
	default_schema="information_schema"
	stacksize=1048576
	server_version="5.5.30"
	connect_timeout_server=3000
# make sure to configure monitor username and password
# https://github.com/sysown/proxysql/wiki/Global-variables#mysql-monitor_username-mysql-monitor_password
	monitor_username="monitor"
	monitor_password="monitor"
	monitor_history=600000
	monitor_connect_interval=60000
	monitor_ping_interval=10000
	monitor_read_only_interval=1500
	monitor_read_only_timeout=500
	ping_interval_server_msec=120000
	ping_timeout_server=500
	commands_stats=true
	sessions_sort=true
	connect_retries_on_failure=10
}

#和数据库相关
# defines all the MySQL servers
mysql_servers =
(
#	{
#		address = "127.0.0.1" # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
#		port = 3306           # no default, required . If port is 0 , address is interpred as a Unix Socket Domain
#		hostgroup = 0	        # no default, required
#		status = "ONLINE"     # default: ONLINE
#		weight = 1            # default: 1
#		compression = 0       # default: 0
#   max_replication_lag = 10  # default 0 . If greater than 0 and replication lag passes such threshold, the server is shunned
#	},
#	{
#		address = "/var/lib/mysql/mysql.sock"
#		port = 0
#		hostgroup = 0
#	},
#	{
#		address="127.0.0.1"
#		port=21891
#		hostgroup=0
#		max_connections=200
#	},
#	{ address="127.0.0.2" , port=3306 , hostgroup=0, max_connections=5 },
#	{ address="127.0.0.1" , port=21892 , hostgroup=1 },
#	{ address="127.0.0.1" , port=21893 , hostgroup=1 }
#	{ address="127.0.0.2" , port=3306 , hostgroup=1 },
#	{ address="127.0.0.3" , port=3306 , hostgroup=1 },
#	{ address="127.0.0.4" , port=3306 , hostgroup=1 },
#	{ address="/var/lib/mysql/mysql.sock" , port=0 , hostgroup=1 }
)

# 和数据库用户相关
# defines all the MySQL users
mysql_users:
(
#	{
#		username = "username" # no default , required
#		password = "password" # default: ''
#		default_hostgroup = 0 # default: 0
#		active = 1            # default: 1
#	},
#	{
#		username = "root"
#		password = ""
#		default_hostgroup = 0
#		max_connections=1000
#		default_schema="test"
#		active = 1
#	},
#	{ username = "user1" , password = "password" , default_hostgroup = 0 , active = 0 }
)


#和mysql查询规则相关
#defines MySQL Query Rules
mysql_query_rules:
(
#	{
#		rule_id=1
#		active=1
#		match_pattern="^SELECT .* FOR UPDATE$"
#		destination_hostgroup=0
#		apply=1
#	},
#	{
#		rule_id=2
#		active=1
#		match_pattern="^SELECT"
#		destination_hostgroup=1
#		apply=1
#	}
)

#和调度相关
scheduler=
(
#  {
#    id=1
#    active=0
#    interval_ms=10000
#    filename="/var/lib/proxysql/proxysql_galera_checker.sh"
#    arg1="0"
#    arg2="0"
#    arg3="0"
#    arg4="1"
#    arg5="/var/lib/proxysql/proxysql_galera_checker.log"
#  }
)

#调度组相关
mysql_replication_hostgroups=
(
#        {
#                writer_hostgroup=30
#                reader_hostgroup=40
#                comment="test repl 1"
#       },
#       {
#                writer_hostgroup=50
#                reader_hostgroup=60
#                comment="test repl 2"
#        }
)
```

#### 多层配置系统

多层配置的目的是为了方便在线配置，在线生效，确保在零停机的状态下做配置变更。主要分三层：

- Runtime: 内容无法直接修改
- Memory：mysql_servers; mysql_users; mysql_query_rules; global_variables; mysql_collations
- Disk&Configuration File;

tip: 因为有runtime层，因此可以实现配置变更的在线生效，不需要做停机处理

这里我们唯一手动修改的是memory层，然后通过LOAD去更改runtime层

ProxySQL接收到LOAD...FROM CONFIG命令时，预期行为如下：

- 如果配置文件和内存表中都存在已加载的条目，则LOAD...FROM CONFIG将会覆盖内存表中已配置的条目
- 如果配置文件中存在但内存表中不存在已加载的条目，则LOAD...FROM CONFIG将会将该条目添加到内存表中
- 如果内存表中存在但配置文件中不存在的条目，则LOAD...FROM CONFIG不会从内存表中删除该条目

```shell
# 将Memory修改的命令加载到Runtime, 下面两条命令等价
Load ... FROM MEMORY
LOAD ... TO RUNTIME

# 将Runtime层的命令同步到MEMORY层, 下面两条命令等价
SAVE ... TO MEMORY
SAVE ... FROM RUNTIME

# 将Disk的内容同步到Memory
Load ... TO MEMORY
LOAD ... FROM DISK

# 将MEMROY的内容落盘的DISK
SAVE ... FROM MEMROY
SAVE ... TO DISK

# 将配置文件的内容加载到内存MEMORY
LOAD ... FROM CONFIG
```

上述的...表示五类条目，分别是

- Active/Persist Mysql Users

```shell
# Active current in-memory MySQL User configuration
LOAD MYSQL USERS TO RUNTIME;

# Save the current in-memory MySQL User Configuration to disk
SAVE MYSQL USERS TO DISK;
```

- Active/Persist MySQL Servers and MySQL Repllication Hostgroup

```shell
# Active current in-memory MySQL Server and Replication Hostgroup configuration
LOAD MYSQL SERVERS TO RUNTIME;

# Save the current in-memory MySQL Server and Replication Hostgroup configuration to disk
SAVE MYSQL SERVERS TO DISK;
```

- Activate/Persist MySQL Query Rules

```shell
# Active current in-memory MySQL Query Rule configuration
LOAD MySQL QUERY RULES TO RUNTIME;

# Save the current in-memory MySQL query Rule configuration to disk
SAVE MYSQL QUERY RULES TO DISK;
```

- Activate/Persist MySQL Variables

```shell
# Active current in-memory MySQL Variable configuration
LOAD MYSQL VARIABLES TO RUNTIME;
# Save the current in-memory MySQL variable configuration to disk
SAVE MYSQL VARIABLES TO DISK;
```

- Activate/Persist ProxySQL Admin Varables

```shell
# Active current in-memory ProxySQL Admin Variable configuration
LOAD ADMIN VARIABLES TO RUNTIME;
# SAVE the current in-memory ProxySQL Admin Variable configuration to disk

```

```
连接到数据库(默认用户默认密码)
mysql -uadmin -padmin -P6032 -h127.0.0.1

mysql> show databases;
+-----+---------------+-------------------------------------+
| seq | name          | file                                |
+-----+---------------+-------------------------------------+
| 0   | main          |                                     |
| 2   | disk          | /var/lib/proxysql/proxysql.db       |
| 3   | stats         |                                     |
| 4   | monitor       |                                     |
| 5   | stats_history | /var/lib/proxysql/proxysql_stats.db |
+-----+---------------+-------------------------------------+
5 rows in set (0.00 sec)
```

proxy配置

```shell
# 启动 
service proxysql start

# 默认监听6032，6033
# 6032是ProxySQL的管理端口，6033是ProxySQL对外提供服务的端口

# 使用mysql客户端连接到ProxySQL的管理接口6032，默认管理员用户和密码都是admin
mysql -uadmin -padmin -P6032 -h127.0.0.1

# main是默认的"数据库"名，表里存放后端db实例，用户验证、路由规则等信息。表名以runtime开头的表示proxysql当前运行的配置内容，不能通过dml语句修改，只能修改对应的不以runtime_开头的(内存)里的表，SAVE使其存到硬盘一共下次启动重载

# disk是持久化到硬盘的配置，sqlite数据文件

# stats是proxysql运行抓取的统计信息，包括到后端各命令的执行次数，流量，process list，查询种类汇总/执行时间，等等

# monitor库存储monitor模块收集的信息，主要是对后端db的健康/延迟检查

# 在main和monitor数据中的表，runtime开头的是运行时的配置，不能修改，只能修改非runtime表

# 修改后必须执行LOAD ... TO RUNTIME才能加载到RUNTIME生效

# 执行save ... to disk 才将配置持久化保存到磁盘，即保存在proxysql.db文件中

# global_variables有许多变量可以设置，其中就包括监听的端口，管理账号等
```

#### proxy实现mysql读写分离

```
master：11
slave：12
proxySQl：13
```

```powershell
proxySQL 上配置后端节点
mysql> use main;
#查看配置
mysql> select * from proxysql_servers;
#配置后端节点，100是master组，200是salve组
mysql> insert into mysql_servers(hostgroup_id,hostname,port)values(100,'10.0.0.11',3306);
mysql> insert into mysql_servers(hostgroup_id,hostname,port)values(200,'10.0.0.12',3306);
注意：
 如果想要添加多个slave，只需要将节点加入到 200小组即可。
#保存配置
#加载到RUNTIME
mysql> load mysql servers to runtime;
#保存到硬盘
mysql> save mysql servers to disk;


#master节点上创建用户并授权
mysql> create user proxyer@'10.0.0.%' identified by '123456';
mysql> grant REPLICATION CLIENT on *.* to proxyer@'10.0.0.%';
mysql> FLUSH PRIVILEGES;

#ProxySQL上配置连接MySQL的用户名和密码
mysql> set mysql-monitor_username='proxyer';
mysql> set mysql-monitor_password='123456';
#保存配置
mysql> load mysql variables to runtime;
mysql> save mysql variables to disk;

#在 ProxySQL 上将后端主机分类
insert into mysql_replication_hostgroups(writer_hostgroup,reader_hostgroup,comment) values(100,200,"hzk");
#保存配置
mysql> load mysql variables to runtime;
mysql> save mysql variables to disk;

#根据主机 group_id 分到 write 组和 read 组
mysql> select * from mysql_replication_hostgroups;
+------------------+------------------+------------+---------+
| writer_hostgroup | reader_hostgroup | check_type | comment |
+------------------+------------------+------------+---------+
| 100              | 200              | read_only  | hzk     |
+------------------+------------------+------------+---------+
1 row in set (0.00 sec)

mysql> select hostgroup_id,hostname,port,status,weight from mysql_servers;
+--------------+-----------+------+--------+--------+
| hostgroup_id | hostname  | port | status | weight |
+--------------+-----------+------+--------+--------+
| 100          | 10.0.0.11 | 3306 | ONLINE | 1      |
| 200          | 10.0.0.12 | 3306 | ONLINE | 1      |
+--------------+-----------+------+--------+--------+
2 rows in set (0.00 sec)

master节点配置
#master节点创建用户并授权 
mysql> create user sqluser@'10.0.0.%' identified by '123456';
mysql> grant all on *.* to sqluser@'10.0.0.%';
mysql> FLUSH PRIVILEGES;

#在ProxySQL配置，将用户sqluser添加到mysql_users表中， default_hostgroup默认组设置为写组100，当读写分离的路由规则不符合时，会访问默认组的数据库
mysql> insert into mysql_users(username,password,default_hostgroup)values('sqluser','123456',100);
#保存配置
mysql> load mysql variables to runtime;
mysql> save mysql variables to disk;

#在proxysql节点上，使用操作数据的用户测试连接，默认连接到 master 节点
mysql -usqluser -p'123456' -P6033 -h127.0.0.1 -e 'select @@server_id,@@read_only';

#如果上一条命令保存，执行下面命令更新一下
UPDATE mysql_users SET password = '123456' WHERE username = 'sqluser';
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;

配置读写分离
#在 proxysql 上配置路由规则，实现读写分离
mysql -uadmin -padmin -P6032 -h127.0.0.1
mysql> insert into mysql_query_rules(rule_id,active,match_digest,destination_hostgroup,apply)VALUES (1,1,'^SELECT.*FOR UPDATE$',100,1),(2,1,'^SELECT',200,1);
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;

root@rocky9-13:apps # mysql -usqluser -p'123456' -P6033 -h127.0.0.1 -e 'select
@@server_id,@@read_only'
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------+-------------+
| @@server_id | @@read_only |
+-------------+-------------+
|         102 |           1 |
+-------------+-------------+
root@rocky9-13:apps # mysql -usqluser -p'123456' -P6033 -h127.0.0.1 -e 'start
transaction;select @@server_id,@@read_only;commit;'
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------+-------------+
| @@server_id | @@read_only |
+-------------+-------------+
|         101 |           0 |
+-------------+-------------+
#如果一直调度到101上面时，重启一下master、slave
#再运行一下
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
```

```powershell
#查看proxysql路由
mysql> SELECT rule_id, active, match_digest, destination_hostgroup, apply FROM mysql_query_rules;
+---------+--------+----------------------+-----------------------+-------+
| rule_id | active | match_digest         | destination_hostgroup | apply |
+---------+--------+----------------------+-----------------------+-------+
| 1       | 1      | ^SELECT.*FOR UPDATE$ | 100                   | 1     |
| 2       | 1      | ^SELECT              | 200                   | 1     |
+---------+--------+----------------------+-----------------------+-------+
2 rows in set (0.00 sec)

#查看主机组和和对应服务器信息
mysql> SELECT hostgroup_id, hostname, port, status, weight FROM mysql_servers;
+--------------+-----------+------+--------+--------+
| hostgroup_id | hostname  | port | status | weight |
+--------------+-----------+------+--------+--------+
| 100          | 10.0.0.11 | 3306 | ONLINE | 1      |
| 200          | 10.0.0.12 | 3306 | ONLINE | 1      |
+--------------+-----------+------+--------+--------+
2 rows in set (0.00 sec)

#查询写操作
mysql> SELECT hostgroup_id, hostname, port FROM mysql_servers WHERE hostgroup_id = 100; -- 写操作
+--------------+-----------+------+
| hostgroup_id | hostname  | port |
+--------------+-----------+------+
| 100          | 10.0.0.11 | 3306 |
+--------------+-----------+------+
1 row in set (0.00 sec)

#查询读操作
mysql> SELECT hostgroup_id, hostname, port FROM mysql_servers WHERE hostgroup_id = 200; -- 读操作
+--------------+-----------+------+
| hostgroup_id | hostname  | port |
+--------------+-----------+------+
| 200          | 10.0.0.12 | 3306 |
+--------------+-----------+------+
1 row in set (0.00 sec)

```

总结：

1. 创建一个后端集群（即一个抽象的条目），并设置好它的读写
2. 将后端服务器节点根据需求分别加入到集群的写组和读组中
3. 配置监控后端的mysql节点的用户账号（用户名，密码）
4. ProxySQL上配置连接mysql的用户名和密码
5. 创建业务账号（在后端数据库和proxysql都创建），之前设置的是为了监控集群的账号，这次的是web链接的账号
6. proxysql中创建的业务账号的缺省是写组
7. 在proxysql上配置规则



## MySQL高可用

### MHA

MHA: Master High Availiability，对主节点进行监控，可以实现自动故障转移至其他从节点；通过提升某一从节点为新的主节点，基于主从复制实现，还需要客户端配合实现，目前MHA主要支持一主多从的架构，要搭建MHA，要求一个复制集群中必须最少有三台数据库服务器，一主二从，即一台充当master，一台充当备用master，另外一台充当从库，处于机器成本的考虑，淘宝进行了改造，目前淘宝TMHA已经支持一主一从。

#### MHA集群架构

一个MHA-manager节点可以管理多个MySQL集群

#### MHA工作原理

- MHA利用SELECT 1 As Value 指定判断master服务器的健康性，一旦master宕机，MHA从宕机崩溃的master保存二进制日志事件(binlog evnets)
- 识别含有最新更新的slave
- 应用差异的中继日志(relay log)到所有slave节点
- 应用从master保存的二进制日志事件(binlog events)到所有slave节点
- 提升一个slave为新的master
- 是其他的slave连接新的master进行复制
- 故障服务器自动被剔除集群，将配置信息去掉
- 旧的master的VIP漂移到新的master上，用户应用就可以访问新的Master

<img src="5day-png\23MHA架构.png" alt="image-20250107172023597" style="zoom:33%;" />

<img src="5day-png\23MHA工作原理.png" alt="image-20250107172055042" style="zoom:33%;" />

#### 选主 

**选举新的 Master **

- 如果设定权重(candidate_master=1)，按照权重强制指定新主，但是默认情况下如果一个 slave 落 后 master 二进制日志超过 100M 的relay logs，即使有权重，也会失效，如果设置 check_repl_delay=0，即使落后很多日志，也强制选择其为新主 

- 如果从库数据之间有差异，最接近于 Master 的 slave 成为新主 

- 如果所有从库数据都一致，按照配置文件顺序最前面的当新主

#### 数据恢复

**数据恢复 **

- 当主服务器的 SSH 还能连接，从库对比主库 position 或者 GTID 号，将二进制日志保存至各个从 节点并且应用(执行save_binary_logs 实现) 

- 当主服务器的 SSH 不能连接，对比从库之间的 relaylog 的差异(执行apply_diff_relay_logs[实现])

注意：为了尽可能的减少主库硬件损坏宕机造成的数据丢失，因此在配置 MHA 的同时建议配置成 MySQL 的半同步复制

#### MHA 架构环境

```
MHA manager 	10.0.0.17
master 			10.0.0.11
slaver-1 		10.0.0.12
slaver-2 		10.0.0.13
```

```powershell
#配置完主从环境
master:
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |      1733 | No        |
+------------------+-----------+-----------+

slave:
#清理slave
mysql> stop slave;
mysql> reset slave all;
从环境清空环境
systemctl stop mysqld
rm -rf /var/lib/mysql/*
rm -rf /data/mysql/logbin/*
systemctl start mysqld
CHANGE MASTER TO
MASTER_HOST='10.0.0.11',
MASTER_USER='repluser',
MASTER_PASSWORD='123456',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=1733;

MHA:
#生成密钥对，并在当前主机完成C/S校验
ssh-keygen
ssh-copy-id 127.1
#分发
scp -r .ssh 10.0.0.11:
scp -r .ssh 10.0.0.12:
scp -r .ssh 10.0.0.13:

ssh root@10.0.0.11
ssh root@10.0.0.12
ssh root@10.0.0.13
ssh root@10.0.0.17

MHA软件安装
mkdir /data/softs -p;cd /data/softs/
wegt 

MHA节点上安装mha4mysql-node-0.58-0.el7.centos.noarch.rpm、mha4mysql-manager-0.58-0.el7.centos.noarch.rpm
yum install mha4mysql-*

for i in 11 12 13
do
  scp mha4mysql-node* root@10.0.0.$i:
done
master和slave上安装mha4mysql-node-0.58-0.el7.centos.noarch.rpm
yum -y install mha4mysql-node-0.58-0.el7.centos.noarch.rpm

#MHA：
#要先部署邮件服务器
yum install mailx postfix
systemctl restart postfix.service
vim /etc/mail.rc
set bsdcompat
set from=2352136389@qq.com
set smtp=imap.qq.com
set smtp-auth-user=2352136389@qq.com
set smtp-auth-password=yhdoxkdnuylmdhge
set smtp-auth=login
set ssl-verifv=ignore
set smtp-use-starttls
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb/

mkdir -p /root/.certs/
echo -n | openssl s_client -connect smtp.qq.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/qq.crt
certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt
certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt
certutil -L -d /root/.certs

systemctl restart postfix.service
vim /usr/local/bin/sendmail.sh
#!/bin/bash
echo 'MHA is failover!' | mail -s 'MHA Warning' 2352136389@qq.com
chmod a+x /usr/local/bin/sendmail.sh
sendmail.sh
```

```powershell
定制故障转移
vim /usr/local/bin/master_ip_failover
```

```perl
#!/usr/bin/env perl

use strict;
use warnings FATAL => 'all';

use Getopt::Long;
use MHA::DBHelper;

my (
  $command,        $ssh_user,         $orig_master_host,
  $orig_master_ip, $orig_master_port, $new_master_host,
  $new_master_ip,  $new_master_port,  $new_master_user,
  $new_master_password
);

my $vip = '10.0.0.100/24';
my $key = "1";
my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";

GetOptions(
  'command=s'              => \$command,
  'ssh_user=s'             => \$ssh_user,
  'orig_master_host=s'     => \$orig_master_host,
  'orig_master_ip=s'       => \$orig_master_ip,
  'orig_master_port=i'     => \$orig_master_port,
  'new_master_host=s'      => \$new_master_host, 
  'new_master_ip=s'        => \$new_master_ip,
  'new_master_port=i'      => \$new_master_port,
  'new_master_user=s'      => \$new_master_user,
  'new_master_password=s'  => \$new_master_password,
);

exit &main();

sub main {
  my $exit_code = 1;  # Default to failure

  if ( $command eq "stop" || $command eq "stopssh" ) {
    eval {
      # updating global catalog, etc
      $exit_code = 0;
    };
    if ($@) {
      warn "Got Error: $@\n";
    }
  }
  elsif ( $command eq "start" ) {
    eval {
      print "Enabling the VIP - $vip on the new master - $new_master_host \n";
      &start_vip();
      &stop_vip();
      $exit_code = 0;
    };
    if ($@) {
      warn $@;
    }
  }
  elsif ( $command eq "status" ) {
    print "Checking the Status of the script.. OK \n";
    `ssh $ssh_user\@$orig_master_host \"$ssh_start_vip\"`;
    $exit_code = 0;
  }
  else {
    &usage();
  }
  
  exit $exit_code;
}

sub start_vip {
 `ssh $ssh_user\@$new_master_host \"$ssh_start_vip\"`;
}

# A simple system call that disables the VIP on the old master 
sub stop_vip {
 `ssh $ssh_user\@$orig_master_host \"$ssh_stop_vip\"`;
}

sub usage {
  print
  "Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}
```

```powershell
# 添加执行权限
chmod a+x /usr/local/bin/master_ip_failover
```

```powershell
定制依赖vip
ifconfig eth0:1 10.0.0.100/24
```

 MHA配置环境

```powershell
在 mha-manager 节点创建相关配置文件
mkdir /etc/mastermha
vim /etc/mastermha/app1.cnf
[server default]
user=mhauser                  								  # mba-manager节点连接远程mysql使用的账户，需要有管理员的权限
password=123456                                               # mha-manager节点连接远程mysql使用的账户密码
manager_workdir=/data/mastermha/app1/                         # mha-manager对于当前集群的工作目录
manager_log=/data/mastermha/app1/manager.log                  # mha-manager对于当前集群的日志
remote_workdir=/data/mastermha/app1/                          # mysql节点mha工作目录，会自动创建
ssh_user=root                                                 # 各节点间的SSH连接账号，提前做好基于key的登录验证，用于访问二进制
repl_user=repluser                                            # mysql节点主从复制用户名
repl_password=123456                                          # mysql节点主从复制密码
ping_interval=1                                               # mha-manager节点对于master节点的心跳检测时间间隔
master_ip_failover_script=/usr/local/bin/master_ip_failover   # 切换VIP的perl脚本，不支持跨网络，也可用keepalived实现
report_script=/usr/local/bin/sendmail.sh                      # 发送告警信息脚本
check_repl_delay=0                                            # 默认值为1，表示如果slave中从库落后主库relaylog超过100M，主库不会选择这个从库作为新的master
master_binlog_dir=/data/mysql/logbin/                         # 指定二进制日志存放的目录，mha4mysql-manager-0.58必须指定，之前版本不需要指定

[server1]
hostname=10.0.0.8
candidate_master=1                                            # 优先候选master，即使不是集群中事件最新的slave，也会优先当master

[server2]
hostname=10.0.0.18
candidate_master=1                                            # 优先候选master，即使不是集群中事件最新的slave，也会优先当master

[server3]
hostname=10.0.0.28
```

精简版

```powershell
[server default]
user=mhauser
password=123456
manager_workdir=/data/mastermha/app1/
manager_log=/data/mastermha/app1/manager.log
remote_workdir=/data/mastermha/app1/
ssh_user=root
repl_user=repluser
repl_password=123456
ping_interval=1
master_ip_failover_script=/usr/local/bin/master_ip_failover
report_script=/usr/local/bin/sendmail.sh
check_repl_delay=0

master_binlog_dir=/data/mysql/logbin/

[server1]
hostname=10.0.0.11
candidate_master=1

[server2]
hostname=10.0.0.12
candidate_master=1
[server3]
hostname=10.0.0.13
```

提升slave节点为master节点的策略

- 如果所有slave节点的日志都相同，则默认会以配置文件的顺序选择一个slave节点提升为master节点
- 如果slave节点上的日志不一致，则会选择数据量最接近master节点的slave节点，将其提升为master节点
- 如果对某个slave节点设置权重(candidate_master=1)，权重节点优先选择，但是此节点日志量落后于master节点超过100M时，也不会选择，可以配合check_repl_delay=0,关闭日志量的检查，强制选择候选节点

```powershell
master:11
#创建 mha-manager 使用的账号并授权
mysql> create user mhauser@'10.0.0.%' identified by '123456';
mysql> grant all on *.* to mhauser@'10.0.0.%';
mysql> flush privileges;

mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
mysql> flush privileges;

#MHA安装mysql
yum install mysql


#检查环境
[root@mha-manager ~]#masterha_check_ssh --conf=/etc/mastermha/app1.cnf
[root@mha-manager ~]#masterha_check_repl --conf=/etc/mastermha/app1.cnf
#查看状态
[root@mha-manager ~]#masterha_check_status --conf=/etc/mastermha/app1.cnf

#开启MHA,默认是前台运行,生产环境一般为后台执行
nohup masterha_manager --conf=/etc/mastermha/app1.cnf --remove_dead_master_conf --ignore_last_failover &> /dev/null 
#测试环境：
masterha_manager --conf=/etc/mastermha/app1.cnf --remove_dead_master_conf --ignore_last_failover
#如果想停止后台执行的MHA,可以执行下面命令
[root@mha-master ~]#masterha_stop --conf=/etc/mastermha/app1.cnf Stopped app1 successfully
#查看状态
masterha_check_status --conf=/etc/mastermha/app1.cnf  
```



### Galera Cluster（GC）

#### Galera Cluster 特点

```powershell
- 多主架构：真正的多点读写的集群，在任何时候读写数据，都是最新的
- 同步复制：改善了主从复制延迟问题，基本上达到了实时同步
- 并发复制：从节点APPLY数据时，支持并行执行，更好的性能
- 故障切换：在出现数据库故障时，因支持多点写入，切换容易
- 热插拔：在服务期间，如果数据库挂了，只要监控程序发现的够快，不可服务时间就会非常少。在节点故障期间，节点本身对集群的影响非常小
- 自动节点克隆：在新增节点，或者停机维护时，增量数据或者基础数据不需要人工手动备份提供，Galera Cluster会自动拉取在线节点数据，最终集群会变为一致
- 对应用透明：集群的维护，对应用程序是透明的
```

#### Galera Cluster 缺点

```powershell
- 任何更新事务都需要全局验证通过，才会在其他节点执行，则集群性能由集群中最差性能节点决定（一般集群节点配置都是一样的）
- 新节点加入或延后较大的节点重新加入需全量拷贝数据 (SST，State Snapshot Transfer)，作为donor ( 贡献者，如： 同步数据时的提供者) 的节点在同步过程中无法提供读写
- 只支持 innodb 存储引擎的表
```

### 冲突异常怎么办

```powershell
如果操作同时符合以下三个条件，则会认为此次提交存在冲突（验证失败）
- 两个事务来源于不同节点。
- 两个事务包含相同的主键。
- 老事务对新事务不可见，即老事务未提交完成。新老事务的划定依赖于全局事务总序，即GTID。

验证失败后，节点将删除写集，集群将回滚原始事务。
	对于所有的节点都是如此，每个节点单独进行验证。因为所有节点都以相同的顺序接收事务，它们对事务的结果都会做出相同的决定，要么全成功，要么都失败。成功后自然就提交了，所有的节点又会重新达到数据一致的状态。
 	节点之间不交换 “是否冲突” 的信息，各个节点独立异步处理事务。由此可见，Galera 本身的数据也不是严格同步的，很明显在每个节点上的验证是异步的，这就是 “虚拟同步”。

最后，启动事务的节点可以通知客户端应用程序是否提交了事务。
```

PXC 最常使用如下4个端口号：

```powershell
- 3306：数据库对外服务的端口号
- 4444：请求SST的端口号
- 4567：组成员之间进行沟通的端口号
- 4568：用于传输IST的端口号
```

状态切换

```
- open：节点启动成功，尝试连接到集群时的状态
- primary：节点已处于集群中，在新节点加入并选取donor进行数据同步时的状态
- joiner：节点处于等待接收同步文件时的状态
- joined：节点完成数据同步工作，尝试保持和集群进度一致时的状态
- synced：节点正常提供服务时的状态，表示已经同步完成并和集群进度保持一致
- donor：节点处于为新加入的节点提供全量数据时的状态
```

### 部署

| 主机ip    | 主机名    | 角色   | 操作系统 | PXC版本                  |
| --------- | --------- | ------ | -------- | ------------------------ |
| 10.0.0.10 | rocky9-10 | node-1 | rocky9   | Percona-XtraDB-Cluster-8 |
| 10.0.0.11 | rocky9-11 | node-2 | rocky9   | Percona-XtraDB-Cluster-8 |
| 10.0.0.12 | rocky9-12 | node-3 | rocky9   | Percona-XtraDB-Cluster-8 |



```powershell
#all
hostnamectl set-hostname rocky9-10
hostnamectl set-hostname rocky9-11
hostnamectl set-hostname rocky9-12
exec /bin/bash
echo '127.0.0.1 rocky' >> /etc/hosts
systemctl disable --now firewalld
getenforce 0
Permissive
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm 
percona-release setup pxc-80
yum install percona-xtradb-cluster
```

```powershell
root@rocky9-11:softs # cat /etc/my.cnf | grep -Ev "^#|^$"
[client]
socket=/var/lib/mysql/mysql.sock
[mysqld]
server-id=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
binlog_expire_logs_seconds=604800
wsrep_provider=/usr/lib64/galera4/libgalera_smm.so	#指定 Galera 库的路径
wsrep_cluster_address=gcomm://						#集群中各节点地址
binlog_format=ROW									#二进制日志格式
wsrep_slave_threads=8								#默认工作线程数
wsrep_log_conflicts
innodb_autoinc_lock_mode=2							#存储引擎加锁方式，当前只能是2
wsrep_cluster_name=pxc-cluster						#集群名称
wsrep_node_name=pxc-cluster-node-1					#当前节点名称
pxc_strict_mode=ENFORCING							#是否使用测试功能，默认不使用
wsrep_sst_method=xtrabackup-v2						#数据同步方式mysqldump|skip|rsync|xtrabackup
#wsrep_node_address=192.168.70.63					#当前节点IP
```

```powershell
#如果启用加密数据传输，则要保证每个节点的公私密钥对和CA证书保持一致，且 mysqld 和 sst 处都要配置
[mysqld]
wsrep_provider_options=”socket.ssl_key=server-key.pem;socket.ssl_cert=server-cert.pem;socket.ssl_ca=ca.pem”

[sst]
encrypt=4
ssl-key=server-key.pem
ssl-ca=ca.pem
ssl-cert=server-cert.pem
```

```powershell
10.0.0.10
vim /etc/my.cnf
...
[mysqld]
server-id=10 # 非必须，建议修改
...
wsrep_cluster_address=gcomm://10.0.0.10,10.0.0.11,10.0.0.12 # 多主机间使用逗号
wsrep_node_address=10.0.0.10
wsrep_cluster_name=pxc-cluster
wsrep_node_name=pxc-cluster-node-10
pxc-encrypt-cluster-traffic=OFF # 不使用安全加密传输

10.0.0.11
vim /etc/my.cnf
...
[mysqld]
server-id=11 # 非必须，建议修改
...
wsrep_cluster_address=gcomm://10.0.0.10,10.0.0.11,10.0.0.12 # 多主机间使用逗号
wsrep_node_address=10.0.0.11
wsrep_cluster_name=pxc-cluster
wsrep_node_name=pxc-cluster-node-11
pxc-encrypt-cluster-traffic=OFF # 不使用安全加密传输

10.0.0.12
vim /etc/my.cnf
...
[mysqld]
server-id=12 # 非必须，建议修改
...
wsrep_cluster_address=gcomm://10.0.0.10,10.0.0.11,10.0.0.12 # 多主机间使用逗号
wsrep_node_address=10.0.0.12
wsrep_cluster_name=pxc-cluster
wsrep_node_name=pxc-cluster-node-12
pxc-encrypt-cluster-traffic=OFF # 不使用安全加密传输
```

```powershell
all：
chown mysql:mysql /etc/sysconfig/mysql.bootstrap
systemctl start mysql@bootstrap.service

cat /var/log/mysqld.log | grep password
2025-01-09T12:18:36.823404Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: gF%hcP+lg0qs
root@rocky9-12:softs # mysql -uroot -p'gF%hcP+lg0qs'
mysql> alter user root@'localhost' identified by '123456';
```

```powershell
all:
mysql -uroot -p'123456'

slave:10.0.0.11,10.0.0.12
systemctl start mysql@bootstrap.service
systemctl stop mysql@bootstrap.service
systemctl start mysql
```

```powershell
10.0.0.10
mysql> create database db1; use db1;
mysql> CREATE TABLE `stu`  (
    -> `id` int UNSIGNED NOT NULL AUTO_INCREMENT,
    -> `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT '',
    -> `age` int NULL DEFAULT 0,
    -> PRIMARY KEY (`id`) USING BTREE
    -> );
mysql> insert into db1.stu(name,age)values('tom',10),('jerry',20),('spike',30);
mysql> insert into db1.stu(name,age)values('tom-15',10),('jerry-15',20),('spike-15',30);
mysql> select * from db1.stu;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | tom      |   10 |
|  4 | jerry    |   20 |
|  7 | spike    |   30 |
| 10 | tom-15   |   10 |
| 13 | jerry-15 |   20 |
| 16 | spike-15 |   30 |
+----+----------+------+
```

```mysql
10.0.0.11
mysql> select * from db1.stu;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | tom      |   10 |
|  4 | jerry    |   20 |
|  7 | spike    |   30 |
| 10 | tom-15   |   10 |
| 13 | jerry-15 |   20 |
| 16 | spike-15 |   30 |
+----+----------+------+
6 rows in set (0.00 sec)

10.0.0.12
mysql> select * from db1.stu;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | tom      |   10 |
|  4 | jerry    |   20 |
|  7 | spike    |   30 |
| 10 | tom-15   |   10 |
| 13 | jerry-15 |   20 |
| 16 | spike-15 |   30 |
+----+----------+------+
6 rows in set (0.00 sec)
```

<img src="5day-png\23GC.png" alt="image-20250109205445321" style="zoom: 33%;" />

```powershell
10.0.0.11
mysql> insert into db1.stu(name,age)values('tom-18',10),('jerry-18',20),('spike-18',30);
mysql> select * from db1.stu;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | tom      |   10 |
|  4 | jerry    |   20 |
|  7 | spike    |   30 |
| 10 | tom-15   |   10 |
| 13 | jerry-15 |   20 |
| 16 | spike-15 |   30 |
| 18 | tom-18   |   10 |
| 21 | jerry-18 |   20 |
| 24 | spike-18 |   30 |
+----+----------+------+
9 rows in set (0.00 sec)
```

**冲突测试**

```powershell
三台服务器同时插入数据
10.0.0.10
insert into db1.stu(name,age)values('tom-18',10),('jerry-18',20),('spike-18',30);
10.0.0.11
insert into db1.stu(name,age)values('tom-18',10),('jerry-18',20),('spike-18',30);
10.0.0.12
insert into db1.stu(name,age)values('tom-18',10),('jerry-18',20),('spike-18',30);

mysql> select * from db1.stu;
+----+----------+------+
| id | name     | age  |
+----+----------+------+
|  1 | tom      |   10 |
|  4 | jerry    |   20 |
|  7 | spike    |   30 |
| 10 | tom-15   |   10 |
| 13 | jerry-15 |   20 |
| 16 | spike-15 |   30 |
| 18 | tom-18   |   10 |
| 21 | jerry-18 |   20 |
| 24 | spike-18 |   30 |
| 25 | tom-18   |   10 |
| 26 | tom-18   |   10 |
| 27 | tom-18   |   10 |
| 28 | jerry-18 |   20 |
| 29 | jerry-18 |   20 |
| 30 | jerry-18 |   20 |
| 31 | spike-18 |   30 |
| 32 | spike-18 |   30 |
| 33 | spike-18 |   30 |
+----+----------+------+
18 rows in set (0.00 sec)
```



### 往PXC集群中加入新节点

前置条件：待加入集群的机器，己经完成前置操作，安装了相同版本的PXC

```
1 将其它节点的 /etc/my.cnf 文件拷贝到新节点进行修改
2 修改配置项,为了保证后续重启等操作，将每个节点中的 wsrep_cluster_address 都进行修改wsrep_cluster_address=gcomm://10.0.0.10,10.0.0.11,10.0.0.12,新主机IP
3 然后，新主机启动mysql服务即可，至于加入集群，都有percona软件自动管理起来。
```



## mysql压力测试

压测工具

```powershell
	对 MySQL 服务进行压力测试，可以让我们提前知晓当前服务节点的性能，从而对整个架构的负载能力有合理的预期，进而做出相应的调整和部署。
	常见的 MySQL 压力测试工具包括 mysqlslap，Sysbench，tpcc-mysql，MySQL Benchmark Suite，MySQL super-smack，MyBench 等，其中 mysqlslap 是 mysql 官方提供的压力测试工具，安装 mysql 服务就可以直接使用。

whereis mysqlslap
```

```powershell
测试完毕后，会自动清理残留的数据
 mysqlslap -a -c100
测试完毕后，不会自动清理残留的数据，测试数据，会保留到一个名为 mysqlslap 的测试数据库中
 mysqlslap -a --no-drop
```

```powershell
root@ubuntu24:~# mysqlslap -a -c100
Benchmark
        Average number of seconds to run all queries: 2.052 seconds #平均查询时长
        Minimum number of seconds to run all queries: 2.052 seconds #查询中最小时长
        Maximum number of seconds to run all queries: 2.052 seconds #查询中最大时长
        Number of clients running queries: 100 		#查询客户端数量
        Average number of queries per client: 0 	#平均每个客户端查询次数，没有指定总的查询次数，值为0
```

```powershell
迭代10次测试,需要登录测试的话，可以通过 -uroot -p123456 方法
mysqlslap -a -i 10 -c 100
mysqlslap -a -c 50 --number-of-queries 200 --engine=myisam,innodb
```



## 面试题

### MySQL 主从复制原理

主从复制是 MySQL 数据库中实现数据同步的一个重要机制，主要用于读写分离、高可用性和数据备份。以下是主从复制的基本原理：

------

#### 1. **核心流程**

主从复制的过程分为三个主要步骤：

##### 1.1 **主库生成二进制日志（Binary Log）**

- 主库在执行数据变更操作（如 `INSERT`、`UPDATE`、`DELETE` 等）时，会将这些操作记录到二进制日志（`binlog`）中。
- 二进制日志以事件的形式存储，记录了数据变更的具体操作。

##### 1.2 **从库读取主库的二进制日志**

- 从库上的 **I/O 线程** 连接到主库，通过复制用户拉取主库的二进制日志内容。
- 拉取到的日志会存储在从库的 **中继日志（Relay Log）** 中。

##### 1.3 **从库执行中继日志**

- 从库上的 **SQL 线程** 读取中继日志中的内容，并按照主库的变更操作顺序依次执行，从而实现数据同步。

------

#### 2. **主从复制的线程角色**

##### 主库：

- **Binlog Dump Thread**
  主库上的线程负责将二进制日志推送给从库的 I/O 线程。

##### 从库：

- **I/O Thread**
  从库上的线程负责从主库拉取二进制日志并写入中继日志。
- **SQL Thread**
  从库上的线程负责读取中继日志并在从库上执行这些日志内容。

------

#### 3. **主从复制的核心组件**

##### 3.1 **二进制日志（Binary Log）**

- 主库记录所有数据变更操作的日志文件，是主从复制的基础。
- 二进制日志记录了每个操作的事务、时间点等信息。

##### 3.2 **中继日志（Relay Log）**

- 从库用来临时存储从主库拉取的二进制日志。
- SQL 线程会读取中继日志进行实际的执行。

------

#### 4. **主从复制的类型**

##### 4.1 基于语句的复制（SBR，Statement-Based Replication）

- 复制的是 SQL 语句。
- 优点：日志较小，效率高。
- 缺点：某些非确定性操作可能导致主从数据不一致。

##### 4.2 基于行的复制（RBR，Row-Based Replication）

- 复制的是每一行数据的变化。
- 优点：数据一致性高。
- 缺点：日志较大。

##### 4.3 混合模式（Mixed-Based Replication）

- 根据具体场景动态选择 SBR 或 RBR。

------

#### 5. **主从复制的特点**

- **异步复制**：从库不需要实时同步主库操作，存在一定延迟。
- **半同步复制（Semi-Synchronous Replication）**：主库等待至少一个从库确认写入后才提交事务。
- **延迟复制（Delayed Replication）**：从库可配置延迟时间，适用于灾难恢复。

------

#### 6. **主从复制的常见用途**

- **读写分离**：主库负责写操作，从库负责读操作，提升数据库性能。
- **数据备份**：从库用于数据备份，避免对主库产生压力。
- **高可用性**：在主库故障时，从库可以快速接管服务。

通过上述机制，MySQL 主从复制实现了高效的数据同步和分布式架构的基础功能。



### SQL 优化方法

SQL 优化是提升数据库性能的关键工作，包括合理设计 SQL 查询、优化索引、减少资源占用等。以下从常见场景出发，介绍 SQL 优化的主要方法。

------

#### 1. **索引优化**

索引是提高查询效率的关键，合理设计索引能显著提升性能。

##### 1.1 创建适当的索引

- **单列索引**：针对频繁查询的字段（如主键、唯一值）添加索引。
- **联合索引**：针对多条件查询，按最常用到最不常用字段的顺序创建索引。
- **覆盖索引**：确保查询字段只用索引就能满足（`SELECT` 中的字段包含在索引中）。

##### 1.2 避免全表扫描

确保查询使用索引。例如：

```
sql复制代码-- 未使用索引
SELECT * FROM employees WHERE salary + 1000 > 50000;

-- 改写使索引生效
SELECT * FROM employees WHERE salary > 49000;
```

------

#### 2. **查询优化**

改写 SQL 语句，减少不必要的操作。

##### 2.1 避免 `SELECT *`

仅查询需要的字段：

```
-- 不推荐
SELECT * FROM employees;

-- 推荐
SELECT name, position FROM employees;
```

##### 2.2 减少子查询

将子查询改为 `JOIN` 提升效率：

```
-- 子查询
SELECT name FROM employees WHERE department_id IN (SELECT id FROM departments WHERE location = 'New York');

-- 改写为 JOIN
SELECT e.name FROM employees e JOIN departments d ON e.department_id = d.id WHERE d.location = 'New York';
```

##### 2.3 使用合理的操作符

避免 `NOT IN`，改用 `NOT EXISTS` 或 `LEFT JOIN`：

```
-- 不推荐
SELECT * FROM employees WHERE id NOT IN (SELECT manager_id FROM departments);

-- 推荐
SELECT * FROM employees e WHERE NOT EXISTS (SELECT 1 FROM departments d WHERE e.id = d.manager_id);
```

------

#### 3. **表结构优化**

##### 3.1 规范化设计

- **拆分大表**：分离频繁访问的字段，减少表的宽度。
- **分区表**：针对大表按时间或其他维度进行分区。

##### 3.2 数据类型优化

- 使用合适的数据类型（`TINYINT` 替代 `INT`，`CHAR` 替代短长度的 `VARCHAR`）。
- 避免使用 `TEXT` 和 `BLOB`，必要时拆分为单独的表存储。

##### 3.3 避免碎片

定期执行 `OPTIMIZE TABLE`，整理表中由于删除或更新导致的碎片。

------

#### 4. **锁与事务优化**

##### 4.1 减少锁冲突

- 尽量使用索引避免锁表。
- 将长事务拆分为短事务，减少锁的持有时间。

##### 4.2 控制事务大小

在事务中只包含必要的操作：

```
-- 不推荐
START TRANSACTION;
UPDATE employees SET salary = salary * 1.1 WHERE department_id = 5;
DELETE FROM log WHERE create_time < '2024-01-01';
COMMIT;

-- 推荐
START TRANSACTION;
UPDATE employees SET salary = salary * 1.1 WHERE department_id = 5;
COMMIT;

START TRANSACTION;
DELETE FROM log WHERE create_time < '2024-01-01';
COMMIT;
```

------

#### 5. **分库分表**

对于大数据量的系统，可以通过以下方式提高性能：

- **垂直分库**：将不同模块的数据拆分到不同数据库。
- **水平分表**：将大表按特定规则分为多个小表（如按用户 ID 或时间分表）。

------

#### 6. **缓存和结果优化**

##### 6.1 使用缓存

- 利用数据库查询缓存（如 MySQL Query Cache）。
- 使用外部缓存工具（如 Redis、Memcached）。

##### 6.2 减少重复查询

将结果保存到临时表，避免重复计算。

------

#### 7. **分析与工具**

- 查询分析工具：
  - 使用 `EXPLAIN` 分析查询计划。
  - 查看是否使用索引及扫描的行数。
- 慢查询日志：
  - 启用慢查询日志，定位耗时较长的 SQL。

```
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 1; -- 设置慢查询时间阈值
```

------

#### 8. **常见场景优化**

##### 8.1 分页查询

避免深度分页的性能问题：

```
-- 不推荐
SELECT * FROM orders LIMIT 100000, 10;

-- 推荐（使用索引提前定位）
SELECT * FROM orders WHERE id > 100000 LIMIT 10;
```

##### 8.2 批量操作

对于大批量更新或删除，分批次执行：

```
-- 不推荐
DELETE FROM logs WHERE create_time < '2023-01-01';

-- 推荐
DELETE FROM logs WHERE create_time < '2023-01-01' LIMIT 1000;
```

------

通过索引设计、查询改写、表结构调整以及事务和工具的配合，SQL 优化可以有效提升数据库性能，满足高效的数据处理需求。





### MHA（Master High Availability）选举原理

MHA 是一种用于 MySQL 主从架构的高可用解决方案，能够在主库发生故障时快速自动切换到新的主库，同时最大程度保证数据的一致性。MHA 的选举过程主要包括以下几个步骤：

------

#### 1. **主库故障检测**

- MHA Manager 会定期监控主库的状态（通过心跳检测或连接测试）。
- 如果检测到主库不可用（如网络断开或 MySQL 服务故障），会立即触发故障切换流程。

------

#### 2. **选择候选主库**

##### 2.1 判断从库状态

- MHA 会检查所有从库的复制状态，包括：
  - **复制延迟**：通过 `SHOW SLAVE STATUS` 检查 `Seconds_Behind_Master`。
  - **中继日志的完整性**：判断从库是否完整接收了主库的二进制日志。
- 排除掉以下从库：
  - 复制延迟过大的从库。
  - 数据不一致或同步中断的从库。
  - 无法正常访问的从库。

##### 2.2 候选主库排序

- 在剩余的从库中，优先选择以下条件的从库作为候选主库：
  1. 数据最完整（接收了最多的二进制日志事件）。
  2. 复制延迟最小。
  3. 网络连接最稳定。

------

#### 3. **应用中继日志**

- 如果候选主库中包含未应用的中继日志（Relay Log），MHA 会先应用这些日志，以确保新主库的数据尽可能与原主库一致。
- 具体操作：
  1. 提取中继日志中的未执行事务。
  2. 在候选主库上重放这些事务。

------

#### 4. **更新复制拓扑**

- 确定新主库后，MHA 将其他从库的复制指向新的主库。
- 步骤：
  1. 清理原主库相关的复制信息。
  2. 在所有从库上执行 `CHANGE MASTER TO`，指向新的主库。
  3. 启动从库的复制进程。

------

#### 5. **虚拟 IP 切换（可选）**

- 如果系统使用虚拟 IP，MHA 可以将虚拟 IP 切换到新的主库，确保业务应用的透明切换。

------

#### 6. **日志记录与报警**

- 故障转移完成后，MHA 会记录整个选举和切换过程，并通过日志或报警通知管理员。

------

#### 选举过程的关键点

1. 一致性优先：
   - MHA 会尽量确保新主库的数据与原主库一致，优先选择数据最完整的从库。
2. 快速恢复：
   - 通过高效的中继日志处理和复制拓扑调整，保证故障切换的快速完成。
3. 自动化与人工介入：
   - MHA 支持自动选举主库，但也允许管理员手动介入并指定新的主库。

------

通过上述机制，MHA 能够实现 MySQL 集群在主库故障后的高效选举和切换，从而保障数据库的高可用性和一致性。





### 主从延迟问题及其原因分析

在 MySQL 主从复制中，从库可能出现延迟问题，即 **`Seconds_Behind_Master`** 显示较大的值。这种延迟会导致从库的数据滞后于主库，影响数据的一致性和查询实时性。

------

#### 1. **主从延迟的常见现象**

- 从库上的查询结果落后于主库。
- 主库负载正常，但从库响应时间变长。
- `SHOW SLAVE STATUS\G` 中的 `Seconds_Behind_Master` 显示非零甚至持续增长。

------

#### 2. **主从延迟的产生原因**

##### 2.1 主库写入压力大

- 主库承载大量并发写操作，生成的二进制日志（`binlog`）数据量大，从库需要花费更多时间从主库读取日志。
- 表现：
  - 从库的 `IO_THREAD` 正常，但 `SQL_THREAD` 处理速度跟不上。

##### 2.2 从库性能瓶颈

- 从库硬件性能不足（CPU、内存、磁盘 IO）。
- 从库的 `SQL_THREAD` 在重放事务时执行效率低，特别是大事务或复杂查询。
- 例子：
  - 大量 `INSERT`、`UPDATE`、`DELETE` 操作执行时间较长。
  - 磁盘 IO 成为瓶颈，写入速度跟不上。

##### 2.3 网络延迟

- 主从之间的网络延迟过高，导致从库获取二进制日志速度变慢。
- 表现：
  - `SHOW SLAVE STATUS` 中 `Read_Master_Log_Pos` 和 `Exec_Master_Log_Pos` 差距较大。
  - 网络抖动或丢包会加剧延迟。

##### 2.4 大事务或批量操作

- 主库执行了大事务或批量操作（如批量 `INSERT`、`UPDATE`），从库需要更长时间重放这些操作。
- 例子：
  - 一次性插入数百万行数据或删除大量数据。

##### 2.5 锁争用

- 从库在执行重放日志时遇到锁争用，例如被其他长时间查询或事务阻塞。
- 表现：
  - 从库 `SQL_THREAD` 停滞或速度减慢。

##### 2.6 从库配置问题

- 参数设置不合理，例如 `slave_parallel_workers` 为 1（单线程复制）。
- 日志缓冲或磁盘刷写参数未优化（如 `sync_binlog` 或 `innodb_flush_log_at_trx_commit`）。
- 表现：
  - SQL 执行线程（`SQL_THREAD`）处理速度较低。

------

#### 3. **解决主从延迟的方法**

##### 3.1 优化主库写入

- 减少主库写操作的压力，例如通过分库分表或读写分离。
- 避免在主库执行大事务，拆分为小事务。

##### 3.2 提升从库性能

- **硬件升级**：提高从库的 CPU、内存或磁盘 IO 性能。

- **索引优化**：确保从库的查询和重放过程高效。

- 参数调整：

  - 使用多线程复制：

    ```
    SET GLOBAL slave_parallel_workers = 4; -- 配置并行复制线程数
    ```

  - 增大复制缓冲区：

    ```
    SET GLOBAL slave_pending_jobs_size_max = 134217728; -- 128MB
    ```

##### 3.3 网络优化

- 提高主从间网络带宽，优化网络延迟。

- 使用压缩传输二进制日志：

  ```
  [mysqld]
  binlog_compression = 1
  ```

##### 3.4 合理拆分事务

- 将大事务拆分为多个小事务，减少单次操作的重放时间。
- 示例：将 10 万行的 `INSERT` 拆分为多次每次 1000 行。

##### 3.5 优化锁争用

- 避免在从库上执行长时间查询。
- 设置 `slave_preserve_commit_order` 以确保并行复制的事务提交顺序。

##### 3.6 延迟复制（针对非实时场景）

- 配置延迟复制从库，以应对主库异常操作或误操作。

  ```
  CHANGE MASTER TO MASTER_DELAY = 3600; -- 延迟 1 小时
  ```

------

#### 4. **监控与分析工具**

- **慢查询日志**：分析从库慢查询，优化相关 SQL。
- 性能指标：
  - 使用 `SHOW SLAVE STATUS` 查看 `Seconds_Behind_Master`。
  - 通过监控工具（如 Prometheus + Grafana）实时监控主从延迟。

------

#### 5. **总结**

主从延迟的产生主要是由主库压力、从库性能瓶颈、网络问题和配置不合理导致。通过合理优化主库、提升从库性能、调整配置和网络优化等措施，可以有效减少主从延迟，保证数据的一致性和查询的实时性。



### 如何保证主从一致性

在 MySQL 主从复制架构中，确保主从数据一致性对于业务的稳定性至关重要。以下是保证主从一致性的一些方法和策略：

------

#### **1. 配置级别的优化**

##### **1.1 确保二进制日志完整性**

- 主库必须启用二进制日志（binlog），以便记录所有写操作：

  ```
  [mysqld]
  log_bin = mysql-bin
  binlog_format = ROW
  sync_binlog = 1
  ```

- 使用 **ROW 格式的日志**（推荐），避免语句级别（STATEMENT）复制带来的不一致问题。

##### **1.2 配置从库同步策略**

- 在从库中启用同步机制，确保事务安全：

  ```
  ini复制代码[mysqld]
  relay_log = relay-bin
  sync_relay_log = 1
  sync_relay_log_info = 1
  innodb_flush_log_at_trx_commit = 1
  ```

- **`innodb_flush_log_at_trx_commit=1`**：确保事务日志在每次提交时都持久化到磁盘。

##### **1.3 使用 GTID**

- 启用 GTID（全局事务标识符），确保事务的全局唯一性和可跟踪性：

  ```
  ini复制代码[mysqld]
  gtid_mode = ON
  enforce_gtid_consistency = ON
  ```

- 优点：

  - 避免复制过程中的数据丢失。
  - 简化故障切换操作。

------

#### **2. 复制过程的优化**

##### **2.1 使用半同步复制**

- 配置主从的 **半同步复制**，确保事务在主库提交时至少有一个从库确认收到二进制日志：

  ```
  INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
  INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
  
  SET GLOBAL rpl_semi_sync_master_enabled = 1;
  SET GLOBAL rpl_semi_sync_slave_enabled = 1;
  ```

- 半同步复制能有效减少数据丢失的风险。

##### **2.2 避免复制延迟**

- 使用并行复制提升从库的日志重放效率：

  ```
  slave_parallel_workers = 4
  slave_preserve_commit_order = 1
  ```

- 优化主库负载，减少大事务和复杂操作的影响。

##### **2.3 避免从库只读操作干扰**

- 设置从库为只读模式，防止非复制事务对数据造成污染：

  ```
  SET GLOBAL read_only = 1;
  ```

  - **注意**：需要保留复制用户的写权限。

------

#### **3. 操作流程的规范化**

##### **3.1 禁止主库误操作**

- 开启数据防护，防止直接操作主库导致数据不一致：
  - 配置 `binlog_format=ROW` 确保复杂操作不带来问题。
  - 使用事务，避免中间状态数据对外暴露。

##### **3.2 监控复制状态**

- 定期检查从库复制状态：

  ```
  SHOW SLAVE STATUS\G;
  ```

  - 关注 `Seconds_Behind_Master` 和 `Slave_IO_Running` / `Slave_SQL_Running`。

##### **3.3 恢复复制一致性**

- 如果主从数据不一致：
  - 停止从库复制线程：`STOP SLAVE;`
  - 校验数据差异（如使用 `pt-table-checksum`）。
  - 同步数据（如使用 `pt-table-sync` 或全量导出导入）。

------

#### **4. 监控与工具支持**

##### **4.1 使用自动化工具**

- Percona Toolkit：
  - `pt-table-checksum`：检测主从数据不一致。
  - `pt-table-sync`：修复主从数据不一致。
- MHA（Master High Availability）：
  - 在主库故障时快速切换主从，避免数据不一致。

##### **4.2 实时监控**

- 使用监控工具（如 Prometheus + Grafana）：
  - 监控复制延迟。
  - 监控主从同步状态。

------

#### **5. 高级策略**

##### **5.1 双主架构中的一致性**

- 双主架构中，避免双写引发的数据冲突：
  - 明确主写方向，防止循环复制。
  - 设置 `read_only` 和 `super_read_only` 限制辅助主库。

##### **5.2 配置延迟从库**

- 使用延迟复制（MASTER_DELAY）：

  - 应对主库误操作导致的数据问题。

  - 配置示例：

    ```
    CHANGE MASTER TO MASTER_DELAY = 3600;
    ```

------

#### **总结**

要保证 MySQL 主从复制的一致性，可以从 **配置优化**、**复制机制**、**操作流程规范化** 和 **监控维护** 等方面入手。同时，利用工具（如 Percona Toolkit 和 MHA）进行数据校验和高可用管理，可以有效降低数据不一致的风险。





### 为什么开启了 binlog 还需要 GTID？

在 MySQL 主从复制中，**开启 binlog（二进制日志）** 和 **开启 GTID（全局事务标识）** 解决的是不同的问题：

1. **Binlog（二进制日志）**
   - 作用：记录数据库的所有更改（如 `INSERT`、`UPDATE`、`DELETE` 等），用于数据恢复和主从复制。
   - 复制方式：默认情况下，主库的 binlog 通过 **基于文件和位置（File+Position）** 的方式让从库进行复制。
   - 存在问题：
     - 需要手动管理 `binlog` 的文件名和位置（`MASTER_LOG_FILE` 和 `MASTER_LOG_POS`）。
     - 进行主从切换（failover）时，可能要重新指定复制位点，管理复杂。
2. **GTID（全局事务标识）**
   - 作用：每个事务分配一个唯一的 GTID（`UUID:事务编号`），在主从复制时保证事务的唯一性和一致性。
   - 复制方式：**基于 GTID**，从库只需要知道事务的 GTID，而不需要手动指定 `binlog` 位置。
   - 优势：
     - **简化主从复制**：无需手动管理 `binlog` 文件和位置，直接使用 `CHANGE MASTER TO MASTER_AUTO_POSITION=1;`。
     - **支持自动故障转移**：GTID 使得主从切换更容易，适合高可用架构（如 MHA、Orchestrator）。
     - **防止重复执行事务**：GTID 机制可自动跳过已执行的事务，避免主从不一致。

为什么开启了 binlog 还需要 GTID？

- **binlog 只是记录事务日志，GTID 则是唯一标识每个事务，二者是互补的**。
- **GTID 提高了主从复制的灵活性和可靠性**，特别是在发生主从切换或故障恢复时，可以更轻松地管理复制关系。

#### 结论：

如果只开启 binlog，而不使用 GTID，主从复制依赖 `binlog` 位置，切换主库较为复杂。开启 GTID 后，复制管理变得更简单，特别适用于高可用场景。因此，在 MySQL 5.6 及以上版本，推荐同时开启 `binlog` 和 `GTID`，以提升复制的灵活性和可靠性。







20250316
