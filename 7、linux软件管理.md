







# 1、软件管理

## 1.1 软件相关概念

### 1.1.1ABI

接口简介

ABI简介

```
ABI（Application Binary Interface，应用程序二进制接口）是一种定义了应用程序与操作系统
或者硬件之间交互方式的接口标准，不同的操作系统，他的ABI接口是不一样的。每个种操作系统，都有对外的ABI接口，软件要想运行，就必须符合其接口规范才可以。它为开发人员提供了在不同平台上编写、编译和执行应用程序的一致性。
```

```
ABI在调用操作的时候，主要涉及到如下几个方面：
 - 数据结构、数据类型等数据信息
 - 底层函数调用的过程
 - 目标文件的二进制格式
 - 系统调用依赖的函数库
 - 平台兼容性
```

```
简单来说，应用程序执行 就是 让操作系统去执行一个 程序二进制文件。
 - 而这个二进制文件在编译的时候，一定要生成统一的信息格式，才有可能让操作系统执行。
```

### 1.1.2 API

API简介

```
API 即 Application Programming Interface，API可以在各种不同的操作系统上实现给应用程
序提供完全相同的接口，而它们本身在这些系统上的实现却可能迥异，主流的操作系统有两种，一种是Windows系统，另一种是Linux系统。
```

早期编码

```
	由于操作系统的不同，API又分为Windows API和Linux API。在Windows平台开发出来的软件在
Linux上无法运行，在Linux上开发的软件在Windows上又无法运行。
 	所以，早期的程序员开发程序的时候，需要在不同的平台，需要使用各自平台的编程语言，将同一个程序逻辑，各自编写一份，这就导致程序的开发效率非常的低下。
 	而对于赶工期，赶出来的程序代码，又无法进行平台的软件移植，那么为了解决程序开发的痛苦，POSIX 标准的出现就是为了解决这个问题。
```

POSUIX简介

```
	POSIX：Portable Operating System Interface 可移植操作系统接口，定义了操作系统应该为应用程序提供的接口标准，是IEEE为要在各种UNIX操作系统上运行的软件而定义的一系列API标准的总称。
 	POSIX为我们提供了统一且强大的接口，方便跨平台开发，POSIX 提供的接口 涵盖了以下内容：系统接口、命令和实用程序、网络文件访问等。
 
 	Linux和windows都要实现基本的posix标准，程序就在源代码级别可移植了。
 
 	用程序员的话来说，接口是一个程序的代码与另一个程序的通信方法。接口期望程序 A 向程序 B 提供特定类型的信息。同样地，程序 A 期望程序 B 用特定类型的数据来回答。
```

当前编码

```
	有了posix规约，程序员们在开发程序的时候不用再关心，同一段程序逻辑功能在进行调用的时候，应该使用哪个平台的哪条底层命令，而只需要使用 基于这些底层平台抽象封装出来的功能函数文件即可，而这个功能函数文件的调用方式 无论在linux平台 还是 windows 平台，都是一样的。
```

### 1.1.3 程序编译

内核

```
	kernel(系统内核)作为操作系统的核心，需要能够正确地识别CPU的类型和特性。Linux内核通过其内置的硬件抽象层来识别这些差异，并据此进行适配。内核是一个由C语言（以及少量汇编语言）编写的、符合POSIX标准的类Unix操作系统内核。
 	从这个方面来说，相较于其他的高阶编程语言来说，c语言是更接近于硬件底层的 通用性的编程语言。几乎所有的底层系统软件、应用软件等都是C语言来写的，性能也是比价好的。
```

```
以C语言为例，程序编译的过程主要有4步：
 	C程序源代码
 		1 --> 预处理
 			2 --> 编译
 				3 --> 汇编
 					4 --> 链接
```

```
预处理（Pre-Processing）
 	1 将所有的#define删除，并且展开所有的宏定义
	2 处理所有的条件预编译指令，比如 #if，#ifdef，#elif，#else，#endif 等
 	3 处理#include 预编译指令，将被包含的文件插入到该预编译指令的位置
 	4 删除所有的注释，//，/**/
 	5 添加行号和文件标识，以便编译时产生调试用的行号及编译错误警告行号
 	6 保留所有的#pragma编译指令，因为编译需要使用它们
```

```
编译（Compiling）
 	编译过程就是把预处理完的文件进行一系列的词法分析，语法分析，语义分析及优化后，最后生成的汇编代码
```

```
汇编（Assembling）
 	汇编是将汇编代码转变成机器码可以执行的命令，每一个汇编语句几乎都对应一条机器指令。汇编相对于编译过程比较简单，根据汇编指令和机器指令的对照表一一翻译即可
```

```
链接（Linking）
 	通过调用链接器ld来链接程序运行需要的一大堆目标文件，以及所依赖的其它库文件，最后生成可执行文件
```

```
分步演示：
    gcc -E hello.c -o hello.i 		#预处理，输出 .i 文件
   		file hello.i
    gcc -S hello.i -o hello.s 		#编译，输出 .s 汇编文件
   		file hello.s
    gcc -c hello.s -o hello.o 		#汇编，输出 .o 目标文件
   		file hello.o
    gcc hello.o -o hello 			#链接，输出可执行文件
   		file hello
   		ldd hello
一步演示：
gcc hello.c -o  hello 				#一步到位
```

注意

```
	这种手动编译的方式，对单个文件来讲没有问题；
 	但实际工作中的商业项目，会有成百上千的源码文件，包括头文件，库文件(库里面又有自定义库，第三方库，标准库)，各种依赖关系，编译的先后顺序等，所以在实际的开发工作中，几乎不会用这种手动编译的方式去编译项目；
```

### 1.1.4 软件连接

类型

```
静态链接：
 	在程序编译时，将程序中使用的所有库文件（包括标准库和用户自定义库）中的代码和数据都 "复制"到最终的可执行文件中。
 	这种方式的优点是程序在运行时不需要额外的库文件支持，但缺点是生成的可执行文件体积较大，且如果库文件更新，需要重新编译整个程序。
```

```
动态链接：
 	在程序编译时，不将库文件中的代码和数据复制到可执行文件中，而是在程序运行时 "由操作系统动态加载" 所需的库文件。
 	这种方式的优点是减少了可执行文件的大小，且库文件更新后不需要重新编译整个程序，但缺点是程序运行时需要依赖外部库文件。
```

```
	已知三个库文件： X(10M) 、 Y(20M)、 Z(30M)。a文件a(10k)运行的时候，依赖 X、Y两个库文件，b文件b(20k)运行的时候，依赖 X、Y、Z三个库文件。
	静态链接
 		a(10k) + X(复制10M) + Y(复制20M) =  30M + 10k
		b(20k) + X(复制10M) + Y(复制20M) + Z(复制30M) =  50M + 20k
 
	动态链接
 		a(10k) + X(指向0M) + Y(指向0M) =  10k
 		b(20k) + X(指向0M) + Y(指向0M) + Z(指向0M) =  20k
```

库文件

模块（库）文件

