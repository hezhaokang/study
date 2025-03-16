# linux基础入门和帮助

## 1、linux基础

### 1、登录方式

- 终端
- 图形
- ssh

### 2、Linux用户

- root用户
- 普通用户
- 管理员用户
- 命令 su 可以切换当前用户身份到另外一个用户
命令 exit 可以回退到刚才的那个用户身份

### 3、linux交互

- tty

	- 可以查看当前所在终端

### 4、shell基础

- 系统变量

	- $PS1

		- 定义主提示符

	- $SHELL

		- 当前用户的默认shell值

	- $PATH

		- 在执行命令时应该在哪里查找可执行文件

	- $BASHPID

		- 当前Bash进程的进程ID（PID)

- cat /etc/shells

	- 查看当前系统环境支持的shell

- chsh sswang -s /bin/sh

	- 更改系统shell

- eval

	- 是一个在许多shell中（如Bash、Zsh等）可用的内置命令，它的功能是读取参数作为shell命令，然后执行这些命令。当你使用 eval 时，它会将传递给它的参数首先进行参数扩展和命令替换，然后再执行结果字符串作为shell命令

### 5、终端样式

- /etc/issue

	- 登录前提示文件

- /etc/motd

	- rocky登陆后提示文件 

- /etc/bashrc

	- 在里面保存修改PS1，修改命令行前置颜色显示内容

### 6、Linux命令

- type 确定一个命令是不是内部命令

	- -a 查看更详细的信息提示

- help 查看内部命令列表
- whereis 查看命令是否存在，若存在显示所有位置

	- 查看外部命令存放路径，除了命令外，还显示和命令相关的帮助文档等文件路径

- which 查看命令是否存在，若存在显示第一个显示位置

	- 查看外部命令存放路径

- which命令找到相关的二进制程序是否已经在搜索路径中
whereis， 该命令会搜索shell的搜索路径之外更大范围的系统目录
- man 查看命令帮助
- 命令默认存在目录

	- $PATH 获取默认命令能够存在的目录

- 命令简化操作

	- alias

		-   为一个命令定义别名，提高命令的操作效率

	- unalias

		- 为一个命令取消别名

- 开机关机命令

	- 重启

		- shutdown -h now

	- 关机

		- shutdown -r

### 7、作用范围

- /etc/profile
/etc/profile.d/xxx.sh

	- 系统级别的环境控制文件，默认只有系统启动的时候，才会加载

- /etc/bashrc

	- 系统用户级别的环境控制文件，默认开启一个终端登录就会加载

- ~/.bashrc | ~/.bash

	- 用户终端级别的环境控制文件，切换一个终端用户就会加载

- /path/to/file.sh

	- 用户终端+文件级别的环境控制文件-执行文件的时候才会加载

- 特殊的命令作用范围-子shell
    () 代表当前shell范围里面生成一个独立的shell，可以开启一个独立的进程

	- $() 代表开启一个独立的进程，执行()范围的可执行命令，然后将信息返回给当前的shell

		- 注意；如果()范围内的命令执行结果内容，不是可执行命令的话，最好用下面的方法来输出

			- echo$(可执行命令)

### 8、软件安装

- centos

	- yum

		- 更新软件源

			- yum makecache

		- 搜索软件园源

			- yum search <软件名>

		- 安装软件源

			- yum [-y] install <软件名>

		- 删除软件源

			- yum [-y] remove <软件名>

	- dnf

		- 更新软件源

			- dnf makecache

		- 搜索软件园源

			- dnf search <软件名>

		- 安装软件源

			- dnf [-y] install <软件名>

		- 删除软件源

			- dnf [-y] remove <软件名>

- ubuntu

	- apt

		- 更新软件源

			- apt update

				- 命令会从您指定的源下载最新的软件包列表

			- apt upgrade

				- 命令会更新您的系统上的所有软件包到最新版本

		- 搜索软件园源

			- apt search <软件名>

		- 安装软件源

			- apt [-y] install <软件名>

		- 删除软件源

			- apt [-y] remove <软件名>

## 2、常见信息获取

### 变量信息

- 变量名=值

	- 定义变量

- echo $变量名

	- 查看变量

- env

	- 查看系统默认的变量

- man bash

	- 查看系统默认的变量

