# 13、linux启动流程和内核管理



```
Linux 启动流程
Linux 系统从启动到完成用户空间的运行，大致可以分为以下几个阶段：

1、BIOS/UEFI 阶段
	BIOS 或 UEFI 初始化硬件（如 CPU、内存、硬盘等）。
	确定启动设备顺序（如硬盘、U盘）。
	将控制权交给引导加载程序（如 GRUB）。
2、引导加载程序（Bootloader）阶段
	常见的引导加载程序有 GRUB、Syslinux 等。
	主要功能：
		加载 Linux 内核到内存。
		提供启动菜单（支持多操作系统）。
		传递内核参数（如 root=/dev/sda1）。
		加载后，将控制权转交给内核。
3、内核初始化阶段
	内核被加载到内存后开始执行：
		初始化硬件设备。
		挂载根文件系统（可能是只读模式）。
		启动第一个用户空间进程，即 init（PID 为 1）。
	内核会加载必要的驱动程序，初始化网络堆栈和进程调度程序。
4、用户空间初始化阶段
	init 系统启动：
		传统：sysvinit。
		现代：systemd（主流发行版使用）。
	启动各个服务（如网络管理、数据库等）：
		Systemd 通过目标（target）管理。
		加载启动脚本（如 /etc/init.d）。
	用户登录界面启动（TTY 或图形界面）。
5、进入多用户环境
	用户可以登录系统并开始交互。

```

```
内核管理
Linux 内核是系统的核心，负责资源管理、进程调度和硬件交互等功能。以下是内核管理的关键点：

1. 内核组成
	内核空间：
		运行内核代码和驱动程序。
	用户空间：
		运行用户进程，不能直接访问硬件。
	模块：
		内核模块（如驱动程序）可以动态加载和卸载。
	常用命令：
		加载模块：modprobe <module> 或 insmod <module.ko>。
		卸载模块：rmmod <module>。
		查看已加载模块：lsmod。
2. 内核版本
	查看内核版本：uname -r。
	版本结构：主版本号.次版本号.修订号（如 5.15.0）。
3. 内核参数管理
	内核启动时接受参数，可通过引导加载程序传递：
		编辑 GRUB 配置：/etc/default/grub。
		更新 GRUB：grub2-mkconfig -o /boot/grub2/grub.cfg。
		常用参数：
			quiet：启动时隐藏消息。
			nomodeset：禁用内核模式设置。
	查看当前内核参数：cat /proc/cmdline。
4. 内核日志
	日志存储在 /var/log/ 或 journalctl。
	查看内核日志：
		dmesg：查看启动时内核信息。
		journalctl -k：查看内核相关日志。
5. 内核更新
	内核更新分为源码编译和软件包安装两种方式。
		源码编译：
			下载源码（如 kernel.org）。
			配置：make menuconfig。
			编译：make。
			安装：make modules_install && make install。
		软件包更新（推荐）：
			Ubuntu：sudo apt install linux-image-<version>。
			Rocky Linux：sudo dnf install kernel-<version>。
6. 动态内核跟踪
	使用工具如 eBPF 和 tracepoint 来监控和优化内核行为。
	其他工具：
		strace：跟踪系统调用。
		perf：性能分析。

```

```
实践建议
	查看启动日志：
		dmesg | less
		journalctl -b
	动态加载模块：
		modprobe <模块名>。
	通过 GRUB 调试启动问题：
		在启动时按 e 编辑启动参数。
	更新内核以获得新特性：
		Ubuntu: sudo apt update && sudo apt upgrade。
		Rocky Linux: sudo dnf update kernel。
```

# 1.1系统启动

#### 1.1.1流程解读

```
1. 加载BIOS的硬件信息，获取第一个启动设备
2. 读取第一个启动设备MBR的引导加载程序(grub)的启动信息
3. 加载核心操作系统的核心信息，核心开始解压缩，并尝试驱动所有的硬件设备
4. 核心执行init程序，并获取默认的运行信息
5. init程序执行/etc/rc.d/rc.sysinit文件，重新挂载根文件系统
6. 启动核心的外挂模块
7. init执行运行的各个批处理文件(scripts)
8. init执行/etc/rc.d/rc.local
9. 执行/bin/login程序，等待用户登录
10. 登录之后开始以Shell控制主机
```