```
	在Linux系统中，模块库文件通常指的是动态加载的库或模块，这些文件可以被操作系统或应用程序在运行时动态地加载和卸载，以实现功能扩展或更新。
 	Linux环境下有几种主要的模块库文件类型，它们各自服务于不同的目的，但最常被提及和使用的是内核模块（Kernel Modules）（.ko或.o 文件）和共享库（Shared Libraries）（.so文件）。
```

```
	内核模块文件通常以.ko(Kernel Object)或.o(Object)作为文件扩展名，并且位于/lib/modules/$(uname -r)/kernel/目录下的子目录中。加载和卸载内核模块可以使用insmod、rmmod、modprobe等命令进行。
 
 	共享库文件在Linux上一般以.so（Shared Object）作为文件扩展名，它们被存储在/lib、/usr/lib或/usr/local/lib等目录下。动态链接器（Dynamic Linker/Loader）如
ld.so（也称为ld-linux.so）在程序运行时负责将需要的共享库加载到内存中。
 	注意：windows中以.dll 为后缀
```

相关命令

```
查看二进制程序所依赖的库文件
 	ldd 二进制文件
 
加载配置文件中指定的库文件
 	ldconfig
```

管理及查看本机装载的库文件

```
加载配置文件中指定的库文件
ldconfig 
显示本机已经缓存的所有可用库文件名及文件路径映射关系
root@rocky9 ~]# ldconfig -p
```

配置文件

```
/etc/ld.so.conf
/etc/ld.so.conf.d/*.conf
```

缓存文件

```
/etc/ld.so.cache
```

```
关于库文件的动态连接问题
	动态链接
    动态连接器 直接 找 依赖的库文件 ，从而 帮助程序执行
    	在生产环境中，有可能会遇到一种情况：
    	编译安装 A 文件的时候，导致 底层某个 so 文件版本发生变动。
    		编译安装的A文件是可以正常运行的，
			但是 之前编译的可运行的 a b c 等等文件，因为 动态链接的 so文件发生变动
			导致无法正常运行
		解决办法
			重新指定一个文件即可
```



## 1.2 软件包管理

### 1.2.1 软件包介绍

软件包

```
	开源软件最初只提供了.tar.gz的打包的源码文件，用户必须自已编译每个想在GNU/Linux上运行的软件。用户急需系统能提供一种更加便利的方法来管理这些软件，当Debian诞生时，这样一个管理工具dpkg也就应运而生，可用来管理deb后缀的"包"文件。从而著名的“package”概念第一次出现在GNU/Linux系统中，稍后Red Hat才开发自己的rpm包管理系统。
```

```
	Linux系统中的软件包是Linux发行版用来组织、安装和管理软件的一种方式。它们通常以压缩包的形式存在，包含了软件的二进制文件、库文件、配置文件、文档等必要组件。
```

Linux系统中的软件包主要分为以下两类

```
源码包（Source Package）：
    包含软件的源代码文件、编译指令和配置文件。
    需要用户自行编译安装，过程相对复杂，但灵活性高，可以定制安装选项。
    文件格式通常为.tar.gz、.tar.bz2等压缩格式。
    注意：软件运行所有的文件都会在同一个包文件里面
    
二进制包（Binary Package）：
    包含已经编译好的可执行文件、库文件、配置文件、帮助文件等，用户可以直接安装使用。
    常见的二进制包格式有RPM包（Red Hat Package Manager）、DEB包（Debian Package）等。
    注意：软件运行所有的文件分别放到相互依赖的多个包文件里面。
```

二进制包格式

```
RPM包：Redhat Package Manager
    主要在Red Hat、Fedora、CentOS等Linux发行版中使用。
    包名通常包含软件名称、版本号、发布号、适用的Linux发行版标识和架构标识等信息。
    可以通过rpm命令进行安装、卸载、查询等操作。
DEB包：Debian Binary Package
    主要在Debian、Ubuntu等Linux发行版中使用。
    使用dpkg命令进行安装、卸载等操作，但apt命令更为常用，因为它能自动处理依赖关系。
```

二进制包管理工具

```
	Linux系统提供了多种软件包管理工具，用于简化软件包的搜索、安装、升级、卸载等操作，这些包管理工具，主要有以下三种：
apt（Debian/Ubuntu）：
 	用于Debian及其衍生版如Ubuntu中的软件包管理，能够自动处理依赖关系，并提供丰富的软件包仓库。
 
yum（CentOS/RHEL）：
 	CentOS和Red Hat Enterprise Linux（RHEL）等发行版中的软件包管理工具，同样支持自动处理依赖关系和软件包的搜索、安装、升级等操作。
 
dnf（Fedora）：
 	Fedora等发行版中的新一代软件包管理工具，旨在替代yum，提供更快的软件包安装速度和更丰富的功能。
```

查看包的命令

```
Centos系统中：
    预览包内文件：rpm2cpio 包文件|cpio -itv
    释放包内文件：rpm2cpio 包文件|cpio -id ".conf"
        
ubuntu系统中：
    预览包内文件：dpkg -c package.deb
    释放包内文件：dpkg-deb -x package.deb 解压目录
```

```
仅获取文件
 Centos:
 	yum --downloadonly --downloaddir=./ xxx
 Ubuntu:
   	apt-get install -d nginx
 	apt download xxx
```

### 1.2.2包命名

**源码包命名规范**

规范简介

```
	tar源码包本身并不遵循特定的命名规范，因为tar主要用于打包文件，而不是作为软件包的分发格式。tar文件（如example.tar、example.tar.gz、example.tar.bz2等）的命名通常是基于其内容、用途或开发者的习惯来确定的。
 	然而，当提到tar源码包时，通常指的是包含源代码的tar归档文件，这些文件可能还经过了
gzip（.gz）或bzip2（.bz2）等压缩工具的压缩。这类文件的命名规范通常包括以下几个方面：

基础文件名：反映软件包名称或项目名称。
版本号（可选）：有时会在文件名中包含版本号，以区分不同的软件版本，样式：主版本号.次版本号.修正版本号
扩展名：.tar表示tar归档，.gz或.bz2表示压缩格式。
```

源代码打包文件命名

```
打包规则：
    name-VERSION.tar.gz|bz2|xz 
    VERSION: major.minor.release
命名示例：
 	nginx-1.25.4.tar.gz 
 	---
    1 major     	主版本号
 	25 minor     	次版本号
 	4 release   	修正版本号
```

**二进制包命名格式**

规范简介

```
	RPM（Red Hat Package Manager）二进制包遵循统一的命名规则，这种命名规则允许用户通过包名直接获取版本、适用平台等信息。
 	RPM二进制包的命名一般遵循以下格式：
 	包全名 = 包名-版本号-发布次数-发行商-Linux平台-适合的硬件平台-包扩展名
```

二进制打包文件命名