- unset my_var

	- 变量my_var已经被删除

- echo $BASH

	- 查看当前是shell名字

- echo $BASHPID

	- 查看当前bash会话的id

- echo $BASH_VERSION

	- 查看当前bash的般本

- echo $MACHTYPE

	- 查看当前系统架构信息【GNU风格】

- echo $HISTFILE

	- 查看当前用户的历史命令文件

- echo $LANG

	- 查看当前系统的语言类型

### 登录信息

- last

	- 显示上次登录的用户列表信息

- whoami 

	- 查看当前登录的是那个用户

- id 

	- 用于查看当前登录用户的全面信息，包括归属的用户管理组信息

- su

	- 可以切换当前用户身份到另外一个用户

- exit

	- 可以回退到刚才的那个用户身份

- pwd

	- 显示当前工作路径

- who

	- 显示当前系统登录的用户

- w

	- 查看当前登录用户的启动程序信息

### 系统信息

- lscpu

	- 查看cpu信息
	- cat /proc/cpuinfo

- lsblk、df

	- 查看磁盘分区信息
	- cat /proc/partitions

- free

	- 查看mam信息
	- cat /proc/meminfo

- blkid

	- 查看磁盘设备信息

- arch

	- 查看系统架构

- uname 

	- -a

		- 获取完整信息

	- -s

		- Linux

	- -n

		- rocky9

	- -r

		- 5.14.0-427.13.1.el9_4.x86_64

	- -v

		- #1 SMP PREEMPT_DYNAMIC Wed May 1 19:11:28 UTC 2024

	- -m

		- x86_64

	- -p

		- x86_64

	- -o

		- GNU/Linux

- /etc/redhat-release[Rocky系统]、/etc/os-release[两个都支持]、/etc/lsb-release[Ubuntu
系统]
- 查看操作系统发行版本

### 日期时间

- ```
  date					#显示和设置系统时间
  clock、hwclock			#显示和设置硬件时间
  
  文件：/etc/localtime、/etc/timezone
  命令：timedatectl
  						#查看时区的问题
  cal –y					#日历信息查看
  timedatectl status	#查看当前时区信息
  timedatectl set-timezone Africa/Ceuta
  						#修改时区
  timedatectl list-timezones
  						#查看系统支持的时区
  timedatectl set-timezone Asia/Shanghai
  						#修改时区信息
  timedatectl			#查看当前时区信息
  ```

  


### 补全功能

- ```
  Tab
  ```

### 历史命令

- history

  - ```
    -c
    ```

    - 删除所有条目从而清空历史列表

  - ```
  	-d 偏移量 
  	```
  	
  	-  从指定位置删除历史列表。负偏移量将从历史条目末尾开始计数

- 上下键

### linux快捷键

- ```
  Ctrl + A
  ```

  - 光标迅速回到行首

- ```
  Ctrl + E
  ```

  - 光标迅速回到行尾

- ```
  Ctrl + C
  ```

  - 临时终止命令行命令

- ```
  Ctrl + b
  ```

  - 移动到当前单词的开头

- ```
  Ctrl + F
  ```

  - 移动到当前单词的结尾

- ```
  !!
  ```

  ​		执行上一条命令

- ```
  !1
  ```

  - 执行历史命令中的第1行命令

- ```
	Ctrl+关键字
	```
	
	- 执行内容匹配的命令

## 3、会话管理

### 会话基础

- ```
  ps aux
  ```

  - 查看程序进程信息

- ```
  kill
  ```

  - ```
    kill -9 PID号
    ```

    - 强制关闭进程

  - - ```
  	  kill -15 PID号
  	  ```
  	
  	  普通关闭进程

- ```
  pstree -p
  ```

  - 查看当前ssh程序所关联的进程信息

- 执行命令 | grep 关键字

	- 过滤相关信息

- ```
  ps aux | grep ssh
  ```

  - 查看当前ssh连接的进程

- ```
	echo $BASHPID
	```
	
	- 查看终端的 bash进程id

### 会话解绑

- 类似的终端复用器还有Screen，Tmux

  - - ```
  	  dnf install -y epel-release
  	  dnf install -y screen
  	  dnf install -y tmux 
  	  ```
  	
  	  安装复用器

