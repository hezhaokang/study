# 文件管理和IO重定向

## 文件系统

### 概述

- 一句话

	- linux下一切皆文件

- 文件系统

	- 文件系统

		- 侧重于理论梳理
		- 操作系统用于组织和存储文件及数据的方法和数据结构。它定义了数据如何在存储设备（如硬盘）上被组
	织、命名、访问和修改。

	- 文件管理系统

		- 侧重于各种文件的操作实践
		- 可以理解为操作系统中与文件管理相关的所有功能和工具的集合

### 目录结构

- 绝对路径&相对路径

	- 绝对路径

		- 以正斜杠/ 即根目录开始,是一个完整的文件的位置路径。可用于指定任何一个文件的时候
示例：/path/to/dir/file.txt

	- 相对路径

		- 不以斜线开始，是指相对于当前工作目录的路径。特殊场景下，是相对于某目录的位置可以作为一个简短的形式指定一个文件名
示例：current_path/to/dir/file.txt

- 目录拆解

	- 用户相关的目录

		- /home

			- 普通用户的主目录。每个用户都有自己的家目录，一般是由用户账号命名的。

		- /root

			- root用户的主目录，或者超级管理员的家目录

	- 系统相关的目录

		- /boot

			- 这里存放的是启动 Linux 时使用的一些核心文件，包括一些连接文件以及镜像文件。

		- /etc

			- etc 是 Etcetera(等等) 的缩写,这个目录用来存放所有的系统管理所需要的配置文件和子目录。

		- /lib

			- lib 是 Library(库) 的缩写这个目录里存放着系统最基本的动态连接共享库，应用程序都需要用到这些共享库。

		- /sys

			- 这是 Linux2.6 内核的一个很大的变化。该目录下安装了 2.6 内核中新出现的一个文件系统 sysfs。
sysfs 文件系统是内核设备树的一个直观反映它集成了下面3种文件系统的信息：
 - 针对进程信息的 proc 文件系统
 - 针对设备的 devfs 文件系统
 - 针对伪终端的 devpts 文件系统。

	- 命令相关的目录

		- /bin

			- 这个目录存放着最经常使用的命令。

		- /sbin

			- 这里存放的是系统管理员使用的系统管理
	程序

		- /usr/bin

			- 系统用户使用的应用程序。

		- /usr/sbin

			- 超级用户使用的比较高级的管理程序和系统守护程序。

	- 程序相关的目录

		- /proc

			- proc 是 Processes(进程) 的缩写，/proc 是一种伪文件系统（也即虚拟文件系统），存储的是当
	前内核运行状态的一系列特殊文件，这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访
	问这个目录来获取系统信息。

		- /srv

			- 该目录存放一些服务启动之后需要提取的数据

		- /usr

			- usr 是 unix shared resources(共享资源)，这是一个非常重要的目录，用户的很多应用程序和文
	件都放在这个目录下，类似于 windows 下的 program files 目录。

		- /usr/src

			- 内核源代码默认的放置目录。

		- /var

			- 这个目录中存放着在不断扩充着的东西，主要用于那些经常被修改
	的场景

		- /run

			- 是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除

		- /usr/share

			- 应用程序软件的帮助信息文件所在目录

	- 设备相关的目录

		- /dev

			- 该目录下存放的是 Linux 的外部设备

		- /media

			- linux 系统会自动识别一些设备，例如U盘、光驱等等，当识别后，Linux 会把识别的设备挂载到这个目录下。

		- /mut

			- 系统提供该目录是为了让用户临时挂载别的文件系统的，简单来说是给光驱使用的。

	- 其他相关目录

		- /lost+found

			- 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件

		- /opt

			- opt 是 optional(可选) 的缩写，这是给主机额外安装软件所摆放的目录。

		- /selinux

			- 这个目录是 Redhat/CentOS 所特有的安全机制目录，类似于 windows 的防火墙目录

		- /tmp

			- tmp 是 temporary(临时) 的缩写这个目录是用来存放一些临时文件的。

### 类型解读