```
命名示例：
 	httpd-2.4.57-15.el9.x86_64.rpm：
 	-----
    httpd：软件包名。
    2.4.57：包的版本号，格式为“主版本号.次版本号.修正号”。
    15：二进制包发布的次数，表示这是第几次编译生成的。
    el9：软件发行商，表示此包是由Red Hat公司发布，适合在RHEL|Centos|Rocky 9.x上使用。
            EL是Red Hat Enterprise Linux（EL）的缩写
    x86_64：表示此包使用的硬件平台。
    rpm：RPM包的扩展名，表明这是编译好的二进制包，可以直接使用rpm命令安装。
```

常见的arch

```
- x86: i386, i486, i586, i686
- x86_64: x64, x86_64, amd64
- powerpc: ppc
- 跟平台无关：noarch
```

包分类

```
软件包为了管理和使用的便利，会将一个大的软件分类，放在不同的子包中。
- Application-VERSION-ARCH.rpm: 主包
- Application-core-VERSION-ARCH.rpm 核心功能包
- Application-devel-VERSION-ARCH.rpm 开发子包
- Application-utils-VERSION-ARHC.rpm 其它子包
- Application-libs-VERSION-ARHC.rpm 其它子包
```

包依赖

```
	软件包之间可能存在依赖关系，甚至循环依赖，即：A包依赖B包，B包依赖C包，C包依赖A包，安装软件包时，会因为缺少依赖的包，而导致安装包失败。
 	关于这些包的依赖关系，我们可以在不同的平台上，借助各自的软件包管理器来实现自动处理。
```

范例：rpm包

```
[root@rocky9 ~]# ls /cdrom/AppStream/Packages/h
haproxy-1.8.27-2.el8.x86_64.rpm
harfbuzz-1.7.5-3.el8.i686.rpm
harfbuzz-1.7.5-3.el8.x86_64.rpm
harfbuzz-devel-1.7.5-3.el8.i686.rpm
harfbuzz-devel-1.7.5-3.el8.x86_64.rpm
harfbuzz-icu-1.7.5-3.el8.i686.rpm
harfbuzz-icu-1.7.5-3.el8.x86_64.rpm
hawtjni-runtime-1.16-1.module+el8.3.0+241+f23502a8.noarch.rpm
hawtjni-runtime-1.16-2.module+el8.3.0+133+b8b54b58.noarch.rpm
HdrHistogram-2.1.11-3.module+el8.4.0+405+66dfe7da.noarch.rpm
```

范例：deb包

```
root@ubuntu24:~# ls /cdrom/pool/main/z/zfs-linux/
libnvpair1linux_0.8.3-1ubuntu12.13_amd64.deb
libuutil1linux_0.8.3-1ubuntu12.13_amd64.deb
libzfs2linux_0.8.3-1ubuntu12.13_amd64.deb
libzpool2linux_0.8.3-1ubuntu12.13_amd64.deb
zfs-initramfs_0.8.3-1ubuntu12.13_amd64.deb
zfsutils-linux_0.8.3-1ubuntu12.13_amd64.deb
zfs-zed_0.8.3-1ubuntu12.13_amd64.deb
```

1.2.3获取软件包

获取软件源码包的地址

```
软件官网、github、第三方软件镜像站
```

获取软件包二进制的地址

```
cdrom、软件官网、github、第三方软件镜像站、自己制作
```

### 1.2.3总结

```
不同的系统平台，以后各自的 软件包，rpm deb
   软件包有配套管理工具 rpm dpkg
   关联的软件包，也有配套的工具 yum  dnf apt
```



## 1.3 包管理器

### 1.3.1 安装

rpm命令介绍

```
命令格式
	rpm {-i|--install} [install-options] PACKAGE_FILE...
		-i 或 --install：用于安装一个或多个 RPM 包。
		PACKAGE_FILE：要安装的 .rpm 文件的路径。
选项
	-q 			# 检查安装软件
 	-ivh 		# 安装软件
 	-evh 		# 卸载软件
 	-Uvh 		# 有旧版程序包，则“升级”，如果不存在旧版程序包，则“安装”
 	-Fvh 		# 有旧版程序包，则“升级”，如果不存在旧版程序包，则不执行安装操作
```

升级注意事项

```
	- 不要频繁对内核做升级操作；
 		Linux支持多内核版本并存，因此直接安装新版本内核
	- 如果原程序包的配置文件安装后曾被修改，
 		升级时，新版本提供的同一个配置文件不会直接覆盖老版本的配置文件，
 		而把旧版本文件重命名(FILENAME.rpmnew)后保留
 		
 一旦确定好系统环境后，就不要再变动  内核环境了
```

卸载注意事项

```
当包卸载时，对应的配置文件不会删除(前提是该文件被改动过)， 以FILENAME.rpmsave形式保留
```

安装演示

```
不需要依赖
[root@rocky9 softs]# rpm -ivh vsftpd-3.0.3-49.el9.x86_64.rpm
需要依赖
[root@rocky9 softs]# rpm -ivh httpd-2.4.57-3.el9.x86_64.rpm
```

升级演示

```
升级新版本vsftpd
[root@rocky9 softs]# rpm -Uvh vsftpd-3.0.5-6.el9.x86_64.rpm
升级旧版本vsftpd
[root@rocky9 softs]# rpm -Uvh vsftpd-3.0.3-49.el9.x86_64.rpm
注意：如果已安装版本高，待安装版本低，默认不安装
```

卸载演示

```
[root@rocky9 softs]# rpm -evh vsftpd
准备中...                          ################################# [100%]
正在清理/删除...
   1:vsftpd-3.0.5-6.el9               ##############################[100%]
   
   
[root@rocky9 softs]# rpm -q vsftpd
未安装软件包 vsftpd
如果存在软件包就卸载
[root@rocky9 softs]# rpm -q vsftpd && rpm -evh vsftpd
```

```
-Fvh	显示某个已安装文件所属的包
[root@rocky9 softs]# rpm -Fvh vsftpd-3.0.3-49.el9.x86_64.rpm

-Uvh	用于升级或安装RPM包
[root@rocky9 softs]# rpm -Uvh vsftpd-3.0.3-49.el9.x86_64.rpm

-q || -evh		查询这个包已安装则删除这个包
[root@rocky9 softs]# rpm -q vsftpd && rpm -evh vsftpd
```

### 1.3.2 包查询

```
格式
	rpm {-q|--query} [select-options] [query-options]
	-q 或 --query：用于查询已安装的软件包。
	[select-options]：选择你要查询的包，通常是包的名字。
	[query-options]：用于定制查询输出的选项。
选项
 	-i 		#information
 	-l 		#查看指定的程序包安装后生成的所有文件
 	-f   	#查看指定的文件由哪个程序包安装生成
 	-d 		#查询程序的文档
 常用组合：-qi、-qf、-ql、-qa
```

