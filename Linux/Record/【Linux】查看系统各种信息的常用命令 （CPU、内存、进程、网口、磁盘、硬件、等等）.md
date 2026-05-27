# 【Linux】查看系统各种信息的常用命令 （CPU、内存、进程、网口、磁盘、硬件、等等）

## 系统基本信息：uname

uname命令可以显示系统的基本信息，如内核版本、操作系统名称、主机名、硬件架构等。它有以下常用的选项：

-   -a：显示所有信息
-   -s：显示内核名称
-   -r：显示内核版本
-   -v：显示内核发布日期
-   -o：显示操作系统名称
-   -n：显示主机名
-   -m：显示硬件架构

例如，输入`uname -a`，可以得到类似下面的输出：

```bash
# uname -a
Linux CQUPTLEI 5.4.0-149-generic #166-Ubuntu SMP Tue Apr 18 16:51:45 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

这表示当前系统是Linux，内核版本是5.4.0-149-generic，发布日期是2023年4月18日，硬件架构是x86_64，操作系统名称是GNU/Linux。

## Linux发行版信息：lsb_release

lsb_release命令可以显示Linux发行版的信息，如发行版名称、版本号、代号等。它有以下常用的选项：

-   -a：显示所有信息
-   -d：显示发行版描述
-   -c：显示发行版代号
-   -r：显示发行版版本号

例如，输入`lsb_release -a`，可以得到类似下面的输出：

```bash
# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.6 LTS
Release:        20.04
Codename:       focal
```

这表示当前系统是Ubuntu发行版，版本号是20.04.6 LTS，代号是focal。

>   `LSB`是Linux标准基础（Linux Standard Base）的简称。它是一个由Linux基础设施提供商、应用程序开发者和Linux社区共同制定的标准，旨在提供一致的软件接口和二进制兼容性，以增强不同Linux发行版之间的互操作性。
>   LSB的目标是定义一组核心规范和标准，确保在符合LSB的Linux系统上开发和运行的应用程序能够在不同的Linux发行版上保持一致的行为。这使得应用程序开发者能够更容易地将其软件移植到不同的Linux环境中，而不需要为每个发行版进行额外的定制和调整。
>   LSB标准涵盖了各种方面，包括文件系统布局、共享库、命令工具、系统调用接口、初始化脚本、包管理和日志系统等。它定义了一些基本的命令和工具，如lsb_release，用于查看Linux发行版的信息，以及其他用于检查和验证系统符合LSB标准的工具。
>   通过遵循LSB标准，Linux发行版可以提供更高的互操作性，允许开发者在不同的Linux系统上更轻松地交付和运行应用程序。这也为企业和组织提供了更大的灵活性，使其能够选择不同的Linux发行版，并确保其应用程序能够在这些发行版上正常运行。

## CPU详细信息：lscpu

lscpu 用于显示关于CPU的详细信息。它提供了有关**处理器架构、逻辑核心数、大小端模式、CPU频率、缓存层次结构和支持的特性等**信息。

常见的参数：

-   -a, --all：显示所有可用的CPU信息，包括默认和扩展的信息。
-   -p, --parse：解析/proc/cpuinfo文件并以可读格式显示处理器信息。
-   -s, --socket：只显示物理插座（socket）的信息，包括插座编号、核心数和线程数等。
-   -c, --cpu：只显示逻辑CPU的信息，包括CPU编号、核心编号、线程编号等。
-   -x, --hex：在显示CPU特性和标志时，以十六进制格式显示。
-   -y, --extended=KEY：显示扩展的CPU信息。KEY可以是以下之一：cache，cpu，flags，[top](https://zhida.zhihu.com/search?content_id=230961441&content_type=Article&match_order=1&q=top&zhida_source=entity)ology。
-   -e, --online：只显示在线的CPU的信息，即正在运行的CPU。
-   -V, --version：显示lscpu命令的版本信息。

使用`lscpu`：

```sh
[root@localhost ~]# lscpu
Architecture:                x86_64
  CPU op-mode(s):            32-bit, 64-bit
  Address sizes:             45 bits physical, 48 bits virtual
  Byte Order:                Little Endian
CPU(s):     4
  On-line CPU(s) list:       0-3