- 系统类型

	- FAT

		- FAT是一种简单且广泛使用的文件系统，主要包括FAT12、FAT16和FAT32等版本。
 它使用文件分配表来管理文件的位置和状态，具有跨平台性和良好的兼容性，几乎支持所有主流的操作系
			统。
			 然而，FAT文件系统不支持高级的文件权限控制。

			- FAT32常用于U盘、存储卡等移动设备

	- NTFS

		- NTFS是Windows操作系统的标准文件系统，支持高级功能，如文件权限、加密、压缩和磁盘配额。
 它提供了更高的安全性和稳定性，支持大文件和分区，并具有文件系统日志，有助于防止数据丢失。

			- NTFS最适合用于Windows操作系统的硬盘分区，以及需要高级文件管理功能的场景

	- ext4

		- ext4是Linux系统中广泛使用的文件系统，它继承了ext3的优点，并在性能和稳定性方面有所提升。
 ext4支持大文件和大容量存储设备，具备日志功能以防止数据损坏，并支持延迟分配以减少文件碎片。

			- ext4最适合用于Linux系统的硬盘分区，以及需要高性能和稳定性的服务器环境

	- XFS

		- XFS是一个高性能的64位文件系统，最初由SGI开发，用于高负载服务器和数据存储。
		 它支持大文件和高并发的读写操作，具有动态空间分配和高效的空间管理能力

			- XFS最适合用于需要处理大量数据的高性能服务器和数据存储系统

- 文件类型

	- 普通文件

		- 这是最常见的文件类型，包含文本、数据、程序代码等

			- 可以使用cat、less、more等命令来查看内容，使用cp、mv、rm等命令来管理

	- 目录文件

		- 目录用于存储其他文件的路径名和相关信息，被视为一种特殊的文件

			- 可以使用mkdir、rmdir来创建和删除目录，使用cd命令来切换当前工作目录

	- 链接文件

		- 链接文件分为硬链接（Hard Links）和软链接（Symbolic Links，也称为符号链接）。
		-  硬链接指向文件的inode节点，允许多个文件名与同一个文件关联。
		-  软链接是一个特殊的文件，它指向另一个文件或目录的路径，可以跨文件系统。

	- 特殊文件

		- 特殊文件主要指的是设备文件，它们通常位于/dev目录下。
		-  设备文件分为块设备文件（block devices）和字符设备文件（character devices）。
		-  块设备文件以块为单位进行读写，如硬盘、光驱等；字符设备文件以字符为单位进行读写，如键盘、鼠标等

	- 可执行文件

		- 可执行文件是包含程序代码的文件，可以被操作系统直接执行。
		-  在Linux中，可执行文件通常需要具有执行权限（x）

	- 其他文件

		- 除了上述几种类型外，Linux系统中还可能存在其他类型的文件，
		-  如管道文件（pipe files，显示为青黄色），管道文件主要用于进程间通讯。
		-  套接字文件（socket files，有时显示为粉红色），用于进程间的网络通信，也可以用于本机之间的非网络通信

- 字体颜色

	- 蓝色：表示目录文件。
	- 白色：表示普通文件。
	- 浅蓝色（或青蓝色）：表示链接文件。
	- 绿色：表示可执行文件。
	- 红色：表示压缩文件或归档文件（如tar包）。
	- 黄色：表示设备文件。
	- 红色闪烁：表示链接文件有问题或无法访问。

## 文件管理

### 文件操作

- 查看文件

	- ls 目标目录位置
	- tree 目标目录位置

		- tree -L n   n 替换为你想要显示的最大目录层级数

			- tree -L 1 将只显示当前目录下的内容，不包括子目录中的内容

- 创建目录

	- mkdir [-p] 目录位置

- 创建文件

	- touch 文件名称

- 切换目录

	- cd 目标目录位置

		- . 代表当前目录
		- .. 代表上一级目录
		- - 回到上个目录

- 删除文件

	- rm -f  普通文件
	- rm -rf 目录文件

### 获取信息

- pwd 查看当前所在路径
- ls 查看目录下有多少文件

	- ls -a  包含隐藏文件
	- ls -l  显示额外的信息
	- ls -R  查看所有目录文件信息
	- ls -ld  目录和符号链接信息
	- ls -h   文件的大小会根据其大小显示为适当的单位

### 文件信息

- file查看文件类型

	- file -b 只输出文件类型，不包括文件名