```
常用格式
	rpm -qa | grep passwd		#查询已安装的包
	rpm -q vsftpd				#查询是否安装
	rpm -qi tree				#查看包详细信息
	rpm -qi vsftpd				#没有安装，可以指定程序包
	rpm -qf /bin/tree			#查看已安装软件的文件
	rpm -qc NetworkManager		#查看已安装软件的配置文件
	rpm -ql tree				#列出已安装软件的所有文件
	rpm -qd tree				#查看已安装软件的文档文件
	rpm -q --scripts postfix	#查询安装脚本
	rpm -q --changelog tree		#查看软件的更新记录信息
```

### 1.3.3包校验

数据库

rpm包安装时生成的信息，都放在rpm数据库中

```
root@rocky9:web $ ll /var/lib/rpm
```

可以重建数据库

```
rpm {--initdb|--rebuilddb}

initdb 			#初始化，如果事先不存在数据库，则新建之，否则，不执行任何操作
rebuilddb 		#重建已安装的包头的数据库索引目录
```

```
rpm --initdb
作用：初始化 RPM 数据库。此命令通常用于在全新安装的系统上，或者在删除了 RPM 数据库文件之后，重新创建数据库。
用法场景：当你第一次使用 RPM 包管理工具，或者在某些情况下，RPM 的数据库文件损坏需要重新初始化时，使用此命令来创建新的数据库。
```

```
rpm --rebuilddb
作用：重建 RPM 数据库。此命令用于修复或重建损坏的 RPM 数据库。在某些情况下，数据库文件可能会损坏，导致无法查询已安装的软件包，或者 RPM 操作失败。使用 --rebuilddb 可以重建数据库。
用法场景：如果你发现 RPM 命令无法正常工作（例如安装或查询包时出错，或者数据库出现问题），你可以使用这个命令来重建数据库，修复损坏的数据。
```

**校验签名**

在安装包时，系统也会检查包的来源是否是合法的

```
格式：
	rpm -K|checksig rpmfile
```

```
检查从软件源仓库中获取的软件
[root@rocky9 ~]# rpm -K web/nginx-1.20.1-16.el9_4.1.x86_64.rpm
web/nginx-1.20.1-16.el9_4.1.x86_64.rpm: digests signatures 确定
```

```
检测自己从互联网上获取的软件
[root@rocky9 ~]# rpm -K /tmp/softs/vsftpd-3.0.3-49.el9.x86_64.rpm
/tmp/softs/vsftpd-3.0.3-49.el9.x86_64.rpm: digests SIGNATURES 不正确
```

```
查看公钥文件
[root@rocky9 ~]# ls /etc/pki/rpm-gpg/

查看已导入的key信息
[root@rocky9 ~]# rpm -qa "gpg-pubkey"

查看已导入公钥的信息
[root@rocky9 ~]# rpm -qi gpg-pubkey-350d275d-6279464b
```

### 1.3.4总结

```
rpm -i				#安装软件包
rpm -e				#卸载删除安装包
rpm -ql				#列出已安装包的文件列表
```



## 1.4yum和dnf

### 1.4.1软件简介

简介

```
	yum 和 dnf 都是 Linux 系统中用于软件包管理的工具，它们允许用户安装、更新、删除和查询软件包。尽管它们在功能上非常相似，但它们属于不同的软件包管理系统，并且在技术实现和背后的设计上存在一些差异。
	YUM 最初是为基于 RPM 的 Linux 发行版（如 Fedora、CentOS、RHEL 等）设计的。它起源于Yellowdog Linux 发行版，后来经过修改和扩展，成为许多主流 Linux 发行版中不可或缺的一部分。
	DNF 是 Fedora 项目开发的一个新的包管理器，旨在作为 YUM 的继任者。它最初是作为 YUM 的一个分支项目出现的，但随着时间的推移，它逐渐发展成为了一个独立且更加先进的软件包管理工具。
	DNF 提供了更加快速和灵活的包管理体验。它支持并行下载和安装软件包，从而减少了总体等待时间。此外，DNF 还提供了更加丰富的命令行选项和输出格式，使得用户可以更轻松地获取所需的信息。DNF 已经成为 Fedora 和一些其他基于 RPM 的 Linux 发行版的默认包管理器。随着时间的推移，它有望逐渐取代YUM，成为更多 Linux 发行版的标准选择。而且 DNF还保留了和yum的兼容性，配置也是通用的，所以在Rocky linux9里面，他们的操作基本上是一样的。
```

```
查看rocky linux 的两条命令
[root@rocky9 ~]# ll /usr/bin/yum
[root@rocky9 ~]# ll /usr/bin/dnf
```

**工作原理**

构架模式

```
yum/dnf 是基于C/S 模式
    - yum 服务器存放rpm包和相关包的元数据库
    - yum 客户端访问yum服务器进行安装或查询等
```

yum实现过程

```
	先在yum服务器上创建 yum repository（仓库），在仓库中事先存储了众多rpm包，以及包的相关的元数据文件（放置于特定目录repodata下），当yum客户端利用yum/dnf工具进行安装包时，会自动下载repodata中的元数据，查询元数据是否存在相关的包及依赖关系，自动从仓库中找到相关包下载并安装。
```

![image-20241105203302290](D:\桌面\mage.md\笔记\5day-png\yum实现过程.png)

### 1.4.2环境配置

客户端配置

yum客户端配置文件

```
/etc/yum.conf 				#为所有仓库提供公共配置
/etc/yum.repos.d/*.repo 	#为每个仓库的提供配置文件
```

基础命令

```
查看帮助
 man 5 yum.conf
 
获取软件源信息
 yum makecache
 
清理软件源信息
 yum clean all
 
查看仓库的信息
 yum repolist
 yum repolist -v 		# 查看更多信息
```

全局配置

```
root@bogon:web $ ll /etc/yum.conf
lrwxrwxrwx. 1 root root 12 Apr 14  2024 /etc/yum.conf -> dnf/dnf.conf
root@bogon:web $ cat /etc/yum.conf
[main]
gpgcheck=1							#安装包前要做包的合法和完整性校验
installonly_limit=3					#同时可以安装3个包，最小值为2，如设为0或1，为不限制
clean_requirements_on_remove=True	#删除包时，是否将不再使用的包删除
best=True							#升级时，自动选择安装最新版，即使缺少包的依赖
skip_if_unavailable=False			#跳过不可用的
```

repo仓库配置文件指向的定义

```
[repositoryID] 
name=Some name for this repository 	#仓库名称
baseurl=url://path/to/repository/ 	#仓库地址
mirrorlist=http://list/ 			#仓库地址列表，在这里写了多个 baseurl指向的地址
enabled={1|0} 						#是否启用,默认值为1，启用
gpgcheck={1|0} 						#是否对包进行校验，默认值为1 
gpgkey={URL|file://FILENAME} 		#校验key的地址
enablegroups={1|0} 					#是否启用yum group,默认值为 1
failovermethod={roundrobin|priority} #有多个baseurl，此项决定访问规则，
 									#roundrobin 随机，priority:按顺序访问
cost=1000 							#开销，或者是成本，
 									#YUM程序会根据此值来决定优先访问哪个源,默认
为1000
metadata_expire=6h 					#rocky-9中新增配置，metadata 过期时间
countme=1 							#rocky-9中新增配置，默认值false，
 									#附加在mirrorlist之后，便于仓库收集客户端信息
```