Vendor ID:  AuthenticAMD
  BIOS Vendor ID:            AuthenticAMD
  Model name:                AMD Ryzen 9 5900HX with Radeon Graphics
    BIOS Model name:         AMD Ryzen 9 5900HX with Radeon Graphics          CPU @ 3.3GHz
    BIOS CPU family:         2
    CPU family:              25
    Model:  80
    Thread(s) per core:      1
    Core(s) per socket:      1
    Socket(s):               4
    Stepping:                0
    BogoMIPS:                6587.36
    Flags:  fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl xtopology tsc_
            reliable nonstop_tsc cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm cr8_legacy abm sse4a m
            isalignsse 3dnowprefetch osvw topoext ibpb vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero arat umip v
            aes vpclmulqdq rdpid overflow_recov succor fsrm
Virtualization features:
  Hypervisor vendor:         VMware
  Virtualization type:       full
Caches (sum of all):
  L1d:      128 KiB (4 instances)
  L1i:      128 KiB (4 instances)
  L2:       2 MiB (4 instances)
  L3:       64 MiB (4 instances)
NUMA:
  NUMA node(s):              1
  NUMA node0 CPU(s):         0-3
Vulnerabilities:
  Gather data sampling:      Not affected
  Indirect target selection: Not affected
  Itlb multihit:             Not affected
  L1tf:     Not affected
  Mds:      Not affected
  Meltdown: Not affected
  Mmio stale data:           Not affected
  Reg file data sampling:    Not affected
  Retbleed: Not affected
  Spec rstack overflow:      Vulnerable: Safe RET, no microcode
  Spec store bypass:         Vulnerable
  Spectre v1:                Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:                Mitigation; Retpolines; IBPB conditional; STIBP disabled; RSB filling; PBRSB-eIBRS Not affected; BHI Not affected
  Srbds:    Not affected
  Tsa:      Mitigation; Clear CPU buffers
  Tsx async abort:           Not affected
  Vmscape:  Not affected
```

## 内存使用情况：free

free命令可以显示系统的内存使用情况，包括物理内存、交换分区、缓冲区和缓存等。它有以下常用的选项：

-   -h：以人类可读的格式显示信息，如KB、MB、GB等
-   -m：以MB为单位显示信息
-   -g：以GB为单位显示信息
-   -t：显示总计信息

例如，输入`free -h`，可以得到类似下面的输出：

```bash
# free
              total        used        free      shared  buff/cache   available
Mem:        2030144      986392      108608       66064      935144      785052
Swap:       1049596      793048      256548
```

## 系统实时进程状态：top

top命令可以实时地显示系统的进程状态，如CPU占用率、内存占用率、运行时间等。它有以下常用的选项：

-   -u ：只显示指定用户的进程
-   -p ：只显示指定进程ID的进程
-   -c：显示完整的命令行
-   -d ：**设置刷新间隔**，单位为秒

例如，输入`top`，可以得到类似下面的输出：

```sh
[root@localhost ~]# top
top - 23:43:31 up 13 min,  2 users,  load average: 0.22, 0.20, 0.12
Tasks: 195 total,   1 running, 194 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.0 us,  1.9 sy,  0.0 ni, 96.4 id,  0.0 wa,  0.5 hi,  0.2 si,  0.0 st
MiB Mem :   3621.8 total,   3124.6 free,    519.7 used,    198.7 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   3102.1 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   1822 root      20   0   16060   8400   5648 S   1.0   0.2   0:05.40 sshd-session
  39110 root      20   0  231604   5408   3248 R   0.7   0.1   0:00.03 top
    124 root      20   0       0      0      0 I   0.3   0.0   0:00.24 kworker/2:2-events
   2031 root      20   0  232652   6348   3188 S   0.3   0.2   0:01.55 top
  33344 root      20   0   16448   6848   5816 S   0.3   0.2   0:00.02 systemd-userwor
      1 root      20   0   48796  40520  10364 S   0.0   1.1   0:03.19 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd
      3 root      20   0       0      0      0 S   0.0   0.0   0:00.00 pool_workqueue_release
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_gp
      5 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-sync_wq
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-slub_flushwq
      7 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-netns
      8 root      20   0       0      0      0 I   0.0   0.0   0:00.20 kworker/0:0-mm_percpu_wq
      9 root      20   0       0      0      0 I   0.0   0.0   0:00.00 kworker/0:1-xfs-buf/dm-0
     10 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/0:0H-events_highpri
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.00 kworker/u512:0-ipv6_addrconf
     12 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-mm_percpu_wq
     13 root      20   0       0      0      0 I   0.0   0.0   0:00.00 kworker/u51