- stat查看文件状态

- ```
  ls 文件名			#只查看文件目录
  ls -l 文件名		#查看文件信息带权限
  
  drwxr-xr-x 2 root root 4096 Oct 26 19:23 a
  "d"				#文件类型
  "rwxr-xr-x 2 root root 4096 Oct 26 19:23"	#文件属性
  "a"				#文件名
  
  ```

  

- 
### 文件存储

- inode，即索引节点，是文件系统中的一个数据结构，它包含了文件的元数据，但不包括文件名或文件数据本身

- ![](D:\桌面\mage.md\文件管理和IO重定向\assets\47a8718092d7246084fc5cca16de3dfac36d16e0df8f98e731d146b2149c76cd.png)

- ```
  当我们查看/data/file_type/directory/dir1/test.txt文件数据时。
   - Linux将在根目录文件中找到/data这个目录文件的inode编号
   - 然后根据inode合成/data的数据块。
   - 随后，根据数据块中的记录，找到file_type/的inode编号
   - 然后根据inode合成file_type/的数据块。
   - 随后，根据数据块中的记录，找到directory/的inode编号
   - 然后根据inode合成directory/的数据块。
   - 随后，根据数据块中的记录，找到dir1/的inode编号
   - 然后根据inode合成dir1/的数据块。
   - 随后，根据数据块中的记录，找到test.txt的inode编号
   - 然后根据inode合成test.txt的数据块。
  整个过程中，我们参考了六个inode：根目录文件、data目录文件、file_type目录文件、directory目录
  文件、datadir1目录文件、text.txt文件的inodes。
  ```

  

### 文件编辑

- nano

- vi

	- vi | vim [文件名]

		- a | i | o ： 进入编辑文件模式。
		- Esc ： 退出编辑模式。
		- :wq ： 保存文件。
		- :q ： 撤销保存文件。

- vim

  vi | vim [文件名]

  - a | i | o ： 进入编辑文件模式。
  - Esc ： 退出编辑模式。
  - :wq ： 保存文件。
  - :q ： 撤销保存文件。

- ![image-20241026204314603](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204314603.png)

- ![image-20241026204321219](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204321219.png)

### 文件格式

- Windows创建的txt文件在Linux系统中无法直接使用，主要是由于两个系统在处理文本文件时存在的一些格式差异，尤其是换行符的差异
- 检测命令

  - ```
  	hexdump [参数] [文件名]
  	```
  	
  	- -C：以规范的十六进制和ASCII字符的混合格式显示文件内容，这是最常用的选项之一。

- 转换命令

	- dos2unix是一个在Linux系统中广泛使用的命令行工具，主要用于将Windows风格的文本文件格式转换为Unix/Linux风格的文本文件格式，从而解决在不同操作系统之间传输文本文件时可能出现的格式不兼容问题。

		- ```
			dos2unix [参数] [文件]
			```
			
			- -n：不覆盖源文件，将转换后的内容保存到一个新文件中。
			- -o：指定输出文件的名称（通常与-n选项一起使用）。

### 文字符号

- *

	- 匹配零个或多个字符，但不匹配 "." 开头的文件

- {1..4}

	- 表示 a-z 范围的所有内容

- [0-9]

	- 匹配数字范围

- [a-z]

	- 相当于a-z + A-Z 范围的一个字母

- [^a-z]

	- 匹配列表中的所有字符以外的字符

- ~

	- 当前用户家目录

### 文件复制

- cp [选项] 源文件 目标文件

	- -r 或 -R：递归复制。用于复制目录及其包含的所有子目录和文件。 
	- -a：归档复制。相当于-d、-p、-r选项的集合，
	-  -i：交互式复制。在覆盖已存在的文件前提示用户确认。
	- -v：详细输出。显示详细的复制进度信息。
	- -p：保留文件属性。复制文件时保留源文件的属性（如修改时间、所有权、权限等）。

### 转移&改名

- mv 源文件或目录 目标文件或目录

	- 如果源文件和目标文件在同一目录，表明该操作是 改名
	- 如果源文件和目标文件不在同一目录，表明该操作是 转移