baseurl 有多种写法，支持多种协议

```
baseurl=file:///cdrom/AppStream/
baseurl=https://mirrors.aliyun.com/rockylinux/9.4/AppStream/x86_64/os/
baseurl=http://mirrors.aliyun.com/rockylinux/9.4/AppStream/x86_64/os/
baseurl=ftp://10.0.0.159/
注意：
 	yum仓库指向的路径一定必须是repodata目录所在目录
```

常见变量

```
$arch 				#CPU架构 aarch64|i586|i686|x86_64
$basearch 			#系统基本体系结构i386|x86_64
$releasever 		#系统版本
#带变量的写法
mirrorlist=https://mirrors.rockylinux.org/mirrorlist?
arch=$basearch&repo=BaseOS-$releasever
#替换后的值
mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=x86_64&repo=BaseOS-9
结果显示；
 	可以看到，mirrorlist里面所包括的所有的能用的镜像地址列表
 	而且第一个条目都是最新的，离我们最近的一个可用地址
```

RockyLinux国内源

| 来源机构         | 地址                                     |
| ---------------- | ---------------------------------------- |
| 阿里云           | https://mirrors.aliyun.com/rockylinux/   |
| 中国科学技术大学 | http://mirrors.ustc.edu.cn/rocky/        |
| 南京大学         | https://mirrors.nju.edu.cn/rocky/        |
| 上海交通大学     | https://mirrors.sjtug.sjtu.edu.cn/rocky/ |
| 东软信息学院     | http://mirrors.neusoft.edu.cn/rocky/     |

**定制软件源**

Rocky9.4上配置aliyun的repo源

```
[root@rocky9 yum.repos.d]# cat aliyun-baseos.repo 
[aliyun-baseos]
name=aliyun baseos
baseurl=https://mirrors.aliyun.com/rockylinux/9.4/BaseOS/x86_64/os/
gpgcheck=0
```

```
更新软件源
[root@rocky9 yum.repos.d]# yum makecache
查看源
[root@rocky9 yum.repos.d]# yum repolist
查看源信息
[root@rocky9 yum.repos.d]# yum repolist -v
查看指定源信息
[root@rocky9 yum.repos.d]# yum repolist --repoid=aliyun-baseos -v
```

**定制其他源**

```
Rocky9.4上配置南京大学的repo源
[root@rocky9 yum.repos.d]# cat nju-extras.repo
[nju-appstream]
name=nju AppStream
baseurl=https://mirrors.nju.edu.cn/rocky/9.4/AppStream/x86_64/os/
gpgcheck=0
[nju-baseos]
name=nju BaseOS
baseurl=https://mirrors.nju.edu.cn/rocky/9.4/BaseOS/x86_64/os/
gpgcheck=0
[nju-extras]
name=nju extras
baseurl=https://mirrors.nju.edu.cn/rocky/9.4/extras/x86_64/os/
gpgcheck=0
```

Rocky9上配置本地repo源

```
挂载本地光盘到目录
[root@rocky9 ~]# mount /dev/cdrom /mnt/
[root@rocky9 ~]# ls /mnt/
AppStream  BaseOS  EFI  images  isolinux  LICENSE  media.repo
定制专属的本地镜像源信息
[root@rocky9 yum.repos.d]# cat cdrom.repo 
[cdrom-appstream]
name=cdrom appstream
baseurl=file:///mnt/AppStream/
gpgcheck=0
[cdrom-baseos]
name=cdrom baseos
baseurl=file:///mnt/BaseOS/
gpgcheck=0
```

```
注意：
	BaseOS目录
        内容：存储着操作系统的核心组件和基本系统工具，如内核、shell工具、系统服务等。
        功能：提供操作系统的基本功能和支持，确保系统的正常运行。
	AppStream目录
        内容：存储着用户可能需要的应用程序和软件包的元数据信息，以及软件包依赖关系等。
        功能：使用户可以方便地安装和管理这些软件，通常包含用户界面软件、开发工具、数据库工具等应用程序。
 	关联关系：
   		BaseOS和AppStream两个目录之间的关系是互补的。
   		在安装和管理软件时，系统会从这两个目录中获取所需的软件包和依赖关系，以确保系统的完整性和稳定性。
```

缓存信息实践

```
查看缓存日志信息
[root@rocky9 ~]# ls /var/cache/dnf/
清理缓存信息
[root@rocky9 ~]# yum clean all
```

```
yum 命令，默认情况下，不允许在同一个主机上的多个终端里面执行
	解决方法：
    1 删除已知的 yum程序
    2 删除提示出来的  lock 文件
    
软件的安装方式
    rpm --- 手工安装 没有依赖功能
    yum|dnf  --- 工具安装，有依赖功能
    							仅限于 软件本身
yum group  --- 工具安装 ，更强大的通用的包依赖功能
编译环境的通用软件包
安全相关的
开发相关的
```



### 1.4.3yum命令

yum命令用法

```
命令格式：
 yum [options] COMMAND
常用子命令
    autoremove               #卸载包，同时卸载依赖
    clean                    #清除本地缓存
    install                  #包安装
    list                     #列出所有包
    makecache                #重建缓存
    search                   #包搜索，包括包名和描述
    
一般子命令
    check-update             #检查可用更新
    downgrade                #包降级
    group                    #包组相关
    help                     #显示帮助信息
    history                  #显示history
    info                     #显示包相关信息
    reinstall                #重装
    remove                   #卸载
    repolist                 #显示或解析repo源
    search                   #包搜索，包括包名和描述
    
常用选项
 	-y|--assumeyes 			#自动回答为 yes
 
一般选项
    -c file|--config file 	#指定配置文件，默认使用
	/etc/yum.conf
    -v|--verbose 			#显示详细信息
    -b|--best 				#尝试在可用包中寻找最匹配的版本
    --nogpgcheck 			#不进行包校验
    --repo repoid|--repoid repoid #指定repo源
    --enablerepo repoid 	#临时启用repo源，可用通配符
    --disablerepo repoid 	#临时禁用repo源，可用通配符
    --nodocs             	#不安装文档
    --skip-broken 			#跳过有问题的包
    --enable 				#启用源，配合 configmanager
    --disable 				#禁用源，配合 configmanager
    -x package|--exclude package|--excludepkgs package #排除指定包，可用通配符
    --downloadonly 			#只下载，不安装
```

```
注意：要理解哪些是子命令，哪些是选项，哪些是参数
 	yum repolist enabled 等价于 yum repolist --enabled
 	yum repolist --enabled 可以写成  yum --enabled repolist
 	yum repolist enabled 不能写成 yum enabled repolist
```

显示仓库默认列表