```

这表示当前系统已经运行了42天，有2个用户登录，平均负载是0.46、0.39、0.26。

共有249个进程，其中一个在运行，248个在睡眠。

CPU的使用率是2.0%用户态，2.2%系统态，95.5%空闲。

内存的使用情况是xxxxxx（看图）。交换分区的使用情况是xxxx。

最后显示了各个进程的信息，如**进程ID、用户、优先级、虚拟内存、物理内存、共享内存、状态、CPU占用率、内存占用率、运行时间、命令等**。

## 网络接口信息：ifconfig

ifconfig命令可以显示和配置网络接口的信息，如**IP地址、子网掩码、广播地址、MAC地址等**。它有以下常用的选项：

-   -a：显示所有接口的信息，包括未激活的
-   -s：只显示摘要信息，不显示详细信息
-   ：只显示指定接口的信息

例如，输入`ifconfig`，可以得到类似下面的输出：

```
[root@localhost ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.117.128  netmask 255.255.255.0  broadcast 192.168.117.255
        inet6 fe80::20c:29ff:fec0:9e91  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:c0:9e:91  txqueuelen 1000  (Ethernet)
        RX packets 470  bytes 47097 (45.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 681  bytes 126643 (123.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

这表示当前系统有2个网络接口。

-   eth0是以太网接口，它的IP地址是[http://xxx.xxx.xxx.xxx](http://xxx.xxx.xxx.xxx)，子网掩码是[http://xxx.xxx.xxx.xxx](http://xxx.xxx.xxx.xxx)，广播地址是[http://xxx.xxx.xxx.xxx](http://xxx.xxx.xxx.xxx)，MAC地址是xx:xx:xx:xx:xx:xx。
-   lo是本地回环接口。

## 网络连接相关信息：netstat

netstat可以显示活动的TCP\UDP连接、监听的端口、路由表、接口统计、多播成员等。

## 无线网络接口信息：iwconfig

显示和配置无线网络接口的信息，包括无线网卡名称、频率和连接状态等。

服务器没有连接无线网，就不展示示例了（懒得切换双系统）。

## 磁盘分区信息：fdisk

列出系统上所有磁盘分区的信息，包括磁盘设备、分区类型和分区大小等。

```
[root@localhost ~]# fdisk -l
Disk /dev/nvme0n1: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual NVMe Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 5ECC7227-D7E7-4424-A812-3CB10616BE19

Device           Start      End  Sectors Size Type
/dev/nvme0n1p1    2048     4095     2048   1M BIOS boot
/dev/nvme0n1p2    4096  2101247  2097152   1G Linux extended boot
/dev/nvme0n1p3 2101248 41940991 39839744  19G Linux LVM


Disk /dev/mapper/rl-root: 17 GiB, 18249416704 bytes, 35643392 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rl-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

## 磁盘使用情况：df

df命令可以显示系统的磁盘使用情况，包括总容量、已用空间、可用空间、使用百分比等。它有以下常用的选项：

-   -h：以人类可读的格式显示信息，如KB、MB、GB等
-   -m：以MB为单位显示信息
-   -g：以GB为单位显示信息
-   -T：显示文件系统类型
-   -a：显示所有文件系统，包括特殊的
-   < directory >：只显示指定目录所在的文件系统

例如，输入`df -hT`，可以得到类似下面的输出：

```sh
[root@localhost ~]# df -hT
Filesystem          Type      Size  Used Avail Use% Mounted on
/dev/mapper/rl-root xfs        17G  4.5G   13G  27% /
devtmpfs            devtmpfs  1.8G     0  1.8G   0% /dev
tmpfs               tmpfs     1.8G     0  1.8G   0% /dev/shm
tmpfs               tmpfs     725M   11M  714M   2% /run
tmpfs               tmpfs     1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/nvme0n1p2      xfs       960M  259M  702M  27% /boot
tmpfs               tmpfs     1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs               tmpfs     363M  4.0K  363M   1% /run/user/0
```

## 系统主机名等信息：hostnamectl

查看和设置系统的主机名，包括主机名、操作系统版本、架构和系统时区等信息。

参数：

1.  status：显示当前系统的主机名、操作系统版本、架构和时区等信息。
2.  set-hostname NAME：设置系统的主机名为指定的名称。需要root权限或使用sudo执行此操作。
3.  set-chassis TYPE：设置系统的机箱类型。可用的类型包括desktop（桌面）、laptop（笔记本电脑）、server（服务器）和vm（虚拟机）等。
4.  set-deployment DEPLOYMENT：设置系统的部署环境。可用的部署环境包括production（生产环境）、development（开发环境）、testing（测试环境）和custom（自定义环境）等。
5.  set-icon-name NAME：设置系统主机的图标名称。该图标名称通常对应于图形用户界面（GUI）中显示的主机图标。
6.  set-location LOCATION：设置系统所在的位置信息。可以是一个自定义的位置字符串，例如办公室名称或机房编号。
7.  set-timezone TIMEZONE：设置系统的时区。需要指定有效的时区标识符，例如"Asia/Shanghai"或"America/New_York"等。
8.  set-volatile BOOL：设置主机名是否为临时（易失性）的。如果设置为"yes"，主机名将在系统重启后重置为默认值。
9.  set-static-hostname NAME：设置系统的静态主机名。静态主机名在系统重启后保持不变。
10.  set-pretty NAME：设置一个用于美化目的的主机名。可以是一个友好的名称，用于在特定环境中显示给用户。
11.  set-transient-hostname NAME：设置系统的临时主机名。临时主机名在系统重启后重置为默认值。

例：`hostnamectl status`

```sh
[root@localhost ~]# hostnamectl status
     Static hostname: localhost
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: cc1ade0699734e5897c7baa615acce69
             Boot ID: 270b5dfd7bc24a75ae98ae1c51d9f42f
        Product UUID: 18364d56-a77c-190e-c841-4d2ec0c09e91
        AF_VSOCK CID: 3233848977
      Virtualization: vmware
    Operating System: Rocky Linux 10.1 (Red Quartz)
         CPE OS Name: cpe:/o:rocky:rocky:10::baseos
      OS Support End: Thu 2035-05-31
OS Support Remaining: 9y 3d
              Kernel: Linux 6.12.0-124.52.1.el10_1.x86_64
        Architecture: x86-64
     Hardware Vendor: VMware, Inc.
      Hardware Model: VMware Virtual Platform
     Hardware Serial: VMware-56 4d 36 18 7c a7 0e 19-c8 41 4d 2e c0 c0 9e 91
    Firmware Version: 6.00
       Firmware Date: Tue 2026-02-17
        Firmware Age: 3month 1w 1d
```

## PCI设备信息：lspci

lspci是一个用于显示系统中所有PCI总线和连接到它们的所有设备的信息的命令。默认情况下，它显示一个简要的设备列表。

>   这里：PCI的意思是Peripheral Component Interconnect，它是一种个人电脑总线，用于连接主板上的各种外围设备，如显卡、声卡、网卡等。

你可以使用以下一些常用参数来请求更详细的输出或者适合其他程序解析的输出 ：

-   -m：以向后兼容的机器可读的格式显示输出

-   -mm：以机器可读的格式显示输出，便于脚本解析

-   -t：以树状图的形式显示输出，包括所有总线、桥、设备和它们之间的连接

-   -v：显示详细的输出，包括设备类别、供应商、子系统、IRQ等

-   -vv：显示更详细的输出，包括能力列表、PCI配置空间等

-   -vvv：显示最详细的输出，包括所有可解析的信息，即使看起来不太有趣（例如，未定义的内存区域）

-   -k：显示每个设备的内核驱动程序和模块

-   -x：以十六进制格式显示标准部分的PCI配置空间（前64字节或者对于CardBus桥是前128字节）

-   -xxx：以十六进制格式显示整个PCI配置空间（256字节）。这个选项只有root用户才能使用，因为一些PCI设备在你试图读取某些部分的配置空间时会出错（这个行为可能不违反PCI标准，但至少很愚蠢）。不过，这样的设备很少见，所以你不必太担心。

-   -xxxx：以十六进制格式显示扩展的（4096字节）PCI配置空间，这个空间在PCI-X 2.0和PCI Express总线上可用。

-   -b：以总线中心视图显示所有IRQ号和地址，而不是内核看到的那样。

-   -D：始终显示PCI域号。默认情况下，在只有域0的机器上，lspci会抑制它们。

-   -P：通过每个桥的路径来识别PCI设备，而不是通过总线号。

-   -PP：通过每个桥的路径来识别PCI设备，同时显示总线号和设备号。

-   -n：以数字形式显示PCI供应商和设备代码，而不是在PCI ID列表中查找它们。

-   -nn：以数字和名称的形式显示PCI供应商和设备代码。

-   -q：如果在本地pci.ids文件中找不到某个设备，则使用DNS查询中央PCI ID数据库，并将结果保存在本地缓存中。如果DNS查询成功，则在后续运行中即使没有给出这个选项也会识别出结果。请只在自动化脚本中谨慎使用这个选项，以避免过载数据库服务器。

-   -qq：无论是否在本地pci.ids文件中找到某个设备，都使用DNS查询中央PCI ID数据库，并重置本地缓存。

-   -Q：即使在本地pci.ids文件中找到某个设备，也使用DNS查询中央PCI ID数据库。如果你怀疑显示的条目是错误的，请使用这个选项。

-   ```sh
    [root@localhost ~]# lspci
    00:00.0 Host bridge: Intel Corporation 440BX/ZX/DX - 82443BX/ZX/DX Host bridge (AGP disabled) (rev 01)
    00:01.0 PCI bridge: Intel Corporation 440BX/ZX/DX - 82443BX/ZX/DX AGP bridge (rev 01)
    00:07.0 ISA bridge: Intel Corporation 82371AB/EB/MB PIIX4 ISA (rev 08)
    00:07.1 IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)
    00:07.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 08)
    00:07.7 System peripheral: VMware Virtual Machine Communication Interface (rev 10)
    00:0f.0 VGA compatible controller: VMware SVGA II Adapter
    00:11.0 PCI bridge: VMware PCI bridge (rev 02)
    00:15.0 PCI bridge: VMware PCI Express Root Port (rev 01)
    00:15.1 PCI bridge: VMware PCI Express Root Port (rev 01)
    00:15.2 PCI bridge: VMware PCI Express Root Port (rev 01)
    00:15.3 PCI bridge: VMware PCI Express Root Port (rev 01)
    00:15.4 PCI bridge: VMware PCI Express Root Port (rev 01)
    ```
## USB设备信息：lsusb

列出连接到系统的所有USB设备的信息，包括设备ID、制造商和设备速度等。

常用参数 ：

-   -v：显示详细的信息，包括设备类别、供应商、子系统、配置描述符等
-   -t：以树状图的形式显示输出，包括所有总线、设备和它们之间的连接
-   -s [ [ bus]: ] [ devnum]：只显示指定总线和/或设备号的设备。两个编号都是十进制的，可以省略。
-   -d [ vendor]: [ product]：只显示指定供应商和产品ID的设备。两个ID都是十六进制的。
-   -D device：不扫描/dev/bus/usb目录，而是只显示给定设备文件的信息。设备文件应该类似于/dev/bus/usb/001/001。这个选项显示详细信息，类似于-v选项；你必须是root用户才能使用这个选项。
-   -V：打印版本信息并成功退出。

```
[root@localhost ~]# lsusb
Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

[root@localhost ~]# lsusb -v

Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Negotiated speed: Full Speed (12Mbps)
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               1.10
  bDeviceClass            9 Hub
  bDeviceSubClass         0 [unknown]
  bDeviceProtocol         0 Full speed (or root) hub
  bMaxPacketSize0        64
  idVendor           0x1d6b Linux Foundation
  idProduct          0x0001 1.1 root hub
  bcdDevice            6.12
  iManufacturer           3 Linux 6.12.0-124.52.1.el10_1.x86_64 uhci_hcd
  iProduct                2 UHCI Host Controller
  iSerial                 1 0000:02:00.0
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0019
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0
    bmAttributes         0xe0
      Self Powered
      Remote Wakeup
    MaxPower                0mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         9 Hub
      bInterfaceSubClass      0 [unknown]
      bInterfaceProtocol      0 Full speed (or root) hub
      iInterface              0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0002  1x 2 bytes
        bInterval             255
Hub Descriptor:
  bLength               9
  bDescriptorType      41
  nNbrPorts             2
  wHubCharacteristic 0x000a
    No power switching (usb 1.0)
    Per-port overcurrent protection
  bPwrOn2PwrGood        1 * 2 milli seconds
  bHubContrCurrent      0 milli Ampere
  DeviceRemovable    0x00
  PortPwrCtrlMask    0xff
 Hub Port Status:
   Port 1: 0000.0100 power
   Port 2: 0000.0100 power
Device Status:     0x0001
  Self Powered

Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Negotiated speed: High Speed (480Mbps)
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            9 Hub
  bDeviceSubClass         0 [unknown]
  bDeviceProtocol         0 Full speed (or root) hub
  bMaxPacketSize0        64
  idVendor           0x1d6b Linux Foundation
  idProduct          0x0002 2.0 root hub
  bcdDevice            6.12
  iManufacturer           3 Linux 6.12.0-124.52.1.el10_1.x86_64 ehci_hcd
  iProduct                2 EHCI Host Controller
  iSerial                 1 0000:02:01.0
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0019
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0
    bmAttributes         0xe0
      Self Powered
      Remote Wakeup
    MaxPower                0mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         9 Hub
      bInterfaceSubClass      0 [unknown]
      bInterfaceProtocol      0 Full speed (or root) hub
      iInterface              0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0004  1x 4 bytes
        bInterval              12
Hub Descriptor:
  bLength               9
  bDescriptorType      41
  nNbrPorts             6
  wHubCharacteristic 0x000a
    No power switching (usb 1.0)
    Per-port overcurrent protection
  bPwrOn2PwrGood       10 * 2 milli seconds
  bHubContrCurrent      0 milli Ampere
  DeviceRemovable    0x00
  PortPwrCtrlMask    0xff
 Hub Port Status:
   Port 1: 0000.0100 power
   Port 2: 0000.0100 power
   Port 3: 0000.0100 power
   Port 4: 0000.0100 power
   Port 5: 0000.0100 power
   Port 6: 0000.0100 power
Device Status:     0x0001
  Self Powered
```
## 系统硬件详细信息：dmidecode

显示有关系统硬件（如主板、BIOS、内存、处理器等）的详细信息。

dmidecode是一个用于解析系统的DMI（也称为SMBIOS）表内容并以人类可读的格式显示的命令。DMI表包含了系统硬件组件的描述，以及一些其他有用的信息，如序列号和BIOS版本。你可以使用以下一些常用参数来控制输出：

-   -d, --dev-mem FILE：从指定的设备文件读取内存（默认是/dev/mem）
-   -h, --help：显示帮助信息并退出
-   -q, --quiet：显示更简洁的输出，不显示未知、非活动和OEM特定的条目
-   -s, --string KEYWORD：只显示指定关键字对应的DMI字符串的值。关键字必须是以下列表中的一个：bios-vendor, bios-version, bios-release-date, system-manufacturer, system-product-name, system-version , system-serial-number, system-uuid, baseboard-manufacturer, baseboard-product-name, baseboard-version , baseboard-serial-number, baseboard-asset-tag, chassis-manufacturer, chassis-type, chassis-version , chassis-serial-number, chassis-asset-tag, processor-family, processor-manufacturer, processor-version , processor-frequency。
-   -t, --type TYPE：只显示指定类型的DMI条目。类型可以是一个数字，或者一个逗号分隔的数字列表，或者一个数字范围，如0-4。类型也可以是以下关键字之一：bios, system, baseboard, chassis, processor, memory, cache, connector, slot。
-   -u：显示未解析的条目内容，以十六进制格式。
-   -V：打印版本信息并成功退出。

```
[root@localhost ~]# dmidecode | head -n 28
# dmidecode 3.6
Getting SMBIOS data from sysfs.
SMBIOS 2.7 present.
620 structures occupying 29956 bytes.
Table at 0x000E0010.

Handle 0x0000, DMI type 0, 24 bytes
BIOS Information
        Vendor: Phoenix Technologies LTD
        Version: 6.00
        Release Date: 02/17/2026
        Address: 0xEA480
        Runtime Size: 88960 bytes
        ROM Size: 64 kB
        Characteristics:
                ISA is supported
                PCI is supported
                PC Card (PCMCIA) is supported
                PNP is supported
                APM is supported
                BIOS is upgradeable
                BIOS shadowing is allowed
                ESCD support is available
                Boot from CD is supported
                Selectable boot is supported
                EDD is supported
                Print screen service is supported (int 5h)
                8042 keyboard services are supported (int 9h)
```