- rename '文件名关键字' '替换内容' 文件列表

    ![image-20241026204454911](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204454911.png)

  ![image-20241026204501672](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204501672.png)

  ![image-20241026204508875](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204508875.png)![image-20241026204508881](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204508881.png)
  
  ```
  root@ubuntu:a $ ls
  a  b  c  d  e
  root@ubuntu:a $ rename s/\$/\.txt/ *
  root@ubuntu:a $ ls
  a.txt  b.txt  c.txt  d.txt  e.txt
  root@ubuntu:a $ touch {1..5}
  root@ubuntu:a $ ls
  1  2  3  4  5  a.txt  b.txt  c.txt  d.txt  e.txt
  root@ubuntu:a $
  root@ubuntu:a $ rename s/$/.txt/ {1..5}
  root@ubuntu:a $ ls
  1.txt  2.txt  3.txt  4.txt  5.txt  a.txt  b.txt  c.txt  d.txt  e.txt
  ```
  
  

### 连接文件

![image-20241026204553038](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204553038.png)

- 硬链接

  - 硬链接是通过文件系统的inode（索引节点）连接来产生新文件名的方式，而不是创建新文件。这意味着多个文件名可以指向同一个inode号，因此它们共享相同的文件数据

  	- ln 源文件 目标文件
  	- ![image-20241026204526639](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204526639.png)
  	- ![image-20241026204533215](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204533215.png)

- 软连接

  - 符号链接是一个独立的文件，它包含了另一个文件的路径名。当访问符号链接时，系统会根据链接中的路径名找到并访问实际的文件

  	- ln -s 源文件 目标文件
  	- ![image-20241026204541991](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204541991.png)

- readlink 链接文件

	- 查看链接指向的文件

### 案例解读

- MongoDB服务启动异常

  - 案例1：mongo服务异常，重启后依然失败，查看报错日志 No space left on device

  - ![image-20241026204618874](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204618874.png)

  - ![image-20241026204631082](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204631082.png)

  			- 1 删除不用的文件和文件夹，释放被占用的inode
  			- 2 格式化新的磁盘，指定一个大一点的inode值
  			- 3 迁移部分数据到新的磁盘上。
  			- 4 根据服务配置文件的属性配置，对新磁盘的目录结构进行软连接配置，避免大规模数据迁移导致的配置路径异常。

- Jenkins应用异常

  - 案例2：jenkins运行的时候，发现jenkins执行项目任务报错 No space left on device
  - ![image-20241026204642707](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204642707.png)
  - ![image-20241026204650299](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204650299.png)
  - ![image-20241026204657343](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204657343.png)

- 软件安装异常

- ![image-20241026204708267](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204708267.png)

		- 1 格式化磁盘
		- 2 创建新的pv
		- 3 将新的pv添加到vg组中
		- 4 扩容逻辑磁盘
		- 5 重载逻辑磁盘
		- 6 测试后，问题解决

## IO实践

### 信息传递

- 重定向

	- > 表示将符号左侧的内容，以覆盖的方式输入到右侧文件中
	-  < 表示将符号右侧的内容，以覆盖的方式输入到左侧文件中
	- >> 表示将符号左侧的内容，以追加的方式输入到右侧文件的末尾行中
	-  << 表示将符号右侧的内容，以追加的方式输入到左侧文件的末尾行中

- 管道符

	- | 这个就是管道符，常用于将两个命令隔开，然后命令间(从左向右)传递信息使用的。
	- 命令1 | 命令2

		-  管道符左侧命令1 执行后的结果，传递给管道符右侧的命令2使用

### 终端输出

- & 就是将一个命令从前台转到后台执行,使用格式

	-  命令 &

- Linux给程序提供三种 I/O 设备

	- 标准输入（STDIN） -0   默认接受来自终端窗口的输入
	- 标准输出（STDOUT）-1   默认输出到终端窗口，正确的输出信息
	- 标准错误（STDERR）-2   默认输出到终端窗口，错误的输出信息
	- 2>&1 代表所有输出的信息,也可以简写为 "&>" 或者 ">&"

		- COMMAND > /path/to/file.out 2>&1

### 合并输出

![image-20241026204735674](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204735674.png)