```
默认显示所有 enable 的 repo
[root@rocky9 yum.repos.d]# yum repolist
[root@rocky9 yum.repos.d]# yum repolist --all
[root@rocky9 yum.repos.d]# yum repolist --enable
--enable、--set-enabled 和 --disable、--set-disabled 必须和 config-manager 命令一起
使用。
显示启用的源
[root@rocky9 yum.repos.d]# yum repolist --enabled
禁用软件源
[root@rocky9 yum.repos.d]# yum-config-manager --disable nju-extras
显示禁用的源
[root@rocky9 yum.repos.d]# yum repolist --disabled
```

显示程序包

```
查看所有软件
[root@rocky9 yum.repos.d]# yum list
查看所有可更新的包
[root@rocky9 yum.repos.d]# yum list --updates
查看所有以 z 开头的包，包括己安装的和可安装的
[root@rocky9 yum.repos.d]# yum list --all z*
查看所有可用的包
[root@rocky9 yum.repos.d]# yum list --available
查看指定软件可用的包
[root@rocky9 yum.repos.d]# yum list --available sos
显示指定软件可用的 |已安装的 软件包
[root@rocky9 yum.repos.d]# yum list --showduplicates sos
指定源查看可用包
[root@rocky9 yum.repos.d]# yum list --repo=nju-appstream php
查看所有已安装软件
[root@rocky9 yum.repos.d]# yum list --installed
查看指定的已安装软件
[root@rocky9 yum.repos.d]# yum list --installed sos
```

指定repo源查看

```
查看软件
[root@rocky9 yum.repos.d]# yum list tar
指定源查看软件
[root@rocky9 yum.repos.d]# yum list --repo=nju-baseos tar
排除源查看软件
[root@rocky9 yum.repos.d]# yum list --disablerepo=nju-baseos tar
指定多源查看软件
[root@rocky9 yum.repos.d]# yum list --repoid=a* --repoid=nj* tar
```

```
yum 安装软件的命令
    
    1 软件源检查
        repolist
    2 搜软件
        list | search 
    3 安装软件
        install
    4 查看软件
        list
    5 删除软件
        remove 
        autoremove
    6 清理
        clean
        autoclean
```



### 1.4.4安装删除

安装软件 - 自动解决相关依赖问题

```
命令格式：
    yum install [options] PACKAGE [...]
    yum reinstall [options] PACKAGE [...]
常用选项
    --installroot path 					#指定安装目录
    --downloadonly 						#只下载，不安装
    --downloaddir path|--destdir path 	#指定下载目录,如果下载目录不存在，则自动创建
```

安装软件 - 使用本地软件包安装

```
命令格式：
    yum localinstall|install [options] rpmfile1 [...] 	#安装本地RPM包
    yum localupdate|update [options] rpmfile1 [...] 	#使用本地RPM包升级
```

删除软件

```
命令格式：
    yum remove [options] PACKAGE [...]
    yum erase [options] PACKAGE [...]
```

安装软件实践

```
交互方式安装软件 - 需要输入 y 确认
[root@rocky9 yum.repos.d]# yum install httpd
直接安装软件 - 无需询问
[root@rocky9 yum.repos.d]# yum install -y httpd
不安装软件，仅下载软件
[root@rocky9 yum.repos.d]# yum install httpd --downloadonly --downloaddir=/tmp/httpd
```

使用本地rpm文件安装

```
安装本地软件包
[root@rocky9 ~]# yum localinstall web/nginx-1.20.1-16.el9_4.1.x86_64.rpm
```

删除软件实践

```
卸载单个软件
[root@rocky9 yum.repos.d]# yum remove httpd
卸载多个软件
[root@rocky9 yum.repos.d]# yum erase httpd ngnix
```

### 1.4.5其他命令

升级和降级

```
命令格式：
    yum update [options] PACKAGE [...] 		#升级
    yum downgrade [options] PACKAGE [...] 	#降级
    yum check-update 						#检查可用升级
```

查询信息

```
命令格式：
    yum info [options] PACKAGE [...] #查看程序包的 information 信息
    yum provides [options] PROVIDE #查看文件是由哪个包提供  
    yum search [options] KEYWORD #根据关健字搜索，范围包括包名和描述信息
    yum deplist [options] PACKAGE [...] #查询包的依赖
    yum history [options] [info|list|redo|undo] #查看历史记录
    
	/var/log/dnf.log|yum.log
```

软件更新实践

```
列出所有可更新的包
[root@rocky9 yum.repos.d]# yum check-update
检查指定的包是否可更新
[root@rocky9 yum.repos.d]# yum check-update sos
升级指定包
[root@rocky9 yum.repos.d]# yum update sos
```

软件信息查询实践

```
查看制定软件包信息
[root@rocky9 yum.repos.d]# yum info sos
查看己安装的包
[root@rocky96 yum.repos.d]# yum info --installed sos
查看已安装包信息
[root@rocky96 yum.repos.d]# rpm -qi sos
```

查看指定的特性(可以是某文件)是由哪个程序包所提供：

```
查看文件归属于哪个软件
[root@rocky9 yum.repos.d]# yum provides /usr/sbin/sos
指定repo源，查看文件归属于哪个软件
[root@rocky96 yum.repos.d]# yum provides /usr/sbin/nginx --repoid=cdromappstream
```

根据软件名和关键字来搜索

```
根据名字检索软件
[root@rocky9 ~]# yum search redis
根据名字和关键字进行搜索
[root@rocky96 ~]# yum search redis key-value
```

查看指定包所依赖的库和程序

```
[root@rocky96 ~]# yum deplist nginx
```

查看yum历史信息

```
查看yum历史文件
[root@rocky9 yum.repos.d]# ls /var/log/dnf.*
查看yum历史命令
[root@rocky9 yum.repos.d]# yum history
查看指定yum命令信息
[root@rocky9 yum.repos.d]# yum history info 2
查看跟指定软件相关的历史命令
[root@rocky9 yum.repos.d]# yum history nginx
```

### 1.4.6软件组管理

包组管理相关命令

```
常用：
    yum grouplist [options] #列出所有包组
    yum groupinstall [options] group1 [...] #包组安装
    
一般命令：
    yum groupupdate [options] group1 [...] #包组升级
    yum groupremove [options] group1 [...] #包组卸载
    yum groupinfo [options] group1 [...] #包组查询
```

注意

```
涉及到软件包组相关操作的时候，最好使用英文语言
临时调整系统环境变量
[root@rocky9 ~]# echo $LANG
zh_CN.UTF-8
[root@rocky9 ~]# LANG=EN
[root@rocky9 ~]# echo $LANG
EN
```

### 1.4.7自建yum仓库