![image-20241121114703790](5day-png\13开机自启动流程.png)

```
一.开机自检（POST）
二.BIOS程序（完成开机自检的程序）
三.按boot第一顺序启动
四.MBR
五./boot里文件的驱动
六.内核文件
七.systemd进程
八.运行级别
九.输入用户名与密码
```

#### 1.1.2硬件启动

BIOS

```
Basic Input and Output System（基本输入输出系统），保存着有关计算机系统最重要的基本输入输出程序，系统信息设置，开机加电自检程序和系统启动自举程序等。
```

```
内部细节
	POST	加点自检
	ROM		只读存储
	RAM		随机存取存
```

```
注意：BIOS 程序无法更改，但是里面的一些属性数据是可以 调整的。
```

系统启动自检程序

```
在BIOS的众多功能中，系统启动自举程序是其核心功能之一。系统启动自举程序是BIOS在完成硬件初始化后，负责搜寻并加载操作系统引导记录的程序。它的主要功能是：
    - 按照RAM中保存的启动顺序，搜寻有效的启动驱动器（如硬盘、光盘驱动器、网络服务器等）。
    - 读入操作系统引导记录。
    - 将系统控制权交给引导记录，由引导记录完成系统的启动。
```

```
所以，对于一台计算机来讲，通电后第一件事件就是运行BIOS程序，BIOS程序最先做的，就是对硬件执行POST（加电自检），如果硬件自检不通过，会显示相应的错误，还会有相应的蜂鸣音。
```

#### 1.1.3启动加载器

定位

```
	对于一台主机来说，它允许在同一台主机上，安装多个操作系统，那么我们如何确定启动哪个系统，这就是bootloader的作用。
```

![image-20241121133925835](5day-png\13.1.1.3 启动加载器.png)

```
Primary Master： 	指的是第一组IDE插槽上的主IDE接口。通常用于连接主硬盘。
Primary Slave： 		指的是第一组IDE插槽上的从IDE接口。通常用于连接从硬盘或光驱等设备。
Secondary Master： 	指的是第二组IDE插槽上的主IDE接口。也可以用于连接硬盘或光驱等设备。
Secondary Slave： 	指的是第二组IDE插槽上的从IDE接口。同样可以用于连接其他IDE设备。
```

```
	如果双系统安装在同一个硬盘上，那么它们可能分别位于Primary Master或Secondary Master的不同分区上。
 	如果双系统安装在不同的硬盘上，那么每个硬盘可能分别连接到Primary Master|Slave、Secondary Master|Slave接口上。在这种情况下，用户同样需要在BIOS的启动顺序设置中指定哪个硬盘或分区应该优先启动。
 	在现代计算机中，双系统通常安装在SATA硬盘的不同分区上，并通过BIOS或UEFI（统一可扩展固件接口）的启动菜单来设置启动顺序。因此，虽然Primary Master、Primary Slave、Secondary Master和Secondary Slave这些术语在现代计算机中可能不再被频繁使用，
```

**bootloader**

```
	Bootloader叫引导加载器，引导程序。它是底层硬件与上层应用软件(操作系统)之间的一个中间接口软件。它不是BIOS中的功能，也不是操作系统中的功能，它是一个独立的软件，运行在BIOS之后，操作系统启动之前。它的主要作用就是引导操作系统启动。
	
	注意：
 		Bootloader 是一个独立软件，可以单独安装，
        	但在一般情况下，安装操作系统时，也会一起安装Bootloader程序
        	不同的操作系统，会安装不同的Bootloader程序
```

**bootloader类型**