```
#查看当前shell的pid
echo $BASHPID
11413
ps aux | grep bash
#root        1485  0.0  0.2   8780  5504 pts/0    Ss   15:42   0:00 -bash
#root        1626  0.0  0.1   6544  2304 pts/0    S+   16:31   0:00 grep --color
(echo $BASHPID; echo haha)
#11661
#haha
(echo $BASHPID; sleep 30)
#11711
```



- 临时shell环境 - 启动子shell

	-  (命令列表)，在子shell中执行命令列表，退出子shell后,不影响后续环境操作

- 临时shell环境 - 不启动子shell

	-  {命令列表}, 在当前shell中运行命令列表,会影响当前shell环境的后续操作。
	
- ()

  ```
  (cd /tmp;pwd)
  #/tmp
  pwd
  #/root
  ```

  ```
  (cal 2019 ; cal 2022) > all.txt
  cat all.txt
  #
                              2019
        January               February               March
  Su Mo Tu We Th Fr Sa  Su Mo Tu We Th Fr Sa  Su Mo Tu We Th Fr Sa
         1  2  3  4  5                  1  2                  1  2
   6  7  8  9 10 11 12   3  4  5  6  7  8  9   3  4  5  6  7  8  9
  13 14 15 16 17 18 19  10 11 12 13 14 15 16  10 11 12 13 14 15 16
  20 21 22 23 24 25 26  17 18 19 20 21 22 23  17 18 19 20 21 22 23
  27 28 29 30 31        24 25 26 27 28        24 25 26 27 28 29 30
                                              31
  ```

- {}

  ```
  { cd /tmp;pwd; }
  #/tmp
  pwd
  /tmp
  ```
  
  ```
  { ls;hostname;} > /tmp/all.log
  #tee-file
  #ubuntu
  ```

### EOF

### cat

- cat > file  <<- EOF
	    ...
	EOF

	注意：

​			<	代表覆盖式增加内容到 file 文件

​			<<	代表追加式增加内容到 file 文件

### tee

- tee命令读取标准输入，把这些内容同时输出到标准输出和（多个）文件中，tee命令可以重定向标准输出到多个文件。要注意的是：在使用管道线时，前一个命令的标准错误输出不会被tee读取。

	- ```
		tee [选项] 文件名
		```
		
		- 输出到标准输出的同时，追加到文件file中。如果文件不存在则创建；如果文件存在则追加。
		
		- ```
		  tee					#只输出到标准输出
		  tee file			#输出到标准输出的同时，保存到文件file中
		  
		  tee -a file
		  tee host2 <<- EOF ... EOF
		  					#输出到标准输出的同时，追加到文件file中。如果文件不存在则创建；如果文件存在则追加。
		  tee -				#输出到标准输出两次。
		  tee file1 file2 -	#输出到标准输出两次，同时保存到file1和file2中。
		  ```
	- ![image-20241026204932360](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20241026204932360.png)

### read

- read	命令可以实现我们脚本内外的信息自由传递功能。
- ```
   	read 从标准输入读取一行并赋值给特定变量REPLY。
      read answer 从标准输入读取输入并赋值给变量answer。
      read first last 从标准输入读取内容，将第一个单词放到first中，其他内容放在last中。
      read -s passwd 从标准输入读取内容，写入passwd，不输入效果
      read -n n name 从标准输入读取内容，截取n个字符，写入name，超过n个字符，直接退出
      read -p "prompt" 打印提示，等待输入，并将输入存储在REPLY中。
      read -r line 允许输入包含反斜杠。
      read -t second 指定超时时间,默认是秒，整数
      read -d sper 指定输入信息的截止符号
  ```

  
### 高级重定向

- 高级样式1：cmd <<< "string"

  - 表示将后面的"string"内容，作为stdin传给cmd，有cmd进行后续处理

  - ```
    高级样式2：cmd1 < <(cmd2)
    	<(cmd2)   表示把cmd2的输出写入一个临时文件
    		注意：<和(之间无空格
    	cmd1 <    这是一个标准的stdin重定向
     	把两个合起来，就是把cmd2的输出stdout传递给cmd1作为输入stdin, 中间通过临时文件做传递
    ```

- tr命令用于替换字符串

 ```
 tr '原内容' '信内容' <<< "字符串"
 ```

 