```
可以根据目录中的 rpm 包生成 repodata 元数据
createrepo [OPTION] <directory_to_index>
createrepo /var/www/html/AppStream/

服务器配置
#安装web服务
yum -y install httpd
#开启httpd服务
systemctl restart httpd
#关闭防火墙
systemctl stop firewalld.service
#关闭防火墙开机自启动
systemctl disable firewalld.service
#删除httpd的原始页面
rm -rf /etc/httpd/conf.d/wecome.conf
rm -rf /etc/httpd/conf.d/autoindex.conf
systemctl restart httpd
#下载EPEL包
yum install -y https://mirrors.aliyun.com/epel/epel-release-latest-8.noarch.rpm
#准备仓库的目录
mount /sev/sr0 /opt
mkdir BaseOS
mkdir AppStream
mkdir EPEL
cp -r /opt/BaseOS/* /var/www/html/BaseOS/
cp /opt/AppStream/Packages/o* /var/www/html/AppStream/ -r
createrepo /var/www/html/AppStream/
createrepo /var/www/html/BaseOS/
createrepo /var/www/html/EPEL/
reposync --repoid=epel -p /var/www/html/EPEL --download-metadata

客户端配置/etc/yum.repo.d/
[BaseOS]
name=BaseOS Repository
baseurl=http://10.0.0.13/BaseOS/BaseOS
gpgcheck=0

[AppStream]
name=AppStream Repository
baseurl=http://10.0.0.13/AppStream
gpgcheck=0

[EPEL]
name=EPEL Repository
baseurl=http://10.0.0.13/EPEL/epel
gpgcheck=0

```





### 1.4.8其他内容

DNF 相关配置文件

```
配置文件
 	/etc/dnf/dnf.conf
仓库文件
	/etc/yum.repos.d/ *.repo
日志
    /var/log/dnf.rpm.log 
    /var/log/dnf.log
使用帮助
 	man dnf
 
注意：
 	yum 程序在安装的过程中，如果被终止，下次再执行将无法解决依赖，DNF可解决此问题
```

yum故障处理

