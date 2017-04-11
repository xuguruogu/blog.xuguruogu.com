---
title: linux 查看系统硬件信息
date: 2017-03-30 16:12:49
tags: linux
---

- cpu
lscpu命令，查看的是cpu的统计信息.

    Architecture:          x86_64          #cpu架构
    CPU op-mode(s):        32-bit, 64-bit
    Byte Order:            Little Endian   #小尾序
    CPU(s):                48              #总共有48核
    On-line CPU(s) list:   0-47
    Thread(s) per core:    2               #每个cpu核，支持2个线程，即支持超线程
    Core(s) per socket:    12              #每个cpu，有12个核
    Socket(s):             2               #总共有2一个cpu
    NUMA node(s):          1               #没有开启MUMA
    Vendor ID:             GenuineIntel    #cpu产商 intel
    CPU family:            6
    Model:                 63
    Stepping:              2
    CPU MHz:               2294.719
    BogoMIPS:              4589.37
    Virtualization:        VT-x            #支持cpu虚拟化技术
    L1d cache:             32K
    L1i cache:             32K
    L2 cache:              256K
    L3 cache:              30720K
    NUMA node0 CPU(s):     0-47

- 磁盘
查看硬盘和分区分布

    # lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0 278.5G  0 disk 
    ├─sda1   8:1    0 953.7M  0 part /boot
    ├─sda2   8:2    0    54G  0 part /
    ├─sda3   8:3    0   7.5G  0 part 
    ├─sda4   8:4    0 111.8G  0 part /tmp
    └─sda5   8:5    0 104.3G  0 part /var/log
    sdb      8:16   0   2.2T  0 disk 
    └─sdb1   8:17   0   2.2T  0 part /home

- 网卡
查看网卡硬件信息

    # lspci | grep -i 'eth'
    02:00.0 Ethernet controller: Intel Corporation Ethernet Controller 10-Gigabit X540-AT2 (rev 01)
    02:00.1 Ethernet controller: Intel Corporation Ethernet Controller 10-Gigabit X540-AT2 (rev 01)


如果要查看某个网络接口的详细信息，例如eth0的详细参数和指标

    # ethtool eth0
    Settings for eth0:
        Supported ports: [ TP ]
        Supported link modes:   100baseT/Full 
                                1000baseT/Full 
                                10000baseT/Full #支持万兆全双工模式
        Supported pause frame use: No
        Supports auto-negotiation: Yes #支持自适应模式，一般都支持
        Advertised link modes:  100baseT/Full 
                                1000baseT/Full 
                                10000baseT/Full 
        Advertised pause frame use: No
        Advertised auto-negotiation: Yes #默认使用自适应模式
        Speed: 10000Mb/s #现在网卡的速度是10000Mb,网卡使用自适应模式
        Duplex: Full  #全双工
        Port: Twisted Pair
        PHYAD: 0
        Transceiver: external
        Auto-negotiation: on
        MDI-X: Unknown
        Supports Wake-on: umbg
        Wake-on: g
        Current message level: 0x00000007 (7)
                    drv probe link
        Link detected: yes #表示有网线连接，和路由是通的

查看bios信息

    # dmidecode -t bios

    # dmidecode 2.12
    SMBIOS 2.8 present.

    Handle 0x001D, DMI type 0, 24 bytes
    BIOS Information
        Vendor: Insyde Corp.
        Version: 1.57
        Release Date: 08/11/2015
        Address: 0xE0000
        Runtime Size: 128 kB
        ROM Size: 16384 kB
        Characteristics:
            PCI is supported
            BIOS is upgradeable
            BIOS shadowing is allowed
            Boot from CD is supported
            Selectable boot is supported
            EDD is supported
            Japanese floppy for NEC 9800 1.2 MB is supported (int 13h)
            Japanese floppy for Toshiba 1.2 MB is supported (int 13h)
            5.25"/360 kB floppy services are supported (int 13h)
            5.25"/1.2 MB floppy services are supported (int 13h)
            3.5"/720 kB floppy services are supported (int 13h)
            3.5"/2.88 MB floppy services are supported (int 13h)
            8042 keyboard services are supported (int 9h)
            CGA/mono video services are supported (int 10h)
            ACPI is supported
            USB legacy is supported
            BIOS boot specification is supported
            Targeted content distribution is supported
            UEFI is supported
        BIOS Revision: 1.0

    Handle 0x0026, DMI type 13, 22 bytes
    BIOS Language Information
        Language Description Format: Long
        Installable Languages: 2
            en|US|iso8859-1
            zh|CN|unicode
        Currently Installed Language: en|US|iso8859-1

dmidecode以一种可读的方式dump出机器的DMI(Desktop Management Interface)信息。这些信息包括了硬件以及BIOS，既可以得到当前的配置，也可以得到系统支持的最大配置，比如说支持的最大内存数等。

如果要查看所有有用信息

    dmidecode -q