```
	Linux的bootloader功能丰富，提供菜单，允许用户选择要启动系统或不同的内核版本；把用户选定的内核装载到内存中的特定空间中，解压、展开，并把系统控制权移交给内核。
 	Linux系统因其开源和灵活性，拥有多种Bootloader选择。每种Bootloader都有其特点和适用场景，选择合适的Bootloader可以提高系统的稳定性和性能。
```

```
GRUB（GRand Unified Bootloader）：
 	Grand 统一启动加载程序,又称多系统引导管理器
 	GRUB是Linux系统中最为常见的Bootloader之一。它支持多种操作系统，包括Linux、Windows、Mac OS等。GRUB具有强大的功能和灵活的配置选项，如通过配置文件进行定制（添加启动项、设置启动顺序等）。
```

#### 1.1.4引导程序

系统启动引导方式有两种，分别是BIOS模式和UEFI模式

**bootloader文件存放**

```
bootloader本身是一个程序，所以需要有物理文件来承载，至于放在哪里，分情况：
 	像ntloader，LILO，GRUB等都是采用这种分段执行的方式。
 	在UEFI模式下,直接由EFI系统分区中的 .efi 引导程序来引导操作系统。
```

```
BIOS里面的 boot 指定我们要启动的位置，0:0 == [第一块磁盘的第一个分区]
```

```
磁盘分区的 0扇区、 1-2047扇区以及2048扇区+ 的分段在硬盘的结构和启动过程中扮演着重要的角色。
0磁道0扇区（512字节）:
 	MBR是主引导记录的简称，它是硬盘的第一个扇区（512字节）的内容，包含了硬盘的:
 		主引导程序[446字节]: Main Boot Loader
 		分区表[64字节]: Disk Partition Table
 		分区结束标志[2字节]: 55 AA,
1-2047：
 	紧随主引导扇区之后，直到第2047扇区结束
 	该区域可能包含一些额外的引导信息或用于特定的硬件或软件需求。
 	为了确保主引导扇区的完整性和安全性，防止因数据写入而破坏主引导扇区的内容。
2048+
 	是实际数据存储的开始位置。
 	简单来说，就是我们的操作系统
```

bootloader的分段存放

| 阶段                  | 阶段顺序  | 存储位置和功能                                               |
| --------------------- | --------- | ------------------------------------------------------------ |
| primary boot loader   | 1st stage | 存储在0磁道0扇区的前446字节空间内                            |
| primary boot loader   | 1.5 stage | 1扇区到2047扇区，存储2阶段的文件系统驱动，保证2nd中的文件可读 |
| secondary boot loader | 2nd stage | hd(0.0)/grub/grub.conf                                       |

```
分段的作用：
	0扇区，通过分区表，获取系统盘在那个分区
	1-2047扇区，常见的文件系统驱动，通过文件系统环境，获取读取os文件的能力
	2048扇区+，根据配置，加载os系统的boot启动文件信息，然后进行系统启动
```

#### 1.1.5Grub功能

grub配置文件

GRUB（GRand Unified Bootloader）是一个启动加载程序，用于启动操作系统。其配置文件在\Linux系统的启动过程中起着至关重要的作用。

```shell
rocky9
/boot/grub2/grub.cfg
ubuntu24-4
/boot/grub/grub.cfg
```



