
# Linux操作系统基础

## 计算机硬盘分区简介

> 硬盘分区分为：主磁盘分区、扩展磁盘分区、逻辑分区。  

linux系统下主分区和逻辑分区都可用于存放系统。分出主分区后，其余的部分可以分成扩展分区，一般是剩下的部分全部分成扩展分区，也可以不全分，那剩下的部分就浪费了。  

扩展分区可分成若干逻辑分区，因此逻辑分区是扩展分区的一部分。在linux中第一块硬盘分区为hda（或sda），主分区编号为hda1-4，逻辑分区从5开始。

MBR(主引导记录)的分区表（主分区表）只能存放4个分区，若要分更多的分区的话就要一个扩展分区表（EBR），扩展分区表放在一个系统ID为0x05的主分区上，这个主分区就是扩展分区。

> 一个硬盘主分区至少1个，最多4个；扩展分区最多1个。且主分区+扩展分区总共不能超过4个。

## 系统引导常识

### BIOS

中文名称为基本输入输出系统，它是一组固化到计算机内主板上一个ROM芯片上的程序，它保存着计算机最重要的基本输入输出程序、系统设置信息、开机后自检程序和系统自启动程序。

### MBR

主引导记录是位于磁盘最前边的一段引导代码。它包含了硬盘的一系列参数和一段引导程序（grub）。MBR是由分区程序（如Fdisk）所产生的的，它不依赖于任何操作系统，而且硬盘引导程序也是可以改变的，从而实现多系统共存。

### GPT

名为GUID分区表，全局唯一的标识符，这是一个逐渐取代MBR的新标准。驱动器上的每一个分区都有一个全局唯一标识符。

### GRUB

GNU项目中的一个多操作系统启动程序。GRUB可用于选择操作系统分区上的不同内核，也可用于向这些内核传递启动参数，是一个多重操作系统启动管理器。

## Linux服务器启动流程

1. 加载BIOS

2. 读取MBR；大小是512字节，系统会将其复制到0x7c00地址所在的物理内存空间

3. BootLoader启动，最常见的有Grub

    - 以Grub为例，系统读取内存中的grub配置信息（一般为menu.lst和grub.lst），并依照此配置信息来启动不同的操作系统。

4. 加载内核

5. 用户层init依据inittab文件来设定运行等级

6. init执行rc.sysinit(/etc/rc.d/rc.sysinit)

7. 启动内核模块

    - 依据/etc/modules.conf文件或/etc/modules.d目录下的文件来装载内核

8. 执行不同运行级别的脚本程序（rc0.d~rc6.d）

9. 执行/etc/rc.d/rc.local

    - rc.local是在一切初始化完毕后，Linux留给用户进行个性化的地方

10. 执行/bin/login程序，进入登录状态

## Linux网络相关概念

Linux服务器默认网卡配置文件在/etc/sysconfig/network-scripts/目录下，命名的名称一般为：ifcfg-eth0 ifcfg-eth1，eth0表示第一块网卡，eth1表示第二块网卡，依次类推。

### TCP/IP

译为传输控制协议/因特网互联协议，协议分为四层

### IP地址

> 译为互联网协议地址，是每个主机上的逻辑地址。

IP地址是一个32位的二进制数，采用“点分十进制”的形式表示。IP地址编址方案将IP地址空间划分为A/B/C/D/E/F五类

### 网卡信息配置

以Ubuntu16为例，默认配置文件为/etc/network/interfaces

默认配置为：

```bash
auto lu
iface lo inet loopback
```

修改配置文件的方式有两种

- 静态IP

```bash
auto enp0s3                 # enp0s3代表自己的网卡名称
iface enp0s3 inet static
address 192.168.0.1         # ip地址
netmask 255.255.255.0       # 子网掩码
gateway 192.168.0.1         # 网关
```

- 动态IP

```bash
auto enp0s3             # enp0s3代表自己的网卡名称
iface enp0s3 inet dhcp  
```

修改DNS服务器地址，方式有两种

- 配置/etc/network/interfaces文件，最后增加一句：

```bash
dns-nameservers 223.5.5.5
```

> 重启后DNS就生效了，这时候再看/etc/resolv.conf，最下面就多了一行"nameserver 223.5.5.5"

- 在/etc/resolvconf/resolv.conf.d/目录下的base文件里面，写入下面的命令， 然后重启，DNS生效。

```bash
nameserver 223.5.5.5
```