- screen

	- ```
	  screen -S 会话名
	  ```

	  - 创建一个独立会话

	- ```
	  screen -x 会话名
	  ```

	  - 加入screen会话

	- ```
	  exit
	  ```

	  - 退出并保存会话

	- ```
	  Ctrl +a,d
	  ```

	  - 剥离当前screen会话：
	
	- ```
	  screen -ls
	  ```
	
	  - 显示所有会话：
	
	- ```
		screen -r 会话名
		```
		
		- 恢复一个会话

## 4、输出格式化

### echo解读

- echo命令的功能是将内容输出到默认显示设备，一般起到一个提示的作用

	- ```
	  -n
	  ```

	  - 不要在最后自动换行

	- ```
		-e
		```
		
		- 若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出:

### 字体颜色

- - ```
	  echo -e "\033[字背景颜色;文字颜色m字符串\033[0m"
	  
	  - echo -e "\033[41;36m 显示的内容 \033[0m"
	  ```
	
	- ```
	  echo -e "\033[44;37m 显示的内容 \033[0m"
	  	"-e" 选项开启反斜杠转义
	  	"\033["：转义符等同于\E或\e
	  	"44;37"：字体色；背景色
	  	"m"：转义终止符
	  	"显示的内容"：要显示的内容
	  	"\033[0m"：恢复最初配色
	  ```
	
	  
	
	- | 色彩 | 字体色 | 背景色 |
	  | ---- | ------ | ------ |
	  | 黑   | 30     | 40     |
	  | 红   | 31     | 41     |
	  | 绿   | 32     | 42     |
	  | 黄   | 33     | 43     |
	  | 蓝   | 34     | 44     |
	  | 紫   | 35     | 45     |
	  | 青   | 36     | 46     |
	  | 灰   | 37     | 47     |
	
	- ```
	  最后控制选线说明
	  	\033[0m		#关闭所有属性
	  	\033[1m		#设置高亮度
	  	\033[4m		#下划线
	  	\033[5m		#闪烁
	  	\033[7m		#反显
	  	\033[8m		#消隐
	  注意：
	  	\033是八进制的ESCAPE字符，也可以用\e代替
	  ```
	
	  

### printf格式化

- printf 命令模仿 C 程序库（library）里的 printf() 程序。由于 printf 由 POSIX 标准所定
	义，因此使用 printf 的脚本比使用 echo 移植性好。
	 printf 使用引用文本或空格分隔的参数，外面可以在 printf 中使用格式化字符串，还可以制定字符
	串的宽度、左右对齐方式等。默认 printf 不会像 echo 自动添加换行符，我们可以手动添加 \n。

	- ```
	  printf [-v var] 格式 [参数]
	  ```
	
	  ```
	  peintf "姓名:%s,語文:%f米,數學:%f公斤" "張三" 77 66
	  #姓名：張三，語文：77，數學：66
	  ```

## 5、自学方法

### 命令帮助

- 通过 --help 参数来获取命令帮助信息

  - ```
    type --help
    ```

- 使用whatis 是一个用于查询命令简短描述的实用工具。当你想要快速了解某个命令的基本用途时，可以使用 whatis 命令紧跟着你想查询的命令名

  - 使用whatis命令：
   使用whatis命令来借助linux内部的命令数据库文档，来显示命令的简单描述

  	- ```
  	  whatis ls
  	  ```
  	
  	  

- man 命令是“手册”（manual）的缩写，它用于显示命令、函数、文件格式或其他文档的手册页（man page）。手册页是软件文档的一种形式，通常包含了关于如何使用特定命令、配置文件或编程接口的详细信息。

	- ```
	  man type
	  ```
	
	  

### man手册

- man手册位于 /usr/share/man 目录下
- ```
  man -a keyword
  ```

  - 列出所有帮助

- ```
  man -k keyword
  ```

  - 搜索man手册所有匹配的页面

- ```
	man -w keyword
	```
	
	- 打印keyword的man帮助文件的路径

### 软件文档

- 在linux系统中，默认安装的各种软件和命令的帮助文档，默认都是在 /usr/share/doc 目录下

### 精简帮助

- TLDR

	- https://github.com/tldr-pages/tldr
	- 185.199.111.133 raw.githubusercontent.com
 在国内访问测试的事后，如果不对网络进行设置的话，有可能会出现一些超时中断的问题







20250316