```shell
ubuntu系统启动信息，直接在grub.cfg文件中定制
rocky系统启动信息，放置在另外一个目录下 /boot/loader/entries
root@rocky9:~ # cat /boot/loader/entries/e06bb9928f4147ad9a094f74665290ac-5.14.0-427.13.1.el9_4.x86_64.conf 
title Rocky Linux (5.14.0-427.13.1.el9_4.x86_64) 9.4 (Blue Onyx)
version 5.14.0-427.13.1.el9_4.x86_64
linux /vmlinuz-5.14.0-427.13.1.el9_4.x86_64
initrd /initramfs-5.14.0-427.13.1.el9_4.x86_64.img
options root=/dev/mapper/rl_bogon-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rl_bogon-swap rd.lvm.lv=rl_bogon/root rd.lvm.lv=rl_bogon/swap 
grub_users $grub_users
grub_arg --unrestricted
grub_class rocky

title: 定义了启动项的标题，显示在GRUB菜单中。
version: 显示了内核的版本号，与title中的内核版本一致。
linux: 指定了要启动的内核文件路径。
initrd: 指定了初始化RAM磁盘（initrd）的路径，$tuned_initrd会根据系统配置被替换成特定的路径或选项。
options: 这一行包含了启动内核时传递的参数。这些参数包括：
    root=/dev/mapper/rl-root：指定了根文件系统的设备路径。
    ro：以只读模式挂载根文件系统（通常在启动过程中，一旦系统完全启动，会重新以读写模式挂载）。
    crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M：为kdump服务保留的内存大小，用于在系统崩溃时捕获内核转储。
    resume=/dev/mapper/rl-swap：指定了用于挂起（休眠）功能的交换分区或文件。
    rd.lvm.lv=rl/root 和 rd.lvm.lv=rl/swap：这些选项告诉GRUB在初始化RAM磁盘中查找LVM逻辑卷的路径，用于挂载根文件系统和交换空间。
    rhgb 和 quiet：控制启动过程中的图形反馈（rhgb）和减少启动信息的详细程度（quiet）。
grub_users: 指定了可以使用GRUB的用户，$grub_users是一个变量，其值取决于系统配置。
grub_arg --unrestricted: 这是一个GRUB参数，允许不受限制的访问GRUB命令，通常用于高级用户或系统管理员。
grub_class rocky: 指定了这个GRUB配置属于哪个类，rocky表明这是为Rocky Linux定制的GRUB配置。

```

#### 1.1.6init程序

```
init进程是内核启动的第一个用户级进程，它的进程ID（PID）通常为1。在Linux系统中，init进程负责如下作用：
 	- 系统初始化：
 		init进程在系统启动时执行一系列初始化操作，如设置系统环境、挂载文件系统等。
 		该程序是内核启动之后的第一个进程，也是进程树中的树根，所以其进程ID始终为1。
 	- 启动其他进程：
 		根据系统配置文件（如Linux中的/etc/inittab），init进程会启动其他必要的系统进程和服务。
 	- 监控与管理：
 		init进程还会监控系统中的其他进程，确保它们正常运行，并在必要时进行重启或终止。
```

```
	/sbin/init 或 /lib/systemd/systemd  程序的功能之一就是要挂载整个根文件系统，/sbin/init 或 /lib/systemd/systemd 这个程序的可执行文件，它自己也是在根文件系统里面，

	也就是说，得先运行 /sbin/init 或 /lib/systemd/systemd  程序才有文件目录结构树，但是得先有目录结构，才能通过 /sbin/init 或 /lib/systemd/systemd 这个路径找到可执行文件
```

initrd机制

```
	initramfss（initial ramdisk filesystem）是一个在内存中的"临时根文件系统"，它在Linux内核启动之前被加载到内存中。
 	initramfs包含了内核启动所需的文件系统模块和驱动程序，使得内核能够顺利地加载真正的根文件系统。一旦真正的根文件系统加载完成，initramfs就会被卸载，系统将转移到真正的根文件系统上运行。
```

```
- 提供必要的文件系统支持：
 initramfs中包含了内核启动所需的文件系统模块和驱动程序，它们使得内核能够识别和挂载真正的根文件系统。
 从而定位到 init 程序，从而完在真正的系统启动。
 
- 简化内核启动过程：
 通过提前加载必要的文件系统支持和驱动程序。
 它可以在内核启动的早期提供一个用户态环境，用于完成在内核启动阶段不易完成的工作。
 
- 支持特定的启动需求：
 initramfs可以根据系统的需求进行定制，包含特定的文件系统支持和驱动程序，以满足特定的启动需求。
 
- 提供系统维护和故障排除的功能：
 在系统无法正常启动时，可以通过initramfs进入救援模式进行修复。initramfs中可以包含一些系统维护和故障排除工具，这些工具可以帮助用户诊断和解决系统启动过程中遇到的问题。
```