| **yum** **和** **dnf** **失败最主要原因** | **解决方法**                        |
| ----------------------------------------- | ----------------------------------- |
| yum的配置文件格式或路径错误               | 检查/etc/yum.repos.d/*.repo文件格式 |
| yum cache                                 | yum clean all                       |
| 网络不通                                  | 网卡配置                            |



### 1.4.9总结

```
yum| 安装软件的命令
    
    1 软件源检查
        repolist
    2 搜软件
        list | search 
    3 安装软件
        install
    4 查看软件
        list
    5 删除软件
        remove 
        autoremove
    6 清理
        clean
        autoclean
        
个人小技巧：显示指定软件可用|已安装的软件包
    yum list 
            --shouduplicates
            | grep xxx
            
软件安装小技巧：
    对于 Centos|rocky|redhat 默认情况下，安装的软件，不会自动启动
    对于 ubuntu，默认情况下，安装的软件，已经启动，而且是 开机自启动
centos|rocky 本地安装小技巧
    yum localinstall /path/to/localfile.rpm
    yum install /path/to/localfile.rpm
    
软件更新：
    原则上来说，最好后面跟上 要更新的软件名字
    否则的话，就是更新所有可以更新的软件
    
yum makecache | install | remove 
yum makecache 		#yum 会更新本地缓存，确保你可以获取到最新的软件包列表
yum install 		#用于安装软件包
yum remove 			#用于卸载软件包，删除指定的包及其安装时的依赖，如果只希望卸载包而不删除配置文件，使用 remove 即可。
```



## 1.5ubuntu软件管理

### 1.5.1基础知识

软件包管理

```
Debian 软件包通常为预编译的二进制格式的扩展名“.deb”，类似 rpm 文件，因此安装快速，无需编
译软件。包文件包括特定功能或软件所必需的文件、元数据和指令
```

```
dpkg：
 package manager for Debian，类似于rpm， dpkg是基于Debian的系统的包管理器。可以安装，
删除和构建软件包，但无法自动下载和安装软件包或其依赖项
apt：
 Advanced Packaging Tool，功能强大的软件管理工具，甚至可升级整个Ubuntu的系统，基于客户/
服务器架构(c/s)
```

rhel 系列与 debian系列包管理对比

| 操作系统 | 包文件后缀 | 本地包管理器 | 网络包管理器 | 网络包管理工作模式 | 配置文件               |
| -------- | ---------- | ------------ | ------------ | ------------------ | ---------------------- |
| rhrl     | rpm        | rpm          | yum/dnf      | C/S                | /etc/yum.repos.d/*repo |
| debian   | deb        | dpkg         | apt          | C/S                | /etc/apt/sourced.list  |

```
root@ubuntu:~ $ cat /etc/apt/sov //etc/apt/sov /etc/apt/sources.list.d/ubuntu.sources
cat: /etc/apt/sov: No such file or directory
Types: deb
URIs: http://cn.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

配置文件格式说明

```
deb URL section1 section2

#字段说明
deb 		#固定开头，表示是二进制包的仓库，如果deb-src开头，表示是源码库
URL 		#库所在的地址，可以是网络地址，也可以是本地镜像地址
section1 	#Ubuntu版本的代号，可用 lsb_release -sc 命令查看，也可以用 cat 
/etc/os-release
section2 	#软件分类，main完全自由软件 restricted不完全自由的软件，universe社区支持的软件，multiverse非自由软件
section1 	#主仓
section1-backports 	#后备仓，该仓中的软件当前版本不一定支持
section1-security 	#修复仓，主要用来打补丁，有重大漏洞，需要在当前版本中修复时，会放此仓
section1-updates 	#非安全性更新仓，不影响系统安全的小版本迭代放此仓
section1-proposed 	#预更新仓，可理解为新版软件的测试放在此仓，
 					#测试一段时间后会移动到 updates仓或security仓，非专业人士勿用
```

Apt包结构

```
在ubuntu库中，有两个重要目录，分别是dists和 pool
 	dists 目录中存放的是该源仓库中的元数据，包括软件包的名称，适用的架构平台，版本号，依赖关系等  
 	pool 目录中存放的是具体包文件
```

### 1.5.2dpkg包管理

dpkg命令

```
格式
	dpkg [<option> ...] <command>
选项
    -i|--install package.deb 		#安装包
    --unpack package.deb 			#解包
    -V|--verify packageName 		#检查包是否安装
    -l|--list [pattern]       		#列出当前己安装的包，类似于rpm -qa
    -c packageFile 					#列出包内文件，类似于 rpm -qpl
    -L|--listfiles  packageName 	#列出属于指定软件包的文件，类似于 rpm -ql
```

```
检测服务的状态
	systemctl status vsftpd		#将显示 vsftpd 服务的状态信息，包括它是否正在运行、启动的时间、日志输出、服务的 PID 等
	systemctl start vsftpd		#启动 vsftpd 服务
	systemctl restart vsftpd	#重新启动服务会停止当前运行的 vsftpd 服务，然后立即重新启动它
	systemctl reload vsftpd		#重新加载 vsftpd 服务的配置，而无需完全停止和重启服务
	systemctl stop vsftpd		#停止 vsftpd 服务。
	systemctl enable vsftpd		#使 vsftpd 服务在系统启动时自动启动。
	systemctl disable vsftpd	#禁用 vsftpd 服务的自动启动。
```

### 1.5.3 apt命令用法

| **apt** **命令** | **被取代的命令**     | **命令的功能**                 |
| ---------------- | -------------------- | ------------------------------ |
| apt install      | apt-get install      | 安装软件包                     |
| apt remove       | apt-get remove       | 移除软件包                     |
| apt purge        | apt-set purge        | 移除软件包及配置文件           |
| apt update       | apt-get update       | 刷新存储库索引                 |
| apt upgrade      | apt-get upgrade      | 升级所有可升级的软件包         |
| apt autoremove   | apt-get autoremove   | 自动删除不需要的包             |
| apt full-upgrade | apt-get dist-upgrade | 在升级软件包时自动处理依赖关系 |
| apt search       | apt-cache search     | 搜索应用程序                   |
| apt show         | apt-cache show       | 显示安装细节                   |

日志文件

```
/var/log/dpkg.log
```

apt命令

```
格式
	apt [options] command
选项      
    -h|--help     		#显示帮助
    -v|--version 		#显示版本
    -y|--yes 			#自动回答yes
    -q|--quiet 			#安静模式
常见命令
    list       			#根据名称列出软件包
    search     			#搜索软件包描述
    show|info       	#显示软件包细节
    install   			#安装软件包
    remove     			#移除软件包
    autoremove 			#卸载所有自动安装且不再使用的软件包
    update     			#更新可用软件包列表，只更新索引文件，不具体更新软件
    upgrade   			#通过 安装/升级 软件来更新系统
    full-upgrade 		#通过 卸载/安装/升级 来更新系统
    edit-sources 		#编辑软件源信息文件
```

```
当遇到apt install 异常严重报错的时候，可以通过下面的命令来修复。
 	apt --fix-broken install
```

搜索软件

```
在包名和描述信息中搜索
root@ubuntu24:~# apt search nginx
使用正则匹配搜索条件
root@ubuntu24:~# apt search "^ngin*"
仅在包名中搜索
root@ubuntu24:~# apt search --names-only nginx
```

查询包的具体信息

```
查看包的基本信息
root@ubuntu24:~# apt info nginx
显示指定软件的所有版本软件
root@ubuntu24:~# apt show nginx -a
```

安装软件

```
安装一个软件
root@ubuntu24:~# apt install nginx
同时安装多个包
root@ubuntu24:~# apt install nginx redis -y
安装指定版本的包，默认安装最新版

root@ubuntu24:~# apt install nginx=1.14.0-0ubuntu1 
安装nginx 包，如果己存在，则不升级
root@ubuntu24:~# apt install nginx --no-upgrade   
只升级不安装
root@ubuntu24:~# apt install nginx --only-upgrade
```

```
个人实践小技巧：
    有旧版本 安装新版本
        -- 直接覆盖安装
    有新版本，安装旧版本
        -- 不做任何处理
        解决方法：
            - 卸载软件
            - 安装新版本软件
            
安装个人小技巧
    不知道安装哪些软件
    apt install nginx*  ^c
    
    apt-cache madison kubeadm
    yum list --showduplicate
    			列出系统中已知的 kubeadm 包的所有可用版本及其对应的仓库地址
```

卸载软件

```
仅卸载nginx包
root@ubuntu24:~# apt remove -y nginx
卸载所有依赖包
root@ubuntu24:~# apt autoremove nginx
```

更新和升级

```
纺计源中可更新的信息，并不执行具体更新
root@ubuntu24:~# apt update
...
更新所有己安装的包
root@ubuntu24:~# apt upgrade
更新某个具体的包
root@ubuntu24:~# apt install --only-upgrade nginx
```

编辑源

```
命令方式
root@ubuntu24:~# apt edit-sources
文件方式
root@ubuntu24:~# vim /etc/apt/sources.list
```

缓存信息

```
显示当前系统安装包的统计信息，包括己安装的数量，大小，占用空间等
root@ubuntu24:~# apt-cache stats
查看源中指定软件的所有版本
root@ubuntu24:~# apt-cache madison nginx
```

查询依赖

```
查看nginx有哪些依赖
root@ubuntu24:~# apt-cache depends nginx
...
查看nginx被哪些包依赖
root@ubuntu24:~# apt-cache rdepends nginx | head
...
```

几个特殊的有趣命令

```
黑客帝国的屏保效果
root@ubuntu24:~# apt install cmatrix
root@ubuntu24:~# cmatrix
root@ubuntu24:~# cmatrix -C blue | red | yellow

好莱坞大片的屏保效果
root@ubuntu24:~# apt install hollywood
root@ubuntu24:~# hollywood

查看系统信息的大屏
root@ubuntu24:~# apt install screenfetch
root@ubuntu24:~# screenfetch
```

```
卸载软件包
 apt-get purge xxx
删除不再需要的依赖包
 apt-get autoremove
清理残留的配置文件
 apt-get purge $(dpkg -l | grep '^rc' | awk '{print $2}')
```

```

```

```
个人小技巧
    有apt 用apt，不用dpkg
     安装第三方，单独下载的deb包
        - 用dpkg  -i | -r
```

### 1.5.4总结

```
apt remove --purge package_name
apt remove：用于卸载指定的软件包，但会保留包的配置文件，以便日后重新安装时可能会恢复这些配置。
--purge：这个选项用于完全卸载软件包，包括它的配置文件。即使在以后重新安装该软件包，之前的配置文件也不会被恢复

apt 个人小技巧：
apt update			#更新系统中已安装软件包的源列表
apt install			#使用它来安装单个包、多个包，或者指定某个特定版本的包
apt-cache madison	#用于查看特定软件包的可用版本，并显示它们所在的软件源
apt purge 			#用于完全删除软件包及其配置文件
apt autoremove 		#用于自动删除系统中不再需要的依赖包
apt purge $(dpkg -l | grep '^rc' | awk '{print $2}')
					#删除所有已卸载但仍残留配置文件的包
```



## 1.6源码包部署

### 1.6.1源码部署介绍

```
没有包的情况怎么办？
	虽然有很多开源软件将软件打成包，供人们使用，但并不是所有源代码都打成包，如果想使用开源软件，可能需要自已下载源码，进行编译安装。另外即使提供了包，但是生产中需要用于软件的某些特性，仍然需要自行编译安装。但是利用源代码编译安装是比较繁琐的，庆幸的是有相关的项目管理工具可以大大减少编译过程的复杂度。

	1 定制化安装应用  （功能模块、安装路径、等等）
	2 二进制文件包的功能，往往会落后于 源码包文件
	3 其他原因
```

源码如何部署

```
1 部署make编译环境
2 获取项目源代码文件
3 解压软件包
4 编译安装软件
 	- configure定制配置
 		-> make编译生成配置文件
 			-> make install转移文件到安装目录
5 将可执行文件路径加入PATH环境变量
6 测试效果


1 下载
2 解压
3 配置
4 编译
5 转移
6 环境变量

注意：
 	如果没有configure可执行文件
 		- autoconf: 生成confifigure脚本
 	对于大部分的开源软件项目来说，他们的编译工具是 make
 	对于特殊的编程语言来说，比如 java语言项目，编译工具是 ant|maven等专用工具
```