#### 1.1.7加载内核

```
kernel 自身初始化过程
    1. 探测可识别到的所有硬件设备
    2. 加载硬件驱动程序（借助于ramdisk加载驱动）
    3. 以只读方式挂载根文件系统
    4. 运行用户空间的第一个应用程序：
 注意：
 Centos6及以下版本的操作系统是 /sbin/init
 Centos7及以上版本的操作系统是 /lib/systemd/systemd
```

```
面对 vmlinuz 和 initramfs 的问题，最好的方法是，通过以下方法来解决问题。
 - 重装内核
 - 使用初始的安装盘
 - 找一台相同模式的服务器，提前做好对应的文件，然后通过网络传输到故障主机
```

#### 1.1.8系统启动

| 运行级别 | 说明                                              |
| -------- | ------------------------------------------------- |
| 0        | 关机                                              |
| 1        | 单用户模式(root自动登录)，single，维护模式        |
| 2        | 多用户模式，启动网络功能，但不会启动NFS；维护模式 |
| 3        | 多用户模式，正常模式；文本界面                    |
| 4        | 预留级别；可同3级别                               |
| 5        | 多用户模式，正常模式；图形界面                    |
| 6        | 重启                                              |

runlevel 和 target 之间的对应关系

| 运行级别 | target           | 指向真实文件      |
| -------- | ---------------- | ----------------- |
| 0        | runlevel0.target | poweroff.target   |
| 1        | runlevel1.target | rescue.target     |
| 2        | runlevel2.target | multi-user.target |
| 3        | runlevel3.target | multi-user.target |
| 4        | runlevel4.target | multi-user.target |
| 5        | runlevel5.target | graphical.target  |
| 6        | runlevel6.target | reboot.target     |

只有/lib/systemd/system/*.target文件中AllowIsolate=yes 才能切换(修改文件需执行systemctl daemon-reload才能生效)

```
查看当前的启动模式
root@rocky9:~ # systemctl get-default 
重启对应target
root@rocky9:~ # ll /usr/lib/systemd/system/default.target
lrwxrwxrwx. 1 root root 16 Apr  8  2024 /usr/lib/systemd/system/default.target -> graphical.target
默认target
root@rocky9:~ # ll /usr/lib/systemd/system/ctrl-alt-del.target 
lrwxrwxrwx. 1 root root 13 Apr  8  2024 /usr/lib/systemd/system/ctrl-alt-del.target -> reboot.target

查看级别
root@rocky9:~ # who -r
         run-level 3  2024-11-21 17:18
root@rocky9:~ # runlevel 
N 3
```

切换级别方式

```
手工方式：
    方法1：
        init N
    方法2：
        systemctl set-default
文件方式：
 /etc/inittab
```

```
标准设置
systemctl set-default multi-user.target
```

# 1.2系统初始化

系统启动

```
1. UEFi或BIOS初始化，运行POST开机自检
2. 选择启动设备
3. 引导装载程序， centos7是grub2，加载装载程序的配置文件：`/etc/grub.d/，/etc/default/grub ，/boot/grub2/grub.cfg`
4. 加载initramfs驱动模块（可以实现根文件系统的挂载）
5. 加载虚拟根中的内核
6. 虚拟根的内核初始化，Centos7使用systemd代替init，第一个进程
7. 执行initrd.target 所有单元，包括挂载 /etc/fstab
8. 从initramfs根文件系统切换到磁盘根目录
```

系统初始化

```
9. systemd执行默认target配置，配置文件/etc/systemd/system/default.target
10. systemd执行sysinit.target初始化系统及basic.target准备操作系统
11. systemd启动multi-user.target 下的本机与服务器服务
12. systemd执行multi-user.target 下的/etc/rc.d/rc.local
13. Systemd执行multi-user.target下的getty.target及登录服务
14. systemd执行graphical需要的服务
```

# 1.3Grub管理

```
1 安装GRUB到启动设备
grub2-install /dev/sda
2 生成GRUB配置文件
grub2-mkconfig -o /boot/grub2/grub.cfg
3 重启主机即可
```

Ubuntu解决root密码丢弃方法

```
开机进入到grub界面
按e键，进入内核编辑页面
修改上面的linux开头的行，删除"ro Quiet Splash $vt_handoff"，替换为 rw init=/bin/bash
按ctrl-x或者F10键，进入下面界面
执行命令passwd 直接修改密码后重启即可
```

rocky解决root密码丢弃方法

```
按 e ，进入内核编辑模式，然后修改内核的启动参数
 "ro" 改成 "rw"
 rhgb 后面添加 init=/bin/bash
 根据提示 Ctrl + x 进行保存后启动，按照标准的方式，对root密码进行修改
 
 设置 SElinux 重启标记
 touch /.autorelabel
最后重启系统：
 exec /usr/sbin/init
```

```
为grub设置登录密码
编辑GRUB配置文件
[root@rocky9 ~]# vim /etc/grub.d/40_custom
#!/usr/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries. Simply type the
# menu entries you want to add after this comment. Be careful not to change
# the 'exec tail' line above.
set superusers="grubname"
password grubname 123456
更新GRUB配置
[root@rocky9 ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
```



# 1.4服务管理

```
/usr/lib/systemd/system/ 	#每个服务最主要的脚本文件目录，类似于之前的/etc/init.d/
/run/systemd/system/ 		#系统执行过程中所产生的服务脚本，比上面目录优先运行
/etc/systemd/system/ 		#管理员建立的执行脚本，类似于/etc/rcN.d/Sxx的功能，比上面目录优先运行
```

```
列出所有在内存中的单元
 systemctl 
 systemctl list-units
列出所有己加载到内存且状态为 active 的 service
 systemctl --type=service 
    systemctl list-units --type=service
列出所有己加载到内存的 service
 systemctl list-units --type=service -a
 systemctl --type=service --all
从硬盘中读取数据，列出所有service，包含己加到内存中的数据 
 systemctl list-unit-files --type service --all
```

```
systemctl restart
systrmctl stop
systemctl status
systemctl enable
systemctl disable
systemctl start
systemctl daemon-reload
```



# 1.5 内核参数管理

```
提高系统性能：通过调整内核参数，可以使系统更高效地利用硬件资源，如CPU、内存和I/O等，从而提升整体性能。
增强系统稳定性：合理的内核参数设置可以减少系统崩溃和死机的概率，提高系统的可靠性。
提升系统安全性：适当的内核参数调整可以减少潜在的安全风险，增强系统的安全性。
```

```
命令格式
 	sysctl [options] [variable[=value] ...]
常用选项
    -a|-A|-x|--all #显示所有内核参数
    -p|--load #重载
    -N|--names #仅显示参数名称
    -n|--values #仅显示参数值
    -w|--write #设置内核参数
```

```
可用的配置文件
系统在启动时，会按下列顺序加载配置文件，读取参数值
/etc/sysctl.d/*.conf
/usr/lib/sysctl.d/*.conf 
/lib/sysctl.d/*.conf
/etc/sysctl.conf
```

```
常用内核参数
net.ipv4.ip_forward 			#是否开启ipv4地址转发
net.ipv4.icmp_echo_ignore_all 	#是否禁用ping功能
net.ipv4.ip_nonlocal_bind   	#允许应用程序可以监听本地不存在的IP
vm.drop_caches 					#缓存回收机制 3 回收所有 2 释放数据区和信息节点1 释放页面缓存
fs.file-max = 1020000 			#内核可以支持的全局打开文件的最大数
vm.overcommit_memory = 0 		#超分 0表示进程申请内存时会判断，不够则返回错误，1表示内核允许分配所有的物理内存，而不管当前的内存状态如何，2表示内核允许分配超过所有物理内存和交换空间总和的内存
vm.swappiness = 10           	#使用swap空间时物量内存还有多少可用，10 表示物理存只有10%时才使用swap

#禁用IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```







20250316
